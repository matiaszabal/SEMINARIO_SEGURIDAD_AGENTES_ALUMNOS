---
title: "Nivelación: Autenticación, JWT y PKI (AuthN/AuthZ, RS256/HS256, mTLS, X.509)"
created: 2026-07-02
updated: 2026-07-02
type: nivelacion
modulos: [3, 4]
tags: [autenticacion, autorizacion, jwt, mtls, pki, x509, nivelacion]
---

# Nivelación 03 — Autenticación, JWT y PKI

## Por qué lo necesitás

El README del seminario pide "conceptos de seguridad (autenticación, control de acceso...)". En la práctica, los labs de los capítulos 3, 4 y 12 no *enseñan* estos conceptos — los dan por sabidos y construyen encima. Si llegás sin esta base, no vas a entender qué está pasando en el código, solo vas a poder copiarlo.

Tres ejemplos concretos, tomados directamente de los labs:

- **ch03, Lab Propuesto 3.2 ("Workload Identity Simulada")**: implementa un `MiniSVIDIssuer` que emite JWTs firmados con **RS256** y expiración corta (`ttl_seconds=3600`), simulando el patrón SPIFFE/SVID. El lab pide demostrar dos cosas que solo tienen sentido si entendés la mecánica de un JWT: qué pasa cuando el token **expiró** (rechazo por `exp`) y qué pasa cuando el token es **revocado** aunque no haya expirado (blocklist por `jti`). Si no sabés qué es un claim, qué es `exp`, o por qué RS256 (par de claves asimétrico) es distinto de simplemente "un token con checksum", el lab se vuelve una caja negra.

- **ch04, Lab Propuesto 4.A ("MCP Server con mTLS + JWT + Role-Check")**: genera con `openssl` una CA propia, un certificado de servidor y un certificado de cliente firmados por esa CA, y arranca un servidor con `ssl_cert_reqs=ssl.CERT_REQUIRED` — es decir, **mTLS real**, no un diagrama. El lab exige explícitamente verificar que sin el certificado de cliente el *handshake* TLS falla antes de llegar a la capa de aplicación. Si nunca armaste una cadena de confianza self-signed a mano, los cinco `openssl` de ese lab son puro *cargo cult*.

- **ch12, Lab Propuesto 12.B ("Secretsless Architecture")**: es el caso más interesante porque el propio enunciado del lab hace una advertencia conceptual: el simulador usa **HS256** con una `SECRET_KEY` simétrica compartida entre el emisor del token y el servicio que lo valida, lo cual **contradice** el principio "secretsless" que el patrón busca ilustrar — seguís teniendo un secreto estático que, si se filtra, permite falsificar tokens indefinidamente. En una arquitectura secretsless real (Aembit, SPIFFE/SPIRE) la atestación es asimétrica: el servicio validador solo necesita la clave *pública* del emisor, nunca un secreto que ambas partes deban conocer. Esa distinción — por qué HS256 con secreto compartido no es lo mismo que RS256 con clave privada/pública — es exactamente lo que este documento de nivelación busca que tengas resuelto *antes* de llegar al lab.

Si podés explicar con tus palabras por qué "quién tiene la clave privada" es la pregunta de seguridad más importante en los tres ejemplos de arriba, estás listo para ch03/ch04/ch12. Si no, seguí leyendo antes de los recursos.

## Autoevaluación

Respondé estas preguntas por escrito, sin buscar la respuesta primero. Si te trabás en 2 o más, revisá los recursos de abajo antes de empezar ch03.

1. ¿Podés explicar la diferencia entre **autenticación** (AuthN) y **autorización** (AuthZ) con un ejemplo concreto? (Pista: un agente puede estar perfectamente autenticado — sabemos quién es — y aun así no tener permiso para hacer lo que está pidiendo.)
2. ¿Qué significa que un JWT esté firmado con **RS256** en vez de **HS256**? ¿Por qué importa *quién tiene la clave privada* en cada caso, y qué pasa si esa clave se filtra en uno y otro escenario?
3. Un JWT no está cifrado, solo firmado. ¿Qué implica esto para los datos que pongas en el `payload`? ¿Cualquiera que intercepte el token puede leer sus claims?
4. ¿Qué es **mTLS** (mutual TLS) y en qué se diferencia de la TLS unidireccional que usás todos los días (el candadito del navegador cuando visitás un sitio HTTPS)? ¿Quién se autentica ante quién en cada caso?
5. ¿Qué es una **Autoridad Certificadora (CA)** y qué significa que exista una **cadena de confianza**? Si generás tu propia CA self-signed para un lab (como en ch04), ¿por qué tu navegador o tu sistema operativo no confían en ella por defecto?
6. ¿Qué campos identificás como mínimos en un **certificado X.509** (sujeto, emisor, clave pública, período de validez) y para qué sirve cada uno en el proceso de verificación?
7. ¿Por qué una credencial de larga duración (por ejemplo, una API key que no expira nunca) es más riesgosa que un token de corta duración con rotación automática, incluso si ambos "funcionan igual" el día que los usás?
8. En el Lab 12.B del curso, el enunciado admite que usar HS256 con un secreto compartido es una simplificación pedagógica que "contradice" el patrón secretsless que se busca enseñar. ¿Podés explicar con tus palabras *por qué* contradice ese patrón, y qué cambiaría si el lab usara RS256 en cambio?

## Recursos

- [JWT.io — Introduction](https://www.jwt.io/introduction) — estructura de un JWT (header.payload.signature), claims, y cómo funcionan las firmas HS256/RS256. Es también un debugger interactivo: pegá un JWT y ves sus tres partes decodificadas.
- [OWASP Cheat Sheet Series — Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) — buenas prácticas de autenticación, incluyendo transporte seguro y autenticación por certificado de cliente (TLS Client Authentication).
- [OWASP Cheat Sheet Series — Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) — define autorización explícitamente como distinta de autenticación; buena referencia para la pregunta 1.
- [Wikipedia — Mutual authentication](https://en.wikipedia.org/wiki/Mutual_authentication) — explica que, por defecto, TLS solo prueba la identidad del servidor ante el cliente, y qué agrega la autenticación mutua (mTLS) sobre eso.
- [Wikipedia — Public key infrastructure (PKI)](https://en.wikipedia.org/wiki/Public_key_infrastructure) — rol de la Autoridad Certificadora (CA), registro y emisión de certificados, y por qué la confianza en un certificado depende de la confianza en la CA que lo firmó.
- [Wikipedia — X.509](https://en.wikipedia.org/wiki/X.509) — formato estándar de certificado de clave pública: qué campos contiene (sujeto, emisor, clave pública, período de validez, algoritmo de firma) y cómo se usa en TLS/mTLS.

## Conexión con el curso

- **ch03 (Lab Propuesto 3.2 — Workload Identity Simulada)**: aplica directamente RS256, expiración corta (`exp`) y revocación por `jti`. Sin esta nivelación, el lab se ejecuta pero no se entiende.
- **ch04 (Lab Propuesto 4.A — MCP Server con mTLS + JWT + Role-Check)**: aplica CA propia, certificados de cliente/servidor y `ssl_cert_reqs=CERT_REQUIRED`. Requiere entender PKI y mTLS de la sección de arriba antes de tocar `openssl`.
- **ch12 (Lab Propuesto 12.B — Secretsless Architecture)**: el lab mismo señala la tensión HS256/secreto-compartido vs. RS256/clave-pública como la lección conceptual central — tenerla resuelta de antemano cambia el lab de "copiar código" a "entender por qué el patrón real usa criptografía asimétrica".
