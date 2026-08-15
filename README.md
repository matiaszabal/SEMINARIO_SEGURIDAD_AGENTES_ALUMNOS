# Seminario Seguridad en Agentes de IA — UNLP 2026 (material para alumnos)

Curso de posgrado sobre seguridad en sistemas de IA agéntica. Este repo es un subconjunto
curado de materiales pensado específicamente para alumnos: labs ejecutables del Capítulo 1,
un ejercicio de red-teaming educativo contra una plataforma pública, módulos de nivelación por
prerrequisito, recursos de aprendizaje adicionales, y las slides de apertura del seminario en PDF.

## Contenido

- **`labs/`** — Los 3 labs del Capítulo 1 (anatomía de un agente, confused deputy, superficie de
  ataque), cada uno en dos variantes: `-ollama` (modelo local, sin costo) y `-gcp` (Gemini vía
  Google Cloud). La guía de cada lab —qué hace, cómo correrlo, qué resultado esperar— está en
  `labs/guias_alumnos/`.
- **`Lakera/`** — Ejercicio de red-teaming educativo contra las 10 apps del catálogo público de
  [Lakera Agent Breaker](https://play.lakera.ai/agent-breaker): extracción de system prompt, tool
  poisoning, memory poisoning, tool abuse y jailbreaks de moderación de contenido, documentados con
  prompts exactos, respuestas y mitigaciones. Empezá por `Lakera/HACK_LLM_LAKERA.md` (índice
  general con el mapeo a OWASP LLM Top 10 / OWASP Agentic Top 10 / MITRE ATLAS) o por el resumen
  ejecutivo en PDF.
- **`nivelacion/`** — Módulos de autoestudio por prerrequisito (Python intermedio, fundamentos de
  LLMs/agentes, autenticación, ML/deep learning, policy-as-code, lógica formal, estadística).
  Empezá por `nivelacion/README.md`.
- **`recursos_aprendizaje/`** — Links externos verificados (canales, papers, repos de
  herramientas open-source de seguridad de IA), una nota práctica sobre correr modelos chicos en
  CPU, el desarrollo completo de gradiente/backpropagation/fine-tuning, y lecturas cortas sobre
  temas transversales del curso: la relación entre Privacidad Diferencial y seguridad agéntica,
  qué es *safety alignment*, el mecanismo de los ataques adversariales de evasión (FGSM/PGD/GCG),
  embeddings y búsqueda vectorial en RAG, y cómo se ataca la capa de explicabilidad (SHAP/LIME).
- **`slides/`** — Slides de apertura del seminario, del panorama de modelos de frontera, la
  explicación de gradiente/fine-tuning, la introducción a los labs de Cap. 1, y la teoría
  condensada del Tema 1 (IA Agéntica: fundamentos, impulsores y riesgos), todas en PDF.

## Cómo correr los labs

```bash
pip install -r requirements.txt
```

Para las variantes `-ollama`: instalar [Ollama](https://ollama.com) y descargar el modelo local
con `ollama pull qwen3.5:9b` antes de correr cualquier script.

Para las variantes `-gcp`: necesitás un proyecto de Google Cloud con Vertex AI habilitado y
credenciales configuradas (`gcloud auth application-default login`), o una API key de Gemini vía
AI Studio.

Cada script tiene un modo `--selftest` que verifica la lógica del lab sin invocar ningún modelo —
es el punto de partida recomendado antes de correr el lab completo.

## Sobre el curso

Curso de posgrado sobre seguridad en sistemas de IA agéntica. Audiencia: programadores,
científicos de datos e ingenieros con Python, ML/LLMs a nivel de uso, y conceptos básicos de
seguridad.

Este repo cubre solo el Capítulo 1 del curso completo. Si estás cursando el seminario, tu docente
te va a compartir el resto del material a medida que avanza el curso.
