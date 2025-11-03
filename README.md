<!doctype html>
<html lang="fr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Darwin — Éditeur simple</title>
  <style>
    :root{--bg:#f3f4f6;--panel:#111827;--accent:#2563eb}
    body{margin:0;font-family:Inter, system-ui, Arial;background:var(--bg);color:#111}
    .app{display:flex;height:100vh}
    .sidebar{width:260px;background:white;border-right:1px solid #e5e7eb;padding:12px;box-sizing:border-box}
    .sidebar h2{margin:6px 0 12px;font-size:18px}
    .btn{display:inline-block;padding:8px 10px;border-radius:8px;background:var(--accent);color:white;text-decoration:none;cursor:pointer;margin:4px 0}
    .controls{margin-bottom:12px}
    .templates{display:flex;gap:8px;flex-wrap:wrap}
    .tpl{width:100px;height:64px;background:#f8fafc;border:1px dashed #cbd5e1;display:flex;align-items:center;justify-content:center;font-size:12px;cursor:pointer}
    .stageWrap{flex:1;display:flex;flex-direction:column;align-items:center;justify-content:flex-start;padding:18px}
    .stage{width:900px;height:600px;background:white;border:1px solid #ddd;position:relative;overflow:hidden;box-shadow:0 6px 18px rgba(15,23,42,0.06)}
    .element{position:absolute;touch-action:none;user-select:none}
    .textElement{min-width:80px;padding:6px 8px;font-size:18px;cursor:move;background:transparent}
    .selected{outline:2px dashed #2563eb}
    .handle{width:10px;height:10px;background:white;border:2px solid #2563eb;position:absolute;right:-8px;bottom:-8px;border-radius:50%;cursor:nwse-resize}
    .topbar{width:100%;display:flex;align-items:center;gap:12px;margin-bottom:12px}
    .toolbar{display:flex;gap:8px;align-items:center}
    input[type=color]{width:36px;height:36px;padding:0;border-radius:6px;border:1px solid #e5e7eb}
    select,input[type=number]{padding:6px;border-radius:6px;border:1px solid #e5e7eb}
    .props{width:240px;background:white;border-left:1px solid #e5e7eb;padding:12px;box-sizing:border-box}
    .row{margin-bottom:8px}
    .muted{color:#6b7280;font-size:13px}
    footer{position:fixed;right:12px;bottom:12px;font-size:12px;color:#374151}
    @media(max-width:1100px){.sidebar{display:none}.props{display:none}.stage{width:100%;height:500px}}
  </style>
</head>
<body>
  <div class="app">
    <aside class="sidebar">
      <h2>darwin</h2>
      <div class="controls">
        <div class="row"><button id="addText" class="btn">+ Ajouter texte</button></div>
        <div class="row"><label class="btn" for="imgFile">+ Ajouter image</label><input id="imgFile" type="file" accept="image/*" style="display:none"></div>
        <div class="row"><button id="removeBg" class="btn" style="background:#fbbf24;margin-top:6px">Supprimer arrière-plan</button></div>
        <div class="row"><button id="clearStage" class="btn" style="background:#ef4444">Effacer</button></div>
        <div class="row"><button id="download" class="btn" style="background:#10b981">Télécharger PNG</button></div>
      </div>
      <div>
        <div class="muted">Templates rapides</div>
        <div class="templates">
          <div class="tpl" data-tpl="title">Titre simple</div>
          <div class="tpl" data-tpl="poster">Poster</div>
          <div class="tpl" data-tpl="card">Carte</div>
        </div>
      </div>
      <hr style="margin:12px 0">
      <div class="muted">Astuces</div>
      <ul>
        <li>Cliquer un élément pour le sélectionner</li>
        <li>Glisser pour déplacer</li>
        <li>Utiliser la poignée pour redimensionner</li>
      </ul>
    </aside>

    <main class="stageWrap">
      <div class="topbar">
        <div class="toolbar">
          <label class="muted">Couleur</label>
          <input id="fillColor" type="color" value="#111827">
          <label class="muted">Taille</label>
          <input id="fontSize" type="number" value="24" min="8" max="200">
          <button id="bringForward" class="btn" style="background:#f59e0b">Avant</button>
          <button id="sendBack" class="btn" style="background:#9ca3af">Arrière</button>
        </div>
      </div>
      <div id="stage" class="stage" tabindex="0"></div>
    </main>

    <aside class="props">
      <div class="muted">Propriétés</div>
      <div class="row">Sélection: <span id="selName">Aucun</span></div>
      <div class="row">X: <input id="posX" type="number" style="width:80px"></div>
      <div class="row">Y: <input id="posY" type="number" style="width:80px"></div>
      <div class="row">Largeur: <input id="w" type="number" style="width:80px"></div>
      <div class="row">Hauteur: <input id="h" type="number" style="width:80px"></div>
    </aside>
  </div>

  <footer> </footerd

  <script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>
  <script>
    const stage = document.getElementById('stage');
    const addTextBtn = document.getElementById('addText');
    const imgFile = document.getElementById('imgFile');
    const fillColor = document.getElementById('fillColor');
    const fontSize = document.getElementById('fontSize');
    const downloadBtn = document.getElementById('download');
    const clearStage = document.getElementById('clearStage');
    const bringForward = document.getElementById('bringForward');
    const sendBack = document.getElementById('sendBack');
    const removeBgBtn = document.getElementById('removeBg');

    let selected = null;

    function makeTextBox(text="Double-clique pour modifier"){
      const el = document.createElement('div');
      el.className = 'element textElement';
      el.style.left = '40px';
      el.style.top = '40px';
      el.contentEditable = true;
      el.innerText = text;
      el.style.fontSize = fontSize.value + 'px';
      el.style.color = fillColor.value;
      addElement(el);
      return el;
    }

    function makeImage(url){
      const wrap = document.createElement('div');
      wrap.className = 'element';
      wrap.style.left = '60px';
      wrap.style.top = '60px';
      wrap.style.width = '240px';
      wrap.style.height = '160px';
      wrap.style.display = 'flex';
      wrap.style.alignItems = 'center';
      wrap.style.justifyContent = 'center';
      const img = document.createElement('img');
      img.src = url;
      img.style.maxWidth = '100%';
      img.style.maxHeight = '100%';
      img.draggable = false;
      wrap.appendChild(img);
      addElement(wrap);
      return wrap;
    }

    function addElement(el){
      el.style.position = 'absolute';
      el.dataset.z = getMaxZ()+1;
      el.style.zIndex = el.dataset.z;
      const h = document.createElement('div');
      h.className = 'handle';
      el.appendChild(h);
      stage.appendChild(el);
      makeDraggable(el);
      el.addEventListener('click', e=>{ e.stopPropagation(); selectElement(el); });
      el.addEventListener('dblclick', e=>{ e.stopPropagation(); if(el.classList.contains('textElement')){ placeCaretAtEnd(el); } });
      selectElement(el);
    }

    function getMaxZ(){
      const els = Array.from(stage.querySelectorAll('.element'));
      return els.reduce((m,e)=>Math.max(m, Number(e.dataset.z||0)), 0);
    }

    function selectElement(el){
      if(selected) selected.classList.remove('selected');
      selected = el;
      if(selected) selected.classList.add('selected');
      updatePropsPanel();
    }

    stage.addEventListener('click', ()=>{ if(selected){ selected.classList.remove('selected'); selected = null; updatePropsPanel(); } });

    function makeDraggable(el){
      let startX, startY, sx, sy, dragging=false, resizing=false;
      const handle = el.querySelector('.handle');
      handle.addEventListener('mousedown', e=>{
        e.stopPropagation(); resizing=true; startX = e.clientX; startY = e.clientY; sx = el.offsetWidth; sy = el.offsetHeight;
        document.addEventListener('mousemove', resize);
        document.addEventListener('mouseup', stop);
      });
      el.addEventListener('mousedown', e=>{
        e.stopPropagation(); dragging=true; startX = e.clientX; startY = e.clientY; sx = el.offsetLeft; sy = el.offsetTop;
        selectElement(el);
        document.addEventListener('mousemove', move);
        document.addEventListener('mouseup', stop);
      });
      function move(e){
        if(!dragging) return;
        const dx = e.clientX - startX; const dy = e.clientY - startY;
        el.style.left = Math.max(0, sx + dx) + 'px';
        el.style.top = Math.max(0, sy + dy) + 'px';
        updatePropsPanel();
      }
      function resize(e){
        if(!resizing) return;
        const dx = e.clientX - startX; const dy = e.clientY - startY;
        el.style.width = Math.max(20, sx + dx) + 'px';
        el.style.height = Math.max(20, sy + dy) + 'px';
        updatePropsPanel();
      }
      function stop(){ dragging=false; resizing=false; document.removeEventListener('mousemove', move); document.removeEventListener('mousemove', resize); document.removeEventListener('mouseup', stop); }
    }

    function placeCaretAtEnd(el){ el.focus(); if(typeof window.getSelection != "undefined" && typeof document.createRange != "undefined"){ var range = document.createRange(); range.selectNodeContents(el); range.collapse(false); var sel = window.getSelection(); sel.removeAllRanges(); sel.addRange(range); } }

    function updatePropsPanel(){
      const selName = document.getElementById('selName');
      const posX = document.getElementById('posX');
      const posY = document.getElementById('posY');
      const w = document.getElementById('w');
      const h = document.getElementById('h');
      if(!selected){ selName.innerText='Aucun'; posX.value=''; posY.value=''; w.value=''; h.value=''; return; }
      selName.innerText = selected.classList.contains('textElement')? 'Texte' : 'Image/Bloc';
      posX.value = parseInt(selected.style.left) || 0;
      posY.value = parseInt(selected.style.top) || 0;
      w.value = parseInt(selected.style.width) || selected.offsetWidth || 0;
      h.value = parseInt(selected.style.height) || selected.offsetHeight || 0;
    }

    ['posX','posY','w','h'].forEach(id=>{ document.getElementById(id).addEventListener('input', e=>{ if(!selected) return; const val = e.target.value; if(id==='posX') selected.style.left = val+'px'; if(id==='posY') selected.style.top = val+'px'; if(id==='w') selected.style.width = val+'px'; if(id==='h') selected.style.height = val+'px'; }); });
    fillColor.addEventListener('input', ()=>{ if(selected && selected.classList.contains('textElement')) selected.style.color = fillColor.value; });
    fontSize.addEventListener('input', ()=>{ if(selected && selected.classList.contains('textElement')) selected.style.fontSize = fontSize.value+'px'; });

    addTextBtn.addEventListener('click', ()=> makeTextBox());
    imgFile.addEventListener('change', (ev)=>{ const f = ev.target.files[0]; if(!f) return; const url = URL.createObjectURL(f); makeImage(url); });
    document.querySelectorAll('.tpl').forEach(t=> t.addEventListener('click', ()=>{ const type = t.dataset.tpl; if(type==='title'){ const a = makeTextBox('Ton super titre'); a.style.left='120px'; a.style.top='40px'; a.style.fontSize='48px'; } if(type==='poster'){ const bg = document.createElement('div'); bg.style.position='absolute'; bg.style.left='0'; bg.style.top='0'; bg.style.width='100%'; bg.style.height='100%'; bg.style.background='linear-gradient(135deg,#60a5fa,#a78bfa)'; bg.style.zIndex=0; bg.dataset.tpl='bg'; stage.appendChild(bg); } if(type==='card'){ const t1 = makeTextBox('Carte'); t1.style.left='80px'; t1.style.top='80px'; t1.style.fontSize='22px'; } }));

    window.addEventListener('keydown', e=>{ if(e.key==='Delete' && selected){ selected.remove(); selected=null; updatePropsPanel(); } });
    clearStage.addEventListener('click', ()=>{ if(confirm('Effacer tout le canvas ?')){ stage.querySelectorAll('.element').forEach(el=>el.remove()); } });
    bringForward.addEventListener('click', ()=>{ if(!selected) return; selected.dataset.z = Number(selected.dataset.z||0)+1; selected.style.zIndex = selected.dataset.z; });
    sendBack.addEventListener('click', ()=>{ if(!selected) return; selected.dataset.z = Math.max(0, Number(selected.dataset.z||0)-1); selected.style.zIndex = selected.dataset.z; });

    downloadBtn.addEventListener('click', ()=>{ document.querySelectorAll('.selected').forEach(s=>s.classList.remove('selected')); html2canvas(stage, {useCORS:true, backgroundColor: null}).then(canvas=>{ const link = document.createElement('a'); link.download = 'design.png'; link.href = canvas.toDataURL('image/png'); link.click(); }).finally(()=>{ if(selected) selected.classList.add('selected'); }); });

    removeBgBtn.addEventListener('click', async () => {
      if (!selected || !selected.querySelector('img')) { alert('Sélectionne d’abord une image !'); return; }
      const imgEl = selected.querySelector('img');
      const formData = new FormData();
      formData.append('image_url', imgEl.src);
      try {
        const response = await fetch('https://api.remove.bg/v1.0/removebg', {
          method: 'POST',
          headers: { 'X-Api-Key': 'qUEcteXFqaaishaBE8jKU7t' },
          body: formData
        });
        if (!response.ok) throw new Error('Erreur API remove.bg');
        const blob = await response.blob();
        imgEl.src = URL.createObjectURL(blob);
      } catch (err) { console.error(err); alert('Erreur lors de la suppression de l’arrière-plan.'); }
    });

    makeTextBox('darwin ');
  </script>
</body>
<!-- FOOTER: coller juste avant </body> -->
<footer id="site-footer" style="box-sizing:border-box;padding:18px 12px;text-align:center;border-top:1px solid rgba(0,0,0,0.08);font-family:system-ui,Segoe UI,Roboto,Arial,sans-serif;">
  <div style="max-width:900px;margin:0 auto;display:flex;gap:12px;align-items:center;justify-content:center;flex-wrap:wrap;">
    <p style="margin:0;font-size:14px;color:#444;">
      Contact :
      <span id="email-txt" style="font-weight:600;letter-spacing:0.2px;">
        darwineditor0@gmail.com
      </span>
    </p>
    <button id="copy-email" type="button" style="background:none;border:1px solid #ddd;padding:6px 10px;border-radius:8px;cursor:pointer;font-size:13px;">
      Copier
    </button>
  </div>
</footer>

<script>
  // Optionnel : bouton "Copier" (le mail reste affiché en texte, non cliquable)
  (function () {
    const btn = document.getElementById('copy-email');
    const emailSpan = document.getElementById('email-txt');

    if (!btn || !emailSpan) return;

    btn.addEventListener('click', function () {
      const text = emailSpan.textContent.trim();
      if (!navigator.clipboard) {
        // fallback
        const ta = document.createElement('textarea');
        ta.value = text;
        document.body.appendChild(ta);
        ta.select();
        try { document.execCommand('copy'); } catch (e) {}
        ta.remove();
        btn.textContent = 'Copié';
        setTimeout(()=> btn.textContent = 'Copier', 1500);
        return;
      }
      navigator.clipboard.writeText(text).then(() => {
        btn.textContent = 'Copié';
        setTimeout(()=> btn.textContent = 'Copier', 1500);
      }, () => {
        btn.textContent = 'Erreur';
        setTimeout(()=> btn.textContent = 'Copier', 1500);
      });
    });
  })();
</script>

