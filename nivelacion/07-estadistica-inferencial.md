# 7. Estadística inferencial básica

## Por qué lo necesitás

Dos labs del curso apoyan una decisión técnica concreta en un concepto de estadística inferencial que el curso no vuelve a explicar en el momento:

- **Lab 9.C** (detección de esteganografía LSB) calcula un **test de chi-cuadrado de Pares de Valores (Pairs of Values)** sobre los bits menos significativos de una imagen y concluye "sospecha de esteganografía" o "imagen limpia" según el p-value resultante. Si no sabés qué es un p-value ni qué significa que sea alto o bajo, ese veredicto aparece como un número mágico en vez de como una conclusión fundamentada.
- **Lab 11.C** (fusión de sensores en un sistema crítico) usa **distancia a la mediana** en lugar de z-score para detectar lecturas anómalas entre sensores redundantes. La razón — que con una muestra de solo 3 sensores el z-score produce falsos positivos — solo se entiende si sabés qué es un z-score, cómo se calcula, y por qué depende de una desviación estándar que con muestras chicas es inestable.

Esta guía no pretende enseñar estadística inferencial de cero: es un repaso dirigido a los conceptos puntuales que estos dos labs dan por sabidos. Si podés responder la autoevaluación de abajo con soltura, estás listo. Si no, los recursos de la sección final alcanzan para llegar a ese punto.

---

## Autoevaluación

Antes de mirar los recursos, probá responder estas preguntas. Si te trabás en la mayoría, arrancá por los recursos de Khan Academy en orden.

1. ¿Qué es un test de hipótesis, en una frase?
2. ¿Qué es la hipótesis nula (H₀) y qué es la hipótesis alternativa (H₁)?
3. ¿Qué significa un p-value bajo vs. un p-value alto? ¿Qué error común se comete al interpretarlo como "la probabilidad de que la hipótesis nula sea cierta"?
4. ¿Qué mide el test de chi-cuadrado, y qué tipo de datos requiere (categóricos vs. continuos)?
5. ¿Qué es un z-score y cómo se calcula a partir de la media y el desvío estándar?
6. ¿Por qué el z-score se vuelve inestable o poco confiable con muestras muy chicas (por ejemplo, 3 sensores)?
7. ¿Qué es la mediana y en qué se diferencia del cálculo de la media?
8. ¿Por qué la mediana es más robusta que la media/desvío estándar ante la presencia de un outlier?

---

## Recursos

**Khan Academy — curso de estadística inferencial (gratuito):**

- [Unit 12: Significance tests (hypothesis testing)](https://www.khanacademy.org/math/statistics-probability/significance-tests-one-sample) — qué es un test de hipótesis, hipótesis nula/alternativa, p-value.
- [Hypothesis testing and p-values (video)](https://www.khanacademy.org/math/statistics-probability/significance-tests-one-sample/more-significance-testing-videos/v/hypothesis-testing-and-p-values) — explicación puntual del p-value y su interpretación (y mala interpretación) más común.
- [Inference for categorical data (chi-square tests)](https://www.khanacademy.org/math/statistics-probability/inference-categorical-data-chi-square-tests) — unidad completa sobre el test de chi-cuadrado: para qué sirve, cómo se calcula, cómo se interpreta.

**Wikipedia (para consulta puntual de definiciones):**

- [Chi-squared test](https://en.wikipedia.org/wiki/Chi-squared_test) — definición formal y contexto de uso del test.
- [Standard score](https://en.wikipedia.org/wiki/Standard_score) — artículo de referencia para z-score: definición, fórmula, y por qué requiere conocer (o estimar bien) la media y el desvío estándar de la población.

---

## Conexión con los labs del curso

- **Lab 9.C (ch09 — esteganografía LSB):** el lab corre un test de chi-cuadrado de Pares de Valores sobre los LSB de cada canal de color. Un p-value alto es consistente con la hipótesis nula ("los bits son aleatorios, como se espera de una imagen sin manipular"); un p-value bajo la rechaza y es indicio de que los LSB fueron alterados para ocultar datos. Sin el concepto de p-value, ese resultado no se puede leer como evidencia — solo como un número.
- **Lab 11.C (ch11 — fusión de sensores):** con 3 sensores redundantes, calcular un z-score por sensor usando la media y el desvío estándar del propio trío es estadísticamente frágil (un solo sensor con lectura extrema arrastra la media y el desvío estándar, escondiendo su propia anomalía). El lab usa distancia a la mediana en su lugar porque la mediana no se mueve por un valor extremo aislado — exactamente la propiedad de robustez ante outliers que este documento repasa.
