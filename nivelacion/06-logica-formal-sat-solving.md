---
title: "Nivelación 06 — Lógica formal y SAT/SMT-solving con Z3"
created: 2026-07-02
updated: 2026-07-02
type: nivelacion
modulos: [11]
tags: [logica-formal, sat-solving, smt, z3, verificacion-formal, nivelacion]
---

# Nivelación: Lógica formal y SAT/SMT-solving con Z3

## Por qué lo necesitás

El README del seminario menciona "conceptos de seguridad" como prerrequisito, pero no menciona lógica formal ni SAT-solving. El **Lab 11.A** (`ch11-labs.md`, capítulo "Sectores Críticos") usa **Z3** — un SMT-solver de Microsoft Research — para **verificar formalmente** las propiedades de seguridad de un *Pre-Trade Risk Gateway* de trading: probar matemáticamente que el gateway nunca acepta una orden que rompe el límite de riesgo (`X > L`), en vez de simplemente *testear* algunos casos y esperar que alcance.

La técnica central del lab es contraintuitiva si nunca la viste: para probar que una propiedad `A` **siempre** se cumple, el código construye la **negación** de `A` (`Not(A)`) y le pide al solver que busque un contraejemplo. Si el solver responde `unsat` ("unsatisfiable" — no hay ninguna asignación de valores que haga `Not(A)` verdadero), eso implica lógicamente que `A` es verdadero en **todos** los casos posibles, no solo en los que se probaron. Sin entender esa lógica (reducción al absurdo, aplicada por un solver automático), el código del Lab 11.A —que imprime `esperado: unsat` y lo trata como éxito— es desconcertante: parece que el lab "quiere que falle".

## Autoevaluación

1. ¿Cuál es la diferencia entre **testear** una propiedad con casos de ejemplo (unit tests) y **verificarla formalmente**? ¿Qué gana la verificación formal que el testing no puede dar, aunque tengas miles de casos de test?
2. ¿Qué es, en una frase, un **SAT solver**? (Pista: el problema que resuelve es determinar si existe alguna asignación de valores verdadero/falso a las variables de una fórmula booleana que la haga verdadera.)
3. ¿Qué significa que una fórmula sea **SAT** (satisfiable) vs. **UNSAT** (unsatisfiable)?
4. En el Lab 11.A, el solver evalúa `Not(propiedad_de_seguridad)` y el resultado esperado es `unsat`. ¿Qué conclusión lógica permite sacar ese resultado sobre la propiedad original (no negada)? ¿Qué significaría, en cambio, que el solver responda `sat` en ese mismo chequeo?
5. Un **SMT solver** (Z3 es uno) extiende un SAT solver puro con teorías adicionales — por ejemplo, aritmética sobre enteros o reales, en vez de solo variables booleanas. En el código de `verify_basic_gateway()` del lab se usan variables `Int` (`O, P, L, S, X = Ints(...)`). ¿Por qué un SAT solver "puro" (solo booleanos) no alcanzaría para modelar esa propiedad del gateway de trading?
6. El comentario en `verify_frequency_throttle()` del lab advierte: sin la línea `D == If(N >= MAX_ORDERS, 0, 1)`, la variable `D` queda "libre" y `Not(throttle_property)` sería trivialmente SAT. ¿Por qué? ¿Qué error conceptual se está evitando ahí — verificar la propiedad, o verificar el *modelo* del sistema que se supone debe cumplirla?
7. ¿Por qué la verificación formal es especialmente relevante para sistemas de trading o finanzas de alto riesgo, como el circuit breaker del Lab 11.A, comparado con, por ejemplo, verificar formalmente el frontend de una aplicación web?

## Recursos

### Z3 — documentación y repositorio oficial

- Repositorio oficial de Z3 (Microsoft Research, código fuente, MIT license, bindings para Python entre otros lenguajes): [https://github.com/Z3Prover/z3](https://github.com/Z3Prover/z3)
- Guía oficial interactiva de Z3 ("Z3 Guide" — incluye tutorial de SMT-LIB, ejemplos de programación en Python/JavaScript y un playground interactivo para probar fórmulas): [https://microsoft.github.io/z3guide/](https://microsoft.github.io/z3guide/)

El lab usa el binding de Python (`z3-solver`, instalable con `pip install z3-solver`), que es exactamente lo que cubre la sección "Programming Z3" de la guía oficial.

### El problema SAT — introducción conceptual

Wikipedia, artículo "Boolean satisfiability problem" (cubre la definición formal, el resultado de Cook-Levin sobre NP-completitud, y por qué solvers heurísticos modernos resuelven en la práctica instancias con decenas de miles de variables pese a la dureza teórica del problema): [https://en.wikipedia.org/wiki/Boolean_satisfiability_problem](https://en.wikipedia.org/wiki/Boolean_satisfiability_problem)

No hace falta entender la teoría de complejidad (NP-completitud) en profundidad para el lab — alcanza con la intuición de qué pregunta responde un SAT solver y qué significan SAT/UNSAT como resultados.

## Conexión con el curso

**Lab 11.A — Pre-Trade Risk Gateway con Z3 + Extensiones** (`labs/ch11-labs.md`, capítulo "Sectores Críticos"): tiene tres partes. La Parte 1 reproduce la verificación básica del capítulo (`verify_basic_gateway()`), probando dos propiedades del gateway por reducción al absurdo:

```python
solver.add(constraints + [A, X > L])
result1 = solver.check()
print(f"[PRUEBA 1] No Breach con Accept: {result1} (esperado: unsat)")
```

Es decir: "si el gateway aceptó la orden (`A`) Y el resultado excede el límite (`X > L`), ¿existe algún caso posible?" — `unsat` prueba que nunca ocurre. La Parte 2 (`verify_frequency_throttle()`) extiende esto a una propiedad de *throttling* de frecuencia y es, además, la ilustración perfecta de la pregunta 6 de la Autoevaluación: el propio código del lab señala explícitamente el error de modelar la propiedad sin modelar la lógica de decisión real del sistema. La Parte 3 conecta el gateway ya verificado a un agente ADK v2 simulado (`trading_agent`) que debe respetar los rechazos del gateway en tiempo de ejecución — el puente entre "lo verificamos matemáticamente una vez" y "lo hacemos cumplir en producción".
