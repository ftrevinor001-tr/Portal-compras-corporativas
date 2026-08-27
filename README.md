# Portal de Compras Corporativas

Tablero de una sola página para monitorear las claves de insumos corporativos.

**En vivo:** https://ftrevinor001-tr.github.io/Portal-compras-corporativas/

El repositorio guarda dos archivos:

| Archivo | Qué es |
|---|---|
| `index.html` | El portal. Se toca solo cuando hay cambios de funcionalidad. |
| `datos.json` | Los datos del último corte (~57 KB). **Este es el que se reemplaza cada vez que hay un corte nuevo.** |

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

Estatus válidos: `FALTANTE`, `RIESGO`, `OPTIMO`, `SOBRE INVENTARIO`, `SIN CONSUMO`
(no importan acentos ni mayúsculas).

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
- **Tabla de detalle** con los dos escenarios lado a lado, filtros de categoría /
  comprador / marca / subgrupo, orden por columna y export a CSV.
- Modo claro / oscuro.

---

## Notas técnicas

- Un solo archivo `index.html`, sin build ni dependencias que instalar.
- El `.xlsx` completo pesa ~2.9 MB (por las hojas de Existencias y Productos);
  `datos.json` guarda solo lo que el tablero usa: ~57 KB, y GitHub lo sirve comprimido
  en ~6 KB. Por eso el portal abre al instante.
- Usa [SheetJS](https://sheetjs.com) por CDN para leer el `.xlsx`, con respaldo automático
  a jsDelivr y unpkg. Si la red de la empresa bloquea los tres CDNs, descargar
  `xlsx.full.min.js`, ponerlo junto al `index.html` y cambiar el `src` del primer
  `<script>`. Nota: esto solo afecta la carga local de Excel — leer `datos.json` del repo
  no usa SheetJS y funciona aunque los CDNs estén bloqueados.
- El portal pide `datos.json` con `cache: no-store` y un parámetro de tiempo, para que no
  se quede con una versión vieja en caché.
