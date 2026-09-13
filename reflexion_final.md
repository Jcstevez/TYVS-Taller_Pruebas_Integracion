\# Reflexión Final — Taller de Pruebas de Integración y Sistema



\## Integrantes

\- Juan Camilo Estevez — Pruebas de Integración (H2), Pruebas con Mocks, Matriz de pruebas, Gestión de defectos

\- Sofy Alejandra Prada Murillo — Testcontainers, Contract Testing (Pact), Pruebas de Sistema (HTTP), Cobertura JaCoCo, CI/CD



\## Qué aprendimos sobre cada tipo de prueba



\*\*Pruebas unitarias con mocks\*\* nos permitieron validar toda la lógica de

negocio de `Registry` (validación de edad, estado vital, duplicados, nulos)

de forma rápida y aislada, sin depender de infraestructura. Fueron las más

rápidas de escribir y correr, pero por diseño no garantizan nada sobre cómo

se comporta el sistema contra una base de datos real.



\*\*Pruebas de integración con H2\*\* cerraron esa brecha parcialmente: nos

dieron confianza de que `Registry` y la capa de persistencia trabajan bien

juntas, incluyendo el caso central de que la base de datos (no un mock)

detecta los duplicados de verdad. Sin embargo, H2 no es PostgreSQL, y esa

diferencia de dialecto SQL, tipos y comportamiento transaccional es

exactamente el problema que resuelven las \*\*pruebas con Testcontainers\*\*.



\*\*Pruebas de sistema (HTTP)\*\* verificaron el comportamiento del proyecto

como caja negra, incluyendo un caso que ninguna de las otras técnicas cubre:

el manejo de errores del cliente (un `gender` inválido) a través del

`RestControllerAdvice`, que traduce lo que sería un error 500 en un 400

correcto.



\*\*Contract Testing con Pact\*\* nos enseñó una forma distinta de pensar la

integración entre servicios: en vez de levantar el consumidor y el

proveedor juntos, cada uno verifica por separado que cumple su parte del

contrato. Esto es valioso en un escenario real donde `CertificadoService` y

la Registraduría podrían desplegarse y evolucionar de forma independiente.



\## El hallazgo más importante: cuando el entorno es el problema



La parte más reveladora del taller no fue código, sino infraestructura.

Pasamos gran parte de dos sesiones de trabajo intentando que Testcontainers

se conectara a Docker en Windows, probando named pipes, TCP, un motor

Docker nativo instalado dentro de WSL, distintas versiones de la librería,

y la variable de entorno diseñada específicamente para este problema

(`DOCKER\_API\_VERSION`) — en tres formas distintas y en dos máquinas

diferentes, siempre con el mismo resultado: un error de versión de API

(`client version 1.32 is too old`) que ni la propia librería lograba

resolver automáticamente.



La solución no fue seguir ajustando el entorno local, sino reconocer que el

problema \*era\* el entorno local, y moverlo a un lugar donde esa variable no

existiera: un pipeline de \*\*GitHub Actions\*\* corriendo sobre un runner

Ubuntu limpio. Ahí, sin ningún ajuste adicional, las 5 pruebas de

Testcontainers pasaron a la primera.



Este fue, para nosotros, el aprendizaje más transferible del taller:

verificar la integración contra infraestructura real (bases de datos,

contenedores) depende de tener un entorno de ejecución consistente. Un

entorno de desarrollo local acumula configuraciones y particularidades

propias de cada máquina que un pipeline de CI, reconstruido desde cero en

cada corrida, elimina por completo. Por eso los equipos de desarrollo reales

no dependen de que Testcontainers funcione en la laptop de cada persona:

dependen de que funcione en CI.



\## Sobre la cobertura obtenida



Con las seis técnicas de prueba funcionando (unitarias, mocks, integración

H2, integración Testcontainers, sistema HTTP, y contract testing en ambos

lados) alcanzamos un 91% de cobertura de instrucciones y 70% de cobertura

de branches, combinando los reportes de Surefire y Failsafe. El punto más

débil quedó en el manejo de excepciones SQL poco comunes dentro de la capa

de persistencia, que representaría el siguiente paso lógico de cobertura

si se continuara este proyecto.



\## Conclusión



Lo más valioso del taller no fue aprender la sintaxis de cada framework de

pruebas (JUnit, Mockito, Testcontainers, Pact), sino entender \*cuándo\* usar

cada una y qué riesgo específico cierra cada una que las demás no cubren.

Un sistema bien probado no depende de un solo tipo de prueba, sino de la

combinación deliberada de varias, cada una respondiendo una pregunta

distinta: ¿funciona la lógica sola? ¿funciona con la base de datos real?

¿funciona a través de HTTP? ¿sigue cumpliendo su contrato con quien lo

consume? Y, algo que no esperábamos aprender: ¿el problema está en el

código, o está en el entorno donde se ejecuta?

