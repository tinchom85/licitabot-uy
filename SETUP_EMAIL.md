# Cómo configurar el Google Form y el envío de emails

## 1. Crear el Google Form

Ir a forms.google.com → Nuevo formulario en blanco.

**Título del form:**
```
LicitaBot UY — Recibí alertas de licitaciones del Estado
```

**Descripción:**
```
Te avisamos por email cuando el Estado uruguayo publique licitaciones en tu rubro.
Gratis. Sin spam. Podés desuscribirte cuando quieras.
```

**Preguntas (en este orden exacto):**

1. Tu nombre → Respuesta corta → Obligatorio
2. Nombre de tu empresa → Respuesta corta → Obligatorio
3. Tu email → Respuesta corta → Validar: texto → es correo electrónico → Obligatorio
4. Rubros de interés → Casillas de verificación → Opciones:
   - Tecnología e IT
   - Construcción e infraestructura
   - Salud e insumos médicos
   - Limpieza y mantenimiento
   - Seguridad y vigilancia
   - Logística y transporte
   - Consultoría y servicios
   - Alimentación y catering
   - Mobiliario y equipamiento
   - Todos los rubros
5. ¿Tu empresa está registrada en el RUPE? → Opción múltiple:
   - Sí, estamos registrados
   - No, pero queremos registrarnos
   - En proceso

**Configuración del Form:**
- Respuestas → Activar "Recopilar direcciones de correo electrónico" = OFF (ya lo pedimos nosotros)
- Respuestas → "Enviar copia a encuestados" = OFF
- Presentación → Mensaje de confirmación:
  ```
  ¡Listo! Te enviaremos alertas cuando haya licitaciones nuevas en tu rubro.
  Podés ver el monitor en tiempo real en: https://TU_USUARIO.github.io/licitabot-uy
  ```

---

## 2. Obtener la URL CSV de la Google Sheet

Cuando creás el Form, Google crea automáticamente una Sheet con las respuestas.

1. En el Form → pestaña "Respuestas" → ícono de Sheet (verde)
2. Eso abre la Sheet con las respuestas
3. En la Sheet → Archivo → Compartir → Publicar en la Web
4. Seleccionar: "Hoja 1" y formato "Valores separados por comas (.csv)"
5. Clic en "Publicar" → Copiar la URL

La URL tiene este formato:
```
https://docs.google.com/spreadsheets/d/XXXXX/pub?gid=0&single=true&output=csv
```

---

## 3. Configurar los secrets en GitHub

En tu repositorio de GitHub:
Settings → Secrets and variables → Actions → New repository secret

Agregar estos 3 secrets:

| Secret | Valor |
|--------|-------|
| `GMAIL_USER` | tu@gmail.com |
| `GMAIL_APP_PASSWORD` | contraseña de aplicación (ver paso 4) |
| `SHEET_CSV_URL` | URL del CSV del paso 2 |

---

## 4. Generar contraseña de aplicación de Gmail

Google no permite usar tu contraseña normal para enviar emails desde scripts.
Necesitás una "contraseña de aplicación":

1. Ir a myaccount.google.com
2. Seguridad → Verificación en dos pasos (debe estar activada)
3. Seguridad → Contraseñas de aplicación
4. Seleccionar app: "Correo" / Dispositivo: "Otro" → escribir "LicitaBot"
5. Clic en "Generar"
6. Copiar la contraseña de 16 caracteres (formato: xxxx xxxx xxxx xxxx)
7. Pegarla como valor del secret `GMAIL_APP_PASSWORD`

---

## 5. Probar localmente antes de publicar

```bash
# Configurar variables de entorno
export GMAIL_USER="tu@gmail.com"
export GMAIL_APP_PASSWORD="xxxx xxxx xxxx xxxx"
export SHEET_CSV_URL="https://docs.google.com/spreadsheets/d/.../pub?output=csv"

# Primero generar data.json
python scraper/arce_scraper.py

# Luego probar el envío
python scraper/email_sender.py
```

---

## 6. Flujo completo automático

Una vez configurado, cada mañana a las 06:00 Uruguay:

```
GitHub Actions corre
  ↓
arce_scraper.py → descarga XML de ARCE → guarda data.json
  ↓
email_sender.py → lee data.json + Google Sheet → envía emails personalizados
  ↓
git push → GitHub Pages sirve el index.html con data.json actualizado
```

---

## Cómo se ve el email que recibe el suscriptor

```
Asunto: 🏛️ 3 licitaciones nuevas para tu rubro — 18/03/2026

Hola Juan — Mi Empresa 👋

Encontramos 3 licitaciones nuevas del Estado uruguayo que coinciden 
con tus rubros. Publicadas en las últimas 24 horas.

[LA] LA-7604/2026
Organismo: ASSE
Objeto: Adquisición de insumos hospitalarios, medicamentos e implantes
Monto: $12.5M estimado UYU
⚠️ Cierra en 4 días
Ver pliego completo →

[CD] CD-2026/35
Organismo: Ministerio de Salud Pública
Objeto: Adquisición de equipos de diagnóstico por imágenes
Monto: $890K estimado UYU
⚠️ Cierra en 3 días
Ver pliego completo →

...

[ Ver monitor completo → ]
[ Próximamente: alertas por WhatsApp ]
```

---

## El copy para LinkedIn cuando el sistema esté listo

```
Construí un monitor de licitaciones del Estado uruguayo que 
actualiza solo cada mañana.

Hoy hay [X] oportunidades abiertas por $[Y]M en total.

Si tu empresa provee al Estado y querés recibir alertas 
personalizadas por email con las licitaciones de tu rubro:

→ [link al form]

Es gratis. Sin spam.
```
