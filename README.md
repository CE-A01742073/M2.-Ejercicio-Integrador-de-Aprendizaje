# MiniScrum Distribuido

## Objetivo

Aplicacion web distribuida para gestionar tareas de un proyecto Scrum.
Permite crear tareas, sugerir automaticamente la cantidad de puntos Scrum
mediante una regresion lineal en Python, guardarlas en MySQL a traves de
un servicio PHP, y cambiar su estado desde una interfaz web servida por
un Gateway en Node.js.

## Arquitectura

```text
Navegador -> Node Gateway (3000) -> PHP API -> MySQL
Navegador -> Node Gateway (3000) -> Python ML API (8000)
```

El unico servicio expuesto publicamente es el Gateway. Los servicios
php-api, python-ml y mysql viven en la red interna de Docker Compose y
no son accesibles desde fuera.

## Servicios

- **gateway**: Node.js + Express. Sirve el frontend (HTML, CSS, JS) y
  redirige las peticiones del navegador hacia PHP o Python.
- **php-api**: PHP 8.2 + Apache. Gestiona el CRUD de tareas usando PDO
  contra MySQL.
- **python-ml**: Python 3.11 + FastAPI + scikit-learn. Entrena una
  regresion lineal en memoria y expone un endpoint de prediccion.
- **mysql**: MySQL 8.0. Almacena las tareas en la tabla `tasks`.

## Endpoints principales

Expuestos por el Gateway (todos bajo `http://<host>:3000`):

- `GET  /api/tasks` - Lista todas las tareas.
- `POST /api/tasks` - Crea una tarea nueva.
- `PUT  /api/tasks/:id/status` - Cambia el estado de una tarea.
- `POST /api/predict` - Devuelve puntos Scrum sugeridos para unas horas.

## Base de datos

Tabla principal: `tasks`

| Campo            | Tipo                                      |
|------------------|-------------------------------------------|
| id               | INT AUTO_INCREMENT PRIMARY KEY            |
| title            | VARCHAR(150) NOT NULL                     |
| description      | TEXT                                      |
| estimated_hours  | DECIMAL(5,2) NOT NULL                     |
| scrum_points     | INT NOT NULL                              |
| status           | ENUM('Pendiente','En proceso','Terminada')|
| created_at       | TIMESTAMP DEFAULT CURRENT_TIMESTAMP       |

## Como ejecutar localmente

```bash
docker compose up --build
```

Luego abrir `http://localhost:3000`.

Para detener y limpiar la base de datos:

```bash
docker compose down -v
```

## URL desplegada

<!-- Pegar aqui la URL publica que asigne Dockploy, por ejemplo:
     https://miniscrum-chris.tu-vps.com -->

TODO: pegar URL de Dockploy.

## Evidencias

<!-- Agregar capturas en esta carpeta o enlazarlas -->

- [ ] Captura de la aplicacion funcionando en Dockploy.
- [ ] Captura de una tarea guardada en la lista.
- [ ] Captura de `docker compose ps` mostrando los 4 contenedores activos.
- [ ] Captura del historial de commits (`git log --oneline`).

## Retrospectiva

**Que salio bien:**

TODO.

**Que fue dificil:**

TODO.

**Que mejoraria:**

TODO.

## Preguntas de reflexion

### 1. Por que el navegador no se conecta directamente a MySQL?

Porque MySQL no habla HTTP: usa un protocolo binario propio y no esta
pensado para recibir conexiones directas desde la web. Ademas, exponer
la base de datos al internet permitiria que cualquiera intente atacarla.
El acceso debe pasar por una capa de aplicacion (PHP) que valida los
datos, ejecuta consultas parametrizadas y controla quien puede hacer que.

### 2. Que ventaja tiene separar el servicio PHP del servicio Python?

Cada servicio puede usar el lenguaje y las librerias que mejor resuelven
su problema: PHP es comodo para CRUD contra MySQL, y Python tiene el
ecosistema de ML (scikit-learn). Se pueden escalar, actualizar o
reemplazar de forma independiente sin afectar al otro. Tambien aisla
fallos: si el servicio de ML se cae, el CRUD sigue operando.

### 3. Que funcion cumple el API Gateway?

Es la unica puerta de entrada publica. Sirve el frontend, unifica las
rutas (todo `/api/*`), oculta a los servicios internos, y traduce las
peticiones del navegador en llamadas a los servicios correctos (PHP o
Python). Tambien centraliza lugares naturales para agregar logging,
autenticacion o rate limiting en el futuro.

### 4. Que pasaria si el servicio Python deja de funcionar?

Las llamadas a `/api/predict` devolverian error 500 desde el Gateway, y
el usuario no podria obtener puntos Scrum sugeridos ni guardar tareas
nuevas (el frontend exige predecir antes de guardar). El resto del
sistema, listar tareas y cambiar estados, seguiria funcionando porque
esas operaciones solo dependen de PHP y MySQL.

### 5. Que diferencia hay entre ejecutar localmente con Docker Compose y desplegar en Dockploy?

Localmente, `docker compose up` levanta los contenedores en tu maquina
y la app solo es accesible en `localhost:3000`. En Dockploy, los
contenedores corren en un VPS compartido, Dockploy clona el repo desde
Git en cada deploy, asigna un dominio/URL publica, gestiona certificados
y mantiene la aplicacion corriendo aunque cierres tu computadora.

### 6. Que parte del sistema representa la capa de datos?

MySQL. Es donde se persisten las tareas de forma permanente en la tabla
`tasks`.

### 7. Que parte representa la capa de presentacion?

El frontend dentro de `gateway/public/` (HTML, CSS y JS) que se ejecuta
en el navegador del usuario, mas el Gateway que lo sirve.

### 8. Que parte representa la logica de negocio?

Esta repartida entre el servicio PHP (reglas de validacion de tareas,
estados permitidos, escrituras en la base) y el servicio Python (la
regresion lineal que traduce horas a puntos Scrum).
