# Diseño: Layout 3 Columnas — PackCalc

**Fecha:** 2026-06-07  
**Archivo afectado:** `PackCalc.html` (único archivo del proyecto)  
**Objetivo:** Añadir un sidebar izquierdo con los contadores y el Pallet Breakdown, reducir el panel de Packing List de 38% a 26%.

---

## Contexto

El layout actual tiene 2 paneles:
- `panel-right` (`order:1`, `width:38%`) — visualmente a la izquierda: contadores, packing list, pallet breakdown, print summary
- `panel-left` (`order:2`, `flex:1`) — visualmente a la derecha: tabs de pallets + área de scan

El usuario identificó que el panel derecho (packing list) ocupa demasiado espacio y quiere que los contadores PALLETS/BOXES/PIECES y el PALLET BREAKDOWN estén en un sidebar izquierdo dedicado.

---

## Estructura final

```
┌─────────────┬─────────────────────────┬──────────────────┐
│   SIDEBAR   │         SCAN            │   PACKING LIST   │
│    ~22%     │        ~52%             │      ~26%        │
│             │                         │                  │
│ 1 PALLETS   │  [PLT 1] [+ PALLET]    │ ☰ HISTORY        │
│ 0 BOXES     │                         │ ↺ NEW SESSION    │
│ 0 PIECES    │  ┌─────────────────┐   │                  │
│             │  │  SCAN BOX HERE  │   │ PACKING LIST ─── │
│ BREAKDOWN   │  └─────────────────┘   │  PO _________    │
│ PLT 1  0pcs │                         │  PN _________    │
│    [TICKET] │  [01][02][03]...        │  DEL ________    │
│             │                         │  COO ________    │
│[PRINT SUMM] │                         │  QTY ________    │
│             │                         │                  │
│             │                         │ ⚠ INCOMPLETE     │
└─────────────┴─────────────────────────┴──────────────────┘
```

---

## Cambios en CSS

### Clases nuevas / modificadas

```css
/* Nuevo panel sidebar */
.panel-sidebar {
  order: 1;
  width: 22%;
  min-width: 220px;
  display: flex;
  flex-direction: column;
  border-right: 1px solid var(--border);
  overflow: hidden;
}

/* Panel central (era panel-left) — pierde order:2, pasa a order:2 igual */
.panel-left {
  order: 2;
  flex: 1;
  /* resto sin cambios */
}

/* Panel derecho (era panel-right) — ancho reducido */
.panel-right {
  order: 3;
  width: 26%;          /* antes: 38% */
  /* resto sin cambios */
}
```

---

## Movimiento de contenido

### Sale de `panel-right` → va a `panel-sidebar`

| Elemento | Nota |
|---|---|
| Grand counters (PALLETS / BOXES / PIECES) | Font-size reducida: 48px → ~32px para ajustarse al ancho |
| PALLET BREAKDOWN label + PLT rows + TICKET buttons | Igual que ahora |
| GRAND TOTAL row | Solo aparece cuando hay más de 1 pallet |
| PRINT SUMMARY — ALL PALLETS button | Al fondo del sidebar |

El sidebar tiene `overflow-y: auto` para scrollear si hay muchos pallets.

### Se queda en `panel-right` (sin cambios de contenido)

- Barra de controles: HISTORY, NEW SESSION, chip de estado
- Formulario PACKING LIST (PO, PN, DEL tags, COO tags, QTY)
- Sección FIELD VERIFICATION (condicional)
- Banner de estado COMPLETE / INCOMPLETE / OVER COUNT

### Panel scan (panel-left) — sin cambios de contenido

El contenido del área de scan permanece idéntico. Solo cambia su posición visual (pasa de ser el panel más a la derecha a estar en el centro).

---

## Detalles de implementación

### Contadores en el sidebar

Los números actuales usan `fontSize:48` con padding `10px 22px`. En el sidebar de 22% (~220–280px), deben reducirse para que los 3 quepan en una fila horizontal:

- `fontSize: 32` (era 48)
- Padding horizontal reducido: `10px 10px`
- El layout de los 3 stats sigue siendo una fila (`display:flex`)

### Sidebar scroll

El sidebar body (`overflowY:'auto'`) para que si hay 10+ pallets en el breakdown, el usuario pueda scrollear sin afectar el panel central.

### Borde izquierdo del sidebar

El `panel-right` actual tiene `border-right` porque está a la izquierda (order:1). El nuevo sidebar hereda esa misma lógica: `border-right: 1px solid var(--border)`.

El `panel-right` en order:3 no necesita `border-right` (es el último). Verificar que el borde no quede duplicado.

---

## Lo que NO cambia

- Todo el contenido del área de scan
- La lógica de pallet tabs
- El formulario de packing list (campos, comportamiento, tags)
- El sistema de impresión (printSummary, printList, printBubble)
- La barra de controles (HISTORY, NEW SESSION)
- localStorage, estado, callbacks — ningún JS cambia

---

## Criterio de éxito

1. El sidebar izquierdo muestra stats + breakdown visibles en todo momento
2. El área de scan ocupa el centro con más espacio que antes
3. El panel de packing list es más estrecho (~26%) pero todos sus campos siguen siendo usables
4. No hay contenido duplicado entre los paneles
5. El scroll del sidebar no afecta al panel central ni al derecho
