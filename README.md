<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="theme-color" content="#16161a">
<title>HTML.run</title>
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&family=Syne:wght@700;800&display=swap" rel="stylesheet">
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg: #0d0d0f;
    --surface: #16161a;
    --border: #2a2a32;
    --accent: #ff5c35;
    --accent2: #ffcc00;
    --text: #e8e6e3;
    --muted: #6b6b7a;
    --code-bg: #111114;
  }

  html, body {
    height: 100%;
    overflow: hidden;
    background: var(--bg);
    color: var(--text);
    font-family: 'JetBrains Mono', monospace;
  }

  .app {
    display: flex;
    flex-direction: column;
    height: 100vh;
  }

  /* HEADER */
  header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 12px 16px;
    background: var(--surface);
    border-bottom: 1px solid var(--border);
    flex-shrink: 0;
  }

  .logo {
    font-family: 'Syne', sans-serif;
    font-weight: 800;
    font-size: 1rem;
    color: var(--text);
  }
  .logo span { color: var(--accent); }

  .header-actions {
    display: flex;
    gap: 8px;
    align-items: center;
  }

  .btn-clear {
    background: transparent;
    color: var(--muted);
    border: 1px solid var(--border);
    padding: 9px 16px;
    border-radius: 8px;
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.78rem;
    cursor: pointer;
    transition: color 0.2s, border-color 0.2s;
  }
  .btn-clear:hover { color: var(--text); border-color: var(--muted); }

  .btn-run {
    background: var(--accent);
    color: #fff;
    border: none;
    padding: 9px 22px;
    border-radius: 8px;
    font-family: 'Syne', sans-serif;
    font-weight: 700;
    font-size: 0.9rem;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 8px;
    transition: background 0.15s, transform 0.1s;
  }
  .btn-run:hover { background: #ff3a14; }
  .btn-run:active { transform: scale(0.95); }

  .play-icon {
    width: 0; height: 0;
    border-top: 5px solid transparent;
    border-bottom: 5px solid transparent;
    border-left: 9px solid white;
  }

  /* TABS */
  .tabs {
    display: flex;
    background: var(--surface);
    border-bottom: 1px solid var(--border);
    flex-shrink: 0;
  }

  .tab {
    padding: 10px 20px;
    font-size: 0.72rem;
    font-weight: 700;
    letter-spacing: 1.5px;
    text-transform: uppercase;
    color: var(--muted);
    cursor: pointer;
    border-bottom: 2px solid transparent;
    transition: color 0.2s, border-color 0.2s;
    user-select: none;
  }

  .tab.active {
    color: var(--accent);
    border-bottom-color: var(--accent);
  }

  /* PANELS */
  .panels {
    flex: 1;
    position: relative;
    overflow: hidden;
  }

  .panel {
    position: absolute;
    inset: 0;
    display: none;
  }

  .panel.active { display: flex; flex-direction: column; }

  /* EDITOR */
  textarea {
    flex: 1;
    background: var(--code-bg);
    color: #a8e6cf;
    border: none;
    outline: none;
    resize: none;
    padding: 18px;
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.83rem;
    line-height: 1.75;
    width: 100%;
    caret-color: var(--accent);
    tab-size: 2;
  }
  textarea::placeholder { color: var(--muted); opacity: 1; }

  /* PREVIEW */
  iframe {
    flex: 1;
    border: none;
    width: 100%;
    height: 100%;
    background: #fff;
  }

  .preview-empty {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 14px;
    color: var(--muted);
    font-size: 0.8rem;
    text-align: center;
    padding: 40px;
  }

  .preview-empty .icon { font-size: 3rem; opacity: 0.2; }
  .preview-empty strong { color: var(--accent); }

  /* STATUS */
  .statusbar {
    flex-shrink: 0;
    background: var(--accent);
    color: rgba(255,255,255,0.85);
    font-size: 0.62rem;
    font-weight: 700;
    letter-spacing: 1px;
    text-transform: uppercase;
    padding: 4px 16px;
    display: flex;
    justify-content: space-between;
  }

  /* TOAST */
  .toast {
    position: fixed;
    bottom: 28px;
    left: 50%;
    transform: translateX(-50%) translateY(80px);
    background: var(--accent2);
    color: #000;
    font-family: 'Syne', sans-serif;
    font-weight: 700;
    font-size: 0.82rem;
    padding: 10px 22px;
    border-radius: 30px;
    transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
    pointer-events: none;
    z-index: 999;
  }
  .toast.show { transform: translateX(-50%) translateY(0); }
</style>
</head>
<body>
<div class="app">

  <header>
    <div class="logo">HTML<span>.</span>run</div>
    <div class="header-actions">
      <button class="btn-clear" onclick="clearCode()">Limpiar</button>
      <button class="btn-run" onclick="runCode()">
        <span class="play-icon"></span> Ejecutar
      </button>
    </div>
  </header>

  <div class="tabs">
    <div class="tab active" id="tab-code" onclick="showTab('code')">Código</div>
    <div class="tab" id="tab-preview" onclick="showTab('preview')">Vista previa</div>
  </div>

  <div class="panels">

    <div class="panel active" id="panel-code">
      <textarea id="editor" placeholder="<!-- Pega tu HTML aquí y pulsa Ejecutar -->" spellcheck="false"></textarea>
    </div>

    <div class="panel" id="panel-preview">
      <div class="preview-empty" id="emptyMsg">
        <div class="icon">▶</div>
        <p>Pulsa <strong>Ejecutar</strong> para ver el resultado aquí.</p>
      </div>
      <iframe id="frame" style="display:none;"></iframe>
    </div>

  </div>

  <div class="statusbar">
    <span>HTML.run</span>
    <span id="statusText">listo</span>
  </div>

</div>

<div class="toast" id="toast"></div>

<script>
  function showTab(name) {
    document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
    document.querySelectorAll('.panel').forEach(p => p.classList.remove('active'));
    document.getElementById('tab-' + name).classList.add('active');
    document.getElementById('panel-' + name).classList.add('active');
  }

  function runCode() {
    const code = document.getElementById('editor').value.trim();
    if (!code) { showToast('Escribe algo primero'); return; }

    const frame = document.getElementById('frame');
    frame.style.display = 'block';
    document.getElementById('emptyMsg').style.display = 'none';

    const doc = frame.contentDocument || frame.contentWindow.document;
    doc.open();
    doc.write(code);
    doc.close();

    showTab('preview');
    document.getElementById('statusText').textContent = '✓ ejecutado · ' + new Date().toLocaleTimeString('es-ES');
    showToast('¡Ejecutado!');
  }

  function clearCode() {
    document.getElementById('editor').value = '';
    document.getElementById('frame').style.display = 'none';
    document.getElementById('emptyMsg').style.display = 'flex';
    document.getElementById('statusText').textContent = 'listo';
    showTab('code');
    showToast('Limpiado');
  }

  function showToast(msg) {
    const t = document.getElementById('toast');
    t.textContent = msg;
    t.classList.add('show');
    setTimeout(() => t.classList.remove('show'), 2000);
  }

  document.getElementById('editor').addEventListener('keydown', e => {
    if ((e.ctrlKey || e.metaKey) && e.key === 'Enter') runCode();
  });
</script>
</body>
</html>