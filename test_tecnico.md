Challenge Infraestructura
Una aplicación python que funciona en kubernetes utiliza 6 pods en un cluster de 8 nodos
La aplicación utiliza una base de datos RDS postgres. El tamaño de las tablas de la base de
datos es de 700GB. Seis de las cuales ocupan el 85% del espacio.
Se puede acceder a los servidores, logs de las aplicaciones y logs de la base de datos.
Se manifiesta una degradación en los tiempos de respuesta de la aplicación.
Recientemente se implementaron nuevas funcionalidades solicitadas por un nuevo
Operador (cliente)
En los últimos días también hubo un incremento del uso de la aplicación (se triplicó la carga
habitual) a raíz del inicio de actividad de dicho Operador.

● Elaborar un camino para obtener un diagnóstico ante el problema planteado.
● Elegir dos de los posibles diagnósticos (uno que involucre a la base de datos y otro que no) 
y definir soluciones/mejoras alternativas