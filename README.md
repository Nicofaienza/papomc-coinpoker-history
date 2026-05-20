# papomc-coinpoker-history

Repo público de datos y auditoría de la promoción **PapoMC x CoinPoker — Mesa Final**.
Contiene snapshots intra-día de los rankings y los archivos finales mensuales,
generados automáticamente por el servicio [LEADERBOARD](https://github.com/Nicofaienza/LEADERBOARD).

> **No editar este repo a mano.** Todo el contenido se genera y publica
> por el scheduler en Railway via la GitHub API. Cualquier cambio manual
> se va a sobreescribir en el próximo ciclo.

---

## Para qué sirve

1. **Auditoría pública.** Cada commit es una snapshot inmutable de los
   rankings en un momento exacto. Cualquier jugador puede verificar el
   estado histórico exacto del leaderboard mirando el `git log`.
2. **Fuente de datos de la landing.** La sección "Ediciones Anteriores"
   de [coinpoker.papomc.com](https://coinpoker.papomc.com) lee los
   archivos finales mensuales directo desde `raw.githubusercontent.com`
   (gratis, con CDN, sin pegarle al backend).

---

## Estructura

```
.
├── index.json                                       Lista de meses archivados
├── snapshots/                                       Snapshots intra-día (cada 2h UTC)
│   └── YYYY-MM/
│       └── YYYY-MM-DD/                              Carpeta por día
│           ├── 00h_rake.json
│           ├── 00h_twitch.json
│           ├── 02h_rake.json
│           ├── 02h_twitch.json
│           └── ...                                  (12 slots × 2 tipos = 24 archivos/día)
└── archives/                                        Final mensual (consolidado el día 1)
    └── YYYY-MM/
        ├── rake.json
        └── twitch.json
```

**Diferencia clave:**

- `snapshots/` = log granular de auditoría (12 fotos por día, una cada 2hs).
- `archives/` = resultado final consolidado del mes, lo que la landing
  muestra como "ranking definitivo" de una edición pasada.

---

## Schemas

### `snapshots/YYYY-MM/YYYY-MM-DD/HHh_<type>.json`

```json
{
  "snap_date":  "2026-05-06",
  "period":     "May 2026",
  "type":       "rake",
  "taken_at":   "2026-05-06T16:04:14Z",
  "slot_label": "16h UTC",
  "players": [
    { "position": 1, "name": "Aldrish",  "value": 1234.56, "prize": "$890" },
    { "position": 2, "name": "GreyGrey", "value": 1100.00, "prize": "$600" }
  ]
}
```

- `type`: `"rake"` (rake en USD) o `"twitch"` (puntos de StreamElements).
- `value`: numérico — cantidad de rake o puntos según el tipo.
- `players`: lista completa de participantes en el momento de la snapshot
  (no limitada a 15 — incluye todos los activos).

### `archives/YYYY-MM/<type>.json`

```json
{
  "period":      "April 2026",
  "period_key":  "2026-04",
  "type":        "rake",
  "archived_at": "2026-05-04T06:19:20Z",
  "top": [
    { "position": 1, "name": "plump3439465025", "value": 441.84, "prize": "$650" },
    { "position": 2, "name": "Aldrish",        "value": 72.34,  "prize": "$450" }
  ]
}
```

- `top`: top 15 ganadores con su premio.
- `archived_at`: momento exacto en que se generó el archivo final (siempre el día 1 del mes siguiente, después de medianoche UTC).
- Los premios reflejan los montos vigentes en esa edición — pueden variar entre meses.

### `index.json`

Lista plana de todos los meses archivados (más nuevo primero):

```json
[
  { "key": "2026-04", "label": "April 2026" }
]
```

Usado por la landing para enumerar las ediciones disponibles sin tener
que listar el contenido del repo.

---

## Cómo consumir los datos

Todos los archivos son accesibles vía CDN público:

```
https://raw.githubusercontent.com/Nicofaienza/papomc-coinpoker-history/main/<path>
```

Ejemplos:

```
# Índice de ediciones
https://raw.githubusercontent.com/Nicofaienza/papomc-coinpoker-history/main/index.json

# Final de Abril 2026 — rake
https://raw.githubusercontent.com/Nicofaienza/papomc-coinpoker-history/main/archives/2026-04/rake.json

# Snapshot del slot 16h del 6 de Mayo 2026 — twitch
https://raw.githubusercontent.com/Nicofaienza/papomc-coinpoker-history/main/snapshots/2026-05/2026-05-06/16h_twitch.json
```

CORS abierto (es `raw.githubusercontent.com`), se puede consumir directo
desde el navegador sin proxy.

---

## Cómo se genera

El servicio LEADERBOARD corre un scheduler que:

1. **Cada 2 horas** (slots fijos en UTC: 00, 02, 04, ..., 22), toma un
   snapshot de ambos rankings y publica el archivo correspondiente en
   `snapshots/` via la GitHub API. Cada commit es independiente.
2. **El día 1 de cada mes** a las 00:00 UTC, consolida el último estado
   del mes anterior en `archives/` y actualiza `index.json`.

Si querés agregar un mes manualmente (backfill por ejemplo), el script
[`backfill.js`](https://github.com/Nicofaienza/LEADERBOARD/blob/main/backfill.js)
del repo LEADERBOARD lo permite — corre con un `GITHUB_TOKEN` con
permisos `contents:write` sobre este repo.

---

## Repos relacionados

- [**LEADERBOARD**](https://github.com/Nicofaienza/LEADERBOARD) — Backend
  Python en Railway que genera estas snapshots y archivos.
- [**papomc-coinpoker**](https://github.com/Nicofaienza/papomc-coinpoker) —
  Landing pública que consume `archives/` para mostrar las ediciones
  anteriores.
