# 🤖 Ecosistema de Automatización IA Autónomo

**Proyecto Final — Curso IA Automation | Coderhouse 2025**

Pipeline de generación de contenido LinkedIn con base de conocimientos RAG, validación humana (HITL) y control de calidad automatizado.

---

## 🏗️ Stack tecnológico

| Componente | Herramienta | Rol |
|---|---|---|
| Orquestador | n8n (cloud) | Flujo principal y lógica de control |
| Base de datos | Airtable | Memoria del sistema + RAG |
| Motor de IA | Groq — LLaMA 3.1 8B Instant | Generación de contenido |
| Canal de salida | Gmail SMTP | Notificación HITL con aprobación |

---

## 📋 Caso de uso

El sistema automatiza la generación de **posts para LinkedIn** sobre operaciones y logística:

1. El editor carga una **Idea Semilla** en Airtable y cambia el Estado a `Generando`
2. n8n detecta el cambio y consulta la **base RAG** con directrices de marca
3. **Groq LLaMA 3.1** genera el post usando el contexto privado
4. El borrador se guarda en Airtable y se envía al editor por **Gmail**
5. El editor **aprueba o rechaza** desde el email con un solo clic
6. El sistema actualiza el Estado en Airtable según la decisión

---

## 🗂️ Estructura del repositorio

```
ecosistema-ia-automation/
├── README.md                              # Este archivo
├── Proyecto_Final_Coder.json              # Workflow exportado de n8n (10 nodos)
├── EntregaFinal_Ecosistema_IA_v2.pdf      # Documentación completa (5 criterios)
└── screenshots/
    ├── 01_flujo_completo_n8n.png          # Workflow activo con todos los nodos
    ├── 02_airtable_contenido.png          # Tabla de comandos con todos los estados
    ├── 03_airtable_rag.png                # Base de conocimientos RAG (6 registros)
    └── 04_email_hitl.png                  # Email HITL con botones Aprobar/Rechazar
```

---

## 🔗 Links importantes

| Recurso | Link |
|---|---|
| 📊 Dashboard de Control (Airtable) | [Ver Dashboard](https://airtable.com/appfj3aaPcP99qmEB/shrRw0NsPSaKQTZ2O) |
| 🗄️ Base de datos (modo lectura) | [Ver Base de datos](https://airtable.com/appfj3aaPcP99qmEB/shrbqn70GkMD7IOcS) |
| 📄 Documentación PDF | Ver `EntregaFinal_Ecosistema_IA_v2.pdf` en este repo |

---

## 🧠 Arquitectura del pipeline

```
Airtable Trigger (Estado='Generando' AND Procesado=FALSE)
        ↓
Buscar directrices RAG (6 registros atómicos)
        ↓
Groq LLaMA 3.1 — Basic LLM Chain ──→ [Error] → Error Handler → Airtable (Estado=Error)
        ↓ [Success]
Edit Fields (Execute Once) — consolida output LLM en 1 item limpio
        ↓
Airtable — Guardar borrador (Estado=En revisión)
        ↓
Update record — Procesado = true (evita reprocesamiento)
        ↓
Gmail SMTP — Send & Wait (HITL) ← PUNTO DE PAUSA HUMANA
        ↓
IF: data.approved = true
        ├── true  → Airtable (Estado=Publicado)
        └── false → Airtable (Estado=Rechazado)
```

---

## 🗄️ Estructura de datos

### Tabla: Contenido (centro de comando)

| Campo | Tipo | Función |
|---|---|---|
| Idea Semilla | Single line | Input del editor — variable dinámica del prompt |
| Estado | Single select | Semáforo del pipeline — 6 estados posibles |
| Borrador | Long text | Output del LLM generado automáticamente |
| Procesado | Checkbox | Evita reprocesamiento — el flujo lo marca true tras generar |
| Feedback | Long text | Notas del editor para regeneración |
| URL Publicado | URL | Se completa tras publicación exitosa |
| Error Log | Long text | Mensaje técnico de error de API |
| Última modificación | Last modified | Trigger Field del Airtable Trigger |

### Tabla: Directrices RAG (base de conocimientos)

| Registro | Tipo | Función |
|---|---|---|
| Voz de marca | Estilo | Tono del agente |
| Estructura post | Formato | Hook + bullets + CTA + hashtags |
| Temas permitidos | Contenido | Perímetro temático |
| Temas prohibidos | Restricción | Filtro negativo |
| Hooks efectivos | Referencia | Ejemplos calibradores |
| Disclaimers compliance | Compliance | Reglas de citado |

---

## 💰 Optimización de costos

| Modelo | Costo/post | Costo 500 posts/mes | Costo anual |
|---|---|---|---|
| GPT-4o mini | ~$0.0003 | ~$0.15 | ~$1.80 |
| Claude Haiku 4.5 | ~$0.002 | ~$1.00 | ~$12.00 |
| **Groq LLaMA 3.1 (elegido)** | **$0.00** | **$0.00** | **$0.00** |

**Justificación:** Para generación de contenido estructurado de longitud media, LLaMA 3.1 8B ofrece calidad suficiente a costo cero. GPT-4o y Claude se reservan para tareas que requieren razonamiento complejo o análisis de documentos densos.

---

## 🔒 Seguridad y resiliencia

- ✅ **Credenciales encriptadas** en n8n credentials manager — nunca hardcodeadas
- ✅ **Error handler** en nodo crítico (Groq) con 3 reintentos automáticos
- ✅ **HITL obligatorio** — ningún post se publica sin aprobación humana explícita
- ✅ **Filtro anti-bucle 1** — trigger solo se activa con Estado='Generando'
- ✅ **Filtro anti-bucle 2** — campo Procesado=FALSE evita reprocesamiento
- ✅ **Minimización de datos** — no se almacenan datos personales de terceros
- ✅ **Variables 100% dinámicas** — sin datos hardcodeados en el workflow

---

## 📊 Estados del pipeline

| Estado | Significado | Acción siguiente |
|---|---|---|
| 🔘 Pendiente | Idea cargada, no procesada | Editor cambia a Generando |
| 🔵 Generando | Trigger activo, flujo corriendo | Automático |
| 🟡 En revisión | Borrador listo, esperando HITL | Editor aprueba o rechaza |
| 🟢 Publicado | Post aprobado | Proceso completo |
| 🔴 Rechazado | Rechazado por el editor | Revisar feedback |
| 🟠 Error | Fallo de API | Reintentar manualmente |

---

## 🚀 Cómo usar este workflow

### Requisitos previos

- Cuenta en [n8n.io](https://n8n.io) (cloud)
- Cuenta en [Airtable](https://airtable.com)
- Cuenta en [Groq](https://console.groq.com) (gratuita)
- Cuenta de Gmail con App Password configurada

### Instalación

1. Importá `Proyecto_Final_Coder.json` en n8n
2. Configurá las credenciales:
   - Airtable Personal Access Token
   - Groq API Key (gratuita en console.groq.com)
   - Gmail SMTP con App Password de Google
3. En Airtable creá dos tablas:
   - **Contenido** con los campos: Idea Semilla, Estado, Borrador, Procesado, Feedback, URL Publicado, Error Log, Ultima modificacion
   - **Directrices RAG** con los campos: Tema, Contenido, Tipo
4. Cargá los 6 registros RAG en la tabla Directrices RAG
5. Reemplazá los IDs de Airtable en los nodos (Base ID + Table ID)
6. Publicá y activá el workflow en n8n
7. Cargá una fila nueva en Airtable con Estado = `Generando` y Procesado sin tildar para probar

---

## 📁 Documentación completa

Ver `EntregaFinal_Ecosistema_IA_v2.pdf` para los 5 criterios de evaluación:

- Criterio 1: Mapa de arquitectura (10 nodos documentados)
- Criterio 2: Estructuras de datos documentadas
- Criterio 3: Optimización de costos
- Criterio 4: Seguridad y resiliencia
- Criterio 5: Dashboard de control

---

*Desarrollado como proyecto final del curso IA Automation — Coderhouse 2025*
