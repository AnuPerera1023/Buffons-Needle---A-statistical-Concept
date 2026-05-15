# Buffons-Needle---A-statistical-Concept

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Buffon's Needle — Estimate π</title>
<link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=Instrument+Serif:ital@0;1&family=DM+Sans:wght@300;400;500&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #0e0e10;
    --surface: #18181c;
    --surface2: #222228;
    --border: rgba(255,255,255,0.08);
    --border2: rgba(255,255,255,0.14);
    --text: #f0efe8;
    --muted: #888880;
    --accent: #c8f547;
    --accent2: #5dcaa5;
    --cross: #f0997b;
    --nocross: #85b7eb;
    --line-col: rgba(255,255,255,0.18);
  }
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  html, body { height: 100%; background: var(--bg); color: var(--text); font-family: 'DM Sans', sans-serif; overflow-x: hidden; }

  .shell { display: grid; grid-template-rows: auto 1fr auto; min-height: 100vh; max-width: 1100px; margin: 0 auto; padding: 0 24px; }

  header { padding: 28px 0 20px; display: flex; align-items: baseline; gap: 20px; border-bottom: 0.5px solid var(--border); }
  header h1 { font-family: 'Instrument Serif', serif; font-size: 32px; font-weight: 400; letter-spacing: -0.5px; }
  header h1 em { font-style: italic; color: var(--accent); }
  header .tagline { font-size: 13px; color: var(--muted); font-family: 'DM Mono', monospace; }

  main { display: grid; grid-template-columns: 1fr 280px; gap: 20px; padding: 20px 0; align-items: start; }

  .canvas-wrap { position: relative; }
  #board { display: block; width: 100%; border-radius: 12px; border: 0.5px solid var(--border2); background: var(--surface); cursor: crosshair; }

  .side { display: flex; flex-direction: column; gap: 14px; }

  .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
  .stat { background: var(--surface); border: 0.5px solid var(--border); border-radius: 10px; padding: 12px 14px; }
  .stat-label { font-size: 11px; font-family: 'DM Mono', monospace; color: var(--muted); text-transform: uppercase; letter-spacing: 0.07em; margin-bottom: 6px; }
  .stat-val { font-size: 26px; font-weight: 500; font-family: 'DM Mono', monospace; color: var(--text); line-height: 1; }
  .stat-val.good { color: var(--accent2); }
  .stat-val.warn { color: var(--cross); }
  .stat.wide { grid-column: 1 / -1; }
  .stat.wide .stat-val { font-size: 36px; }
  .pi-true { font-size: 11px; color: var(--muted); margin-top: 4px; font-family: 'DM Mono', monospace; }

  /* crossing probability panel */
  .prob-panel { background: var(--surface); border: 0.5px solid var(--border); border-radius: 10px; padding: 13px 14px; }
  .prob-panel-title { font-size: 11px; font-family: 'DM Mono', monospace; color: var(--muted); text-transform: uppercase; letter-spacing: 0.07em; margin-bottom: 10px; }
  .prob-row { display: flex; justify-content: space-between; align-items: center; padding: 7px 0; border-bottom: 0.5px solid var(--border); }
  .prob-row:last-child { border-bottom: none; padding-bottom: 0; }
  .prob-row:first-of-type { padding-top: 0; }
  .prob-key { font-size: 12px; font-family: 'DM Mono', monospace; color: var(--muted); }
  .prob-key em { font-style: normal; color: var(--text); }
  .prob-num { font-size: 18px; font-weight: 500; font-family: 'DM Mono', monospace; }
  .prob-num.theory { color: var(--cross); }
  .prob-num.expt { color: var(--accent2); }
  .prob-num.expt.good { color: var(--accent); }
  .prob-num.expt.warn { color: var(--cross); }
  .prob-arrow { font-size: 11px; font-family: 'DM Mono', monospace; color: var(--muted); margin-top: 4px; text-align: center; }

  .controls { background: var(--surface); border: 0.5px solid var(--border); border-radius: 10px; padding: 14px; display: flex; flex-direction: column; gap: 10px; }
  .ctrl-label { font-size: 11px; font-family: 'DM Mono', monospace; color: var(--muted); text-transform: uppercase; letter-spacing: 0.07em; margin-bottom: 2px; }
  .btn-row { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 6px; }
  .btn { background: var(--surface2); color: var(--text); border: 0.5px solid var(--border2); border-radius: 8px; padding: 9px 6px; font-size: 13px; font-family: 'DM Sans', sans-serif; cursor: pointer; transition: background 0.15s, transform 0.1s; text-align: center; }
  .btn:hover { background: rgba(255,255,255,0.07); }
  .btn:active { transform: scale(0.97); }
  .btn.accent { background: var(--accent); color: #0e0e10; border-color: var(--accent); font-weight: 500; }
  .btn.accent:hover { background: #d4fc5a; }
  .btn.danger { background: rgba(240,153,123,0.12); color: var(--cross); border-color: rgba(240,153,123,0.25); }
  .btn.danger:hover { background: rgba(240,153,123,0.2); }
  .btn.stop { background: rgba(240,153,123,0.18); color: var(--cross); border-color: rgba(240,153,123,0.3); font-weight: 500; }
  .btn-wide { grid-column: 1 / -1; }

  .slider-row { display: flex; align-items: center; gap: 10px; }
  .slider-row input[type=range] { flex: 1; accent-color: var(--accent); cursor: pointer; }
  .slider-val { font-family: 'DM Mono', monospace; font-size: 12px; color: var(--accent); min-width: 32px; text-align: right; }

  .legend { display: flex; gap: 14px; flex-wrap: wrap; }
  .leg-item { display: flex; align-items: center; gap: 6px; font-size: 12px; color: var(--muted); }
  .leg-dot { width: 18px; height: 3px; border-radius: 2px; }

  .formula { background: var(--surface); border: 0.5px solid var(--border); border-radius: 10px; padding: 12px 14px; }
  .formula-line { font-family: 'DM Mono', monospace; font-size: 13px; color: var(--muted); line-height: 1.9; }
  .formula-line span { color: var(--accent); }

  .chart-wrap { background: var(--surface); border: 0.5px solid var(--border); border-radius: 10px; padding: 12px 14px; }
  .chart-title { font-size: 11px; font-family: 'DM Mono', monospace; color: var(--muted); text-transform: uppercase; letter-spacing: 0.07em; margin-bottom: 8px; }
  #chart { display: block; width: 100%; }

  footer { border-top: 0.5px solid var(--border); padding: 14px 0; display: flex; justify-content: space-between; align-items: center; font-size: 12px; color: var(--muted); font-family: 'DM Mono', monospace; }

  #toast { position: fixed; bottom: 30px; left: 50%; transform: translateX(-50%) translateY(20px); background: var(--surface2); border: 0.5px solid var(--border2); border-radius: 8px; padding: 10px 20px; font-size: 13px; font-family: 'DM Mono', monospace; color: var(--accent); opacity: 0; transition: opacity 0.3s, transform 0.3s; pointer-events: none; white-space: nowrap; }
  #toast.show { opacity: 1; transform: translateX(-50%) translateY(0); }

  #milestone { position: absolute; top: 12px; right: 12px; background: var(--accent); color: #0e0e10; border-radius: 20px; padding: 4px 14px; font-size: 12px; font-weight: 500; font-family: 'DM Mono', monospace; opacity: 0; transition: opacity 0.4s; pointer-events: none; }
  #milestone.show { opacity: 1; }

  @media (max-width: 720px) {
    main { grid-template-columns: 1fr; }
    .side { flex-direction: row; flex-wrap: wrap; }
    .stats-grid { min-width: 200px; flex: 1; }
    .controls { min-width: 200px; flex: 1; }
  }
</style>
</head>
<body>
<div class="shell">

  <header>
    <h1>Buffon's <em>Needle</em></h1>
    <span class="tagline">// estimating π with randomness</span>
  </header>

  <main>
    <div class="canvas-wrap">
      <canvas id="board"></canvas>
      <div id="milestone"></div>
      <div style="display:flex; justify-content:space-between; align-items:center; margin-top:8px;">
        <div class="legend">
          <div class="leg-item"><div class="leg-dot" style="background:var(--cross)"></div>crosses line</div>
          <div class="leg-item"><div class="leg-dot" style="background:var(--nocross)"></div>no crossing</div>
          <div class="leg-item"><div class="leg-dot" style="background:var(--line-col); width:24px;"></div>parallel lines (D = 52px)</div>
        </div>
        <span style="font-size:12px; color:var(--muted); font-family:'DM Mono',monospace;">click board to drop manually</span>
      </div>

      <div class="chart-wrap" style="margin-top:14px;">
        <div class="chart-title">π estimate convergence</div>
        <canvas id="chart" height="90"></canvas>
      </div>
    </div>

    <div class="side">

      <!-- estimated pi -->
      <div class="stats-grid">
        <div class="stat wide">
          <div class="stat-label">Estimated π</div>
          <div class="stat-val" id="m-pi">—</div>
          <div class="pi-true">true π = 3.14159265...</div>
        </div>
        <div class="stat">
          <div class="stat-label">Drops (N)</div>
          <div class="stat-val" id="m-n">0</div>
        </div>
        <div class="stat">
          <div class="stat-label">Crossings (C)</div>
          <div class="stat-val" id="m-c">0</div>
        </div>
      </div>

      <!-- crossing probability comparison -->
      <div class="prob-panel">
        <div class="prob-panel-title">Crossing probability</div>
        <div class="prob-row">
          <div>
            <div class="prob-key">theoretical &nbsp;<em>2 / π</em></div>
          </div>
          <div class="prob-num theory">0.6366</div>
        </div>
        <div class="prob-row">
          <div>
            <div class="prob-key">estimated &nbsp;<em>C / N</em></div>
          </div>
          <div class="prob-num expt" id="m-exp-prob">—</div>
        </div>
        <div class="prob-arrow" id="m-prob-diff"></div>
      </div>

      <!-- controls -->
      <div class="controls">
        <div class="ctrl-label">Drop needles</div>
        <div class="btn-row">
          <button class="btn" id="btn1">+1</button>
          <button class="btn" id="btn10">+10</button>
          <button class="btn" id="btn100">+100</button>
        </div>
        <button class="btn btn-wide accent" id="btn-auto">▶  Auto-drop</button>
        <div>
          <div class="ctrl-label" style="margin-bottom:6px;">Speed</div>
          <div class="slider-row">
            <input type="range" id="speed" min="1" max="30" value="8" step="1">
            <span class="slider-val" id="speed-out">8/s</span>
          </div>
        </div>
        <button class="btn btn-wide danger" id="btn-reset">↺  Reset</button>
      </div>

      <!-- formula -->
      <div class="formula">
        <div class="ctrl-label" style="margin-bottom:8px;">The math</div>
        <div class="formula-line">When L = D:</div>
        <div class="formula-line">P(cross) = <span>2 / π ≈ 0.6366</span></div>
        <div class="formula-line">P ≈ C / N</div>
        <div class="formula-line">∴  π ≈ <span>2N / C</span></div>
      </div>

    </div>
  </main>

  <footer>
    <span>Seminar Course · Buffon's Needle Experiment · May 2026</span>
    <span id="footer-stat">waiting for first drop...</span>
  </footer>
</div>

<div id="toast"></div>

<script>
(function () {
  const board = document.getElementById('board');
  const ctx = board.getContext('2d');
  const chartEl = document.getElementById('chart');
  const cCtx = chartEl.getContext('2d');

  const LINE_SPACING = 52;
  const NEEDLE = 52;
  const TRUE_PI = Math.PI;
  const THEORY_PROB = 2 / Math.PI;  // 0.63662...

  let W = 0, H = 0, numLines = 0;
  let drops = [];
  let N = 0, C = 0;
  let piHist = [];
  let autoTimer = null;
  let isAuto = false;
  let lastMilestone = 0;

  const MILESTONES = [50, 100, 200, 500, 1000, 2000, 5000];

  function resize() {
    const rect = board.parentElement.getBoundingClientRect();
    W = Math.floor(rect.width);
    H = Math.max(320, Math.floor(window.innerHeight * 0.48));
    board.width = W;
    board.height = H;
    chartEl.width = Math.floor(rect.width);
    numLines = Math.ceil(H / LINE_SPACING) + 1;
    redrawAll();
    drawChart();
  }

  function lineY(i) {
    return ((H % LINE_SPACING) / 2) + i * LINE_SPACING;
  }

  function drawLines() {
    ctx.strokeStyle = 'rgba(255,255,255,0.14)';
    ctx.lineWidth = 1;
    for (let i = 0; i < numLines; i++) {
      const y = lineY(i);
      ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(W, y); ctx.stroke();
    }
  }

  function doesCross(x, y, angle) {
    const hh = (NEEDLE / 2) * Math.abs(Math.sin(angle));
    const y1 = y - hh, y2 = y + hh;
    for (let i = 0; i < numLines; i++) {
      const ly = lineY(i);
      if (ly >= y1 && ly <= y2) return true;
    }
    return false;
  }

  function drawNeedle(d, highlight) {
    const hw = (NEEDLE / 2) * Math.cos(d.angle);
    const hh = (NEEDLE / 2) * Math.sin(d.angle);
    ctx.strokeStyle = highlight ? '#c8f547' : (d.crosses ? '#f0997b' : '#85b7eb');
    ctx.globalAlpha = highlight ? 1 : 0.72;
    ctx.lineWidth = highlight ? 2.5 : 1.5;
    ctx.beginPath();
    ctx.moveTo(d.x - hw, d.y - hh);
    ctx.lineTo(d.x + hw, d.y + hh);
    ctx.stroke();
    ctx.globalAlpha = 1;
    if (highlight) {
      ctx.fillStyle = '#c8f547';
      ctx.beginPath();
      ctx.arc(d.x, d.y, 3, 0, Math.PI * 2);
      ctx.fill();
    }
  }

  function redrawAll() {
    ctx.clearRect(0, 0, W, H);
    drawLines();
    for (let i = 0; i < drops.length; i++) drawNeedle(drops[i], i === drops.length - 1);
  }

  function drop(px, py) {
    const x = px !== undefined ? px : Math.random() * W;
    const y = py !== undefined ? py : Math.random() * H;
    const angle = Math.random() * Math.PI;
    const crosses = doesCross(x, y, angle);
    const d = { x, y, angle, crosses };
    drops.push(d);
    N++;
    if (crosses) C++;
    piHist.push(C > 0 ? 2 * N / C : null);
    drawNeedle(d, true);
    if (drops.length > 1) drawNeedle(drops[drops.length - 2], false);
    updateStats();
    drawChart();
    checkMilestone();
  }

  function checkMilestone() {
    const ms = MILESTONES.find(m => N >= m && m > lastMilestone);
    if (ms) {
      lastMilestone = ms;
      const el = document.getElementById('milestone');
      const est = C > 0 ? (2 * N / C).toFixed(5) : '—';
      el.textContent = ms + ' drops → π ≈ ' + est;
      el.classList.add('show');
      setTimeout(() => el.classList.remove('show'), 2800);
    }
  }

  function toast(msg) {
    const el = document.getElementById('toast');
    el.textContent = msg;
    el.classList.add('show');
    setTimeout(() => el.classList.remove('show'), 2200);
  }

  function updateStats() {
    document.getElementById('m-n').textContent = N.toLocaleString();
    document.getElementById('m-c').textContent = C.toLocaleString();

    if (N === 0) {
      document.getElementById('m-pi').textContent = '—';
      document.getElementById('m-exp-prob').textContent = '—';
      document.getElementById('m-prob-diff').textContent = '';
      document.getElementById('footer-stat').textContent = 'waiting for first drop...';
      return;
    }

    // experimental crossing probability
    const expProb = C / N;
    const diff = expProb - THEORY_PROB;
    const absDiff = Math.abs(diff);

    const expEl = document.getElementById('m-exp-prob');
    expEl.textContent = expProb.toFixed(4);
    expEl.className = 'prob-num expt' + (absDiff < 0.02 ? ' good' : absDiff > 0.08 ? ' warn' : '');

    const diffEl = document.getElementById('m-prob-diff');
    if (N >= 5) {
      const sign = diff >= 0 ? '+' : '';
      diffEl.textContent = 'diff from theory: ' + sign + diff.toFixed(4);
    } else {
      diffEl.textContent = '';
    }

    if (C === 0) {
      document.getElementById('m-pi').textContent = '—';
      return;
    }

    const est = 2 * N / C;
    const errPct = Math.abs(est - TRUE_PI) / TRUE_PI * 100;
    const piEl = document.getElementById('m-pi');
    piEl.textContent = est.toFixed(5);
    piEl.className = 'stat-val' + (errPct < 1 ? ' good' : errPct > 5 ? ' warn' : '');

    document.getElementById('footer-stat').textContent =
      'π ≈ ' + est.toFixed(6) + '  |  est. P(cross) = ' + expProb.toFixed(4) + '  |  theory = ' + THEORY_PROB.toFixed(4);
  }

  function drawChart() {
    const cW = chartEl.width;
    const cH = 90;
    cCtx.clearRect(0, 0, cW, cH);

    const valid = piHist.filter(v => v !== null);
    if (valid.length < 3) return;

    const pad = { t: 8, b: 20, l: 48, r: 12 };
    const iW = cW - pad.l - pad.r;
    const iH = cH - pad.t - pad.b;
    const minV = Math.max(1.5, Math.min(...valid) - 0.15);
    const maxV = Math.min(5.5, Math.max(...valid) + 0.15);

    function fx(i) { return pad.l + (i / (piHist.length - 1)) * iW; }
    function fy(v) { return pad.t + (1 - (v - minV) / (maxV - minV)) * iH; }

    cCtx.strokeStyle = 'rgba(255,255,255,0.06)';
    cCtx.lineWidth = 0.5;
    [minV, TRUE_PI, maxV].forEach(v => {
      cCtx.beginPath(); cCtx.moveTo(pad.l, fy(v)); cCtx.lineTo(cW - pad.r, fy(v)); cCtx.stroke();
    });

    const pyT = fy(TRUE_PI);
    cCtx.strokeStyle = 'rgba(240,153,123,0.5)';
    cCtx.lineWidth = 1;
    cCtx.setLineDash([5, 4]);
    cCtx.beginPath(); cCtx.moveTo(pad.l, pyT); cCtx.lineTo(cW - pad.r, pyT); cCtx.stroke();
    cCtx.setLineDash([]);

    cCtx.strokeStyle = '#c8f547';
    cCtx.lineWidth = 1.5;
    cCtx.beginPath();
    let started = false;
    for (let i = 0; i < piHist.length; i++) {
      if (piHist[i] === null) continue;
      const x = fx(i), y = fy(piHist[i]);
      if (!started) { cCtx.moveTo(x, y); started = true; }
      else cCtx.lineTo(x, y);
    }
    cCtx.stroke();

    cCtx.font = '11px DM Mono, monospace';
    cCtx.fillStyle = 'rgba(240,153,123,0.7)';
    cCtx.fillText('π', pad.l - 14, pyT + 4);
    cCtx.fillStyle = 'rgba(255,255,255,0.3)';
    cCtx.fillText(minV.toFixed(1), 2, fy(minV) + 4);
    cCtx.fillText(maxV.toFixed(1), 2, fy(maxV) + 4);
    cCtx.fillStyle = 'rgba(200,245,71,0.5)';
    cCtx.fillText('est.', pad.l + 4, pad.t + 12);
    cCtx.fillStyle = 'rgba(255,255,255,0.2)';
    cCtx.fillText('n=' + N.toLocaleString(), cW - pad.r - 46, cH - 4);
  }

  function startAuto() {
    isAuto = true;
    const btn = document.getElementById('btn-auto');
    btn.textContent = '■  Stop';
    btn.classList.add('stop');
    btn.classList.remove('accent');
    const spd = parseInt(document.getElementById('speed').value);
    autoTimer = setInterval(() => { drop(); }, Math.max(20, Math.round(1000 / spd)));
  }

  function stopAuto() {
    isAuto = false;
    clearInterval(autoTimer);
    const btn = document.getElementById('btn-auto');
    btn.textContent = '▶  Auto-drop';
    btn.classList.remove('stop');
    btn.classList.add('accent');
  }

  function reset() {
    stopAuto();
    drops = []; N = 0; C = 0; piHist = []; lastMilestone = 0;
    ctx.clearRect(0, 0, W, H);
    drawLines();
    drawChart();
    updateStats();
    toast('reset — ready for new experiment');
  }

  board.addEventListener('click', e => {
    const r = board.getBoundingClientRect();
    drop((e.clientX - r.left) * (W / r.width), (e.clientY - r.top) * (H / r.height));
  });

  document.getElementById('btn1').addEventListener('click', () => drop());
  document.getElementById('btn10').addEventListener('click', () => { for (let i = 0; i < 10; i++) drop(); });
  document.getElementById('btn100').addEventListener('click', () => { for (let i = 0; i < 100; i++) drop(); redrawAll(); });
  document.getElementById('btn-auto').addEventListener('click', () => isAuto ? stopAuto() : startAuto());
  document.getElementById('btn-reset').addEventListener('click', reset);

  const speedSlider = document.getElementById('speed');
  speedSlider.addEventListener('input', () => {
    document.getElementById('speed-out').textContent = speedSlider.value + '/s';
    if (isAuto) { stopAuto(); startAuto(); }
  });

  window.addEventListener('resize', () => { clearTimeout(window._rt); window._rt = setTimeout(resize, 120); });
  resize();
  toast('click the board or use buttons to drop needles');
})();
</script>
</body>
</html>
