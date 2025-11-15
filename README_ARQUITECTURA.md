# Arquitectura del Proyecto WorkFlowConnect

## 📋 Tabla de Contenidos
- [Tipo de Arquitectura](#tipo-de-arquitectura)
- [Justificación del Diseño](#justificación-del-diseño)
- [Componentes Principales](#componentes-principales)
- [Comunicación Frontend-Backend](#comunicación-frontend-backend)
- [Diagramas de Arquitectura](#diagramas-de-arquitectura)

---

## 🏗️ Tipo de Arquitectura

**WorkFlowConnect utiliza una arquitectura Cliente-Servidor de dos capas** con las siguientes características:

### Características Principales:
- **Frontend (Cliente)**: Aplicación React SPA (Single Page Application)
- **Backend (Servidor)**: API REST con Node.js + Express
- **Base de Datos**: PostgreSQL
- **Comunicación en Tiempo Real**: WebSockets (Socket.io)

### Por qué Cliente-Servidor y no Microservicios:

#### ✅ Ventajas de Cliente-Servidor para este Proyecto:

1. **Simplicidad de Desarrollo y Mantenimiento**
   - Menor complejidad operacional
   - Un solo repositorio de backend
   - Despliegue más sencillo
   - Debugging más directo

2. **Cohesión del Dominio**
   - WorkFlowConnect es un sistema cohesivo donde todas las funcionalidades están estrechamente relacionadas:
     - Usuarios → Jobs → Mensajes → Archivos
   - No hay subdominios independientes que justifiquen la separación en microservicios

3. **Tamaño y Escala del Proyecto**
   - Proyecto de tamaño mediano
   - No requiere escalado independiente de componentes
   - El tráfico esperado puede ser manejado eficientemente por un monolito modular

4. **Consistencia de Datos**
   - Transacciones ACID más sencillas
   - No hay complejidad de transacciones distribuidas
   - Integridad referencial directa en PostgreSQL

5. **Performance**
   - Menor latencia (no hay comunicación entre servicios)
   - Sin overhead de serialización entre microservicios
   - Consultas JOIN eficientes en la base de datos

### Por qué NO es Monolítico Tradicional:

Aunque usa arquitectura cliente-servidor, **NO es un monolito tradicional** porque:

1. **Separación Clara de Capas**
   ```
   Frontend (React) ←→ API REST (Express) ←→ Base de Datos (PostgreSQL)
   ```

2. **Modularidad Interna**
   - Código organizado por módulos funcionales
   - Controladores, servicios, modelos y rutas separados
   - Arquitectura de componentes en el frontend

3. **Comunicación por API REST**
   - El frontend y backend están desacoplados
   - Podrían desplegarse en servidores diferentes
   - Permite escalado horizontal del backend si es necesario

---

## 🔧 Componentes Principales

### 1. Frontend (Cliente)
```
src/
├── components/     # Componentes React reutilizables
├── pages/         # Vistas principales de la aplicación
├── contexts/      # Estado global (Auth, Chat, Data, Job)
├── services/      # Servicios para comunicación con API
├── hooks/         # Custom hooks
└── types/         # Definiciones TypeScript
```

**Tecnologías:**
- React 18.3 con TypeScript
- Vite (build tool)
- Tailwind CSS + shadcn/ui
- React Router DOM (navegación)
- Socket.io Client (WebSockets)
- React Hook Form + Zod (formularios)

### 2. Backend (Servidor)
```
backend/
├── controllers/   # Lógica de negocio
├── models/        # Modelos de datos
├── routes/        # Definición de endpoints
├── middleware/    # Autenticación y validaciones
├── socket/        # Manejo de WebSockets
└── config/        # Configuración de DB
```

**Tecnologías:**
- Node.js + Express
- PostgreSQL (base de datos)
- JWT (autenticación)
- Socket.io (tiempo real)
- Bcrypt (seguridad)

### 3. Base de Datos
- **PostgreSQL**: Base de datos relacional
- **Entidades principales**: Users, Jobs, Comments, Replies, Chats, Messages, Files

---

## 🔄 Comunicación Frontend-Backend

### 1. Comunicación HTTP (REST API)

El frontend se comunica con el backend principalmente a través de una **API REST** usando Axios:

<lov-mermaid>
sequenceDiagram
    participant F as Frontend React
    participant A as Axios Service
    participant B as Backend API
    participant DB as PostgreSQL

    F->>A: Acción del usuario (ej: crear job)
    A->>B: POST /api/jobs
    Note over A,B: HTTP Request con JWT token
    B->>B: Validar autenticación
    B->>DB: INSERT query
    DB-->>B: Respuesta
    B-->>A: HTTP 200 + JSON data
    A-->>F: Actualizar estado React
    F->>F: Re-render UI
</lov-mermaid>

#### Flujo de Autenticación:
```typescript
// 1. Usuario inicia sesión
POST /api/auth/login
Body: { email, password }
Response: { user, token }

// 2. Token almacenado en localStorage
localStorage.setItem('token', token)

// 3. Todas las peticiones incluyen el token
axios.get('/api/jobs', {
  headers: { Authorization: `Bearer ${token}` }
})
```

#### Endpoints Principales:

**Autenticación:**
- `POST /api/auth/register` - Registro de usuario
- `POST /api/auth/login` - Inicio de sesión
- `GET /api/auth/me` - Obtener usuario actual

**Jobs:**
- `GET /api/jobs` - Listar trabajos
- `POST /api/jobs` - Crear trabajo
- `GET /api/jobs/:id` - Obtener detalles
- `PUT /api/jobs/:id` - Actualizar trabajo
- `DELETE /api/jobs/:id` - Eliminar trabajo

**Mensajes:**
- `GET /api/chats` - Listar chats del usuario
- `POST /api/chats` - Crear nuevo chat
- `GET /api/messages/:chatId` - Obtener mensajes de un chat
- `POST /api/messages` - Enviar mensaje

**Archivos:**
- `POST /api/files` - Subir archivo
- `GET /api/files/:id` - Descargar archivo
- `DELETE /api/files/:id` - Eliminar archivo

### 2. Comunicación en Tiempo Real (WebSockets)

Para la funcionalidad de **mensajería instantánea**, el proyecto usa **Socket.io**:

<lov-mermaid>
sequenceDiagram
    participant U1 as Usuario 1 (Frontend)
    participant WS as Socket.io Server
    participant U2 as Usuario 2 (Frontend)
    participant DB as PostgreSQL

    U1->>WS: Conectar WebSocket
    Note over U1,WS: Socket.io handshake
    WS-->>U1: Conexión establecida
    
    U1->>WS: emit('join-chat', chatId)
    Note over WS: Usuario se une a sala
    
    U1->>WS: emit('send-message', {chatId, content})
    WS->>DB: Guardar mensaje
    DB-->>WS: Mensaje guardado
    
    WS-->>U1: emit('message-sent', message)
    WS-->>U2: emit('new-message', message)
    Note over U2: Usuario 2 recibe mensaje en tiempo real
    
    U2->>U2: Actualizar UI automáticamente
</lov-mermaid>

#### Eventos de Socket.io:

**Cliente → Servidor:**
```typescript
// Conectar y autenticar
socket.emit('authenticate', { token })

// Unirse a un chat
socket.emit('join-chat', chatId)

// Enviar mensaje
socket.emit('send-message', { chatId, content })

// Usuario escribiendo...
socket.emit('typing', { chatId, userId })
```

**Servidor → Cliente:**
```typescript
// Confirmación de autenticación
socket.on('authenticated', (user) => {...})

// Nuevo mensaje recibido
socket.on('new-message', (message) => {...})

// Usuario escribiendo
socket.on('user-typing', ({ chatId, userId }) => {...})

// Estado de conexión de usuarios
socket.on('user-online', (userId) => {...})
socket.on('user-offline', (userId) => {...})
```

### 3. Arquitectura de Comunicación Completa

<lov-mermaid>
graph TB
    subgraph "Cliente (Navegador)"
        UI[Interfaz React]
        RC[React Context]
        AX[Axios Client]
        SC[Socket.io Client]
    end

    subgraph "Servidor"
        API[API REST Express]
        SW[Socket.io Server]
        AUTH[JWT Middleware]
        CTRL[Controllers]
    end

    subgraph "Datos"
        DB[(PostgreSQL)]
        FS[Sistema de Archivos]
    end

    UI -->|Acciones usuario| RC
    RC -->|HTTP Requests| AX
    RC -->|Eventos tiempo real| SC
    
    AX -->|REST API| API
    SC <-->|WebSocket| SW
    
    API --> AUTH
    AUTH --> CTRL
    CTRL --> DB
    CTRL --> FS
    
    SW --> CTRL
    
    style UI fill:#61dafb
    style API fill:#68a063
    style DB fill:#336791
</lov-mermaid>

---

## 📊 Diagramas de Arquitectura

### Arquitectura General del Sistema

<lov-mermaid>
graph TD
    subgraph "Cliente"
        A[Frontend React + TypeScript]
        A1[React Router]
        A2[React Context API]
        A3[Axios HTTP Client]
        A4[Socket.io Client]
        
        A --> A1
        A --> A2
        A --> A3
        A --> A4
    end

    subgraph "Servidor"
        B[Backend Node.js + Express]
        B1[Rutas API REST]
        B2[Middleware JWT]
        B3[Controladores]
        B4[Socket.io Server]
        
        B --> B1
        B --> B2
        B --> B3
        B --> B4
    end

    subgraph "Persistencia"
        C[(PostgreSQL)]
        C1[Usuarios]
        C2[Jobs]
        C3[Mensajes]
        C4[Archivos]
        
        C --> C1
        C --> C2
        C --> C3
        C --> C4
    end

    subgraph "Monitoreo y Calidad"
        M1[SonarQube]
        M2[Prometheus + Grafana]
        M3[Sentry / ELK]
    end

    A3 -->|HTTPS| B1
    A4 <-->|WSS| B4
    B2 --> B3
    B3 --> C
    B4 --> C
    
    B --> M1
    B --> M2
    B --> M3
</lov-mermaid>

### Flujo de Datos de una Petición Completa

<lov-mermaid>
sequenceDiagram
    autonumber
    participant U as Usuario
    participant R as React UI
    participant C as Context API
    participant S as Axios Service
    participant A as Express API
    participant M as Middleware
    participant Ctrl as Controller
    participant DB as PostgreSQL

    U->>R: Click "Crear Propuesta"
    R->>C: Dispatch action
    C->>S: jobService.createJob(data)
    S->>A: POST /api/jobs
    A->>M: Verificar JWT token
    M-->>A: Token válido
    A->>Ctrl: jobController.create()
    Ctrl->>DB: INSERT INTO jobs
    DB-->>Ctrl: Job creado
    Ctrl-->>A: Response 201
    A-->>S: { success: true, job }
    S-->>C: Actualizar estado
    C-->>R: Re-render
    R-->>U: Mostrar confirmación
</lov-mermaid>

### Modelo de Datos Simplificado

<lov-mermaid>
erDiagram
    Users ||--o{ Jobs : "crea"
    Users ||--o{ Messages : "envía"
    Users ||--o{ ChatParticipants : "participa"
    Jobs ||--o{ Comments : "tiene"
    Comments ||--o{ Replies : "tiene"
    Chats ||--o{ Messages : "contiene"
    Chats ||--o{ ChatParticipants : "incluye"
    Messages ||--o| Files : "puede tener"

    Users {
        uuid id PK
        string name
        string email UK
        string password
        string role
        text bio
        jsonb skills
    }

    Jobs {
        uuid id PK
        string title
        text description
        decimal budget
        string category
        jsonb skills
        string status
        uuid userId FK
    }

    Chats {
        uuid id PK
        string name
        boolean isGroup
        timestamp lastMessageAt
    }

    Messages {
        uuid id PK
        text content
        boolean read
        uuid chatId FK
        uuid userId FK
        uuid fileId FK
    }
</lov-mermaid>

---

## 🚀 Ventajas de la Arquitectura Actual

### 1. **Desarrollo Ágil**
- Stack único y consistente (JavaScript/TypeScript)
- Curva de aprendizaje reducida
- Desarrollo más rápido

### 2. **Mantenibilidad**
- Código organizado y modular
- Separación clara de responsabilidades
- Fácil de testear

### 3. **Performance**
- Latencia baja entre componentes
- Cache eficiente en PostgreSQL
- WebSockets para tiempo real

### 4. **Escalabilidad**
- Escalado horizontal del backend posible
- Base de datos PostgreSQL escalable
- CDN para assets estáticos (futuro)

### 5. **Costos**
- Infraestructura simple
- Un solo servidor puede manejar toda la carga inicial
- Costos operacionales reducidos

---

## 🔮 Evolución Futura

Si el proyecto crece significativamente, podría evolucionar hacia microservicios separando:

1. **Servicio de Autenticación**
2. **Servicio de Jobs y Propuestas**
3. **Servicio de Mensajería en Tiempo Real**
4. **Servicio de Archivos y Storage**

Pero por ahora, la arquitectura cliente-servidor es la opción más eficiente y pragmática.

---

## 📚 Referencias

- [Arquitectura Cliente-Servidor](https://en.wikipedia.org/wiki/Client%E2%80%93server_model)
- [REST API Best Practices](https://restfulapi.net/)
- [Socket.io Documentation](https://socket.io/docs/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

---

## 👥 Contribución

Para contribuir al proyecto, consulta `ARCHITECTURE.md` para ver las métricas de calidad y estándares del código.
