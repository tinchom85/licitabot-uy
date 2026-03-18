# LicitaBot UY — Monitor de Licitaciones Públicas

Monitor público de licitaciones del Estado uruguayo que actualiza automáticamente cada 24 horas usando datos de [comprasestatales.gub.uy](https://www.comprasestatales.gub.uy) (ARCE).

**Arquitectura: costo USD 0/mes**
- GitHub Actions (scraper cron diario) — gratis
- GitHub Pages (hosting del frontend) — gratis
- Sin base de datos, sin servidor, sin factura

---

## Estructura

```
licitabot/
├── index.html                    # Frontend (lee data.json)
├── data.json                     # Generado automáticamente por el scraper
├── scraper/
│   └── arce_scraper.py           # Scraper Python del XML de ARCE
└── .github/
    └── workflows/
        └── update_data.yml       # GitHub Actions — cron cada 24h
```

---

## Setup en 5 pasos

### 1. Crear el repositorio en GitHub

```bash
# Subir este código a un repo público en GitHub
git init
git add .
git commit -m "init: LicitaBot UY"
git remote add origin https://github.com/TU_USUARIO/licitabot-uy.git
git push -u origin main
```

### 2. Activar GitHub Pages

En el repositorio de GitHub:
- Settings → Pages → Source: **Deploy from a branch**
- Branch: `main` → Folder: `/ (root)`
- Guardar

Tu demo queda en: `https://TU_USUARIO.github.io/licitabot-uy`

### 3. Correr el scraper por primera vez manualmente

En GitHub: Actions → "Actualizar licitaciones ARCE" → **Run workflow**

Esto genera el `data.json` con datos reales de ARCE. Desde ese momento corre solo todos los días a las 06:00 Uruguay.

### 4. (Opcional) Probar localmente antes de subir

```bash
pip install httpx
python scraper/arce_scraper.py

# Abrir index.html en un servidor local (no funciona desde file://)
python -m http.server 8000
# → http://localhost:8000
```

### 5. Personalizar el dominio (opcional)

Si tenés `licitabot.uy` registrado en nic.uy:
- En GitHub Pages → Custom domain → poner `licitabot.uy`
- En tu proveedor DNS → agregar CNAME `www` → `TU_USUARIO.github.io`

---

## Cómo funciona el scraper

La interfaz G2B de ARCE acepta requests HTTP GET y devuelve XML.

```
GET https://www.comprasestatales.gub.uy/comprasenlinea/servlet
    ?tipoPublicacion=VIG
    &tipoCompra=
    &fechaInicio=DD/MM/AAAA
    &fechaFin=DD/MM/AAAA
```

El scraper hace 3 requests de 10 días cada uno (máximo permitido por ARCE) para cubrir 30 días. Parsea el XML, clasifica cada licitación por rubro con palabras clave, calcula días para cierre, y guarda el resultado en `data.json`.

El frontend lee `data.json` directamente — sin llamadas a APIs externas, sin CORS, sin problemas.

---

## Frecuencia de actualización

El workflow corre a las **09:00 UTC = 06:00 Uruguay** todos los días.

Para cambiar el horario, editar `.github/workflows/update_data.yml`:
```yaml
schedule:
  - cron: '0 9 * * *'   # 09:00 UTC = 06:00 UY
```

---

## Datos

- **Fuente:** comprasestatales.gub.uy (ARCE — Agencia Reguladora de Compras Estatales)
- **Licencia:** Datos Abiertos — Licencia DAG Uruguay
- **Cobertura:** Licitaciones publicadas en los últimos 30 días
- **Tipos incluidos:** LP, LA, CD, CP y otros
- **Límite:** 200 licitaciones por actualización (las más urgentes primero)

---

## Contacto

hola@agente.uy | [agente.uy](https://agente.uy)
