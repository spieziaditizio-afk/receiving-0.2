# Diseño: Packing List — COO con cantidades

**Fecha:** 2026-06-07  
**Archivo afectado:** `PackCalc.html` (único archivo del proyecto)  
**Objetivo:** Reordenar los campos del formulario de Packing List (PO → DEL → PN → COO) y cambiar el campo COO de lista de códigos a lista de entradas `{code, qty}`, eliminando el campo QTY manual y calculando automáticamente el total esperado de piezas.

---

## Contexto

Un packing list tiene:
- **PO** — un Purchase Order
- **DEL** — uno o más números de entrega
- **PN** — Part Number, siempre el mismo para todas las deliveries del PO
- **COO** — uno o más países de origen, cada uno con su cantidad de piezas

La suma de piezas de todos los COO es el total de piezas esperadas del packing list. Actualmente este total se introduce manualmente en el campo QTY; con este cambio se calcula automáticamente.

El COO puede escanearse (si la etiqueta de caja tiene barcode) o escribirse a mano desde el papel.

---

## Cambio de modelo de datos

### `packingList` state

**Antes:**
```js
{ po: '', del: [], pn: '', coo: [], qty: '' }
// coo: string[]       — lista de códigos
// qty: string         — total piezas esperadas (manual)
```

**Después:**
```js
{ po: '', del: [], pn: '', coo: [] }
// coo: { code: string, qty: number }[]  — lista de entradas con cantidad
// qty: eliminado
```

### `expPcs` — piezas esperadas

**Antes:** `parseInt(packingList.qty) || 0`

**Después:** `packingList.coo.reduce((s, c) => s + (c.qty || 0), 0)`

Este cambio se aplica en **dos lugares**:
1. Línea ~710 en el componente React (estado derivado)
2. Línea ~369 dentro de la función `printSummary` (para el print HTML)

---

## UI del formulario

### Orden de campos (todos full-width, sin grid 2 columnas)

```
PO ___________________________
DEL  [NL123] [BE456]  +
PN ___________________________
COO — COUNTRIES OF ORIGIN
[ código ]  [ qty  ]  [+ ADD]
─────────────────────────────
NL  ·  1,200 pcs  [×]
DE  ·    350 pcs  [×]
─────────────────────────────
TOTAL: 1,550 pcs
```

### COO entry inputs

Dos inputs inline:
- **code input**: texto uppercase, placeholder `"COO code…"`, escaneable con barcode scanner
- **qty input**: numérico, placeholder `"Pcs"`, ancho fijo (~80px)
- **botón `+ ADD`**: añade la entrada si ambos inputs son válidos

**Interacción de teclado/scanner:**
- En code input → `Enter` o `Tab` → foco pasa a qty input
- En qty input → `Enter` → añade la entrada, limpia ambos inputs, foco vuelve a code input
- En qty input → `Tab` → si ambos inputs válidos, añade la entrada y foca code input; si no, comportamiento estándar

**Validación de entrada:**
- `code` debe ser non-empty (trim)
- `qty` debe ser entero positivo > 0
- Si ya existe un entry con el mismo code, reemplaza la cantidad (no duplica)

### Lista de entradas COO

Cada entrada muestra: `{CODE} · {qty.toLocaleString()} pcs [×]`

El botón `[×]` elimina la entrada.

### Total auto-calculado

Debajo de la lista:
```
TOTAL: 1,550 pcs
```
Visible solo cuando hay al menos una entrada COO. Sustituye visualmente al campo QTY que se elimina.

---

## Estado local nuevo

Dos variables de estado locales al componente (no en `packingList`):

```js
const [cooCode, setCooCode] = useState('');
const [cooQty, setCooQty] = useState('');
```

Función helper `addCooEntry()`:
- Valida que `cooCode.trim()` y `parseInt(cooQty) > 0`
- Si existe entry con mismo code → reemplaza qty
- Si no existe → añade al final
- Limpia ambos inputs y enfoca el code input

---

## Impacto en otras partes del código

### `plCrossCheck` — verificación COO (línea ~773)

**Antes:**
```js
if(pl.coo.length){
  const vals=[...new Set(allBoxes.map(b=>b.coo).filter(Boolean))];
  const bad=vals.filter(v=>!pl.coo.includes(v));
  chk.coo={label:'COO',expected:pl.coo,actual:vals,ok:bad.length===0,unexpected:bad};
}
```

**Después:**
```js
if(pl.coo.length){
  const coosCodes = pl.coo.map(c=>c.code);
  const vals=[...new Set(allBoxes.map(b=>b.coo).filter(Boolean))];
  const bad=vals.filter(v=>!coosCodes.includes(v));
  chk.coo={label:'COO',expected:coosCodes,actual:vals,ok:bad.length===0,unexpected:bad};
}
```

### `plHtml` en print templates (línea ~392-400)

**Condición de presencia** (`packingList.qty` → `expPcs > 0`):
```js
// antes: packingList.qty
// después: expPcs > 0  (expPcs calculado igual que en el componente)
```

**Línea COO** (`.join(' · ')` → mostrar code + qty):
```js
// antes:
packingList.coo.join(' · ')
// después:
packingList.coo.map(c=>`${c.code} (${c.qty.toLocaleString()} pcs)`).join(' · ')
```

**Línea TOTAL QTY** (calcular en lugar de leer `qty`):
```js
// antes: parseInt(packingList.qty).toLocaleString()
// después: expPcs.toLocaleString()
// (expPcs calculado desde pl.coo.reduce dentro de printSummary)
```

### Reset de sessión (línea ~888)

**Antes:** `setPackingList({po:'',del:[],pn:'',coo:[],qty:''})`  
**Después:** `setPackingList({po:'',del:[],pn:'',coo:[]})`

Y también limpiar los inputs locales COO:
```js
setCooCode('');
setCooQty('');
```

### localStorage — migración al cargar (línea ~680)

Al cargar sesión guardada, `packingList.coo` podría ser el formato antiguo (`string[]`). Migración defensiva:

```js
if(s.packingList){
  const pl = {...s.packingList};
  // Migrar coo: string[] → {code, qty}[]
  if(Array.isArray(pl.coo) && pl.coo.length && typeof pl.coo[0]==='string'){
    pl.coo = pl.coo.map(code=>({code, qty:0}));
  }
  // Eliminar qty obsoleto
  delete pl.qty;
  setPackingList(p=>({...p,...pl}));
}
```

---

## Lo que NO cambia

- El TagField component para DEL — sin cambios
- La lógica de cross-check para PO, PN, DEL — sin cambios
- El estado de las cajas escaneadas (`boxes`) — sin cambios
- El sidebar (stats + pallet breakdown) — sin cambios
- Los templates de impresión `printList` y `printBubble` — sin cambios (no usan `packingList.coo` ni `qty`)

---

## Criterio de éxito

1. El formulario muestra los campos en orden: PO → DEL → PN → COO con qty → total
2. El operador puede escanear o escribir el código COO, tabular al qty, pulsar Enter para añadir
3. El total de piezas esperadas se calcula automáticamente sumando los qty de los COO
4. El campo QTY manual ya no aparece en el formulario
5. La verificación de campos (FIELD VERIFICATION) sigue comparando códigos COO correctamente
6. El print summary muestra correctamente los COO con sus cantidades y el total
7. Sesiones antiguas del localStorage no causan errores (migración defensiva)
