# Descripción matemática y conceptual de las variables utilizadas en los modelos predictivos

Este documento resume las variables, transformaciones y parámetros utilizados en los modelos **Carolina**, **ELISA** y **Dixon–Coles**.

Los tres modelos utilizan información histórica de los partidos, pero representan el problema de predicción de manera diferente:

* **Carolina**: semejanza geométrica entre perfiles de partidos.
* **ELISA**: equilibrio competitivo e intensidad esperada.
* **Dixon–Coles**: proceso probabilístico de generación de goles.

---

# 1. Notación general

Para un equipo (e), considerando exclusivamente partidos **anteriores** de la temporada actual:

* $n$: partidos jugados.
* $GF$: goles a favor.
* $GA$: goles en contra.
* $TF$: tiros a favor.
* $TC$: tiros en contra.
* $TPF$: tiros a portería a favor.
* $TPC$: tiros a portería en contra.
* $PTS$: puntos obtenidos.
* $A$: tarjetas amarillas.
* $R$: tarjetas rojas.

Se definen además:

$$
DG=GF-GA
$$

$$
DT=TF-TC
$$

$$
DTP=TPF-TPC
$$

Para un partido:

* $L$: equipo local.
* $V$: equipo visitante.

En Carolina y en el bloque de balance de ELISA, muchas variables se construyen mediante:

$$
x_L-x_V
$$

Por ello:

* valores positivos suelen indicar ventaja o predominio del local;
* valores cercanos a cero indican equilibrio;
* valores negativos suelen indicar ventaja del visitante.

---

# 2. Variables originales de los datos

Las estadísticas básicas utilizadas por los modelos proceden de los resultados históricos.

| Variable                  | Significado                      |
| ------------------------- | ------------------------------   |
| `FTHG`                    | goles del equipo local           |
| `FTAG`                    | goles del equipo visitante       |
| `HS`                      | tiros del equipo local           |
| `AS`                      | tiros del equipo visitante       |
| `HST`                     | tiros a portería del local       |
| `AST`                     | tiros a portería del visitante   |
| `HY`                      | tarjetas amarillas del local     |
| `AY`                      | tarjetas amarillas del visitante |
| `HR`                      | tarjetas rojas del local         |
| `AR`                      | tarjetas rojas del visitante     |
| `Date`                    | fecha del partido                |
| `HomeTeam`                | equipo local                     |
| `AwayTeam`                | equipo visitante                 | 

Los momios como:

* `B365H`
* `B365D`
* `B365A`

no se utilizan para entrenar Carolina, ELISA o Dixon–Coles en las versiones desarrolladas.

Se utilizan posteriormente para evaluar métricas económicas como el ROI.

---

# 3. Modelo Carolina

Carolina utiliza un vector de **12 variables de comparación local–visitante**.

Su representación general es:

$$
\mathbf{x}=
[x_1,x_2,\ldots,x_{12}]
$$

---

## 3.1 Diferencia de goles por partido

Para cada equipo:

$$
GPP_e=
\frac{GF_e-GA_e}{n_e}
$$

La variable de Carolina es:

$$
x_1=
GPP_L-GPP_V
$$

Nombre:

`dif_goles_por_partido`

### Interpretación

Mide la diferencia entre los saldos goleadores medios de ambos equipos.

* $x_1>0$: mejor balance goleador del local.
* $x_1\approx0$: equilibrio.
* $x_1<0$: mejor balance goleador del visitante.

---

## 3.2 Diferencia de tiros por partido

Para un equipo:

$$
TPP_e=
\frac{TF_e-TC_e}{n_e}
$$

La variable es:

$$
x_2=
TPP_L-TPP_V
$$

Nombre:

`dif_tiros_por_partido`

### Interpretación

Representa el dominio relativo mediante volumen de tiros.

Un equipo que genera sistemáticamente más tiros de los que permite tendrá un valor positivo.

---

## 3.3 Diferencia de puntos por condición

Para el local solamente se consideran sus partidos como local:

$$
\frac{PTS_L^{casa}}
{n_L^{casa}}
$$

Para el visitante solamente se consideran sus partidos como visitante:

$$
\frac{PTS_V^{fuera}}
{n_V^{fuera}}
$$

Entonces:

$$
PPP_V^{fuera}
$$

Nombre:

`dif_puntos_condicion`

### Interpretación

Compara directamente:

> rendimiento del local en casa contra rendimiento del visitante fuera.

---

## 3.4 Diferencia de goles por condición

Para el equipo local:

$$
\frac{
GF_L^{casa}-GA_L^{casa}
}{
n_L^{casa}
}
$$

Para el visitante:

$$
\frac{
GF_V^{fuera}-GA_V^{fuera}
}{
n_V^{fuera}
}
$$

Entonces:

$$
DG_V^{fuera}
$$

Nombre:

`dif_goles_condicion`

### Interpretación

Mide la diferencia entre la capacidad goleadora neta del local jugando en casa y la del visitante jugando fuera.

---

## 3.5 Diferencia de puntos de forma

Carolina utiliza una ventana reciente de cinco partidos:

[
w=5
]

Para cada equipo:

[
PF_e=
\frac{1}{w}
\sum_{j=1}^{w}PTS_{e,j}
]

Entonces:

[
x_5=
PF_L-PF_V
]

Nombre:

`dif_puntos_forma`

### Interpretación

Representa la diferencia de rendimiento reciente entre ambos equipos.

---

## 3.6 Diferencia de goles de forma

Para cada partido reciente:

[
DG_j=
GF_j-GA_j
]

El promedio reciente es:

[
DGF_e=
\frac{1}{w}
\sum_{j=1}^{w}DG_j
]

Entonces:

[
x_6=
DGF_L-DGF_V
]

Nombre:

`dif_goles_forma`

### Interpretación

Mide la diferencia entre los saldos goleadores recientes de ambos equipos.

---

## 3.7 Diferencia de tiros a portería de forma

Para cada partido:

[
DTP_j=
TPF_j-TPC_j
]

El promedio reciente es:

[
DTPF_e=
\frac{1}{w}
\sum_{j=1}^{w}DTP_j
]

Entonces:

[
x_7=
DTPF_L-DTPF_V
]

Nombre:

`dif_tiros_porteria_forma`

### Interpretación

Mide el dominio reciente utilizando tiros a portería generados y recibidos.

---

## 3.8 Precisión de tiro

Carolina utiliza la expresión suavizada:

[
Prec_e=
\frac{TPF_e+1}{TF_e+2}
]

Entonces:

[
x_8=
Prec_L-Prec_V
]

Nombre:

`dif_precision_tiro`

### Interpretación

Mide la capacidad para convertir tiros totales en tiros dirigidos a portería.

Conceptualmente:

[
\text{tiros}
\rightarrow
\text{tiros a portería}
]

---

## 3.9 Eficiencia de definición

Se utiliza:

[
Def_e=
\frac{GF_e+1}{TPF_e+5}
]

Entonces:

[
x_9=
Def_L-Def_V
]

Nombre:

`dif_eficiencia_definicion`

### Interpretación

Mide la capacidad para convertir tiros a portería en goles:

[
\text{tiros a portería}
\rightarrow
\text{goles}
]

La precisión y la definición representan dos etapas ofensivas distintas.

---

## 3.10 Resistencia defensiva

Se define:

[
Res_e=
\frac{TPC_e-GA_e+1}{TPC_e+2}
]

Entonces:

[
x_{10}=
Res_L-Res_V
]

Nombre:

`dif_resistencia_defensiva`

### Interpretación

Aproxima la proporción de tiros a portería recibidos que no terminan en gol.

Puede reflejar conjuntamente:

* calidad defensiva;
* calidad del guardameta;
* calidad de las ocasiones concedidas.

No debe interpretarse exclusivamente como una medida del portero.

---

## 3.11 Diferencia de amarillas por partido

Para cada equipo:

[
AP_e=
\frac{A_e}{n_e}
]

Entonces:

[
x_{11}=
AP_L-AP_V
]

Nombre:

`dif_amarillas_por_partido`

### Interpretación

Mide diferencias disciplinarias entre los equipos.

Un valor positivo solamente significa que el local recibe más tarjetas amarillas por partido.

---

## 3.12 Diferencia de rojas por partido

Para cada equipo:

[
RP_e=
\frac{R_e}{n_e}
]

Entonces:

[
x_{12}=
RP_L-RP_V
]

Nombre:

`dif_rojas_por_partido`

### Interpretación

Representa diferencias en frecuencia de expulsiones.

---

# 4. Estandarización de Carolina

Las doce variables tienen escalas distintas.

Por ello se transforman mediante:

[
z_j=
\frac{x_j-\mu_j}{\sigma_j}
]

donde:

* (\mu_j): media de la variable en los datos de entrenamiento.
* (\sigma_j): desviación estándar.

El partido queda representado mediante:

[
\mathbf z=
[z_1,z_2,\ldots,z_{12}]
]

---

# 5. Prototipos de Carolina

Carolina construye dos prototipos.

## Prototipo de victoria local

[
\mathbf p_L=
E(\mathbf z\mid victoria\ local)
]

## Prototipo de victoria visitante

[
\mathbf p_V=
E(\mathbf z\mid victoria\ visitante)
]

Cada partido puede interpretarse como un punto dentro de un espacio de doce dimensiones.

---

# 6. Pesos de Carolina

Cada dimensión recibe un peso:

[
w_j>0
]

con:

[
\sum_{j=1}^{12}w_j=12
]

Los pesos se obtienen mediante una transformación `softmax` de los parámetros optimizados.

Una variable con mayor peso ejerce una influencia mayor sobre la distancia entre el partido y los prototipos.

---

# 7. Distancias de Carolina

La distancia respecto al prototipo de victoria local es:

[
d_L=
\sqrt{
\sum_{j=1}^{12}
w_j
(z_j-p_{L,j})^2
}
]

La distancia respecto al prototipo visitante es:

[
d_V=
\sqrt{
\sum_{j=1}^{12}
w_j
(z_j-p_{V,j})^2
}
]

Se define:

[
m=d_L-d_V
]

### Interpretación

* (m<0): partido más parecido al prototipo de victoria local.
* (m>0): partido más parecido al prototipo de victoria visitante.
* valores intermedios: región potencial de equilibrio.

El centro de la región de empate no tiene por qué ser exactamente:

[
m=0
]

---

# 8. Probabilidad de empate de Carolina

Carolina ver12 utiliza:

[
P(D\mid m)
==========

\sigma
\left[
\ell_{\max}
-----------

\left(
\frac{m-c}{s}
\right)^2
\right]
]

donde:

[
\sigma(x)=
\frac{1}{1+e^{-x}}
]

Los parámetros son:

* (c): centro óptimo de la región de empate.
* (s): anchura de esa región.
* (\ell_{\max}): controla la probabilidad máxima.

Carolina reduce así las doce dimensiones originales a una coordenada geométrica:

[
m=d_L-d_V
]

---

# 9. Modelo ELISA

ELISA significa:

**Modelo de Equilibrio e Intensidad Latente para la Selección de Empates**

Su idea principal consiste en separar la información del partido en dos dimensiones:

[
\text{Balance}
]

e

[
\text{Intensidad}
]

El bloque de balance responde aproximadamente:

> ¿Qué tan superiores son uno u otro equipo?

El bloque de intensidad responde:

> ¿Qué cantidad de actividad ofensiva y goles puede esperarse?

---

# 10. Suavizado mediante priors en ELISA

ELISA utiliza información promedio de la propia liga para estabilizar las estadísticas cuando existen pocos partidos.

Para una cantidad por partido:

[
\tilde{x}
=========

\frac{
X+s\bar{x}_{liga}
}{
n+s
}
]

donde:

* (X): acumulado observado del equipo.
* (n): número de partidos.
* (\bar{x}_{liga}): promedio actual de la liga.
* (s): `prior_strength`.

Cuando existen pocos partidos, la estimación permanece próxima al promedio de la liga.

Cuando aumenta (n):

[
\tilde{x}
\rightarrow
\frac{X}{n}
]

---

# 11. Variables de balance de ELISA

ELISA utiliza nueve variables de balance:

[
\mathbf b=
[b_1,b_2,\ldots,b_9]
]

Todas representan fundamentalmente comparaciones:

[
local-visitante
]

---

## 11.1 Diferencia de goles global

Se estiman de forma suavizada:

[
\widetilde{GF}_e
]

y:

[
\widetilde{GA}_e
]

Después:

[
GDPM_e=
\widetilde{GF}_e-
\widetilde{GA}_e
]

Entonces:

[
b_1=
GDPM_L-GDPM_V
]

Nombre:

`b_dif_goles_por_partido`

### Interpretación

Representa la diferencia de rendimiento goleador neto entre ambos equipos.

---

## 11.2 Diferencia de puntos por condición

Para el local:

[
PCond_L=
\frac{
PTS_L^{casa}
+
s\overline{PTS}^{casa}
}{
n_L^{casa}+s
}
]

Para el visitante:

[
PCond_V=
\frac{
PTS_V^{fuera}
+
s\overline{PTS}^{fuera}
}{
n_V^{fuera}+s
}
]

Entonces:

[
b_2=
PCond_L-PCond_V
]

Nombre:

`b_dif_puntos_condicion`

---

## 11.3 Diferencia de goles por condición

Para el local:

[
GDCond_L=
\frac{
DG_L^{casa}
+
s\overline{DG}^{casa}
}{
n_L^{casa}+s
}
]

Para el visitante:

[
GDCond_V=
\frac{
DG_V^{fuera}
+
s\overline{DG}^{fuera}
}{
n_V^{fuera}+s
}
]

Entonces:

[
b_3=
GDCond_L-GDCond_V
]

Nombre:

`b_dif_goles_condicion`

---

## 11.4 Diferencia de puntos de forma

ELISA utiliza una ventana:

[
w=5
]

o:

[
w=8
]

dependiendo de la configuración elegida para cada liga.

Para cada equipo:

[
PF_e=
\frac{1}{w}
\sum_{j=1}^{w}PTS_j
]

Entonces:

[
b_4=
PF_L-PF_V
]

Nombre:

`b_dif_puntos_forma`

---

## 11.5 Diferencia de goles de forma

[
GDF_e=
\frac{1}{w}
\sum_{j=1}^{w}
(GF_j-GA_j)
]

Entonces:

[
b_5=
GDF_L-GDF_V
]

Nombre:

`b_dif_goles_forma`

---

## 11.6 Diferencia de tiros a portería de forma

[
DTPF_e=
\frac{1}{w}
\sum_{j=1}^{w}
(TPF_j-TPC_j)
]

Entonces:

[
b_6=
DTPF_L-DTPF_V
]

Nombre:

`b_dif_tiros_porteria_forma`

---

## 11.7 Diferencia de precisión

Conceptualmente:

[
Prec_e=
\frac{\text{tiros a portería}}
{\text{tiros totales}}
]

ELISA regulariza la estimación mediante información de la liga:

[
Prec_e
\approx
\frac{
TPF_e+s_pPrec_{liga}
}{
TF_e+s_p
}
]

Entonces:

[
b_7=
Prec_L-Prec_V
]

Nombre:

`b_dif_precision`

---

## 11.8 Diferencia de definición

Conceptualmente:

[
Def_e=
\frac{\text{goles}}
{\text{tiros a portería}}
]

ELISA utiliza una versión regularizada:

[
Def_e
\approx
\frac{
GF_e+s_dDef_{liga}
}{
TPF_e+s_d
}
]

Entonces:

[
b_8=
Def_L-Def_V
]

Nombre:

`b_dif_definicion`

---

## 11.9 Diferencia de resistencia defensiva

Se define primero:

[
Stops_e=
TPC_e-GA_e
]

Conceptualmente:

[
Res_e
\approx
\frac{
\text{tiros a portería recibidos no convertidos}
}{
\text{tiros a portería recibidos}
}
]

ELISA utiliza una versión regularizada:

[
Res_e=
\frac{
Stops_e+s_rRes_{liga}
}{
TPC_e+s_r
}
]

Entonces:

[
b_9=
Res_L-Res_V
]

Nombre:

`b_dif_resistencia`

---

# 12. Vector de balance de ELISA

El vector completo es:

[
\mathbf b=
[
b_1,
b_2,
b_3,
b_4,
b_5,
b_6,
b_7,
b_8,
b_9
]
]

Sus componentes representan:

1. diferencia de goles global;
2. puntos por condición;
3. goles por condición;
4. puntos de forma;
5. goles de forma;
6. tiros a portería de forma;
7. precisión;
8. definición;
9. resistencia.

Su objetivo es representar el **desequilibrio competitivo** entre los equipos.

---

# 13. Variables de intensidad de ELISA

ELISA utiliza además nueve variables de intensidad:

[
\mathbf i=
[i_1,i_2,\ldots,i_9]
]

Aquí predominan combinaciones:

[
local+visitante
]

en lugar de diferencias.

---

## 13.1 Goles anotados combinados

[
i_1=
GFPM_L+GFPM_V
]

Nombre:

`i_goles_anotados_combinados`

### Interpretación

Representa el potencial ofensivo conjunto de ambos equipos.

---

## 13.2 Goles recibidos combinados

[
i_2=
GAPM_L+GAPM_V
]

Nombre:

`i_goles_recibidos_combinados`

### Interpretación

Representa la vulnerabilidad goleadora conjunta.

---

## 13.3 Tiros combinados

[
i_3=
TFPM_L+TFPM_V
]

Nombre:

`i_tiros_combinados`

### Interpretación

Aproxima el volumen ofensivo conjunto esperado.

---

## 13.4 Tiros a portería combinados

[
i_4=
TPFPM_L+TPFPM_V
]

Nombre:

`i_tiros_porteria_combinados`

### Interpretación

Representa una medida de generación conjunta de oportunidades ofensivas de mayor calidad.

---

## 13.5 Definición combinada

[
i_5=
Def_L+Def_V
]

Nombre:

`i_definicion_combinada`

### Interpretación

Dos equipos con alta capacidad de conversión producen valores elevados.

---

## 13.6 Vulnerabilidad combinada

Se define:

[
Vul_e=
1-Res_e
]

Entonces:

[
i_6=
(1-Res_L)+(1-Res_V)
]

Nombre:

`i_vulnerabilidad_combinada`

### Interpretación

Valores elevados indican que ambos equipos permiten que una proporción relativamente alta de tiros a portería recibidos termine en gol.

---

## 13.7 Goles totales de forma

Para cada partido reciente:

[
GT_j=
GF_j+GA_j
]

Para cada equipo:

[
GTF_e=
\frac{1}{w}
\sum_{j=1}^{w}GT_j
]

Después:

[
i_7=
\frac{
GTF_L+GTF_V
}{2}
]

Nombre:

`i_goles_totales_forma`

### Interpretación

Mide cuántos goles contienen recientemente los partidos en los que participan ambos equipos.

---

## 13.8 Tiros a portería totales de forma

Para un encuentro reciente:

[
TPT_j=
TPF_j+TPC_j
]

Para un equipo:

[
TPTF_e=
\frac{1}{w}
\sum_{j=1}^{w}TPT_j
]

Entonces:

[
i_8=
\frac{
TPTF_L+TPTF_V
}{2}
]

Nombre:

`i_tiros_porteria_totales_forma`

### Interpretación

Representa el nivel reciente de actividad ofensiva de los partidos de ambos equipos.

---

## 13.9 Propensión a marcador bajo

Para cada partido reciente:

[
Low_j=
\begin{cases}
1,&GF_j+GA_j\leq2\
0,&GF_j+GA_j>2
\end{cases}
]

Para un equipo:

[
Low_e=
\frac{1}{w}
\sum_{j=1}^{w}Low_j
]

Después:

[
i_9=
\frac{
Low_L+Low_V
}{2}
]

Nombre:

`i_propension_marcador_bajo`

### Interpretación

* valor cercano a 1: alta frecuencia reciente de partidos con dos goles o menos;
* valor cercano a 0: predominio de encuentros de tres goles o más.

---

# 14. Vector de intensidad de ELISA

El vector completo es:

[
\mathbf i=
[
i_1,
i_2,
i_3,
i_4,
i_5,
i_6,
i_7,
i_8,
i_9
]
]

Su objetivo no es determinar quién es mejor, sino caracterizar la **intensidad esperada del partido**.

Por tanto:

[
\mathbf b
\rightarrow
\text{¿quién debería dominar?}
]

mientras que:

[
\mathbf i
\rightarrow
\text{¿qué tipo de partido debería producirse?}
]

---

# 15. Balance latente de ELISA

Las nueve variables de balance son estandarizadas:

[
z_j=
\frac{
b_j-\mu_j
}{
\sigma_j
}
]

Posteriormente se ajusta una regresión Ridge cuya variable objetivo es:

[
Y_B=
G_L-G_V
]

El modelo es:

[
B=
\beta_0+
\sum_{j=1}^{9}
\beta_jz_j
]

con regularización:

[
\lambda_B
\sum_{j=1}^{9}
\beta_j^2
]

Conceptualmente:

[
B
\approx
\text{diferencia esperada de goles}
]

### Interpretación

* (B>0): ventaja esperada del local.
* (B<0): ventaja esperada del visitante.
* (B\approx0): equilibrio.

En el modelo se restringe aproximadamente a:

[
-6\leq B\leq6
]

---

# 16. Intensidad latente de ELISA

Las variables de intensidad también son estandarizadas.

La variable objetivo es:

[
Y_I=
G_L+G_V
]

El modelo Ridge es:

[
I=
\gamma_0+
\sum_{j=1}^{9}
\gamma_jz_j
]

con regularización:

[
\lambda_I
\sum_{j=1}^{9}
\gamma_j^2
]

Conceptualmente:

[
I
\approx
\text{goles totales esperados}
]

La intensidad se restringe aproximadamente a:

[
0\leq I\leq8
]

---

# 17. Variables finales del clasificador ELISA

ELISA no utiliza directamente las 18 variables originales para clasificar el empate.

Primero las reduce a:

[
(B,I)
]

Después utiliza:

[
|B|
]

porque para un empate interesa principalmente la **magnitud del desequilibrio**, independientemente de cuál equipo sea favorito.

El vector final es:

[
[
1,
|B|,
|B|^2,
I,
I^2,
|B|I
]
]

---

# 18. Probabilidad de empate de ELISA

La función logística utiliza:

[
\eta=
\theta_0+
\theta_1|B|+
\theta_2|B|^2+
\theta_3I+
\theta_4I^2+
\theta_5|B|I
]

Entonces:

[
P(D)=
\frac{
1
}{
1+e^{-\eta}
}
]

### Interpretación

* (|B|): grado de desequilibrio.
* (|B|^2): relación no lineal del desequilibrio.
* (I): intensidad.
* (I^2): relación no lineal de la intensidad.
* (|B|I): interacción entre equilibrio e intensidad.

Esto permite que el efecto del equilibrio dependa del tipo de partido esperado.

---

# 19. Umbral adaptativo de ELISA ver03

ELISA ver03 no utiliza un umbral absoluto fijo.

Supóngase que antes del nuevo partido existen las probabilidades congeladas:

[
p_1,p_2,\ldots,p_n
]

correspondientes a partidos elegibles ya concluidos de la temporada actual.

Con cobertura objetivo:

[
q=0.10
]

se calcula:

[
k=
\lceil0.10n\rceil
]

Las probabilidades históricas se ordenan de mayor a menor.

El umbral adaptativo es:

[
T_t=
p_{(k)}
]

donde (p_{(k)}) representa aproximadamente el límite inferior del 10 % superior histórico.

---

# 20. Percentil histórico de ELISA

Para un nuevo partido con probabilidad:

[
p_t
]

se puede calcular:

[
Percentil_t=
100
\frac{
#{p_j\leq p_t}
}{
n
}
]

Por ejemplo:

[
Percentil_t=94
]

significa que la puntuación del nuevo partido es superior a aproximadamente el 94 % de las observadas anteriormente.

---

# 21. Margen respecto al umbral

Puede definirse:

[
M_t=
p_t-T_t
]

Entonces:

* (M_t>0): el partido supera el umbral adaptativo.
* (M_t<0): queda fuera.

La regla operativa es:

[
p_t\geq T_t
\Rightarrow
EMPATE
]

[
p_t<T_t
\Rightarrow
ABSTENERSE
]

---

# 22. Congelamiento de las predicciones históricas de ELISA

Cuando un partido es evaluado antes de disputarse, se almacenan:

* probabilidad prepartido;
* percentil prepartido;
* umbral adaptativo;
* decisión.

Estos valores permanecen congelados.

Por ejemplo:

[
P(D)=0.3347
]

seguirá siendo:

[
0.3347
]

aunque posteriormente se disputen muchos otros encuentros.

Lo que sí puede cambiar es su posición dentro del historial completo de la temporada, por ejemplo:

* `rango_actual`;
* `percentil_actual`.

Eso no modifica la predicción original.

---

# 23. Dixon–Coles

Dixon–Coles utiliza una representación completamente diferente.

Sus principales datos son:

[
\text{goles}
]

y:

[
\text{identidad de los equipos}
]

No utiliza directamente:

* tiros;
* tiros a portería;
* tarjetas;
* precisión;
* definición;
* resistencia;
* forma construida manualmente.

---

# 24. Fuerza ofensiva de Dixon–Coles

Cada equipo (e) tiene un parámetro:

[
\alpha_e
]

que representa su capacidad ofensiva relativa.

Para garantizar identificabilidad se impone:

[
\sum_e\alpha_e=0
]

Un valor mayor de (\alpha_e) implica una mayor capacidad para generar goles.

---

# 25. Parámetro defensivo de Dixon–Coles

Cada equipo tiene un parámetro:

[
\delta_e
]

En nuestra formulación:

[
\lambda_L=
e^{h+\alpha_L+\delta_V}
]

y:

[
\mu_V=
e^{\alpha_V+\delta_L}
]

Como (\delta) se suma a la tasa esperada de goles del rival, un valor mayor representa realmente mayor **debilidad defensiva**.

Por tanto, conceptualmente:

[
\delta_e
\approx
\text{vulnerabilidad defensiva}
]

---

# 26. Ventaja de localía

Dixon–Coles utiliza un parámetro global:

[
h
]

Los goles esperados del equipo local son:

[
\lambda=
\exp(
h+\alpha_L+\delta_V
)
]

Para el visitante:

[
\mu=
\exp(
\alpha_V+\delta_L
)
]

El parámetro (h) representa la ventaja media de jugar como local en escala logarítmica.

---

# 27. Goles esperados de Dixon–Coles

Las variables derivadas principales son:

[
\lambda=
E(G_L)
]

y:

[
\mu=
E(G_V)
]

Por ejemplo:

[
\lambda=1.70
]

[
\mu=0.95
]

significa que el modelo espera aproximadamente:

* 1.70 goles del local;
* 0.95 goles del visitante.

---

# 28. Distribución Poisson

Inicialmente:

[
P(G_L=x)
========

e^{-\lambda}
\frac{\lambda^x}{x!}
]

y:

[
P(G_V=y)
========

e^{-\mu}
\frac{\mu^y}{y!}
]

Sin la corrección Dixon–Coles, ambos resultados serían tratados como variables Poisson independientes.

---

# 29. Parámetro (\rho) de Dixon–Coles

Dixon–Coles introduce:

[
\rho
]

para modificar específicamente la probabilidad de marcadores bajos.

La función de corrección es:

[
\tau(x,y)=
\begin{cases}
1-\lambda\mu\rho,&x=0,\ y=0\
1+\lambda\rho,&x=0,\ y=1\
1+\mu\rho,&x=1,\ y=0\
1-\rho,&x=1,\ y=1\
1,&\text{resto}
\end{cases}
]

Entonces:

[
P(x,y)
======

Pois(x;\lambda)
Pois(y;\mu)
\tau(x,y)
]

La corrección es especialmente relevante para los empates porque afecta directamente a:

[
0-0
]

y:

[
1-1
]

---

# 30. Peso temporal de Dixon–Coles

Los partidos más antiguos tienen menor influencia.

El peso puede expresarse como:

[
w_i=
0.5^{a_i/H}
]

donde:

* (a_i): antigüedad del partido en días.
* (H): vida media o `half_life_days`.

Por ejemplo:

[
H=365
]

significa que un partido ocurrido 365 días antes tiene aproximadamente la mitad del peso que uno actual.

---

# 31. Regularización L2 de Dixon–Coles

Para evitar valores extremos de ataque o defensa se utiliza:

[
Penalty=
\lambda_{L2}
\left(
\sum_e\alpha_e^2+
\sum_e\delta_e^2
\right)
]

La regularización favorece soluciones más estables.

---

# 32. Probabilidades 1-X-2 de Dixon–Coles

A partir de la matriz completa de marcadores:

[
P(H)=
\sum_{x>y}P(x,y)
]

[
P(D)=
\sum_{x=y}P(x,y)
]

[
P(A)=
\sum_{x<y}P(x,y)
]

donde:

* (H): victoria local.
* (D): empate.
* (A): victoria visitante.

Dixon–Coles produce directamente una distribución probabilística completa del resultado.

---

# 33. Comparación conceptual de los tres modelos

| Modelo      | Representación fundamental                                      |
| ----------- | --------------------------------------------------------------- |
| Carolina    | posición del partido en un espacio geométrico de 12 dimensiones |
| ELISA       | equilibrio latente (B) e intensidad latente (I)                 |
| Dixon–Coles | fuerzas ofensivas y defensivas que generan (\lambda) y (\mu)    |

---

## Carolina

Pregunta fundamental:

> ¿A qué región geométrica del espacio histórico se parece este partido?

Cadena conceptual:

[
\text{estadísticas históricas}
\rightarrow
\mathbf x_{12}
\rightarrow
\mathbf z
\rightarrow
(d_L,d_V)
\rightarrow
m
\rightarrow
P(D)
]

---

## ELISA

Pregunta fundamental:

> ¿Existe suficiente equilibrio competitivo y una intensidad compatible con el empate?

Cadena conceptual:

[
\text{estadísticas históricas}
\rightarrow
(\mathbf b_9,\mathbf i_9)
\rightarrow
(B,I)
\rightarrow
[
|B|,
|B|^2,
I,
I^2,
|B|I
]
\rightarrow
P(D)
\rightarrow
\text{percentil adaptativo}
]

---

## Dixon–Coles

Pregunta fundamental:

> ¿Qué distribución de marcadores resulta de las capacidades ofensivas y defensivas de los equipos?

Cadena conceptual:

[
\text{goles históricos}
\rightarrow
(\alpha,\delta,h,\rho)
\rightarrow
(\lambda,\mu)
\rightarrow
P(G_L,G_V)
\rightarrow
P(H),P(D),P(A)
]

---

# 34. Variables compartidas entre Carolina y ELISA

| Concepto                           | Carolina | ELISA |
| ---------------------------------- | -------- | ----- |
| diferencia de goles global         | Sí       | Sí    |
| diferencia de tiros global         | Sí       | No    |
| puntos casa/fuera                  | Sí       | Sí    |
| diferencia de goles casa/fuera     | Sí       | Sí    |
| puntos recientes                   | Sí       | Sí    |
| goles recientes                    | Sí       | Sí    |
| tiros a portería recientes         | Sí       | Sí    |
| precisión                          | Sí       | Sí    |
| definición                         | Sí       | Sí    |
| resistencia                        | Sí       | Sí    |
| amarillas                          | Sí       | No    |
| rojas                              | Sí       | No    |
| goles anotados combinados          | No       | Sí    |
| goles recibidos combinados         | No       | Sí    |
| tiros combinados                   | No       | Sí    |
| tiros a portería combinados        | No       | Sí    |
| definición combinada               | No       | Sí    |
| vulnerabilidad combinada           | No       | Sí    |
| goles totales recientes            | No       | Sí    |
| tiros a portería totales recientes | No       | Sí    |
| propensión a marcador bajo         | No       | Sí    |

---

# 35. Diferencia fundamental entre Carolina y ELISA

Carolina utiliza principalmente diferencias:

[
L-V
]

Esto permite representar quién parece superior.

ELISA utiliza dos perspectivas.

Para balance:

[
L-V
]

Para intensidad:

[
L+V
]

Esto permite distinguir partidos que pueden tener el mismo grado de equilibrio pero una intensidad completamente diferente.

Por ejemplo:

## Partido A

[
GFPM_L=2.0
]

[
GFPM_V=2.0
]

Diferencia:

[
2.0-2.0=0
]

Suma:

[
2.0+2.0=4.0
]

Existe equilibrio, pero alta intensidad.

## Partido B

[
GFPM_L=0.8
]

[
GFPM_V=0.8
]

Diferencia:

[
0.8-0.8=0
]

Suma:

[
0.8+0.8=1.6
]

Existe exactamente el mismo equilibrio, pero mucha menor intensidad.

Por tanto:

[
\boxed{
\text{equilibrio}
\neq
\text{intensidad}
}
]

Esta separación constituye uno de los principios fundamentales de ELISA.

---

# 36. Variables objetivo de los modelos

Las variables objetivo no deben confundirse con las variables predictoras.

## Carolina

Durante el entrenamiento se utiliza la clasificación:

[
Y=
\begin{cases}
+1,&\text{victoria local}\
0,&\text{empate}\
-1,&\text{victoria visitante}
\end{cases}
]

Entre otras funciones, esta clasificación permite construir los prototipos de victoria local y visitante.

---

## ELISA: objetivo del balance

[
Y_B=
G_L-G_V
]

---

## ELISA: objetivo de la intensidad

[
Y_I=
G_L+G_V
]

---

## ELISA: objetivo final

[
Y_D=
\begin{cases}
1,&G_L=G_V\
0,&G_L\neq G_V
\end{cases}
]

---

## Dixon–Coles

Dixon–Coles utiliza directamente el marcador observado:

[
(G_L,G_V)
]

para estimar los parámetros que maximizan la verosimilitud de los resultados históricos.

---

# 37. Variables que no intervienen en el entrenamiento

Los momios:

[
B365H
]

[
B365D
]

[
B365A
]

no participan como predictores en ninguno de estos modelos.

Por tanto, los modelos no intentan reproducir directamente las probabilidades del mercado.

Los momios se utilizan posteriormente para analizar rentabilidad.

---

# 38. ROI

El retorno sobre la inversión puede expresarse como:

[
ROI=
\frac{
\text{beneficio neto}
}{
\text{capital apostado}
}
]

Esto permite separar dos preguntas distintas:

1. ¿El modelo identifica partidos con mayor probabilidad de empate?
2. ¿Los momios ofrecidos por el mercado son suficientemente altos para obtener rentabilidad?

Un modelo puede responder positivamente a la primera pregunta y negativamente a la segunda.

---

# 39. Resumen de variables principales

## Carolina

Vector de doce variables:

[
\mathbf x_C=
[
DG,
DT,
PTS_{cond},
DG_{cond},
PTS_{forma},
DG_{forma},
DTP_{forma},
Prec,
Def,
Res,
A,
R
]
]

expresadas principalmente como diferencias local–visitante.

---

## ELISA: balance

[
\mathbf b=
[
DG,
PTS_{cond},
DG_{cond},
PTS_{forma},
DG_{forma},
DTP_{forma},
Prec,
Def,
Res
]
]

---

## ELISA: intensidad

[
\mathbf i=
[
GF,
GA,
TF,
TPF,
Def,
Vul,
GT_{forma},
TPT_{forma},
Low
]
]

---

## ELISA: variables latentes

[
B
\approx
\text{diferencia esperada de goles}
]

[
I
\approx
\text{goles totales esperados}
]

---

## Dixon–Coles

Principales parámetros:

[
\alpha_e
========

\text{capacidad ofensiva}
]

[
\delta_e
========

\text{vulnerabilidad defensiva}
]

[
h
=

\text{ventaja de localía}
]

[
\rho
====

\text{corrección de marcadores bajos}
]

Variables derivadas:

[
\lambda
=======

\text{goles esperados del local}
]

[
\mu
===

\text{goles esperados del visitante}
]

---

# 40. Síntesis conceptual final

Los tres modelos pueden resumirse de la siguiente manera.

## Carolina

[
\boxed{
\text{Semejanza geométrica}
}
]

El partido se representa mediante doce diferencias entre local y visitante y se compara con prototipos históricos.

---

## ELISA

[
\boxed{
\text{Equilibrio + Intensidad}
}
]

El partido se comprime en dos variables latentes:

[
(B,I)
]

y posteriormente se estima la probabilidad de empate.

ELISA ver03 añade una regla operativa adaptativa basada en la posición de la predicción dentro de la distribución histórica de la temporada.

---

## Dixon–Coles

[
\boxed{
\text{Generación probabilística de goles}
}
]

Los equipos poseen capacidades ofensivas y defensivas que determinan sus tasas esperadas de gol:

[
(\lambda,\mu)
]

A partir de ellas se obtiene una distribución completa de posibles marcadores.

---

# 41. Resumen general de las arquitecturas

### Carolina

[
\boxed{
\text{estadísticas}
\rightarrow
12\text{ diferencias}
\rightarrow
\text{estandarización}
\rightarrow
\text{distancias a prototipos}
\rightarrow
m
\rightarrow
P(D)
}
]

### ELISA

[
\boxed{
\text{estadísticas}
\rightarrow
9\text{ variables de balance}
+
9\text{ variables de intensidad}
\rightarrow
(B,I)
\rightarrow
P(D)
\rightarrow
\text{percentil adaptativo}
\rightarrow
EMPATE/ABSTENERSE
}
]

### Dixon–Coles

[
\boxed{
\text{goles históricos}
\rightarrow
\text{ataque/defensa/localía}
\rightarrow
(\lambda,\mu)
\rightarrow
\text{distribución de marcadores}
\rightarrow
P(H),P(D),P(A)
}
]

La diferencia conceptual fundamental puede resumirse así:

> **Carolina busca semejanza geométrica; ELISA busca equilibrio e intensidad; Dixon–Coles modela directamente el proceso probabilístico de generación de goles.**
