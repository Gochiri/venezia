# Correcciones del CRM — errores reportados por el cliente (oct 2026)

Lista enviada por Stiward ("ERRORES DEL CRM") + 2 capturas (Fernando Cabrera, Paul). Este documento dice, por error: qué pasó, por qué, qué se cambia y **dónde se toca en GHL**. Todo lo del bot es configuración de UI (el CLI no llega a Conversation AI).

Artefactos nuevos en el repo:
- `docs/bot/prompt-v2.md` — prompt completo v2 (también en ClickUp T3).
- `docs/bot/kb-v2.md` + `docs/Base_Conocimiento_Bot_Venezia.pdf` regenerado — KB v2.

Orden recomendado de aplicación: **1 → KB → 2/3/5 (prompt) → handover → 4 (teléfono) → pruebas**.

---

## Error 1 · "Contestó que los gabinetes no son plywood" (Fernando Cabrera)

**Síntoma.** Cliente: "Son en playwood". Bot: "…paneles de MDF de alta calidad, no usamos plywood."

**Causa raíz.** La KB ya decía *"caja de ¾ de plywood"*. El bot lo inventó porque (a) el prompt no tenía ningún dato de construcción/materiales en *Key facts*, así que respondió de memoria, y (b) probablemente la KB corregida no está **asociada** al agente (T3 ya lo advertía: subirla no basta).

**Cambios.**
- Prompt v2: bloque nuevo *"Construction & materials"* + regla dura (nunca MDF, nunca "no usamos plywood", si no está en la lista → confirmar y derivar). Ejemplo 5 nuevo.
- KB v2: línea de construcción explícita en 2.1 + FAQ "¿Son de plywood? — Sí".

**Dónde en GHL.**
1. Settings → Conversation AI → agente **Venezia** → pestaña *Knowledge Base* → subir `Base_Conocimiento_Bot_Venezia.pdf` (v2) → **marcar la casilla para asociarla al agente** → quitar la v1.
2. Mismo agente → *Prompt* → reemplazar todo por el bloque de `prompt-v2.md`.

**Prueba.** Escribir por WhatsApp: "son de plywood?" → debe decir sí, caja de ¾ plywood. "son de MDF?" → debe decir que la caja es plywood (no confirmar MDF).

---

## Error 2 · "Pide repetidamente información de contacto que ya dieron" (Carolina)

**Causa raíz.** Tres cosas sumadas:
- El prompt v1 decía *"Always get their name and email, and confirm the best phone"*. En WhatsApp/SMS el teléfono ya se tiene → pedirlo es redundante y se siente insistente.
- **Human Handover → "Reactivate bot after: 8 hours"**: a las 8 h el bot despierta en una conversación que ya lleva una persona y vuelve a calificar desde cero.
- Conversation AI no siempre relee los campos guardados; había que decírselo explícitamente.

**Cambios.**
- Prompt v2: sección *"NEVER REPEAT A QUESTION"* — nunca pedir teléfono por WhatsApp/SMS; nombre y email máximo una vez; usar `{{contact.first_name}}` si existe; si ya preguntó y no contestaron, no insistir; con nombre + necesidad → parar y derivar.
- Handover: *Reactivate bot after* **8 h → 72 h** (ver T6).

**Dónde en GHL.**
1. Prompt (igual que error 1).
2. Agente Venezia → *Setup Your Actions* → **Human Handover** → en los 4 escenarios: *Reactivate bot after* = **72 Hours**.

**Prueba.** Conversación donde el cliente da nombre en el primer mensaje → el bot no debe volver a pedirlo. Por WhatsApp nunca debe pedir número.

---

## Error 3 · "Pregunta precio y desvía a dirección o visita"

**Causa raíz.** Estaba en el prompt v1, literal (Ejemplo 3): *"…un asesor te arma el estimado exacto. ¿Ya tienes medidas o coordinamos una visita?"* — el cliente lo percibe como evasiva. Además la guía 3 ("pregunta qué showroom queda más cerca") se disparaba sin contexto.

**Cambios.**
- Prompt v2: sección *"Price questions — answer directly, then hand off"*: en UNA línea explica por qué no cotiza por chat y que un asesor manda el estimado el mismo día; pide solo lo que falte (normalmente el nombre) y deriva. **Prohibido** responder precio con dirección/horario/visita. Ejemplo 3 reescrito. Guía 3 ahora solo pregunta showroom cuando hay visita o pickup.
- Handover: confirmar que el escenario **"Ready for salesperson"** (precio/cotización/comprar) está **activo** — en el handoff de T6 figuraba como pendiente de configurar por Oliver.

**Dónde en GHL.** Prompt + Human Handover (verificar que existen y están ON los 4 escenarios de T6).

**Prueba.** "¿cuánto cuesta una cocina white shaker?" → explicación directa + pide nombre → handover. No debe mencionar Medley/West Palm Beach.

---

## Error 4 · "Las llamadas no entran al CRM, caen al celular; en el CRM solo llega notificación de llamada perdida"

**No es del bot.** Es el enrutamiento del número de teléfono.

**Causa probable.** El número de GHL está configurado con *Forward to phone number* (el celular de Stiward). Entonces GHL registra la llamada pero **no hace sonar la app** a los usuarios del CRM; ellos solo ven "missed call".

**Primero, una pregunta al cliente (decide todo):** ¿los clientes llaman al **número de GHL** o al **celular de siempre**?
- Si llaman al celular de siempre → GHL no puede verlo. Opciones: (a) **portar** ese número a GHL (coincide con el pendiente "teléfono definitivo para A2P" del HANDOFF), o (b) publicar el número de GHL en web/Google/redes y dejar el celular solo como respaldo.
- Si llaman al número de GHL → aplicar lo de abajo.

**Cambios en GHL** (los nombres de menú varían un poco según versión):
1. Settings → **Phone Numbers** → el número de Venezia → *Edit / Call settings*:
   - *Inbound calls*: cambiar de **"Forward to number"** a **"Ring users"** (sonar en la app web + móvil) → elegir Stiward (+ Aura/Alex cuando estén).
   - *Ring timeout*: 20–30 s; *fallback*: forward al celular de Stiward si nadie contesta.
   - *Call recording*: ON (queda en la conversación del contacto).
   - *Voicemail*: grabar uno bilingüe.
2. Settings → **My Staff / Team** → cada usuario → *Call & Voicemail settings*: activar recibir llamadas en la app; poner su celular como forward.
3. Cada usuario instala la app **LeadConnector** (iOS/Android) e inicia sesión — sin la app no suena nada.
4. Settings → Business Profile / Phone → **Missed Call Text Back: ON**, texto bilingüe: "Hola, vimos tu llamada 🙂 ¿En qué te ayudamos? / Hi, we saw your call, how can we help?".

**Prueba.** Llamar al número de GHL desde un celular externo: debe sonar en la app de Stiward; colgar sin contestar → debe caer al celular y luego llegar el SMS automático.

---

## Error 5 · "Sigue siendo intenso con clientes ya registrados en el celular (no entraron por lead), les responde insistentemente y se molestan"

**Causa raíz.** El bot responde a **todo** contacto que escribe. Un cliente existente escribe por un pedido y el bot lo trata como lead nuevo: califica, pide datos, insiste. Y a las 8 h del handover vuelve a empezar.

**Cambios.**
- Prompt v2: sección *"Existing customers and post-sale — do NOT qualify"* (pedido, entrega, instalación, factura, garantía, reclamo → deriva de inmediato, sin preguntas) y regla de retirada (ocupado / no me interesa / ya compré / no me escribas → una línea y silencio). Ejemplos 6 y 7.
- Handover (T6): escenario nuevo **"Existing customer / post-sale"** + *Reactivate bot after* 72 h + frases extra en *Stop Bot* ("no me interesa", "ya compré", "no me escribas", "stop", "not interested").
- **Apagar el bot para la base existente** con la etiqueta nativa `stop bot` (ya documentada en T6 como interruptor manual):
  1. Contacts → filtro *Created date < 2026-07-15* (fecha de arranque del bot) **o** la lista de 647 de SP04 **o** contactos con oportunidad en *Proyectos Activos* → seleccionar todos → *Bulk actions* → **Add tag `stop bot`**.
  2. Workflow nuevo (Standard builder, no se puede por CLI porque el trigger es por etapa): **"AP-02 Stop bot clientes"** — Trigger: *Opportunity Stage Changed* → pipeline Ventas, etapa "Estimado enviado" (y cualquier etapa de *Proyectos Activos*) → Action: *Add Contact Tag* `stop bot`. Así todo el que ya es cliente deja de recibir al bot automáticamente.
  3. Capacitación al equipo: cualquier contacto que NO sea lead → ponerle `stop bot` a mano.
- **Diseño del workflow "AP-02 Stop bot clientes"** (Standard builder, 2 min):
  - Trigger 1: *Contact Tag Added* → tag `cliente-historico` (importación Clover) — y también `compro` si se usa.
  - Trigger 2: *Opportunity Status Changed* → status **Won** (cualquier pipeline). Alternativa: *Opportunity Stage Changed* → pipeline Proyectos Activos, cualquier etapa.
  - Acción 1: *Add Contact Tag* → `stop bot`.
  - Acción 2: *Conversation AI* → **Off** (si la versión de GHL muestra esa acción; si no, con el tag basta).
  - NO poner "Remove from all workflows": AP01 corre sobre Proyectos Activos y se cortaría.
- **Cómo ubicar la base importada** (para el tag masivo): Contacts → *Smart Lists* → filtro *Tags* = `cliente-historico`. Si no existe el tag, filtro *Date Added* = fecha de la importación (Contacts → *Bulk Actions* muestra el historial de importaciones con fecha y cantidad).
- ⚠️ **Conflicto con SP04 (reactivación de los 647):** el diseño original decía que cuando uno de esos contesta "entra a SP01 (calificación)" = el bot. Con la instrucción de Stiward eso cambia: la respuesta de un cliente existente debe ir a una **persona** (notificación interna + asignar usuario), no al bot. Revisar SP04 antes de lanzar la reactivación. Confirmar con Stiward si los 647 cuentan como "ya eran clientes".
- Opción B (más estricta, si el cliente la quiere): en el agente, *Bot settings* → "responder solo a contactos con etiqueta" `lead nuevo`, y que LS01 ponga esa etiqueta al crear el contacto. Cambia la lógica de entrada; proponer solo si con `stop bot` no alcanza.

**Prueba.** Con un contacto que tenga `stop bot`, escribir por WhatsApp → el bot no responde. Con un contacto sin etiqueta escribir "cuándo me entregan mi pedido" → el bot deriva sin calificar.

---

## Error 6 (probable) · Captura de Paul: "1 week after payment" para un **pickup** de RTA

No está en la lista del cliente; es el círculo verde en la captura de Paul. La KB dice *"tamaños estándar con retiro inmediato"* y *"~1 semana"* es **entrega + instalación**; el bot mezcló las dos cosas.

**Cambio.** Prompt v2 y KB v2 separan: stock estándar → retiro inmediato tras pago (vendedor confirma disponibilidad); entrega + instalación RTA ≈ 1 semana; a medida ≈ 15 días + 1 semana.

**⚠️ Confirmar con Stiward antes de dar por cerrado**: ¿el retiro inmediato aplica a todos los estilos estándar o solo a algunos?

---

## Checklist de aplicación

- [ ] KB v2 subida **y asociada** al agente; v1 retirada
- [ ] Prompt v2 pegado en el agente
- [ ] Human Handover: 4 escenarios ON + escenario "Existing customer / post-sale" + Reactivate 72 h
- [ ] Stop Bot: frases de retirada agregadas
- [ ] Tag `stop bot` masivo a la base existente
- [ ] Workflow "AP-02 Stop bot clientes" (Standard builder)
- [ ] Teléfono: respuesta del cliente (¿qué número llaman?) → enrutamiento "Ring users" + app instalada + Missed Call Text Back
- [ ] Confirmación de Stiward sobre retiro inmediato (error 6)
- [ ] Pruebas E2E (T7) con los 6 casos de arriba
