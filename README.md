# Entrega Final: Ecosistema de Automatización IA (B2B Leads)
**Estudiante:** Inés Huerta Morán  
**Repositorio Oficial:** https://github.com/InezMoran/Entrega-Final-Coder-AI-automation
**CASO DE USO:** 

---

## 1. Mapa de Arquitectura (20%)
El flujo está diseñado e implementado en Make e integra Airtable, OpenAI (GPT-4o-mini) y Gmail con validación Human-in-the-Loop (HITL).
- **Trigger:** Airtable (`Watch Records`) para la detección de nuevos registros en estado "Pendiente".
- **Orquestación:** Creación de registro borrador, procesamiento mediante prompt dinámico en OpenAI, estructuración limpia mediante `Parse JSON` y actualización en Airtable a "Procesado por IA".
- **Resiliencia:** Enrutamiento de errores con nodo alternativo en Airtable (`Airtable 8`) para capturar caídas de la API o límites de tokens sin detener el flujo.
- **Salida / HITL:** Notificación de revisión vía Gmail enviada al aprobador humano antes de cualquier envío al cliente final.

> 📄 **Diagramas Adjuntos en PDF:**
> - [Diagrama_de_Arquitectura_Redactado.pdf](./Diagrama_de_Arquitectura_Redactado.pdf) (Especificación técnica de nodos y conectores).
> - [Diagrama_Flujo_Tecnico_Draw_Graficado.pdf](./Diagrama_Flujo_Tecnico_Draw_Graficado.pdf) (Diagrama de flujo visual con simbología ISO/ANSI estilo Draw.io).

---

## 2. Manual Operativo de Datos y Esquemas JSON (20%)
### Esquema de Tablas en Airtable
1. **Tabla `Leads`:** Almacena la información de contacto e ingesta (`Nombre`, `Email`, `Empresa`, `Presupuesto Estimado`, `Necesidad del Cliente`).
2. **Tabla `Propuestas`:** Almacena la generación de la IA vinculada relacionalmente al cliente.
   - **Estados de Control:** `Pendiente` -> `Procesado por IA` -> `Aprobado por Humano` / `Error`.

### Esquema JSON de Transferencia (Payload de OpenAI)

El flujo fuerza la salida estructurada de la IA mediante el nodo `Parse JSON`:
{
"clasificacion": "VIP | Estándar",
"justificacion": "Análisis objetivo del presupuesto e impacto del cliente",
"propuesta_borrador": "Texto de la propuesta comercial adaptada a las necesidades expresadas"
}

---

## 3. Matriz de Optimización de Costos (20%)

| Tarea del Sistema | Modelo Evaluado | Modelo Seleccionado | Justificación Técnica y Económica |
| :--- | :--- | :--- | :--- |
| **Clasificación y Redacción Borrador** | GPT-4o vs Claude 3.5 Sonnet | **GPT-4o-mini** | **Costo/Rendimiento:** GPT-4o-mini ofrece un costo por token 95% menor que GPT-4o, siendo óptimo para extracción y estructuración en JSON. Para el caso evaluado que es un caso sencillo, el mini nos parecio el mas adecuado que los modelos mas potentes analizados |
| **Lectura Densa / Documentos Largos** | Claude 3.5 Sonnet | *N/A (No Requerido)* | Reservado exclusivamente para análisis de contratos extensos (+50 págs), en este caso no aplica. |
| **Procesamiento Masivo Batch** | OpenAI Batch API | *N/A (Proceso On-demand)* | No aplica por requerir ejecución inmediata trigger-based ante cada lead. Ademas se necesita velocidad de respuesta comercial para no dejar enfriar el lead ni bien llega. |

### Estrategias de Ahorro de Tokens:
- **Límite Estricto:** Control mediante `max_tokens` en el nodo de OpenAI para evitar respuestas sobreextensas. Se coloco 500, para que actue como una barrera de seguridad financiera. Evita que la IA genere respuestas sobreextensas por error o alucinaciones, garantizando que cada propuesta mantenga un costo por ejecución fijo, bajo y controlado.
- **Minimización de Contexto:** Se envían únicamente las 4 variables críticas (`Nombre`, `Empresa`, `Presupuesto`, `Necesidad`) en el prompt del sistema.

---

## 4. Documentación de Seguridad, Resiliencia y Human-In-The-Loop (20%)

1. **Minimización de Datos:** Se omiten datos sensibles PII no relevantes para la generación comercial al comunicarse con OpenAI.
2. **Rutas de Error Handling (Resiliencia):** 
   - Se integró un nodo de captura de excepciones en Make (`Airtable 8`). Si la API de OpenAI falla por timeout o cuota, el flujo deriva a esta rama, registra el log del error y cambia el estado a "Error", evitando bucles infinitos o la caída del escenario.
3. **Human-in-the-loop (HITL):** 
   - El sistema no despacha correos automáticos al cliente final. Notifica vía Gmail al operador humano para que revise la propuesta en Airtable y apruebe manualmente el cambio de estado.

---

## 5. Dashboard de Control, KPIs y Enlaces Obligatorios (20%)

- **🔗 Enlace a la Base de Datos (Modo Lectura):** [Airtable Base - Modo Lectura](https://airtable.com/invite/l?inviteId=invinbP1r1O0E6yqd&inviteToken=cef39db3fab929d5a22de814d7fce9f6aa67f72bc67ef08a78de584a8dc6f237&utm_medium=email&utm_source=product_team&utm_content=transactional-alerts)
- **📊 Enlace al Dashboard de Control (Shared View KPIs):** [Airtable Dashboard KPIs - Shared View](https://airtable.com/invite/l?inviteId=invinbP1r1O0E6yqd&inviteToken=cef39db3fab929d5a22de814d7fce9f6aa67f72bc67ef08a78de584a8dc6f237&utm_medium=email&utm_source=product_team&utm_content=transactional-alerts)

### KPIs Monitoreados en el Dashboard:
- **Distribución de Leads:** Volumen de Leads clasificados como `VIP` vs `Estándar`.
- **Métrica de Resiliencia:** Conteo de registros capturados en estado `Error`.
- **Pipeline de Aprobación:** Volumen de propuestas en `Procesado por IA` pendientes de validación humana.

### Video demo:
- **Demostracion del funcionamiento:** https://1drv.ms/v/c/696b63e1415f83d8/IQDfKVshX9PaS4uIBSRc3jIlAWFywAZGTysqqz_02s_A2Nk?e=5RbJoF


  
