# Portal de Compras Corporativas

Tablero de una sola página para monitorear las claves de insumos corporativos.

**En vivo:** https://ftrevinor001-tr.github.io/Portal-compras-corporativas/

El repositorio guarda dos archivos:

| Archivo | Qué es |
|---|---|
| `index.html` | El portal. Se toca solo cuando hay cambios de funcionalidad. |
| `datos.json` | Todos los cortes acumulados (~275 KB con 14 cortes; GitHub lo sirve comprimido en ~15 KB). **Este es el que se reemplaza cada vez que hay un corte nuevo.** |

Al abrir el portal, este lee `datos.json` del repositorio automáticamente. Cualquier
persona, en cualquier computadora, ve los mismos datos sin cargar nada.

---

## Publicar un corte nuevo

Hay dos caminos. Los dos dejan el mismo resultado; elige el que te acomode.

### Camino A — subirlo a mano (sin token)

1. Abrir el portal → pestaña **📂 Datos**.
2. Arrastrar el `Stock compras corporativas.xlsx` nuevo. Se ve una **vista previa local**
   (solo la ves tú; el equipo sigue viendo lo anterior).
3. Revisar que todo cuadre en el Tablero.
4. Volver a **Datos** → botón **⬇ Descargar datos.json**.
5. En el repo: **Add file → Upload files** → arrastrar el `datos.json` → **Commit changes**.
   GitHub reemplaza el anterior.
6. En ~1 minuto, todo el equipo ve el corte nuevo.

### Camino B — publicar desde el portal (con token)

Un solo clic, pero requiere configurar un token una vez.

**Crear el token (una sola vez):**

1. GitHub → foto de perfil → **Settings**
2. Hasta abajo: **Developer settings**
3. **Personal access tokens** → **Fine-grained tokens** → **Generate new token**
4. *Token name:* `portal-compras` · *Expiration:* lo que prefieras
5. *Repository access:* **Only select repositories** → marcar solo `Portal-compras-corporativas`
6. *Permissions → Repository permissions* → **Contents: Read and write**. Nada más.
7. **Generate token** y copiarlo (GitHub solo lo muestra una vez).

**Usarlo:**

1. Portal → pestaña **📂 Datos** → cargar el Excel
2. **⚙ Configurar publicación directa** → pegar usuario, repo y token → **📤 Publicar**
3. Listo. El token queda guardado en ese navegador; la próxima vez solo cargas y publicas.

> ⚠️ **El token nunca se escribe dentro de `index.html`.** Ese archivo es público en
> GitHub Pages y cualquiera podría leerlo. El portal solo lo guarda en el navegador de
> quien lo escribió (`localStorage`), nunca lo sube al repositorio. El botón
> **Olvidar token** lo borra de esa computadora.
>
> Si por accidente se expone un token, se revoca en la misma pantalla de GitHub donde se
> creó y se genera otro.

**Si sale `403 Resource not accessible by personal access token`:**

El token puede leer el repo pero no escribir. En orden de probabilidad:

1. **Contents está en `Read-only`.** Tiene que ser **Read and write**. Es la causa en la
   gran mayoría de los casos. Se corrige en el token existente — GitHub aplica el cambio
   de inmediato, no hace falta generar otro.
2. **Repository access** quedó en *Public Repositories (read-only)* en vez de
   **Only select repositories** con este repo marcado.
3. El repo es de una **organización**: un administrador tiene que aprobar el token en
   *Organization → Settings → Personal access tokens → Pending requests*.
4. El **usuario/dueño** está mal escrito en el portal.

El botón **🔍 Probar token** del portal verifica el token, el acceso al repo, la rama y
la lectura de contenidos, y dice cuál de estos falla. La escritura solo se puede
comprobar publicando de verdad.

Si se atora, el **camino A** (manual) deja exactamente el mismo resultado y no necesita
token.

### ¿Y si alguien más quiere revisar su propio Excel?

Cualquiera puede cargar un archivo en la pestaña **Datos**: lo ve solo en su computadora
como vista previa. No cambia lo que ve el equipo. Solo publicar (camino A o B) modifica
lo que todos ven, y eso requiere acceso al repositorio.

---

## Publicar el portal por primera vez

### 1. Crear el repositorio

1. https://github.com/new
2. **Repository name:** `Portal-compras-corporativas`
3. **Public** (Pages gratis requiere repo público)
4. Marcar **Add a README file** → **Create repository**

### 2. Subir los archivos

1. **Add file** → **Upload files**
2. Arrastrar `index.html` y `datos.json` (y este `README.md` si se quiere)
3. **Commit changes**

> `index.html` **debe** llamarse así y estar en la raíz del repo.

### 3. Prender GitHub Pages

1. **Settings** → menú izquierdo **Pages**
2. **Source:** `Deploy from a branch` · **Branch:** `main` · carpeta `/ (root)` → **Save**
3. Esperar 1–2 minutos. Queda en
   `https://ftrevinor001-tr.github.io/Portal-compras-corporativas/`

### 4. Enlazarlo desde el Portal de operaciones

Agregar un ítem al menú que apunte a esa URL.

---

## Qué necesita el archivo de Excel

El portal lee la hoja **`Stock_compras acumulado`** (es la que trae `Fecha`, y por eso
permite la tendencia). Si no existe, cae a `Stock_compras corporativas`.

Una fila por clave y por corte. Para agregar un corte nuevo se pegan las filas abajo con
la fecha del día — **no se borra lo anterior**, de ahí sale la tendencia.

| Bloque | Columnas |
|---|---|
| Identificación | `Fecha`, `ID`, `Descripción` |
| Inventario | `EXIUNIMIN` (existencia), `UNIDAD_MINIMA`, `Inventario objetivo`, `Tiempo de entrega` |
| Clasificación | `MARCA`, `SUBGRUPO`, `COMPRADOR` |
| Antecedente | `Consumo promedio Ant`, `Meses de inv. Ant.`, `Estatus Ant.` |
| Pronóstico | `Consumo promedio 2025`, `Meses de inv. Pron.`, `Estatus Pron.` |
| Opcionales | `Propuesta Ant.`, `Propuesta Pron.` — si no vienen, el portal las calcula |

Estatus válidos: `FALTANTE`, `RIESGO`, `OPTIMO`, `SOBRE INVENTARIO`, `SIN CONSUMO`
(no importan acentos ni mayúsculas).

### Propuesta de compra

Es cuánto pedir de cada clave, en su unidad mínima:

```
Propuesta = máximo(0, Inventario objetivo × Consumo promedio − Existencia)
```

Se calcula dos veces, una con el consumo del antecedente y otra con el del pronóstico.

Si el archivo trae las columnas `Propuesta Ant.` y `Propuesta Pron.` (hoy solo están en
`Stock_compras corporativas`), se usan tal cual. Si no vienen — el caso de
`Stock_compras acumulado`, que es la hoja que lee el portal — se calculan con la fórmula
de arriba. Verificado contra las 108 claves del Excel: coincide en todas, sin una sola
diferencia.

La pestaña **Datos** → *Columnas detectadas* dice cuál de los dos casos aplica en cada
carga: muestra el nombre de la columna, o `🧮 calculada`.

> **Pendiente conocido:** en `Stock_compras acumulado` las tres columnas del pronóstico se
> llaman `Columna1`, `Columna2` y `Columna3`. El portal las reconoce por posición, pero
> conviene renombrarlas: si se inserta una columna en medio, ese mapeo es lo primero que
> se rompe. La pestaña **Datos** marca con ⚠ los encabezados genéricos.

---

## Qué trae el tablero

- **Selector Antecedente / Pronóstico** — cambia con qué consumo se calcula el estatus y
  mueve todo el portal.
  - *Antecedente:* consumo real de los 4 meses anteriores. Es el default.
  - *Pronóstico:* consumo de los siguientes 4 meses, tomado del año anterior.
- **Aviso de divergencia** — claves que el antecedente da por sanas pero el pronóstico
  manda a riesgo o faltante, con botón para filtrarlas.
- **5 tarjetas KPI** clicables que filtran todo el tablero.
- **Tendencia por categoría** a través de los cortes del archivo.
- **Meses de inventario** con línea de objetivo.
- **Claves más críticas** por consumo promedio, con unidad mínima, marca y subgrupo.
- **Propuesta de compra** en la tabla, junto a Subgrupo: cuánto pedir según el
  antecedente y según el pronóstico. El botón **🛒 Sólo por comprar** filtra las claves
  con propuesta mayor a cero del escenario activo.
- **Tabla de detalle** con los dos escenarios lado a lado, filtros de categoría /
  comprador / marca / subgrupo, orden por columna y export a CSV.
- Modo claro / oscuro.

---

## Notas técnicas

- Un solo archivo `index.html`, sin build ni dependencias que instalar.
- El `.xlsx` completo pesa ~2.9 MB (por las hojas de Existencias y Productos);
  `datos.json` guarda solo la hoja acumulada: ~275 KB con 14 cortes, que GitHub sirve
  comprimida en ~15 KB. Crece unos 20 KB por corte nuevo (unos 1.5 KB ya comprimido),
  así que aguanta meses de historia sin problema.
- Usa [SheetJS](https://sheetjs.com) por CDN para leer el `.xlsx`, con respaldo automático
  a jsDelivr y unpkg. Si la red de la empresa bloquea los tres CDNs, descargar
  `xlsx.full.min.js`, ponerlo junto al `index.html` y cambiar el `src` del primer
  `<script>`. Nota: esto solo afecta la carga local de Excel — leer `datos.json` del repo
  no usa SheetJS y funciona aunque los CDNs estén bloqueados.
- El portal pide `datos.json` con `cache: no-store` y un parámetro de tiempo, para que no
  se quede con una versión vieja en caché.
