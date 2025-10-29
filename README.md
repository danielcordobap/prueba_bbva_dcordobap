### Analisis resultados de prueba Cientifico De Datos:

Para poder visualizar los primeros resultados exploratorios por favor ingresar a este [reporte](https://app.powerbi.com/view?r=eyJrIjoiODVjMTI5M2MtMjMyZi00YmI0LTg2ZmYtZmUxNWU2Nzk2ZmM0IiwidCI6ImRhZjc5OTBlLThhM2YtNDA5Yy05Yjc2LTJhNTQ3NTA5ODAwMCIsImMiOjR9).

Los resultados de este tablero revelan varias conclusiones importantes. En primer lugar, fue necesario intervenir las variables tipo de vivienda (tipovivi) y tipo de contrato (tipocont), reemplazando los valores nulos por la categoría "Sin_Información". Al analizar la variable default en relación con estas dos características, se observa que la mayoría de las personas clasificadas como "Malo" se concentran precisamente en este grupo de "Sin_Información".
Asimismo, la mayor parte de los clientes con mal comportamiento de pago se encuentra en el grupo ocupacional "Empleado". Al enfocarnos específicamente en este segmento y filtrar por los estratos socioeconómicos 2 y 3, encontramos diferencias relevantes en la antigüedad laboral promedio:

Empleados "Buenos": En el estrato 3 permanecen aproximadamente 10 meses en sus empleos, mientras que en el estrato 2 la permanencia es de 9 meses en promedio.
Empleados "Malos": Tanto en el estrato 2 como en el 3, la permanencia promedio es de 8 meses.

Un hallazgo particularmente interesante es que el ingreso promedio resulta ser mayor en las personas clasificadas como "Malas" que en el grupo "Bueno". Sin embargo, esta aparente paradoja se explica al observar que quienes mantienen saldos más altos tienden a presentar un comportamiento de pago deficiente.

Ahora bien ajustando el modelo, para predecir a nuevos clientes basandonos en sus caracteristicas. Tenemos que el metodo que se uso para obtener el modelo mas parsimonico fue ajustar un xgboost y tomar las top 20 variables mas importantes y luego un modelo logistico.

Adicionalmente antes de este ajuste se hicieon un par de transformaciones a algunas variables ( las cuales se podran detallar en el jupyter notebook de este repositorio).

Comparando el performance de ambos modelos, obtuvimos estos resultados:

<img width="970" height="389" alt="image" src="https://github.com/user-attachments/assets/029f117d-50b0-45e5-abf7-1859ede84d26" />
Esto indica que ajustando ambos algoritmos con estas variables : 

<br>

| Variable |
|----------|
| tasa |
| monto_log |
| cuota_log |
| estrato_5 |
| tipocont_sin_informacion |
| tipovivi_sin_informacion |
| niveledu_primaria |
| oficina_c85z |
| niveledu_bachillerato |
| tipocont_termino_indefinido |
| saldo_log |
| oficina_c84a |
| ocupacio_independiente |
| oficina_c89d |
| egrtot_log |
| aportes_log |
| sexo_femenino |
| oficina_c87z |
| estrato_3 |
| ingtot_log |
<br>
El Xgboost tiene mejor performance, pero como tambien necesitamos la interpretabilidad de las columnas vs el riesgo. Estos serian los coeficientes obtenidos en una regresion logistica:

| Variable | Coeficiente | Odds Ratio | Importancia XGB |
|----------|-------------|------------|-----------------|
| tasa | 6.270930 | 528.969174 | 0.108825 |
| monto_log | -1.638842 | 0.194205 | 0.037489 |
| cuota_log | 1.360600 | 3.898531 | 0.030168 |
| estrato_5 | -1.285315 | 0.276563 | 0.022848 |
| tipocont_sin_informacion | 1.170500 | 3.223605 | 0.040426 |
| tipovivi_sin_informacion | 1.125301 | 3.081143 | 0.149683 |
| niveledu_primaria | -0.978438 | 0.375898 | 0.019642 |
| oficina_c85z | 0.442034 | 1.555868 | 0.018410 |
| niveledu_bachillerato | 0.436765 | 1.547692 | 0.022289 |
| tipocont_termino_indefinido | 0.435784 | 1.546174 | 0.045968 |
| saldo_log | 0.378609 | 1.460252 | 0.033259 |
| oficina_c84a | -0.246417 | 0.781596 | 0.018500 |
| ocupacio_independiente | 0.165978 | 1.180547 | 0.019243 |
| oficina_c89d | -0.133893 | 0.874684 | 0.017153 |
| egrtot_log | -0.108610 | 0.897080 | 0.149843 |
| aportes_log | -0.087192 | 0.916501 | 0.027737 |
| sexo_femenino | -0.073820 | 0.928839 | 0.017366 |
| oficina_c87z | 0.073126 | 1.075866 | 0.017032 |
| estrato_3 | 0.072799 | 1.075515 | 0.018788 |
| ingtot_log | -0.066539 | 0.935626 | 0.022064 |

Obteniendo de este resultado estos insights:

1. TASA (Coeficiente: 6.27, OR: 528.97) ⚠️ FACTOR MÁS CRÍTICO

Interpretación: Por cada punto porcentual adicional en la tasa de interés, la probabilidad de default se multiplica por 529 veces.
Implicación: La tasa de interés es el predictor más poderoso del modelo. Tasas altas están fuertemente asociadas con incumplimiento.
Recomendación: Implementar políticas estrictas de pricing basadas en perfil de riesgo.

2. Falta de Información (Tipovivienda y Tipocontrato sin información)

Tipovivienda sin información (OR: 3.08): Clientes que no reportan tipo de vivienda tienen 3 veces más probabilidad de default.
Tipocontrato sin información (OR: 3.22): Ausencia de información contractual incrementa el riesgo 3.2 veces.
Implicación: La falta de información es una señal de alerta crítica.
Recomendación: Establecer requisitos obligatorios de documentación completa y considerar rechazo automático ante información faltante.

3. Cuota del Crédito (Coeficiente: 1.36, OR: 3.90)

Interpretación: Cuotas más altas (en escala logarítmica) aumentan casi 4 veces la probabilidad de incumplimiento.
Implicación: La capacidad de pago del cliente está comprometida con cuotas elevadas.
Recomendación: Evaluar ratio cuota/ingreso y establecer límites máximos de endeudamiento.

4. Monto del Crédito (Coeficiente: -1.64, OR: 0.19)

Interpretación: Créditos de mayor monto se asocian con menos probabilidad de default.
Implicación: Clientes con créditos grandes tienden a ser más confiables (posiblemente mejor perfil crediticio).
Insight: Paradójicamente, prestar más dinero a clientes calificados puede ser menos riesgoso.

5. Estrato Socioeconómico Alto (Estrato 5: OR: 0.28)

Interpretación: Clientes de estrato 5 tienen menos probabilidad de incumplimiento.
Implicación: El nivel socioeconómico es un predictor robusto de buen comportamiento crediticio.

6. Nivel Educativo Primaria (OR: 0.38)

Interpretación: Sorprendentemente, educación primaria reduce el riesgo.
Posible explicación: Puede reflejar un segmento específico (ej: comerciantes informales con flujo de caja estable pero baja educación formal).
Recomendación: Analizar en detalle este segmento para entender el comportamiento.

7. Egresos Totales (Coeficiente: -0.11, OR: 0.90)

Interpretación: Mayores egresos declarados se asocian con menor riesgo.

8 .Saldo del Crédito : Mayor saldo aumenta riesgo.
9. Aportes : Mayores aportes reducen levemente el riesgo.
10. Ingresos Totales : Mayores ingresos reducen riesgo mínimamente.

Ademas de este analisis, se hizo una particion de la poblacion evaluada por tasa de malos y esto se obtuvo:

| Banda Score XGB | Total Clientes | % Clientes | Buenos | Malos | Tasa Malos % | Score Promedio | Score Min | Score Max |
|-----------------|----------------|------------|--------|-------|--------------|----------------|-----------|-----------|
| 0-100 | 24141 | 97.397725 | 24072 | 69 | 0.285821 | 4.710327 | 0 | 100 |
| 100-200 | 248 | 1.000565 | 203 | 45 | 18.145161 | 137.665323 | 101 | 199 |
| 200-300 | 67 | 0.270314 | 37 | 30 | 44.776119 | 242.626866 | 201 | 299 |
| 300-400 | 34 | 0.137174 | 13 | 21 | 61.764706 | 348.117647 | 301 | 400 |
| 400-500 | 17 | 0.068587 | 1 | 16 | 94.117647 | 445.352941 | 403 | 497 |
| 500-600 | 26 | 0.104898 | 1 | 25 | 96.153846 | 538.307692 | 503 | 595 |
| 600-700 | 14 | 0.056483 | 0 | 14 | 100.000000 | 637.214286 | 602 | 696 |
| 700-800 | 20 | 0.080691 | 2 | 18 | 90.000000 | 758.700000 | 702 | 799 |
| 800-900 | 29 | 0.117002 | 2 | 27 | 93.103448 | 859.655172 | 804 | 900 |
| 900-1000 | 190 | 0.766562 | 3 | 187 | 98.421053 | 976.752632 | 906 | 999 |

Con esto tambien se obtuvo un punto de corte para aprobar o rechazar a un nuevo usuario con este modelo, este punto es igual a 100 puntos ( Metodologia optimizacionpor costos asociados). El cual se alinea a los resultados de la tabla ya que despues del segundo grupo la tasa de malos aumenta de forma significativa y este punto de corte se encuentra entre el primer y segundo grupo.


### Conclusiones Finales:

Alta Prioridad 🔴

* Repricing de Cartera: Revisar tasas de interés para evitar sobreendeudamiento.
* Política de Información Completa: Rechazar o asignar mayor tasa a solicitudes con información faltante.
* Límites de Cuota/Ingreso: Establecer ratio máximo de 30-35% de endeudamiento.
* Segmentación por Estrato: Ofrecer mejores condiciones a estratos altos (menor riesgo).

Media Prioridad 🟡

* Capacitación por Oficina: Estandarizar procesos de originación en sucursales de alto riesgo.
* Análisis de Montos: Considerar incrementar tickets promedio para clientes de bajo riesgo.
* Validación de Ingresos Independientes: Implementar verificación adicional para trabajadores por cuenta propia.






