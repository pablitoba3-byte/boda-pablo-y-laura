# Nuestra boda — versión Firebase

Misma app, mismo diseño, mismas 8 secciones (resumen, tareas, calendario,
presupuesto, invitados, proveedores, ideas, documentos). Ya no depende de
Claude: usa **Firebase Authentication** (acceso) y **Cloud Firestore**
(datos en tiempo real), lista para **Firebase Hosting**.

## 1. Qué ha cambiado

- Eliminado todo `window.claude.use(...)` y la lógica de capacidades de Claude.
- `state.db` ahora es `firebase.firestore()` en vez del almacenamiento de Claude.
  La forma de llamarlo (`collection().doc().add/update/delete/onSnapshot`) es
  prácticamente idéntica a como ya estaba escrito, así que **todas las
  funciones de guardado, lectura y las 8 secciones se quedan tal cual** —
  no se ha tocado `SCHEMAS`, ni el renderizado, ni el diseño.
- Añadida una pantalla de inicio de sesión (email + contraseña) que bloquea
  la app hasta que alguien se autentica. Sin ella, nadie ve ni toca nada.
- Añadido un botón "Cerrar sesión" junto al título.
- `dbErrorMsg` actualizado a los códigos de error reales de Firestore.
- Cargados los SDK de Firebase (versión compat) justo antes del script
  propio, vía CDN de Google (`gstatic.com`).

No se ha tocado ningún otro comportamiento ni estilo.

## 2. Qué activar/configurar en la Consola de Firebase

Ve a [console.firebase.google.com](https://console.firebase.google.com) y:

1. **Crear proyecto** (botón "Add project"). No hace falta tarjeta de crédito
   para lo que necesitamos (plan Spark/gratuito).
2. **Authentication** → pestaña "Sign-in method" → habilita **"Email/Password"**.
3. **Authentication** → pestaña "Users" → **"Add user"** → crea dos cuentas a
   mano, una para ti y otra para Laura (email + contraseña que elijáis). No
   hay registro público en la app: solo estas dos cuentas existirán, así que
   solo vosotros dos podréis entrar.
4. **Firestore Database** → "Create database" → modo producción → elige una
   región (p. ej. `eur3` si estáis en España).
5. **Firestore Database** → pestaña "Rules" → pega el contenido de
   `firestore.rules` (incluido en este proyecto) y publica.
6. **Project settings** (el engranaje) → baja a "Your apps" → icono `</>` para
   añadir una app web → ponle un nombre → **no** marques Firebase Hosting en
   este paso (lo configuramos por CLI) → te dará un bloque `firebaseConfig`.

## 3. Dónde introducir tu configuración

Abre `public/index.html`, busca cerca del principio del `<script>` el bloque:

```js
var firebaseConfig = {
  apiKey: "TU_API_KEY",
  authDomain: "TU_PROYECTO.firebaseapp.com",
  projectId: "TU_PROYECTO",
  storageBucket: "TU_PROYECTO.appspot.com",
  messagingSenderId: "TU_SENDER_ID",
  appId: "TU_APP_ID"
};
```

Sustitúyelo por el bloque exacto que te dio Firebase en el paso 6 anterior.
El `apiKey` no es secreto — la seguridad real la dan las reglas de Firestore
y el login, no ocultar esta clave, así que no pasa nada porque quede visible
en el HTML.

## 4. Desplegar con Firebase Hosting

Desde una terminal, dentro de esta carpeta (`wedding-firebase-project/`):

```bash
# Una sola vez en tu ordenador:
npm install -g firebase-tools
firebase login

# Vincular esta carpeta a tu proyecto:
firebase init hosting
#  - "Use an existing project" → selecciona tu proyecto
#  - "What do you want to use as your public directory?" → public
#  - "Configure as a single-page app?" → No
#  - "Set up automatic builds with GitHub?" → No
#  - Si te pregunta si quieres sobrescribir public/index.html → NO
#    (ya tienes el tuyo con la app dentro)

# Publicar:
firebase deploy --only hosting
```

Al terminar te dará una URL tipo `https://TU-PROYECTO.web.app` — esa es la
web ya online, solo accesible con las 2 cuentas que creaste en el paso 3
de la Consola.

Cada vez que quieras publicar un cambio futuro del archivo, repites solo
`firebase deploy --only hosting`.

