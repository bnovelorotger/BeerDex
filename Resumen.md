# BeerDex — Documento de producto y técnico (Alpha → Futuro)

## 1) Resumen ejecutivo

**BeerDex** es una “Pokédex de cervezas” con estética de **taberna interactiva**: el usuario registra cervezas que va probando durante el año (marca concreta), opcionalmente añade **rating**, **nota mini** y **foto-evidencia**, y recibe feedback cercano (XP, niveles, badges). La experiencia está diseñada para ser **ultra simple** y usable en un bar, pub o cena con amigos, incluso con poca atención o “happy”.

El resultado actual es una **versión alpha sólida** en **HTML/CSS/JS** sin backend: el estado vive en **localStorage**, y la app se puede portar a otro dispositivo mediante **export/import**.

---

## 2) Problema que resuelve

### Problema principal

En contextos sociales (bares, pubs), la gente:

* olvida qué cervezas probó,
* no recuerda cuál le gustó,
* no tiene un “registro divertido” que apetezca usar,
* y no tiene un sistema sencillo de colección/progreso.

### Solución

BeerDex convierte el registro en:

* **colección** (estilo pokédex),
* **progreso** (XP y niveles),
* **recompensa** (badges),
* **evidencia rápida** (foto opcional),
* y una UX minimalista para apuntar en segundos.

---

## 3) Usuario objetivo y contexto de uso

**Target**: jóvenes adultos **25–45**.

### Contexto

* bar, pub, terraza, concierto, cena con amigos
* iluminación irregular, manos ocupadas, atención limitada
* motivación social: “a ver quién las consigue todas”

### Principios UX derivados

* “**menos toques**”: registrar es lo central
* campos **opcionales** (rating, foto, nota) para no frenar el registro
* lenguaje cercano, gratificante, tono “taberna”

---

## 4) Inspiraciones y estilo de marca

Inspiraciones iniciales (conceptuales):

* **Strava**: claridad, módulos, “resumen” fácil y sensación de progreso.
* **Letterboxd**: catálogo + colección + listas + ranking personal.
* **BeReal**: simplicidad radical y evidencia (foto) sin fricción.

### Estilo visual definido: “taberna interactiva”

* **paleta oscura** (fondo tipo madera / barrica)
* **acentos cálidos** (naranja/ámbar para CTA y elementos clave)
* tarjetas suaves, sombras y blur tipo “barra nocturna”
* microcopy y toasts que “dan palmaditas en la espalda”

---

## 5) Producto actual: MVP (alcance real)

### Funcionalidades que tiene (Alpha)

#### Colección / Dex

* Catálogo de cervezas (marca concreta) desde `beers.json`.
* Vista de listado + búsqueda.
* Rarezas (common/rare/mythic) para coleccionismo.

#### Captura (registro)

* Modal “Captura en la taberna” desde botón “+”.
* Selección de cerveza del catálogo.
* Rating opcional (0–5).
* Foto opcional (evidencia).
* Nota mini opcional (micro review).
* Guardado sin bloquear (si no pones nada, también guarda).

#### Perfil

* Nombre y avatar (emoji o foto propia).
* Niveles + XP (sensación RPG suave).
* Badges (insignias) y diario de registros.
* Export/Import para mover datos sin backend.
* Reset local.

### Ajustes de “feel” ya implementados

* Badges **ocultas** hasta conseguirlas (no aparecen bloqueadas).
* Eliminación de la burbuja “JSON” en el header.
* Modal renombrado: “Tu personaje” → “Tu avatar”.
* Guardado rápido con **Enter** en el campo nombre (además del botón).
* Inputs `file` estilizados con botón “taberna” (no look nativo del sistema).

### Lo que se dejó fuera a propósito (para foco MVP)

* Feed / compartir / social.
* Ranking global.
* Sincronización multi-dispositivo.
* Backend, cuentas, autenticación.
* PNGs por cerveza (por ahora icono/emoji).

---

## 6) Datos y modelo (JSON + almacenamiento local)

### 6.1 `beers.json` (catálogo)

Se decidió un catálogo gestionable por fichero, fácil de ampliar sin tocar código.

**Esquema actual (por cerveza):**

```json
{ "id", "name", "brand", "style", "country", "abv", "rarity", "emoji" }
```

**Notas de diseño del catálogo:**

* `id` estable: clave para registros y para futuras imágenes (`/assets/beers/<id>.png`).
* `rarity`: permite gamificación sin complicar el modelo.
* `emoji` fijo por ahora, con plan de sustituir por imagen PNG.

**Extensiones previstas (futuro):**

* `img`: ruta a PNG.
* `brewery`, `region`, `ibu`, `tags` (si se decide).
* `available_in`: supermercados / comunidades (si se quiere “modo realista España”).

### 6.2 `badges.json` (insignias)

Badges definidas por reglas simples para no crear complejidad.

**Esquema actual:**

```json
{ "id", "name", "icon", "desc", "rule" }
```

**Tipos de reglas contempladas en el MVP**

* `total_records`
* `unique_beers_year`
* `style_count_year`
* `photos_year`
* `mythics_year`
* `notes_year` (mencionada como mejora; se puede añadir fácilmente)

### 6.3 Estado local (localStorage)

**`user`**

* `name`, `avatar` (emoji), `avatarPhoto` (dataURL), `xp`, `badges[]`.

**`records` (diario)**
Cada registro guarda:

* `beerId`
* timestamp (fecha/hora)
* `rating` opcional
* `note` opcional (máx 120)
* `photo` opcional (dataURL comprimido)

**Export/Import**

* Se serializa el estado a JSON y se reinyecta en localStorage.
* Permite “mover” el progreso sin backend.

**Limitaciones conocidas**

* localStorage tiene límite (varía por navegador).
* fotos en DataURL ocupan mucho: se recomienda compresión y uso moderado.

---

## 7) Arquitectura y tecnología (actual)

### Arquitectura actual (Alpha)

**SPA ligera en HTML/CSS/JS**, sin frameworks:

* `index.html` (estructura + modales)
* `styles.css` (tema taberna)
* `app.js` (lógica: render, almacenamiento, reglas badges, XP, import/export)
* `beers.json` y `badges.json` como “contenido” editable

### Carga de datos

* En local con `file://` el fetch de JSON puede fallar por CORS.
* Se recomienda ejecutar con servidor local (`python -m http.server`).

### Por qué este enfoque

* máxima velocidad para iterar
* cero dependencias
* fácil de entender por no programadores
* muy portable: abrir, probar, editar

---

## 8) Backend, API y seguridad (estado actual y futuro)

### Estado actual: sin backend

* No hay cuentas, ni login, ni red.
* No hay superficie de ataque de autenticación.
* La “seguridad” se centra en:

  * evitar XSS al renderizar texto (escapar HTML en notas/nombres),
  * limitar tamaño de imágenes (compresión y tamaño máximo),
  * no ejecutar contenido del usuario como HTML.

### Futuro: backend (cuando llegue el momento)

Si se añade social/ranking/global:

**Servicios necesarios**

1. **Auth**

   * Email / OAuth (Apple/Google) o magic link.
2. **DB**

   * Users
   * Beer catalog (versionado)
   * Captures/Reviews
   * Badges/Progress
3. **Storage de imágenes**

   * S3/GCS o equivalente con URLs firmadas.
4. **API**

   * REST o GraphQL.
   * Endpoints típicos:

     * `GET /catalog/beers?version=...`
     * `POST /captures`
     * `GET /users/me`
     * `GET /leaderboards/...`
     * `POST /upload-url` (pre-signed)
5. **Moderación**

   * fotos (contenido indebido)
   * reviews (spam)
6. **Privacidad / GDPR**

   * exportar / borrar cuenta
   * consentimiento imágenes

**Seguridad**

* rate limiting
* validación server-side (IDs, tamaños, campos)
* control anti-cheat en ranking (evitar spam de capturas)
* firmas de subida a storage + scanning básico si escalara

---

## 9) Rendimiento y calidad (consideraciones)

### Rendimiento actual

* Catálogo en memoria (150 items) es trivial.
* Render de listas debe:

  * filtrar con debounce (si el catálogo sube a miles)
  * evitar repintados innecesarios
* Fotos en DataURL:

  * comprimir antes de guardar
  * limitar número o tamaño
  * limpiar referencias si se borra registro

### Mejoras “baratas” recomendadas

* Índice simple por nombre/marca (precomputed) para búsqueda rápida.
* Paginación o “virtual list” si el catálogo crece mucho.
* Lazy rendering en secciones no visibles.

---

## 10) Roadmap propuesto (de Alpha a producto serio)

### Fase 1 — Calidad de contenido (muy inmediata)

1. **Depurar `beers.json`**

   * normalizar estilos (`IPA`, `NEIPA`, `Wheat`, etc.)
   * revisar duplicados
   * completar ABV cuando falte
   * añadir marcas blancas (ya iniciado)
2. **Depurar `badges.json`**

   * 10–30 badges “divertidas”
   * balance de dificultad
   * añadir `notes_year` si se quiere premiar reviews
3. **PNG por cerveza**

   * carpeta `/assets/beers/<id>.png`
   * fallback al emoji si no existe imagen
4. **UX “captura rápida”**

   * reforzar: “elige cerveza → Guardar” como camino principal

### Fase 2 — Gamificación fina

* niveles con nombres (Novato / Habitual / Épico / Legendario)
* “stamp”/animación al capturar nueva cerveza
* rareza que se sienta (micro confetti, brillo, sonido opcional)
* “colección completada” por categorías (IPA, Trappist, etc.)

### Fase 3 — Social (cuando el core ya engancha)

* amigos / grupos
* comparar colecciones
* retos (“semana IPA”, “ruta taberna”)
* ranking de cervezas “entre amigos”
* **Feed (posts) como se mencionó al inicio**: formato tipo Strava donde cada captura puede convertirse en un post con foto-evidencia, rating y nota mini; los amigos pueden reaccionar/comentar y esto alimenta el uso recurrente.

### Fase 4 — Global / plataforma

* cuentas, sync multi-dispositivo
* ranking global por ciudad/país
* recomendaciones (top personal + top global + “parecidas a tu gusto”)
* moderación y políticas

---

## 11) Decisiones clave del proceso (qué se aprendió y por qué se hizo así)

1. **No bloquear el registro con campos obligatorios**

   * En el bar, el usuario quiere “apuntar rápido”.
   * Rating/foto/nota quedan opcionales sin penalización.

2. **Colección primero; feed después**

   * Si la colección no engancha, el social no salva el producto.
   * La pokédex + progreso crea hábito.

3. **Tema taberna oscuro**

   * Se buscó calidez y “madera/barrica”.
   * Acentos cálidos hacen CTAs claros sin fondo blanco.

4. **Badges ocultas**

   * Evita frustración visual (“tengo 50 bloqueadas”).
   * Refuerza sorpresa y recompensa real.

5. **Sin backend en MVP**

   * Velocidad de iteración.
   * Menos mantenimiento.
   * Export/import como sustituto funcional.

---

## 12) Pendientes actuales (lista práctica)

### Contenido

* completar y validar las 150 cervezas (nombres, ABV, estilos, rarezas)
* ampliar el catálogo por capas (macro, regional, import)
* crear PNGs coherentes (estilo consistente)

### Badges

* definir set final (divertidas, coleccionismo, estilos, streaks)
* implementar reglas adicionales si se desean (`notes_year`, `streak_days`, etc.)

### UX

* toast especial cuando se desbloquea badge
* micro animación “nueva captura”
* mejorar pantalla Stats: top estilos, media rating, rarezas capturadas

---

## 13) Cómo enseñar el proyecto a otra persona (guion breve)

* “BeerDex es una Pokédex de cervezas con estética de taberna.”
* “El MVP está hecho en HTML/CSS/JS sin backend: todo vive en localStorage.”
* “Se puede exportar/importar para mover el progreso sin backend.”
* “El corazón es: catálogo → captura → diario + progreso (XP, niveles, badges).”
* “La siguiente fase es depurar catálogo/badges y añadir PNGs; luego social, incluyendo feed.”
