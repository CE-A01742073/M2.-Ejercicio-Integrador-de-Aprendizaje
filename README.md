
### 1. Por que el navegador no se conecta directamente a MySQL?

Porque MySQL no habla HTTP usa un protocolo binario propio y no esta
pensado para recibir conexiones directas desde la web.

### 2. Que ventaja tiene separar el servicio PHP del servicio Python?

Cada servicio puede usar el lenguaje y las librerias que mejor resuelven
sus problemas. Se pueden actualizar de forma independiente sin afectar al otro. Tambien aisla
fallos: si el servicio de ML se cae, el CRUD sigue operando.

### 3. Que funcion cumple el API Gateway?

Es la unica puerta de entrada publica. Sirve el frontend y oculta a los servicios internos

### 4. Que pasaria si el servicio Python deja de funcionar?

Las llamadas devolverian error 500 desde el Gateway, y
el usuario no podria obtener puntos Scrum sugeridos ni guardar tareas
nuevas

### 5. Que diferencia hay entre ejecutar localmente con Docker Compose y desplegar en Dockploy?

Localmente levantas los contenedores en tu maquina
y la app solo es accesible en localhost:3000. En Dockploy, los
contenedores corren en un VPS compartido

### 6. Que parte del sistema representa la capa de datos?

MySQL. Es donde se persisten las tareas de forma permanente en la tabla
tasks

### 7. Que parte representa la capa de presentacion?

El frontend dentro de gateway/public/

### 8. Que parte representa la logica de negocio?

Esta repartida entre el servicio PHP y el servicio Python
