# CLAUDE.md — Fichas Técnicas (lives-fichas-tecnicas)

Las reglas generales de trabajo están en el repo privado `lives-instrucciones` (CLAUDE.md). Este archivo solo describe esta herramienta.

## Qué hace
Buscador de fichas técnicas en PDF y control de medidas contra la ficha, desde el portal LIVES. Cuatro pantallas:
- `index.html` — **consulta** "Buscador de Fichas Técnicas".
  - Busca por estilo, grupo, cliente o archivo, y filtra por tipo (Técnica, Acabados, Corte, RM).
  - Muestra la ficha vigente con sus versiones y la abre en un visor propio (pdf.js).
  - El botón "Medir" lleva a `medidas.html`.
  - Solo la verificadora ve el panel "Ver todo lo cargado" y el acceso a `tablas.html`.
  - Tiene Excel LIVES.
- `medidas.html` — **formulario** "Control de medidas".
  - Se elige el taller, la etapa (Prenda terminada, Antes de vapor o Corte), el combo (Missy, Plus o Petite) y el color.
  - Se anota la diferencia contra la ficha, prenda por prenda y talla por talla, y se termina con "Aprobar y guardar" o "Rechazar y guardar".
  - Solo se abre desde el buscador (`?estilo=&tipo=`).
- `historial.html` — **consulta** "Mediciones realizadas": filtros, indicadores, la hoja imprimible de cada auditoría de medidas y Excel LIVES.
  - La hoja abre con "Ver figura" (dibujo de la prenda de la ficha, con zoom y cambio de página, como "Ver dibujo" del formulario) y con "Ver ficha" (PDF completo). La hoja impresa no lleva firmas (28/10/2026).
- `tablas.html` — **administración** de la verificadora. Sirve para:
  - subir fichas en PDF;
  - leer las tablas de medidas automáticamente;
  - corregirlas a mano y confirmar "La tabla es correcta";
  - quitar fichas.

## Base de datos (Supabase lives-datos)
Todo pasa por funciones; las pantallas no leen tablas directo.
- Leer:
  - `fichas_inicio`, `fichas_variantes`, `fichas_espec`, `fichas_medir` y `fichas_archivos` leen `fichas_indice` y `fichas_especificaciones`.
  - `fichas_listas` trae talleres, auditoras y la lista de quién verifica.
  - `fichas_mediciones` y `fichas_medicion` leen `fichas_mediciones` y `fichas_mediciones_detalle`.
  - `fichas_quien` lee `cat_personal`.
- Guardar mediciones (`medidas.html`): `fichas_guardar_medicion`, que escribe en `fichas_mediciones` y `fichas_mediciones_detalle`. El `p_id_envio` evita duplicados si se reintenta.
  - Desde el 28/10/2026 rechaza el guardado sin color y, si la ficha no trae cliente, lo saca del prefijo del grupo en `cat_clientes` (ej. COY-004 da COYUCHI; si el prefijo no está en el catálogo queda vacío). Respaldo de la versión anterior: tabla `respaldo_fichas_guardar_medicion_20261028`.
- Administrar fichas (`tablas.html`): `fichas_pdf_registrar` (tabla `fichas_pdf`), `fichas_indexar` (`fichas_indice`), `fichas_guardar_espec` y `fichas_borrar_espec_archivo` (`fichas_especificaciones`), y `fichas_quitar`.
- Registro de aperturas: `fichas_registrar_acceso`, en la tabla `fichas_accesos`.
- PDFs: bucket privado `fichas`, con la ruta `<id>.pdf`. Los PDFs también se guardan en el equipo, en IndexedDB `lives_fichas_pdf`.

## Quién la usa
Según las entradas por el portal de los últimos 30 días al 21/10/2026: 23 personas. Son sobre todo auditoras de calidad (18); también entran las supervisoras de calidad, la supervisora de Top Sample y la digitadora de Calidad Manufactura.

## Quién puede guardar
- Medir: quien figura como auditora activa en `cat_personal`. Lo comprueba `fichas_guardar_medicion`, que corre con permisos propios (security definer).
  - Christian (usuario `ctalepcio`) siempre puede medir y probar (desde el 28/10/2026), sin estar en `cat_personal`: lo permiten `fichas_guardar_medicion` y `fichas_quien`. Rodrigo no guarda. Respaldo de las versiones anteriores: tabla `respaldo_fichas_funciones_20261028`.
  - Sin modo demostración desde el 21/10/2026: la lista `DEMO` está vacía, así que todos ven la pantalla tal como es.
- Subir, indexar, confirmar o quitar fichas: lo decide la función de permiso `fichas_puede_subir`. Hoy lo hace la verificadora, que tiene el permiso "Confirmación de Fichas Técnicas" del portal.
- A Sistemas: `fichas_puede_subir` es la pieza que reemplazan con su seguridad.

## Pendientes vistos al escribir esto (21/10/2026)
- Queda un respaldo muerto: `accion=bytes` / `porAppsScript()` ya no funciona y siempre termina en "No se pudo bajar la ficha". También quedan nombres heredados de Drive (`abrirDeDrive`) que en realidad usan Supabase.
