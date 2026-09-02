
## Respuesta misión 1

| Problema | Capa | Patrón que lo resuelve | Por qué usar este y no otro | Cuándo no aplicarlo |
| :--- | :--- | :--- | :--- | :--- |
| Hay que copiar el código de la sesión y bitácora en cada archivo. | Políticas transversales | Intercepting Filter (como un middleware de Express) | Porque intercepta y limpia la petición antes de que toque la lógica. No es Front Controller porque ese solo enruta, no aplica reglas a todos los request. | No se usa si la validación es muy específica para una sola pantalla y no aplica para el resto del portal. |
| El script de pagos es un switch gigante y usa jerga del banco. | Aplicación / Dominio | Strategy + Adapter | Strategy te deja cambiar el método de pago sin tocar el código principal. Adapter traduce su lenguaje al nuestro. No es Facade porque no orquesta varios sistemas, solo traduce. | Strategy sobra si el sistema jamás aceptará otra forma de pago. Adapter no sirve si la API del banco ya te responde limpio. |
| Los queries de SQL están repetidos y pegados directamente en las vistas. | Datos | Repository | Saca la persistencia de las vistas y centraliza los queries para devolver objetos limpios. A diferencia de Active Record, no acopla la tabla directamente a la lógica. | En un CRUD muy básico donde un ORM rápido soluciona facilmente sin agregar tantas capas. |
| El registro se rompe si falla el correo, si dan doble clic, cobran doble. | Aplicación / Integración | Observer + Unit of Work | Unit of Work asegura la transacción en la BD. Observer dispara el correo en segundo plano para que, si falla, no afecte la inscripción principal. | Observer no sirve si el paso secundario es crítico y obligatorio para que el flujo principal tenga sentido. |
| La app móvil sufre haciendo 12 peticiones y el quiosco exige otro formato. | Presentación | Facade (Backend for Frontend) | Envuelve las 12 llamadas y arma un solo endpoint a la medida de la app. No es Adapter porque aquí sí estás resumiendo múltiples salidas en una. | Si el equipo frontend ya usa GraphQL y ellos mismos pueden decidir qué datos pedir dinámicamente. |
| Se cae el sistema del banco/CURP y la página se queda cargando infinito. | Integración | Circuit Breaker | Corta la conexión rápido si el servicio externo está caído y devuelve un error controlado, en lugar de trabar el servidor. | En llamadas locales dentro del mismo servidor donde la red no va a fallar de la nada. |

## Respuesta misión 2

1. Enrutador: El request llega primero al front controller. Es la puerta de entrada que recibe todas las peticiones web y sabe a dónde despacharlas.
2. Middleware: Antes de procesar el pago, pasa por un intercepting filter para checar que el alumno tenga la sesión activa y registrar el acceso en la bitácora.
3. Controlador: El page controller agarra los datos del formulario de pago. Su única funcion es delegar el trabajo, no hacer los cálculos.
4. Servicio: Entra a la lógica de negocio. Para evitar el doble cobro por el doble clic rápido, aplicamos el patrón idempotent receiver (con una llave de sesión o folio) que bloquea peticiones duplicadas.
5. Motor de Pagos: Usando strategy, el código decide de forma limpia qué algoritmo ejecutar dependiendo de si el alumno eligió tarjeta, SPEI o ventanilla.
6. Cliente de Integración: Para comunicarnos con el banco, pasamos por un adapter. Esto agarra sus códigos raros de "créditos" y los traduce internamente a "pagado" o "rechazado".
7. Base de Datos: Para guardar el pago y el alta de materias, el mecanismo usa unit of work. Esto arma una sola transacción SQL; si algo falla, hace un rollback limpio y no deja datos a medias.
8. Gestor de Eventos: Una vez guardado el registro, el sistema avisa que el pago pasó. Un observer escucha este evento y dispara la llamada asíncrona para mandar el correo, sin poner en riesgo la inscripción si el servidor de correos no responde.