# Análisis del Dataset

## Actividad 1 — El retrato del dataset

 1. Predicción: 
* Cuando llueve, dominará la camioneta o el automóvil.
* Para el centro de la ciudad, dominará la bicicleta o la moto.
* La distancia máxima de una bicicleta será de alrededor de 5 a 7 km.
* Sí habrá envíos que rompan las reglas de negocio, ya que los datasets reales siempre contienen ruido o excepciones operativas.

 2. Comprobación: 
* Lluvia vs Medio: Con lluvia, el 70.83% es camioneta y el 29.17% moto. Dron y bicicleta representan el 0%.
* Centro vs Medio: En el centro, el 74.07% es bicicleta, 21.01% moto, y 4.90% camioneta. Dron es el 0%.
* Resumen numérico: La distancia máxima de la bicicleta es exactamente 7.00 km.
* Comparación: Mi predicción sobre los límites físicos fue correcta, el clima adverso restringe fuertemente a los vehículos descubiertos y ligeros.

 3. Decisión (Preguntas): 
1.  Zonas y climas sin dron:  El dron nunca aparece en el "centro" ni con "lluvia". La regla del paso 12 lo prohíbe explícitamente (paquete["clima"] != "despejado" o paquete["zona_destino"] == "centro") por seguridad aérea y regulaciones urbanas.
2.  Bicicleta a la periferia:  La columna que lo explica es distancia_km. El máximo que recorre una bicicleta en los datos es de  7.00 km .
3.  Límites adivinables:  Se pueden adivinar los topes físicos de  distancia , peso  y  volumen  mirando el valor "max" por medio. No se pueden adivinar los límites dinámicos (como la prohibición del dron de noche o en lluvia) porque requeriría aislar múltiples variables categóricas de forma conjunta.
4.  Camionetas en < 2 km:  Hay  174  camionetas para viajes de menos de 2 km. Aquellas que sobrepasen el peso o volumen máximo de una moto (25 kg o 90 L) tienen razón de ser. Para las que son paquetes pequeños, la explicación es que estaba lloviendo, lo que inhabilita bicicletas y drones.
*  Para pensar:  A partir de los datos solo se pueden deducir los límites duros (como máximos absolutos). Reconstruir toda la lógica del negocio solo con un modelo predictivo tiene el enorme riesgo de que este descubra sesgos y correlaciones accidentales en lugar de reglas estrictas de operación, mandando, por ejemplo, drones cuando llueve si es que "no aprendió" esa regla por falta de datos anómalos.

---

## Actividad 2 — La hora y el dron

 1. Predicción: 
* A las 3:00 saldrán 0 drones por limitaciones visuales nocturnas.
* El porcentaje de envíos nocturnos será bajo, cercano al 10-15%.

 2. Comprobación: 
* Tabla hora vs dron: A las 3:00, hay 0 envíos en dron.
* Porcentaje de noche (< 6 o >= 21):  10.96% 
* Comparación: Mi predicción fue precisa respecto al bajo volumen nocturno de operación.

 3. Decisión (Preguntas): 
1.  Horas sin dron:  Entre las 21:00 y las 5:00 de la mañana hay exactamente 0 drones. Coincide exactamente con la regla  noche = paquete["hora_salida"] >= 21 or paquete["hora_salida"] < 6 .
2.  Envíos de noche vs día:  Representan el 10.96%. Al ser casi el 11%, el modelo capta una señal débil de la noche comparada con las horas pico del día, haciendo que sus predicciones nocturnas sean menos robustas frente a las diurnas.
3.  Bicicletas a las 20:00:  Disminuyen drásticamente para las 20:00 y desaparecen a las 21:00 porque el negocio penaliza el envío en bicicleta en horarios de baja visibilidad por seguridad del trabajador.
4.  Árbol y la noche:  El árbol hace divisiones simples ( variable <= valor ). Para delimitar un rango en una variable cíclica (como la noche, que cubre el fin y el inicio del día), necesita al menos  dos preguntas  (ej:  hora_salida <= 5  y  hora_salida >= 21 ).
*  Para pensar:  Crearía una columna derivada booleana:  es_de_noche = (hora_salida < 6) | (hora_salida >= 21) . Podría comprobar su efectividad entrenando de nuevo y validando si esa variable entra en el feature_importances_ con un valor superior a cero o comprobando si disminuyen los falsos positivos nocturnos.

---

## Actividad 3 — Anatomía de los errores

 1. Predicción: 
* La confusión principal será entre medios adyacentes en límites de capacidad, como motocicleta y bicicleta.

 2. Comprobación: 
* Top 3 confusiones del modelo:  moto  clasificado como  bicicleta  (23 veces),  camioneta  clasificado como  moto  (16 veces) y  bicicleta  clasificada como  moto  (16 veces).
* Comparación: Correcto, el modelo se confunde en las "zonas grises" operativas.

 3. Decisión (Preguntas): 
1.  Errores frecuentes:  Hay confusión principalmente entre medios de transporte de dimensiones y alcances adyacentes. Esto nos dice que las fronteras naturales (como envíos de 5 a 10 km) son donde el modelo requiere mayor precisión de los atributos del paquete.
2.  Motos mandadas a Dron:  De los  29  errores donde era moto pero predijo dron,  26  ocurrieron de noche. Las reglas de negocio sí corregirán los nocturnos (al forzar un descarte mediante  es_posible ), pero para los 3 restantes el modelo falló porque muy probablemente estaban justo en el límite de peso (2.5 kg).
3.  Motos mandadas a Bici:  Se deben al solapamiento en distancias cortas (~6.5 a 7 km). Lo que distingue a estas motos de las bicicletas reales que el modelo sí acierta es típicamente un factor de volumen o peso que la bici no puede cargar.
*  Para pensar:  Elegiría arreglar los falsos positivos de drones. Modificando  los datos  (agregando la variable  es_de_noche ) evitas que el modelo aprenda a adivinar mal, sin tener que sobrecargar la lógica dura del negocio, y evitas el costo enorme de desplegar un dron erróneamente.

---

## Actividad 4 — ¿Qué columnas usa de verdad el modelo?

 1. Predicción: 
* Más importantes:  distancia_km ,  peso_kg ,  volumen_l .
* Menos importante:  zona_destino .

 2. Comprobación: 
* Las cuatro principales importancias calculadas por scikit-learn fueron:  distancia_km  (0.322),  volumen_l  (0.193),  refrigerado  (0.131) y  peso_kg  (0.126).
* Comparación: Acerté en las variables físicas, aunque me sorprendió que  refrigerado  fuera tan vital para los nodos superiores.

 3. Decisión (Preguntas): 
1.  Top 4 columnas:   distancia_km ,  volumen_l ,  refrigerado  y  peso_kg . Sí coinciden con las limitantes primordiales operativas dictadas en el paso 8 y con mis predicciones base.
2.  Impacto al quitar:  Quitar variables como la distancia alteraría sustancialmente la métrica de exactitud, porque todo el ecosistema perimetral depende de ella, mientras que variables como la zona_destino podrían retirarse casi sin penalización en el accuracy.
3.  Misterio de la zona cero:  El modelo no utiliza  zona_destino  porque esta información ya se manifiesta a través de sus fuertes correlaciones (ej. el centro suele presentar distancias muy cortas y mucho tráfico). El modelo deduce las limitaciones sin necesidad de usar esa columna específica.
*  Para pensar:  Sí se debe seguir pidiendo. Aunque el árbol de decisiones no ocupe esa columna, los algoritmos de logística (ruteo GPS), las facturas de la empresa y los dashboards de inteligencia de negocios necesitan conocer a dónde van los envíos, independiente del modelo de machine learning.

---

## Actividad 5 — El dron es minoría: ¿hay que balancear el dataset?

 Parte A: 
1. Un modelo que nunca dijera dron y acertara siempre en lo demás obtendría un  90.5% de exactitud . Esto nos demuestra que el *accuracy* maquilla totalmente la ineficacia del modelo en clases minoritarias, por lo que es preferible utilizar la Precision, el Recall y el F1-Score.
2. Hay pocos drones porque refleja el ecosistema actual de la empresa (son pocos los equipos, regulaciones de vuelo). El desbalance no es un error de recolección, sino un retrato de la realidad.
3. Se balancea estrictamente  después  de aislar la muestra de prueba, y el conjunto de prueba  nunca  se balancea, porque este debe funcionar como una fotografía imparcial de la vida real de la empresa en producción.

 Parte B - Estrategias (Tabla de Métricas sobre 2000 casos de prueba) 

| Estrategia | Exactitud total | Precision dron | Recall dron | Veces que predice dron (Reales: 190) | Sugerencias imposibles |
| :--- | :--- | :--- | :--- | :--- | :--- |
|  Sin balancear  | 0.92 | 0.84 | 0.91 | 205 | 31 |
|  Con balanceo (class_weight)  | 0.90 | 0.77 | 1.00 | 248 | 35 |

 Parte C: 
1.  Precision vs Recall:  Al forzar los pesos para darle importancia a los drones, el recall sube al  1.00  (detecta absolutamente todos los drones útiles), pero el *precision* cae a  0.77  porque empezó a clasificar equivocadamente otras instancias dudando si eran dron (aumentando falsos positivos).
2.  Estrategia con menos datos:  Si se usa la estrategia de submuestreo (undersampling), terminamos tirando datos valiosos de las motos a la basura, perdiendo patrones de comportamiento muy enriquecedores para el árbol.
3.  Misma idea:  Oversampling (repetir datos) y usar pesos ponderados (class_weight) operan bajo el mismo motor: obligan matemáticamente a la función de error del modelo a castigar fuertemente cada fallo donde sí se requería un dron.
4.  Confianza tergiversada:  Tras balancear, el "84% de confianza" reportado ya no representa la probabilidad de la vida real. Es una métrica distorsionada porque la herramienta fue entrenada en una simulación de proporciones artificiales.

 Parte D - Para pensar (Recomendación final): 
Recomiendo  no forzar el balanceo  a costa de inflar las sugerencias imposibles. Para la plataforma, el costo de fallar por "mandar un dron" (falso positivo) implica riesgos de regulaciones y pérdida económica considerable del equipo por accidentes (por ejemplo, en climas erráticos); la  Precision  alta de 0.84 sin balancear resguarda mejor los activos de la empresa.
Aumentar indiscriminadamente el Recall eleva las predicciones de dron a 248, incrementando inevitablemente las sugerencias imposibles (de 31 a 35), que deben ser filtradas duramente en backend gastando cómputo. Además, si mañana la empresa compra el doble de drones, los nuevos flujos orgánicos de datos balancearán gradualmente el dataset con la vida operativa real de la empresa sin falsificar proporciones.