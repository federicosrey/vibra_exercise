## Challenge Infraestructura
Una aplicación python que funciona en kubernetes utiliza 6 pods en un cluster de 8 nodos. La aplicación utiliza una base de datos RDS postgres. El tamaño de las tablas de la base de datos es de 700GB. Seis de las cuales ocupan el 85% del espacio. Se puede acceder a los servidores, logs de las aplicaciones y logs de la base de datos. Se manifiesta una degradación en los tiempos de respuesta de la aplicación. Recientemente se implementaron nuevas funcionalidades solicitadas por un nuevo Operador (cliente). En los últimos días también hubo un incremento del uso de la aplicación (se triplicó la carga habitual) a raíz del inicio de actividad de dicho Operador.

- Elaborar un camino para obtener un diagnóstico ante el problema planteado.
- Elegir dos de los posibles diagnósticos (uno que involucre a la base de datos y otro que no) y definir soluciones/mejoras alternativas

## Diagnostico

### Cluster Kubernetes

Como primer paso y teniendo en cuenta que la carga de trabajo fue multiplicada por 3, se debe revisar el comportamiento del cluster de la siguiente manera:

- Revisar metricas de carga de hardware tales como CPU y RAM de los pods en particular y analizar como crecio la carga, que tanto tiempo se sostuvo y en que horario surgio eso, las metricas historicas pueden ser obtenidas mediante software de monitoreo como, prometheus + grafana, datadog, etc en el caso que este este instalado, de lo contratio analizar mediante `kubectl top pod` que nos dara el consumo en ese momento.
- Tambien es recomendable revisar los eventos para revisar si los pods fueron reiniciandose o algun comportamiento no esperado a traves de `kubectl get events`.
- Los nodos tambien se revisan sus metricas de utilizacion de CPU, RAM y es deseable que tambien se analizen los tiempos de disco tales como latencias de escritura y lectura y la cantidad de datos transferidos ultimamente, esto directamente lo podemos observar desde la misma instancia o para buscar mas detalle ir a AWS CloudWatch.
- En este punto tenemos varios caminos por seguir
  - Si los pods no pudieron hacer frente a la demanda por el hardware y replicas asignadas, y los nodos no fueron afectados por esa carga, se deberia revisar tanto los HPA de ese deployment como los recursos asignados para cada replica de ese servicio
  - Si los pods no pueden hacerle frente a esa demanda pero los nodos tambien estan saturados, lo ideal es revisar el tamaño de las instancias y poder repartir mejor la carga de trabajo como se mencino en el punto anterior ya que 2 de esos 8 nodos no recibieron carga de la aplicacion
  - Si no se presenta una sobrecarga de trabajo tanto en los pods como en los nodos worker, se debera dar paso al diagnostico de la base de datos (el cual se tocara en puntos siguientes)
  - Si en todos estos pasos no fue posible conseguir pruebas suficientes para diagnosticar el problema, lo recomendado es revisar dentro de los nodos worker logs de kubelet, en el caso de tener estos sobre linux un ejemplo podria ser `journalctl -u kubectl`

A fines de clarificar un poco el texto escrito se añade un flujo de desicion basico de como se podria proceder ante una situacion similar

![image](https://github.com/federicosrey/vibra_exercise/assets/63884877/355b14a3-ca51-446b-9c3c-1873cc9e129f?style=centerme)

### Base de datos

Es necesario revisar la base de datos, sea que esta se haya saturado o no, se pueden revisar los siguientes puntos:

- Antes que nada revisar metricas de comportamiento de hardware clasicas como CPU y RAM, en el caso de las bases de datos RDS es importante revisar tambien los tiempos de respuesta de los discos y tambien la cantidad de conexiones maximas, mas que nada por que si no son modificadas en el parameter group, el numero viene dado en funcion al tipo de instancia desplegado. Este es un numero general y no se adapta en todos los casoa al consumo real de hardware por conexion establecida.
- Es un dato importante a tener en cuenta que el 85% de todos los datos estan solo en 6 tablas seria interesante analizar las consultas que existen sobre esas tablas, si es factible optimizarlsa, revisar la normalizacion de estas mismas para evitar la redundancia de datos. Si todos los datos son consultados habitualmente y si no es necesario separar esa informacion en otro motor de base de datos sea OLAP, OLTP o alguna solucion de datos no estructurados segun sea la necesidad. Si las metricas nos arrojan mucha lectura exclusiva, quiza seria bueno pensar en una replica o un cluster con n nodos.
- Revisar las conexiones ya que si estas se agotaron las conexiones son rechazadas por la base de datos y haciendo que esta queda totalmente inutilizable, puede llegar a tener problemas de inconsistencia de datos en un futuro.
- Revisar el performance insights en busca de consultas que tomaron mucho tiempo y por que
