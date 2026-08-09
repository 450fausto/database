# 1. Datos mínimos que necesita ELISA

Para cada partido necesita:

* fecha;
* equipo local;
* equipo visitante;
* goles local;
* goles visitante;
* tiros local;
* tiros visitante;
* tiros a portería local;
* tiros a portería visitante.

En la nomenclatura habitual:

$$
\text{Date},\text{HomeTeam},\text{AwayTeam},\text{FTHG},\text{FTAG},\text{HS},\text{AS},\text{HST},\text{AST}
$$

Los momios son opcionales y **no intervienen en el modelo**.

ELISA tampoco necesita:

* amarillas;
* rojas;
* corners;
* faltas.

---

# 2. Regla temporal fundamental

Cada temporada comienza completamente desde cero.

Para cada equipo se inicializa:

$$
n=GF=GA=SF=SA=SOTF=SOTA=PTS=0
$$

y también:

$$
n_H=PTS_H=GD_H=0
$$

$$
n_A=PTS_A=GD_A=0
$$

donde:

* $n$: partidos jugados;
* $GF$: goles anotados;
* $GA$: goles recibidos;
* $SF$: tiros realizados;
* $SA$: tiros recibidos;
* $SOTF$: tiros a portería realizados;
* $SOTA$: tiros a portería recibidos;
* $PTS$: puntos;
* $H$: condición de local;
* $A$: condición de visitante.

También se almacena el historial reciente de cada equipo.

Los partidos se procesan cronológicamente.

**Todos los partidos de una misma fecha se calculan antes de incorporar los resultados de esa fecha.**

Por tanto, si en una fecha hay diez partidos:

1. se calculan las diez características prepartido;
2. se generan las diez predicciones;
3. después se actualizan los estados con los diez resultados.

Esto evita contaminación temporal dentro de una jornada.

---

# 3. Puntos obtenidos en cada partido

Para un equipo con $GF$ y $GA$:

$$
PTS = 
\begin{cases} 
3, & \text{GF} \gt \text{GA} \\
1, & \text{GF} = \text{GA} \\
0, & \text{GF} \lt \text{GA} 
\end{cases}
$$


---

# 4. Estado acumulado de la liga

Además de los estados de cada equipo, ELISA mantiene un estado global de la liga.

Después de $M$ partidos concluidos almacena:

* $G$: goles totales;
* $S$: tiros totales;
* $T$: tiros a portería totales;
* $P$: puntos totales;
* $P_H$: puntos obtenidos por locales;
* $P_A$: puntos obtenidos por visitantes;
* $G_H$: goles de locales;
* $G_A$: goles de visitantes.

Estos valores se utilizan para construir los **priors dinámicos de la liga**.

---

# 5. Priors de la liga

Si ya se han jugado $M$ partidos, existen:

$$
2M
$$

observaciones-equipo.

ELISA calcula:

### Goles por equipo y partido

$$
\pi_{GF}=\frac{G}{2M}
$$

y utiliza el mismo valor como prior de goles recibidos:

$$
\pi_{GA}=\pi_{GF}
$$

### Tiros por equipo y partido

$$
\pi_S=\frac{S}{2M}
$$

### Tiros a portería por equipo y partido

$$
\pi_T=\frac{T}{2M}
$$

### Puntos por equipo y partido

$$
\pi_P=\frac{P}{2M}
$$

### Precisión de tiro de la liga

$$
\pi_{Prec}=\frac{T}{S}
$$

### Definición de la liga

$$
\pi_{Def}=\frac{G}{T}
$$

### Resistencia defensiva de la liga

$$
\pi_{Res}=
\frac{T-G}{T}
$$

### Puntos del local por partido

$$
\pi_{PH}=
\frac{P_H}{M}
$$

### Puntos del visitante por partido

$$
\pi_{PA}=
\frac{P_A}{M}
$$

### Diferencia de goles local

$$
\pi_{GD,H}=
\frac{G_H-G_A}{M}
$$

### Diferencia de goles visitante

$$
\pi_{GD,A}=
\frac{G_A-G_H}{M}
$$

Por construcción:

$$
\pi_{GD,A}=-\pi_{GD,H}
$$

### Goles totales por partido

$$
\pi_{GT}=
\frac{G}{M}
$$

### Tiros a portería totales por partido

$$
\pi_{TT}=
\frac{T}{M}
$$

Estos priors se recalculan constantemente usando **solo partidos anteriores**.

---

# 6. Suavizado exacto de ELISA

Para una tasa cuyo total observado es $X$, obtenida en $n$ partidos, ELISA utiliza:

$$
\boxed{
\tilde x=
\frac{X+s\pi_x}{n+s}
}
$$

donde:

* $s$ es `prior_strength`;
* $\pi_x$ es el promedio actual correspondiente de la liga.

Por ejemplo, los goles anotados por partido de un equipo son:

$$
\boxed{
GFPM=
\frac{GF+s\pi_{GF}}{n+s}
}
$$

Los goles recibidos:

$$
\boxed{
GAPM=
\frac{GA+s\pi_{GA}}{n+s}
}
$$

Los tiros:

$$
\boxed{
SPM=
\frac{SF+s\pi_S}{n+s}
}
$$

Los tiros a portería:

$$
\boxed{
SOTPM=
\frac{SOTF+s\pi_T}{n+s}
}
$$

Por tanto, ELISA **no usa simplemente $GF/n$**.

---

# 7. Precisión de tiro

La precisión, necesita un suavizado ligeramente diferente.

Se define primero una fuerza efectiva del prior:

$$
s_{Prec}=
s\max(\pi_S,1)
$$

Entonces:

$$
\boxed{
Prec=
\frac{
SOTF+s_{Prec}\pi_{Prec}
}{
SF+s_{Prec}
}
}
$$

Esto es exactamente lo que implementa el código.

---

# 8. Definición

Se calcula:

$$
s_{Def}=
s \max(\pi_T,1)
$$

y:

$$
\boxed{
Def=
\frac{
GF+s_{Def}\pi_{Def}
}{
SOTF+s_{Def}
}
}
$$

Por tanto, mide la conversión:

$$
\text{tiros a portería}\rightarrow\text{goles}
$$

regularizada hacia la media de la liga.

---

# 9. Resistencia defensiva

Primero se calcula el número de tiros a portería recibidos que no acabaron en gol:

$$
Stops=
\max(SOTA-GA,0)
$$

La fuerza del prior es:

$$
s_{Res}=
s\max(\pi_T,1)
$$

Entonces:

$$
\boxed{
Res=
\frac{
Stops+s_{Res}\pi_{Res}
}{
SOTA+s_{Res}
}
}
$$

Una resistencia alta indica una mayor proporción de tiros a portería recibidos que no terminan en gol.

---

# 10. Variables por condición local/visitante

Para el equipo que jugará como local se usan exclusivamente sus antecedentes como local.

### Puntos como local

$$
\boxed{
PtsCond_L=
\frac{
PTS_H+s\pi_{PH}
}{
n_H+s
}
}
$$

### Diferencia de goles como local

$$
\boxed{
GDCond_L=
\frac{
GD_H+s\pi_{GD,H}
}{
n_H+s
}
}
$$

Para el visitante:

$$
\boxed{
PtsCond_V=
\frac{
PTS_A+s\pi_{PA}
}{
n_A+s
}
}
$$

$$
\boxed{
GDCond_V=
\frac{
GD_A+s\pi_{GD,A}
}{
n_A+s
}
}
$$

---

# 11. Forma reciente

ELISA guarda para cada partido de cada equipo seis cantidades:

$$
PTS_j
$$

$$
GD_j=GF_j-GA_j
$$

$$
SOTD_j=SOTF_j-SOTA_j
$$

$$
GT_j=GF_j+GA_j
$$

$$
TSOT_j=SOTF_j+SOTA_j
$$

y:

$$
Low_j=
\begin{cases}
1, & GF_j+GA_j \le 2 \\
0, & GF_j+GA_j \gt 2 
\end{cases}
$$

ELISA toma los últimos $w$ partidos, donde `form_window` puede ser, por ejemplo, 5 u 8.

### Puntos de forma

$$
PF=
\frac{1}{w}
\sum_{j=1}^{w}PTS_j
$$

### Diferencia de goles de forma

$$
GDF=
\frac{1}{w}
\sum_{j=1}^{w}GD_j
$$

### Diferencia de tiros a portería de forma

$$
SOTDF=
\frac{1}{w}
\sum_{j=1}^{w}SOTD_j
$$

### Goles totales de forma

$$
GTF=
\frac{1}{w}
\sum_{j=1}^{w}GT_j
$$

### Tiros a portería totales de forma

$$
TSOTF=
\frac{1}{w}
\sum_{j=1}^{w}TSOT_j
$$

### Propensión a marcador bajo

$$
LowF=
\frac{1}{w}
\sum_{j=1}^{w}Low_j
$$

Estos promedios recientes **no reciben el suavizado mediante $s$**.

---

# 12. Las 9 variables de balance

Con los perfiles anteriores, para un partido local $L$ contra visitante $V$, ELISA construye:

### $b_1$: diferencia de goles por partido

Primero:

$$
GDPM_L=GFPM_L-GAPM_L
$$

$$
GDPM_V=GFPM_V-GAPM_V
$$

Después:

$$
\boxed{
b_1=GDPM_L-GDPM_V
}
$$

---

### $b_2$: diferencia de puntos por condición

$$
\boxed{
b_2=PtsCond_L-PtsCond_V
}
$$

---

### $b_3$: diferencia de goles por condición

$$
\boxed{
b_3=GDCond_L-GDCond_V
}
$$

---

### $b_4$: diferencia de puntos de forma

$$
\boxed{
b_4=PF_L-PF_V
}
$$

---

### $b_5$: diferencia de goles de forma

$$
\boxed{
b_5=GDF_L-GDF_V
}
$$

---

### $b_6$: diferencia de tiros a portería de forma

$$
\boxed{
b_6=SOTDF_L-SOTDF_V
}
$$

---

### $b_7$: diferencia de precisión

$$
\boxed{
b_7=Prec_L-Prec_V
}
$$

---

### $b_8$: diferencia de definición

$$
\boxed{
b_8=Def_L-Def_V
}
$$

---

### $b_9$: diferencia de resistencia

$$
\boxed{
b_9=Res_L-Res_V
}
$$

El vector de balance es entonces:

$$
\boxed{
\mathbf b=
[b_1,b_2,b_3,b_4,b_5,b_6,b_7,b_8,b_9]
}
$$

---

# 13. Las 9 variables de intensidad

Paralelamente se construyen:

### $i_1$: goles anotados combinados

$$
\boxed{
i_1=GFPM_L+GFPM_V
}
$$

### $i_2$: goles recibidos combinados

$$
\boxed{
i_2=GAPM_L+GAPM_V
}
$$

### $i_3$: tiros combinados

$$
\boxed{
i_3=SPM_L+SPM_V
}
$$

Ojo: aquí son **tiros a favor**, no diferencia tiros a favor menos tiros recibidos.

### $i_4$: tiros a portería combinados

$$
\boxed{
i_4=SOTPM_L+SOTPM_V
}
$$

### $i_5$: definición combinada

$$
\boxed{
i_5=Def_L+Def_V
}
$$

### $i_6$: vulnerabilidad combinada

La vulnerabilidad individual es:

$$
Vul=1-Res
$$

Por tanto:

$$
\boxed{
i_6=(1-Res_L)+(1-Res_V)
}
$$

### $i_7$: goles totales recientes

$$
\boxed{
i_7=
\frac{GTF_L+GTF_V}{2}
}
$$

### $i_8$: tiros a portería totales recientes

$$
\boxed{
i_8=
\frac{TSOTF_L+TSOTF_V}{2}
}
$$

### $i_9$: propensión a marcador bajo

$$
\boxed{
i_9=
\frac{LowF_L+LowF_V}{2}
}
$$

Así:

$$
\boxed{
\mathbf i=
[i_1,i_2,i_3,i_4,i_5,i_6,i_7,i_8,i_9]
}
$$

---

# 14. Cómo se obtiene el balance latente $B$

Aquí comienza el aprendizaje del modelo.

Tenemos una matriz de entrenamiento:

$$
X_B=
\begin{bmatrix}
b_{11} & \cdots & b_{19} \\
b_{21} & \cdots & b_{29} \\
\vdots & & \vdots \\
b_{N1} & \cdots & b_{N9}
\end{bmatrix}
$$

Para cada columna se calcula la **media de entrenamiento**:

$$
\mu_j=
\frac1N\sum_{k=1}^{N}X_{kj}
$$

y la desviación estándar poblacional:

$$
\sigma_j=
\sqrt{
\frac1N
\sum_{k=1}^{N}(X_{kj}-\mu_j)^2
}
$$

El código utiliza:

$$
ddof=0
$$

Si:

$$
\sigma_j<10^{-8}
$$

se sustituye por:

$$
\sigma_j=1
$$

Cada variable se transforma exactamente mediante:

$$
\boxed{
Z_{kj}=
\frac{X_{kj}-\mu_j}{\sigma_j}
}
$$

---

# 15. Objetivo del modelo de balance

Para cada partido de entrenamiento:

$$
\boxed{
y_B=G_L-G_V
}
$$

Por ejemplo:

* 2-0: $y_B=2$;
* 1-1: $y_B=0$;
* 0-3: $y_B=-3$.

Se añade una columna de unos:

$$
A_B=
\begin{bmatrix}
1 & Z_{11} & \cdots & Z_{19} \\
1 & Z_{21} & \cdots & Z_{29} \\ 
\vdots & & & \vdots
\end{bmatrix}
$$

---

# 16. Ridge exacta para $B$

Los parámetros se calculan mediante:

$$
\boxed{
\hat{\boldsymbol\beta}=
\left(
A_B^TA_B+
\lambda_RP
\right)^{-1}
A_B^Ty_B
}
$$

donde:

$$
P=
\begin{bmatrix}
0 & 0 & \cdots & 0 \\
0 & 1 & & 0 \\
\vdots & & \ddots & \\
0 & 0 & & 1
\end{bmatrix}
$$

El intercepto **no se penaliza**.

El parámetro:

$$
\lambda_R
$$

es `ridge_latent`.

Por tanto, para un nuevo partido:

$$
\boxed{
B=
\beta_0+
\sum_{j=1}^{9}\beta_j
\frac{b_j-\mu_{B,j}}{\sigma_{B,j}}
}
$$

Finalmente:

$$
\boxed{
B\leftarrow\min(6,\max(-6,B))
}
$$

Es decir:

$$
-6 \le B \le 6
$$

---

# 17. Cómo se obtiene la intensidad latente $I$

Exactamente el mismo procedimiento se aplica al vector de intensidad.

Cada característica se estandariza usando **sus propias medias y desviaciones de entrenamiento**:

$$
Z_{I,j}=
\frac{i_j-\mu_{I,j}}{\sigma_{I,j}}
$$

El objetivo ahora es:

$$
\boxed{
y_I=G_L+G_V
}
$$

Ejemplos:

* 0-0: $y_I=0$;
* 1-1: $y_I=2$;
* 3-2: $y_I=5$.

Se estima otra regresión Ridge:

$$
\boxed{
\hat{\boldsymbol\gamma}=
\left(
A_I^TA_I+
\lambda_RP
\right)^{-1}
A_I^Ty_I
}
$$

Utiliza **el mismo valor de `ridge_latent`** que el modelo de balance.

Para un partido nuevo:

$$
\boxed{
I=
\gamma_0+
\sum_{j=1}^{9}
\gamma_j
\frac{i_j-\mu_{I,j}}{\sigma_{I,j}}
}
$$

y después:

$$
\boxed{
I\leftarrow\min(8,\max(0,I))
}
$$

Por tanto:

$$
0 \le I \le 8
$$

---

# 18. De $B$ e $I$ a las variables finales

ELISA sustituye primero:

$$
b=|B|
$$

y vuelve a limitarlo:

$$
0 \le b \le 6
$$

La intensidad también permanece:

$$
0 \le I \le 8
$$

Después construye exactamente seis columnas:

$$
\boxed{
\mathbf m=
[
1,
b,
b^2,
I,
I^2,
bI
]
}
$$

Es decir:

$$
\boxed{
\mathbf m=
[
1,
|B|,
|B|^2,
I,
I^2,
|B|I
]
}
$$

---

# 19. Variable objetivo final

Se define:

$$
\boxed{
y_D=
\begin{cases}
1, & G_L = G_V \\
0, & G_L \neq G_V
\end{cases}
}
$$

---

# 20. Regresión logística exacta

El predictor es:

$$
\boxed{
\eta=
\theta_0+
\theta_1|B|+
\theta_2|B|^2+
\theta_3I+
\theta_4I^2+
\theta_5|B|I
}
$$

La probabilidad de empate es:

$$
\boxed{
P(D)=
\frac{1}{1+\exp(-\eta)}
}
$$

Antes de calcular la exponencial, el código limita:

$$
-30 \le \eta \le 30
$$

por estabilidad numérica.

> Durante el ajuste de la instancia de ELISA, la regresión logística se entranan con los valores $B$ e $I$ generados por los modelos Ridge ajustados sobre ese mismo conjunto de entrenamiento. La validación fuera de muestra se realiza posteriormente a nivel de la arquitectura completa. 

---

# 21. Función que optimiza la regresión logística

Esto también es importante para reproducirla exactamente.

ELISA minimiza la **suma**, no la media, de la entropía cruzada:

$$
-\sum_{k=1}^{N}
\left[
y_k\log(p_k)
+
(1-y_k)\log(1-p_k)
\right]
$$

más una penalización L2:

$$
\lambda_L
\sum_{j=1}^{5}\theta_j^2
$$

Por tanto:

$$
\boxed{
J(\boldsymbol\theta)=-\sum_{k=1}^{N}
\left[
y_k\log(p_k)+
(1-y_k)\log(1-p_k)
\right]+
\lambda_L
\sum_{j=1}^{5}\theta_j^2
}
$$

El intercepto:

$$
\theta_0
$$

**no se penaliza**.

---

# 22. Restricciones de los coeficientes logísticos

Éste es otro detalle importante que antes no había explicitado.

ELISA no deja todos los coeficientes completamente libres.

Sus límites son:

$$
-10 \le \theta_0 \le 10
$$

$$
\boxed{
-6 \le \theta_1 \le 0
}
$$

para (|B|);

$$
\boxed{
-3 \le \theta_2 \le 0
}
$$

para $|B|^2$;

$$
-6 \le \theta_3 \le 6
$$

para $I$;

$$
\boxed{
-3\le\theta_4\le0
}
$$

para $I^2$;

$$
-3\le\theta_5\le3
$$

para $|B|I$.

Esto impone una hipótesis estructural importante:

> El aumento del desequilibrio no puede tener un efecto lineal positivo sobre el logit del empate.

Porque:

$$
\theta_1 \le 0
$$

y:

$$
\theta_2 \le 0
$$

También se obliga:

$$
\theta_4\le0
$$

lo que permite que el efecto de la intensidad presente concavidad.

---

# 23. Inicialización de la logística

Sea la tasa de empates del entrenamiento:

$$
\bar{y}_D=
\frac{1}{N}\sum y_D
$$

Se inicializa:

$$
\theta_0^{(0)}=
\log
\left(
\frac{\bar y_D}{1-\bar y_D}
\right)
$$

Además:

$$
\theta_1^{(0)}=-0.5
$$

$$
\theta_4^{(0)}=-0.02
$$

y:

$$
\theta_2^{(0)}=
\theta_3^{(0)}=
\theta_5^{(0)}=
0
$$

La optimización se realiza mediante:

**L-BFGS-B**

con:

* máximo de 800 iteraciones;
* `ftol = 10^{-12}`;
* gradiente analítico.

---

# 24. Relación exacta con las 12 variables de Carolina

Ahora podemos establecerla correctamente.

| Carolina                    | ELISA                  |
| --------------------------- | ---------------------- |
| 1. diferencia goles/partido | $b_1$                  |
| 2. diferencia tiros/partido | no existe directamente |
| 3. puntos condición         | $b_2$                  |
| 4. goles condición          | $b_3$                  |
| 5. puntos forma             | $b_4$                  |
| 6. goles forma              | $b_5$                  |
| 7. tiros a portería forma   | $b_6$                  |
| 8. precisión                | $b_7$                  |
| 9. definición               | $b_8$                  |
| 10. resistencia             | $b_9$                  |
| 11. amarillas               | no se utiliza          |
| 12. rojas                   | no se utiliza          |

Pero hay una salvedad crucial:

$$
\boxed{
b_j^{ELISA}
\neq
x_j^{Carolina}
}
$$

en general.

Aunque conceptualmente representen lo mismo, **ELISA aplica priors dinámicos de la liga**.

Por ejemplo, Carolina puede calcular una precisión usando:

$$
\frac{SOTF+1}{SF+2}
$$

mientras que ELISA utiliza:

$$
\frac{
SOTF+s_{Prec}\pi_{Prec}
}{
SF+s_{Prec}
}
$$

Son variables relacionadas, pero no idénticas.

---

# 25. ¿Qué hizo ELISA con la diferencia de tiros de Carolina?

No la incorporó al bloque de balance.

En su lugar utiliza:

$$
\boxed{
i_3=SPM_L+SPM_V
}
$$

Es decir, transforma la idea de los tiros desde:

$$
\text{¿quién domina en tiros?}
$$

hacia:

$$
\text{¿cuánto volumen de tiros aportan conjuntamente?}
$$

Eso la convierte en una variable de **intensidad**, no de balance.

---

# 26. ¿Qué ocurrió con amarillas y rojas?

Simplemente se eliminaron.

ELISA ver03 **no las calcula ni las utiliza**.

---

# 27. Sintonización exacta de ELISA

Para cada liga se prueban inicialmente:

### Ventana de forma

$$
w \in \\{ 5,8 \\}
$$

### Fuerza del prior

$$
s \in \\{ 3,6 \\}
$$

### Ridge latente

$$
\lambda_R \in \\{ 0.1,1,10 \\}
$$

### L2 logística

$$
\lambda_L \in \\{ 0.01,0.1,1 \\}
$$

Esto produce:

$$
2\times2\times3\times3=36
$$

configuraciones.

---

# 28. Cómo se elige una configuración sin mezclar temporadas

Supongamos:

* temporada A = 2022-2023;
* temporada B = 2023-2024.

Para cada combinación de hiperparámetros se hacen dos folds:

### Fold 1

Entrenar:

$$
2022\text{-}2023
$$

predecir:

$$
2023\text{-}2024
$$

### Fold 2

Entrenar:

$$
2023\text{-}2024
$$

predecir:

$$
2022\text{-}2023
$$

Las predicciones de ambos folds se concatenan.

Sobre ellas se calcula el criterio, por defecto:

$$
\boxed{\text{logloss de empate}}
$$

La configuración con menor logloss OOF es la seleccionada.

---

# 29. Modelo final congelado

Después de elegir:

$$
(w,s,\lambda_R,\lambda_L)
$$

se concatenan los ejemplos elegibles de:

$$
2022\text{-}2023
$$

y:

$$
2023\text{-}2024
$$

y se vuelve a entrenar **desde cero** el modelo definitivo:

1. Ridge de balance;
2. Ridge de intensidad;
3. regresión logística.

Este modelo queda congelado.

Luego se aplica sin modificar parámetros a:

$$
2024\text{-}2025
$$

y:

$$
2025\text{-}2026
$$

---

# 30. Calentamiento de equipos

Un partido solo genera una predicción si ambos equipos tienen:

$$
\boxed{n\ge8}
$$

partidos anteriores en esa temporada.

Si alguno tiene menos, el partido se omite.

Los ocho partidos no tienen que ser exactamente las primeras ocho jornadas del campeonato; se cuentan individualmente por equipo.

---

# 31. Regla adaptativa de ELISA ver03

La probabilidad:

$$
P(D)
$$

queda congelada cuando se genera.

Supongamos que antes de un nuevo partido existen $n$ probabilidades prepartido de partidos elegibles ya concluidos:

$$
p_1,p_2,\ldots,p_n
$$

Todas pertenecen a la **temporada actual**.

Se ordenan de mayor a menor:

$$
p_{(1)}
\ge
p_{(2)}
\ge
\cdots
\ge
p_{(n)}
$$

Para una cobertura:

$$
q=0.10
$$

se calcula:

$$
\boxed{
k=
\max
\left(
1,
\lceil qn\rceil
\right)
}
$$

El umbral es:

$$
\boxed{
T=p_{(k)}
}
$$

es decir, el (k)-ésimo valor más alto.

---

# 32. Decisión final

Cuando existen al menos 30 probabilidades históricas:

$$
n\ge30
$$

ELISA decide:

$$
\boxed{
P(D)_{\text{nuevo}} \ge T
\Rightarrow EMPATE
}
$$

y:

$$
\boxed{
P(D)_{\text{nuevo}} \lt T
\Rightarrow ABSTENERSE
}
$$

Si:

$$
n<30
$$

la salida es:

$$
\boxed{
CONSTRUYENDO \textunderscore HISTORIAL
}
$$

---

# 33. Percentil histórico exacto

Para una probabilidad nueva $p$:

$$
\boxed{
Percentil=100\frac{ \\# \\{ p_j \le p \\} }{n}
}
$$

Así, un percentil 93 significa que el nuevo valor es igual o superior aproximadamente al 93 % de las puntuaciones históricas.

---

# 34. Qué entra al historial

Éste es un detalle fundamental para reproducir ver03.

Al historial entran:

$$
\boxed{\text{TODOS los partidos elegibles concluidos}}
$$

No solamente:

* los empates;
* los seleccionados;
* los aciertos.

Entran todas sus probabilidades prepartido:

$$
p_1, p_2, \ldots
$$

El **resultado real no se usa para construir el percentil**.

---

# 35. Tratamiento de una misma fecha

Supongamos que antes de una jornada tenemos:

$$
H=
[p_1,\ldots,p_{100}]
$$

y ese día hay diez encuentros.

Los diez se evalúan contra exactamente:

$$
H
$$

No ocurre:

$$
H \rightarrow H+p_{101}
$$

antes de evaluar el siguiente partido.

Solo después de calcular los diez:

$$
H_{\text{nuevo}}=
H+
[p_{101}, \ldots, p_{110}]
$$

---

# 36. ELISA completo en una sola cadena matemática

La arquitectura reproducible puede resumirse así:

$$
\boxed{
\text{partidos anteriores}
}
$$

$$
\downarrow
$$

$$
\boxed{
\text{estado de equipos + estado de liga}
}
$$

$$
\downarrow
$$

$$
\boxed{
\text{priors dinámicos de liga}
}
$$

$$
\downarrow
$$

$$
\boxed{
\mathbf b\in\mathbb R^9,\qquad
\mathbf i\in\mathbb R^9
}
$$

$$
\downarrow
$$

Estandarización independiente:

$$
z_j=\frac{x_j-\mu_j}{\sigma_j}
$$

$$
\downarrow
$$

Dos modelos Ridge:

$$
\boxed{
B=\beta_0+\mathbf z_B^T\boldsymbol\beta
}
$$

$$
\boxed{
I=\gamma_0+\mathbf z_I^T\boldsymbol\gamma
}
$$

$$
\downarrow
$$

Clipping:

$$
B\in[-6,6],
\qquad
I\in[0,8]
$$

$$
\downarrow
$$

Transformación:

$$
\boxed{
[1,|B|,|B|^2,I,I^2,|B|I]
}
$$

$$
\downarrow
$$

Regresión logística:

$$
\boxed{
P(D)=
\frac{1}{
1+\exp[
-(\theta_0+
\theta_1|B|+
\theta_2|B|^2+
\theta_3I+
\theta_4I^2+
\theta_5|B|I)
]
}
}
$$

$$
\downarrow
$$

Historial de probabilidades prepartido concluidas:

$$
H_t={p_1,\ldots,p_{t-1}}
$$

$$
\downarrow
$$

Corte top 10 %:

$$
T_t=p_{(\lceil0.10n\rceil)}
$$

$$
\downarrow
$$

$$
\boxed{
P(D)_t\ge T_t
\Rightarrow EMPATE
}
$$

o:

$$
\boxed{
P(D)_t \lt T_t
\Rightarrow ABSTENERSE
}
$$

---

Eso ya es una **especificación de ELISA ver03 suficientemente explícita para que otra persona pueda reimplementarla sin ver el código**, salvo naturalmente los valores numéricos finales de (\boldsymbol\beta), (\boldsymbol\gamma), medias, desviaciones y (\boldsymbol\theta), que dependen de la liga y quedan almacenados en `modelo_elisa_ver03.json`. Para reproducir **un modelo ya entrenado exactamente**, esos valores deben conservarse; para reproducir **el procedimiento de entrenamiento**, las ecuaciones y reglas anteriores son las que usa la implementación.
