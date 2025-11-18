# Diagramas de Arquitectura - Sistemas de Agentes IA

## 🏗️ Vista General del Ecosistema

### 1. Arquitectura Completa del Sistema

```mermaid
graph TB
    subgraph "User Interfaces"
        A[Web Chat Interface]
        B[Phone System]
    end
    
    subgraph "Entry Points"
        C[FastAPI Gateway]
        D[ElevenLabs Voice]
    end
    
    subgraph "Orchestration Layer"
        E[Agent Router<br/>NLP Classification]
        F[Voice Agent System<br/>n8n + ElevenLabs]
    end
    
    subgraph "Specialized Agents"
        G[FAQ-RAG Agent<br/>Pinecone + DuckDuckGo]
        H[Sales Agent]
        I[Support Agent]
    end
    
    subgraph "Data & Integration"
        J[Pinecone Vector DB]
        K[Zoho CRM]
        L[AWS S3 + Google Sheets]
    end
    
    subgraph "Observability"
        M[Langfuse Tracing]
    end
    
    A --> C
    B --> D
    C --> E
    D --> F
    
    E --> G
    E --> H
    E --> I
    F --> K
    
    G --> J
    H --> K
    I --> K
    F --> L
    
    G --> M
    
    style E fill:#e3f2fd
    style G fill:#f3e5f5
    style F fill:#fff3e0
    style M fill:#e8f5e8
```

---

## 📞 Sistema de Voz Automatizado

### 2. Flujo de Gestión de Leads y Llamadas Automatizadas

```mermaid
flowchart TD
    A[Equipo de Negocio<br/>Selecciona Clientes] --> B[Google Sheets<br/>Alimentación Manual de Leads]
    B --> C[Lead Data<br/>Sin Fecha de Procesamiento]
    
    D[Cron Scheduler<br/>9:00 AM Lunes-Viernes] --> E[Consultar Google Sheets]
    E --> F{Filtrar Leads<br/>fecha_procesamiento = BLANK}
    
    F -->|Leads Sin Procesar| G[Validar Datos del Lead]
    F -->|Todos Procesados| H[No hay llamadas pendientes]
    
    G --> I[Verificar Prioridad<br/>y Disponibilidad]
    I -->|Lead Válido| J[Preparar Datos de Llamada]
    I -->|Lead No Válido| K[Marcar como Omitido]
    
    J --> L[Llamada API ElevenLabs]
    L --> M[ElevenLabs + Twilio Integration]
    M --> N[Llamada Saliente Iniciada]
    
    N --> O[Conversación con IA]
    O --> P[Finalización de Llamada]
    P --> Q[Actualizar Google Sheets<br/>fecha_procesamiento = HOY]
    
    Q --> R[Lead Marcado como Procesado<br/>No se volverá a llamar]
    
    style A fill:#f3e5f5
    style C fill:#fff3e0
    style F fill:#e3f2fd
    style Q fill:#e8f5e8
    style R fill:#e1f5fe
```

### 3. Flujo de Datos Completo del Sistema de Voz

```mermaid
sequenceDiagram
    participant BIZ as Equipo de Negocio
    participant GS as Google Sheets
    participant CRON as Cron Scheduler
    participant EL as ElevenLabs Agent
    participant TW as Twilio (Números)
    participant N8N as n8n Webhooks
    participant ZOHO as Zoho CRM
    participant S3 as AWS S3
    
    Note over BIZ,GS: Alimentación inicial de leads
    BIZ->>GS: Agregar nuevos leads (fecha_procesamiento = BLANK)
    
    Note over CRON,GS: Procesamiento automático diario
    CRON->>GS: Consultar leads con fecha_procesamiento = BLANK
    GS-->>CRON: Lista de leads sin procesar
    
    alt Hay leads pendientes
        CRON->>EL: Iniciar llamada automática
        EL->>TW: Usar número Twilio para llamada
        TW-->>EL: Número disponible
        
        Note over EL: ElevenLabs maneja toda la conversación
        
        EL->>N8N: Tool call - Actualizar CRM
        N8N->>ZOHO: Actualizar información del lead
        ZOHO-->>N8N: Confirmación de actualización
        N8N-->>EL: Respuesta del tool
        
        EL->>N8N: Webhook post-llamada (con audio URL y transcripción)
        N8N->>S3: Descargar y almacenar audio desde ElevenLabs
        S3-->>N8N: URL de archivo almacenado en S3
        N8N->>GS: Actualizar fecha_procesamiento = HOY + datos de llamada
        GS-->>N8N: Lead marcado como procesado
        
        Note over GS: Lead no volverá a ser llamado
    else No hay leads pendientes
        CRON->>CRON: Esperar próxima ejecución
    end
```

---

## 🤖 Sistema Multi-Agente FAQ-RAG

### 4. Arquitectura del Agent Router y FAQ-RAG

```mermaid
graph TB
    subgraph "Entry Point"
        A[Usuario<br/>Consulta]
        B[FastAPI Router<br/>Punto de Entrada]
    end
    
    subgraph "Agent Router Layer"
        C[Agent Router<br/>Supervisor Inteligente]
        D[Query Analysis<br/>Análisis de Consulta]
        E[Agent Selection<br/>Selección de Agente]
    end
    
    subgraph "Specialized Agents"
        F[FAQ-RAG Agent<br/>Preguntas Frecuentes]
        G[Sales Agent<br/>Proceso de Ventas]
        H[Support Agent<br/>Soporte Técnico]
    end
    
    subgraph "FAQ-RAG Agent Core"
        I[LangGraph State Manager<br/>Gestión de Estado]
        J[ReAct Agent Engine<br/>Motor de Razonamiento]
        K[Tool Selector<br/>Selector de Herramientas]
    end
    
    subgraph "Tools Layer"
        L[FAQ Vector Search<br/>Búsqueda Vectorial]
        M[Web RAG Search<br/>Búsqueda Web]
        N[Context Processor<br/>Procesador de Contexto]
    end
    
    subgraph "Data Sources"
        O[Pinecone Vector DB<br/>Base de Conocimientos]
        P[DuckDuckGo API<br/>Información Web]
        Q[Langfuse Tracing<br/>Trazabilidad]
    end
    
    A --> B
    B --> C
    C --> D
    D --> E
    
    E -->|FAQ Query| F
    E -->|Sales Query| G
    E -->|Support Query| H
    
    F --> I
    I --> J
    J --> K
    
    K --> L
    K --> M
    L --> O
    M --> P
    
    N --> Q
    
    style C fill:#e3f2fd
    style F fill:#f3e5f5
    style I fill:#fff3e0
    style O fill:#e8f5e8
```

### 5. Flujo del FAQ-RAG Agent con Búsqueda Híbrida

```mermaid
flowchart TD
    A[Agent Router<br/>Envía Consulta FAQ] --> B[FAQ-RAG Agent<br/>Recibe Consulta]
    B --> C[LangGraph State<br/>Inicialización]
    C --> D[Análisis de Consulta<br/>Query Processing]
    
    D --> E{Tipo de<br/>Información}
    
    E -->|Conocimiento Interno| F[Activar FAQ Tool]
    E -->|Información Externa| G[Activar RAG Tool]
    E -->|Información Completa| H[Activar Ambas Tools]
    
    F --> I[Búsqueda Vectorial<br/>Pinecone]
    G --> J[Búsqueda Web<br/>DuckDuckGo]
    H --> K[Ejecución Paralela<br/>de Tools]
    
    I --> L[Procesar Resultados FAQ]
    J --> M[Procesar Resultados Web]
    K --> N[Combinar Resultados]
    
    L --> O[Generación de Respuesta<br/>LLM Processing]
    M --> O
    N --> O
    
    O --> P[Respuesta Final]
    P --> Q[Logging Langfuse<br/>Trazabilidad]
    Q --> R[Métricas DataDog<br/>Monitoreo]
    R --> S[Retorno al Usuario]
    
    style A fill:#e3f2fd
    style F fill:#fff3e0
    style G fill:#f3e5f5
    style O fill:#e1f5fe
    style S fill:#e8f5e8
```

---


