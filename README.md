# Sistema RAG Multimodal Híbrido para la Clasificación de Deficiencias Nutricionales en Cultivos

Este repositorio contiene el desarrollo de un flujo de trabajo (workflow) avanzado de **Generación Aumentada por Recuperación (RAG) Multimodal e Híbrida**. El objetivo principal es mejorar la precisión en el diagnóstico de deficiencias de nutrientes en hojas de plantas, utilizando como caso de estudio el dataset de **Hojas de Plátano (Banana Deficiency)**.

---

## Arquitectura del Workflow

El sistema mitiga el sesgo de clasificación directa y el *Data Leakage* dividiéndose en cuatro fases críticas:

1. **Configuración y Línea Base (Baseline):** Preparación del entorno con soporte para aceleración por hardware (GPU) y clonación del repositorio oficial de AgriCLIP.
2. **Construcción del Índice Vectorial Semántico:** Indexación de captions agrícolas enriquecidos mediante embeddings de texto generados con el *Text Encoder* de **CLIP (ViT-B/32)** en una base de datos vectorial persistentente **ChromaDB** (usando distancia coseno).
3. **Canalización RAG Multimodal Híbrida:** * *Subtitulado Ciego (Blind Captioning):* Un LLM de visión (`gpt-4o-mini`) analiza las imágenes y genera una descripción puramente morfológica (colores, formas, manchas) de máximo 25 palabras, sin tecnicismos ni nombres de enfermedades para evitar sesgos.
   * *Fusión Temprana:* Combinación analítica promediada del vector visual (proyectado mediante el alineador lineal de AgriCLIP desde Dino ResNet50) y el embedding textual de la descripción visual.
   * *Recuperación e Inferencia:* Búsqueda de los $K=3$ vecinos más cercanos en ChromaDB y envío del contexto recuperado a un prompt estructurado que obliga a `gpt-4o-mini` a dictar un veredicto final en formato JSON.
4. **Evaluación de Métricas:** Medición desacoplada del clasificador, del motor de recuperación y de la fidelidad del texto generado.

---

## Resultados y Métricas Obtenidas

Tras evaluar el comportamiento del pipeline con 90 muestras del conjunto de prueba downstream, se consolidaron las siguientes métricas en la Fase 4:

| Dimensión Evaluada | Métrica | Valor Obtenido |
| :--- | :--- | :--- |
| **Clasificación** | Accuracy (vs. Baseline Zero-Shot) | **14.44%** |
| **Recuperador (ChromaDB)** | Recall@K | **10.00%** |
| **Recuperador (ChromaDB)** | Precision@K (Pureza) | **10.00%** |
| **Recuperador (ChromaDB)** | MRR (Mean Reciprocal Rank) | **0.1000** |
| **Generador (LLM)** | Token F1-Score | **0.1833** |
| **Generador (LLM)** | ROUGE-L | **0.1832** |
| **Generador (LLM)** | BERTScore (Similitud Semántica) | **0.7440** |

---

## 🛠️ Requisitos de Instalación (Dependencies)

El entorno requiere Python 3.10+ y un entorno con aceleración **GPU (ej. T4)**. Las librerías de software críticas e indispensables para ejecutar este proyecto son:

* **Conectividad con LLMs:** `openai` 
* **Visión y Modelos Multimodales:** `clip` (repositorio oficial de OpenAI), `timm`, `open_clip_torch`, `torchvision`, `torch`
* **Almacenamiento Vectorial:** `chromadb`
* **Métricas y Evaluación Lingüística:** `scikit-learn`, `rouge-score`, `bert-score`
* **Procesamiento de Datos:** `pandas`, `numpy`, `kagglehub` (para descarga del dataset)

---

## Prerrequisitos de Archivos y Datos

Para reproducir con éxito el flujo de trabajo dentro de `Proyecto_PLN_Avanzado.ipynb`, se deben colocar manualmente los siguientes elementos en la raíz del entorno de ejecución:

1. **`AgriClip-Image-Prompts-Customized.csv`**: Base de datos de conocimiento provista por los autores de AgriCLIP para alimentar ChromaDB.
2. **`dataset_downstream_subtitulado.csv`**: Archivo maestro de evaluación que mapea los identificadores, rutas de imágenes y etiquetas reales.
3. **API Key de OpenAI**: Registrada obligatoriamente en la sección de secretos (**Secretos / Userdata**) de tu entorno de trabajo bajo la etiqueta exacta de `OpenAI`.
