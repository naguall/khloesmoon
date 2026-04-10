# Setup de Firebase para Khloe's Moon

Khloe's Moon comparte el proyecto Firebase **`astroconnect-463b3`** con tu app Moon Tracker / AstroConnect, pero usa colecciones separadas con el prefijo `khloesmoon_` para que **las dos apps nunca se afecten mutuamente**.

---

## 1. Aislamiento entre apps (ya implementado en el código)

| Recurso                | AstroConnect / Moon Tracker        | Khloe's Moon                       |
|------------------------|------------------------------------|------------------------------------|
| Firebase App           | default app (`[DEFAULT]`)          | named app `khloesmoon`             |
| Perfil usuaria         | `users/{uid}`                      | `khloesmoon_users/{uid}`           |
| Chats 1:1              | `chats/{id}/messages`              | `khloesmoon_chats/{id}/messages`   |
| Comunidades            | `communities/*`                    | —                                  |
| Feed                   | `feed/*`                           | —                                  |
| Notificaciones         | `notifications/*`                  | —                                  |
| localStorage prefix    | `astroconnect_*`                   | `khloesmoon_*`                     |

**Auth (login Google) es compartida**: una misma cuenta de Google sirve para las dos apps, pero cada app lee su propia colección. Si inicias sesión en Khloe's Moon también quedarás logueada en Moon Tracker (y viceversa) — esto es normal porque ambas apps viven en el mismo proyecto Firebase. Los datos son totalmente independientes.

---

## 2. Pasos una sola vez en la consola de Firebase

### Paso A — Publicar reglas de Firestore

1. Abre https://console.firebase.google.com/project/astroconnect-463b3/firestore/rules
2. Reemplaza las reglas actuales con el contenido del archivo **`firestore.rules`** (está en esta misma carpeta `mi-app/`)
3. Click **Publicar**

⚠️ Si ya tenías reglas personalizadas para AstroConnect que no están en mi archivo, copia SOLO los bloques `match /khloesmoon_users` y `match /khloesmoon_chats` al final de tu bloque `match /databases/{database}/documents {}`. Así preservas lo tuyo.

### Paso B — Autorizar dominio para Google Sign-in

1. Abre https://console.firebase.google.com/project/astroconnect-463b3/authentication/settings
2. En **Dominios autorizados** verifica que estén:
   - `localhost` (para pruebas locales)
   - `naguall.github.io` (para GitHub Pages)
3. Si no aparece, click **Añadir dominio** → escribe `naguall.github.io` → Guardar

Si ya tienes AstroConnect funcionando en GitHub Pages, este paso probablemente ya está hecho.

### Paso C — Activar el proveedor Google (si no está)

1. https://console.firebase.google.com/project/astroconnect-463b3/authentication/providers
2. Google → estado debe ser **Habilitado**. Si está deshabilitado, actívalo y guarda.

---

## 3. Cómo funciona el chat entre amigas

1. **Tú y tu amiga** abren la app en sus teléfonos → entran a Ajustes → "Conectar con amigas"
2. Las dos hacen click en **Entrar con Google**
3. Una copia su código (`XXXX-XXXX`) y se lo manda a la otra (por WhatsApp, SMS, etc.)
4. La otra lo pega en "Añadir amiga" con su nombre → +
5. El sistema la busca en Firestore por `shareCode` y guarda su `uid`
6. Al tocar 💬 se abre el chat en tiempo real
7. Cuando llega un mensaje nuevo y no estás en el chat:
   - Se incrementa el contador rojo en la lista de amigas
   - Dispara una notificación del navegador (si diste permiso)

---

## 4. Qué pasa si una amiga NO se loguea con Google

- Puedes agregarla igual, pero quedará en modo **local** (sin `uid`).
- El chat caerá al fallback Web Share (mandará el mensaje por WhatsApp/SMS).
- Cuando ella se loguee más tarde, la próxima vez que tú abras la app, el sistema hace un **backfill automático** que busca su código en Firestore y llena el `uid`, activando chat en tiempo real sin que tengas que agregarla de nuevo.

---

## 5. Cómo verificar que no chocan las dos apps

Prueba rápida:

1. Abre AstroConnect/Moon Tracker en una pestaña → haz algo (un post, un chat con alguien)
2. Abre Khloe's Moon en otra pestaña → entra con Google, agrega amiga, chatea
3. Vuelve a Moon Tracker → recarga → los posts, chats y datos originales **siguen intactos**
4. En la consola de Firebase → Firestore → verás colecciones separadas:
   - `users`, `chats`, `feed`, `communities` (Moon Tracker)
   - `khloesmoon_users`, `khloesmoon_chats` (Khloe's Moon)

---

## 6. Costos

Como compartes el proyecto, todo suma al mismo plan (Spark/gratis o el que tengas). Khloe's Moon es muy ligera en escrituras:
- 1 perfil doc por usuaria (se actualiza al login)
- 1 doc chat por par de amigas
- N mensajes (1 escritura cada uno)

Muy lejos de los límites del plan gratuito (50K lecturas / 20K escrituras por día).
