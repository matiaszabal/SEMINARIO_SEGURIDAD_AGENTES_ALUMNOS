# 0. Setup del entorno

Antes de la primera clase, verificá que podés hacer esto.

Este checklist no es conceptual — es puramente técnico. El objetivo es que el primer encuentro no se pierda en instalar dependencias. Hacelo con margen de un par de días antes, no la noche anterior: `torch` es pesado y algunos pasos pueden requerir resolver conflictos de versión.

---

## Checklist

### Python

- [ ] Tengo Python 3.11 o superior instalado. Verificalo con:
  ```bash
  python3 --version
  ```

### Entorno virtual

- [ ] Puedo crear y activar un entorno virtual (`venv`) para aislar las dependencias del curso del resto de mi sistema:
  ```bash
  python3 -m venv .venv
  source .venv/bin/activate   # en Windows: .venv\Scripts\activate
  ```

### Dependencias del curso

- [ ] Instalé las dependencias de `requirements.txt` sin errores dentro del entorno virtual:
  ```bash
  pip install -r requirements.txt
  ```
  - `torch` y `torchvision` pueden tardar **varios minutos** en descargarse (son las librerías más pesadas del archivo). Es normal.
  - Por defecto se instala la variante **CPU-only** de PyTorch, que es suficiente para los labs del curso (no hace falta GPU ni CUDA). Si tenés GPU con CUDA y querés usarla, instalá la variante correspondiente desde [pytorch.org/get-started/locally](https://pytorch.org/get-started/locally/) *antes* de correr `pip install -r requirements.txt`.

### Docker

- [ ] Tengo Docker instalado y funcionando. Verificalo con:
  ```bash
  docker run hello-world
  ```
  Docker es necesario para correr **OPA (Open Policy Agent)** como servicio en los labs de ch03 y ch05:
  ```bash
  docker run -p 8181:8181 openpolicyagent/opa:latest run --server
  ```
  - Si no querés o no podés instalar Docker, hay una alternativa **sin Docker**: el binario standalone `opa eval`, documentado en `labs/ch05-labs.md` §5.C (Lab Propuesto 5.C).
  - Instalación oficial de Docker: [Get Docker (Docker Docs)](https://docs.docker.com/get-started/get-docker/) — cubre Windows, Mac y Linux. Para servidores/Linux sin interfaz gráfica, también existe [Docker Engine](https://docs.docker.com/engine/install/) como alternativa más liviana a Docker Desktop.

### API key de Google AI Studio

- [ ] Obtuve una API key de Google AI Studio y la configuré como variable de entorno:
  ```bash
  export GOOGLE_API_KEY="tu-api-key-de-google-ai-studio"
  ```
  - Generá la key en [Google AI Studio](https://aistudio.google.com/app/apikey).
  - Documentación oficial: [Using Gemini API keys](https://ai.google.dev/gemini-api/docs/api-key).
  - Es **opcional** para varios labs: la mayoría tiene un *guard* que salta la parte que invoca al LLM si no hay key configurada. Es **necesaria** solo para los labs que sí llaman al LLM en vivo (por ejemplo, los que usan Google ADK + Gemini directamente).

### (Opcional) Cuenta de GCP

- [ ] Si vas a correr alguno de los labs marcados **"requiere GCP"** (Cloud Logging, Vision API, Healthcare API, Security Center, etc. — ver `requirements.txt` para la lista de extras comentados), tengo una cuenta de Google Cloud Platform con un proyecto activo y facturación habilitada.
  - Todos estos labs tienen una alternativa local o simulada si no tenés GCP disponible — no es un bloqueante para el curso.

---

## Si algo falla

- **Conflicto de versiones de `numpy` entre `torch` y `scikit-image`**: es un problema conocido de compatibilidad entre estas dos librerías. Probá instalar primero `numpy` en la versión que pide `requirements.txt` (`numpy>=1.26.0`), y después el resto de las dependencias en un paso separado. Si persiste, revisá el mensaje de error de `pip` — suele indicar exactamente qué versión resolvería el conflicto.
- **Falta de espacio en disco durante la instalación**: `torch` y sus dependencias (CUDA runtime incluido en algunos builds, aunque el curso usa la variante CPU) pueden ocupar varios GB entre la descarga y la instalación. Verificá que tengas al menos 5 GB libres antes de correr `pip install -r requirements.txt`.
- **`docker run hello-world` falla o el daemon no responde**: en Linux, confirmá que el servicio Docker está corriendo (`sudo systemctl status docker`) y que tu usuario pertenece al grupo `docker` (si no, hay que agregarlo y reiniciar sesión). Si el problema persiste o preferís no instalar Docker, usá la alternativa con el binario `opa` standalone (`labs/ch05-labs.md` §5.C).

---

## Recursos

- [Get Docker (Docker Docs)](https://docs.docker.com/get-started/get-docker/) — instalación oficial de Docker Desktop / Docker Engine por plataforma.
- [Google AI Studio — API Keys](https://aistudio.google.com/app/apikey) — página oficial para generar tu `GOOGLE_API_KEY`.
- [Using Gemini API keys (ai.google.dev)](https://ai.google.dev/gemini-api/docs/api-key) — documentación oficial sobre cómo usar y proteger la API key.
