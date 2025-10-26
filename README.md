<html lang="fr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Roue interactive – v7 (overlay persistant + close)</title>
<style>
  :root { --bg:#f7f7f7; --fg:#111; --ring:#e5e7eb; }
  html,body{margin:0;height:100%;background:var(--bg);color:var(--fg);font-family:Helvetica,Arial,sans-serif;}
  .wrap{min-height:100%;display:grid;place-items:center;padding:16px;}
  .container{width:min(92vw,650px);display:grid;gap:10px;justify-items:center;}
  h1{margin:0;font-size:clamp(14px,2.4vw,18px);font-weight:700;letter-spacing:.2px;text-transform:none;}
  .board{background:#fff;border:1px solid var(--ring);border-radius:10px;padding:10px;display:grid;gap:8px;justify-items:center;}
  .wheel-area{position:relative; width:100%; display:grid; place-items:center;}
  /* Pointer at the TOP, rotated so the triangle visually points downward */
  .pointer{
    position:absolute; top:-2px; left:50%; transform:translateX(-50%) rotate(180deg);
    width:0;height:0;border-left:12px solid transparent;border-right:12px solid transparent;border-bottom:20px solid #1f2937;
    z-index:3;
  }
  canvas{width:min(58vw,540px);max-width:540px;aspect-ratio:1/1;display:block;border-radius:50%;background:#fafafa;box-shadow:inset 0 0 0 5px #f0f0f0; z-index:1;}
  .controls{display:flex;gap:8px;align-items:center;justify-content:center;flex-wrap:wrap;}
  button.primary{background:#111;color:#fff;border:none;border-radius:8px;padding:8px 12px;font-size:12px;font-weight:700;cursor:pointer;}
  button.primary:disabled{opacity:.5;cursor:not-allowed;}
  .legend{font-size:9px;opacity:.7;}

  /* Overlay result centered; persists until closed */
  .overlay{
    position:absolute; inset:0; display:none; align-items:center; justify-content:center;
    z-index:2; pointer-events:auto;
  }
  .overlay .bubble{
    position:relative;
    max-width:78%; padding:16px 18px; border-radius:12px; background:#111; color:#fff;
    box-shadow:0 8px 24px rgba(0,0,0,.25);
    font-size:clamp(16px,2.8vw,24px); font-weight:700; text-align:center; line-height:1.25;
  }
  .overlay .close{
    position:absolute; top:6px; right:8px; width:26px; height:26px; border-radius:50%;
    border:1px solid rgba(255,255,255,.25); background:rgba(255,255,255,.08); color:#fff;
    display:grid; place-items:center; cursor:pointer; font-size:16px; line-height:1;
  }
</style>
</head>
<body>
<div class="wrap">
  <div class="container">
    <h1>modèle social français</h1>
    <div class="board">
      <div class="wheel-area">
        <div class="pointer"></div>
        <canvas id="wheel" width="800" height="800" aria-label="Roue de tirage"></canvas>
        <div id="overlay" class="overlay">
          <div class="bubble">
            <div id="overlayText"></div>
            <div id="overlayClose" class="close" title="Fermer">×</div>
          </div>
        </div>
      </div>
      <div class="controls">
        <button id="spinBtn" class="primary">Tourner la roue</button>
        <span class="legend" id="countInfo"></span>
      </div>
    </div>
  </div>
</div>

<script>
let ENTRIES = ["Droits agricoles Recette : 179,7 millions d'euros Date de création : 1962", "Droits de douane Recette : 1605,3 millions d'euros Date de création : 1970", "Taxe à la production sur l[...]"];

// ===== Canvas setup =====
const canvas = document.getElementById('wheel');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;
const CX = W/2, CY = H/2;
const R = Math.min(W, H) * 0.48;

let angle = -Math.PI/2; // pointer at top
let spinning = false;
let colors = [];
let lastPointerIndex = null;
let lastPingTime = 0;

// ===== Colors (flashy) =====
function buildColors(n){
  const arr = [];
  for(let i=0;i<n;i++){
    const hue = (i*360/n)%360;
    arr.push(`hsl(${hue}deg, 88%, 52%)`);
  }
  return arr;
}

// ===== Font sizing =====
function computeFont(n){
  if(n>340) return 7;
  if(n>260) return 8;
  if(n>200) return 9;
  if(n>140) return 10;
  return 11;
}

// ===== Draw wheel =====
function drawWheel(a){
  ctx.clearRect(0,0,W,H);
  const n = ENTRIES.length;
  if(n===0){ return; }
  const step = (Math.PI*2)/n;

  ctx.save();
  ctx.translate(CX, CY);
  ctx.rotate(a);

  for(let i=0;i<n;i++){
    const start = i*step, end = start+step;
    ctx.beginPath();
    ctx.moveTo(0,0);
    ctx.arc(0,0,R,start,end);
    ctx.closePath();
    ctx.fillStyle = colors[i % colors.length];
    ctx.fill();
    if(n<=600){
      ctx.strokeStyle = "rgba(0,0,0,.18)";
      ctx.lineWidth = 0.5;
      ctx.stroke();
    }
  }

  // Labels
  const fontPx = computeFont(n);
  ctx.fillStyle = "#111";
  ctx.textAlign = "center";
  ctx.textBaseline = "middle";
  ctx.font = `${fontPx}px Helvetica, Arial, sans-serif`;
  const labelRadius = R*0.78;
  for(let i=0;i<n;i++){
    const mid = (i+0.5)*step;
    const x = Math.cos(mid)*labelRadius;
    const y = Math.sin(mid)*labelRadius;
    const raw = String(ENTRIES[i] ?? "");
    const label = raw.length>36 ? raw.slice(0,36)+"…" : raw;
    ctx.save();
    ctx.translate(x,y);
    ctx.rotate(mid);
    ctx.fillText(label,0,0);
    ctx.restore();
  }

  // Center cap
  ctx.beginPath();
  ctx.arc(0,0,R*0.12,0,Math.PI*2);
  ctx.fillStyle = "#fff";
  ctx.strokeStyle = "#e5e7eb";
  ctx.lineWidth = 1;
  ctx.fill();
  ctx.stroke();

  ctx.restore();
}

// ===== Selection at pointer =====
function getSelectedIndex(a){
  const n = ENTRIES.length;
  const step = (Math.PI*2)/n;
  let theta = (-Math.PI/2 - a) % (Math.PI*2);
  if(theta<0) theta += Math.PI*2;
  return Math.floor(theta/step);
}

// ===== Animation ease =====
function easeInOutCubic(x){ return x<0.5 ? 4*x*x*x : 1 - Math.pow(-2*x + 2, 3)/2; }

// ===== Overlay control =====
const overlay = document.getElementById('overlay');
const overlayText = document.getElementById('overlayText');
const overlayClose = document.getElementById('overlayClose');
function showOverlay(text){
  overlayText.textContent = text;
  overlay.style.display = 'flex';
}
overlayClose.addEventListener('click', () => {
  overlay.style.display = 'none';
});

// ===== Audio (WebAudio): "ping" per segment + coin at end =====
let audioCtx = null;
function ensureAudio(){
  if(!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
}
function ping(){
  ensureAudio();
  const t = audioCtx.currentTime;
  const osc = audioCtx.createOscillator();
  const g = audioCtx.createGain();
  osc.type = "sine";
  osc.frequency.setValueAtTime(1000 + Math.random()*120, t);
  g.gain.setValueAtTime(0.04, t);
  g.gain.exponentialRampToValueAtTime(0.0001, t + 0.08);
  osc.connect(g); g.connect(audioCtx.destination);
  osc.start(t);
  osc.stop(t + 0.1);
}
function coin(){
  ensureAudio();
  const t = audioCtx.currentTime;
  const o1 = audioCtx.createOscillator();
  const g1 = audioCtx.createGain();
  o1.type = "triangle";
  o1.frequency.setValueAtTime(1500, t);
  g1.gain.setValueAtTime(0.08, t);
  g1.gain.exponentialRampToValueAtTime(0.0001, t + 0.25);
  o1.connect(g1); g1.connect(audioCtx.destination);
  o1.start(t); o1.stop(t+0.27);
}

// ===== Spin =====
function spin(){
  if(spinning) return;
  if(ENTRIES.length===0){ alert("Plus aucun élément à tirer."); return; }
  spinning = true;
  const btn = document.getElementById('spinBtn');
  btn.disabled = true;

  const duration = 4000;
  const totalTurns = 4 + Math.random()*2;
  const finalOffset = Math.random() * Math.PI * 2;
  const totalAngle = totalTurns * Math.PI*2 + finalOffset;
  const start = performance.now();
  lastPointerIndex = getSelectedIndex(angle);
  lastPingTime = start;

  function animate(ts){
    const t = Math.min(1, (ts - start) / duration);
    const eased = easeInOutCubic(t);
    angle = -Math.PI/2 + eased * totalAngle;
    drawWheel(angle);

    // Segment crossing ping (throttled)
    const idx = getSelectedIndex(angle);
    if(idx !== lastPointerIndex && (ts - lastPingTime) > 25){
      try { ping(); } catch(e) {}
      lastPointerIndex = idx;
      lastPingTime = ts;
    }

    if(t<1){
      requestAnimationFrame(animate);
    }else{
      const winnerIdx = getSelectedIndex(angle);
      const chosen = ENTRIES[winnerIdx];
      try { coin(); } catch(e) {}
      showOverlay("🎯 " + chosen);

      // Remove winner (sans remise)
      ENTRIES.splice(winnerIdx,1);
      angle = -Math.PI/2;
      colors = buildColors(ENTRIES.length);
      document.getElementById('countInfo').textContent = ENTRIES.length + " éléments restants";
      drawWheel(angle);
      btn.disabled = ENTRIES.length===0;
      spinning = false;
    }
  }
  requestAnimationFrame(animate);
}

document.getElementById('spinBtn').addEventListener('click', spin);

// ===== Init =====
colors = buildColors(ENTRIES.length);
document.getElementById('countInfo').textContent = ENTRIES.length + " éléments";
drawWheel(angle);
</script>
</body>
</html>
