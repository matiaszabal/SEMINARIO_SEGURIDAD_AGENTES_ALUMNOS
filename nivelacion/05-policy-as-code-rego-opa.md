---
title: "Nivelación: Policy-as-Code con Rego y Open Policy Agent (OPA)"
created: 2026-07-02
updated: 2026-07-02
type: nivelacion
modulos: [3, 5]
tags: [rego, opa, policy-as-code, abac, nivelacion]
---

# Nivelación 05 — Policy-as-Code con Rego y OPA

## Por qué lo necesitás

Varios labs del curso implementan autorización con **Open Policy Agent (OPA)**, un motor de políticas de propósito general, y sus políticas están escritas en **Rego** — un lenguaje declarativo propio de OPA, que no es Python ni pseudocódigo con sintaxis rara: tiene su propio modelo mental (reglas, no funciones; conjuntos de hechos que se evalúan, no un flujo de control secuencial).

Dos ejemplos concretos:

- **ch03, Lab Propuesto 3.1 ("OPA Policy Engine — Autorización ABAC para Agentes")**: levanta OPA como servidor (`docker run ... openpolicyagent/opa run --server`), carga una política Rego con `default allow := false`, una regla `allow` basada en el `spiffe_id` del agente, una regla `deny` que bloquea `DELETE` en producción, y una regla final `decision` que combina ambas (`allow` **and not** `deny`). El código del lab incluye un comentario explícito y deliberado: *"En Rego, `deny` no sobreescribe automáticamente a `allow` — son reglas independientes. Para que 'deny gana siempre' sea un invariante real, hay que consultar `decision`, no `allow` a secas."* Si no entendés que en Rego las reglas son declaraciones independientes que se combinan explícitamente (no un `if/elif` con precedencia implícita como en Python), este es el punto exacto donde el lab te va a confundir — y es intencional: el lab está enseñando un error común de diseño de políticas.

- **ch05, Lab Propuesto 5.C ("Kubernetes Data Residency con OPA Admission Controller")**: evalúa una política Rego (`data_residency.rego`) que decide si un pod con label `data-residency: EU` puede schedulearse en un nodo determinado, usando el binario `opa eval` en modo standalone (sin necesidad de Kubernetes real, hay una variante ligera con `opa eval --format=raw -d policy.rego -i input.json`). Igual que en ch03, la política implementa **ABAC** (Attribute-Based Access Control): la decisión depende de atributos del *input* (label del pod, región del nodo), no de un rol fijo asignado de antemano.

Si podés leer una política Rego con reglas `allow`/`deny` separadas y predecir correctamente qué decisión final produce — sin ejecutarla — estás listo para ch03/ch05. Si no, seguí con los recursos de abajo antes de tocar el binario `opa`.

## Autoevaluación

Respondé estas preguntas por escrito antes de empezar los labs de OPA.

1. ¿Qué significa que una política sea **"default deny"** (o, como aparece en el lab del curso, `default allow := false`)? ¿Por qué es la postura de seguridad recomendada frente a "default allow"?
2. En una política con reglas `allow` y `deny` escritas por separado (como en el Lab 3.1 del curso), ¿por qué `deny` *debería* tener precedencia sobre `allow`? Concretamente: ¿qué pasa si tu código de aplicación consulta directamente `data.agent_authz.allow` en vez de una regla combinada como `decision`?
3. ¿Entendés la diferencia entre un lenguaje **declarativo** como Rego y uno **imperativo** como Python para expresar una política de acceso? En Rego no escribís los pasos para *calcular* la decisión — escribís los *hechos* bajo los cuales una decisión es verdadera. ¿Qué implica eso para cómo se combinan varias reglas con el mismo nombre?
4. ¿Qué es **ABAC** (Attribute-Based Access Control) y en qué se diferencia de **RBAC** (Role-Based Access Control)? Dá un ejemplo donde RBAC no alcanza y hace falta ABAC (pista: horario, ubicación, o el monto de una transacción son atributos, no roles).
5. En el Lab 5.C, la política decide si un pod puede schedulearse según su label `data-residency` y la región del nodo. ¿Por qué este tipo de decisión es más natural de expresar en Rego (evaluando atributos de un `input` estructurado) que como una cadena de `if` en Python?
6. ¿Qué ventaja concreta tiene separar la política de autorización (Rego, evaluada por OPA) del código de la aplicación (Python, el agente) en vez de hardcodear los `if` de permisos directamente en el agente? Pensá en qué pasa cuando alguien necesita cambiar una regla de negocio sin re-deployar el agente.
7. Si nunca escribiste Rego, ¿podés al menos leer una política corta (5-10 líneas) con `package`, `default`, y 2-3 reglas `allow`/`deny`, y explicar en español qué decisión produce para un `input` de ejemplo dado?

## Recursos

- [Open Policy Agent — Policy Language (documentación oficial)](https://www.openpolicyagent.org/docs/policy-language) — introducción oficial a Rego: qué es, cómo se estructuran los paquetes y las reglas, y el modelo de evaluación declarativo.
- [Open Policy Agent — Rego Cheat Sheet](https://www.openpolicyagent.org/docs/cheatsheet) — referencia rápida con patrones comunes, incluyendo el patrón `default allow := false` que usa el Lab 3.1 del curso, con ejemplos ejecutables ("Try It") vinculados al Playground.
- [Rego Playground](https://play.openpolicyagent.org/) — entorno interactivo oficial para escribir y probar políticas Rego contra un `input` de ejemplo sin instalar nada. Ideal para resolver la pregunta 7 de la autoevaluación: pegá una política corta y un input, y verificá si tu predicción de la decisión fue correcta.
- [NIST SP 800-162 — Guide to Attribute Based Access Control (ABAC) Definition and Considerations](https://csrc.nist.gov/pubs/sp/800/162/upd2/final) — define ABAC formalmente y lo contrasta con modelos de control de acceso basados en roles; útil para la pregunta 4.

## Conexión con el curso

- **ch03 (Lab Propuesto 3.1 — OPA Policy Engine)**: implementa exactamente el patrón `default deny` + reglas `allow`/`deny` separadas + regla `decision` combinada que se explica arriba. Requiere Docker para levantar OPA como servidor.
- **ch05 (Lab Propuesto 5.C — Kubernetes Data Residency con OPA Admission Controller)**: reutiliza el mismo modelo ABAC pero evaluado con el binario `opa eval` en modo standalone (variante ligera sin necesidad de Kubernetes/minikube), aplicado a residencia de datos en vez de acceso a herramientas de un agente.
