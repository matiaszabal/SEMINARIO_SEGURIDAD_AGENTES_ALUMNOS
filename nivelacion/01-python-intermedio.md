---
title: "Nivelación — Python intermedio real"
---

# Python intermedio real

**Por qué lo necesitás**: los labs de este curso no son scripts de scripting básico — asumen que podés leer y modificar código que combina Pydantic, `async`/`await`, decoradores y manejo de excepciones específico. Por ejemplo: el Lab 2.1 (ch02) define un `output_schema` con modelos Pydantic anidados (un modelo que contiene `Dict[str, OtroModelo]`) para forzar la salida estructurada del LLM; los labs de ch03 y ch04 usan `async def` y `await` porque el `Runner` de Google ADK es asíncrono de punta a punta; y el Lab 8.A (ch08) usa decoradores para instrumentar funciones (logging, medición, control de acceso) sin tocar su cuerpo. Si estos tres patrones no te resultan naturales, vas a poder *ejecutar* los labs pero no vas a entender ni poder *modificar* lo que hacen — que es el objetivo real del seminario.

## Autoevaluación

Respondé estas preguntas para vos mismo, sin mirar documentación. Si dudás en alguna, andá directo al recurso sugerido antes de seguir.

1. **¿Podés explicar la diferencia entre una función `async def` y una función sincrónica, y cuándo hace falta `await`?** ¿Sabés qué pasa si llamás una corrutina sin `await`? → si no, ver *Documentación oficial: asyncio (conceptual)*.

2. **¿Entendés qué es un event loop y por qué `asyncio.run()` u otro mecanismo similar tiene que estar "arriba" de todo tu código async?** En los labs vas a ver que `Runner.run_async(...)` del ADK se invoca dentro de una función `async def main()`, no suelta en el módulo. → *Documentación oficial: asyncio (conceptual)*.

3. **¿Sabés qué hace un decorador propio — por ejemplo `@log_execution`, `@retry`, `@require_permission`— y por qué casi siempre necesita `functools.wraps`?** Si te muestro un decorador con `*args, **kwargs` en la función interna, ¿podés explicar para qué están? → *Tutorial: decoradores en Python (freeCodeCamp)*.

4. **¿Entendés la diferencia entre capturar `except Exception as e` y capturar una excepción específica como `except ValueError` o `except FileNotFoundError`, y por qué en código de seguridad casi siempre conviene lo segundo?** Pista: capturar todo silenciosamente puede ocultar el error real (por ejemplo, una inyección detectada que termina tragada como "excepción genérica"). → *Documentación oficial: Errores y excepciones*.

5. **¿Podés leer un modelo Pydantic con campos anidados — por ejemplo `class Agente(BaseModel): permisos: Dict[str, Permiso]` donde `Permiso` es otro `BaseModel` — y decir de memoria qué forma de JSON validaría ese modelo y cuál fallaría?** → *Documentación oficial: Pydantic — Models*.

6. **¿Sabés qué diferencia hay entre `Optional[str]` y `str` en un type hint, y qué implica en la práctica si un campo Pydantic es `Optional[str] = None` vs. un campo obligatorio sin default?** → *Documentación oficial: Pydantic — Models*.

7. **¿Podés distinguir, leyendo un `try/except/finally`, en qué orden se ejecuta cada bloque si la excepción se lanza dentro del `try` y hay un `return` en el `finally`?** → *Documentación oficial: Errores y excepciones*.

8. **¿Entendés la diferencia entre un decorador que envuelve una función sincrónica y uno que tiene que envolver una función `async def` (el wrapper también tiene que ser `async` y usar `await`)?** Este patrón aparece cuando se combinan decoradores de instrumentación con el `Runner` async del ADK. → combiná los dos primeros recursos de la sección siguiente.

## Recursos

**Documentación oficial**
- [asyncio — A Conceptual Overview](https://docs.python.org/3/howto/a-conceptual-overview-of-asyncio.html) — HOWTO oficial de Python que explica event loop, corrutinas, tasks y `await` con enfoque didáctico (no es solo referencia de API).
- [Errores y Excepciones — Tutorial de Python](https://docs.python.org/3/tutorial/errors.html) — capítulo del tutorial oficial sobre `try`/`except`, jerarquía de excepciones y por qué conviene capturar tipos específicos.
- [Pydantic — Models](https://pydantic.dev/docs/validation/latest/concepts/models/) — documentación oficial de Pydantic sobre cómo definir `BaseModel`, incluyendo modelos anidados (un modelo como tipo de campo de otro modelo).

**Tutorial introductorio**
- [Python Decorators – How to Create and Use Decorators in Python With Examples (freeCodeCamp)](https://www.freecodecamp.org/news/python-decorators-explained-with-examples/) — explica funciones como objetos, funciones anidadas, la sintaxis `@decorador` y el uso de `functools.wraps`.

**Práctica interactiva (opcional)**
- [Python en Exercism](https://exercism.org/tracks/python) — track gratuito con ejercicios cortos y mentoría; útil para practicar excepciones, type hints y funciones de orden superior antes de meterte con los labs.

---

Este prerrequisito pega directo en **ch02** (Lab 2.1, `output_schema` con Pydantic anidado), **ch03** y **ch04** (labs con `async`/`await` y el `Runner` de Google ADK), y **ch08** (Lab 8.A, decoradores).
