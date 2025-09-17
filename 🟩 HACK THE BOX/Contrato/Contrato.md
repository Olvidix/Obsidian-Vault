![[Pasted image 20250724130127.png]]

#### Pre-compromiso

`Pre-engagement`Se está capacitando al cliente y ajustando el contrato. Todas las pruebas necesarias y sus componentes están estrictamente definidos y registrados contractualmente. En una reunión presencial o una conferencia telefónica, se realizan diversos preparativos, como:

- `Non-Disclosure Agreement`
- `Goals`
- `Scope`
- `Time Estimation`
- `Rules of Engagement`

#### Recopilación de información

`Information gathering`Describe cómo obtenemos información sobre los componentes necesarios de diversas maneras. Buscamos información sobre la empresa objetivo y el software y hardware utilizados para identificar posibles vulnerabilidades de seguridad que podamos aprovechar para establecernos.

#### Evaluación de vulnerabilidad

Una vez que llegamos a la `Vulnerability Assessment`etapa, analizamos los resultados de nuestra `Information Gathering`etapa, buscando vulnerabilidades conocidas en los sistemas, aplicaciones y sus distintas versiones para descubrir posibles vectores de ataque. La evaluación de vulnerabilidades consiste en evaluar las vulnerabilidades potenciales, tanto de forma manual como automatizada. Esto se utiliza para determinar el nivel de amenaza y la susceptibilidad de la infraestructura de red de una empresa a los ciberataques.

#### Explotación

En esta `Exploitation`etapa, utilizamos los resultados para probar nuestros ataques contra los vectores potenciales y ejecutarlos contra los sistemas objetivo para obtener acceso inicial a esos sistemas.

#### Post-explotación

En esta etapa de la prueba de penetración, ya tenemos acceso a la máquina atacada y nos aseguramos de seguir teniendo acceso incluso si se realizan modificaciones y cambios. Durante esta fase, podemos intentar escalar nuestros privilegios para obtener los máximos derechos posibles y buscar datos confidenciales, como credenciales u otros datos que el cliente desea proteger (saqueo). En ocasiones, realizamos una postexplotación para demostrar al cliente el impacto de nuestro acceso. En otras ocasiones, la realizamos como entrada para el proceso de movimiento lateral que se describe a continuación.

#### Movimiento lateral

El movimiento lateral describe el movimiento dentro de la red interna de nuestra empresa objetivo para acceder a hosts adicionales con el mismo nivel de privilegio o uno superior. Suele ser un proceso iterativo combinado con actividades posteriores a la explotación hasta alcanzar nuestro objetivo. Por ejemplo, nos afianzamos en un servidor web, escalamos privilegios y encontramos una contraseña en el registro. Realizamos una enumeración adicional y comprobamos que esta contraseña permite acceder a un servidor de bases de datos como usuario administrador local. Desde aquí, podemos extraer datos confidenciales de la base de datos y encontrar otras credenciales para acceder a mayor profundidad en la red. En esta etapa, normalmente utilizaremos diversas técnicas basadas en la información encontrada en el host o servidor explotado.

#### Prueba de concepto

En esta etapa, documentamos paso a paso los pasos que tomamos para comprometer la red o lograr cierto nivel de acceso. Nuestro objetivo es mostrar cómo logramos encadenar múltiples debilidades para alcanzar nuestro objetivo, de modo que puedan tener una visión clara de cómo encaja cada vulnerabilidad y ayudar a priorizar sus esfuerzos de remediación. Si no documentamos bien nuestros pasos, es difícil que el cliente comprenda lo que logramos hacer y, por lo tanto, se dificultan sus esfuerzos de remediación. De ser posible, podríamos crear uno o más scripts para automatizar los pasos que tomamos y ayudar a nuestro cliente a replicar nuestros hallazgos. Abordamos esto en profundidad en este `Documentation & Reporting`módulo.

#### Después del compromiso

Durante la fase posterior a la intervención, se prepara documentación detallada para que tanto los administradores como la dirección de la empresa cliente comprendan la gravedad de las vulnerabilidades detectadas. En esta etapa, también eliminamos cualquier rastro de nuestras acciones en todos los hosts y servidores. En esta etapa, creamos los entregables para nuestro cliente, realizamos una reunión de análisis del informe y, en ocasiones, realizamos una presentación ejecutiva a los ejecutivos de la empresa objetivo o a su junta directiva. Finalmente, archivamos los datos de las pruebas según nuestras obligaciones contractuales y la política de la empresa. Normalmente, conservamos estos datos durante un periodo determinado o hasta que realizamos una evaluación posterior a la remediación (nueva prueba) para comprobar las correcciones del cliente.
