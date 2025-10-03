# WorkFlowConnect - Arquitectura y Métricas de Calidad

## Descripción General

WorkFlowConnect es una plataforma orientada a conectar freelancers con clientes mediante un sistema robusto de mensajería, gestión de proyectos y control de calidad. El sistema integra herramientas de monitoreo, análisis de código y métricas de rendimiento para garantizar un software confiable, mantenible y de alto desempeño.

---

## 1. Objetivo de los Diagramas

Los diagramas constituyen una herramienta visual que permite entender cómo se articula la arquitectura de WorkFlowConnect con los mecanismos de aseguramiento de calidad definidos en este documento. A través de ellos se busca:

- **Mostrar los componentes principales del sistema y sus interacciones**
- **Representar la integración de herramientas de calidad, monitoreo y métricas**
- **Evidenciar cómo cada grupo de métricas se relaciona con elementos técnicos específicos** (frontend, backend, base de datos, servicios y herramientas externas)

---

## 2. Arquitectura de WorkFlowConnect (Vista Simplificada)

La siguiente vista de arquitectura refleja los módulos principales de WorkFlowConnect:

<lov-mermaid>
graph TD
    subgraph Cliente
        A[Frontend React] --> B[Backend Node.js + Express]
    end

    subgraph Servidor
        B --> C[(PostgreSQL DB)]
        B --> D[Servicio WebSockets<br/>Mensajería en tiempo real]
        B --> E[Servicio de Archivos<br/>Subida/Descarga]
    end

    subgraph Monitoreo y Calidad
        F[SonarQube<br/>Calidad del código]
        G[Prometheus + Grafana<br/>Rendimiento]
        H[Sentry / ELK<br/>Confiabilidad y errores]
    end

    B --> F
    B --> G
    B --> H
</lov-mermaid>

### Componentes Principales

- **Frontend React**: Interfaz de usuario responsiva construida con React, TypeScript y Tailwind CSS
- **Backend Node.js + Express**: API REST que maneja la lógica de negocio y autenticación
- **PostgreSQL DB**: Base de datos relacional para almacenar usuarios, trabajos, mensajes y archivos
- **Servicio WebSockets**: Comunicación en tiempo real para el sistema de mensajería
- **Servicio de Archivos**: Gestión de subida y descarga de archivos adjuntos

### Herramientas de Calidad

- **SonarQube**: Análisis estático de código, detección de bugs y vulnerabilidades
- **Prometheus + Grafana**: Monitoreo de rendimiento y visualización de métricas
- **Sentry / ELK**: Seguimiento de errores y logs para análisis de confiabilidad

---

## 3. Relación entre Métricas y Componentes

Este diagrama representa cómo cada grupo de métricas se conecta con los componentes de la arquitectura y con las herramientas utilizadas para su evaluación:

<lov-mermaid>
graph LR
    subgraph Métricas
        C1[Calidad del código]
        C2[Rendimiento]
        C3[Confiabilidad]
        C4[Mantenibilidad]
    end

    subgraph Herramientas
        F[SonarQube]
        G[Prometheus / JMeter / Grafana]
        H[Sentry / ELK]
    end

    C1 --> F
    C4 --> F
    C2 --> G
    C3 --> H
</lov-mermaid>

---

## 4. Cuadro de Trazabilidad (Métricas ↔ Componentes ↔ Herramientas)

| Métrica | Componente evaluado | Herramienta principal | Estándar asociado |
|---------|---------------------|----------------------|-------------------|
| **Cobertura de pruebas / Complejidad ciclomática** | Código backend (Node.js, Express) | SonarQube, Jest | ISO/IEC 25010, McCall |
| **Latencia / Throughput** | API REST, WebSockets, DB | JMeter, Prometheus, Grafana | ISO/IEC 25010, Boehm |
| **Disponibilidad / MTTR** | Backend + Servicios | Sentry, ELK, Prometheus | ISO/IEC 25010, 27001 |
| **Índice de mantenibilidad / Duplicación de código** | Arquitectura modular / Código compartido | SonarQube | ISO/IEC 25010, 12207 |

---

## 5. Métricas de Calidad Detalladas

### 5.1. Calidad del Código

- **Cobertura de pruebas**: Porcentaje de código cubierto por tests automatizados (objetivo: >80%)
- **Complejidad ciclomática**: Medida de la complejidad de los métodos (objetivo: <10 por función)
- **Vulnerabilidades**: Detección de problemas de seguridad (objetivo: 0 críticas)
- **Code smells**: Problemas de diseño que afectan la mantenibilidad

### 5.2. Rendimiento

- **Latencia**: Tiempo de respuesta de las APIs (objetivo: <200ms p95)
- **Throughput**: Número de peticiones procesadas por segundo
- **Tiempo de carga**: Velocidad de carga del frontend (objetivo: <3s)
- **Uso de recursos**: CPU y memoria del servidor

### 5.3. Confiabilidad

- **Disponibilidad**: Porcentaje de tiempo operativo (objetivo: 99.5%)
- **MTTR** (Mean Time To Recovery): Tiempo promedio de recuperación ante fallos
- **Tasa de errores**: Porcentaje de peticiones fallidas (objetivo: <1%)
- **Manejo de excepciones**: Cobertura de casos de error

### 5.4. Mantenibilidad

- **Índice de mantenibilidad**: Facilidad para modificar el código (0-100, objetivo: >70)
- **Duplicación de código**: Porcentaje de código duplicado (objetivo: <5%)
- **Modularidad**: Separación de responsabilidades y acoplamiento
- **Documentación**: Cobertura de comentarios y documentación técnica

---

## 6. Stack Tecnológico

### Frontend
- React 18.3
- TypeScript
- Vite
- Tailwind CSS
- shadcn/ui
- React Router DOM
- React Hook Form + Zod
- Socket.io Client

### Backend
- Node.js
- Express.js
- PostgreSQL
- Socket.io
- JWT Authentication
- Bcrypt

### Herramientas de Desarrollo
- ESLint
- Jest (testing)
- SonarQube (análisis estático)
- Prometheus (monitoreo)
- Grafana (visualización)

---

## 7. Flujo de Datos Principal

<lov-mermaid>
sequenceDiagram
    participant U as Usuario
    participant F as Frontend React
    participant B as Backend API
    participant DB as PostgreSQL
    participant WS as WebSocket

    U->>F: Interacción (crear propuesta, enviar mensaje)
    F->>B: HTTP Request (REST API)
    B->>DB: Query/Insert
    DB-->>B: Response
    B-->>F: HTTP Response
    F-->>U: Actualización UI
    
    Note over WS: Para mensajería en tiempo real
    U->>F: Enviar mensaje
    F->>WS: Emit message
    WS->>B: Procesar mensaje
    B->>DB: Guardar mensaje
    WS-->>F: Broadcast a usuarios
    F-->>U: Mostrar mensaje
</lov-mermaid>

---

## 8. Arquitectura de Base de Datos

### Entidades Principales

- **Users**: Información de usuarios (freelancers y clientes)
- **Jobs**: Propuestas de trabajo publicadas
- **Comments**: Comentarios en propuestas
- **Replies**: Respuestas a comentarios
- **Chats**: Conversaciones entre usuarios
- **Messages**: Mensajes individuales
- **Files**: Archivos adjuntos
- **JobLikes**: Likes en propuestas
- **SavedJobs**: Propuestas guardadas por usuarios

<lov-mermaid>
erDiagram
    Users ||--o{ Jobs : creates
    Users ||--o{ Comments : writes
    Users ||--o{ Replies : writes
    Users ||--o{ Messages : sends
    Users ||--o{ ChatParticipants : participates
    Jobs ||--o{ Comments : has
    Jobs ||--o{ JobLikes : receives
    Jobs ||--o{ SavedJobs : saved_by
    Comments ||--o{ Replies : has
    Chats ||--o{ Messages : contains
    Chats ||--o{ ChatParticipants : has
    
    Users {
        uuid id PK
        string name
        string email UK
        string password
        string role
        text bio
        array skills
        string photoURL
        float hourlyRate
        boolean isOnline
        timestamp lastSeen
    }
    
    Jobs {
        uuid id PK
        string title
        text description
        float budget
        string category
        array skills
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
    }
</lov-mermaid>

---

## 9. Seguridad

- **Autenticación JWT**: Tokens seguros para sesiones de usuario
- **Bcrypt**: Hash de contraseñas con salt
- **Validación de datos**: Zod schemas en frontend, validación en backend
- **SQL Injection Protection**: Uso de queries parametrizadas
- **CORS**: Configuración de orígenes permitidos
- **Rate Limiting**: Protección contra ataques de fuerza bruta

---

## 10. Próximos Pasos de Mejora

1. **Implementación de CI/CD**: Pipeline automatizado con GitHub Actions
2. **Testing E2E**: Cypress o Playwright para pruebas de integración
3. **Caché**: Redis para mejorar rendimiento
4. **CDN**: Distribución de assets estáticos
5. **Escalabilidad**: Arquitectura de microservicios
6. **Monitoring Avanzado**: APM (Application Performance Monitoring)

---

## Referencias

- **ISO/IEC 25010**: Estándar de calidad de software
- **ISO/IEC 27001**: Gestión de seguridad de la información
- **ISO/IEC 12207**: Procesos del ciclo de vida del software

---

## Contacto y Contribución

Para contribuir al proyecto o reportar issues, por favor revisa la documentación en `src/README.md` y sigue las guías de contribución establecidas.
