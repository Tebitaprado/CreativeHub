# 🚀 CreativeHub — PWA Marketplace de Talento Digital

Plataforma para conectar **Community Managers y Editores** con empresas.
MVP funcional, mobile-first, instalable como PWA y escalable a APK.

---

## 📁 Estructura del Proyecto

```
creativehub/
├── index.html          ← App completa (single-page application)
├── manifest.json       ← Configuración PWA
├── sw.js               ← Service Worker (caché offline + push)
├── favicon.ico         ← Favicon del sitio
└── icons/
    ├── icon-72.png
    ├── icon-96.png
    ├── icon-128.png
    ├── icon-144.png
    ├── icon-152.png
    ├── icon-192.png
    ├── icon-384.png
    └── icon-512.png
```

---

## ⚡ Deploy Rápido (5 minutos)

### Opción 1 — Netlify (Recomendado, GRATIS)
1. Entrá a [netlify.com](https://netlify.com) y creá una cuenta gratuita
2. Arrastrá la carpeta `creativehub/` al panel de Netlify
3. ¡Listo! Tu URL será `creativehub-xxxx.netlify.app`
4. Para dominio propio: Dominio → Add custom domain

### Opción 2 — Vercel (GRATIS)
```bash
npm install -g vercel
cd creativehub/
vercel --prod
```

### Opción 3 — GitHub Pages (GRATIS)
1. Subí todos los archivos a un repositorio GitHub
2. Settings → Pages → Source: main branch / root
3. Tu URL: `tuusuario.github.io/creativehub`

### Opción 4 — Servidor propio (Apache/Nginx)
Copiá los archivos a `/var/www/html/creativehub/` y configurá:

**Nginx:**
```nginx
server {
    listen 80;
    server_name tudominio.com;
    root /var/www/html/creativehub;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Headers para PWA
    add_header Cache-Control "public, max-age=31536000" ~\.(?:ico|png|json)$;
    add_header Service-Worker-Allowed "/";
}
```

> ⚠️ **IMPORTANTE:** El Service Worker requiere **HTTPS**. En desarrollo local usá `localhost` o un servidor con SSL.

---

## 📱 Instalar como PWA

### En Android (Chrome):
1. Abrí la app en Chrome
2. Menú (⋮) → "Añadir a pantalla de inicio"
3. Confirmá → La app aparece como ícono nativo

### En iOS (Safari):
1. Abrí la app en Safari
2. Botón compartir → "Añadir a pantalla de inicio"
3. La app funciona en modo standalone sin barra del navegador

---

## 📦 Escalar a APK (Android)

### Método 1 — PWABuilder (Recomendado, sin código)
1. Desplegá tu PWA con HTTPS
2. Entrá a [pwabuilder.com](https://www.pwabuilder.com)
3. Pegá la URL de tu PWA → Analizar
4. Seleccioná "Android" → Generar APK
5. Descargá el `.apk` o bundle `.aab` para Google Play

### Método 2 — Bubblewrap (Google, más control)
```bash
# Requiere Node.js 18+ y Android Studio
npm install -g @bubblewrap/cli

# Inicializar proyecto TWA (Trusted Web Activity)
bubblewrap init --manifest https://tu-url.com/manifest.json

# Compilar APK
bubblewrap build

# Archivo generado: ./app-release-signed.apk
```

### Método 3 — Capacitor (escalar a iOS también)
```bash
npm install @capacitor/core @capacitor/cli @capacitor/android

# Inicializar
npx cap init CreativeHub com.creativehub.app

# Agregar Android
npx cap add android

# Copiar archivos web
npx cap copy android

# Abrir en Android Studio
npx cap open android

# En Android Studio: Build → Generate Signed Bundle/APK
```

### Método 4 — Ionic AppFlow (CI/CD automático)
Conectá tu repo a [appflow.ionicframework.com](https://appflow.ionicframework.com) para builds automáticos.

---

## 🔥 Migrar a Firebase (Backend Real)

El MVP usa `localStorage`. Para producción con múltiples usuarios:

### 1. Setup Firebase
```bash
npm install firebase
```

### 2. Reemplazar las funciones de DB
```javascript
// En lugar de localStorage, usar Firestore:
import { initializeApp } from 'firebase/app';
import { getFirestore, collection, addDoc, getDocs, query, where } from 'firebase/firestore';
import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword } from 'firebase/auth';

const firebaseConfig = {
  apiKey: "TU_API_KEY",
  authDomain: "tu-proyecto.firebaseapp.com",
  projectId: "tu-proyecto",
  // ...
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
const auth = getAuth(app);

// Registrar usuario:
async function registerUser(email, password, userData) {
  const credential = await createUserWithEmailAndPassword(auth, email, password);
  await addDoc(collection(db, 'users'), {
    uid: credential.user.uid,
    ...userData
  });
}

// Obtener perfiles:
async function getPerfiles() {
  const snap = await getDocs(collection(db, 'perfiles'));
  return snap.docs.map(d => ({ id: d.id, ...d.data() }));
}
```

### 3. Reglas de Firestore
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth.uid == userId;
    }
    match /perfiles/{perfilId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null;
    }
    match /ofertas/{ofertaId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null;
      allow update: if resource.data.perfil_id == request.auth.uid;
    }
  }
}
```

---

## 🗺️ Roadmap de Funcionalidades Futuras

| Feature | Prioridad | Tech sugerida |
|---------|-----------|---------------|
| Sistema de ratings/reviews | Alta | Firestore |
| Notificaciones push reales | Alta | Firebase Cloud Messaging |
| Chat en tiempo real | Alta | Firestore realtime |
| Pagos / Escrow | Media | MercadoPago API |
| Verificación de identidad | Media | Persona API |
| Búsqueda avanzada | Media | Algolia |
| Panel de analytics | Baja | Firebase Analytics |
| Blog/contenido | Baja | CMS headless |

---

## 🛡️ Seguridad en Producción

- Mover autenticación a Firebase Auth (bcrypt real)
- Implementar rate limiting en endpoints de registro
- Validar y sanitizar inputs en el servidor
- Agregar reCAPTCHA v3 al registro
- Implementar CORS policies en el backend
- Usar HTTPS always (obligatorio para SW + PWA)

---

## 📞 Soporte WhatsApp
Número configurado: **+54 9 261 246-3768**
URL directa: [wa.me/5492612463768](https://wa.me/5492612463768)

Para cambiar el número, buscá en `index.html`:
```
https://wa.me/5492612463768
```

---

## 🧪 Datos Demo Incluidos

La app viene con 5 perfiles de demostración pre-cargados:
- **Valentina Ruiz** — Community Manager Senior (Remoto)
- **Lucas Fernández** — Editor de Video Semi Senior (Híbrido)
- **Sofía Morales** — Diseño Gráfico Junior (Remoto)
- **Mateo García** — Ads / Publicidad Senior (Remoto)
- **Camila López** — Community Manager Semi Senior (Híbrido)

Para ver los perfiles, registrate como empresa y exploralos desde el dashboard.
Para limpiar los datos demo, ejecutá en la consola del navegador:
```javascript
localStorage.clear(); location.reload();
```

---

*CreativeHub MVP v1.0.0 — Desarrollado con HTML/CSS/JS vanilla, PWA-ready*
