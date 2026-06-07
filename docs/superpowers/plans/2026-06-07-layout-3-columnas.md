# Layout 3 Columnas Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Añadir un sidebar izquierdo con stats + Pallet Breakdown, reducir el panel de Packing List de 38% a 26%, resultando en un layout de 3 columnas (sidebar 22% | scan flex:1 | packing list 26%).

**Architecture:** Un único archivo `PackCalc.html` contiene todo el CSS, JSX y lógica. El cambio es puramente de layout: (1) nuevas reglas CSS, (2) nuevo div `panel-sidebar` en el JSX, (3) mover bloques de contenido de `panel-right` al sidebar, (4) eliminar esos bloques de `panel-right`. No cambia ningún estado, callback ni lógica de negocio.

**Tech Stack:** React 18 via CDN + Babel standalone. Standalone HTML. No build step. Verificación abriendo el archivo en un navegador.

---

## Archivos

- Modificar: `PackCalc.html` (único archivo del proyecto)

---

### Task 1: Cambios en CSS

**Files:**
- Modify: `PackCalc.html:93-94`

- [ ] **Step 1: Leer las líneas CSS actuales para confirmar el contenido exacto**

Abrir `PackCalc.html` y verificar que las líneas 93-94 digan exactamente:
```
    .panel-left{order:2;flex:1;display:flex;flex-direction:column;overflow:hidden;}
    .panel-right{order:1;width:38%;display:flex;flex-direction:column;border-right:1px solid var(--border);overflow:hidden;}
```

- [ ] **Step 2: Reemplazar esas dos líneas con tres líneas**

Usar el Edit tool con:

```
old_string:
    .panel-left{order:2;flex:1;display:flex;flex-direction:column;overflow:hidden;}
    .panel-right{order:1;width:38%;display:flex;flex-direction:column;border-right:1px solid var(--border);overflow:hidden;}

new_string:
    .panel-sidebar{order:1;width:22%;min-width:220px;display:flex;flex-direction:column;border-right:1px solid var(--border);overflow:hidden;}
    .panel-left{order:2;flex:1;display:flex;flex-direction:column;overflow:hidden;}
    .panel-right{order:3;width:26%;display:flex;flex-direction:column;border-left:1px solid var(--border);overflow:hidden;}
```

Cambios clave:
- `.panel-sidebar` es nuevo: `order:1`, `width:22%`, `min-width:220px`, hereda el `border-right`
- `.panel-left`: sin cambios
- `.panel-right`: `order:1→3`, `width:38%→26%`, `border-right` cambia a `border-left` (ahora es el panel más a la derecha)

- [ ] **Step 3: Verificar visualmente en el navegador**

Abrir `PackCalc.html` en el navegador. El layout debe mostrar un espacio vacío a la izquierda (sidebar sin contenido aún), el scan al centro y el packing list más estrecho a la derecha. Si el layout se rompe, revisar que `order:1/2/3` estén correctos.

---

### Task 2: Insertar el div `panel-sidebar` en el JSX

**Files:**
- Modify: `PackCalc.html:931-934`

- [ ] **Step 1: Leer el bloque app-body actual para confirmar el contenido exacto**

Verificar que alrededor de la línea 931 diga:
```jsx
      <div className="app-body">
        {/* LEFT PANEL */}
        <div className="panel-left">
```

- [ ] **Step 2: Insertar el panel-sidebar antes del panel-left**

Usar el Edit tool:

```
old_string:
      <div className="app-body">
        {/* LEFT PANEL */}
        <div className="panel-left">

new_string:
      <div className="app-body">

        {/* SIDEBAR — Stats + Pallet Breakdown */}
        <div className="panel-sidebar">
          <div style={{flex:1,overflowY:'auto',padding:'16px 14px',display:'flex',flexDirection:'column'}}>

            {/* Grand counters — font reducida para sidebar estrecho */}
            <div style={{display:'flex',justifyContent:'center',marginBottom:16}}>
              <div style={{display:'flex',flexDirection:'column',alignItems:'center',padding:'10px 10px',borderRight:`1px solid ${C.border}`}}>
                <span style={{fontFamily:"'Chakra Petch',sans-serif",fontSize:32,lineHeight:1,color:C.text,letterSpacing:'.02em'}}>{pallets.length}</span>
                <span style={{fontFamily:"'Chakra Petch',sans-serif",fontSize:9,fontWeight:700,color:C.muted,letterSpacing:'.1em',marginTop:2}}>PALLETS</span>
              </div>
              <div style={{display:'flex',flexDirection:'column',alignItems:'center',padding:'10px 10px',borderRight:`1px solid ${C.border}`}}>
                <span style={{fontFamily:"'Chakra Petch',sans-serif",fontSize:32,lineHeight:1,color:C.amber,letterSpacing:'.02em'}}>{grandBoxes}</span>
                <span style={{fontFamily:"'Chakra Petch',sans-serif",fontSize:9,fontWeight:700,color:C.muted,letterSpacing:'.1em',marginTop:2}}>BOXES</span>
              </div>
              <div style={{display:'flex',flexDirection:'column',alignItems:'center',padding:'10px 10px'}}>
                <span style={{fontFamily:"'Chakra Petch',sans-serif",fontSize:32,lineHeight:1,color:C.green,letterSpacing:'.02em'}}>{grandPcs.toLocaleString()}</span>
                <span style={{fontFamily:"'Chakra Petch',sans-serif",fontSize:9,fontWeight:700,color:C.muted,letterSpacing:'.1em',marginTop:2}}>PIECES</span>
              </div>
            </div>

            {/* Pallet breakdown */}
            <div style={{fontSize:10,fontWeight:700,color:C.muted,letterSpacing:'.1em',marginBottom:8,fontFamily:"'Chakra Petch',sans-serif"}}>
              PALLET BREAKDOWN
            </div>
            {pallets.map((p,i)=>{
              const pPcs=p.boxes.reduce((s,b)=>s+b.qty,0);
              const isActive=i===idx;
              const pct=expPcs>0?Math.round(pPcs/expPcs*100):null;
              return(
                <div key={p.id} className="dlv-row"
                  style={{background:isActive?'rgba(255,183,0,.04)':C.s2,borderColor:isActive?C.amber:C.border}}>
                  <button onClick={()=>switchPallet(i)}
                    style={{background:'none',border:'none',cursor:'pointer',padding:0}}>
                    <span style={{fontFamily:"'JetBrains Mono',monospace",fontSize:13,fontWeight:800,color:isActive?C.amber:C.text}}>
                      PLT {i+1}
                    </span>
                  </button>
                  <span style={{fontSize:11,color:C.sec,fontFamily:"'Chakra Petch',sans-serif"}}>
                    {p.boxes.length} box{p.boxes.length!==1?'es':''}
                  </span>
                  <span style={{fontFamily:"'JetBrains Mono',monospace",fontSize:12,fontWeight:800,color:C.green}}>
                    {pPcs.toLocaleString()} pcs
                  </span>
                  {pct!==null&&(
                    <span style={{fontSize:10,color:C.muted,fontFamily:"'Chakra Petch',sans-serif"}}>{pct}%</span>
                  )}
                  <div style={{flex:1}}/>
                  <button onClick={()=>handlePrintPallet(p,i)} disabled={p.boxes.length===0}
                    style={{display:'flex',alignItems:'center',gap:5,padding:'6px 12px',borderRadius:6,
                      border:`1px solid ${p.boxes.length?C.border:C.s3}`,background:'transparent',
                      cursor:p.boxes.length?'pointer':'not-allowed',
                      color:p.boxes.length?C.text:C.muted,
                      fontSize:11,fontWeight:700,fontFamily:"'Chakra Petch',sans-serif",letterSpacing:'.05em'}}>
                    <Svg name="print" size={12} color={p.boxes.length?C.amber:C.muted}/>
                    TICKET
                  </button>
                </div>
              );
            })}

            {/* Grand total row */}
            {pallets.length>1&&(
              <div className="dlv-total-row" style={{marginTop:6}}>
                <span style={{fontSize:10,fontWeight:700,color:C.muted,letterSpacing:'.1em',fontFamily:"'Chakra Petch',sans-serif"}}>GRAND TOTAL</span>
                <div style={{flex:1}}/>
                <span style={{fontFamily:"'JetBrains Mono',monospace",fontSize:13,fontWeight:800,color:C.amber,marginRight:16}}>{grandBoxes} boxes</span>
                <span style={{fontFamily:"'JetBrains Mono',monospace",fontSize:13,fontWeight:800,color:C.green}}>{grandPcs.toLocaleString()} pcs</span>
              </div>
            )}

            {/* Print Summary button */}
            <button
              disabled={grandBoxes===0}
              onClick={handlePrintSummary}
              style={{
                display:'flex',alignItems:'center',justifyContent:'center',gap:8,
                width:'100%',marginTop:14,padding:'13px 0',
                borderRadius:8,border:`1px solid ${grandBoxes?C.borderHi:C.border}`,
                background:grandBoxes?C.s3:'transparent',
                cursor:grandBoxes?'pointer':'not-allowed',
                fontFamily:"'Chakra Petch',sans-serif",
                fontSize:14,fontWeight:800,letterSpacing:'.08em',
                color:grandBoxes?C.text:C.muted,
                minHeight:48,
              }}>
              <Svg name="print" size={15} color={grandBoxes?C.amber:C.muted}/>
              PRINT SUMMARY — ALL PALLETS
            </button>

          </div>
        </div> {/* end panel-sidebar */}

        {/* LEFT PANEL */}
        <div className="panel-left">
```

- [ ] **Step 3: Verificar en el navegador**

El sidebar izquierdo debe mostrar los contadores (números más pequeños), las filas de PLT y el botón PRINT SUMMARY. El contenido del scan debe estar al centro. El panel de packing list se verá a la derecha pero con contenido duplicado (breakdown + contadores) — eso se corrige en Task 3.

---

### Task 3: Eliminar contenido duplicado de `panel-right`

**Files:**
- Modify: `PackCalc.html` (sección panel-right, alrededor de líneas 1197–1390)

> **Nota:** Las líneas habrán cambiado respecto a las originales porque en Task 2 se insertaron ~90 líneas. Usar el contenido exacto como referencia, no los números de línea.

- [ ] **Step 1: Eliminar los grand counters de panel-right**

Buscar en `panel-right` el bloque de grand counters y eliminarlo. Usar el Edit tool:

```
old_string:
              {/* Grand counters */}
              <div style={{display:'flex',justifyContent:'center',marginBottom:20}}>
                <div style={{display:'flex',flexDirection:'column',alignItems:'center',padding:'10px 22px',borderRight:`1px solid ${C.border}`}}>
                  <span style={{fontFamily:"'Chakra Petch',sans-serif",fontSize:48,lineHeight:1,color:C.text,letterSpacing:'.02em'}}>{pallets.length}</span>
                  <span style={{fontFamily:"'Chakra Petch',sans-serif",fontSize:9,fontWeight:700,color:C.muted,letterSpacing:'.1em',marginTop:2}}>PALLETS</span>
                </div>
                <div style={{display:'flex',flexDirection:'column',alignItems:'center',padding:'10px 22px',borderRight:`1px solid ${C.border}`}}>
                  <span style={{fontFamily:"'Chakra Petch',sans-serif",fontSize:48,lineHeight:1,color:C.amber,letterSpacing:'.02em'}}>{grandBoxes}</span>
                  <span style={{fontFamily:"'Chakra Petch',sans-serif",fontSize:9,fontWeight:700,color:C.muted,letterSpacing:'.1em',marginTop:2}}>BOXES</span>
                </div>
                <div style={{display:'flex',flexDirection:'column',alignItems:'center',padding:'10px 22px'}}>
                  <span style={{fontFamily:"'Chakra Petch',sans-serif",fontSize:48,lineHeight:1,color:C.green,letterSpacing:'.02em'}}>{grandPcs.toLocaleString()}</span>
                  <span style={{fontFamily:"'Chakra Petch',sans-serif",fontSize:9,fontWeight:700,color:C.muted,letterSpacing:'.1em',marginTop:2}}>PIECES</span>
                </div>
              </div>

              {/* Packing list form */}

new_string:
              {/* Packing list form */}
```

- [ ] **Step 2: Eliminar el Pallet Breakdown + Grand Total + Print Summary de panel-right**

Buscar el bloque desde el comentario `{/* Pallet breakdown */}` hasta el cierre `</div>` de `width:'100%'`. Usar el Edit tool:

```
old_string:
              {/* Pallet breakdown */}
              <div style={{fontSize:10,fontWeight:700,color:C.muted,letterSpacing:'.1em',marginBottom:8,fontFamily:"'Chakra Petch',sans-serif"}}>
                PALLET BREAKDOWN
              </div>
              {pallets.map((p,i)=>{
                const pPcs=p.boxes.reduce((s,b)=>s+b.qty,0);
                const isActive=i===idx;
                const pct=expPcs>0?Math.round(pPcs/expPcs*100):null;
                return(
                  <div key={p.id} className="dlv-row"
                    style={{background:isActive?'rgba(255,183,0,.04)':C.s2,borderColor:isActive?C.amber:C.border}}>
                    <button onClick={()=>switchPallet(i)}
                      style={{background:'none',border:'none',cursor:'pointer',padding:0}}>
                      <span style={{fontFamily:"'JetBrains Mono',monospace",fontSize:13,fontWeight:800,color:isActive?C.amber:C.text}}>
                        PLT {i+1}
                      </span>
                    </button>
                    <span style={{fontSize:11,color:C.sec,fontFamily:"'Chakra Petch',sans-serif"}}>
                      {p.boxes.length} box{p.boxes.length!==1?'es':''}
                    </span>
                    <span style={{fontFamily:"'JetBrains Mono',monospace",fontSize:12,fontWeight:800,color:C.green}}>
                      {pPcs.toLocaleString()} pcs
                    </span>
                    {pct!==null&&(
                      <span style={{fontSize:10,color:C.muted,fontFamily:"'Chakra Petch',sans-serif"}}>{pct}%</span>
                    )}
                    <div style={{flex:1}}/>
                    <button onClick={()=>handlePrintPallet(p,i)} disabled={p.boxes.length===0}
                      style={{display:'flex',alignItems:'center',gap:5,padding:'6px 12px',borderRadius:6,
                        border:`1px solid ${p.boxes.length?C.border:C.s3}`,background:'transparent',
                        cursor:p.boxes.length?'pointer':'not-allowed',
                        color:p.boxes.length?C.text:C.muted,
                        fontSize:11,fontWeight:700,fontFamily:"'Chakra Petch',sans-serif",letterSpacing:'.05em'}}>
                      <Svg name="print" size={12} color={p.boxes.length?C.amber:C.muted}/>
                      TICKET
                    </button>
                  </div>
                );
              })}

              {/* Grand total row */}
              {pallets.length>1&&(
                <div className="dlv-total-row" style={{marginTop:6}}>
                  <span style={{fontSize:10,fontWeight:700,color:C.muted,letterSpacing:'.1em',fontFamily:"'Chakra Petch',sans-serif"}}>GRAND TOTAL</span>
                  <div style={{flex:1}}/>
                  <span style={{fontFamily:"'JetBrains Mono',monospace",fontSize:13,fontWeight:800,color:C.amber,marginRight:16}}>{grandBoxes} boxes</span>
                  <span style={{fontFamily:"'JetBrains Mono',monospace",fontSize:13,fontWeight:800,color:C.green}}>{grandPcs.toLocaleString()} pcs</span>
                </div>
              )}

              {/* Print Summary button */}
              <button
                disabled={grandBoxes===0}
                onClick={handlePrintSummary}
                style={{
                  display:'flex',alignItems:'center',justifyContent:'center',gap:8,
                  width:'100%',marginTop:14,padding:'13px 0',
                  borderRadius:8,border:`1px solid ${grandBoxes?C.borderHi:C.border}`,
                  background:grandBoxes?C.s3:'transparent',
                  cursor:grandBoxes?'pointer':'not-allowed',
                  fontFamily:"'Chakra Petch',sans-serif",
                  fontSize:14,fontWeight:800,letterSpacing:'.08em',
                  color:grandBoxes?C.text:C.muted,
                  minHeight:48,
                }}>
                <Svg name="print" size={15} color={grandBoxes?C.amber:C.muted}/>
                PRINT SUMMARY — ALL PALLETS
              </button>

            </div>
          </div>
        </div> {/* end panel-right */}

new_string:
            </div>
          </div>
        </div> {/* end panel-right */}
```

---

### Task 4: Verificación final y commit

**Files:**
- Verify: `PackCalc.html` (browser)

- [ ] **Step 1: Abrir en el navegador y verificar el layout**

Checklist visual:
- [ ] Sidebar izquierdo visible con contadores PALLETS/BOXES/PIECES (números ~32px)
- [ ] PALLET BREAKDOWN en el sidebar con filas PLT y botones TICKET
- [ ] Botón PRINT SUMMARY al fondo del sidebar
- [ ] Si hay más de 1 pallet, GRAND TOTAL aparece en el sidebar
- [ ] El sidebar scrollea si hay muchos pallets (agregar 5+ pallets para probar)
- [ ] Panel central (scan) con tabs y área de scan — sin cambios de comportamiento
- [ ] Panel derecho más estrecho con solo: controles (HISTORY/NEW SESSION), PACKING LIST, FIELD VERIFICATION, banner de estado
- [ ] NO aparece contenido duplicado en el panel derecho (sin contadores ni breakdown)
- [ ] Hacer click en "PLT 2" en el sidebar activa el pallet correcto (highlight en tab y en fila del sidebar)
- [ ] El botón TICKET del sidebar dispara la impresión del pallet correcto

- [ ] **Step 2: Commit**

```bash
git add PackCalc.html
git commit -m "Refactor layout to 3 columns: add sidebar with stats and pallet breakdown"
```
