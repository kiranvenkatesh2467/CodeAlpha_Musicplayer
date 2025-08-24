# CodeAlpha_Musicplayer

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Advanced Music Player</title>
  <style>
    :root{
      --bg:#0f1115; --panel:#151923; --panel-2:#1b2130; --accent:#6ee7b7; --accent-2:#60a5fa; --text:#e5e7eb; --muted:#9ca3af;
      --danger:#ef4444; --warn:#f59e0b; --radius:16px;
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0; font-family:ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Noto Sans, Ubuntu, Cantarell, Helvetica Neue, Arial, "Apple Color Emoji","Segoe UI Emoji";
      background:linear-gradient(180deg, #0b0d12, #0f1115 40%, #0f1115); color:var(--text);
      display:flex; align-items:center; justify-content:center; padding:20px;
    }
    .app{width:min(1100px, 96vw); background:var(--panel); border-radius:var(--radius); box-shadow:0 10px 40px rgba(0,0,0,.5); overflow:hidden; border:1px solid #202636}

    .topbar{display:flex; align-items:center; gap:12px; padding:14px 16px; background:var(--panel-2); border-bottom:1px solid #202636}
    .brand{display:flex; align-items:center; gap:10px; font-weight:700; letter-spacing:.3px}
    .logo{width:28px; height:28px; border-radius:8px; background:linear-gradient(135deg, var(--accent), var(--accent-2)); display:grid; place-items:center; box-shadow:0 4px 12px rgba(0,0,0,.35)}
    .logo svg{width:18px; height:18px; fill:#0b0d12}
    .spacer{flex:1}
    .btn{appearance:none; border:0; background:#222838; color:var(--text); padding:10px 14px; border-radius:12px; cursor:pointer; display:inline-flex; align-items:center; gap:8px; font-weight:600}
    .btn:hover{filter:brightness(1.08)}
    .btn.secondary{background:#1f2433}
    .btn.ghost{background:transparent; border:1px solid #2a3144}
    .btn.danger{background:#3a1e1e; color:#fecaca; border:1px solid #4a2323}
    .btn svg{width:18px; height:18px; fill:currentColor}
    input[type="file"]{display:none}

    .layout{display:grid; grid-template-columns: 360px 1fr; gap:0; min-height:560px}
    @media (max-width: 920px){ .layout{grid-template-columns:1fr;}}

    /* Playlist */
    .sidebar{border-right:1px solid #202636; background:linear-gradient(180deg, #121624, #131829)}
    .side-head{display:flex; align-items:center; gap:8px; padding:12px}
    .search{flex:1; display:flex; background:#111520; border:1px solid #202636; border-radius:12px; overflow:hidden}
    .search input{flex:1; background:transparent; border:0; outline:none; color:var(--text); padding:10px 12px}
    .search .icon{display:grid; place-items:center; padding:0 10px}

    .playlist{max-height:calc(100% - 56px); overflow:auto; padding:8px}
    .track{display:grid; grid-template-columns: 24px 1fr 70px; gap:10px; align-items:center; padding:10px; border-radius:12px; border:1px solid #222a3d; background:#0f1422; margin:6px 0}
    .track.dragging{opacity:.6;}
    .track .title{font-weight:600; white-space:nowrap; overflow:hidden; text-overflow:ellipsis}
    .track .meta{font-size:12px; color:var(--muted)}
    .track .actions{display:flex; gap:6px; justify-content:flex-end}
    .badge{font-size:11px; color:#0b0d12; background:var(--accent); padding:2px 8px; border-radius:999px; font-weight:700}
    .handle{cursor:grab; opacity:.7}

    /* Player */
    .player{padding:18px; display:flex; flex-direction:column; gap:16px}
    .now{display:flex; align-items:center; gap:14px}
    .cover{width:68px; height:68px; border-radius:16px; background:linear-gradient(135deg, #2a3144, #1b2130); display:grid; place-items:center; border:1px solid #2a3144}
    .cover svg{width:36px; height:36px; opacity:.8}
    .now .info{min-width:0}
    .now .title{font-size:18px; font-weight:800}
    .now .artist{font-size:13px; color:var(--muted)}

    .controls{display:flex; align-items:center; gap:8px; flex-wrap:wrap}
    .circle{width:42px; height:42px; border-radius:50%; border:1px solid #2a3144; background:#12172a; display:grid; place-items:center; cursor:pointer}
    .circle.big{width:58px; height:58px}
    .toggle.active{background:linear-gradient(135deg, #172036, #19243d); box-shadow:inset 0 0 0 2px #2a3144}

    .progress{display:grid; grid-template-columns:48px 1fr 48px; gap:8px; align-items:center}
    .bar{height:10px; background:#0e1322; border:1px solid #2a3144; border-radius:999px; position:relative}
    .bar .fill{position:absolute; left:0; top:0; bottom:0; width:0%; background:linear-gradient(90deg, var(--accent), var(--accent-2)); border-radius:999px}
    .bar .thumb{position:absolute; top:50%; transform:translate(-50%, -50%); width:16px; height:16px; border-radius:50%; background:#fff; opacity:.9; display:none}
    .bar:hover .thumb{display:block}
    .time{font-variant-numeric:tabular-nums; font-size:12px; color:var(--muted)}

    .sliders{display:flex; align-items:center; gap:14px; flex-wrap:wrap}
    input[type="range"]{accent-color:var(--accent);}

    .footer{display:flex; justify-content:space-between; gap:10px; align-items:center; padding:12px 16px; border-top:1px solid #202636; background:var(--panel-2); font-size:12px; color:var(--muted)}

    .empty{padding:24px; text-align:center; color:var(--muted)}
    .dropzone{border:2px dashed #2a3144; border-radius:16px; padding:22px; text-align:center; background:#0c1120}
    .kbd{font-family:ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; border:1px solid #2a3144; background:#111520; padding:2px 6px; border-radius:6px}
  </style>
</head>
<body>
  <div class="app" id="app">
    <div class="topbar">
      <div class="brand"><span class="logo" title="Audio"><svg viewBox="0 0 24 24"><path d="M3 9v6a6 6 0 0 0 12 0V7h2a4 4 0 0 1 4 4v6h-2v-6a2 2 0 0 0-2-2h-2v6a8 8 0 0 1-16 0V9h4z"/></svg></span> Advanced Music Player</div>
      <div class="spacer"></div>
      <label class="btn">
        <svg viewBox="0 0 24 24"><path d="M12 5v14m-7-7h14" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round"/></svg>
        Add Files
        <input id="fileInput" type="file" accept="audio/*" multiple />
      </label>
      <button class="btn ghost" id="addUrlBtn" title="Add audio by URL">
        <svg viewBox="0 0 24 24"><path d="M7 17l5-5-5-5" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round"/><path d="M12 17l5-5-5-5" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round"/></svg>
        Add URL
      </button>
      <button class="btn secondary" id="clearBtn" title="Clear playlist">
        <svg viewBox="0 0 24 24"><path d="M3 6h18M8 6l1 14h6l1-14M10 6V4h4v2" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round"/></svg>
        Clear
      </button>
    </div>

    <div class="layout">
      <!-- Sidebar / Playlist -->
      <aside class="sidebar">
        <div class="side-head">
          <div class="search">
            <span class="icon"><svg width="16" height="16" viewBox="0 0 24 24"><path d="M10 18a8 8 0 1 1 5.293-14.293A8 8 0 0 1 10 18zm7-1 5 5" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round"/></svg></span>
            <input id="search" placeholder="Search songs or artists…" />
          </div>
        </div>
        <div id="playlist" class="playlist" aria-label="Playlist"></div>
      </aside>

      <!-- Player Panel -->
      <main class="player">
        <div id="drop" class="dropzone" hidden>
        
          <span class="badge" id="badge">STOPPED</span>
        </div>

        <div class="controls">
          <button class="circle" id="prevBtn" title="Previous (P)"><svg viewBox="0 0 24 24"><path d="M7 6v12M20 18l-10-6 10-6v12z" fill="currentColor"/></svg></button>
          <button class="circle big" id="playBtn" title="Play/Pause (Space)"><svg id="playIcon" viewBox="0 0 24 24"><path d="M8 5v14l11-7-11-7z" fill="currentColor"/></svg></button>
          <button class="circle" id="nextBtn" title="Next (N)"><svg viewBox="0 0 24 24"><path d="M17 6v12M4 18l10-6L4 6v12z" fill="currentColor"/></svg></button>

          <button class="circle toggle" id="shuffleBtn" title="Shuffle (S)"><svg viewBox="0 0 24 24"><path d="M17 3h4v4M3 7h6l8 10h4M21 17v4h-4M3 17h6l2.5-3" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round"/></svg></button>
          <button class="circle toggle" id="repeatBtn" title="Repeat (R)"><svg viewBox="0 0 24 24"><path d="M17 1l4 4-4 4M3 5h12a4 4 0 0 1 0 8H7m0 10l-4-4 4-4m14 0H12a4 4 0 0 1 0-8h8" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round"/></svg></button>
        </div>

        <div class="progress">
          <div class="time" id="currentTime">0:00</div>
          <div class="bar" id="progressBar" role="slider" aria-valuemin="0" aria-valuemax="100" aria-label="Seek">
            <div class="fill" id="progressFill"></div>
            <div class="thumb" id="progressThumb"></div>
          </div>
          <div class="time" id="duration">0:00</div>
        </div>

        <div class="sliders">
          <label>Speed
            <input id="speed" type="range" min="0.5" max="2" step="0.05" value="1" />
          </label>
          <label>Volume
            <input id="volume" type="range" min="0" max="1" step="0.01" value="1" />
          </label>
          <label>Jump (sec)
            <input id="jump" type="range" min="2" max="30" step="1" value="5" />
          </label>
        </div>

        <div id="emptyState" class="empty">
          <p><strong>Your playlist is empty.</strong></p>
          <p>Drag & drop audio files anywhere or click <em>Add Files</em>. You can also add a direct audio URL.</p>
          <p>Shortcuts: <span class="kbd">Space</span> Play/Pause · <span class="kbd">N</span>/<span class="kbd">P</span> Next/Prev · <span class="kbd">S</span> Shuffle · <span class="kbd">R</span> Repeat · <span class="kbd">←/→</span> Seek · <span class="kbd">↑/↓</span> Volume</p>
        </div>
      </main>
    </div>

    <div class="footer">
      <div>Session playlist is saved in memory while this tab is open. URL tracks persist via localStorage.</div>
      <div>Built with vanilla HTML/CSS/JS.</div>
    </div>
  </div>

  <template id="trackTemplate">
    <div class="track" draggable="true">
      <div class="handle" title="Drag to reorder">
        <svg width="20" height="20" viewBox="0 0 20 20"><path d="M7 4h2v2H7V4zm4 0h2v2h-2V4zM7 9h2v2H7V9zm4 0h2v2h-2V9zM7 14h2v2H7v-2zm4 0h2v2h-2v-2z" fill="currentColor"/></svg>
      </div>
      <div>
        <div class="title"></div>
        <div class="meta"></div>
      </div>
      <div class="actions">
        <button class="btn ghost playOne" title="Play"><svg viewBox="0 0 24 24"><path d="M8 5v14l11-7-11-7z" fill="currentColor"/></svg></button>
        <button class="btn ghost replaceOne" title="Replace file"><svg viewBox="0 0 24 24"><path d="M12 5v14m-7-7h14" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round"/></svg></button>
        <button class="btn danger removeOne" title="Remove"><svg viewBox="0 0 24 24"><path d="M3 6h18M8 6l1 14h6l1-14M10 6V4h4v2" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round"/></svg></button>
      </div>
    </div>
  </template>

  <input id="hiddenFile" type="file" accept="audio/*" hidden />

  <script>
    // --- State ---
    const audio = new Audio();
    audio.preload = 'metadata';

    let playlist = []; // {id, title, artist, src, kind:'blob'|'url', duration}
    let queue = []; // indices to play next when shuffle is on
    let current = -1;
    let isShuffle = false;
    let repeatMode = 'off'; // 'off' | 'one' | 'all'

    // --- Elements ---
    const els = {
      fileInput: document.getElementById('fileInput'),
      addUrlBtn: document.getElementById('addUrlBtn'),
      clearBtn: document.getElementById('clearBtn'),
      playlist: document.getElementById('playlist'),
      search: document.getElementById('search'),
      nowTitle: document.getElementById('nowTitle'),
      nowArtist: document.getElementById('nowArtist'),
      badge: document.getElementById('badge'),
      playBtn: document.getElementById('playBtn'),
      playIcon: document.getElementById('playIcon'),
      prevBtn: document.getElementById('prevBtn'),
      nextBtn: document.getElementById('nextBtn'),
      shuffleBtn: document.getElementById('shuffleBtn'),
      repeatBtn: document.getElementById('repeatBtn'),
      progressBar: document.getElementById('progressBar'),
      progressFill: document.getElementById('progressFill'),
      progressThumb: document.getElementById('progressThumb'),
      currentTime: document.getElementById('currentTime'),
      duration: document.getElementById('duration'),
      speed: document.getElementById('speed'),
      volume: document.getElementById('volume'),
      jump: document.getElementById('jump'),
      drop: document.getElementById('drop'),
      emptyState: document.getElementById('emptyState'),
      hiddenFile: document.getElementById('hiddenFile'),
    };

    // --- Utils ---
    const uid = () => crypto.randomUUID ? crypto.randomUUID() : Math.random().toString(36).slice(2);
    const fmt = (s) => {
      if (!isFinite(s)) return '0:00';
      s = Math.max(0, Math.floor(s));
      const m = Math.floor(s/60), ss = (s%60).toString().padStart(2, '0');
      return `${m}:${ss}`;
    };

    function saveState(){
      // Persist URL-based entries so they survive refresh.
      const urls = playlist.filter(x => x.kind === 'url').map(({id,title,artist,src})=>({id,title,artist,src}));
      localStorage.setItem('amp_urls', JSON.stringify(urls));
    }
    function loadState(){
      try{
        const urls = JSON.parse(localStorage.getItem('amp_urls')||'[]');
        for(const u of urls){
          playlist.push({ ...u, kind:'url', duration:NaN });
          resolveDuration(playlist.length-1);
        }
      }catch{ /* ignore */ }
      render();
    }

    // --- Rendering ---
    function render(){
      els.playlist.innerHTML = '';
      const term = els.search.value.toLowerCase();
      const frag = document.createDocumentFragment();
      playlist.forEach((t, i) => {
        if (term && !(t.title.toLowerCase().includes(term) || (t.artist||'').toLowerCase().includes(term))) return;
        const node = renderTrack(t, i);
        frag.appendChild(node);
      });
      els.playlist.appendChild(frag);
      els.emptyState.hidden = playlist.length > 0;
      els.drop.hidden = playlist.length > 0;
    }

    function renderTrack(t, index){
      const tpl = document.getElementById('trackTemplate');
      const node = tpl.content.firstElementChild.cloneNode(true);
      node.dataset.id = t.id;
      node.querySelector('.title').textContent = t.title || 'Untitled';
      node.querySelector('.meta').textContent = `${t.artist||'Unknown'} · ${isFinite(t.duration)?fmt(t.duration):'--:--'}`;

      // Actions
      node.querySelector('.playOne').addEventListener('click', () => playIndex(index));
      node.querySelector('.removeOne').addEventListener('click', () => removeIndex(index));
      node.querySelector('.replaceOne').addEventListener('click', async () => {
        const file = await pickOneFile();
        if (file) replaceIndexWithFile(index, file);
      });

      // Drag handle reorder
      node.addEventListener('dragstart', (e)=>{
        node.classList.add('dragging');
        e.dataTransfer.setData('text/plain', String(index));
        e.dataTransfer.effectAllowed = 'move';
      });
      node.addEventListener('dragend', ()=> node.classList.remove('dragging'));
      node.addEventListener('dragover', (e)=>{
        e.preventDefault();
        e.dataTransfer.dropEffect = 'move';
      });
      node.addEventListener('drop', (e)=>{
        e.preventDefault();
        const from = Number(e.dataTransfer.getData('text/plain'));
        const to = index;
        if (!Number.isInteger(from)) return;
        moveItem(from, to);
      });

      return node;
    }

    function moveItem(from, to){
      if (from === to) return;
      const item = playlist.splice(from,1)[0];
      playlist.splice(to,0,item);
      if (current === from) current = to; else if (from < current && to >= current) current--; else if (from > current && to <= current) current++;
      render();
    }

    // --- Playlist operations ---
    function addFiles(files){
      [...files].forEach(file => addFile(file));
      render();
    }
    function addFile(file){
      const url = URL.createObjectURL(file);
      const title = file.name.replace(/\.[^.]+$/, '');
      const t = { id: uid(), title, artist: 'Local File', src: url, kind:'blob', duration:NaN, _file:file };
      playlist.push(t);
      resolveDuration(playlist.length-1);
    }

    function addUrl(){
      const src = prompt('Enter direct audio URL (mp3, m4a, ogg, wav, etc.):');
      if (!src) return;
      const title = prompt('Title (optional):') || new URL(src, location.href).pathname.split('/').pop();
      const artist = prompt('Artist (optional):') || 'URL';
      playlist.push({ id: uid(), title, artist, src, kind:'url', duration:NaN });
      saveState();
      resolveDuration(playlist.length-1);
      render();
    }

    function removeIndex(index){
      const t = playlist[index];
      if (!t) return;
      if (!confirm(`Remove "${t.title}" from playlist?`)) return;
      if (t.kind==='blob') try{ URL.revokeObjectURL(t.src); }catch{}
      playlist.splice(index,1);
      if (index === current){
        stop();
        if (playlist.length) {
          current = Math.min(index, playlist.length-1);
          playIndex(current);
        }
      } else if (index < current){
        current--;
      }
      saveState();
      render();
    }

    async function replaceIndexWithFile(index, file){
      const t = playlist[index];
      if (!t) return;
      if (t.kind==='blob') try{ URL.revokeObjectURL(t.src);}catch{}
      const url = URL.createObjectURL(file);
      t.src = url; t.kind='blob'; t._file=file; t.title=file.name.replace(/\.[^.]+$/, ''); t.artist='Local File'; t.duration=NaN;
      resolveDuration(index);
      if (index === current){ audio.src = t.src; audio.play(); }
      render();
    }

    async function pickOneFile(){
      return new Promise((resolve)=>{
        els.hiddenFile.onchange = () => {
          const f = els.hiddenFile.files[0];
          els.hiddenFile.value = '';
          resolve(f||null);
        };
        els.hiddenFile.click();
      });
    }

    function resolveDuration(index){
      const t = playlist[index];
      if (!t) return;
      const probe = new Audio();
      probe.src = t.src;
      probe.preload = 'metadata';
      probe.addEventListener('loadedmetadata', ()=>{
        t.duration = probe.duration;
        render();
      });
      // Avoid playing sound
      probe.muted = true;
    }

    function clearAll(){
      if (!playlist.length) return;
      if (!confirm('Clear entire playlist?')) return;
      for(const t of playlist){ if (t.kind==='blob') try{ URL.revokeObjectURL(t.src);}catch{} }
      playlist = [];
      current = -1; queue = [];
      saveState();
      stop();
      render();
    }

    // --- Playback ---
    function updateNow(){
      const t = playlist[current];
      els.nowTitle.textContent = t ? t.title : 'No song';
      els.nowArtist.textContent = t ? (t.artist||'') : '—';
      els.duration.textContent = t && isFinite(t.duration) ? fmt(t.duration) : '0:00';
    }

    function playIndex(index){
      if (!playlist[index]) return;
      current = index;
      audio.src = playlist[index].src;
      audio.playbackRate = parseFloat(els.speed.value);
      audio.volume = parseFloat(els.volume.value);
      audio.play();
      if (isShuffle) rebuildQueue();
      setBadge('PLAYING');
      setPlayIcon(true);
      updateNow();
      highlightActive();
    }

    function stop(){
      audio.pause();
      audio.currentTime = 0;
      setBadge('STOPPED');
      setPlayIcon(false);
      updateProgress(0, audio.duration||0);
      highlightActive();
    }

    function togglePlay(){
      if (!playlist.length){ alert('Add some songs first!'); return; }
      if (current === -1) return playIndex(0);
      if (audio.paused){ audio.play(); setBadge('PLAYING'); setPlayIcon(true);} else { audio.pause(); setBadge('PAUSED'); setPlayIcon(false);}  
    }

    function next(){
      if (!playlist.length) return;
      if (repeatMode === 'one') return playIndex(current);
      let nextIndex = -1;
      if (isShuffle){
        if (!queue.length) rebuildQueue();
        nextIndex = queue.shift();
      } else {
        nextIndex = current + 1;
        if (nextIndex >= playlist.length){ nextIndex = (repeatMode==='all') ? 0 : -1; }
      }
      if (nextIndex === -1){ stop(); return; }
      playIndex(nextIndex);
    }

    function prev(){
      if (!playlist.length) return;
      if (audio.currentTime > 3) { audio.currentTime = 0; return; }
      const idx = current - 1 < 0 ? (repeatMode==='all' ? playlist.length-1 : -1) : current - 1;
      if (idx === -1){ stop(); return; }
      playIndex(idx);
    }

    function rebuildQueue(){
      const indices = playlist.map((_,i)=>i).filter(i=>i!==current);
      for(let i=indices.length-1;i>0;i--){ const j=Math.floor(Math.random()*(i+1)); [indices[i],indices[j]]=[indices[j],indices[i]]; }
      queue = indices;
    }

    function setBadge(state){ els.badge.textContent = state; }
    function setPlayIcon(isPlaying){
      els.playIcon.innerHTML = isPlaying
        ? '<path d="M6 5h5v14H6zM13 5h5v14h-5z" fill="currentColor"/>'
        : '<path d="M8 5v14l11-7-11-7z" fill="currentColor"/>';
    }
    function highlightActive(){
      [...els.playlist.querySelectorAll('.track')].forEach((n, i)=>{
        if (i===current) n.style.outline='2px solid var(--accent)'; else n.style.outline='none';
      });
    }

    function updateProgress(cur, dur){
      const pct = dur ? (cur/dur)*100 : 0;
      els.progressFill.style.width = pct+'%';
      els.progressThumb.style.left = pct+'%';
      els.currentTime.textContent = fmt(cur);
      els.duration.textContent = isFinite(dur) ? fmt(dur) : '0:00';
      els.progressBar.setAttribute('aria-valuenow', String(Math.round(pct)));
    }

    // --- Events ---
    els.fileInput.addEventListener('change', (e)=>{ addFiles(e.target.files); e.target.value=''; });
    els.addUrlBtn.addEventListener('click', addUrl);
    els.clearBtn.addEventListener('click', clearAll);

    els.playBtn.addEventListener('click', togglePlay);
    els.prevBtn.addEventListener('click', prev);
    els.nextBtn.addEventListener('click', next);

    els.shuffleBtn.addEventListener('click', ()=>{ isShuffle=!isShuffle; els.shuffleBtn.classList.toggle('active', isShuffle); if (isShuffle) rebuildQueue(); });
    els.repeatBtn.addEventListener('click', ()=>{
      repeatMode = repeatMode==='off' ? 'one' : repeatMode==='one' ? 'all' : 'off';
      els.repeatBtn.classList.add('active');
      els.repeatBtn.title = `Repeat (${repeatMode.toUpperCase()})`;
      setTimeout(()=>els.repeatBtn.classList.remove('active'), 150);
    });

    els.speed.addEventListener('input', ()=>{ audio.playbackRate = parseFloat(els.speed.value); });
    els.volume.addEventListener('input', ()=>{ audio.volume = parseFloat(els.volume.value); });

    // Seeking
    let seeking = false;
    const seekAt = (clientX)=>{
      const rect = els.progressBar.getBoundingClientRect();
      const pct = Math.min(1, Math.max(0, (clientX - rect.left)/rect.width));
      if (isFinite(audio.duration)) audio.currentTime = pct * audio.duration;
    };
    ['mousedown','touchstart'].forEach(ev=>{
      els.progressBar.addEventListener(ev, (e)=>{ seeking=true; seekAt(e.touches?e.touches[0].clientX:e.clientX); });
    });
    ['mousemove','touchmove'].forEach(ev=>{
      document.addEventListener(ev, (e)=>{ if(!seeking) return; seekAt(e.touches?e.touches[0].clientX:e.clientX); });
    });
    ['mouseup','touchend','touchcancel','mouseleave'].forEach(ev=>{
      document.addEventListener(ev, ()=> seeking=false);
    });

    audio.addEventListener('timeupdate', ()=> updateProgress(audio.currentTime, audio.duration));
    audio.addEventListener('loadedmetadata', ()=> updateNow());
    audio.addEventListener('ended', next);
    audio.addEventListener('play', ()=>{ setPlayIcon(true); setBadge('PLAYING'); });
    audio.addEventListener('pause', ()=>{ setPlayIcon(false); setBadge('PAUSED'); });

    // Keyboard shortcuts
    document.addEventListener('keydown', (e)=>{
      if (['INPUT','TEXTAREA'].includes(document.activeElement.tagName)) return;
      const jump = parseFloat(els.jump.value)||5;
      if (e.code==='Space'){ e.preventDefault(); togglePlay(); }
      else if (e.key==='ArrowRight'){ audio.currentTime = Math.min((audio.currentTime||0) + jump, audio.duration||audio.currentTime); }
      else if (e.key==='ArrowLeft'){ audio.currentTime = Math.max((audio.currentTime||0) - jump, 0); }
      else if (e.key==='ArrowUp'){ audio.volume = Math.min(1, audio.volume + 0.05); els.volume.value = audio.volume; }
      else if (e.key==='ArrowDown'){ audio.volume = Math.max(0, audio.volume - 0.05); els.volume.value = audio.volume; }
      else if (e.key.toLowerCase()==='n'){ next(); }
      else if (e.key.toLowerCase()==='p'){ prev(); }
      else if (e.key.toLowerCase()==='s'){ isShuffle=!isShuffle; els.shuffleBtn.classList.toggle('active', isShuffle); if (isShuffle) rebuildQueue(); }
      else if (e.key.toLowerCase()==='r'){ repeatBtn.click(); }
    });

    // Drag & drop add
    ['dragenter','dragover'].forEach(ev=>{
      document.addEventListener(ev, (e)=>{ e.preventDefault(); els.drop.hidden=false; els.drop.style.borderColor='var(--accent)'; });
    });
    ;['dragleave','drop'].forEach(ev=>{
      document.addEventListener(ev, (e)=>{ e.preventDefault(); els.drop.style.borderColor='#2a3144'; if (ev==='drop') els.drop.hidden = playlist.length>0; });
    });
    document.addEventListener('drop', (e)=>{
      const files = e.dataTransfer?.files; if (files?.length) addFiles(files);
    });

    // Init
    loadState();
    updateNow();
  </script>
</body>
</html>
