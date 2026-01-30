# Documentación Técnica del Proyecto: AIStock

Este documento detalla la arquitectura, el diseño de datos y las estrategias de desarrollo del MVP de **AIStock**.

---

#### 📄 0. User Stories and Mockups

### Actores del Sistema
* **Gerente de Depósito:** Supervisión global, análisis de racks y movimientos entre depósitos.
* **Usuario Normal:** Gestión de stock personal, registro de entradas/salidas y consultas rápidas.

### User Stories (MoSCoW)
* **Must Have:** * Interacción por **texto** con Chatbot (estilo Alexa) para estadísticas.
    * Aislamiento de datos por cliente (**Multi-tenant**).
    * Control de stock por Racks y Estanterías.
* **Won't Have (Out of Scope):** * Entrenamiento automático de modelos desde la app.
    * Predicciones de compra automáticas.

---

#### 📄 1. Design System Architecture

# 🏛️ Arquitectura del Ecosistema

Este sistema utiliza un patrón de diseño **BFF (Backend for Frontend)** con una estrategia de **Persistencia Políglota**, separando las responsabilidades en capas especializadas.

### 🖼️ Diagrama de Infraestructura
```mermaid
graph TD
    %% Styles
    classDef client fill:#e1f5fe,stroke:#01579b,stroke-width:2px,color:#01579b;
    classDef backend fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:#e65100;
    classDef db fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px,color:#1b5e20;
    classDef ai fill:#f3e5f5,stroke:#4a148c,stroke-width:2px,color:#4a148c;

    subgraph Clients ["💻 Capa de Presentación"]
        Desktop[Web Dashboard<br/><i>Next.js + Tailwind</i>]:::client
    end

    subgraph Cloud ["⚙️ Lógica de Negocio"]
        BFF[BFF API Gateway<br/><i>Node.js Runtime</i>]:::backend
    end

    subgraph Data ["💾 Persistencia Políglota"]
        Postgres[(PostgreSQL<br/>Relational Data)]:::db
        Supabase[(Supabase/Mongo<br/>Flexible Product Data)]:::db
    end

    subgraph External ["🤖 Inteligencia Artificial"]
        Gemini[Gemini API<br/><i>NLP & Intent Recognition</i>]:::ai
    end

    %%Connections
    Desktop -->|HTTPS / REST| BFF
    BFF -->|ORM / Prisma| Postgres
    BFF -->|JSON Schema| Supabase
    BFF -.->|Prompt Eng| Gemini
    Gemini -.->|Structured Response| BFF
```

---

### 🛠️ Desglose de la Infraestructura

#### 🎨 Capa de Cliente (Presentación)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat&logo=nextdotjs&logoColor=white) ![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=flat&logo=tailwind-css&logoColor=white)
* **Frontend:** Dashboard interactivo construido con **Next.js**.
* **Comunicación:** Intercambio de datos mediante **HTTPS/REST**, optimizado para tiempos de respuesta bajos y una interfaz reactiva.

#### 🧠 Lógica de Negocio (BFF Layer)
![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat&logo=nodedotjs&logoColor=white) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white)
* **Backend for Frontend (BFF):** Actúa como orquestador único, centralizando la seguridad y la lógica de negocio.
* **Aislamiento:** Garantiza que cada consulta respete los límites del **Multi-tenancy** mediante validación estricta de `tenant_id`.

#### 💾 Persistencia de Datos Híbrida
El sistema separa la información según su naturaleza para maximizar la eficiencia:

| Almacenamiento | Tecnología | Datos Gestionados |
| :--- | :--- | :--- |
| **Relacional** | `PostgreSQL` | Usuarios, permisos, racks y trazabilidad de stock. |
| **Documental** | `Supabase` | Catálogo de productos con atributos flexibles y esquemas variables. |

#### 🤖 Inteligencia Artificial (NLP)
![Google Gemini](https://img.shields.io/badge/Google_Gemini-8E75B2?style=flat&logo=googlegemini&logoColor=white)
* **Motor Cognitivo:** Utiliza **Gemini API** para la interpretación de intenciones (*Intent Recognition*).
* **Procesamiento:** Traduce las peticiones en lenguaje natural del usuario a parámetros de consulta técnicos y viceversa.

> [!NOTE]
> **Flujo de Ejecución:** El BFF coordina la entrada del usuario, solicita la interpretación a la IA, consulta las bases de datos correspondientes y devuelve una respuesta estructurada y humanizada.

---
---

#### 📄 Tarea 2 — Componentes, Clases y Diseño de Base de Datos
* **2.1** Componentes del Sistema
    El sistema utiliza una arquitectura de Backend for Frontend (BFF) con persistencia políglota.

    **Frontend** (Mobile - Flutter & Desktop - Next.js):

    **Responsabilidad:** Capa de presentación.

    **Mobile:** Enfocado en operarios (escaneo, movimientos rápidos, consulta en planta).

    **Desktop:** Enfocado en administradores (gestión de racks, ABM de productos, dashboard).

    **Interacción:** Se comunica exclusivamente con el Backend (Node.js) vía REST/JSON. No accede a la BD ni a la IA directamente.

    * **Backend (Node.js):**

    **Responsabilidad:** Orquestador de lógica de negocio, autenticación, autorización y validación. Actúa como intermediario entre el usuario, los datos y la inteligencia artificial.

    **Interacción:** Recibe peticiones del Frontend. Consulta PostgreSQL para datos relacionales y MongoDB para fichas de productos. Invoca a la API de Gemini para procesamiento de texto.

    **Agente** de IA (Gemini API):

    **Responsabilidad:** Procesamiento de Lenguaje Natural (NLP).

    **Funciones:**

    Input: Interpretar la intención del usuario (ej: "mover stock") y extraer entidades (ej: "Rack A", "Producto X").

    Output: Generar respuestas en lenguaje natural basadas en los datos estructurados que le provee el backend.

    Restricción: No almacena contexto a largo plazo ni entrena modelos.

    **Base de Datos Relacional (PostgreSQL):**

    Responsabilidad: Integridad referencial y datos estructurados. Almacena Tenants, Usuarios, Racks, Conteos de Inventario (IDs y cantidades) y Logs de movimientos.

    **Base de Datos Documental (MongoDB):**

    Responsabilidad: Flexibilidad de esquema. Almacena la información descriptiva de los Productos. Dado que cada tenant puede vender cosas distintas (ej: uno vende ropa con "talla/color", otro vende electrónica con "voltaje/potencia"), se requiere un esquema flexible.


# 📂2.2 Clases del Backend (Descripción UML)

A continuación, se definen las clases principales que residen en la capa de lógica del Backend (**Node.js**).

> [!IMPORTANT]
> **Nota de Implementación:** Todos los métodos asumen manejo asíncrono (**Promises / Async-Await**).

---

### 📊 Diagrama de Clases General
Visualización de las entidades y sus métodos principales:

```mermaid
---
config:
  layout: elk
  look: neo
  theme: default
---
classDiagram
    classDef core fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    classDef data fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    classDef ai #fff3e0,stroke:#e65100,stroke-width:2px;

    class Tenant:::core {
        +UUID id
        +String companyName
        +Enum status
        +isActive() Boolean
    }
    class User:::core {
        +UUID id
        +UUID tenantId
        +String name
        +role Enum
        +hasPermission(permission) Boolean
    }
    class Product:::data {
        +String id
        +UUID tenantId
        +String sku
        +JSON attributes
        +validateAttributes() Boolean
    }
    class Rack:::data {
        +UUID id
        +UUID tenantId
        +String code
        +location String
        +getStock() InventoryItem[]
    }
    class InventoryItem:::data {
        +UUID id
        +Integer quantity
        +increase(amount)
        +decrease(amount)
    }
    class AIService:::ai {
        +String apiKey
        +interpretIntent(text)
        +formatResponse(data, query)
    }

    Tenant "1" -- "*" User
    Tenant "1" -- "*" Product
    Tenant "1" -- "*" Rack
    Product "1" -- "*" InventoryItem
    Rack "1" -- "*" InventoryItem

    AIService ..> Product: Consult info
    AIService ..> InventoryItem : Consults stock
```


#### 📄 2.3 Diseño de Base de Datos
A. Base de Datos Relacional — PostgreSQL (ERD)
Este diagrama representa la estructura rígida para manejar la ubicación y cantidad del inventario, asegurando consistencia transaccional.
.

```mermaid
erDiagram
    TENANTS ||--o{ USERS : "has"
    TENANTS ||--o{ RACKS : "owns"
    TENANTS {
        uuid id PK
        string company_name
        string status
    }

    USERS {
        uuid id PK
        uuid tenant_id FK
        string email
        string password_hash
        string role
    }

    RACKS ||--o{ INVENTORY_ITEMS : "contains"
    RACKS ||--o{ STOCK_MOVEMENTS_FROM : "source_of"
    RACKS ||--o{ STOCK_MOVEMENTS_TO : "dest_of"
    RACKS {
        uuid id PK
        uuid tenant_id FK
        string code
        string location_desc
    }

    INVENTORY_ITEMS {
        uuid id PK
        string product_id "Ref a Mongo"
        uuid rack_id FK
        int quantity
        timestamp last_updated
    }

    STOCK_MOVEMENTS {
        uuid id PK
        string product_id "Ref a Mongo"
        uuid from_rack_id FK "Nullable"
        uuid to_rack_id FK "Nullable"
        int quantity
        timestamp created_at
        uuid user_id FK
    }

    %% Relaciones para Stock Movements
    RACKS ||--|{ STOCK_MOVEMENTS : "origin/dest"
    USERS ||--o{ STOCK_MOVEMENTS : "executes"
```

---

## 💾 B. Base de Datos Documental — MongoDB

Se utiliza para almacenar la información descriptiva de los productos, permitiendo que cada **Tenant** defina sus propios atributos sin afectar la estructura global.

### 📁 Colección: `products`

Esta colección maneja un esquema híbrido: campos fijos para integridad del sistema y un objeto flexible para datos comerciales.

#### 📄 Estructura del Documento (Ejemplo)

```json
{
  "_id": "64b8f...scan", 
  "tenant_id": "uuid-del-tenant-postgresql",
  "sku": "PROD-001",
  "name": "Zapatilla Running X",
  "description": "Zapatilla de alto rendimiento",
  "attributes": {
      "size": 42,
      "color": "Rojo",
      "material": "Sintético",
      "batch_number": "L-2024"
  },
  "created_at": "2024-01-20T10:00:00Z"
}
```
* Campos Obligatorios: _id, tenant_id (para aislamiento), sku, name.

* Campos Opcionales/Flexibles: Todo lo contenido dentro del objeto attributes.

#### 📄 2.4 Frontend — Componentes UI
* Login: Formulario simple (Email/Pass). Al loguearse, el backend determina el tenant_id del usuario y carga la configuración correspondiente.

* Dashboard: Vista resumen. Muestra métricas simples (Total de productos, Racks casi llenos, últimos movimientos). Sin predicciones.

* Gestión de Productos (Catalog): Tabla con buscador. Permite crear/editar productos (Define el JSON que va a Mongo).

* Gestión de Racks: Vista de lista o grilla de ubicaciones físicas. Permite ver el contenido actual de un rack.

* Movimientos de Stock: Interfaz transaccional. Selectores: "Desde Rack A" -> "Hacia Rack B" -> "Producto" -> "Cantidad". Botón de confirmar.

* **Chatbot Assistant:**

        UI: Botón flotante o panel lateral.

        Input: Campo de texto libre.

        Output: Burbujas de chat. Muestra texto plano ("Hay 5 unidades...") y, si corresponde, tarjetas de datos simples (mini tabla de resultados).

---

## 3. Sequence Diagram (Resumen de Proceso)

Secuencia 1 — Consulta de Stock por Chatbot
Caso: Usuario pregunta "¿Qué stock hay en el Rack B?". Nota: El Backend actúa como puente. Gemini solo "entiende" y luego "redacta", no consulta la DB.


```mermaid
sequenceDiagram
    participant User as Usuario
    participant FE as Frontend
    participant BE as Backend (Node)
    participant AI as Gemini API
    participant DB as PostgreSQL

    User->>FE: Escribe: "¿Qué stock hay en el Rack B?"
    FE->>BE: POST /api/chat (text, tenantId)
    
    rect rgb(240, 248, 255)
        note right of BE: Interpretación
        BE->>AI: Prompt: "Interpreta intención: '¿Qué stock hay en el Rack B?'"
        AI-->>BE: JSON: { intent: "GET_STOCK", filters: { rack_code: "B" } }
    end

    rect rgb(255, 250, 240)
        note right of BE: Consulta de Datos
        BE->>DB: SELECT * FROM racks WHERE code='B' AND tenant_id=...
        DB-->>BE: RackID: 123
        BE->>DB: SELECT * FROM inventory_items WHERE rack_id=123
        DB-->>BE: List: [{productId: "X", qty: 50}, {productId: "Y", qty: 10}]
    end

    rect rgb(240, 248, 255)
        note right of BE: Generación de Respuesta
        BE->>AI: Prompt: "Genera respuesta natural con estos datos: Rack B tiene Prod X (50), Prod Y (10)"
        AI-->>BE: Texto: "En el Rack B encontré 50 unidades del Producto X y 10 del Producto Y."
    end

    BE-->>FE: JSON { response: "En el Rack B..." }
    FE-->>User: Muestra mensaje en el chat
```

Secuencia 2 — Movimiento de Stock
Caso: Mover mercancía físicamente de un lugar a otro.


```mermaid
sequenceDiagram
    participant User as Usuario
    participant FE as Frontend
    participant BE as Backend (Node)
    participant DB as PostgreSQL

    User->>FE: Solicita Mover: 10 u. Prod X del Rack A al Rack B
    FE->>BE: POST /api/movements (fromRack, toRack, prodId, qty)
    
    BE->>BE: validateToken() & hasPermission()

    rect rgb(255, 230, 230)
        note right of BE: Transacción Atómica
        BE->>DB: BEGIN TRANSACTION
        
        BE->>DB: SELECT quantity FROM inventory_items WHERE rack='A' AND prod='X'
        DB-->>BE: qty: 15 (Validación OK, 15 >= 10)
        
        BE->>DB: UPDATE inventory_items SET qty = qty - 10 WHERE rack='A'
        BE->>DB: INSERT/UPDATE inventory_items SET qty = qty + 10 WHERE rack='B'
        BE->>DB: INSERT INTO stock_movements (log data...)
        
        BE->>DB: COMMIT
    end

    BE-->>FE: HTTP 200 OK (Success)
    FE-->>User: Muestra notificación "Movimiento exitoso"

```

Secuencia 3 — Creación de Producto
Caso: Alta de un nuevo producto con atributos flexibles (MongoDB).

```mermaid
sequenceDiagram
    participant User as Usuario
    participant FE as Frontend
    participant BE as Backend (Node)
    participant Mongo as MongoDB

    User->>FE: Completa form: SKU "T-100", Nombre "Camisa", Talla "L"
    FE->>BE: POST /api/products (JSON data)
    
    BE->>BE: validateAttributes() (Reglas básicas)

    rect rgb(230, 255, 230)
        note right of BE: Persistencia Documental
        BE->>Mongo: db.products.insertOne({ tenant_id, sku, name, attributes... })
        Mongo-->>BE: Ack (ObjectId: 507f1f77bcf86...)
    end

    BE-->>FE: HTTP 201 Created (Product ID)
    FE-->>User: Muestra "Producto creado correctamente"
```

---

## 4. API Specifications

* **External:** Google Gemini API (Análisis y Chatbot).
* **Internal POST `/api/v1/ai/chat`**: Punto de entrada para el agente de texto.
* **Internal GET `/api/v1/inventory/racks`**: Estado de los racks en tiempo real.

---

## 5. SCM and QA Strategies

### Estrategia de Código
* **Repositorio:** GitHub.
* **Branching:** GitHub Flow. Se requiere revisión de código antes de cualquier integración a `main`.
* **Editor:** Compatible 100% con Visual Studio Code.

### Estrategia de Calidad
* **Aislamiento (Security):** Verificación de que los contenedores de Cloud Run no compartan datos entre clientes.
* **IA Testing:** Validación de las respuestas del modelo LSTM sobre datos históricos.

---

## 6. Technical Justifications

* **LSTM:** Elegido por su capacidad de procesar secuencias y considerar eventos pasados para el análisis de inventario.
* **Multi-tenancy:** Implementado para cumplir con la seguridad empresarial, asegurando que cada cliente es dueño de su información.
* **MongoDB:** Justificado por la heterogeneidad de los productos que los clientes pueden registrar.
