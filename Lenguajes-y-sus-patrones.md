# Lenguajes de Programación y Patrones de Diseño

## Python
   Paradigma:  Multiparadigma (Orientado a objetos, funcional, imperativo).
   Patrones comunes:  Decorator, Factory, Strategy, Iterator.
   Patrones que no encajan:  Abstract Factory o Singleton clásico. No encajan porque resultan excesivamente verbosos y rígidos frente a la simplicidad del tipado dinámico y la estructura de módulos nativa del lenguaje.

## JavaScript / TypeScript
   Paradigma:  Multiparadigma (Basado en prototipos, reactivo, funcional).
   Patrones comunes:  Module, Observer, Middleware, Factory.
   Patrones que no encajan:  Herencia profunda de clases. No encaja porque el lenguaje está diseñado para componer objetos a través de prototipos, por lo que forzar jerarquías rígidas vuelve el código frágil e impredecible.

## Java
   Paradigma:  Orientado a objetos estricto, imperativo.
   Patrones comunes:  Singleton, Builder, Factory Method, DAO.
   Patrones que no encajan:  Active Record. No encaja en aplicaciones empresariales complejas porque acopla fuertemente la lógica de negocio con la base de datos, siendo preferible separar responsabilidades usando Data Mapper.

## C#
   Paradigma:  Multiparadigma (Orientado a objetos, declarativo, funcional).
   Patrones comunes:  Inyección de Dependencias, Unit of Work, Repository.
   Patrones que no encajan:  Service Locator. No encaja porque oculta las dependencias reales de una clase, lo que dificulta el rastreo de errores y complica la ejecución de pruebas unitarias.

## C++
   Paradigma:  Multiparadigma (Sistemas, genérico, orientado a objetos).
   Patrones comunes:  RAII (Resource Acquisition Is Initialization), Pimpl, Factory.
   Patrones que no encajan:  Singleton implementado con punteros crudos. No encaja porque es incompatible con la gestión segura de memoria del lenguaje y es una causa frecuente de fugas de recursos.

## Rust
   Paradigma:  Funcional, imperativo, basado en propiedad (ownership).
   Patrones comunes:  Type-State, Builder, Newtype.
   Patrones que no encajan:  Patrones de la POO clásica que requieren referencias circulares o compartidas. No encajan porque chocan directamente con el borrow checker y las reglas estrictas de seguridad de memoria en tiempo de compilación.

## Go
   Paradigma:  Imperativo, concurrente.
   Patrones comunes:  CSP (Channels/Goroutines), Opciones Funcionales, Decorator.
   Patrones que no encajan:  Patrones basados en jerarquías de herencia. No encajan porque Go no tiene clases; forzar un polimorfismo clásico va en contra de su filosofía de composición simple mediante interfaces.

## Kotlin
   Paradigma:  Multiparadigma (Orientado a objetos, funcional, corrutinas).
   Patrones comunes:  Delegate, DSL Builder, Singleton nativo (object).
   Patrones que no encajan:  Builders rígidos estilo Java. No encajan porque la sintaxis del lenguaje permite argumentos nombrados y valores por defecto, haciendo que las estructuras extensas de creación sean redundantes.

## Swift
   Paradigma:  Orientado a protocolos, funcional, orientado a objetos.
   Patrones comunes:  MVVM, Coordinator, Delegate.
   Patrones que no encajan:  Singletons globales masivos. No encajan porque acoplan el estado a lo largo de toda la aplicación, rompiendo la arquitectura limpia y modular que fomentan los protocolos.

## Ruby
   Paradigma:  Orientado a objetos puro, reflexivo.
   Patrones comunes:  Metaprogramación (DSL), Decorator, Observer.
   Patrones que no encajan:  Fábricas abstractas rígidas. No encajan porque el tipado dinámico (duck typing) del lenguaje permite crear y modificar objetos al vuelo, haciendo innecesaria tanta infraestructura de creación.

## PHP
   Paradigma:  Multiparadigma (Orientado a objetos, imperativo).
   Patrones comunes:  Front Controller, Inyección de Dependencias, Factory.
   Patrones que no encajan:  Uso de estado global u "Objetos Dios" (God Object). No encajan porque acoplan el código de forma irreversible, lo cual viola los estándares modernos de arquitectura del ecosistema (PSR).

## Scala
   Paradigma:  Multiparadigma (Funcional puro y orientado a objetos estricto).
   Patrones comunes:  Type Classes, Cake Pattern, Monad.
   Patrones que no encajan:  Estructuras procedimentales basadas en estado mutable. No encajan porque destruyen las garantías matemáticas y compositivas que ofrece la parte funcional del lenguaje.

## Haskell
   Paradigma:  Funcional puro, evaluación perezosa.
   Patrones comunes:  Mónadas, Functores, Lens.
   Patrones que no encajan:  Observer o State clásico. No encajan porque dependen inherentemente de mutar el estado en tiempo real, lo cual es imposible sin romper la pureza referencial y la inmutabilidad estricta del lenguaje.

## Elixir
   Paradigma:  Funcional, concurrente (basado en actores).
   Patrones comunes:  OTP (GenServer, Supervisor), Pipe-and-filter.
   Patrones que no encajan:  Singleton de memoria compartida. No encaja porque los procesos en la máquina virtual BEAM están totalmente aislados; intentar compartir memoria destruye la concurrencia segura y tolerante a fallos.

## React
   Paradigma:  Declarativo, basado en componentes, funcional.
   Patrones comunes:  Custom Hooks, Provider/Context, Componentes Compuestos.
   Patrones que no encajan:  Mixins y Componentes de Orden Superior (HOCs). No encajan hoy en día porque generan árboles de componentes excesivamente anidados y colisiones de propiedades, habiendo sido reemplazados por la simplicidad de los Hooks.

## Angular
   Paradigma:  Declarativo, reactivo, basado en componentes.
   Patrones comunes:  Inyección de Dependencias, Observables (RxJS), MVVM.
   Patrones que no encajan:  Modelos que incluyen lógica de red o de negocio pesada. No encajan porque el framework exige una separación estricta donde esa lógica pertenece exclusivamente a la capa de Servicios.

## Django
   Paradigma:  Basado en objetos, orientado a configuración.
   Patrones comunes:  MVT (Model-View-Template), Active Record (vía su ORM).
   Patrones que no encajan:  Implementar un patrón Repository complejo y manual. No encaja porque genera redundancia y fricción operativa al competir directamente con el ORM nativo y altamente optimizado del framework.

## Spring Boot
   Paradigma:  Orientado a objetos, Inversión de Control (IoC).
   Patrones comunes:  Inyección de Dependencias, MVC, Data Access Object (DAO).
   Patrones que no encajan:  Creación manual de servicios con la palabra reservada new. No encaja porque al hacerlo, el objeto queda fuera del contenedor de Spring, perdiendo la gestión de dependencias y el ciclo de vida automático.

## FastAPI
   Paradigma:  Declarativo, asíncrono, fuertemente tipado.
   Patrones comunes:  Inyección de Dependencias, DTO (mediante Pydantic), Middleware.
   Patrones que no encajan:  Bloqueos sincrónicos de I/O dentro de las rutas. No encajan porque pausan el hilo principal de ejecución, destruyendo instantáneamente el rendimiento del bucle de eventos asíncrono (event loop).

## Flutter (Dart)
   Paradigma:  Declarativo, reactivo, basado en widgets.
   Patrones comunes:  BLoC, Provider, State Pattern.
   Patrones que no encajan:  Manipulación imperativa directa de los elementos visuales en pantalla. No encaja porque el motor exige que la interfaz se reconstruya automáticamente a partir de los cambios en el estado general, no mediante modificaciones manuales.