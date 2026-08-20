# centinela-pages

Front del panel **Centinela** — estado en vivo de los flujos n8n de GIWA.

`https://uptime.giwa-ia.com`

## Por qué este repo es público y no filtra nada

GitHub Pages en plan free exige repo público, así que el HTML **no contiene ningún dato sensible**: ni nombres de clientes, ni IDs de workflows, ni endpoints de health, ni tokens. Todo eso llega desde la API **después** de autenticar.

Lo único público es la URL de la API y el `CLIENT_ID` de OAuth, que son públicos por diseño en cualquier app de navegador.

El runbook que se ve en la segunda pestaña usa ejemplos genéricos (`<cliente>`, `mi-servicio`). La versión con los datos reales vive en el repo privado `centinela`.

## Acceso

Login con Google restringido a `@giwa-ia.com`, mismo esquema que [redes-panel-pages](https://github.com/sechavarria-star/redes-panel-pages):

1. El front usa Google Identity Services con `hd: 'giwa-ia.com'` y manda el `id_token`
2. La API lo valida **server-side** contra `oauth2.googleapis.com/tokeninfo` y chequea `aud === CLIENT_ID` y `hd === giwa-ia.com`
3. Recién ahí consulta Postgres y devuelve datos

No hay clave compartida. El `id_token` vive ~1h, se guarda solo en memoria (no `localStorage`); al vencer, la API responde 401 y el front vuelve al login.

## API

Workflow n8n `GIWA - Uptime API` (`V9zTRv138RL7IYQX`), webhook `POST /webhook/centinela-api`.

Se llama con `Content-Type: text/plain` a propósito: así es un *simple request* y no dispara preflight CORS.

```
POST { "token": "<id_token>" }
200  { ok, usuario, resumen{total,ok,falla,desconocido,pausados}, servicios[], eventos[] }
401  { ok: false, motivo }
```

Lee de `giwa.sla_estado` y `giwa.sla_eventos`. **Nunca** devuelve `sla_servicios.secreto`, que es donde viven los tokens de bot.

## Config

En `index.html`, objeto `CONFIG`:

| Clave | Qué es |
|---|---|
| `API` | URL del webhook n8n |
| `CLIENT_ID` | OAuth Web del proyecto GCP `n8nGiwa` (el mismo del panel de Redes) |
| `DOMINIO` | Dominio permitido para el login |
| `REFRESCO_MS` | Auto-refresco del panel (60 s) |

El subdominio nuevo hay que agregarlo a **Orígenes de JavaScript autorizados** del Client ID en GCP, si no da `origin_mismatch` (tarda 5-10 min en propagar).

## Relacionado

- `centinela` (privado) — runbook con datos reales y `schema.sql`
- Workflows: Uptime (triggers), Uptime (errores), Avisar, Latido
