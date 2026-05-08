<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Plan de Lotissement — Visualiseur</title>
  <link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet"/>
  <style>
    :root {
      --bg:       #07111c;
      --surface:  #0d1f30;
      --card:     #112233;
      --border:   #1e3a52;
      --accent:   #00c8e0;
      --accent2:  #06e5a0;
      --warn:     #f59e0b;
      --danger:   #ef4444;
      --text:     #e2edf5;
      --muted:    #7a9ab5;
      --font:     'Syne', sans-serif;
      --mono:     'DM Mono', monospace;
    }
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      background: var(--bg);
      color: var(--text);
      font-family: var(--font);
      min-height: 100vh;
      overflow-x: hidden;
    }

    /* ── HEADER ── */
    header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 18px 36px;
      background: var(--surface);
      border-bottom: 1px solid var(--border);
      position: sticky;
      top: 0;
      z-index: 100;
    }
    .logo {
      display: flex;
      align-items: center;
      gap: 14px;
    }
    .logo-icon {
      width: 40px;
      height: 40px;
      background: linear-gradient(135deg, var(--accent), var(--accent2));
      border-radius: 10px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 20px;
    }
    .logo-title { font-size: 18px; font-weight: 700; letter-spacing: -0.3px; }
    .logo-sub { font-size: 11px; color: var(--muted); font-family: var(--mono); margin-top: 2px; }
    .header-actions { display: flex; gap: 10px; align-items: center; }
    .btn {
      padding: 9px 20px;
      border-radius: 8px;
      border: none;
      cursor: pointer;
      font-family: var(--font);
      font-size: 13px;
      font-weight: 600;
      transition: all .18s;
    }
    .btn-primary { background: var(--accent); color: #000; }
    .btn-primary:hover { background: #00e0fa; transform: translateY(-1px); }
    .btn-ghost { background: transparent; color: var(--muted); border: 1px solid var(--border); }
    .btn-ghost:hover { color: var(--text); border-color: var(--accent); }
    .btn-success { background: var(--accent2); color: #000; }
    .btn-success:hover { background: #00ffb3; transform: translateY(-1px); }
    .btn:disabled { opacity: 0.4; cursor: default; transform: none; }

    /* ── STATS BAR ── */
    .stats-bar {
      display: flex;
      gap: 1px;
      background: var(--border);
      border-bottom: 1px solid var(--border);
    }
    .stat-item {
      flex: 1;
      padding: 14px 24px;
      background: var(--surface);
      display: flex;
      flex-direction: column;
      gap: 3px;
    }
    .stat-value {
      font-size: 24px;
      font-weight: 800;
      color: var(--accent);
      font-family: var(--mono);
      line-height: 1;
    }
    .stat-label { font-size: 11px; color: var(--muted); text-transform: uppercase; letter-spacing: 1px; }

    /* ── LAYOUT ── */
    .layout {
      display: grid;
      grid-template-columns: 1fr 400px;
      height: calc(100vh - 135px);
    }

    /* ── CANVAS AREA ── */
    .canvas-area {
      position: relative;
      background: #060f18;
      overflow: hidden;
    }
    .canvas-toolbar {
      position: absolute;
      top: 16px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 6px;
      z-index: 10;
      background: rgba(7,17,28,.85);
      border: 1px solid var(--border);
      border-radius: 12px;
      padding: 6px;
      backdrop-filter: blur(8px);
    }
    .tool-btn {
      width: 36px;
      height: 36px;
      border-radius: 8px;
      border: 1px solid transparent;
      background: transparent;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 16px;
      color: var(--muted);
      transition: all .15s;
    }
    .tool-btn:hover, .tool-btn.active { background: var(--border); color: var(--accent); border-color: var(--accent); }
    #planCanvas {
      width: 100%;
      height: 100%;
      cursor: grab;
      display: block;
    }
    #planCanvas:active { cursor: grabbing; }

    /* ── DROP ZONE (shown when no file) ── */
    .drop-zone {
      position: absolute;
      inset: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 20px;
      border: 2px dashed var(--border);
      margin: 32px;
      border-radius: 20px;
      cursor: pointer;
      transition: all .2s;
    }
    .drop-zone:hover, .drop-zone.drag-over {
      border-color: var(--accent);
      background: rgba(0,200,224,.04);
    }
    .drop-icon {
      font-size: 64px;
      filter: grayscale(.6);
      animation: float 3s ease-in-out infinite;
    }
    @keyframes float { 0%,100%{transform:translateY(0)} 50%{transform:translateY(-10px)} }
    .drop-title { font-size: 22px; font-weight: 700; color: var(--text); }
    .drop-sub { font-size: 13px; color: var(--muted); text-align: center; max-width: 340px; line-height: 1.6; }
    .drop-formats {
      display: flex;
      gap: 8px;
    }
    .format-badge {
      padding: 4px 12px;
      border-radius: 20px;
      font-size: 12px;
      font-family: var(--mono);
      font-weight: 500;
    }
    .format-dxf { background: rgba(0,200,224,.15); color: var(--accent); border: 1px solid rgba(0,200,224,.3); }
    .format-dwg { background: rgba(245,158,11,.12); color: var(--warn); border: 1px solid rgba(245,158,11,.25); }

    /* ── LOADING OVERLAY ── */
    .loading-overlay {
      position: absolute;
      inset: 0;
      background: rgba(7,17,28,.85);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 16px;
      z-index: 50;
    }
    .spinner {
      width: 48px;
      height: 48px;
      border: 3px solid var(--border);
      border-top-color: var(--accent);
      border-radius: 50%;
      animation: spin .8s linear infinite;
    }
    @keyframes spin { to { transform: rotate(360deg); } }
    .loading-text { font-size: 14px; color: var(--muted); font-family: var(--mono); }

    /* ── ERROR BANNER ── */
    .error-banner {
      position: absolute;
      bottom: 16px;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(239,68,68,.12);
      border: 1px solid rgba(239,68,68,.4);
      color: #fca5a5;
      border-radius: 10px;
      padding: 12px 20px;
      font-size: 13px;
      max-width: 560px;
      text-align: center;
      line-height: 1.5;
      z-index: 20;
    }

    /* ── RIGHT PANEL ── */
    .panel {
      background: var(--surface);
      border-left: 1px solid var(--border);
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }
    .panel-tabs {
      display: flex;
      border-bottom: 1px solid var(--border);
    }
    .tab {
      flex: 1;
      padding: 14px;
      text-align: center;
      font-size: 12px;
      font-weight: 700;
      text-transform: uppercase;
      letter-spacing: 1px;
      cursor: pointer;
      color: var(--muted);
      border-bottom: 2px solid transparent;
      transition: all .15s;
    }
    .tab.active { color: var(--accent); border-bottom-color: var(--accent); }
    .panel-body {
      flex: 1;
      overflow-y: auto;
      padding: 0;
      scrollbar-width: thin;
      scrollbar-color: var(--border) transparent;
    }

    /* ── LOT CARDS ── */
    .lot-card {
      padding: 14px 18px;
      border-bottom: 1px solid var(--border);
      cursor: pointer;
      transition: background .15s;
      position: relative;
    }
    .lot-card:hover { background: rgba(255,255,255,.02); }
    .lot-card.selected { background: rgba(0,200,224,.07); border-left: 3px solid var(--accent); padding-left: 15px; }
    .lot-card-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 8px;
    }
    .lot-id {
      font-family: var(--mono);
      font-size: 11px;
      font-weight: 500;
      color: var(--muted);
      background: var(--card);
      padding: 2px 8px;
      border-radius: 4px;
    }
    .lot-num {
      font-size: 16px;
      font-weight: 800;
      color: var(--text);
    }
    .lot-status {
      font-size: 10px;
      font-weight: 600;
      padding: 3px 9px;
      border-radius: 20px;
    }
    .status-disponible { background: rgba(6,229,160,.15); color: var(--accent2); }
    .status-vendu { background: rgba(239,68,68,.15); color: #f87171; }
    .status-reserve { background: rgba(245,158,11,.15); color: var(--warn); }
    .lot-meta {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 6px;
    }
    .meta-item { display: flex; flex-direction: column; gap: 2px; }
    .meta-label { font-size: 10px; color: var(--muted); text-transform: uppercase; letter-spacing: .5px; }
    .meta-value { font-size: 13px; font-weight: 600; font-family: var(--mono); }

    /* ── LAYERS PANEL ── */
    .layer-list { padding: 12px; }
    .layer-item {
      display: flex;
      align-items: center;
      gap: 10px;
      padding: 8px 10px;
      border-radius: 8px;
      font-size: 13px;
      font-family: var(--mono);
      cursor: pointer;
      transition: background .15s;
    }
    .layer-item:hover { background: var(--card); }
    .layer-dot {
      width: 10px;
      height: 10px;
      border-radius: 50%;
      flex-shrink: 0;
    }
    .layer-name { flex: 1; color: var(--text); }
    .layer-count { font-size: 11px; color: var(--muted); }

    /* ── IMPORT NOTICE ── */
    .import-notice {
      padding: 14px 18px;
      background: rgba(0,200,224,.06);
      border-bottom: 1px solid rgba(0,200,224,.15);
      font-size: 12px;
      color: var(--muted);
      display: flex;
      align-items: center;
      gap: 8px;
    }
    .import-notice strong { color: var(--accent); }

    /* ── EMPTY STATE ── */
    .empty-state {
      padding: 40px 20px;
      text-align: center;
      color: var(--muted);
    }
    .empty-state-icon { font-size: 40px; margin-bottom: 12px; opacity: .5; }
    .empty-state-text { font-size: 13px; line-height: 1.6; }

    /* ── DWG WARNING ── */
    .dwg-warning {
      margin: 20px;
      padding: 18px;
      border-radius: 12px;
      background: rgba(245,158,11,.08);
      border: 1px solid rgba(245,158,11,.3);
      color: #fde68a;
      font-size: 13px;
      line-height: 1.7;
    }
    .dwg-warning strong { color: var(--warn); display: block; margin-bottom: 6px; font-size: 14px; }
    .dwg-steps { margin-top: 10px; padding-left: 16px; color: var(--muted); }
    .dwg-steps li { margin-bottom: 4px; }

    /* ── ZOOM INDICATOR ── */
    .zoom-indicator {
      position: absolute;
      bottom: 12px;
      right: 12px;
      font-family: var(--mono);
      font-size: 11px;
      color: var(--muted);
      background: rgba(7,17,28,.7);
      padding: 4px 10px;
      border-radius: 6px;
      border: 1px solid var(--border);
    }

    /* ── SCROLLBAR ── */
    ::-webkit-scrollbar { width: 4px; }
    ::-webkit-scrollbar-track { background: transparent; }
    ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 4px; }

    /* ── RESPONSIVE ── */
    @media (max-width: 900px) {
      .layout { grid-template-columns: 1fr; grid-template-rows: 60vh 1fr; }
      .panel { border-left: none; border-top: 1px solid var(--border); }
    }

    /* hidden file input */
    #fileInput { display: none; }
  </style>
</head>
<body>

<!-- ── HEADER ── -->
<header>
  <div class="logo">
    <div class="logo-icon">🗺️</div>
    <div>
      <div class="logo-title">Plan de Lotissement</div>
      <div class="logo-sub">Visualiseur DXF · Extraction automatique</div>
    </div>
  </div>
  <div class="header-actions">
    <button class="btn btn-ghost" onclick="document.getElementById('fileInput').click()">📂 Ouvrir fichier</button>
    <button class="btn btn-success" id="btnImport" disabled onclick="importToRegistry()">⬆️ Importer dans le registre</button>
    <a href="PROJET_1.html" class="btn btn-primary">📋 Registre foncier →</a>
  </div>
</header>
<input type="file" id="fileInput" accept=".dxf,.dwg" onchange="handleFileInput(event)"/>

<!-- ── STATS BAR ── -->
<div class="stats-bar">
  <div class="stat-item">
    <div class="stat-value" id="statLots">0</div>
    <div class="stat-label">Lots extraits</div>
  </div>
  <div class="stat-item">
    <div class="stat-value" id="statSurface">0</div>
    <div class="stat-label">Surface totale (m²)</div>
  </div>
  <div class="stat-item">
    <div class="stat-value" id="statLayers">0</div>
    <div class="stat-label">Calques détectés</div>
  </div>
  <div class="stat-item">
    <div class="stat-value" id="statEntities">0</div>
    <div class="stat-label">Entités lues</div>
  </div>
</div>

<!-- ── MAIN LAYOUT ── -->
<div class="layout">

  <!-- ── CANVAS AREA ── -->
  <div class="canvas-area" id="canvasArea"
       ondragover="event.preventDefault();document.getElementById('dropZone').classList.add('drag-over')"
       ondragleave="document.getElementById('dropZone').classList.remove('drag-over')"
       ondrop="handleDrop(event)">

    <!-- Drop zone overlay (shown before file load) -->
    <div class="drop-zone" id="dropZone" onclick="document.getElementById('fileInput').click()">
      <div class="drop-icon">📐</div>
      <div class="drop-title">Glissez votre plan ici</div>
      <div class="drop-sub">Déposez votre fichier plan de lotissement pour le visualiser directement et extraire les lots automatiquement.</div>
      <div class="drop-formats">
        <span class="format-badge format-dxf">✓ DXF — Recommandé</span>
        <span class="format-badge format-dwg">⚠ DWG → à convertir</span>
      </div>
      <button class="btn btn-primary" style="margin-top:8px">Parcourir les fichiers</button>
    </div>

    <!-- Loading overlay -->
    <div class="loading-overlay" id="loadingOverlay" style="display:none">
      <div class="spinner"></div>
      <div class="loading-text" id="loadingText">Lecture du fichier…</div>
    </div>

    <!-- Toolbar (shown after load) -->
    <div class="canvas-toolbar" id="canvasToolbar" style="display:none">
      <button class="tool-btn" title="Zoom +" onclick="zoom(1.2)">🔍</button>
      <button class="tool-btn" title="Zoom –" onclick="zoom(0.83)">🔎</button>
      <button class="tool-btn" title="Ajuster" onclick="fitView()">⛶</button>
      <div style="width:1px;background:var(--border);margin:4px 2px"></div>
      <button class="tool-btn" title="Réinitialiser" onclick="resetFile()">🗑️</button>
    </div>

    <!-- Canvas -->
    <canvas id="planCanvas" style="display:none"></canvas>

    <!-- Zoom indicator -->
    <div class="zoom-indicator" id="zoomIndicator" style="display:none">100%</div>

    <!-- Error banner -->
    <div class="error-banner" id="errorBanner" style="display:none"></div>
  </div>

  <!-- ── RIGHT PANEL ── -->
  <div class="panel">
    <div class="panel-tabs">
      <div class="tab active" id="tabLots" onclick="switchTab('lots')">🏗 Lots</div>
      <div class="tab" id="tabLayers" onclick="switchTab('layers')">🎨 Calques</div>
    </div>

    <div class="panel-body" id="panelLots">
      <div class="empty-state" id="emptyLots">
        <div class="empty-state-icon">📐</div>
        <div class="empty-state-text">Importez un fichier DXF pour extraire<br/>automatiquement les lots et terrains.</div>
      </div>
    </div>

    <div class="panel-body" id="panelLayers" style="display:none">
      <div class="empty-state" id="emptyLayers">
        <div class="empty-state-icon">🎨</div>
        <div class="empty-state-text">Les calques apparaîtront ici après<br/>l'importation du plan.</div>
      </div>
    </div>
  </div>
</div>

<script>
// ═══════════════════════════════════════════════════════════════════
// ── ÉTAT GLOBAL ──────────────────────────────────────────────────
// ═══════════════════════════════════════════════════════════════════
let gEntities = [];
let gLots = [];
let gLayers = {};
let gSelectedLot = null;
let gTransform = { x: 0, y: 0, scale: 1 };
let gBounds = null;
let gPan = null;

const LAYER_PALETTE = [
  '#00c8e0','#06e5a0','#f59e0b','#ef4444','#a78bfa',
  '#34d399','#f472b6','#60a5fa','#fb923c','#a3e635'
];
let layerColorMap = {};
let colorIdx = 0;

function getLayerColor(layer) {
  if (!layerColorMap[layer]) {
    layerColorMap[layer] = LAYER_PALETTE[colorIdx++ % LAYER_PALETTE.length];
  }
  return layerColorMap[layer];
}

// ═══════════════════════════════════════════════════════════════════
// ── DXF PARSER ───────────────────────────────────────────────────
// ═══════════════════════════════════════════════════════════════════
function parseDxf(text) {
  const lines = text.split(/\r?\n/);
  const pairs = [];
  for (let i = 0; i + 1 < lines.length; i += 2) {
    const code = parseInt(lines[i].trim(), 10);
    const value = lines[i + 1];
    if (!isNaN(code)) pairs.push([code, value.trim()]);
  }

  const entities = [];
  let inEntities = false;
  let i = 0;

  while (i < pairs.length) {
    const [code, value] = pairs[i];

    if (code === 0 && value === 'SECTION') {
      i++;
      if (i < pairs.length && pairs[i][0] === 2) {
        inEntities = (pairs[i][1] === 'ENTITIES');
      }
      i++;
      continue;
    }

    if (code === 0 && value === 'ENDSEC') {
      inEntities = false;
      i++;
      continue;
    }

    if (inEntities && code === 0) {
      const SUPPORTED = ['LWPOLYLINE','POLYLINE','LINE','TEXT','MTEXT','INSERT','CIRCLE','ARC','HATCH'];
      if (SUPPORTED.includes(value)) {
        const result = parseEntity(pairs, i);
        if (result.entity.vertices.length > 0 || result.entity.type === 'TEXT' || result.entity.type === 'MTEXT' || result.entity.type === 'LINE' || result.entity.type === 'CIRCLE') {
          entities.push(result.entity);
        }
        i = result.nextIdx;
        continue;
      }
    }
    i++;
  }

  return entities;
}

function parseEntity(pairs, startIdx) {
  const type = pairs[startIdx][1];
  const entity = {
    type, layer: '0', vertices: [],
    text: '', x: 0, y: 0, x2: 0, y2: 0,
    closed: false, color: null, radius: 0, height: 2.5
  };
  let pendingX = null;
  let i = startIdx + 1;

  while (i < pairs.length) {
    const [code, raw] = pairs[i];
    if (code === 0) break;

    switch (code) {
      case 8:  entity.layer = raw; break;
      case 62: entity.color = parseInt(raw); break;
      case 1:  entity.text = raw; break;
      case 3:  entity.text += raw; break;
      case 70:
        if (type === 'LWPOLYLINE') entity.closed = !!(parseInt(raw) & 1);
        break;
      case 10:
        pendingX = parseFloat(raw);
        if (type === 'LWPOLYLINE') {
          entity.vertices.push({ x: pendingX, y: 0 });
        } else {
          entity.x = pendingX;
        }
        break;
      case 20: {
        const y = parseFloat(raw);
        if (type === 'LWPOLYLINE' && entity.vertices.length > 0) {
          entity.vertices[entity.vertices.length - 1].y = y;
        } else {
          entity.y = y;
        }
        break;
      }
      case 11: entity.x2 = parseFloat(raw); break;
      case 21: entity.y2 = parseFloat(raw); break;
      case 40:
        entity.radius = parseFloat(raw);
        entity.height  = parseFloat(raw);
        break;
    }
    i++;
  }

  // For LINE, add as 2-vertex polyline internally
  if (type === 'LINE') {
    entity.vertices = [{ x: entity.x, y: entity.y }, { x: entity.x2, y: entity.y2 }];
  }

  return { entity, nextIdx: i };
}

// ═══════════════════════════════════════════════════════════════════
// ── GEOMETRY ─────────────────────────────────────────────────────
// ═══════════════════════════════════════════════════════════════════
function shoelaceArea(verts) {
  let a = 0;
  const n = verts.length;
  for (let i = 0; i < n; i++) {
    const j = (i + 1) % n;
    a += verts[i].x * verts[j].y - verts[j].x * verts[i].y;
  }
  return Math.abs(a / 2);
}

function centroid(verts) {
  let cx = 0, cy = 0;
  for (const v of verts) { cx += v.x; cy += v.y; }
  return { x: cx / verts.length, y: cy / verts.length };
}

function pointInPoly(pt, poly) {
  let inside = false;
  for (let i = 0, j = poly.length - 1; i < poly.length; j = i++) {
    const xi = poly[i].x, yi = poly[i].y;
    const xj = poly[j].x, yj = poly[j].y;
    if (((yi > pt.y) !== (yj > pt.y)) && pt.x < ((xj - xi) * (pt.y - yi) / (yj - yi) + xi)) {
      inside = !inside;
    }
  }
  return inside;
}

function getBounds(entities) {
  let minX = Infinity, minY = Infinity, maxX = -Infinity, maxY = -Infinity;
  for (const e of entities) {
    for (const v of e.vertices) {
      if (v.x < minX) minX = v.x; if (v.x > maxX) maxX = v.x;
      if (v.y < minY) minY = v.y; if (v.y > maxY) maxY = v.y;
    }
    if (['TEXT','MTEXT'].includes(e.type)) {
      if (e.x < minX) minX = e.x; if (e.x > maxX) maxX = e.x;
      if (e.y < minY) minY = e.y; if (e.y > maxY) maxY = e.y;
    }
  }
  return isFinite(minX) ? { minX, minY, maxX, maxY } : null;
}

// ═══════════════════════════════════════════════════════════════════
// ── LOT EXTRACTOR ────────────────────────────────────────────────
// ═══════════════════════════════════════════════════════════════════
function extractLots(entities) {
  // Closed polylines with ≥3 vertices → potential lots
  const polys = entities.filter(e =>
    e.type === 'LWPOLYLINE' && e.closed && e.vertices.length >= 3
  );

  // All text entities
  const texts = entities.filter(e => ['TEXT','MTEXT'].includes(e.type) && e.text.trim());

  return polys.map((poly, idx) => {
    const area   = shoelaceArea(poly.vertices);
    const center = centroid(poly.vertices);

    // Texts inside this lot boundary
    const inside = texts.filter(t => pointInPoly({ x: t.x, y: t.y }, poly.vertices));

    let lotNum = '', superficie = '', ilot = '', utilisation = '';

    for (const t of inside) {
      const txt = t.text.trim();
      // Lot number: "LOT 12", "N°3", "12", "L12"
      const mLot  = txt.match(/^(?:LOT\s*|N[°oa]\s*)?(\d+)$/i);
      // Area: "150.25" or "150,25" or "150.25 m²"
      const mArea = txt.match(/(\d[\d\s]*[,.]?\d*)\s*m?[²2]?/i);
      // Ilot: "ILOT A", "ÎLOT B2"
      const mIlot = txt.match(/[ÎI]LOT\s+([A-Z0-9]+)/i);

      if (mLot  && !lotNum)    lotNum    = mLot[1];
      if (mArea && !superficie) superficie = mArea[1].replace(',', '.').replace(/\s/g, '');
      if (mIlot && !ilot)      ilot      = mIlot[1];
    }

    if (!lotNum) lotNum = String(idx + 1);

    // Detect usage from layer name or texts
    const layerUpper = poly.layer.toUpperCase();
    const allText    = inside.map(t => t.text.toUpperCase()).join(' ');
    if (layerUpper.includes('COMM') || allText.includes('COMM'))          utilisation = 'Commercial';
    else if (layerUpper.includes('EQUIP') || allText.includes('EQUIP'))   utilisation = 'Équipement';
    else if (allText.match(/ESPACE\s*VERT|JARDIN|PARC/))                  utilisation = 'Espace vert';
    else if (layerUpper.match(/VOIRIE|ROUTE|RUE/))                        utilisation = 'Voirie';
    else                                                                    utilisation = 'Habitation';

    const surfNum = parseFloat(superficie);

    return {
      id:          idx + 1,
      lotNum,
      ilot:        ilot || '',
      zone:        poly.layer,
      superficie:  !isNaN(surfNum) ? surfNum : Math.round(area * 10) / 10,
      utilisation,
      statut:      'Disponible',
      acheteur:    '',
      prix:        '',
      vertices:    poly.vertices,
      center,
      layer:       poly.layer,
      _entity:     poly,
    };
  });
}

// ═══════════════════════════════════════════════════════════════════
// ── RENDERER ─────────────────────────────────────────────────────
// ═══════════════════════════════════════════════════════════════════
function toScreen(wx, wy) {
  const { x, y, scale } = gTransform;
  const canvas = document.getElementById('planCanvas');
  const bounds  = gBounds;
  const W = canvas.width, H = canvas.height;
  const bw = (bounds.maxX - bounds.minX) || 1;
  const bh = (bounds.maxY - bounds.minY) || 1;
  const baseScale = Math.min((W - 80) / bw, (H - 80) / bh);
  const s = baseScale * scale;
  const ox = x + (W - bw * s) / 2 - bounds.minX * s;
  const oy = y + H - (H - bh * s) / 2 + bounds.minY * s;
  return { sx: ox + wx * s, sy: oy - wy * s };
}

function renderAll() {
  const canvas = document.getElementById('planCanvas');
  if (!canvas || !gBounds || gEntities.length === 0) return;
  const ctx = canvas.getContext('2d');
  const W = canvas.width, H = canvas.height;

  ctx.clearRect(0, 0, W, H);

  // Background grid
  ctx.fillStyle = '#060f18';
  ctx.fillRect(0, 0, W, H);
  ctx.strokeStyle = '#0e2030';
  ctx.lineWidth = 1;
  const gs = 40;
  for (let gx = 0; gx < W; gx += gs) { ctx.beginPath(); ctx.moveTo(gx, 0); ctx.lineTo(gx, H); ctx.stroke(); }
  for (let gy = 0; gy < H; gy += gs) { ctx.beginPath(); ctx.moveTo(0, gy); ctx.lineTo(W, gy); ctx.stroke(); }

  // Draw entities
  for (const e of gEntities) {
    const col = getLayerColor(e.layer);
    const pts = e.vertices.map(v => toScreen(v.x, v.y));

    if (e.type === 'LWPOLYLINE' && pts.length >= 2) {
      const isSelected = gSelectedLot && gSelectedLot._entity === e;
      ctx.beginPath();
      ctx.moveTo(pts[0].sx, pts[0].sy);
      for (let k = 1; k < pts.length; k++) ctx.lineTo(pts[k].sx, pts[k].sy);
      if (e.closed) ctx.closePath();

      if (e.closed && pts.length >= 3) {
        ctx.fillStyle = isSelected ? 'rgba(255,200,50,.22)' : col + '18';
        ctx.fill();
      }
      ctx.strokeStyle = isSelected ? '#ffc832' : col;
      ctx.lineWidth   = isSelected ? 2.5 : 1.5;
      ctx.stroke();

      // Lot number label
      if (e.closed && isSelected) {
        const ctr = toScreen(gSelectedLot.center.x, gSelectedLot.center.y);
        ctx.fillStyle = '#ffc832';
        ctx.font = 'bold 14px Syne, sans-serif';
        ctx.textAlign = 'center';
        ctx.fillText(`LOT ${gSelectedLot.lotNum}`, ctr.sx, ctr.sy);
      }
    }

    if (e.type === 'LINE' && pts.length === 2) {
      ctx.beginPath();
      ctx.moveTo(pts[0].sx, pts[0].sy);
      ctx.lineTo(pts[1].sx, pts[1].sy);
      ctx.strokeStyle = col;
      ctx.lineWidth = 1;
      ctx.stroke();
    }

    if (e.type === 'CIRCLE') {
      const { sx, sy } = toScreen(e.x, e.y);
      const bounds2 = gBounds;
      const bw2 = (bounds2.maxX - bounds2.minX) || 1;
      const bh2 = (bounds2.maxY - bounds2.minY) || 1;
      const canvas2 = document.getElementById('planCanvas');
      const baseS = Math.min((canvas2.width - 80) / bw2, (canvas2.height - 80) / bh2);
      const r = e.radius * baseS * gTransform.scale;
      ctx.beginPath();
      ctx.arc(sx, sy, r, 0, Math.PI * 2);
      ctx.strokeStyle = col;
      ctx.lineWidth = 1;
      ctx.stroke();
    }
  }

  // Draw text entities (on top)
  for (const e of gEntities) {
    if (['TEXT','MTEXT'].includes(e.type) && e.text) {
      const { sx, sy } = toScreen(e.x, e.y);
      const W2 = canvas.width, H2 = canvas.height;
      if (sx < -10 || sx > W2 + 10 || sy < -10 || sy > H2 + 10) continue;
      const bw3 = (gBounds.maxX - gBounds.minX) || 1;
      const bh3 = (gBounds.maxY - gBounds.minY) || 1;
      const baseS3 = Math.min((W2 - 80) / bw3, (H2 - 80) / bh3);
      const fs = Math.max(7, Math.min(16, (e.height || 2.5) * baseS3 * gTransform.scale * 0.8));
      ctx.fillStyle = '#ffffff88';
      ctx.font = `${fs}px DM Mono, monospace`;
      ctx.textAlign = 'center';
      ctx.fillText(e.text.substring(0, 30), sx, sy);
    }
  }

  // Update zoom indicator
  document.getElementById('zoomIndicator').textContent = Math.round(gTransform.scale * 100) + '%';
}

// ═══════════════════════════════════════════════════════════════════
// ── FILE HANDLING ────────────────────────────────────────────────
// ═══════════════════════════════════════════════════════════════════
function handleDrop(e) {
  e.preventDefault();
  document.getElementById('dropZone').classList.remove('drag-over');
  const file = e.dataTransfer.files[0];
  if (file) processFile(file);
}

function handleFileInput(e) {
  const file = e.target.files[0];
  if (file) processFile(file);
  e.target.value = '';
}

async function processFile(file) {
  setError('');
  showLoading('Lecture du fichier…');

  try {
    // Check for DWG magic bytes
    const header = await readSlice(file, 6);
    if (header.startsWith('AC')) {
      hideLoading();
      showDwgWarning();
      return;
    }

    setLoading('Analyse DXF…');
    const text = await file.text();

    setLoading('Extraction des entités…');
    await tick();
    const entities = parseDxf(text);

    setLoading('Détection des lots…');
    await tick();
    const lots = extractLots(entities);

    // Build layer map
    const layers = {};
    for (const e of entities) {
      if (!layers[e.layer]) layers[e.layer] = { name: e.layer, count: 0, color: getLayerColor(e.layer) };
      layers[e.layer].count++;
    }

    gEntities = entities;
    gLots     = lots;
    gLayers   = layers;
    gBounds   = getBounds(entities);
    gTransform = { x: 0, y: 0, scale: 1 };

    // Update stats
    document.getElementById('statLots').textContent     = lots.length;
    document.getElementById('statSurface').textContent  = Math.round(lots.reduce((s, l) => s + (l.superficie || 0), 0)).toLocaleString('fr-FR');
    document.getElementById('statLayers').textContent   = Object.keys(layers).length;
    document.getElementById('statEntities').textContent = entities.length;

    // Show canvas
    showCanvas();
    renderLotPanel();
    renderLayerPanel();

    document.getElementById('btnImport').disabled = lots.length === 0;

  } catch (err) {
    hideLoading();
    setError('❌ Erreur lors de la lecture : ' + err.message);
  }
}

function readSlice(file, bytes) {
  return new Promise(resolve => {
    const reader = new FileReader();
    reader.onload = e => resolve(e.target.result || '');
    reader.readAsText(file.slice(0, bytes));
  });
}

function tick() {
  return new Promise(r => setTimeout(r, 10));
}

function showDwgWarning() {
  document.getElementById('dropZone').style.display = 'none';
  document.getElementById('errorBanner').style.display = 'none';
  const area = document.getElementById('canvasArea');
  const existing = document.getElementById('dwgWarn');
  if (!existing) {
    const div = document.createElement('div');
    div.id = 'dwgWarn';
    div.className = 'dwg-warning';
    div.style.cssText = 'position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);width:90%;max-width:500px;z-index:20';
    div.innerHTML = `
      <strong>⚠️ Format DWG détecté — conversion nécessaire</strong>
      Le format DWG est un format binaire propriétaire d'AutoCAD. Pour visualiser votre plan ici, exportez-le en <strong style="color:var(--accent)">DXF</strong> (format texte ouvert).
      <ol class="dwg-steps">
        <li>Ouvrez votre fichier dans AutoCAD / AutoCAD LT</li>
        <li>Menu <em>Fichier → Enregistrer sous…</em></li>
        <li>Choisissez le type : <strong>AutoCAD 2010 DXF (*.dxf)</strong></li>
        <li>Enregistrez et importez le fichier .dxf ici</li>
      </ol>
      <button class="btn btn-ghost" style="margin-top:12px;font-size:12px" onclick="resetFile()">↩ Choisir un autre fichier</button>
    `;
    area.appendChild(div);
  }
}

function showCanvas() {
  hideLoading();
  document.getElementById('dropZone').style.display = 'none';
  const dw = document.getElementById('dwgWarn');
  if (dw) dw.remove();

  const canvas = document.getElementById('planCanvas');
  canvas.style.display = 'block';
  document.getElementById('canvasToolbar').style.display = 'flex';
  document.getElementById('zoomIndicator').style.display = 'block';

  // Resize canvas to fill area
  resizeCanvas();
  renderAll();
  setupInteraction();
}

function resizeCanvas() {
  const canvas = document.getElementById('planCanvas');
  const area   = document.getElementById('canvasArea');
  canvas.width  = area.clientWidth;
  canvas.height = area.clientHeight;
}

function showLoading(msg) {
  document.getElementById('loadingText').textContent = msg;
  document.getElementById('loadingOverlay').style.display = 'flex';
}
function setLoading(msg) { document.getElementById('loadingText').textContent = msg; }
function hideLoading() { document.getElementById('loadingOverlay').style.display = 'none'; }

function setError(msg) {
  const el = document.getElementById('errorBanner');
  if (msg) { el.textContent = msg; el.style.display = 'block'; }
  else       el.style.display = 'none';
}

function resetFile() {
  gEntities = []; gLots = []; gLayers = {}; gBounds = null; gSelectedLot = null;
  document.getElementById('dropZone').style.display = 'flex';
  document.getElementById('planCanvas').style.display = 'none';
  document.getElementById('canvasToolbar').style.display = 'none';
  document.getElementById('zoomIndicator').style.display = 'none';
  document.getElementById('btnImport').disabled = true;
  setError('');
  const dw = document.getElementById('dwgWarn');
  if (dw) dw.remove();
  document.getElementById('statLots').textContent = '0';
  document.getElementById('statSurface').textContent = '0';
  document.getElementById('statLayers').textContent = '0';
  document.getElementById('statEntities').textContent = '0';
  document.getElementById('panelLots').innerHTML = `<div class="empty-state"><div class="empty-state-icon">📐</div><div class="empty-state-text">Importez un fichier DXF pour extraire<br/>automatiquement les lots et terrains.</div></div>`;
  document.getElementById('panelLayers').innerHTML = `<div class="empty-state"><div class="empty-state-icon">🎨</div><div class="empty-state-text">Les calques apparaîtront ici après<br/>l'importation du plan.</div></div>`;
}

// ═══════════════════════════════════════════════════════════════════
// ── PANEL RENDERING ──────────────────────────────────────────────
// ═══════════════════════════════════════════════════════════════════
function renderLotPanel() {
  const panel = document.getElementById('panelLots');
  if (gLots.length === 0) {
    panel.innerHTML = `<div class="empty-state"><div class="empty-state-icon">🔍</div><div class="empty-state-text">Aucun lot fermé détecté.<br/>Vérifiez que vos parcelles sont des polylignes fermées.</div></div>`;
    return;
  }

  let html = `<div class="import-notice">✅ <strong>${gLots.length} lots</strong> extraits automatiquement — prêts à importer</div>`;
  for (const lot of gLots) {
    const statusClass = lot.statut === 'Vendu' ? 'status-vendu' : lot.statut === 'Réservé' ? 'status-reserve' : 'status-disponible';
    html += `
      <div class="lot-card" id="lotcard-${lot.id}" onclick="selectLot(${lot.id})">
        <div class="lot-card-header">
          <span class="lot-id">ID-${String(lot.id).padStart(3,'0')}</span>
          <span class="lot-num">LOT ${lot.lotNum}</span>
          <span class="lot-status ${statusClass}">${lot.statut}</span>
        </div>
        <div class="lot-meta">
          <div class="meta-item">
            <span class="meta-label">Îlot</span>
            <span class="meta-value">${lot.ilot || '—'}</span>
          </div>
          <div class="meta-item">
            <span class="meta-label">Superficie</span>
            <span class="meta-value">${lot.superficie.toLocaleString('fr-FR')} m²</span>
          </div>
          <div class="meta-item">
            <span class="meta-label">Utilisation</span>
            <span class="meta-value">${lot.utilisation}</span>
          </div>
          <div class="meta-item">
            <span class="meta-label">Zone / Calque</span>
            <span class="meta-value" style="color:${getLayerColor(lot.layer)}">${lot.zone}</span>
          </div>
        </div>
      </div>`;
  }
  panel.innerHTML = html;
}

function renderLayerPanel() {
  const panel = document.getElementById('panelLayers');
  const entries = Object.values(gLayers).sort((a, b) => b.count - a.count);
  if (entries.length === 0) {
    panel.innerHTML = `<div class="empty-state"><div class="empty-state-icon">🎨</div><div class="empty-state-text">Aucun calque trouvé.</div></div>`;
    return;
  }
  let html = '<div class="layer-list">';
  for (const l of entries) {
    html += `<div class="layer-item">
      <div class="layer-dot" style="background:${l.color}"></div>
      <span class="layer-name">${l.name}</span>
      <span class="layer-count">${l.count} entité${l.count > 1 ? 's' : ''}</span>
    </div>`;
  }
  html += '</div>';
  panel.innerHTML = html;
}

function selectLot(id) {
  const lot = gLots.find(l => l.id === id);
  if (!lot) return;

  // Deselect previous
  if (gSelectedLot) {
    const prev = document.getElementById(`lotcard-${gSelectedLot.id}`);
    if (prev) prev.classList.remove('selected');
  }

  gSelectedLot = lot;
  const card = document.getElementById(`lotcard-${id}`);
  if (card) {
    card.classList.add('selected');
    card.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
  }

  // Pan canvas to lot center
  panToWorld(lot.center.x, lot.center.y);
  renderAll();
}

function panToWorld(wx, wy) {
  if (!gBounds) return;
  const canvas = document.getElementById('planCanvas');
  const W = canvas.width, H = canvas.height;
  const bw = (gBounds.maxX - gBounds.minX) || 1;
  const bh = (gBounds.maxY - gBounds.minY) || 1;
  const baseScale = Math.min((W - 80) / bw, (H - 80) / bh) * gTransform.scale;
  const targetSX = W / 2, targetSY = H / 2;
  const ox = (gBounds.maxX - gBounds.minX) / 2 * baseScale / gTransform.scale;
  const oy = (gBounds.maxY - gBounds.minY) / 2 * baseScale / gTransform.scale;
  gTransform.x = -(wx - gBounds.minX) * baseScale / gTransform.scale + W / 2 - (W - 80) / 2 + 40 - ox / gTransform.scale;
  gTransform.y = (wy - gBounds.minY) * baseScale / gTransform.scale - H / 2 + (H - 80) / 2 + 40 + oy / gTransform.scale;
  renderAll();
}

// ═══════════════════════════════════════════════════════════════════
// ── CANVAS INTERACTION ───────────────────────────────────────────
// ═══════════════════════════════════════════════════════════════════
function setupInteraction() {
  const canvas = document.getElementById('planCanvas');
  canvas.onmousedown = startPan;
  canvas.onwheel = onWheel;
  canvas.ondblclick = onDblClick;
}

function startPan(e) {
  const canvas = document.getElementById('planCanvas');
  const startX = e.clientX - gTransform.x;
  const startY = e.clientY - gTransform.y;

  function onMove(e) {
    gTransform.x = e.clientX - startX;
    gTransform.y = e.clientY - startY;
    renderAll();
  }
  function onUp() {
    canvas.removeEventListener('mousemove', onMove);
    canvas.removeEventListener('mouseup', onUp);
  }
  canvas.addEventListener('mousemove', onMove);
  canvas.addEventListener('mouseup', onUp);
}

function onWheel(e) {
  e.preventDefault();
  const factor = e.deltaY < 0 ? 1.15 : 0.87;
  zoom(factor, e.offsetX, e.offsetY);
}

function zoom(factor, cx, cy) {
  const canvas = document.getElementById('planCanvas');
  if (!cx) cx = canvas.width / 2;
  if (!cy) cy = canvas.height / 2;
  const prevScale = gTransform.scale;
  gTransform.scale = Math.max(0.05, Math.min(50, gTransform.scale * factor));
  const scaledBy = gTransform.scale / prevScale;
  gTransform.x = cx + (gTransform.x - cx) * scaledBy;
  gTransform.y = cy + (gTransform.y - cy) * scaledBy;
  renderAll();
}

function fitView() {
  gTransform = { x: 0, y: 0, scale: 1 };
  renderAll();
}

function onDblClick(e) {
  // Click on lot to select it
  if (!gBounds || gLots.length === 0) return;
  const canvas = document.getElementById('planCanvas');
  const W = canvas.width, H = canvas.height;
  const bw = (gBounds.maxX - gBounds.minX) || 1;
  const bh = (gBounds.maxY - gBounds.minY) || 1;
  const baseScale = Math.min((W - 80) / bw, (H - 80) / bh) * gTransform.scale;
  const ox = gTransform.x + (W - bw * baseScale / gTransform.scale * gTransform.scale) / 2 - gBounds.minX * baseScale;
  const oy = gTransform.y + H - (H - bh * baseScale / gTransform.scale * gTransform.scale) / 2 + gBounds.minY * baseScale;

  // Convert click to world coords — use toScreen inverse
  // Instead: find lot whose centroid screen position is nearest to click
  let nearest = null, minDist = Infinity;
  for (const lot of gLots) {
    const s = toScreen(lot.center.x, lot.center.y);
    const d = Math.hypot(s.sx - e.offsetX, s.sy - e.offsetY);
    if (d < minDist) { minDist = d; nearest = lot; }
  }
  if (nearest && minDist < 80) {
    switchTab('lots');
    selectLot(nearest.id);
  }
}

// ═══════════════════════════════════════════════════════════════════
// ── UI ───────────────────────────────────────────────────────────
// ═══════════════════════════════════════════════════════════════════
function switchTab(tab) {
  document.getElementById('tabLots').classList.toggle('active', tab === 'lots');
  document.getElementById('tabLayers').classList.toggle('active', tab === 'layers');
  document.getElementById('panelLots').style.display    = tab === 'lots'   ? 'block' : 'none';
  document.getElementById('panelLayers').style.display  = tab === 'layers' ? 'block' : 'none';
}

// ═══════════════════════════════════════════════════════════════════
// ── EXPORT TO REGISTRY ───────────────────────────────────────────
// ═══════════════════════════════════════════════════════════════════
function importToRegistry() {
  if (gLots.length === 0) return;

  // Build localStorage data matching PROJET_1.html format
  const existing = JSON.parse(localStorage.getItem('lots') || '[]');
  const existingIds = new Set(existing.map(l => l.lotNum));

  const toAdd = gLots
    .filter(l => !existingIds.has(l.lotNum))
    .map(l => ({
      id:          Date.now() + '_' + l.id,
      lotNum:      l.lotNum,
      ilot:        l.ilot,
      zone:        l.zone,
      superficie:  l.superficie,
      utilisation: l.utilisation,
      statut:      l.statut,
      acheteur:    '',
      prix:        '',
    }));

  const merged = [...existing, ...toAdd];
  localStorage.setItem('lots', JSON.stringify(merged));

  // Feedback
  const btn = document.getElementById('btnImport');
  btn.textContent = `✅ ${toAdd.length} lots importés !`;
  btn.style.background = '#06e5a0';
  setTimeout(() => {
    btn.textContent = '⬆️ Importer dans le registre';
    btn.style.background = '';
  }, 3000);

  if (toAdd.length > 0) {
    // Offer to navigate
    setTimeout(() => {
      if (confirm(`${toAdd.length} lot(s) importé(s) avec succès.\nOuvrir le registre foncier maintenant ?`)) {
        window.location.href = 'PROJET_1.html';
      }
    }, 500);
  } else {
    alert('Tous les lots sont déjà dans le registre.');
  }
}

// ═══════════════════════════════════════════════════════════════════
// ── RESIZE ───────────────────────────────────────────────────────
// ═══════════════════════════════════════════════════════════════════
window.addEventListener('resize', () => {
  if (gEntities.length > 0) {
    resizeCanvas();
    renderAll();
  }
});
</script>
</body>
</html>
