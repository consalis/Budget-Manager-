<html lang="de">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover" />
  <title>Budget Manager — Responsive</title>
  <meta name="description" content="Responsive Budget Manager: Desktop & Mobile — lokal, offlinefähig, CSV-Import/Export" />
  <style>
    /* ---------- Variables & Base (mobile-first) ---------- */
    :root{
      --bg:#0f1113;
      --card: rgba(255,255,255,0.03);
      --muted: rgba(255,255,255,0.6);
      --glass-border: rgba(255,255,255,0.06);
      --accent-start: #7f60ff;
      --accent-end: #00c8ff;
      --success: #26c281;
      --danger: #ff7979;
      --radius:14px;
      --gap:12px;
      font-family: -apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,'Helvetica Neue',Arial;
      color-scheme: dark;
    }

    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0; background: radial-gradient(800px 450px at 10% 10%, rgba(127,96,255,0.08), transparent),
                      radial-gradient(600px 350px at 90% 90%, rgba(0,200,255,0.035), transparent),
                      var(--bg);
      color:#fff; -webkit-font-smoothing:antialiased; -moz-osx-font-smoothing:grayscale;
      padding:16px; display:flex; align-items:flex-start; justify-content:center; gap:16px;
    }

    /* App shell */
    .app{
      width:100%; max-width:1200px; display:flex; gap:16px; flex-direction:column;
    }

    header{display:flex; align-items:center; justify-content:space-between; gap:12px}
    .brand{display:flex; gap:12px; align-items:center}
    .logo{width:44px;height:44px;border-radius:12px;background:linear-gradient(135deg,var(--accent-start),var(--accent-end));display:grid;place-items:center;font-weight:700}
    h1{font-size:16px;margin:0}
    p.lead{margin:0;color:var(--muted);font-size:13px}

    /* Layout */
    .content{display:grid; grid-template-columns: 1fr; gap:16px}

    /* Cards */
    .card{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); border:1px solid var(--glass-border); padding:14px; border-radius:var(--radius); backdrop-filter: blur(8px) saturate(120%); box-shadow:0 8px 30px rgba(0,0,0,0.6)}

    /* Top summary row */
    .summary{display:flex; gap:12px; overflow:auto}
    .stat{flex:1; min-width:140px}
    .label{font-size:12px;color:var(--muted)}
    .value{font-weight:700;font-size:18px}

    /* Form */
    form{display:grid; gap:10px}
    input,select,button{font:inherit}
    .input-row{display:grid; grid-template-columns: 1fr auto; gap:8px}
    input[type=text], input[type=number], select{padding:10px;border-radius:10px;border:1px solid rgba(255,255,255,0.04); background:transparent; color:inherit}
    .btn{padding:10px 12px;border-radius:10px;border:0;background:linear-gradient(90deg,var(--accent-start),var(--accent-end)); color:#021;font-weight:700;cursor:pointer}
    .muted{color:var(--muted);font-size:13px}

    /* Transactions list */
    .tx-list{display:flex;flex-direction:column;gap:8px; max-height:420px; overflow:auto}
    .tx{display:flex;justify-content:space-between;align-items:center;padding:10px;border-radius:10px;border:1px solid rgba(255,255,255,0.03)}
    .tx .left{display:flex;gap:8px;align-items:center}
    .avatar{width:44px;height:44;border-radius:10px;display:grid;place-items:center;background:linear-gradient(135deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.03)}
    .amount{font-weight:800}
    .amount.in{color:var(--success)}
    .amount.out{color:var(--danger)}

    /* Chart */
    .chart-wrap{height:220px}
    canvas{width:100% !important; height:100% !important}

    /* Small controls */
    .controls{display:flex;gap:8px;flex-wrap:wrap}
    .chip{padding:8px 10px;border-radius:999px;background:rgba(255,255,255,0.03);border:1px solid rgba(255,255,255,0.04);cursor:pointer}

    /* Footer helpers */
    footer{display:flex;justify-content:space-between;align-items:center;color:var(--muted);font-size:13px}

    /* Desktop layout */
    @media(min-width:900px){
      .content{grid-template-columns: 360px 1fr; align-items:start}
      .summary{flex-direction:column}
      .summary .stat{min-width:unset}
    }

    /* Accessibility focus */
    :focus{outline:3px solid rgba(127,96,255,0.18);outline-offset:2px}

    /* small tweaks for very small screens */
    @media(max-width:420px){
      body{padding:10px}
      .logo{width:40px;height:40px}
    }
  </style>
</head>
<body>
  <main class="app" id="app">
    <header>
      <div class="brand">
        <div class="logo">BM</div>
        <div>
          <h1>Budget Manager</h1>
          <p class="lead">Desktop &amp; Mobile — lokal | Offline</p>
        </div>
      </div>

      <div class="controls">
        <button id="importBtn" class="chip">Import CSV</button>
        <button id="exportBtn" class="chip">Export JSON</button>
        <button id="clearBtn" class="chip">Alle löschen</button>
      </div>
    </header>

    <section class="content">
      <!-- SIDEBAR / FORM -->
      <aside class="card" aria-label="Neuer Eintrag">
        <div class="summary" aria-hidden>
          <div class="stat">
            <div class="label">Kontostand</div>
            <div class="value" id="balance">€0,00</div>
          </div>
          <div class="stat">
            <div class="label">Einnahmen (30d)</div>
            <div class="value" id="income">€0,00</div>
          </div>
          <div class="stat">
            <div class="label">Ausgaben (30d)</div>
            <div class="value" id="expense">€0,00</div>
          </div>
        </div>

        <form id="entryForm" autocomplete="off">
          <label class="muted">Titel</label>
          <input id="title" required placeholder="z. B. Miete, Gehalt" />

          <label class="muted">Betrag (€)</label>
          <div class="input-row">
            <input id="amount" type="number" step="0.01" placeholder="0.00" required />
            <select id="type" aria-label="Typ">
              <option value="out">Ausgabe</option>
              <option value="in">Einnahme</option>
            </select>
          </div>

          <label class="muted">Kategorie</label>
          <select id="category">
            <option>Allgemein</option>
            <option>Miete</option>
            <option>Lebensmittel</option>
            <option>Transport</option>
            <option>Freizeit</option>
            <option>Gehalt</option>
          </select>

          <div style="display:flex;gap:8px;margin-top:6px">
            <button class="btn" type="submit">Hinzufügen</button>
            <button type="button" id="resetForm" class="chip">Zurücksetzen</button>
          </div>
        </form>

        <div style="margin-top:10px;color:var(--muted);font-size:13px">
          Tip: Tippe auf einen Eintrag, um ihn zu bearbeiten.
        </div>
      </aside>

      <!-- MAIN -->
      <main>
        <div class="card">
          <div style="display:flex;align-items:center;justify-content:space-between;gap:12px">
            <div style="display:flex;gap:12px;align-items:center;flex:1">
              <input id="search" placeholder="Transaktionen durchsuchen..." style="flex:1;padding:10px;border-radius:10px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:inherit" />
              <div class="chip" id="filterMonth">30 Tage</div>
              <div class="chip" id="groupByCat">Nach Kategorie</div>
            </div>
          </div>

          <div style="margin-top:12px;" class="chart-wrap card-chart">
            <canvas id="chart"></canvas>
          </div>
        </div>

        <div class="card" style="margin-top:12px">
          <h3 style="margin:0 0 8px 0">Letzte Transaktionen</h3>
          <div id="txList" class="tx-list" role="list" aria-live="polite"></div>
        </div>

        <footer style="margin-top:10px">
          <div class="muted">Daten werden lokal im Browser gespeichert.</div>
          <div class="muted">Made with ♥ — Offline-ready</div>
        </footer>
      </main>
    </section>
  </main>

  <!-- Hidden file input for CSV import -->
  <input id="fileInput" type="file" accept="text/csv,application/json" style="display:none" />

  <script>
    // ---------- Utilities ----------
    const $ = (sel) => document.querySelector(sel);
    const txKey = 'budget_manager_v1';
    let txns = JSON.parse(localStorage.getItem(txKey) || '[]');

    function fmt(n){ return n.toLocaleString('de-DE', {style:'currency', currency:'EUR'}); }
    function save(){ localStorage.setItem(txKey, JSON.stringify(txns)); }

    // ---------- Elements ----------
    const form = $('#entryForm');
    const txList = $('#txList');
    const balanceEl = $('#balance');
    const incomeEl = $('#income');
    const expenseEl = $('#expense');
    const chartCanvas = $('#chart');
    const fileInput = $('#fileInput');

    // ---------- CRUD + UI ----------
    function renderList(filter=''){
      txList.innerHTML='';
      const list = txns.slice().reverse().filter(t=> (t.title + ' ' + t.category).toLowerCase().includes(filter.toLowerCase()));
      if(list.length===0){ txList.innerHTML = '<div style="color:var(--muted);padding:12px;border-radius:8px">Keine Einträge</div>'; return; }
      list.forEach(t=>{
        const item = document.createElement('div'); item.className='tx'; item.tabIndex=0; item.role='listitem';
        item.innerHTML = `
          <div class="left">
            <div class="avatar">${t.category[0]||'•'}</div>
            <div>
              <div style="font-weight:700">${escapeHtml(t.title)}</div>
              <div class="muted" style="font-size:13px">${escapeHtml(t.category)} • ${new Date(t.date).toLocaleString()}</div>
            </div>
          </div>
          <div style="display:flex;flex-direction:column;align-items:flex-end;gap:6px">
            <div class="amount ${t.type==='in'?'in':'out'}">${t.type==='in'?'+':'-'}${fmt(Number(t.amount))}</div>
            <div style="display:flex;gap:8px">
              <button class="chip edit" data-id="${t.id}">Bearbeiten</button>
              <button class="chip del" data-id="${t.id}">Löschen</button>
            </div>
          </div>
        `;
        txList.appendChild(item);
      });
    }

    function escapeHtml(s){ return String(s).replace(/[&<>"']/g, m=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":"&#039;"}[m])); }

    function recalc(){
      const income = txns.filter(t=>t.type==='in').reduce((s,t)=>s+Number(t.amount),0);
      const expense = txns.filter(t=>t.type==='out').reduce((s,t)=>s+Number(t.amount),0);
      const bal = income - expense;
      incomeEl.textContent = fmt(income);
      expenseEl.textContent = fmt(expense);
      balanceEl.textContent = fmt(bal);
    }

    // ---------- Form handlers ----------
    form.addEventListener('submit', e=>{
      e.preventDefault();
      const title = form.title.value.trim();
      const amount = Math.abs(Number(form.amount.value) || 0).toFixed(2);
      const type = form.type.value;
      const category = form.category.value || 'Allgemein';
      if(!title || !amount) return;
      const entry = { id: Date.now().toString(36), title, amount, type, category, date: new Date().toISOString() };
      txns.push(entry); save(); recalc(); renderList($('#search').value); form.reset(); updateChart();
    });

    document.addEventListener('click', e=>{
      if(e.target.classList.contains('del')){
        const id = e.target.dataset.id; if(confirm('Eintrag löschen?')){ txns = txns.filter(t=>t.id!==id); save(); recalc(); renderList($('#search').value); updateChart(); }
      }
      if(e.target.classList.contains('edit')){
        const id = e.target.dataset.id; const t = txns.find(x=>x.id===id); if(!t) return;
        form.title.value = t.title; form.amount.value = t.amount; form.type.value = t.type; form.category.value = t.category;
        txns = txns.filter(x=>x.id!==id); save(); recalc(); renderList($('#search').value); updateChart();
      }
    });

    $('#resetForm').addEventListener('click', ()=> form.reset());
    $('#clearBtn').addEventListener('click', ()=>{ if(confirm('Alle Transaktionen löschen?')){ txns=[]; save(); recalc(); renderList(); updateChart(); } });

    // Search
    $('#search').addEventListener('input', e=> renderList(e.target.value));

    // Export JSON
    $('#exportBtn').addEventListener('click', ()=>{
      const blob = new Blob([JSON.stringify(txns, null, 2)], {type:'application/json'});
      const url = URL.createObjectURL(blob); const a = document.createElement('a'); a.href = url; a.download = 'budget-export.json'; a.click(); URL.revokeObjectURL(url);
    });

    // CSV Import/Export (simple)
    $('#importBtn').addEventListener('click', ()=> fileInput.click());
    fileInput.addEventListener('change', async (e)=>{
      const file = e.target.files[0]; if(!file) return; const text = await file.text();
      if(file.type.includes('json') || file.name.endsWith('.json')){
        try{ const data = JSON.parse(text); if(Array.isArray(data)){ txns = txns.concat(data.map(normalizeImported)); save(); recalc(); renderList(); updateChart(); alert('JSON importiert'); } }
        catch(err){ alert('JSON konnte nicht gelesen werden'); }
      } else {
        // parse CSV (very small parser: Titel;Betrag;Typ;Kategorie;Datum)
        const rows = text.split(/
?
/).map(r=>r.trim()).filter(Boolean);
        const parsed = rows.map(r=>{
          const cols = r.split(/[,;	]/).map(c=>c.trim());
          return { title: cols[0]||'Import', amount: Math.abs(Number(cols[1]||0)).toFixed(2), type: (cols[2]||'out'), category: cols[3]||'Allgemein', date: cols[4]||new Date().toISOString() };
        });
        txns = txns.concat(parsed.map(normalizeImported)); save(); recalc(); renderList(); updateChart(); alert('CSV importiert');
      }
      fileInput.value='';
    });

    function normalizeImported(t){ return { id: Date.now().toString(36) + Math.random().toString(36).slice(2,6), title:String(t.title||'Import'), amount: (Number(t.amount)||0).toFixed(2), type: (t.type==='in'?'in':'out'), category: t.category||'Allgemein', date: t.date||new Date().toISOString() } }

    // ---------- Simple chart (Canvas) ----------
    const ctx = chartCanvas.getContext('2d');
    function updateChart(){
      const days = 30; const now = new Date();
      // create array of days
      const buckets = Array.from({length:days}, (_,i)=>({date:new Date(now.getFullYear(), now.getMonth(), now.getDate() - (days-1-i)), value:0}));
      txns.forEach(t=>{
        const d = new Date(t.date); const dayIndex = Math.floor((d - buckets[0].date)/(1000*60*60*24));
        if(dayIndex>=0 && dayIndex<days) buckets[dayIndex].value += (t.type==='in'?1:-1)*Number(t.amount);
      });
      // cumulative
      let cum=0; const cumArr = buckets.map(b=>cum+=b.value);

      const dpr = devicePixelRatio || 1;
      const w = chartCanvas.width = chartCanvas.clientWidth * dpr;
      const h = chartCanvas.height = chartCanvas.clientHeight * dpr;
      ctx.clearRect(0,0,w,h);

      // background
      const g = ctx.createLinearGradient(0,0,0,h); g.addColorStop(0,'rgba(255,255,255,0.02)'); g.addColorStop(1,'rgba(255,255,255,0.01)'); ctx.fillStyle=g; ctx.fillRect(0,0,w,h);

      const min = Math.min(...cumArr,0); const max = Math.max(...cumArr,100); const range = (max-min)||1;
      ctx.lineWidth = 3 * dpr;
      ctx.beginPath();
      cumArr.forEach((v,i)=>{
        const x = (i/(days-1)) * w; const y = h - ((v-min)/range) * h; if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
      });
      const sg = ctx.createLinearGradient(0,0,w,0); sg.addColorStop(0,'rgba(127,96,255,0.95)'); sg.addColorStop(1,'rgba(0,200,255,0.95)'); ctx.strokeStyle = sg; ctx.stroke();
      // fill under
      ctx.lineTo(w,h); ctx.lineTo(0,h); ctx.closePath(); const fg = ctx.createLinearGradient(0,0,0,h); fg.addColorStop(0,'rgba(127,96,255,0.12)'); fg.addColorStop(1,'rgba(0,200,255,0.02)'); ctx.fillStyle=fg; ctx.fill();
    }

    // ---------- Init ----------
    recalc(); renderList(); updateChart();
    window.addEventListener('resize', updateChart);

    // ---------- Small UX improvements ----------
    // Keyboard: Enter on list item edits first item
    txList.addEventListener('keydown', e=>{
      if(e.key==='Enter'){
        const first = txList.querySelector('.edit'); if(first) first.click();
      }
    });

  </script>
</body>
</html>
