# Sistema RAG Multimodal Híbrido para la Clasificación de Deficiencias Nutricionales en Cultivos

Este repositorio contiene el desarrollo, reproducción y propuesta de mejora de un flujo de trabajo (workflow) avanzado de **Generación Aumentada por Recuperación (RAG) Multimodal e Híbrida** enfocado en el ámbito fitopatológico.

---

## 📝 Descripción del Problema y Artículo de Referencia

### Artículo Original de Referencia
* **Título:** *AgriCLIP: A Multimodal Framework for Agricultural Agriculture and Life Science Applications*
* **Autores:** Umar Nawaz, et al.
* **Repositorio Base:** [GitHub Oficial de AgriCLIP](https://github.com/umair1221/AgriCLIP.git)

### Descripción del Problema
El diagnóstico temprano de deficiencias nutricionales en hojas de plantas es crucial para mitigar pérdidas en la producción agrícola. Los clasificadores zero-shot tradicionales o los modelos de visión directos suelen verse afectados por sesgos de etiquetas previas o por la falta de explicaciones morfológicas detalladas. 

### Objetivo de la Reproducción y Mejora
El objetivo principal es reproducir el alineamiento multimodal del espacio de características visuales y textuales propuesto por AgriCLIP (utilizando el dataset downstream de **Hojas de Plátano / Banana Deficiency**), e implementar un **sistema RAG Multimodal Híbrido** como propuesta de mejora. Esto permite desacoplar la señal sensorial cruda de la inferencia diagnóstica, evitando el *Data Leakage* a través de un proceso de subtitulado ciego combinado con recuperación de conocimiento experto.

---

## 🚀 Resumen de la Metodología (Propuesta de Mejora)

El pipeline de mejora mitiga el sesgo de clasificación directa dividiéndose en cuatro fases críticas:

1. **Configuración y Línea Base (Baseline):** Preparación del entorno con aceleración por hardware (GPU), clonación del framework AgriCLIP y carga de pesos base de Dino.
2. **Construcción del Índice Vectorial Semántico:** Lectura robusta y depuración del corpus agrícola enriquecido por los autores (`AgriClip-Image-Prompts-Customized.csv`). Generación de embeddings mediante el *Text Encoder* de **CLIP (ViT-B/32)** e indexación en una base de datos vectorial persistente **ChromaDB** empleando la métrica de distancia coseno.
3. **Canalización RAG Multimodal Híbrida:** * *Subtitulado Ciego (Blind Captioning):* Un LLM de visión (`gpt-4o-mini`) analiza las imágenes downstream y genera una descripción puramente morfológica (formas, manchas, patrones de color) de máximo 25 palabras, omitiendo nombres de enfermedades para evitar sesgos cognitivos.
   * *Fusión Temprana (Embedding Híbrido):* Combinación analítica promediada y normalizada ($L_2$) del vector visual proyectado (Dino ResNet50 + Alineador Lineal de AgriCLIP) con el embedding textual del subtítulo ciego.
   * *Recuperación e Inferencia:** Búsqueda de los $K=3$ vecinos más cercanos en ChromaDB. El contexto experto recuperado se introduce en un prompt estructurado orientado a objetos junto al modelo `gpt-4o-mini`, el cual dicta un veredicto sintomático en formato JSON.
4. **Evaluación de Métricas:** Medición integral del rendimiento del clasificador, del motor de búsqueda y del texto generado.

---

## 📂 Estructura del Repositorio

Para garantizar el correcto funcionamiento del entorno experimental, el repositorio debe organizarse de la siguiente manera:

```text
├── Proyecto_PLN_Avanzado (1).ipynb   # Notebook principal con las 4 fases del experimento
├── dataset_downstream_subtitulado.csv # Conjunto de datos downstream con etiquetas reales (90 muestras)
├── requirements.txt                   # Archivo de dependencias de software para Python
└── README.md                          # Documentación e informe del proyecto

```

---

# 🛠️ Documentación Experimental e Instrucciones de Ejecución

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

## ⚙️ Configuraciones e Hiperparámetros Utilizados

| Componente | Configuración |
|------------|--------------|
| **Modelo de Visión** | Dino ResNet50 (`dino_pretrain.pth`) |
| **Alineador** | AgriCLIP Linear Aligner (`Agri_Dino_aligner_DPT_CPT.pth`) |
| **Codificador de Texto** | CLIP ViT-B/32 |
| **Base de Datos Vectorial** | ChromaDB |
| **Métrica de Similitud** | Cosine |
| **Batch de Indexación** | 128 |
| **Recuperador** | K-Nearest Neighbors |
| **Valor de K** | 3 |
| **Modelo Generativo** | GPT-4o-mini |
| **Temperatura (Subtitulado)** | 0.2 |
| **Temperatura (Síntesis JSON)** | 0.0 |

---

# 📊 Resultados Obtenidos y Comparación

Tras evaluar el comportamiento del pipeline utilizando las **90 muestras del conjunto de prueba downstream de Banana**, se obtuvieron los siguientes resultados:

| Dimensión Evaluada | Métrica / Indicador | Valor |
|-------------------|--------------------|--------|
| Clasificación | Accuracy (vs. Baseline Zero-Shot del artículo) | **14.44%** |
| Recuperador (ChromaDB) | Recall@K | **10.00%** |
| Recuperador (ChromaDB) | Precision@K (Pureza) | **10.00%** |
| Recuperador (ChromaDB) | MRR (Mean Reciprocal Rank) | **0.1000** |
| Generador (LLM) | Token F1-Score | **0.1833** |
| Generador (LLM) | ROUGE-L | **0.1832** |
| Generador (LLM) | BERTScore | **0.7440** |

---

## 🔍 Análisis de Hallazgos

- El artículo original reporta un **23.55% de Accuracy** utilizando la arquitectura **AgriCLIP** pura en la tarea *zero-shot* para cultivos de plátano.

- El componente generador (**GPT-4o-mini**) exhibió una alta comprensión conceptual, respaldada por un **BERTScore de 0.7440**, indicando que las descripciones sintomáticas generadas mantienen una sólida coherencia semántica con las referencias patológicas reales.

- Los resultados sugieren que el principal factor limitante del sistema no se encuentra en la generación de respuestas, sino en la calidad y relevancia de los documentos recuperados por la base vectorial.

---

# 🎯 Conclusiones Principales y Trabajo Futuro

El desarrollo de este proyecto demuestra el potencial de las arquitecturas **Retrieval-Augmented Generation (RAG)** para aplicaciones de diagnóstico fitopatológico guiado.

Aunque el rendimiento global de clasificación alcanzó un **Accuracy de 14.44%**, el análisis desacoplado evidencia que el componente generador (**GPT-4o-mini**) presentó un desempeño semántico sólido.

El principal cuello de botella se encuentra en la etapa de recuperación de información, afectada por la limitada disponibilidad de registros específicos y la escasez de datos especializados sobre **Banana Deficiency** dentro del corpus **ALive** indexado en **ChromaDB**, situación reflejada en el **Recall@K de 10.00%**.

Como trabajo futuro, se propone:

- Incorporar nuevo conocimiento especializado de imagen y texto relacionado exclusivamente con deficiencias nutricionales en cultivos de plátano.
- Ampliar el corpus indexado para mejorar la cobertura documental.
- Evaluar estrategias híbridas de recuperación para incrementar la precisión del sistema.

Una ventaja importante de este enfoque RAG es que permite aprovechar los pesos preentrenados y la alineación multimodal de **AgriCLIP** junto con la infraestructura de **ChromaDB**, mejorando progresivamente el rendimiento sin requerir costosos procesos de *fine-tuning* ni infraestructura de cómputo de alto desempeño.

---

# 📚 Referencias Bibliográficas

1. Nawaz, U., et al. *AgriCLIP: A Multimodal Framework for Agricultural and Life Science Applications.*

2. OpenAI. *CLIP (Contrastive Language-Image Pre-Training).* GitHub Repository.

3. ChromaDB. *The AI-Native Open-Source Embedding Database.* Documentation.

4. Warcoder. *Nutrient Deficient Banana Plant Leaves Dataset.* Kaggle.

---
