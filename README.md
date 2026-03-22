# Diseño e Implementación de un Sistema RAG

Este repositorio contiene el código y la documentación del proyecto final del curso de IA Generativa. El proyecto consiste en el desarrollo de un sistema RAG (Retrieval-Augmented Generation) enfocado en el dominio del turismo en el Perú. 

## Descripción del Proyecto
El objetivo principal de este sistema es superar las limitaciones clásicas de los LLMs, como las alucinaciones y la dependencia de un conocimiento estático. Para lograrlo, se implementó un pipeline que recupera fragmentos relevantes de documentos reales y los utiliza como contexto para que el modelo de lenguaje genere respuestas fundamentadas, verificables y con citas exactas. La base de conocimiento está conformada por 5 documentos en formato PDF sobre el turismo peruano.

## Arquitectura del Sistema
El flujo completo abarca desde el pipeline de ingesta de documentos hasta la generación de la respuesta.

* **Ingesta y Chunking**: Los documentos se procesaron dividiéndolos en fragmentos con un tamaño de chunk de 800 caracteres y un overlap de 160 caracteres (20%). Esta configuración fue validada empíricamente tras evaluar 5 combinaciones, logrando capturar párrafos completos y obteniendo el mayor Grounding Score (0.9983) con un total de 1,288 chunks.
* **Embeddings**: Se utilizó el modelo `all-MiniLM-L6-v2` de 384 dimensiones para la representación vectorial de los textos.
* **Recuperación Híbrida (Hybrid Search)**: El sistema combina la búsqueda semántica mediante FAISS HNSW (con parámetro M=32) y la búsqueda exacta por palabras clave usando BM25 (con parámetro k=5). Esta combinación recupera hasta 10 documentos candidatos únicos, mitigando las debilidades individuales de cada método.
* **Reranking**: Se aplicó un modelo Cross-Encoder multilingüe (`mmarco-mMiniLMv2`) con un umbral de 0.05 para evaluar y reordenar los pares de consulta y documento. Este paso filtra la información irrelevante y envía únicamente los 3 mejores documentos (Top-3) al LLM.
* **Generación**: La respuesta es formulada por el LLM `Phi-3.5-mini-instruct`, configurado de manera determinista (do_sample=False) para priorizar la precisión.

## Características Principales
* **Trazabilidad Total (Citation per Sentence)**: El sistema fue diseñado con instrucciones explícitas en el system prompt para garantizar la trazabilidad. Cada afirmación de la respuesta termina indicando la fuente original con el formato `[Doc X]`, lo que permite detectar y evitar alucinaciones.
* **Búsqueda Robusta**: La integración híbrida demostró ser superior a la recuperación semántica aislada, capturando eficazmente términos técnicos y exactos (ej. MINCETUR, Cajamarca) junto con sinónimos y paráfrasis.
* **Interfaz Interactiva Web**: Se incluye una interfaz gráfica desarrollada en Gradio e integrada en Google Colab. Permite realizar consultas en lenguaje natural y visualiza un panel derecho con los chunks fuente recuperados para una auditoría completa.

## Evaluación
El desempeño del sistema RAG fue validado rigurosamente mediante las siguientes métricas:
* **Grounding Score (NLI)**: Evaluado con el modelo `mDeBERTa-v3-base-mnli-xnli`, alcanzó un puntaje aproximado de 0.99 en múltiples consultas (como "Turismo comunitario en Cajamarca" y "¿Qué es el turismo sostenible?"), demostrando que las respuestas están fuertemente fundamentadas en el contexto.
* **BLEU y ROUGE**: El sistema reportó un ROUGE-1 de 0.3097 y un ROUGE-L de 0.1677. Se obtuvo un puntaje BLEU de 0.0, lo cual es el resultado esperado y correcto, ya que un sistema RAG eficiente debe parafrasear y sintetizar el contexto en lugar de copiarlo textualmente.

## Autores
* Milner Vega Pardo
* Angel Omar Terrones Escobedo

---
*Este proyecto es parte de una investigación técnica aplicada orientada al desarrollo de soluciones avanzadas en ciencia de datos.*
