# Documentación Técnica del Proyecto: AIStock

Este documento detalla la arquitectura, el diseño de datos y las estrategias de desarrollo del MVP de **AIStock**.

---

## 0. User Stories and Mockups

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

## 1. Design System Architecture

El sistema utiliza una infraestructura en la nube basada en **Cloud Run** para garantizar que cada empresa (tenant) tenga su entorno aislado.


* **Frontend:** Flutter (Mobile) y Next.js + Tailwind (Desktop).
* **BFF (Backend for Frontend):** Node.js.
* **IA Core:** Python con modelos **LSTM** (para memoria de eventos anteriores y secuencias) y **XGBoost**.
* **Agente:** Gemini API para la interpretación de lenguaje natural.

---

## 2. Components and Database Design

### Modelo de Datos Híbrido
1.  **PostgreSQL (Relacional):** Control estricto de inventario, usuarios y transacciones.
2.  **MongoDB (NoSQL):** Catálogo de productos con **esquema flexible**, permitiendo que cada cliente defina sus propios atributos.
3.  **Redis:** Caché de alto rendimiento y sistema de notificaciones en tiempo real.

### Estructura Física
* El sistema mapea el centro de almacenaje mediante **Estanterías** y **Racks**, permitiendo un seguimiento exacto de la ubicación de cada producto.

---

## 3. Sequence Diagram (Resumen de Proceso)

| Paso | Actor | Acción |
| :--- | :--- | :--- |
| 1 | Usuario | Envía consulta de texto: "¿Qué stock hay en el Rack B?" |
| 2 | Backend | Gemini interpreta el texto y genera la intención de búsqueda. |
| 3 | DB | Se consulta PostgreSQL para obtener la cantidad exacta. |
| 4 | Agente | Gemini formatea el dato y responde de forma natural. |

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