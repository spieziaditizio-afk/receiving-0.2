# Packing List COO with Quantities Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reorder packing list fields (PO → DEL → PN → COO) and change COO from a list of strings to a list of `{code, qty}` entries so the total expected pieces is auto-calculated from COO quantities instead of entered manually.

**Architecture:** Single file `PackCalc.html` (React 18 via CDN + Babel standalone). Changes touch: state shape, two `expPcs` computations, `plCrossCheck` logic, `plHtml` print template, form UI JSX, reset handler, and localStorage migration. No new files. No build step.

**Tech Stack:** React 18 (CDN), Babel standalone, vanilla HTML/CSS. No automated tests — verification is opening the file in a browser.

---

## Files

- Modify: `PackCalc.html` (único archivo)

---

### Task 1: State shape, expPcs, reset, localStorage migration

**Files:**
- Modify: `PackCalc.html` (lines ~651, ~662, ~680, ~710, ~369, ~784, ~888)

**Context:** `packingList.coo` changes from `string[]` to `{code:string, qty:number}[]`. The `qty` field is removed from `packingList`. Two new local state vars (`cooCode`, `cooQty`) and two refs (`cooCodeRef`, `cooQtyRef`) are added for the COO entry inputs. `expPcs` is now derived from the sum of COO quantities. `addCooEntry` helper is added with the other handlers.

- [ ] **Step 1: Change packingList state and add cooCode/cooQty state**

Find (line ~651):
```js
  const [packingList,  setPackingList] = useState({ po:'', del:[], pn:'', coo:[], qty:'' });
```
Replace with:
```js
  const [packingList,  setPackingList] = useState({ po:'', del:[], pn:'', coo:[] });
  const [cooCode,      setCooCode]     = useState('');
  const [cooQty,       setCooQty]      = useState('');
```

- [ ] **Step 2: Add cooCodeRef and cooQtyRef**

Find (line ~662):
```js
  const inputRef = useRef();
```
Replace with:
```js
  const inputRef   = useRef();
  const cooCodeRef = useRef();
  const cooQtyRef  = useRef();
```

- [ ] **Step 3: Update localStorage migration**

Find (lines ~680–681):
```js
        if(s.packingList) setPackingList(p=>({...p,...s.packingList}));
        else if(s.expectedPcs!=null) setPackingList(p=>({...p,qty:s.expectedPcs}));
```
Replace with:
```js
        if(s.packingList){
          const pl={...s.packingList};
          if(Array.isArray(pl.coo)&&pl.coo.length&&typeof pl.coo[0]==='string')
            pl.coo=pl.coo.map(code=>({code,qty:0}));
          delete pl.qty;
          setPackingList(p=>({...p,...pl}));
        }
```

- [ ] **Step 4: Update expPcs computation in the React component**

Find (line ~710):
```js
  const expPcs      = parseInt(packingList.qty)||0;
```
Replace with:
```js
  const expPcs      = packingList.coo.reduce((s,c)=>s+(c.qty||0),0);
```

- [ ] **Step 5: Update expPcs computation inside printSummary function**

Find (line ~369, inside `const printSummary = ...`):
```js
  const expPcs = parseInt(packingList.qty) || 0;
```
Replace with:
```js
  const expPcs = packingList.coo.reduce((s,c)=>s+(c.qty||0),0);
```

- [ ] **Step 6: Add addCooEntry handler**

Find (line ~785):
```js
  const doFlash    = ()=>{ setFlash(true); setFlashKey(k=>k+1); setTimeout(()=>setFlash(false),700); };
  const resetDraft = ()=>{ setDraft({}); setStepIdx(0); setInput(''); setTimeout(()=>inputRef.current?.focus(),40); };
```
Replace with:
```js
  const doFlash    = ()=>{ setFlash(true); setFlashKey(k=>k+1); setTimeout(()=>setFlash(false),700); };
  const resetDraft = ()=>{ setDraft({}); setStepIdx(0); setInput(''); setTimeout(()=>inputRef.current?.focus(),40); };

  const addCooEntry = () => {
    const code = cooCode.trim().toUpperCase();
    const qty  = parseInt(cooQty);
    if(!code || !(qty > 0)) return;
    setPackingList(p=>({
      ...p,
      coo: p.coo.some(c=>c.code===code)
        ? p.coo.map(c=>c.code===code ? {...c,qty} : c)
        : [...p.coo, {code,qty}],
    }));
    setCooCode(''); setCooQty('');
    setTimeout(()=>cooCodeRef.current?.focus(), 40);
  };
```

- [ ] **Step 7: Update handleNewSession reset**

Find (line ~888):
```js
    setPackingList({po:'',del:[],pn:'',coo:[],qty:''});
```
Replace with:
```js
    setPackingList({po:'',del:[],pn:'',coo:[]});
    setCooCode(''); setCooQty('');
```

- [ ] **Step 8: Verify in browser**

Open `PackCalc.html`. The packing list panel should still render (form may look slightly off — the COO field still uses the old TagField at this point). No console errors. The app should load without crashing.

- [ ] **Step 9: Commit**

```bash
git add PackCalc.html
git commit -m "Refactor packingList state: coo becomes {code,qty}[], remove qty field"
```

---

### Task 2: Fix plCrossCheck and plHtml print template

**Files:**
- Modify: `PackCalc.html` (lines ~773–776 and ~392–400)

**Context:** `plCrossCheck` uses `pl.coo.includes(v)` which breaks with the new object format. `plHtml` uses `packingList.coo.join(' · ')` and `packingList.qty` which both need updating.

- [ ] **Step 1: Fix plCrossCheck COO check**

Find (lines ~773–776):
```js
    if(pl.coo.length){
      const vals=[...new Set(allBoxes.map(b=>b.coo).filter(Boolean))];
      const bad=vals.filter(v=>!pl.coo.includes(v));
      chk.coo={label:'COO',expected:pl.coo,actual:vals,ok:bad.length===0,unexpected:bad};
    }
```
Replace with:
```js
    if(pl.coo.length){
      const coosCodes=pl.coo.map(c=>c.code);
      const vals=[...new Set(allBoxes.map(b=>b.coo).filter(Boolean))];
      const bad=vals.filter(v=>!coosCodes.includes(v));
      chk.coo={label:'COO',expected:coosCodes,actual:vals,ok:bad.length===0,unexpected:bad};
    }
```

- [ ] **Step 2: Fix plHtml in printSummary**

Find (lines ~392–400):
```js
  const plHtml = (packingList.po || packingList.pn || packingList.del.length || packingList.coo.length || packingList.qty) ? `
    <div class="pl-block">
      <div class="section-lbl">PACKING LIST</div>
      <div class="pl-grid">
        ${packingList.po  ? `<div class="pl-field"><span class="pl-lbl">PO</span><span class="pl-val">${packingList.po}</span></div>` : ''}
        ${packingList.pn  ? `<div class="pl-field"><span class="pl-lbl">PN</span><span class="pl-val">${packingList.pn}</span></div>` : ''}
        ${packingList.del.length ? `<div class="pl-field"><span class="pl-lbl">DEL</span><span class="pl-val">${packingList.del.join(' · ')}</span></div>` : ''}
        ${packingList.coo.length ? `<div class="pl-field"><span class="pl-lbl">COO</span><span class="pl-val">${packingList.coo.join(' · ')}</span></div>` : ''}
        ${packingList.qty ? `<div class="pl-field"><span class="pl-lbl">TOTAL QTY</span><span class="pl-val">${parseInt(packingList.qty).toLocaleString()} pcs</span></div>` : ''}
      </div>
    </div>` : '';
```
Replace with:
```js
  const plHtml = (packingList.po || packingList.pn || packingList.del.length || packingList.coo.length || expPcs>0) ? `
    <div class="pl-block">
      <div class="section-lbl">PACKING LIST</div>
      <div class="pl-grid">
        ${packingList.po  ? `<div class="pl-field"><span class="pl-lbl">PO</span><span class="pl-val">${packingList.po}</span></div>` : ''}
        ${packingList.del.length ? `<div class="pl-field"><span class="pl-lbl">DEL</span><span class="pl-val">${packingList.del.join(' · ')}</span></div>` : ''}
        ${packingList.pn  ? `<div class="pl-field"><span class="pl-lbl">PN</span><span class="pl-val">${packingList.pn}</span></div>` : ''}
        ${packingList.coo.length ? `<div class="pl-field"><span class="pl-lbl">COO</span><span class="pl-val">${packingList.coo.map(c=>`${c.code} (${c.qty.toLocaleString()} pcs)`).join(' · ')}</span></div>` : ''}
        ${expPcs>0 ? `<div class="pl-field"><span class="pl-lbl">TOTAL QTY</span><span class="pl-val">${expPcs.toLocaleString()} pcs</span></div>` : ''}
      </div>
    </div>` : '';
```

- [ ] **Step 3: Verify in browser**

Open `PackCalc.html`. Add a COO entry via the browser console to test:
```js
// Paste in browser console to test plCrossCheck with new format
// (actual UI form is replaced in Task 3 — this verifies the logic)
```
Confirm no console errors. The FIELD VERIFICATION section should still appear when boxes are scanned that have COO values.

- [ ] **Step 4: Commit**

```bash
git add PackCalc.html
git commit -m "Fix plCrossCheck and plHtml for new COO {code,qty} format"
```

---

### Task 3: Replace packing list form UI

**Files:**
- Modify: `PackCalc.html` (lines ~1290–1331)

**Context:** Remove the old 2-column grid (PO+PN) and the old COO TagField + QTY input. Replace with: PO (full width) → DEL TagField → PN (full width) → COO entry UI (code input + qty input + ADD button + list of entries + auto-total). The `addCooEntry`, `cooCode`, `cooQty`, `cooCodeRef`, `cooQtyRef` variables are all defined in Task 1.

- [ ] **Step 1: Replace the packing list form JSX**

Find (lines ~1290–1331) — use the exact text to match:
```jsx
              {/* Packing list form */}
              <div style={{marginBottom:14,padding:'14px 16px',borderRadius:9,border:`1px solid ${C.border}`,background:C.s2}}>
                <div style={{fontSize:9,fontWeight:700,color:C.amber,letterSpacing:'.1em',marginBottom:12,fontFamily:"'Chakra Petch',sans-serif"}}>
                  PACKING LIST
                </div>
                <div style={{display:'grid',gridTemplateColumns:'1fr 1fr',gap:10,marginBottom:10}}>
                  <div>
                    {fldLbl('PO — PURCHASE ORDER')}
                    <input value={packingList.po}
                      onChange={e=>setPackingList(p=>({...p,po:e.target.value.toUpperCase()}))}
                      placeholder="Scan or type PO…"
                      className="field-inp"/>
                  </div>
                  <div>
                    {fldLbl('PN — PART NUMBER')}
                    <input value={packingList.pn}
                      onChange={e=>setPackingList(p=>({...p,pn:e.target.value.toUpperCase()}))}
                      placeholder="Scan or type PN…"
                      className="field-inp"/>
                  </div>
                </div>
                <div style={{marginBottom:10}}>
                  <TagField label="DEL — DELIVERY NUMBERS (one or more)"
                    values={packingList.del}
                    onChange={v=>setPackingList(p=>({...p,del:v}))}
                    placeholder="Scan DEL barcode then Enter to add…"/>
                </div>
                <div style={{marginBottom:10}}>
                  <TagField label="COO — COUNTRIES OF ORIGIN (one or more)"
                    values={packingList.coo}
                    onChange={v=>setPackingList(p=>({...p,coo:v}))}
                    placeholder="Scan COO barcode then Enter to add…"/>
                </div>
                <div>
                  {fldLbl('TOTAL EXPECTED PIECES')}
                  <input value={packingList.qty}
                    onChange={e=>setPackingList(p=>({...p,qty:e.target.value.replace(/\D/g,'')}))}
                    placeholder="Expected pieces from packing list…"
                    className="field-inp" inputMode="numeric" type="number" min="1"
                    style={{fontSize:17,fontWeight:800,height:50}}/>
                </div>
              </div>
```

Replace with:
```jsx
              {/* Packing list form */}
              <div style={{marginBottom:14,padding:'14px 16px',borderRadius:9,border:`1px solid ${C.border}`,background:C.s2}}>
                <div style={{fontSize:9,fontWeight:700,color:C.amber,letterSpacing:'.1em',marginBottom:12,fontFamily:"'Chakra Petch',sans-serif"}}>
                  PACKING LIST
                </div>
                <div style={{marginBottom:10}}>
                  {fldLbl('PO — PURCHASE ORDER')}
                  <input value={packingList.po}
                    onChange={e=>setPackingList(p=>({...p,po:e.target.value.toUpperCase()}))}
                    placeholder="Scan or type PO…"
                    className="field-inp"/>
                </div>
                <div style={{marginBottom:10}}>
                  <TagField label="DEL — DELIVERY NUMBERS (one or more)"
                    values={packingList.del}
                    onChange={v=>setPackingList(p=>({...p,del:v}))}
                    placeholder="Scan DEL barcode then Enter to add…"/>
                </div>
                <div style={{marginBottom:10}}>
                  {fldLbl('PN — PART NUMBER')}
                  <input value={packingList.pn}
                    onChange={e=>setPackingList(p=>({...p,pn:e.target.value.toUpperCase()}))}
                    placeholder="Scan or type PN…"
                    className="field-inp"/>
                </div>
                <div>
                  {fldLbl('COO — COUNTRIES OF ORIGIN')}
                  <div style={{display:'flex',gap:6,marginBottom:8}}>
                    <input
                      ref={cooCodeRef}
                      value={cooCode}
                      onChange={e=>setCooCode(e.target.value.toUpperCase())}
                      onKeyDown={e=>{if((e.key==='Enter'||e.key==='Tab')&&cooCode.trim()){e.preventDefault();cooQtyRef.current?.focus();}}}
                      placeholder="COO code…"
                      className="field-inp"
                      style={{flex:1}}
                    />
                    <input
                      ref={cooQtyRef}
                      value={cooQty}
                      onChange={e=>setCooQty(e.target.value.replace(/\D/g,''))}
                      onKeyDown={e=>{if(e.key==='Enter'){e.preventDefault();addCooEntry();}}}
                      placeholder="Pcs"
                      className="field-inp"
                      inputMode="numeric"
                      type="number"
                      min="1"
                      style={{width:80}}
                    />
                    <button onClick={addCooEntry}
                      style={{display:'flex',alignItems:'center',justifyContent:'center',padding:'0 12px',
                        borderRadius:6,border:`1px solid ${C.border}`,background:'transparent',
                        cursor:'pointer',color:C.amber,fontWeight:800,fontFamily:"'Chakra Petch',sans-serif",
                        fontSize:12,letterSpacing:'.05em',minHeight:38,flexShrink:0}}>
                      + ADD
                    </button>
                  </div>
                  {packingList.coo.map((entry,i)=>(
                    <div key={i} style={{display:'flex',alignItems:'center',gap:8,padding:'5px 8px',
                      marginBottom:4,borderRadius:5,background:C.s3,border:`1px solid ${C.border}`}}>
                      <span style={{fontFamily:"'JetBrains Mono',monospace",fontSize:12,fontWeight:800,color:C.text,flex:1}}>
                        {entry.code} · {entry.qty.toLocaleString()} pcs
                      </span>
                      <button onClick={()=>setPackingList(p=>({...p,coo:p.coo.filter((_,j)=>j!==i)}))}
                        style={{background:'none',border:'none',cursor:'pointer',color:C.muted,fontSize:16,lineHeight:1,padding:'0 2px'}}>
                        ×
                      </button>
                    </div>
                  ))}
                  {packingList.coo.length>0&&(
                    <div style={{display:'flex',justifyContent:'flex-end',paddingTop:6,borderTop:`1px solid ${C.border}`,marginTop:4}}>
                      <span style={{fontFamily:"'Chakra Petch',sans-serif",fontSize:10,fontWeight:700,color:C.amber,letterSpacing:'.08em'}}>
                        TOTAL: {expPcs.toLocaleString()} pcs
                      </span>
                    </div>
                  )}
                </div>
              </div>
```

- [ ] **Step 2: Verify in browser — golden path**

Open `PackCalc.html` and test:
- [ ] Formulario muestra: PO → DEL → PN → COO (en ese orden)
- [ ] El campo QTY ya no aparece
- [ ] Escribir `NL` en código COO + Tab/Enter → foco pasa al campo Pcs
- [ ] Escribir `1200` en Pcs + Enter → aparece fila `NL · 1,200 pcs` + total `TOTAL: 1,200 pcs`
- [ ] Añadir segunda entrada `DE / 350` → total pasa a `1,550 pcs`
- [ ] Botón `×` en una fila la elimina y el total se recalcula
- [ ] Si se añade el mismo código COO dos veces → se reemplaza la cantidad (no duplica)
- [ ] El status banner (COMPLETE/INCOMPLETE) usa el total de COO como expected

- [ ] **Step 3: Verify FIELD VERIFICATION with COO**

Escanear una caja con COO = `NL`. Con `NL` en la lista COO del packing list → verificación muestra ✓. Con un COO diferente → muestra ✗.

- [ ] **Step 4: Commit**

```bash
git add PackCalc.html
git commit -m "Replace packing list form: new field order PO>DEL>PN>COO with qty entries"
```
