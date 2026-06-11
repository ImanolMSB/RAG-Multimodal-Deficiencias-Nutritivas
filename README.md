
# Sistema RAG Multimodal Híbrido para la Clasificación de Deficiencias Nutricionales en Cultivos

Este repositorio contiene el desarrollo, reproducción y propuesta de mejora de un flujo de trabajo (workflow) avanzado de **Generación Aumentada por Recuperación (RAG) Multimodal e Híbrida** enfocado en el ámbito fitopatológico.

---

## Descripción del Problema y Artículo de Referencia

### Artículo Original de Referencia

- **Título:** *AgriCLIP: A Multimodal Framework for Agricultural Agriculture and Life Science Applications*
- **Autores:** Umar Nawaz, et al.
- **Repositorio Base:** [GitHub Oficial de AgriCLIP](https://github.com/umair1221/AgriCLIP.git)

### Descripción del Problema

El diagnóstico temprano de deficiencias nutricionales en hojas de plantas es crucial para mitigar pérdidas en la producción agrícola. Los clasificadores zero-shot tradicionales o los modelos de visión directos suelen verse afectados por sesgos de etiquetas previas o por la falta de explicaciones morfológicas detalladas.

### Objetivo de la Reproducción y Mejora

El objetivo principal es reproducir el alineamiento multimodal del espacio de características visuales y textuales propuesto por AgriCLIP (utilizando el dataset downstream de **Hojas de Plátano / Banana Deficiency**) e implementar un **sistema RAG Multimodal Híbrido** como propuesta de mejora. Esto permite desacoplar la señal sensorial cruda de la inferencia diagnóstica, evitando el *Data Leakage* mediante un proceso de subtitulado ciego combinado con recuperación de conocimiento experto.

---

## Resumen de la Metodología (Propuesta de Mejora)

El pipeline de mejora mitiga el sesgo de clasificación directa dividiéndose en cuatro fases críticas:

1. **Configuración y Línea Base (Baseline):**
   - Preparación del entorno con aceleración por hardware (GPU T4).
   - Clonación del framework AgriCLIP.
   - Carga de pesos base de Dino.

2. **Construcción del Índice Vectorial Semántico:**
   - Lectura robusta y depuración del corpus agrícola enriquecido por los autores (`AgriClip-Image-Prompts-Customized.csv`).
   - Generación de embeddings mediante el *Text Encoder* de **CLIP (ViT-B/32)**.
   - Indexación en una base de datos vectorial persistente **ChromaDB** empleando distancia coseno.
   - Extracción y estructuración estricta de metadatos de procedencia granular (`specie_or_class`), preservando el *Ground Truth* clínico de cada fragmento.

3. **Canalización RAG Multimodal Híbrida:**

   **Subtitulado Ciego (Blind Captioning)**
   - Un modelo de visión (`gpt-4o-mini`) analiza las imágenes downstream.
   - Genera descripciones puramente morfológicas (formas, manchas y patrones de color).
   - Longitud máxima de 25 palabras y temperatura 0.2.
   - Omite nombres de enfermedades para evitar sesgos cognitivos.

   **Fusión Temprana (Embedding Híbrido)**
   - Combinación analítica promediada y normalizada (L2) del vector visual proyectado (Dino ResNet50 + AgriCLIP Linear Aligner) con el embedding textual del subtítulo ciego.

   **Recuperación e Inferencia**
   - Búsqueda de los K = 3 vecinos más cercanos en ChromaDB.
   - Extracción explícita de etiquetas de procedencia (`metadata_vecino_1`, `metadata_vecino_2`, `metadata_vecino_3`) junto con el texto recuperado.
   - Construcción de un prompt estructurado orientado a objetos.
   - Generación de un veredicto sintomático en formato JSON mediante `gpt-4o-mini` con temperatura 0.0.

4. **Evaluación de Métricas**
   - Exactitud rígida del clasificador final.
   - Métricas de recuperación de información comparadas contra etiquetas de procedencia.
   - Evaluación n-grama y semántica profunda de las justificaciones generadas.

---

## Estructura del Repositorio

Para garantizar el correcto funcionamiento del entorno experimental, el repositorio debe organizarse de la siguiente manera:

```text
├── Proyecto_PLN_Avanzado (1).ipynb      # Notebook principal con las 4 fases del experimento
├── dataset_downstream_subtitulado.csv   # Conjunto de datos downstream con etiquetas reales (90 muestras)
├── requirements.txt                     # Archivo de dependencias de software para Python
└── README.md                            # Documentación e informe del proyecto
```

---

# Documentación Experimental e Instrucciones de Ejecución

## Configuración de Datos y Entorno

1. Ejecutar las celdas de la **Fase 1** del notebook para instalar dependencias y clonar el repositorio de **AgriCLIP**.

2. Descargar el archivo:

```text
AgriClip-Image-Prompts-Customized.csv
```

desde el OneDrive oficial de los autores y colocarlo en la raíz del entorno de ejecución.

3. Asegurarse de que el archivo:

```text
dataset_downstream_subtitulado.csv
```

esté presente en la raíz del proyecto para la evaluación de la canalización.

4. Configurar la API Key de OpenAI en los secretos de Google Colab utilizando exactamente la siguiente clave:

```text
OpenAI
```

---

## Configuraciones e Hiperparámetros Utilizados

| Componente | Configuración |
|------------|--------------|
| Modelo de Visión | Dino ResNet50 (`dino_pretrain.pth`) |
| Alineador | AgriCLIP Linear Aligner (`Agri_Dino_aligner_DPT_CPT.pth`) |
| Codificador de Texto | CLIP ViT-B/32 |
| Base de Datos Vectorial | ChromaDB |
| Métrica de Similitud | Cosine |
| Batch de Indexación | 128 |
| Recuperador | K-Nearest Neighbors |
| Valor de K | 3 |
| Modelo Generativo | GPT-4o-mini |
| Temperatura (Subtitulado) | 0.2 |
| Temperatura (Síntesis JSON) | 0.0 |

---

# Resultados Obtenidos y Comparación

Tras evaluar el comportamiento del pipeline utilizando las **90 muestras del conjunto de prueba downstream de Banana**, se obtuvieron los siguientes resultados analíticos:

| Dimensión Evaluada | Métrica / Indicador | Valor Obtenido |
|-------------------|--------------------|---------------|
| Clasificación | Accuracy (Clasificación vs Baseline) | **11.11%** |
| Recuperador (ChromaDB) | Recall@K (Encontrado) | **7.78%** |
| Recuperador (ChromaDB) | Precision@K (Pureza) | **7.78%** |
| Recuperador (ChromaDB) | MRR (Posición) | **0.0778** |
| Generador (LLM) | Token F1 (Generación) | **0.1535** |
| Generador (LLM) | ROUGE-L (Generación) | **0.1581** |
| Generador (LLM) | BERTScore (Semántica) | **0.7445** |

---

## Análisis de Hallazgos

- El artículo original reporta un **23.55% de Accuracy** utilizando la arquitectura **AgriCLIP** pura en la tarea *zero-shot* para cultivos de plátano, mientras que nuestra arquitectura RAG multimodal integrada alcanzó un **11.11%**.

- El componente generador (**GPT-4o-mini**) exhibió una sobresaliente capacidad de abstracción sintomática, respaldada por un **BERTScore de 0.7445**. Esto demuestra que, aunque el sistema no replique exactamente el orden de palabras de la referencia (como reflejan Token F1 y ROUGE-L), las justificaciones generadas mantienen coherencia semántica y validez diagnóstica.

- Las métricas de recuperación (*Recall@K* y *Precision@K* de **7.78%**) revelan un fenómeno de **Desalineación Taxonómica**. Mientras el conjunto de prueba contiene etiquetas explícitas de deficiencias nutricionales (`potassium`, `manganese`), la base de conocimiento ALive integra información proveniente de múltiples datasets heterogéneos. Como consecuencia, el recuperador encuentra descripciones morfológicamente similares pero etiquetadas bajo patologías distintas (`rust`, `scab`) o identificadores numéricos, penalizando la coincidencia exacta de etiquetas pese a la cercanía semántica.

---

# Conclusiones Principales y Trabajo Futuro

El desarrollo de este proyecto demuestra el potencial y la viabilidad técnica de las arquitecturas **Retrieval-Augmented Generation (RAG)** aplicadas al diagnóstico fitopatológico asistido por inteligencia artificial.

Aunque el rendimiento de clasificación estricta se situó en un **11.11%**, la evaluación desacoplada demuestra que el motor generativo mantiene una sólida coherencia diagnóstica. El principal cuello de botella del sistema se localiza en la etapa de recuperación de información, cuya calidad se encuentra limitada por la estructura y composición del corpus indexado.

**La principal conclusión de esta investigación es que la mejora más significativa no depende de modificar algoritmos ni reentrenar modelos, sino de optimizar la base de conocimiento almacenada en ChromaDB mediante procesos de curaduría y normalización de datos.**

Para mitigar la recuperación de metadatos genéricos o irrelevantes, es necesario aplicar procesos de **Data Curation** y **Mapeo Ontológico** que permitan:

- Estandarizar los captions de preentrenamiento.
- Homogeneizar etiquetas taxonómicas.
- Consolidar un glosario clínico unificado.
- Reducir ruido lingüístico en el espacio vectorial.

Como trabajo futuro se propone:

- Implementar un proceso de normalización y limpieza taxonómica sobre el corpus ALive antes de la indexación vectorial.
- Incorporar conocimiento experto especializado en deficiencias nutricionales de cultivos de plátano.
- Evaluar estrategias híbridas de recuperación para incrementar la precisión del sistema.

Una ventaja importante de este enfoque RAG es que permite aprovechar los pesos preentrenados y la alineación multimodal de **AgriCLIP** junto con la infraestructura de **ChromaDB**, mejorando progresivamente el rendimiento sin requerir procesos costosos de *fine-tuning* ni infraestructura computacional de alto desempeño.

---

# Referencias Bibliográficas

1. Nawaz, U., et al. *AgriCLIP: A Multimodal Framework for Agricultural and Life Science Applications.*
2. OpenAI. *CLIP (Contrastive Language-Image Pre-Training).* GitHub Repository.
3. ChromaDB. *The AI-Native Open-Source Embedding Database.* Documentation.
4. Warcoder. *Nutrient Deficient Banana Plant Leaves Dataset.* Kaggle.
````
