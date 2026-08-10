# Nivelación — módulos de autoestudio por prerrequisito

Cada archivo de este directorio corresponde a un prerrequisito identificado a partir de la ejecución real de los labs del curso (no de una lista teórica). Están pensados para autodiagnóstico: leé la autoevaluación de cada módulo, y si no podés responder con confianza la mayoría de las preguntas, revisá los recursos antes del encuentro correspondiente.

No hay obligación de completarlos en orden ni todos — cada uno indica con qué capítulo/lab conecta, así que podés priorizar según tu propio calendario.

## Módulos

| # | Archivo | Tema | Conecta con |
|---|---|---|---|
| 00 | [`00-setup-entorno.md`](00-setup-entorno.md) | Checklist técnico de instalación (no conceptual) | Todo el curso — hacer antes de E1 |
| 01 | [`01-python-intermedio.md`](01-python-intermedio.md) | Python intermedio: async/await, decoradores, excepciones, Pydantic anidado | ch02, ch03, ch04, ch08 |
| 02 | [`02-llm-agentes-fundamentos.md`](02-llm-agentes-fundamentos.md) | Function calling, RAG, system prompts, ReAct, Google ADK | ch01, ch02, y todos los labs con ADK |
| 03 | [`03-autenticacion-jwt-pki.md`](03-autenticacion-jwt-pki.md) | AuthN vs. AuthZ, JWT (RS256/HS256), mTLS, PKI/X.509 | ch03, ch04, ch12 |
| 04 | [`04-machine-learning-deep-learning.md`](04-machine-learning-deep-learning.md) | Gradientes, backpropagation, CNN, FGSM | ch07 |
| 05 | [`05-policy-as-code-rego-opa.md`](05-policy-as-code-rego-opa.md) | Rego / Open Policy Agent, default-deny, ABAC | ch03, ch05 |
| 06 | [`06-logica-formal-sat-solving.md`](06-logica-formal-sat-solving.md) | SAT/SMT-solving, verificación formal con Z3 | ch11 |
| 07 | [`07-estadistica-inferencial.md`](07-estadistica-inferencial.md) | Test de hipótesis, chi-cuadrado, p-value, z-score vs. mediana | ch09, ch11 |

## Cómo usar esto en el calendario de encuentros

Los módulos 01–02 (Python + LLM/agentes) son transversales y conviene resolverlos antes de E1. El resto se puede repasar la semana previa al encuentro donde más pesan:

- **Antes de E1** (ch01, ch02): 00, 01, 02.
- **Antes de E3** (ch03, ch04): 03, 05.
- **Antes de E4** (ch06, ch07 — ver rebalanceo confirmado): 04.
- **Antes de E5** (ch08, ch10, ch11, ch12): 06, 07.

Todos los links externos de estos módulos fueron verificados (WebSearch + WebFetch) al momento de escribirlos, 2026-07-02. Si alguno rompe con el tiempo, avisar para reemplazarlo.
