# Ecosistema IA — Propuestas Comerciales Automatizadas

Pipeline de automatización de extremo a extremo para generación y envío de propuestas comerciales personalizadas usando n8n, Claude (Anthropic), Airtable, Slack y Gmail.

---

## Stack tecnológico

| Herramienta | Rol |
|---|---|
| **n8n** | Orquestador principal (2 flujos separados) |
| **Airtable** | Base de datos: Leads, RAG de contexto, Log de operaciones |
| **Claude Sonnet 4.6** | Generación de propuestas con razonamiento RAG |
| **Slack** | Notificación HITL al equipo revisor |
| **Gmail** | Envío de la propuesta aprobada al cliente |

---

## Arquitectura del sistema

```
FLUJO 1: Trigger (Airtable) → RAG → Claude → Parse → HITL (Airtable + Slack)
FLUJO 2: Trigger (Checkbox Aprobado) → IF doble → Gmail → Log
```

---

## Cómo importar el flujo en n8n

1. Abrí n8n → **Settings → Import Workflow**
2. Seleccioná `flujo_propuestas_n8n.json`
3. Configurá las credenciales (Settings → Credentials):
   - **Airtable**: Personal Access Token
   - **Anthropic**: API Key
   - **Slack**: Bot Token (scope: `chat:write`)
   - **Gmail**: OAuth2
4. Reemplazá `YOUR_AIRTABLE_BASE_ID` por tu Base ID real en todos los nodos
5. Reemplazá `YOUR_BASE_LINK` en el mensaje de Slack por el enlace compartido de tu base
6. Activá ambos workflows (toggle en la esquina superior derecha)

---

## Estructura de Airtable

### Tabla 1: [LEADS] Solicitudes comerciales
Campos clave: `Nombre cliente`, `Empresa`, `Email cliente`, `Servicio solicitado`, `Presupuesto estimado`, `Estado` (Single select), `Aprobado` (Checkbox), `Borrador IA`, `Puntuación lead`, `Timestamp IA`

**Estados del campo Estado:**
- `Solicitud nueva` → activa el Flujo 1
- `Pendiente HITL` → el borrador está listo para revisar
- `Propuesta enviada` → aprobada y enviada al cliente
- `Error` → fallo en el procesamiento

### Tabla 2: [RAG] Base de conocimiento
Campos: `Cliente`, `Tema`, `Contenido RAG`, `Verificado` (Checkbox)

> ⚠️ Solo los registros con `Verificado = TRUE` son accesibles por Claude.

### Tabla 3: [LOG] Operaciones
Campos: `Fecha`, `Acción`, `Cliente`, `Estado`, `Detalle`

---

## Link a la base de Airtable (modo lectura)

> 🔗 https://airtable.com/YOUR_SHARED_LINK

---

## Test de estrés — 5 casos de prueba

| # | Caso | Resultado esperado |
|---|---|---|
| 1 | Solicitud completa con cliente en RAG | Propuesta generada → Slack → Gmail |
| 2 | Cliente sin registros en RAG | Claude usa 'A confirmar' → propuesta genérica |
| 3 | Presupuesto vacío | Claude infiere rango → propuesta igual se genera |
| 4 | API Claude sin respuesta (timeout) | Estado = Error → Slack #ops-errores |
| 5 | Checkbox aprobado sin pasar por HITL | IF doble verificación rechaza la ejecución |

---

## Screenshots de evidencia

Ver carpeta `/screenshots/`:

- `01_airtable_tabla_leads.png` — Tabla de leads con todos los campos
- `02_airtable_tabla_rag.png` — Base RAG con registros atómicos verificados
- `03_n8n_flujo1_completo.png` — Canvas del Flujo 1 con nodos nombrados
- `04_n8n_flujo2_hitl.png` — Canvas del Flujo 2 con IF y distribución
- `05_slack_notificacion_hitl.png` — Mensaje de aprobación en Slack
- `06_gmail_propuesta_enviada.png` — Email enviado al cliente
- `07_airtable_log_operaciones.png` — Log con las 5 ejecuciones de prueba

---

## Arquitectura completa (PDF)

Ver `docs/entrega_final_ecosistema_ia.pdf`

---

## Checklist de seguridad

- [x] El flujo tiene filtro anti-loops (Estado cambia a 'Procesando IA' en el primer nodo)
- [x] Las comparaciones de tipos son correctas (Boolean vs boolean en el IF HITL)
- [x] El prompt de IA usa variables dinámicas (no datos hardcodeados)
- [x] Las API Keys están ocultas en todos los screenshots
- [x] La base de Airtable está compartida en modo lectura (no edición)
