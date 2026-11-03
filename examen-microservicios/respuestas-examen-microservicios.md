
# EXAMEN — MICROSERVICIOS, GATEWAY Y EL MONOLITO CON API
## Tópicos Selectos de Tecnologías Web y Móvil - Eduardo Alcaraz

# Nombre: Figueroa Merino Cesar  


## 1

a) Con la API cree separar al cliente del servidor. Eso tiene sentido si existe un contrato claro, el cliente puede pedir datos sin conocer el código interno de Django. Pero tener una API no garantiza que el cliente deje de depender de detalles internos. Con Docker cree separar los servidores entre sí. El error es que Docker empaqueta y ejecuta aplicaciones, pero no divide sus responsabilidades. Si todo sigue dentro del mismo proyecto y se despliega junto, sigue siendo un monolito.

b) Un proyecto Django con usuarios, pedidos y recomendaciones, que expone una API y se ejecuta completo en un contenedor, sigue siendo un monolito. Para empezar a dividirlo, sacaría recomendar a otro proceso con su propia API y despliegue. Pedidos lo llamaría por red. Así podría actualizar el modelo sin desplegar de nuevo todo Django. También evitaría que ambos servicios modificaran directamente las mismas tablas.

c) La frase habla principalmente de límites y responsabilidades. La API define cómo se comunican cliente y servidor, los microservicios separan partes del sistema que pueden ejecutarse y desplegarse de manera independiente. Docker y REST pueden ayudar, pero no crean esa separación por sí solos.

## 2

a) Las rutas organizan las operaciones disponibles. Tener /usuarios, /pedidos y /recomendar no demuestra que existan tres servicios independientes. Las réplicas ejecutan varias copias de la misma aplicación. Ayudan a repartir carga y atender solicitudes si una copia falla, pero no separan responsabilidades.

Tres réplicas tampoco garantizan alta disponibilidad. Se necesitan reparto de tráfico, detección de fallos y datos que no dependan de una sola instancia.

b) Sigue siendo un monolito porque las tres réplicas ejecutan la aplicación completa. El pedido que estaba en proceso puede quedar sin guardar o haberse guardado antes de perder la respuesta. Repetir el POST sin control podría duplicarlo.

c) Propondría estos servicios:

- gateway: recibe las llamadas públicas y las dirige.
- usuarios: administra usuarios.
- pedidos: crea pedidos y controla sus folios.
- recomendar: carga el modelo y sugiere el transporte.

Cada uno tendría su proceso y podría desplegarse por separado. Para este diseño usaría cuatro imágenes distintas. El mínimo técnico de imágenes distintas puede ser una, porque varios servicios pueden usar la misma imagen con comandos diferentes. Lo importante es que tengan responsabilidades y despliegues independientes. Usar imágenes separadas hace más clara esa independencia y evita empaquetar todo junto.

## 3

a) Tiene funciones de gateway, pero el conjunto funciona como una aplicación monolítica que concentra todo. Es como un recepcionista que también decide el transporte, registra los pedidos y reparte los avisos. Validar el JWT y dirigir solicitudes corresponde al gateway, elegir el transporte y guardar pedidos son parte de la logica del negocio.

b) Tres consecuencias serían:

1. Actualizar el modelo obliga a intervenir el mismo despliegue que recibe clientes y guarda pedidos.
2. Si el correo se envía dentro de la petición, sus fallos pueden retrasar la respuesta o hacerla fallar aunque el pedido ya esté guardado.
3. El modelo pesado comparte recursos con todo lo demás. Si consume mucha memoria o CPU, también afecta la entrada y la creación de pedidos.

c) Lo separaría así:

- En el borde, gateway: valida el JWT, dirige llamadas y puede servir el HTML.
- Detrás, Pedidos: valida la compra, crea el folio y guarda el pedido.
- Detrás, Recomendar: carga el .joblib y propone el transporte.
- Detrás, un trabajador de avisos si hace falta: procesa notificaciones pendientes y reintenta cuando falla el correo.

Pedidos dejaría registrado el aviso pendiente junto con el pedido. Así, una falla del correo no obliga al cliente a repetir la compra.

## 4

a) Dentro de Pedidos, 127.0.0.1 apunta al propio contenedor de Pedidos. La petición busca ahí un proceso que escuche en el puerto 8002, no en el contenedor de Recomendar.

b) http://recomendar:8002 puede funcionar porque Compose permite encontrar los servicios por su nombre dentro de la red compartida. El nombre sale de la definición del servicio y Docker lo resuelve mediante su DNS interno.

También se necesita que Recomendar escuche en el puerto interno 8002 y acepte conexiones desde fuera de su propio localhost.

c) Usaría:


RECOMENDAR_URL=http://192.168.0.57:8002

Deben cumplirse estas dos condiciones:

1. Recomendar debe escuchar en una interfaz accesible, por ejemplo 0.0.0.0:8002. Si está en Docker, debe publicar ese puerto hacia la laptop.
2. La red y el firewall deben permitir la conexión entre los equipos al puerto 8002.

Cambiar la URL no arregla un puerto cerrado, un servidor que solo escucha en localhost o una wifii que impide la comunicación entre alumnos.

## 5

a) El cliente espera aproximadamente 205 ms, más cualquier otro tiempo de procesamiento y red que no esté incluido en el ejercicio. Recomendar aporta 200 ms y el gateway 5 ms. El tiempo dominante lo pone recomendar, los 3 segundos solo se esperan cuando no responde.

b) El cliente ve el pedido creado con respuesta 200.

La respuesta tarda aproximadamente 3055 ms, primero espera el timeout de 3 segundos, después se ejecuta el plan B en 50 ms y se agregan los 5 ms del gateway. La mentira es aceptar moto como una solución válida aunque quizá no sea el mejor transporte. El pedido debe quedar guardado con el medio que realmente se usará.

c) Los reintentos pueden mantener cada petición esperando unos 30 segundos. Durante diez minutos llegarían alrededor de 167 pedidos, suponiendo tráfico uniforme. Los reintentos generan más llamadas al servicio caído y mantienen ocupados recursos de Pedidos y conexiones del gateway. Si se agota la capacidad, las solicitudes se acumulan y el gateway puede terminar respondiendo con timeout.

El plan B no siempre es peor. Si permite cumplir el envío, puede ser mejor responder con una opción aceptable que dejar esperando al cliente. Reintentaría pocas veces ante fallos y dentro de un tiempo máximo. Si el fallo continúa, dejaría de llamar temporalmente a Recomendar y usaría el respaldo.

## 6

Supongo que el gateway sirve el HTML, que es la única entrada pública y que Pedidos tiene el plan B del ejercicio anterior.

1. Se apaga gateway: La página deja de cargar y no se pueden crear pedidos. Pedidos y Recomendar siguen activos y podrían responder si alguien les llama desde la red interna.

2. Se apaga pedidos: El HTML puede cargar porque lo sirve el gateway, pero crear un pedido falla. El gateway puede devolver 502 o 503. Gateway y Recomendar siguen activos.

3. Se apaga recomendar: El HTML carga y todavía se pueden crear pedidos. Pedidos espera hasta el timeout y después usa moto como respaldo. Gateway y Pedidos siguen funcionando.

Si el HTML lo sirviera Pedidos, también dejaría de cargar al apagar ese servicio. Si Pedidos no tuviera plan B, crear pedidos podría fallar al apagar Recomendar.

Al apagar el gateway, el sistema parece más muerto desde fuera, aunque Pedidos y Recomendar sigan funcionando. Esto demuestra que, si toda la entrada depende de un solo gateway, su caída deja al cliente sin acceso al sistema.

## 7

a) El problema es tener dos entradas que los clientes eligen manualmente y que pueden terminar con reglas distintas de autenticación, rutas o límites de uso. Que ambas devuelvan 200 no garantiza que se comporten igual.

Dos instancias de gateway sí pueden ayudar a la disponibilidad, pero deberían ofrecer una misma entrada lógica, compartir configuración y tener un mecanismo para repartir tráfico o cambiar de instancia si una falla.

b) localhost apunta a la máquina de Beto, o al contenedor de Pedidos si corre en Docker, no a Carla. Debe configurar la IP de Carla, por ejemplo:

RECOMENDAR_URL=http://192.168.0.57:8002


Además, Carla debe exponer el servicio y permitir el acceso al puerto.

c) Con aislamiento entre clientes no pueden hacer la práctica tal como está planteada. Que falle ping por sí solo no prueba que HTTP esté bloqueado, pero un aislamiento efectivo sí impide las conexiones necesarias.

Como plan B, ejecutaría gateway, Pedidos y Recomendar en una sola computadora con Compose, cada uno en su contenedor. Se conserva la lección, son procesos separados, se llaman por red y pueden fallar por separado.

## 8

a) 
1. El navegador conoce las IP internas, mover un servicio obliga a cambiar el front.
2. Las llamadas pueden saltarse controles del gateway, y exponer directamente los servicios aumenta los puntos de entrada.
3. El front coordina tareas del negocio, si Recomendar responde y Pedidos falla, debe decidir qué hacer con ese resultado y cómo repetir la operación sin duplicarla.

Además, al usar distintos orígenes aparecen requisitos de CORS.

b) El navegador hace una sola llamada al gateway para crear el pedido. El gateway la dirige a Pedidos. Pedidos consulta a Recomendar, aplica el plan B si hace falta, guarda el pedido y devuelve el folio y el transporte a través del gateway.

Pedidos habla con Recomendar porque controla el flujo de creación y decide qué hacer si la recomendación falla. El navegador no necesita conocer dónde corre ese servicio.

c) Pedidos debe crear el folio. Recomendar recibe los datos y el selector de modelo, como arbol, bosque o red, y devuelve una sugerencia.

Si ambos crean un pedido o un folio por su cuenta, se rompe la regla de que una operación de compra corresponde a un único pedido con una identidad clara. Podrían aparecer duplicados o registros que no coinciden.

## 9

a) cerebro es un nombre demasiado general porque no explica qué responsabilidad tiene.

Mezcla dos tareas diferentes: elegir transporte y generar texto de ventas. Si ambas deben terminar para responder /inteligente, los 8 segundos del LLM hacen imposible cumplir el límite de menos de 500 ms del despacho.


b) Propondría:

- POST /textos-producto: genera el texto de ventas. Lo usa el front de tienda a través del gateway y puede mostrar un texto ya guardado mientras espera.
- POST /sugerir-transporte: devuelve el transporte recomendado. Lo usa Pedidos dentro del flujo de envío.

Los pondría en servicios separados porque tienen dependencias, tiempos y necesidades de recursos diferentes.

c) El front apunta al gateway. El servicio de textos se comunica con el servidor de GPU.

Si cambia el proveedor, modificaría una sola configuración del servicio de textos. Si la API del proveedor también cambia, ajustaría su integración dentro de ese servicio. 

## 10

a)

En siete días podemos reducir riesgos concretos y mejorar la app de clientes.  
El panel funciona y tiene poco tráfico, dividirlo ahora no tiene una ventaja demostrada.  
Con cinco personas, separar seis servicios añade conexiones, despliegues y posibles fallos.  
Si lo hacemos con prisa, podemos romper funciones que hoy sí sirven.  
Propongo proteger el gateway y aislar la actualización del modelo esta semana.  
Después dividimos el panel donde los datos muestren una necesidad real.

b) Mi orden sería 3, 2, 1, 4.

1. Auth y rate limit en el gateway público: protegen una entrada que ya está expuesta y ayudan a controlar accesos y abuso.
2. Separar el reentrenamiento: permite actualizar el modelo sin tumbar Pedidos. El modelo nuevo se valida antes de ponerlo en uso.
3. Partir el panel en seis servicios: lo dejaría pendiente; funciona, tiene poco tráfico y no se menciona un problema que justifique esa división.
4. Subir el LLM sin timeouts ni plan B: no lo llevaría a producción en esas condiciones. Primero necesita límites de espera y una respuesta alternativa.

Ese orden supone que hoy faltan controles en el gateway y que el reentrenamiento no está causando una caída activa. Si ya estuviera tumbando Pedidos, atendería primero esa falla.

c) Una métrica sería, durante cuatro semanas, los reportes del panel consumen más del 60% de la CPU y hacen que el 95% de las demás solicitudes tarde más de dos segundos cuando se ejecutan.
Eso daría una razón observable para evaluar sacar los reportes a un proceso separado, después de revisar si se puede resolver optimizando las consultas.

d) Los microservicios sirven para resolver problemas especificos, tener más servicios no significa tener un mejor sistema.
