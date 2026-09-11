# Agente IA para Librería — Checkpoint 1

## Construcción del Agente Base y Motor de Razonamiento

Este repositorio contiene el **Checkpoint 1** del proyecto integrador desarrollado con **n8n**.

El proyecto consiste en la construcción progresiva de un **agente de inteligencia artificial para una librería**, capaz de interpretar consultas de usuarios y utilizar herramientas externas para consultar información del catálogo.

Este checkpoint constituye la primera versión del proyecto, que será ampliada posteriormente con nuevas funcionalidades como memoria, integraciones, RAG, voz y arquitectura multi-agente.

---

## Objetivo

Construir y validar un agente de IA utilizando el nodo **AI Agent de n8n en modo Tools Agent**, incorporando:

* Un disparador de entrada.
* Un modelo de lenguaje.
* Un System Message profesional.
* Una herramienta externa para consultar el catálogo.
* Un límite de iteraciones para evitar loops.
* Un mecanismo de observabilidad y notificación.

El objetivo principal es demostrar que el agente puede **decidir autónomamente cuándo utilizar una herramienta** en función de la consulta del usuario.

---

## Arquitectura

La estructura general del workflow es:

```text
┌─────────────────┐
│   Chat Trigger  │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────┐
│          AI Agent           │
│        Tools Agent          │
│                             │
│  Max Iterations: 5          │
└───────┬──────────────┬──────┘
        │              │
        │              ▼
        │      ┌─────────────────┐
        │      │ Google Sheets   │
        │      │      Tool       │
        │      └─────────────────┘
        │
        ▼
┌─────────────────────────────┐
│     Observabilidad          │
│       Slack / Gmail         │
└─────────────────────────────┘
```

La herramienta de Google Sheets está conectada directamente al **AI Agent como Tool**, permitiendo que el modelo determine cuándo necesita consultar el catálogo.

---

## AI Agent

El nodo principal del workflow es un:

**AI Agent → Tools Agent**

El agente funciona como asistente virtual especializado en la librería.

### Responsabilidades

El agente puede:

* Consultar información de libros.
* Consultar disponibilidad.
* Consultar precios.
* Consultar stock.
* Buscar libros por autor.
* Buscar libros por género.
* Consultar información específica del catálogo.
* Informar cuando un libro no se encuentra en el catálogo.

El agente no debe inventar información sobre libros, precios o disponibilidad.

---

## System Message

El System Message está estructurado de manera modular para delimitar claramente el comportamiento del agente.

La estructura utilizada es:

```text
ROL
ÁMBITO
OBJETIVO
REGLAS
USO DE HERRAMIENTAS
ESCALAMIENTO
SEGURIDAD
```

Una de las reglas principales es que el agente debe consultar el catálogo antes de proporcionar información concreta sobre:

* Existencia de un libro.
* Precio.
* Stock.
* Disponibilidad.
* Autor.
* Género.
* Editorial.
* Listados del catálogo.

Si la herramienta no devuelve resultados, el agente debe informar que el libro o información solicitada no fue encontrada en el catálogo.

No debe inventar información para completar una respuesta.

---

## 🛠️ Herramienta — Google Sheets

El catálogo de la librería se encuentra representado mediante una hoja de **Google Sheets**.

La estructura utilizada es:

| Campo     | Descripción             |
| --------- | ----------------------- |
| ISBN      | Identificador del libro |
| Título    | Nombre del libro        |
| Autor     | Autor de la obra        |
| Género    | Género literario        |
| Editorial | Editorial               |
| Precio    | Precio del libro        |
| Stock     | Cantidad disponible     |

### Ejemplo

| ISBN        | Título                             | Autor             | Género          | Editorial     | Precio | Stock |
| ----------- | ---------------------------------- | ----------------- | --------------- | ------------- | -----: | ----: |
| LIB-TOL-001 | El Hobbit                          | J.R.R. Tolkien    | Fantasía        | HarperCollins |  22000 |    12 |
| LIB-HP-001  | Harry Potter y la piedra filosofal | J.K. Rowling      | Fantasía        | Bloomsbury    |  24000 |    10 |
| LIB-DUN-001 | Dune                               | Frank Herbert     | Ciencia Ficción | Ace Books     |  26000 |     9 |
| LIB-SLA-001 | El camino de los reyes             | Brandon Sanderson | Fantasía épica  | Gollancz      |  32000 |     5 |
| LIB-LOV-001 | En las montañas de la locura       | H.P. Lovecraft    | Terror cósmico  | Del Rey       |  18000 |     7 |

---

## 📚 Catálogo de prueba

Para las pruebas del agente se utiliza un catálogo de ejemplo compuesto por obras pertenecientes a diferentes autores y géneros.

### Tolkien

* El Hobbit
* El Señor de los Anillos
* El Silmarillion
* Los Hijos de Húrin
* La caída de Númenor

### Harry Potter

* Harry Potter y la piedra filosofal
* Harry Potter y la cámara secreta
* Harry Potter y el prisionero de Azkaban
* Harry Potter y el cáliz de fuego
* Harry Potter y la Orden del Fénix
* Harry Potter y el misterio del príncipe
* Harry Potter y las reliquias de la muerte

### Dune

* Dune
* El Mesías de Dune
* Hijos de Dune
* Dios Emperador de Dune
* Herejes de Dune
* Casa Capitular: Dune

### The Stormlight Archive

* El camino de los reyes
* Palabras radiantes
* Juramentada
* El ritmo de la guerra
* El hombre iluminado

### H.P. Lovecraft

* La llamada de Cthulhu
* En las montañas de la locura
* El color que cayó del cielo
* El horror de Dunwich
* La sombra sobre Innsmouth

### Literatura clásica

El catálogo también contiene obras clásicas de distintos autores, géneros y editoriales.

---

## Guardrail de iteraciones

El AI Agent tiene configurado un límite máximo de:

```text
Max Iterations = 5
```

Este límite permite evitar ciclos de ejecución excesivos y controlar el consumo de recursos del workflow.

También funciona como una medida de seguridad ante comportamientos inesperados del agente.

---

## Observabilidad

El workflow incluye un mecanismo de observabilidad mediante un nodo de notificación.

Su finalidad es permitir la supervisión humana de las ejecuciones del agente.

El reporte puede contener información como:

* Consulta realizada.
* Respuesta generada.
* Resultado de la herramienta.
* Estado de ejecución.
* Información relevante de la operación.

La observabilidad no expone el razonamiento interno privado del modelo. Se centra en el resultado y en los eventos relevantes de ejecución.

---

## Pruebas

El workflow debe probarse desde n8n mediante:

**Execute Workflow / Test Step**

### Prueba 1 — Consulta de disponibilidad

```text
¿Tienen El Hobbit?
```

El agente debe consultar Google Sheets y determinar el stock.

---

### Prueba 2 — Consulta de precio

```text
¿Cuánto cuesta Dune?
```

El agente debe consultar el catálogo antes de informar el precio.

---

### Prueba 3 — Libro sin stock

```text
¿Está disponible Harry Potter y el misterio del príncipe?
```

Si el registro tiene:

```text
Stock = 0
```

el agente debe indicar que el libro **existe en el catálogo pero actualmente no está disponible**.

---

### Prueba 4 — Consulta por autor

```text
¿Qué libros de Tolkien tienen?
```

El agente debe utilizar la herramienta para consultar los registros correspondientes.

---

### Prueba 5 — Consulta por género

```text
¿Qué libros de ciencia ficción tienen?
```

El agente debe consultar el catálogo y devolver los resultados encontrados.

---

### Prueba 6 — Libro inexistente

```text
¿Tienen Juego de Tronos?
```

Si no existe en el catálogo, el agente debe indicar que **no se encontró en el catálogo actual**.

No debe inventar precio, stock ni disponibilidad.

---

## Importación

Para utilizar el workflow:

1. Abrir n8n.
2. Crear o seleccionar un proyecto.
3. Importar el archivo:

```text
checkpoint1_nombre_apellido.json
```

4. Configurar las credenciales correspondientes.
5. Configurar la conexión con Google Sheets.
6. Configurar las credenciales del modelo de lenguaje.
7. Configurar Slack o Gmail para la observabilidad.
8. Ejecutar las pruebas.

---

## Credenciales

El archivo JSON exportado **no debe contener credenciales sensibles**.

Después de importar el workflow será necesario configurar las credenciales desde n8n para:

* Modelo de lenguaje.
* Google Sheets.
* Slack o Gmail, según corresponda.

No se incluyen API Keys, contraseñas ni tokens en este repositorio.

---

## Estructura del repositorio

```text
/
├── README.md
└── checkpoint1_nombre_apellido.json
```

---

## Evolución del proyecto

Este workflow constituye la **primera versión del proyecto integrador**.

La arquitectura está diseñada para continuar evolucionando sin reconstruir el proyecto desde cero.

### Roadmap

```text
M1 · Agente Base
    │
    ├── Trigger
    ├── AI Agent
    ├── System Prompt
    ├── Tool
    └── Observabilidad
          │
          ▼
M2 · Multi-agente
          │
          ▼
M3 · Memoria y Contexto
          │
          ▼
M4 · Integraciones
          │
          ▼
M5 · RAG / Base Documental
          │
          ▼
M6 · Voz
          │
          ▼
...
          │
          ▼
M11 · Proyecto Final Integrador
```

La regla p
