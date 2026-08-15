# RLHF y PPO: cómo se entrena la alineación de seguridad de un LLM

> **Por qué este documento.** `safety-alignment-seguridad-agentica.md` explica
> **qué es** la alineación de seguridad y por qué es una pieza de seguridad
> —no solo de "buen comportamiento"— pero remite deliberadamente el mecanismo
> de entrenamiento a otro lado. `gradiente_finetuning_explicacion.md` ya
> cubre ese mecanismo a nivel general, como una de las tres técnicas modernas
> de fine-tuning: RLHF en tres etapas (preferencias → *reward model* → PPO).
> Este documento es el punto medio entre ambos: no repite el pipeline
> general, pero sí profundiza en **cómo se aplica específicamente cuando el
> objetivo es seguridad**, qué tensión introduce (*helpfulness* vs.
> *harmlessness*), qué tan universal es entre los laboratorios de frontera, y
> qué exige en infraestructura.
>
> **Qué asume ya sabido.** El ciclo de gradiente/backpropagation y el
> pipeline general de RLHF de 3 etapas (`gradiente_finetuning_explicacion.md`,
> Partes 1-2 y 4.3), y qué es la alineación de seguridad como propiedad —
> aprendida, no una regla dura verificada matemáticamente
> (`safety-alignment-seguridad-agentica.md`).
>
> **Qué no cubre.** La derivación matemática completa de PPO o DPO (las
> funciones de pérdida exactas, la formalización de la divergencia KL). Se
> queda en la intuición mecánica y en la arquitectura del pipeline —con esa
> base, la matemática se lee después sin sorpresas.

## De "predecir texto plausible" a "rechazar con criterio"

Un modelo recién preentrenado no tiene ninguna noción de "peligroso". Si se
le pide un exploit o instrucciones para sintetizar un compuesto tóxico,
intenta completar el texto de la forma más verosímil posible según su
corpus de entrenamiento —exactamente el mismo mecanismo que usa para
completar cualquier otro texto, sin ningún concepto implícito de norma o
riesgo.

La alineación de seguridad no agrega una regla nueva al código del modelo
—no existe tal cosa como una regla dentro de una red neuronal—. Lo que hace
es **re-entrenar los pesos** para que el modelo *prefiera*, de forma
aprendida, generar rechazos seguros ante pedidos dañinos por sobre completar
esos pedidos, aunque técnicamente pudiera hacerlo. Esa preferencia se instala
en dos fases, apoyadas sobre el mismo pipeline de RLHF que ya conocés, pero
con un curado específico para seguridad:

- **La fase de SFT usa ejemplos de rechazo seguro.** Además de los pares
  (instrucción, respuesta ideal) para pedidos benignos, el dataset incluye
  pares donde el pedido es dañino y la respuesta ideal es un rechazo
  educado, directo y no evasivo —esto establece el comportamiento base antes
  de tocar RLHF.
- **El *reward model* se entrena con criterios de seguridad explícitos.**
  Los evaluadores (humanos, o un modelo evaluador en variantes como RLAIF)
  no solo comparan qué respuesta es "mejor" en general: juzgan
  específicamente *harmlessness* (¿el modelo se negó apropiadamente ante un
  pedido de riesgo?) junto con *helpfulness* (¿la respuesta sirve para algo
  ante un pedido legítimo?). Un dataset de *red teaming* —prompts
  adversariales diseñados para provocar fallas— alimenta buena parte de esa
  evaluación, específicamente para cubrir vectores como generación de
  malware, contenido CBRN, o sesgos severos.
- **PPO optimiza contra ese *reward model*, con la misma penalización KL**
  que ya viste —acá cumple un rol extra: evita que el modelo colapse hacia
  una única respuesta de rechazo genérica que maximiza el puntaje de
  seguridad sin aportar nada (un caso particular de *reward hacking*, el
  mismo fenómeno que se estudia como superficie de ataque en el Cap. 7 de
  este curso).

## El dilema central: *helpfulness* vs. *harmlessness*

El desafío técnico de fondo es que estas dos métricas tiran para lados
opuestos:

- **Helpfulness**: el modelo debe responder a la mayor cantidad posible de
  pedidos legítimos.
- **Harmlessness**: el modelo no debe generar contenido dañino, bajo ningún
  vector de ataque.

Si el término de recompensa por seguridad domina demasiado el entrenamiento,
el modelo cae en ***over-refusal***: rechaza pedidos benignos que comparten
vocabulario con temas sensibles. "¿Cómo funciona un virus informático a
nivel conceptual?" o "explicame la síntesis química del paracetamol" son
justo el tipo de pregunta legítima que un *reward model* mal calibrado
castiga por asociación superficial de palabras, no por intención real.

Vale la pena notar el paralelo con seguridad de software convencional: un
sistema que rechaza todo por las dudas no es "seguro", es **inútil por
sobre-restricción** —el equivalente, en alineamiento, de un firewall tan
agresivo que bloquea también el tráfico legítimo. La forma en que los
laboratorios atacan este problema es agregando datasets de *red teaming*
mixtos —pedidos benignos que rozan temas sensibles, junto a pedidos
genuinamente dañinos— para que el *reward model* aprenda a distinguir
intención, no solo vocabulario.

## ¿Se hace en todos los modelos de frontera?

Sí, pero no todos con la misma receta. Todos los laboratorios de frontera
usan alguna variante de alineamiento basado en preferencias, pero el método
concreto varió con el tiempo, sobre todo por costo de cómputo:

| Enfoque | Descripción | Ejemplos reportados |
|---|---|---|
| RLHF clásico (SFT + *reward model* + PPO) | El pipeline completo de tres etapas, con evaluadores humanos generando los datos de preferencia | El caso fundacional documentado es InstructGPT/GPT-3.5 de OpenAI; Llama 2 (Meta) también reporta esta receta, combinada con *rejection sampling* |
| RLAIF / Constitutional AI | Un modelo de IA, guiado por un conjunto explícito de principios ("constitución"), genera y evalúa las respuestas en vez de anotadores humanos en el bucle continuo | Es el nombre que usa específicamente Anthropic para el pipeline de Claude |
| DPO / IPO / KTO (familia de *preference optimization* directa) | Elimina el *reward model* separado y el ciclo explícito de RL; optimiza la política directamente sobre los pares de preferencia con una función de pérdida cerrada | Llama 3 (Meta) se apoya fuertemente en DPO combinado con *rejection sampling*; ampliamente adoptado también en modelos abiertos (Mistral, Qwen) por su menor costo de cómputo |

La tendencia general en los últimos años fue alejarse de PPO puro hacia DPO
y variantes —el próximo apartado explica por qué en términos de costo, y
más abajo por qué PPO, con todo, sigue siendo relevante para entender el
resto de la familia.

## PPO en términos intuitivos

**PPO (*Proximal Policy Optimization*)** es el algoritmo de aprendizaje por
refuerzo que ajusta los pesos del modelo durante RLHF clásico. Si el *reward
model* es el profesor que pone la nota, PPO es el método de estudio que usa
el modelo para mejorar esa nota sin volverse loco en el intento.

Pensalo como entrenar a un perro con premios. Si el perro hace bien un
truco, recibe una golosina y tiende a repetir esa acción. Pero corregir de
golpe, con pasos grandes, trae dos problemas:

- **Aprender demasiado rápido puede romper el modelo.** Si encuentra una
  respuesta que saca un puntaje altísimo —por ejemplo, contestar siempre
  "no puedo ayudarte con eso" porque así evita cualquier riesgo—, una
  actualización agresiva de pesos puede empujarlo de golpe hacia esa
  estrategia, perdiendo fluidez y capacidad de razonar en el proceso (el
  equivalente, en una red neuronal, del *catastrophic forgetting*).
- ***Reward hacking*.** Los modelos son buenos encontrando grietas en el
  sistema de evaluación para sumar puntaje sin cumplir el objetivo real —el
  mismo fenómeno que ya se mencionó arriba y que reaparece en el Cap. 7.

El adjetivo *proximal* (cercano) resume la solución: "buscá respuestas que
aumenten tu recompensa, pero no te alejes demasiado de cómo respondías hace
un instante — pasos chicos y controlados". Tres mecanismos concretos
implementan esa idea:

1. **El límite en la actualización (*clipped surrogate objective*).** PPO
   calcula cuánto mejora una respuesta nueva respecto de la anterior. Si esa
   mejora es descomunal —la probabilidad de una respuesta sube, por decir
   algo, un 500%—, el algoritmo **recorta** el gradiente en vez de aplicarlo
   completo. No importa cuán buena haya sido la recompensa: la actualización
   queda topada por una banda de tolerancia (típicamente 10-20%).
2. **El freno lingüístico (penalización KL).** PPO compara la respuesta del
   modelo actual contra el modelo de referencia —el mismo modelo, tal como
   quedó después de SFT, antes de tocar RLHF—. Si la distribución de
   palabras se aleja demasiado de ese punto de partida, se aplica una multa
   al puntaje. Esto mantiene coherencia sintáctica y evita que el modelo
   "descubra" un estilo de respuesta que maximiza el *reward model* pero ya
   no se lee como lenguaje natural.
3. **El crítico (*value function*).** Una segunda red, entrenada en
   paralelo, estima —token por token, no solo al final de la respuesta—
   cuánta recompensa espera acumular el modelo desde ese punto en adelante.
   Esto le permite al algoritmo atribuir qué palabra o frase específica hizo
   que la respuesta completa fuera buena o mala, en vez de juzgarla entera
   y a ciegas.

**Por qué la tendencia se movió hacia DPO**: PPO es estable y efectivo, pero
caro — necesita mantener **cuatro modelos simultáneos en memoria** durante
el entrenamiento (el que aprende, el de referencia, el *reward model* y el
crítico). DPO logra un efecto equivalente de "no alejarse demasiado" +
"maximizar preferencia", pero reformulando el problema como una única
función de pérdida cerrada que se optimiza con descenso de gradiente
estándar —sin *reward model* separado, sin crítico, sin ciclo de RL
explícito. Es una simplificación real de ingeniería, no solo una elección de
gusto.

## La infraestructura detrás del pipeline

A nivel de ingeniería, un pipeline de RLHF para seguridad tiene tres bloques
que se alimentan en cadena:

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│  Dataset de  │       │    Reward    │       │  Loop de PPO │
│ red teaming  │ ────► │    model     │ ────► │  (o DPO como │
│ + rechazos   │       │  (harmless + │       │  alternativa │
│   seguros    │       │   helpful)   │       │  más liviana)│
└──────────────┘       └──────────────┘       └──────────────┘
```

Dos puntos prácticos que explican por qué esto es costoso, no solo
conceptualmente distinto del fine-tuning ordinario:

- **Reward models especializados por categoría.** En vez de un único
  clasificador de "bueno/malo", suele entrenarse más de un *reward model* —
  uno para odio/violencia, otro para contenido de ciberseguridad, otro para
  riesgo CBRN— y combinar sus puntajes con una ponderación para obtener la
  recompensa final que ve PPO.
- **Costo de memoria de PPO.** Como se vio arriba, PPO necesita tener
  activos simultáneamente el modelo en entrenamiento (*policy*), el modelo
  de referencia congelado, el *reward model* y el crítico — del orden de
  4 veces el consumo de VRAM de una inferencia estándar sobre el mismo
  modelo base. Esto exige orquestación distribuida (frameworks como
  DeepSpeed-Chat, Megatron-LM o similares) para paralelizar tanto la
  generación de respuestas como la actualización de gradientes. Es, en la
  práctica, la razón de ingeniería más concreta detrás del giro de la
  industria hacia DPO y variantes más livianas.

## Para seguir pensando

1. El *reward model* de seguridad se entrena con datos de *red teaming*
   —prompts adversariales pensados para provocar fallas—. ¿Qué pasa si el
   conjunto de ataques usado para generar esos datos queda desactualizado
   frente a técnicas nuevas? ¿Es un problema que resuelve mejor RLHF o mejor
   DPO, o es independiente del algoritmo de optimización elegido?
2. El *over-refusal* y la *safety alignment degradation* (el otro documento)
   son fallas opuestas del mismo mecanismo: una es demasiada alineación, la
   otra demasiado poca. ¿Qué característica del *reward model* o del dataset
   de entrenamiento controla ese balance, y por qué ese balance nunca puede
   fijarse una sola vez y olvidarse?
