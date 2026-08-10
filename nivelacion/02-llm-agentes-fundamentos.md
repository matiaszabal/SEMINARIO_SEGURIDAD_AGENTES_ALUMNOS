---
title: "Nivelación — LLM y agentes: fundamentos más allá de usar un chat"
---

# LLM y agentes: fundamentos más allá de "usar un chat"

**Por qué lo necesitás**: haber usado ChatGPT, Gemini o Claude como usuario final no alcanza para seguir este curso. Desde la primera clase, ch01 (anatomía y superficie de ataque de un agente) y ch02 (inyección indirecta y RAG poisoning) dan por sabido qué es function calling / tool use, qué es un system prompt, qué es RAG y cómo funciona el patrón ReAct — porque las vulnerabilidades que se estudian son ataques *sobre esos mecanismos*. Además, todos los labs del curso están construidos sobre el framework Google ADK (Agent Development Kit): vas a ver `Agent`, `Runner` e `InMemorySessionService` en prácticamente todo el código. Si estos conceptos no son familiares, un ejercicio como "el agente ejecutó una tool que no debía por una instrucción escondida en un documento" no se entiende como ataque — parece simplemente que el programa "hizo algo raro".

## Autoevaluación

1. **¿Podés explicar qué es function calling / tool use y por qué el modelo no ejecuta la función directamente?** El LLM emite una *intención estructurada* (nombre de función + argumentos en JSON); es tu código el que decide si ejecutarla, con qué permisos y validando qué. → *Documentación de tool use (Anthropic)*.

2. **¿Qué es RAG (Retrieval-Augmented Generation) y por qué mezclar datos recuperados con la instrucción del sistema crea un riesgo de seguridad?** Pista: instrucción del sistema y datos recuperados terminan en el mismo canal de texto que ve el modelo — no hay una barrera técnica que le impida al modelo "obedecer" texto que vino de un documento externo en lugar de tratarlo como dato. Este es el problema central de ch02 (RAG poisoning, inyección indirecta). → *Wikipedia: Retrieval-augmented generation* + *ReAct prompting guide*.

3. **¿Qué es el patrón ReAct (Reasoning + Acting)?** ¿Podés describir el ciclo Thought → Action → Observation y por qué la mayoría de los agentes actuales (incluido lo que arma el ADK) lo siguen, aunque no lo llamen explícitamente "ReAct"? → *ReAct Prompting (Prompting Guide)*.

4. **¿Entendés la diferencia entre el `instruction`/`system_instruction` de un agente y el input del usuario?** ¿Por qué, técnicamente, ambos terminan concatenados en el mismo prompt que ve el modelo, y qué implica eso para un atacante que logra inyectar texto en cualquier punto de esa cadena (un documento, el resultado de una tool, etc.)? → *Documentación de tool use (Anthropic)* + *Get started con ADK*.

5. **¿Sabés qué hacen concretamente `Agent`, `Runner` e `InMemorySessionService` en Google ADK?** En una frase cada uno: `Agent` define el comportamiento (modelo, instrucción, tools); `Runner` orquesta la ejecución de un turno (llama al modelo, ejecuta tools, gestiona el loop); `InMemorySessionService` guarda el estado de la conversación en memoria del proceso (sin persistencia). → *Google ADK — Get started (Python)*.

6. **¿Podés explicar por qué un agente con acceso a una tool de "leer archivos" y otra de "enviar email" es más peligroso que un chatbot sin tools, incluso si el modelo subyacente es el mismo?** La superficie de ataque no es el modelo — es lo que el modelo puede *hacer* a través de las tools que vos le diste. → tema central de ch01.

7. **¿Qué diferencia hay entre un system prompt "ingenuo" (una sola instrucción de texto) y una estrategia de defensa que separa explícitamente instrucciones de datos (delimitadores, roles, sanitización)?** ¿Por qué esa separación es difícil de garantizar en un LLM basado en texto plano? → *Wikipedia: RAG* + *Documentación de tool use (Anthropic)*.

8. **¿Podés seguir mentalmente un loop de agente de al menos 3 pasos** (el modelo pide una tool, tu código la ejecuta, el resultado vuelve al modelo, el modelo decide el siguiente paso) **y señalar en qué punto exacto un atacante podría inyectar contenido controlado por él?** → *ReAct Prompting (Prompting Guide)* + *Google ADK — Get started*.

## Recursos

**Documentación oficial**
- [Tool use with Claude (Anthropic)](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview) — documentación oficial y actualizada de function calling / tool use: cómo el modelo emite una llamada estructurada, quién la ejecuta (tools "de cliente" vs. "de servidor") y cómo vuelve el resultado.
- [Agent Development Kit (ADK) — Get started, Python](https://adk.dev/get-started/python/) — quickstart oficial de Google ADK, el framework usado en los labs del curso; muestra cómo se define un `Agent` con modelo, instrucción y tools.

**Tutorial introductorio**
- [Retrieval-augmented generation — Wikipedia](https://en.wikipedia.org/wiki/Retrieval-augmented_generation) — explicación clara y estable de qué es RAG, cómo funciona (embeddings, recuperación, generación) y sus limitaciones.
- [ReAct Prompting — Prompt Engineering Guide](https://www.promptingguide.ai/techniques/react) — explica el ciclo Thought/Action/Observation y por qué reduce alucinaciones en tareas multi-paso; incluye ejemplo de código.

---

Este prerrequisito pega directo en **ch01** (anatomía del agente y superficie de ataque: tools, ReAct, Confused Deputy), **ch02** (RAG poisoning, inyección indirecta) y, en general, en todos los labs del curso que están construidos sobre `Agent`/`Runner`/`InMemorySessionService` de Google ADK.
