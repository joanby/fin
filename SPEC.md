# FinAlly — Estación de Trabajo de Trading con IA (AI Trading Workstation)

## Especificación del Proyecto

## 1. Visión

FinAlly (Finance Ally / Aliado Financiero) es una estación de trabajo de trading visualmente impresionante y potenciada por IA que transmite datos de mercado en vivo, permite a los usuarios operar con una cartera simulada e integra un asistente de chat basado en LLM que puede analizar posiciones y ejecutar operaciones en nombre del usuario. Tiene el aspecto y la sensación de una terminal de Bloomberg moderna con un copiloto de IA.

Este es el proyecto final (capstone) para un curso de programación con IA de base agentica. Está construido completamente por Agentes de Programación, demostrando cómo agentes de IA orquestados pueden producir una aplicación full-stack con calidad de producción. Los agentes interactúan a través de archivos en `planning/`.

## 2. Experiencia de Usuario

### Primer Lanzamiento

El usuario ejecuta un único comando de Docker (o un script de inicio proporcionado). Se abre un navegador en `http://localhost:8000`. Sin inicio de sesión, sin registro. Inmediatamente ven:

- Una lista de seguimiento (watchlist) de 10 tickers por defecto con precios actualizados en vivo en una cuadrícula.
- $10,000 en efectivo virtual.
- Una estética de terminal de trading oscura y rica en datos.
- Un panel de chat de IA listo para ayudar.

### Qué Puede Hacer el Usuario

- **Ver la transmisión de precios en tiempo real** — los precios parpadean en verde (subida) o rojo (bajada) con sutiles animaciones CSS que se desvanecen.
- **Ver minigráficos sparkline** — la evolución del precio al lado de cada ticker en la lista de seguimiento, acumulada en el frontend a partir de la transmisión SSE desde la carga de la página (los sparklines se van dibujando progresivamente).
- **Hacer clic en un ticker** para ver un gráfico más grande y detallado en el área principal de gráficos.
- **Comprar y vender acciones** — solo órdenes de mercado, ejecución instantánea al precio actual, sin comisiones, sin diálogos de confirmación.
- **Monitorear su cartera** — un mapa de calor (treemap) que muestra las posiciones dimensionadas por peso y coloreadas por pérdidas y ganancias (P&L), además de un gráfico de P&L que rastrea el valor total de la cartera a lo largo del tiempo.
- **Ver una tabla de posiciones** — ticker, cantidad, costo promedio, precio actual, P&L no realizado, % de cambio.
- **Chatear con el asistente de IA** — preguntar sobre su cartera, obtener análisis y hacer que la IA ejecute operaciones y gestione la lista de seguimiento mediante lenguaje natural.
- **Gestionar la lista de seguimiento** — añadir/eliminar tickers manualmente o a través del chat de IA.

### Visual Design

- **Tema oscuro**: fondos alrededor de `#0d1117` o `#1a1a2e`, bordes grises apagados, sin negro puro.
- **Animaciones de parpadeo de precios**: un breve resaltado de fondo verde/rojo al cambiar el precio, desvaneciéndose en aproximadamente 500 ms a través de transiciones CSS.
- **Indicador de estado de conexión**: un pequeño punto de color (verde = conectado, amarillo = reconectando, red = desconectado) visible en el encabezado.
- **Diseño profesional y denso en datos**: inspirado en las terminales de Bloomberg/trading — cada píxel se gana su lugar.
- **Responsivo pero enfocado en escritorio**: optimizado para pantallas anchas, funcional en tabletas.

### Paleta de Colores
- Amarillo de Acento: `#ecad0a`
- Azul Primario: `#209dd7`
- Morado Secundario: `#753991` (botones de enviar)

## 3. Descripción General de la Arquitectura

### Contenedor Único, Puerto Único

```
┌─────────────────────────────────────────────────┐
│  Contenedor Docker (puerto 8000)                │
│                                                 │
│  FastAPI (Python/uv)                            │
│  ├── /api/*          Endpoints REST             │
│  ├── /api/stream/*   Transmisión SSE            │
│  └── /*              Servicio de archivos       │
│                      estáticos (export Next.js) │
│                                                 │
│  Base de datos SQLite (montada por volumen)     │
│  Tarea en segundo plano: simulación/consulta    │
│  de datos de mercado                            │
└─────────────────────────────────────────────────┘
```

- **Frontend**: Next.js con TypeScript, construido como una exportación estática (`output: 'export'`), servido por FastAPI como archivos estáticos.
- **Backend**: FastAPI (Python), gestionado como un proyecto `uv`.
- **Base de datos**: SQLite, un único archivo en `db/finally.db`, montado mediante volumen para persistencia.
- **Datos en tiempo real**: Server-Sent Events (SSE) — más simple que WebSockets, empuje unidireccional servidor→cliente, funciona en todas partes.
- **Integración de IA**: LiteLLM → OpenRouter (Cerebras para inferencia rápida), con salidas estructuradas (Structured Outputs) para la ejecución de operaciones.
- **Datos de mercado**: Dirigidos por variables de entorno — simulador por defecto, datos reales a través de Massive API si se proporciona la clave.

### Por Qué Estas Decisiones

| Decisión | Justificación |
|---|---|
| SSE sobre WebSockets | El empuje unidireccional es todo lo que necesitamos; más simple, sin complejidad bidireccional, soporte universal del navegador. |
| Exportación estática de Next.js | Origen único, sin problemas de CORS, un puerto, un contenedor, despliegue sencillo. |
| SQLite sobre Postgres | Sin autenticación = sin multiusuario = sin necesidad de un servidor de base de datos; autocontenido, configuración cero. |
| Contenedor Docker único | Los estudiantes ejecutan un solo comando; sin docker-compose para producción, sin orquestación de servicios. |
| uv para Python | Gestión de proyectos de Python rápida y moderna; archivo de bloqueo (lockfile) reproducible; lo que los estudiantes deben aprender. |
| Solo órdenes de mercado | Elimina el libro de órdenes, la lógica de órdenes límite y ejecuciones parciales — matemáticas de cartera drásticamente más simples. |

---

## 4. Estructura de Directorios

```
finally/
├── frontend/                 # Proyecto Next.js TypeScript (exportación estática)
├── backend/                  # Proyecto FastAPI uv (Python)
│   └── db/                   # Schema definitions, seed data, migration logic
├── planning/                 # Project-wide documentation for agents
│   ├── PLAN.md               # Este documento
│   └── ...                   # Additional agent reference docs
├── scripts/
│   ├── start_mac.sh          # Iniciar contenedor Docker (macOS/Linux)
│   ├── stop_mac.sh           # Detener contenedor Docker (macOS/Linux)
│   ├── start_windows.ps1     # Iniciar contenedor Docker (Windows PowerShell)
│   └── stop_windows.ps1      # Detener contenedor Docker (Windows PowerShell)
├── test/                     # Pruebas E2E de Playwright + docker-compose.test.yml
├── db/                       # Destino del montaje de volumen (el archivo SQLite reside aquí en tiempo de ejecución)
│   └── .gitkeep              # El directorio existe en el repositorio; finally.db está en el .gitignore
├── Dockerfile                # Construcción multi-etapa (Node → Python)
├── docker-compose.yml        # Envolvedor (wrapper) de conveniencia opcional
├── .env                      # Variables de entorno (ignorado por git, se incluye .env.example en los commits)
└── .gitignore
```

### Límites Clave

- **`frontend/`** es un proyecto Next.js autocontenido. No sabe nada de Python. Se comunica con el backend a través de endpoints `/api/*` y endpoints SSE `/api/stream/*`. La estructura interna depende del agente Ingeniero de Frontend.
- **`backend/`** es un proyecto uv autocontenido con su propio `pyproject.toml`. Es dueño de toda la lógica del servidor, incluyendo la inicialización de la base de datos, el esquema, los datos semilla, las rutas de la API, la transmisión SSE, los datos de mercado y la integración del LLM. La estructura interna depende de los agentes de Backend/Datos de Mercado.
- **`backend/db/`** contiene definiciones SQL del esquema y lógica de inicialización (seed). El backend inicializa la base de datos de forma perezosa (lazy) en la primera solicitud — creando tablas y sembrando datos por defecto si el archivo SQLite no existe o está vacío.
- **`db/`** en el nivel superior es el punto de montaje del volumen en tiempo de ejecución. El archivo SQLite (`db/finally.db`) es creado aquí por el backend y persiste a través de los reinicios del contenedor mediante el volumen de Docker.
- **`planning/`** contiene documentación de todo el proyecto, incluyendo este plan. Todos los agentes hacen referencia a los archivos aquí como el contrato compartido.
- **`test/`** contiene pruebas E2E de Playwright e infraestructura de soporte (por ejemplo, `docker-compose.test.yml`). Las pruebas unitarias residen dentro de `frontend/` y `backend/` respectivamente, siguiendo las convenciones de cada framework.
- **`scripts/`** contiene scripts de inicio/parada que envuelven los comandos de Docker.

---

## 5. Variables de Entorno

```bash
# Requerido: Clave API de OpenRouter para la funcionalidad de chat de IA
OPENROUTER_API_KEY=tu-clave-api-de-openrouter-aqui

# Opcional: Clave API de Massive (Polygon.io) para datos de mercado reales
# Si no se configura, se utiliza el simulador de mercado integrado (recomendado para la mayoría de los usuarios)
MASSIVE_API_KEY=

# Opcional: Configurar en "true" para respuestas simuladas deterministas del LLM (pruebas)
LLM_MOCK=false
```

### Comportamiento

- Si `MASSIVE_API_KEY` está configurado y no está vacío → el backend utiliza la API REST de Massive para los datos de mercado.
- Si `MASSIVE_API_KEY` está ausente o vacío → el backend utiliza el simulador de mercado integrado.
- Si `LLM_MOCK=true` → el backend devuelve respuestas simuladas deterministas del LLM (para pruebas E2E).
- El backend lee `.env` desde la raíz del proyecto (montado en el contenedor o leído a través de docker con `--env-file`).

---

## 6. Datos de Mercado

### Dos Implementaciones, Una Interfaz

Tanto el simulador como el cliente de Massive implementan la misma interfaz abstracta. El backend selecciona cuál usar basándose en la variable de entorno. Todo el código aguas abajo (transmisión SSE, caché de precios, frontend) es agnóstico a la fuente.

### Simulador (Por Defecto)

- Genera precios utilizando el movimiento browniano geométrico (GBM) con deriva (drift) y volatilidad configurables por ticker.
- Se actualiza en intervalos de aproximadamente 500 ms.
- Movimientos correlacionados entre tickers (por ejemplo, las acciones tecnológicas se mueven juntas).
- "Eventos" aleatorios ocasionales — movimientos repentinos del 2-5% en un ticker para añadir drama.
- Comienza desde precios semilla realistas (por ejemplo, AAPL ~$190, GOOGL ~$175, etc.).
- Se ejecuta como una tarea en segundo plano dentro del proceso — sin dependencias externas.

### Massive API (Opcional)

- Consulta periódica (polling) de la API REST (no WebSocket) — más simple, funciona en todos los planes.
- Realiza consultas para la unión de todos los tickers vigilados en un intervalo configurable.
- Plan gratuito (5 llamadas/min): consulta cada 15 segundos.
- Planes de pago: consulta cada 2 a 15 segundos dependiendo del plan.
- Procesa la respuesta REST en el mismo formato que el simulador.

### Caché de Precios Compartida

- Una única tarea en segundo plano (simulador o consultor de Massive) escribe en una caché de precios en memoria.
- La caché almacena el último precio, el precio anterior y la marca de tiempo para cada ticker.
- Las transmisiones SSE leen de esta caché y envían las actualizaciones a los clientes conectados.
- Esta arquitectura soporta futuros escenarios multiusuario sin cambios en la capa de datos.

### Transmisión SSE (Server-Sent Events)

- Endpoint: `GET /api/stream/prices`
- Conexión SSE de larga duración; el cliente utiliza la API nativa de `EventSource`.
- El servidor envía actualizaciones de precios para todos los tickers conocidos por el sistema a un ritmo regular (~500ms) — en el modelo de usuario único, esto equivale a la lista de seguimiento del usuario.
- Cada evento SSE contiene el ticker, el precio, el precio anterior, la marca de tiempo y la dirección del cambio.
- El cliente maneja la reconexión automáticamente (EventSource tiene reintento integrado).

---

## 7. Base de Datos

### SQLite con Inicialización Perezosa (Lazy)

El backend comprueba la base de datos SQLite al iniciar (o en la primera solicitud). Si el archivo no existe o faltan tablas, crea el esquema y siembra los datos por defecto. Esto significa:

- Sin paso de migración separado.
- Sin configuración manual de la base de datos.
- Los nuevos volúmenes de Docker comienzan con una base de datos limpia y con datos semilla automáticamente.

### Esquema

Todas las tablas incluyen una columna `user_id` que por defecto es `"default"`. Esto está predeterminado por ahora (usuario único) pero permite un futuro soporte multiusuario sin migraciones de esquema.

**users_profile** — Estado del usuario (saldo de efectivo)
- `id` TEXT PRIMARY KEY (por defecto: `"default"`)
- `cash_balance` REAL (por defecto: `10000.0`)
- `created_at` TEXT (marca de tiempo ISO)

**watchlist** — Tickers que el usuario está siguiendo
- `id` TEXT PRIMARY KEY (UUID)
- `user_id` TEXT (por defecto: `"default"`)
- `ticker` TEXT
- `added_at` TEXT (marca de tiempo ISO)
- Restricción UNIQUE en `(user_id, ticker)`

**positions** — Tenencias actuales (una fila por ticker por usuario)
- `id` TEXT PRIMARY KEY (UUID)
- `user_id` TEXT (por defecto: `"default"`)
- `ticker` TEXT
- `quantity` REAL (soporta acciones fraccionarias)
- `avg_cost` REAL
- `updated_at` TEXT (marca de tiempo ISO)
- Restricción UNIQUE en `(user_id, ticker)`

**trades** — Historial de operaciones (registro de solo adición/append-only)
- `id` TEXT PRIMARY KEY (UUID)
- `user_id` TEXT (por defecto: `"default"`)
- `ticker` TEXT
- `side` TEXT (`"buy"` o `"sell"`)
- `quantity` REAL (soporta acciones fraccionarias)
- `price` REAL
- `executed_at` TEXT (marca de tiempo ISO)

**portfolio_snapshots** — Valor de la cartera a lo largo del tiempo (para el gráfico de P&L). Registrado cada 30 segundos por una tarea en segundo plano, e inmediatamente después de la ejecución de cada operación.
- `id` TEXT PRIMARY KEY (UUID)
- `user_id` TEXT (por defecto: `"default"`)
- `total_value` REAL
- `recorded_at` TEXT (marca de tiempo ISO)

**chat_messages** — Historial de conversación con el LLM
- `id` TEXT PRIMARY KEY (UUID)
- `user_id` TEXT (por defecto: `"default"`)
- `role` TEXT (`"user"` o `"assistant"`)
- `content` TEXT
- `actions` TEXT (JSON — operaciones ejecutadas, cambios en la lista de seguimiento realizados; nulo para mensajes de usuario)
- `created_at` TEXT (marca de tiempo ISO)

### Datos Semilla por Defecto

- Un perfil de usuario: `id="default"`, `cash_balance=10000.0`
- Diez entradas en la lista de seguimiento: AAPL, GOOGL, MSFT, AMZN, TSLA, NVDA, META, JPM, V, NFLX

---

## 8. Endpoints de la API

### Datos de Mercado
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/stream/prices` | Transmisión SSE de actualizaciones de precios en vivo |

### Cartera (Portfolio)
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/portfolio` | Posiciones actuales, saldo de efectivo, valor total, P&L no realizado |
| POST | `/api/portfolio/trade` | Ejecutar una operación: `{ticker, quantity, side}` |
| GET | `/api/portfolio/history` | Instantáneas del valor de la cartera a lo largo del tiempo (para el gráfico de P&L) |

### Lista de Seguimiento (Watchlist)
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/watchlist` | Tickers actuales de la lista de seguimiento con los últimos precios |
| POST | `/api/watchlist` | Añadir un ticker: `{ticker}` |
| DELETE | `/api/watchlist/{ticker}` | Eliminar un ticker |

### Chat
| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/api/chat` | Enviar un mensaje, recibir respuesta JSON completa (mensaje + acciones ejecutadas) |

### Sistema
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/health` | Verificación de estado (para Docker/despliegue) |

---

## 9. Integración del LLM

Al escribir código para hacer llamadas a los LLMs, utiliza la habilidad de inferencia de Cerebras (cerebras-inference skill) para usar LiteLLM a través de OpenRouter apuntando al modelo `openrouter/openai/gpt-oss-120b` con Cerebras como el proveedor de inferencia. Se deben utilizar Salidas Estructuradas (Structured Outputs) para interpretar los resultados.

Hay una `OPENROUTER_API_KEY` en el archivo `.env` en la raíz del proyecto.

### Cómo Funciona

Cuando el usuario envía un mensaje de chat, el backend:

1. Carga el contexto de la cartera actual del usuario (efectivo, posiciones con P&L, lista de seguimiento con precios en vivo, valor total de la cartera).
2. Carga el historial de conversación reciente de la tabla `chat_messages`.
3. Construye un prompt con un mensaje del sistema, el contexto de la cartera, el historial de conversación y el nuevo mensaje del usuario.
4. Llama al LLM a través de LiteLLM → OpenRouter, solicitando una salida estructurada, utilizando la habilidad de inferencia de Cerebras.
5. Procesa la respuesta JSON estructurada completa.
6. Ejecuta automáticamente cualquier operación o cambio en la lista de seguimiento especificado en la respuesta.
7. Almacena el mensaje y las acciones ejecutadas en `chat_messages`.
8. Devuelve la respuesta JSON completa al frontend (sin transmisión token por token — la inferencia de Cerebras es lo suficientemente rápida como para que un indicador de carga sea suficiente).

### Esquema de Salida Estructurada

Se le instruye al LLM que responda con un JSON que coincida con este esquema:

```json
{
  "message": "Tu respuesta conversacional para el usuario",
  "trades": [
    {"ticker": "AAPL", "side": "buy", "quantity": 10}
  ],
  "watchlist_changes": [
    {"ticker": "PYPL", "action": "add"}
  ]
}
```

- `message` (requerido): El texto conversacional mostrado al usuario.
- `trades` (opcional): Matriz de operaciones a ejecutar automáticamente. Cada operación pasa por la misma validación que las operaciones manuales (efectivo suficiente para compras, acciones suficientes para ventas).
- `watchlist_changes` (opcional): Matriz de modificaciones de la lista de seguimiento.

### Auto-Ejecución

Las operaciones especificadas por el LLM se ejecutan automáticamente — sin diálogo de confirmación. Esta es una decisión de diseño deliberada:
- Es un entorno simulado con dinero ficticio, por lo que el riesgo es cero.
- Crea una experiencia de demostración fluida e impresionante.
- Demuestra las capacidades de la IA agentica — el tema central del curso.

Si una operación falla la validación (por ejemplo, efectivo insuficiente), el error se incluye en la respuesta del chat para que el LLM pueda informar al usuario.

### Guía del Prompt del Sistema (System Prompt)

El LLM debe ser guiado como "FinAlly, un asistente de trading con IA" con instrucciones para:
- Analizar la composición de la cartera, la concentración del riesgo y el P&L.
- Sugerir operaciones con su respectivo razonamiento.
- Ejecutar operaciones cuando el usuario lo solicite o esté de acuerdo.
- Gestionar la lista de seguimiento de forma proactiva.
- Ser conciso y basarse en datos en las respuestas.
- Responder siempre con un JSON estructurado válido.

### Modo Simulador del LLM (LLM Mock Mode)

Cuando `LLM_MOCK=true`, el backend devuelve respuestas simuladas deterministas en lugar de llamar a OpenRouter. Esto permite:
- Pruebas E2E rápidas, gratuitas y reproducibles.
- Desarrollo sin una clave de API.
- Pipelines de CI/CD.

---

## 10. Diseño del Frontend

### Distribución (Layout)

El frontend es una aplicación de una sola página con una distribución densa inspirada en una terminal. La arquitectura específica de componentes y el sistema de distribución dependen del Ingeniero de Frontend, pero la interfaz de usuario debe incluir estos elementos:

- **Panel de la lista de seguimiento** — cuadrícula/tabla de tickers bajo seguimiento con: símbolo del ticker, precio actual (parpadeando en verde/rojo al cambiar), % de cambio diario y un minigráfico sparkline (acumulado desde la transmisión SSE desde la carga de la página).
- **Área principal del gráfico** — gráfico más grande para el ticker seleccionado actualmente, con al menos el precio a lo largo del tiempo. Al hacer clic en un ticker de la lista de seguimiento se selecciona aquí.
- **Mapa de calor de la cartera** — visualización de mapa de árbol (treemap) donde cada rectángulo es una posición, dimensionada por su peso en la cartera, coloreada por P&L (verde = ganancia, rojo = pérdida).
- **Gráfico de P&L** — gráfico de líneas que muestra el valor total de la cartera a lo largo del tiempo, utilizando datos de `portfolio_snapshots`.
- **Tabla de posiciones** — vista tabular de todas las posiciones: ticker, cantidad, costo promedio, precio actual, P&L no realizado, % de cambio.
- **Barra de operaciones** — área de entrada simple: campo de ticker, campo de cantidad, botón de compra, botón de venta. Órdenes de mercado, ejecución instantánea.
- **Panel de chat de IA** — barra lateral acoplada/colapsable. Entrada de mensajes, historial de conversación con desplazamiento, indicador de carga mientras se espera la respuesta del LLM. Las ejecuciones de operaciones y cambios de lista de seguimiento se muestran en línea como confirmaciones.
- **Encabezado** — valor total de la cartera (actualizado en vivo), indicador de estado de conexión, saldo de efectivo.

### Notas Técnicas

- Usar `EventSource` para la conexión SSE a `/api/stream/prices`.
- Se prefiere una biblioteca de gráficos basada en Canvas (como Lightweight Charts o Recharts) por rendimiento.
- Efecto de parpadeo de precios: al recibir un nuevo precio, aplicar brevemente una clase CSS con transición de color de fondo y luego eliminarla.
- Todas las llamadas a la API van al mismo origen (`/api/*`) — no se necesita configuración de CORS.
- Tailwind CSS para los estilos con un tema oscuro personalizado.

---

## 11. Docker y Despliegue

### Dockerfile Multietapa

```
Etapa 1: Node 20 slim
  - Copiar frontend/
  - npm install && npm run build (produce la exportación estática)

Etapa 2: Python 3.12 slim
  - Instalar uv
  - Copiar backend/
  - uv sync (instalar dependencias de Python desde el archivo de bloqueo)
  - Copiar el resultado de la construcción del frontend en un directorio static/
  - Exponer el puerto 8000
  - CMD: uvicorn sirviendo la aplicación FastAPI
```

FastAPI sirve los archivos estáticos del frontend y todas las rutas de la API en el puerto 8000.

### Volumen de Docker

La base de datos SQLite persiste a través de un volumen nombrado de Docker:

```bash
docker run -v finally-data:/app/db -p 8000:8000 --env-file .env finally
```

El directorio `db/` en la raíz del proyecto se mapea a `/app/db` en el contenedor. El backend escribe `finally.db` en esta ruta.

### Scripts de Inicio/Parada

**`scripts/start_mac.sh`** (macOS/Linux):
- Construye la imagen de Docker si aún no está construida (o si se pasa la bandera `--build`).
- Ejecuta el contenedor con el montaje del volumen, el mapeo de puertos y el archivo `.env`.
- Imprime la URL para acceder a la aplicación.
- Opcionalmente abre el navegador.

**`scripts/stop_mac.sh`** (macOS/Linux):
- Detiene y elimina el contenedor en ejecución.
- NO elimina el volumen (los datos persisten).

**`scripts/start_windows.ps1`** / **`scripts/stop_windows.ps1`**: Equivalentes en PowerShell para Windows.

Todos los chips deben ser idempotentes — seguros de ejecutar múltiples veces.

### Despliegue en la Nube Opcional

El contenedor está diseñado para desplegarse en AWS App Runner, Render o cualquier plataforma de contenedores. Se puede proporcionar una configuración de Terraform para App Runner en un directorio `deploy/` como un objetivo adicional, pero no forma parte de la construcción principal.

---

## 12. Estrategia de Pruebas

### Pruebas Unitarias (dentro de `frontend/` y `backend/`)

**Backend (pytest)**:
- Datos de mercado: el simulador genera precios válidos, las matemáticas del GBM son correctas, el procesamiento de la respuesta de la API de Massive funciona, ambas implementaciones se ajustan a la interfaz abstracta.
- Cartera: lógica de ejecución de operaciones, cálculos de P&L, casos límite (vender más de lo que se posee, comprar con efectivo insuficiente, vender con pérdidas).
- LLM: el procesamiento de la salida estructurada maneja todos los esquemas válidos, manejo elegante de respuestas mal formadas, validación de operaciones dentro del flujo de chat.
- Rutas de la API: códigos de estado correctos, formas de respuesta, manejo de errores.

**Frontend (React Testing Library o similar)**:
- Renderizado de componentes con datos simulados (mock).
- El efecto de parpadeo de precios se activa correctamente ante cambios de precio.
- Operaciones CRUD de la lista de seguimiento.
- Cálculos de visualización de la cartera.
- Renderizado de mensajes de chat y estado de carga.

### Pruebas E2E (en `test/`)

**Infraestructura**: Un archivo `docker-compose.test.yml` separado en `test/` que levanta el contenedor de la aplicación más un contenedor de Playwright. Esto mantiene las dependencias del navegador fuera de la imagen de producción.

**Entorno**: Las pruebas se ejecutan con `LLM_MOCK=true` por defecto para mayor velocidad y determinismo.

**Escenarios Clave**:
- Inicio fresco: aparece la lista de seguimiento por defecto, se muestra el saldo de $10k, los precios se están transmitiendo.
- Añadir y eliminar un ticker de la lista de seguimiento.
- Comprar acciones: el efectivo disminuye, aparece la posición, la cartera se actualiza.
- Vender acciones: el efectivo aumenta, la posición se modifica o desaparece.
- Visualización de la cartera: el mapa de calor se renderiza con los colores correctos, el gráfico de P&L tiene puntos de datos.
- Chat de IA (simulado): enviar un mensaje, recibir una respuesta, la ejecución de la operación aparece en línea.
- Resiliencia de SSE: desconectar y verificar la reconexión.
