# TP-EVALUATIVO-LPR3-G1-G2

## Laboratorio de Programación —  App Full Stack (FastAPI + SQLite + JavaScript Vanilla)

Este trabajo práctico tiene como objetivo que desarrollen una **aplicación full stack** completa, entendiendo cómo se comunican un **backend** (servidor de API) y un **frontend** (cliente web) a través del protocolo HTTP.

Requisitos:

- **Backend**: API REST en **FastAPI** con base de datos **SQLite**
- **Frontend**: aplicación web en **JavaScript Vanilla** (sin frameworks, sin librerías externas)
- **Documentación**: README con respuestas a preguntas de arquitectura y gráficos explicativos
- **Defensa oral**: presentación individual del proyecto

---

## Consigna

Deberán desarrollar una aplicación sobre un **tema a eleccion**. Algunas ideas:

- Agenda de contactos
- Gestor de tareas / notas
- Inventario de productos
- Biblioteca de libros
- Registro de mascotas de una veterinaria
- Otro

El tema debe tener **al menos una entidad principal** con **mínimo 5 campos** (por ejemplo: `id`, `nombre`, `descripcion`, `precio`, `activo`).

---

## Requisitos del Backend

### Tecnologías

| Componente | Tecnología |
|---|---|
| Framework | FastAPI |
| Base de datos | SQLite (mediante `sqlite3`) |
| Validación de datos | Pydantic |

### Modelos con Pydantic

Deberán definir:

- Un **modelo base** (ej. `ProductoBase`) con las validaciones correspondientes:
  - Campos obligatorios y opcionales
- Un modelo para **crear** recursos (`POST`): sin `id`
- Un modelo para **actualizar** recursos (`PUT` / `PATCH`): todos los campos opcionales
- Un modelo de **respuesta** con `id` incluido

### Managers con Programación Orientada a Objetos

El acceso a la base de datos debe estar **separado de los endpoints** mediante una clase **Manager**:

```python
class ProductoManager:
    def get_all(self, ...) -> list[Producto]: ...
    def get_by_id(self, id: int) -> Producto | None: ...
    def create(self, data: ProductoCreate) -> Producto: ...
    def update(self, id: int, data: ProductoUpdate) -> Producto | None: ...
    def delete(self, id: int) -> bool: ...
```

Requisitos:

- ✅ Los endpoints **no deben ejecutar SQL directamente**
- ✅ El manager debe **encapsular** toda la lógica de acceso a datos
- ✅ Respetar los principios de encapsulamiento y responsabilidad única

### Inyección de dependencias para la conexión a la DB

La conexión a la base de datos debe obtenerse mediante el sistema de **`Depends()`** de FastAPI:

```python
def get_db():
    conn = sqlite3.connect("app.db")
    conn.row_factory = sqlite3.Row
    try:
        yield conn
    finally:
        conn.close()

@app.get("/productos")
def listar_productos(db: sqlite3.Connection = Depends(get_db)):
    ...
```

### Endpoints requeridos (CRUD + filtro)

| Método | Ruta | Descripción |
|---|---|---|
| `GET` | `/productos` | Listar todos. |
| `GET` | `/productos/{id}` | Obtener uno por ID (404 si no existe) |
| `POST` | `/productos` | Crear (validado con Pydantic, 422 si falla) |
| `PUT` | `/productos/{id}` | Actualizar |
| `DELETE` | `/productos/{id}` | Eliminar (404 si no existe) |

> El **filtro es obligatorio**: al menos dos campos filtrables por query string, incluyendo uno **numérico con rango** (ej. `precio_min` / `precio_max`).

### CORS

El backend debe permitir solicitudes desde el frontend (que corre en otro origen/puerto):

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # en producción: origen específico
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Documentación automática

En el README deben indicar cómo acceder a la **documentación interactiva** que genera FastAPI (`/docs`).

---

## Requisitos del Frontend

Aplicación web en **JavaScript Vanilla puro** (`fetch` + DOM).

La interfaz debe permitir:

- ✅ **Listar** los recursos, mostrando el resultado del filtro en pantalla
- ✅ **Buscar/filtrar** mediante un formulario que envíe query params a la API
- ✅ **Crear** recursos mediante un formulario con validación visual de errores (mostrar los errores 422 que devuelve el backend)
- ✅ **Editar** recursos existentes
- ✅ **Eliminar** recursos (con confirmación)
- ✅ **Manejo de errores de red**: mostrar mensajes al usuario si la API no responde
- ✅ **Estados de carga** (loading) en las consultas

**Estructura mínima sugerida:**

```
frontend/
├── index.html
├── css/
│   └── styles.css
└── js/
    ├── api.js        → funciones fetch (capa de comunicación con la API)
    ├── ui.js         → manipulación del DOM
    └── main.js       → orquestación / eventos
```

> **Requisito de arquitectura**: la comunicación con la API debe estar **separada** de la lógica de la interfaz (no mezclar `fetch` dentro de los handlers de eventos).

---

## Preguntas de arquitectura cliente-servidor 

En esta sección del README, cada estudiante debe **responder con sus palabras** y **adjuntar gráficos explicativos** (Excalidraw, otro).

> Las respuestas copiadas de internet o de IA sin desarrollo propio serán motivo de desaprobación de la documentación y proyecto. Lo que buscamos es que **puedan explicar el viaje completo de una petición**.

### Pregunta 1 — El ciclo de vida de una petición
Expliquen **paso a paso** qué sucede cuando el usuario hace clic en "Guardar" en el formulario del frontend, hasta que el dato queda persistido en SQLite y la pantalla se actualiza. Incluyan un **diagrama de secuencia**.

### Pregunta 2 — ¿Quién es el cliente y quién es el servidor?
Identifiquen cliente y servidor en su app. ¿Qué diferencia hay entre el modelo cliente-servidor y una aplicación de escritorio tradicional? Acompañen con un **diagrama de la arquitectura** de su proyecto.

### Pregunta 3 — HTTP como lenguaje común
Expliquen qué son los **métodos HTTP** (GET, POST, PUT, DELETE) y los **códigos de estado** (200, 201, 404, 422). ¿Qué código devuelve su API en cada caso y por qué? Completen una tabla:

| Operación | Método | Ruta | Código de éxito | Código de error |
|---|---|---|---|---|
| Crear | | | | |
| ... | | | | |

### Pregunta 4 — CORS
¿Qué es el SOP (Same-Origin Policy)? ¿Por qué el navegador bloquea las peticiones entre orígenes distintos y qué papel cumple el middleware CORS en su backend? Dibujen un **diagrama del flujo de una petición CORS**.

### Pregunta 5 — Separación de responsabilidades
¿Por qué separar el acceso a la DB (Manager) de los endpoints? ¿Qué beneficios tiene la **inyección de dependencias** para la conexión? ¿Qué pasaría si abrieran una conexión nueva por cada línea de código en vez de inyectarla?

### Pregunta 6 — JSON como formato de intercambio
¿Por qué cliente y servidor se comunican con JSON? ¿Cómo convierte Pydantic los datos que llegan en el body y qué garantiza la validación? Den un ejemplo de un request inválido y la respuesta de error del servidor.

### Pregunta 7 — Statelessness (sin estado)
HTTP es un protocolo *stateless*. ¿Qué significa esto? ¿Dónde queda guardado el "estado" de la aplicación (los datos) en su proyecto? ¿Qué ventaja tiene esto si mañana quisieran tener 3 servidores en lugar de 1?

---

## Estructura de entrega (repositorio GitHub)

```
tp-fullstack-<apellido>/
├── README.md            ← ¡esta documentación completada!
├── backend/
│   ├── main.py          → app FastAPI y endpoints
│   ├── models.py        → modelos Pydantic
│   ├── managers.py      → clases Manager (POO)
│   ├── database.py      → conexión e inicialización de SQLite
│   └── requirements.txt
├── frontend/
│   ├── index.html
│   ├── css/
│   └── js/
└── docs/
    └── (gráficos y diagramas si no están embebidos en el README)
```

El README debe incluir:

- [ ] Descripción del tema elegido
- [ ] Instrucciones para **instalar y ejecutar** el proyecto (paso a paso, probado desde cero)
- [ ] Listado de **endpoints con ejemplos** de request/response
- [ ] Respuestas a las **7 preguntas** de arquitectura
- [ ] **Gráficos/diagramas** obligatorios: diagrama de arquitectura general, diagrama de secuencia de una petición POST, diagrama del flujo CORS
- [ ] Capturas de pantalla del frontend funcionando
- [ ] Criterios de diseño: decisiones tomadas y por qué
- [ ] Descripcion de las tecnologias usadas
- [ ] Nombre, apellido y curso.

---

## Defensa oral

Cada estudiante deberá defender el trabajo individualmente.

---

## Para arrancar

```bash
# Backend
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install fastapi uvicorn
uvicorn main:app --reload     # API en http://localhost:8000/docs

# Frontend
# Opción simple: abrir index.html con Live Server (VS Code)
# o servir la carpeta: python -m http.server 5500
```
