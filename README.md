# Enterprise Universal Agent Platform (LangGraph + MCP)

[![LangGraph](https://img.shields.io/badge/Orchestrator-LangGraph%20v0.2-blue?logo=langchain&logoColor=white)]()
[![Protocol](https://img.shields.io/badge/Standard-Model%20Context%20Protocol%20%28MCP%29-10b981)]()
[![Vector DB](https://img.shields.io/badge/Retrieval-Milvus%20Vector%20Store-00A4E4)]()
[![Runtime](https://img.shields.io/badge/Python-3.12%2B%20AsyncIO-3776AB?logo=python&logoColor=white)]()
[![LLM Architecture](https://img.shields.io/badge/Architecture-Stateful%20Multi--Agent-orange)]()

## 📌 Resumen Ejecutivo
**LangGraph Agent Platform** es una arquitectura de orquestación multi-agente de nivel empresarial diseñada para integrar de forma desacoplada y estandarizada modelos de lenguaje avanzados (LLMs) con herramientas externas, APIs y bases de conocimiento mediante el **Model Context Protocol (MCP)**.

La solución implementa grafos dirigidos con memoria conversacional persistente (*State Graph with Summarization*), enrutamiento semántico dinámico mediante índices vectoriales (Milvus) y una capa de abstracción basada en el patrón *Strategy* para consumir servidores MCP locales o remotos (Stdio/SSE).

---

## 🏗️ Arquitectura de la Solución (Mermaid)

```mermaid
flowchart TD
    classDef client fill:#3b82f6,stroke:#fff,stroke-width:2px,color:#fff;
    classDef router fill:#8b5cf6,stroke:#fff,stroke-width:2px,color:#fff;
    classDef agent fill:#10b981,stroke:#fff,stroke-width:2px,color:#fff;
    classDef mcp fill:#f59e0b,stroke:#fff,stroke-width:2px,color:#fff;
    classDef store fill:#6366f1,stroke:#fff,stroke-width:2px,color:#fff;

    User([Usuario / Canal Corporativo]) --> RouterNode[Router Graph - Semantic Classifier]:::router
    
    subgraph VectorEngine ["🔍 Motor de Enrutamiento Semántico"]
        RouterNode <-->|Embeddings Match| VectorDB[(Milvus Vector Database)]:::store
        VectorDB -->|MCP Server Metadata| RouterNode
    end
    
    RouterNode --> Assistant[Assistant State Graph]:::agent
    
    subgraph AgentCore ["🧠 Núcleo de Decisión y Estado"]
        Assistant <--> StateMemory[(State Checkpointer & Summarizer)]:::store
        Assistant --> DecisionNode[LLM Tool Reasoner]:::agent
    end
    
    DecisionNode --> MCPWrapper[Generic MCP Wrapper Engine]:::mcp
    
    subgraph MCPServices ["🔌 Ecosistema de Herramientas MCP (Stdio / SSE)"]
        MCPWrapper <--> DBServer[Database MCP Server]:::mcp
        MCPWrapper <--> GitHubServer[GitHub / DevOps MCP]:::mcp
        MCPWrapper <--> CloudServer[Cloud Infrastructure MCP]:::mcp
    end
    
    MCPWrapper --> Assistant
    Assistant --> UserResponse([Respuesta Contextualizada]):::client
```

---

## 🔄 Flujo de Ejecución de la Memoria y Resumen

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuario
    participant AG as Assistant Graph
    participant SG as Summarizer Engine
    participant LLM as Modelo Foundation (LLM)
    participant MCP as MCP Tool Server

    U->>AG: Consulta con historial previo
    AG->>SG: Verificar ventana de contexto y tokens
    alt Si el contexto excede el umbral
        SG->>LLM: Generar síntesis progresiva de la conversación
        LLM-->>SG: Estado comprimido (Summary)
        SG-->>AG: State actualizado
    end
    AG->>LLM: Inferencia con herramientas seleccionadas
    LLM-->>AG: Tool Call (MCP Command)
    AG->>MCP: Ejecución asíncrona de la herramienta
    MCP-->>AG: Retorno de datos estructurados
    AG->>LLM: Inferencia final con resultados
    LLM-->>U: Respuesta ejecutiva resuelta
```

---

## 📚 Estructura de Componentes

```text
langgraph-mcp-agent/
├── src/
│   └── langgraph_mcp/
│       ├── assistant_graph.py                     # Grafo principal del asistente
│       ├── assistant_graph_with_summarization.py  # Grafo con gestión de memoria y síntesis
│       ├── build_router_graph.py                  # Generador del índice de enrutamiento
│       ├── configuration.py                       # Parámetros y schemas tipados
│       ├── mcp_wrapper.py                         # Wrapper genérico (Strategy Pattern)
│       ├── prompts.py                             # Plantillas de razonamiento
│       ├── retriever.py                           # Integración con Milvus Vector DB
│       ├── state.py                               # Modelos de estado tipados (Pydantic)
│       └── utils/                                 # Utilidades y conversores OpenAPI
├── tests/                                         # Batería de pruebas unitarias y de integración
├── langgraph.json                                 # Despliegue en LangGraph Cloud / Server
├── mcp-servers-config.sample.json                 # Configuración de topología de herramientas
└── README.md                                      # Documentación ejecutiva
```

---

## 🔐 Capacidades Enterprise
1. **Desacoplamiento Total:** Los agentes no requieren hardcodear herramientas; descubren dinámicamente capacidades leyendo el catálogo expuesto por los servidores MCP.
2. **Resiliencia de Memoria:** Manejo automático de conversaciones extensas mediante algoritmos de compresión y resumen en tiempo real.
3. **Escalabilidad Vectorial:** Capacidad para enrutar entre cientos de herramientas MCP sin saturar el contexto del modelo, gracias a la búsqueda por similitud en Milvus.
