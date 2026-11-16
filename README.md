# 1. ¿Por qué el frontend no necesita conectarse a PostgreSQL?
Porque sigue una arquitectura por capas. El frontend únicamente se encarga de servir la interfaz visual y redirigir las peticiones /api/ hacia el backend. Es el backend quien valida la lógica de negocio y consulta la base de datos. Mantener el frontend desconectado de PostgreSQL mejora la segurida ya que si alguien vulnera el servidor web expuesto al exterior, no tiene acceso directo a la base de datos.

# 2. ¿Por qué el backend pertenece a dos redes?
Porque actúa como punto de unión entre la capa de presentación y la capa de datos. Pertenece a la red frontend para poder recibir las peticiones que le envía Nginx, y pertenece a la red datos para poder comunicarse con PostgreSQL, manteniendo ambas redes aisladas entre sí.

# 3. ¿Qué servicio necesita publicar un puerto en el anfitrión?
Únicamente el servicio frontend, ya que es el único componente con el que debe interactuar directamente el usuario desde su navegador web. Ni la API ni la base de datos necesitan ser accesibles directamente desde fuera de Docker.

# 4. ¿Por qué publicar PostgreSQL hacia fuera no es la solución adecuada para un fallo de comunicación interno?
Porque abrir el puerto 5432 en el equipo anfitrión expone la base de datos al exterior e incrementa innecesariamente la superficie de ataque. Los contenedores de una misma aplicación deben comunicarse de forma privada a través de las redes internas de Docker, sin atravesar la interfaz de red del host.

# 5. ¿Qué ventaja ofrece Docker Compose respecto a ejecutar los tres contenedores manualmente?
Permite definir toda la infraestructura de forma declarativa en un único archivo el compose.yaml. Esto evita tener que recordar y ejecutar múltiples comandos largos de docker run, docker network y docker volume, garantizando que el despliegue sea reproducible, rápido y libre de errores humanos con un solo comando.

# 6. ¿Por qué hemos dividido la arquitectura en dos redes?
Para aplicar el principio de mínimo privilegio y segmentación de red. Al separar frontend y datos, garantizamos que cada contenedor solo pueda ver y comunicarse con los servicios estrictamente necesarios para su función, impidiendo cualquier conexión directa entre el servidor web público y la base de datos.

# 7. ¿Por qué el backend utiliza db como dirección de PostgreSQL?
Porque Docker Compose proporciona un servidor DNS interno dentro de cada red definida. Este DNS resuelve automáticamente el nombre lógico de cada servicio hacia la dirección IP interna asignada a su contenedor. Usar db evita depender de direcciones IP dinámicas que cambian cada vez que se recrea un contenedor, y evita usar localhost, que dentro de un contenedor apunta al propio contenedor y no a PostgreSQL.

# 8. ¿Qué diferencia existe entre detener un proyecto y eliminar sus volúmenes?
Detener o bajar un proyecto con docker compose stop o docker compose down para o elimina los contenedores y las redes, pero conserva intactos los volúmenes nombrados, de modo que al volver a arrancarlo los datos siguen disponibles. En cambio, eliminar los volúmenes borra de forma permanente el almacenamiento persistente en disco, perdiendo toda la información guardada en la base de datos.

# 9. ¿Qué consecuencias tendría para la empresa perder el volumen de PostgreSQL?
Supondría la pérdida total e irreversible de todo el historial de incidencias registradas, usuarios y estados de resolución almacenados hasta ese momento, ya que los contenedores no guardan estado por sí mismos. Esto provocaría una interrupción grave del servicio de soporte y un impacto directo en la continuidad del negocio si no existieran copias de seguridad externas.