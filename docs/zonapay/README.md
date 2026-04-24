# Zonapay — Paquete de documentación para docs.equipozonapagos.com

Este paquete contiene la documentación técnica de **Zonapay** (la pasarela de pagos) lista para integrarse al sitio Mintlify existente en `docs.equipozonapagos.com`.

## Qué incluye

- **49 páginas MDX** con contenido técnico listo.
- **1 archivo `_NAVIGATION_FRAGMENT.json`** con el fragmento de navegación que debe añadirse al `docs.json` del sitio padre.
- Total: ~50 archivos organizados bajo la estructura esperada.

## Estructura

```
zonapay/
├── _NAVIGATION_FRAGMENT.json    ← Instrucciones para docs.json del sitio padre
├── introduccion.mdx             ← Reemplaza la "Introduccion" vacía actual
├── empezar/
│   ├── como-funciona.mdx
│   ├── ambientes.mdx
│   ├── quickstart.mdx
│   └── changelog.mdx
├── guias/                       ← Integración + casos de uso + buenas prácticas
│   ├── iniciar-pago.mdx
│   ├── redirigir-usuario.mdx
│   ├── recibir-callback.mdx
│   ├── verificar-estado.mdx
│   ├── implementar-sonda.mdx
│   ├── manejo-errores.mdx
│   ├── idempotencia.mdx
│   ├── seguridad-callback.mdx
│   ├── logging.mdx
│   ├── checklist-produccion.mdx
│   ├── certificacion-pse.mdx    ← Requisitos + mensajes + pruebas consolidados
│   ├── pse-simple.mdx
│   ├── tc-simple.mdx
│   ├── pagos-mixtos.mdx         ← Incluye fraccionados
│   ├── pagos-recurrentes.mdx
│   └── cobro-transaccion.mdx    ← Incluye multimoneda (USD)
├── api/
│   ├── introduccion.mdx
│   ├── autenticacion.mdx
│   ├── convenciones.mdx
│   ├── endpoints/               ← InicioPago, VerificacionPago, callback
│   ├── objetos/                 ← Request + response schemas
│   ├── referencia/              ← Tablas: estados, medios, tipos ID, etc.
│   └── formatos/                ← Parser de str_res_pago + campos por medio
├── recetas/
│   ├── curl.mdx
│   ├── nodejs.mdx
│   ├── python.mdx
│   ├── csharp.mdx
│   ├── php.mdx
│   └── postman.mdx
└── soporte/
    ├── faq.mdx                  ← Reemplaza "Preguntas-frecuentes" actual
    ├── errores-comunes.mdx
    └── sandbox.mdx
```

## Pasos para integrar al sitio

### 1. Clonar el repo del sitio

```bash
git clone <repo-de-docs.equipozonapagos.com>
cd <repo>
git checkout -b feature/zonapay-docs-v6.1
```

### 2. Copiar los archivos MDX

Desempaqueta el ZIP en la raíz del repo, dentro de `docs/zonapay/`:

```bash
# Desde la raíz del repo
rm -rf docs/zonapay/Introduccion.mdx docs/zonapay/Preguntas-frecuentes.mdx
unzip /ruta/al/zonapay-docs.zip -d docs/
# Ahora docs/zonapay/ contiene toda la nueva estructura
```

### 3. Actualizar docs.json

Abre `docs.json` del sitio. Busca el grupo actual de Zonapay — debería verse más o menos así:

```json
{
  "group": "Pasarelas de pagos - Zonapay",
  "pages": [
    "docs/zonapay/Introduccion",
    "docs/zonapay/Preguntas-frecuentes"
  ]
}
```

Reemplázalo por el contenido de `fragmento_navegacion_a_reemplazar` del archivo `_NAVIGATION_FRAGMENT.json`.

### 4. Verificar en local

```bash
npm i -g mintlify    # si no lo tienes
mintlify dev
```

Abre `http://localhost:3000` y navega a **Pasarelas de pagos - Zonapay** en el sidebar. Deberías ver la nueva estructura completa.

### 5. Validar enlaces

Búsquedas útiles para confirmar que no hay rutas rotas:

```bash
# Verificar que todas las rutas internas apuntan a /docs/zonapay/ o /docs/empieza-ahora/
grep -rn 'href="/' docs/zonapay/ | grep -v 'docs/zonapay\|docs/empieza-ahora\|mailto:\|http'

# Verificar que no hay rutas viejas del paquete anterior
grep -rn 'href="/api\|href="/guias\|href="/empezar\|href="/recetas\|href="/soporte' docs/zonapay/
```

Ambos comandos deben devolver cero líneas.

### 6. PR y despliegue

```bash
git add docs/zonapay/ docs.json
git commit -m "docs: restructure Zonapay section with technical integration guides"
git push origin feature/zonapay-docs-v6.1
```

Abre PR. Mintlify despliega automáticamente al merge.

## Referencias externas del paquete

Algunas páginas hacen referencia a contenido que **ya vive en el sitio padre** y no hay que crear:

- `/docs/empieza-ahora/glosario-tecnico` — reemplaza el glosario interno de Zonapay.
- `/docs/empieza-ahora/medios-de-pago` — contenido cross-producto.

Al integrar, verifica que esas rutas existan. Si cambian de path en el sitio padre, búscalas en los MDX con:

```bash
grep -rn 'docs/empieza-ahora' docs/zonapay/
```

## Items pendientes de validación con TI antes de publicar

Varias páginas tienen bloques `<Warning>` marcando información que no se pudo validar al 100% porque el API REST sandbox no está disponible públicamente. **Revísalas con el equipo técnico antes de hacer público el sitio.**

Búscalos con:

```bash
grep -rn "Pendiente" docs/zonapay/ --include="*.mdx"
```

Los principales (listados también en `_NAVIGATION_FRAGMENT.json`):

1. Valor correcto de `int_modalidad` (el instructivo oficial v6.0 se contradice entre `-1` y `1`).
2. Disponibilidad pública o no de endpoint REST sandbox.
3. Normalización de mensajes de error del API (actualmente expone excepción `.NET` cruda).
4. Remover header `X-AspNet-Version` del servidor.
5. Documentar algoritmo del token `rut` del ciclo de pago.
6. Evaluar firma HMAC del callback al comercio.
7. Publicar rango de IPs del callback para allowlist.
8. Confirmar si `flt_total_con_iva` va en USD o COP al usar código `117: "US"`.
9. Documentar formalmente flujo de notificaciones de cobros recurrentes.
10. Resolver ambigüedad del formato de porcentaje (`"0195"` 4 dígitos vs `"02"` 2 dígitos).

Cada pendiente está marcado puntualmente en el archivo correspondiente.

## Postman collection

La página `recetas/postman.mdx` referencia dos archivos:

- `ZonaPagos_API_v6.0_TestSuite.postman_collection.json`
- `ZonaPagos-Sandbox.postman_environment.json`

Ambos ya los tienes del trabajo anterior. Deben subirse a un lugar público (idealmente al workspace público de Postman de Zonapagos) o alojarse en el mismo repo bajo `public/files/` y ajustar los hrefs.

## Contacto

Mantenedor técnico del paquete: (por definir)  
Soporte a desarrolladores: [soporte@zonavirtual.com.co](mailto:soporte@zonavirtual.com.co)

---

**Versión del contenido:** 6.1 (abril 2026)  
**Basado en:** instructivo oficial I-TI-008 v6.0
