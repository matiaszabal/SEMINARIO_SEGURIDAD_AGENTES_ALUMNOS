---
title: "Nivelación 04 — Machine Learning / Deep Learning: gradientes, backpropagation, CNN y ataques adversariales"
created: 2026-07-02
updated: 2026-07-02
type: nivelacion
modulos: [7]
tags: [machine-learning, deep-learning, gradiente, backpropagation, cnn, fgsm, adversarial-ml, nivelacion]
---

# Nivelación: Machine Learning / Deep Learning más allá del "nivel de uso"

## Por qué lo necesitás

El README del seminario pide "familiaridad con ML/LLMs a nivel de uso" — suficiente para los primeros capítulos, donde el LLM es una caja negra a la que le mandás prompts. Pero el **Lab 7.1** (`ch07-labs.md`, capítulo "RL Security") no usa el modelo como caja negra: **entrena** una CNN pequeña sobre MNIST con PyTorch y después ejecuta un **ataque adversarial FGSM real**, calculando explícitamente

```
δ = ε · sign(∇_x L(θ, x, y))
```

Si nunca viste qué es un gradiente, qué hace `backward()` en PyTorch, o por qué existe una diferencia entre "el gradiente respecto a los pesos" (lo que se usa para *entrenar*) y "el gradiente respecto a la entrada" (lo que se usa para *atacar*), el código del lab se ejecuta pero no se entiende — es una caja negra que rota otra caja negra. La idea central del capítulo 7 (que un atacante puede perturbar imperceptiblemente una entrada para forzar una clasificación errónea) depende enteramente de esa distinción.

Esta guía no pretende enseñarte deep learning completo — pretende darte la intuición mínima para que el Lab 7.1 dejе de ser magia y se convierta en una fórmula que podés leer.

---

## Autoevaluación

Antes de abrir el lab, intentá responder estas preguntas sin buscar ayuda. Si más de dos o tres te generan dudas, andá primero a la sección de Recursos.

1. ¿Podés explicar qué es un gradiente y qué significa "la dirección de mayor crecimiento de una función"? (Pensalo primero en una función de una variable — la derivada — y después en una función de muchas variables, como una imagen de miles de píxeles.)
2. ¿Qué es una función de pérdida (*loss function*) y qué mide, en una frase?
3. ¿Qué es *backpropagation*, en una frase intuitiva? (No hace falta la derivación matemática completa — alcanza con entender que es un algoritmo para calcular cómo cada peso de la red contribuyó al error final.)
4. En el entrenamiento normal de una red, el gradiente se calcula respecto a los **pesos** del modelo (`θ`) para después ajustarlos y reducir el error. En el ataque FGSM del Lab 7.1, el gradiente se calcula respecto a la **entrada** (`x`, la imagen). ¿Por qué esa diferencia es la clave del ataque? ¿Qué se está "entrenando" en cada caso: el modelo, o la imagen?
5. ¿Qué hace la función `sign()` en la fórmula FGSM (`δ = ε · sign(∇_x L)`) y por qué se usa el signo del gradiente en vez del gradiente crudo?
6. En el código del lab, `epsilon = 0.07` limita la magnitud de la perturbación por píxel. ¿Por qué querría un atacante mantener `epsilon` chico, si con un `epsilon` grande el ataque sería "más efectivo"?
7. ¿Qué es una red neuronal convolucional (CNN) a alto nivel? ¿Qué problema resuelve una capa convolucional que no resuelve una capa densa (fully connected) al procesar una imagen?
8. En el código de `SimpleCNN` del lab hay dos capas `Conv2d` seguidas de dos capas `Linear` (`fc1`, `fc2`). ¿Podés ubicar dónde termina la parte "convolucional" (extracción de features espaciales) y empieza la parte "densa" (clasificación)?

---

## Recursos

### Intuición visual de redes neuronales, gradiente y backpropagation — 3Blue1Brown

Serie "Deep Learning" de 3Blue1Brown (Grant Sanderson), la referencia más citada para intuición visual de estos temas. Cuatro capítulos, cada uno de ~15-20 minutos:

- Capítulo 1 — "But what is a neural network?": [https://www.3blue1brown.com/lessons/neural-networks](https://www.3blue1brown.com/lessons/neural-networks) (también en YouTube: [https://www.youtube.com/watch?v=aircAruvnKk](https://www.youtube.com/watch?v=aircAruvnKk))
- Capítulo 2 — "Gradient descent, how neural networks learn": [https://www.youtube.com/watch?v=IHZwWFHWa-w](https://www.youtube.com/watch?v=IHZwWFHWa-w)
- Capítulo 3 — "What is backpropagation really doing?": [https://www.3blue1brown.com/lessons/backpropagation](https://www.3blue1brown.com/lessons/backpropagation) (también en YouTube: [https://www.youtube.com/watch?v=Ilg3gGewQ5U](https://www.youtube.com/watch?v=Ilg3gGewQ5U))
- Playlist completa "Neural Networks": [https://www.youtube.com/playlist?list=PLZZWrBYkx7Otcjr3eCLZDCgfpqnxMY29s](https://www.youtube.com/playlist?list=PLZZWrBYkx7Otcjr3eCLZDCgfpqnxMY29s)

Recomendación: mirá al menos los capítulos 1 y 3 antes del Lab 7.1. El capítulo 2 (gradient descent) da contexto extra si querés entender por qué el mismo mecanismo matemático que entrena la red es el que un atacante reutiliza para atacarla.

### Gradiente — intuición matemática

Khan Academy, curso de cálculo multivariable, artículo sobre el gradiente:
[https://www.khanacademy.org/math/multivariable-calculus/multivariable-derivatives/partial-derivative-and-gradient-articles/a/the-gradient](https://www.khanacademy.org/math/multivariable-calculus/multivariable-derivatives/partial-derivative-and-gradient-articles/a/the-gradient)

Con esto alcanza para entender qué es el gradiente como vector de derivadas parciales y por qué apunta en la dirección de mayor crecimiento de la función — la pieza matemática detrás de `∇_x L` en la fórmula FGSM.

### PyTorch — tutoriales oficiales

- **"Deep Learning with PyTorch: A 60 Minute Blitz"** (tutorial oficial introductorio: tensores, autograd, redes neuronales, entrenamiento de un clasificador de imágenes): [https://docs.pytorch.org/tutorials/beginner/deep_learning_60min_blitz.html](https://docs.pytorch.org/tutorials/beginner/deep_learning_60min_blitz.html)
- **"Adversarial Example Generation"** (tutorial oficial de PyTorch sobre el ataque FGSM exacto que reproduce el Lab 7.1 — mismo dataset MNIST, misma fórmula, con explicación paso a paso del código): [https://docs.pytorch.org/tutorials/beginner/fgsm_tutorial.html](https://docs.pytorch.org/tutorials/beginner/fgsm_tutorial.html)

Este último tutorial es, en la práctica, la mejor preparación puntual para el Lab 7.1: usa el mismo ataque, el mismo dataset y una arquitectura de CNN comparable.

---

## Conexión con el curso

**Lab 7.1 — FGSM Adversarial Attack on RL Agent Perception** (`labs/ch07-labs.md`, capítulo "RL Security"): entrena `SimpleCNN` sobre MNIST (proxy didáctico de un clasificador de señales de tránsito) y aplica `fgsm_attack()` para generar una imagen adversarial que cambia la predicción del modelo sin alterar visualmente la imagen para un humano. La función `fgsm_attack` del lab es literalmente la fórmula de la Autoevaluación pregunta 4-6, escrita en PyTorch:

```python
x = x.clone().detach().requires_grad_(True)   # el gradiente se pide sobre x, no sobre los pesos
output = model(x)
loss = F.cross_entropy(output, y)
loss.backward()                                # backpropagation calcula ∇_x L
grad_sign = x.grad.data.sign()                 # sign(∇_x L)
x_adv = torch.clamp(x + epsilon * grad_sign, 0, 1)  # δ = ε · sign(∇_x L), aplicado a x
```

Entender esta guía es prerrequisito directo para ese bloque de código. El **Lab 7.3** (Adversarial Training Defense, mismo capítulo) reutiliza el mismo concepto en la dirección defensiva: entrena con una mezcla de imágenes limpias y adversariales para que el modelo sea más robusto frente a este tipo de ataque.
