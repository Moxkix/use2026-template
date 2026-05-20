# Consulta de aula PAU/USaP — web del alumnado (template)

Web estática (GitHub Pages) donde el alumnado de un tribunal de la **Prueba de
Acceso a la Universidad (PAU/USaP)** consulta su aula de examen, por QR o por
código. Diseño Material 3 verde institucional EHU, bilingüe euskera/castellano.

Este repositorio es un **template saneado**: trae datos de ejemplo, sin
endpoints ni datos reales de ningún tribunal. No publiques contra él — cada
coordinador crea su propia copia con **"Use this template"** y la rellena. Los
datos personales del alumnado **nunca** llegan aquí: solo el JSON anónimo de
aulas (código de examen → asignatura, aula, día, hora). Sin nombres, sin DNIs.

---

## Cómo crear tu copia (otro coordinador)

### Opción A — Web (recomendada)

1. En la página del repo, botón verde **"Use this template" → Create a new
   repository**.
2. **Owner**: tu usuario u organización de GitHub. **Repository name**:
   p. ej. `consulta-2026`. Visibilidad **Public** (GitHub Pages gratuito lo
   exige). Deja "Include all branches" desmarcado.
3. **Create repository from template**. Obtienes una copia limpia, sin historial
   común y sin secrets ni tokens del autor.
4. En tu repo: **Settings → Pages → Source: Deploy from a branch → `main` /
   root → Save**. En 1-2 min estará en
   `https://<tu-usuario>.github.io/<tu-repo>/`.

### Opción B — CLI

```powershell
gh repo create consulta-2026 --template <owner>/<repo-template> --public --clone
cd consulta-2026
# ...personaliza (ver abajo)...
git add -A; git commit -m "Personalizacion tribunal X"; git push
```

---

## Personalización obligatoria

> **Seguridad:** los tres primeros puntos son obligatorios. Si los saltas, tu
> web enviará estadísticas al Worker de otro tribunal o las enviará al vacío
> (la CSP las bloquea). Revísalos antes del primer `push`.

### 1. `asignaturas_aulas.json` — tus datos

Lo genera y sube **tu PWA del coordinador** (Exportación · Publicación JSON en
GitHub). En el template viene un **ejemplo con códigos ficticios** (`DEMO01`…):
deja que la primera publicación de la PWA lo sobrescriba. **No lo edites a
mano.**

### 2. `script.js` — URL de tu Worker QR

```js
// Línea ~20
const QR_LOG_URL = '';   // ← pon AQUÍ la URL de TU Worker, o déjala vacía
```

- Con Worker propio (estadísticas de escaneos):
  `const QR_LOG_URL = 'https://qr-log-tribunal.<tu-usuario>.workers.dev';`
- Sin estadísticas: déjala como `''`. La web funciona igual; no cuenta nada.
  **Nunca dejes la URL del Worker de otro coordinador.**

### 3. `index.html` — CSP `connect-src` (va de la mano del punto 2)

En la cabecera `Content-Security-Policy`, el `connect-src` debe apuntar a **tu**
Worker (o quedarse en `'self'` si no usas estadísticas):

```
connect-src 'self' https://qr-log-tribunal.<tu-usuario>.workers.dev;
```

Si `QR_LOG_URL` y `connect-src` no coinciden, el navegador bloquea el beacon en
silencio.

### 4. Plano e identidad de tu sede (`index.html`)

- `plano_farmacia_2planta.png` → sustitúyelo por el plano de **tu** planta
  (mismo nombre de fichero, o cambia la referencia en `index.html`).
- Texto de la sede (`Zentroaren izena · Nombre del centro`, planta, localidad).
- Enlace de **Google Maps** (`query=...`).
- `USE-PAU-2026-OHARRAK.pdf` → sube tu PDF de "información importante", o quita
  esa sección si no lo usas (en el template el enlace apunta a un PDF que aún no
  existe).

### 5. Lo que normalmente NO tocas

`tailwind-config.js` (paleta EHU), `blanco_mediano.jpg` (logo EHU), la lógica de
`script.js`. Compártelos tal cual entre tribunales.

---

## Estructura del repositorio

| Fichero | Qué es | ¿Personalizar? |
|---|---|---|
| `index.html` | Página única. CSP estricta, sin scripts inline. | Sí — CSP §3, sede §4 |
| `script.js` | Lógica: lee `?codigos=`, búsqueda manual, beacon QR. Render por DOM API (sin `innerHTML`), códigos validados con regex `^[A-Z0-9]{2,12}$`. | Sí — `QR_LOG_URL` §2 |
| `tailwind-config.js` | Paleta Material 3 verde EHU. | No |
| `asignaturas_aulas.json` | Datos de consulta (ejemplo `DEMO01`…; lo sube la PWA). | Sí — lo genera tu PWA §1 |
| `blanco_mediano.jpg` | Logo EHU blanco (cabecera + marca de agua). | No |
| `plano_farmacia_2planta.png` | Plano de la planta de examen (ejemplo). | Sí — §4 |
| `USE-PAU-2026-OHARRAK.pdf` | PDF de información al alumnado (NO incluido). | Sí — añádelo o quita la sección §4 |

---

## Modelo de privacidad (resumen)

- Esta web **no recoge datos personales**. El JSON publicado no contiene nombres
  ni DNIs; el código de examen no es reconducible al alumno sin la matrícula
  privada del tribunal.
- El beacon al Worker QR (si lo activas) es un `POST` **sin cuerpo**: el Worker
  solo incrementa contadores agregados y un HyperLogLog no reversible. Sin IPs,
  sin User-Agents, sin códigos persistidos.
- No hay cookies de tracking ni analítica de terceros.
- Detalle completo en el RAT del tribunal (`RAT_TRIBUNAL_PAU.md`).

## Licencia

MIT (ver `LICENSE`). Sustituye el titular del copyright por el tuyo si procede.
