# virtual-laboratory-01
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Virtual Lab - Getaran & Gelombang</title>
  <style>
    * { box-sizing: border-box; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 0; padding: 0; }
    body { background-color: #f0f4f8; color: #333; padding: 20px; }
    .container { max-width: 900px; margin: 0 auto; background: white; border-radius: 12px; padding: 20px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
    h1 { text-align: center; color: #0284c7; margin-bottom: 20px; font-size: 1.5rem; }
    
    .lab-layout { display: flex; flex-wrap: wrap; gap: 20px; }
    .canvas-container { flex: 1; min-width: 300px; background: #1e293b; border-radius: 8px; display: flex; justify-content: center; align-items: center; position: relative; height: 350px; }
    canvas { background: transparent; }
    
    .controls { flex: 1; min-width: 280px; display: flex; flex-direction: column; gap: 15px; }
    .control-group { background: #f8fafc; padding: 12px; border-radius: 8px; border: 1px solid #e2e8f0; }
    label { font-weight: bold; font-size: 0.9rem; display: block; margin-bottom: 5px; color: #475569; }
    input[type=range] { width: 100%; margin-top: 5px; }
    
    .btn-group { display: flex; gap: 10px; margin-top: 10px; }
    button { flex: 1; padding: 10px; border: none; border-radius: 6px; font-weight: bold; cursor: pointer; transition: 0.2s; }
    .btn-start { background-color: #10b981; color: white; }
    .btn-reset { background-color: #ef4444; color: white; }
    button:hover { opacity: 0.9; }
    
    .data-panel { margin-top: 20px; background: #e0f2fe; padding: 15px; border-radius: 8px; border-left: 5px solid #0284c7; }
    .data-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(130px, 1fr)); gap: 10px; margin-top: 8px; }
    .data-card { background: white; padding: 10px; border-radius: 6px; text-align: center; }
    .data-value { font-size: 1.2rem; font-weight: bold; color: #0369a1; }
  </style>
</head>
<body>

<div class="container">
  <h1>Virtual Lab: Getaran Pendulum Sederhana</h1>
  
  <div class="lab-layout">
    <!-- Area Simulasi Canvas -->
    <div class="canvas-container">
      <canvas id="simCanvas" width="350" height="330"></canvas>
    </div>
    
    <!-- Control Panel -->
    <div class="controls">
      <div class="control-group">
        <label for="length">Panjang Tali (L): <span id="valLength">1.0</span> m</label>
        <input type="range" id="length" min="0.2" max="2.0" step="0.1" value="1.0">
      </div>
      
      <div class="control-group">
        <label for="angle">Sudut Simpangan Awal (θ): <span id="valAngle">15</span>°</label>
        <input type="range" id="angle" min="5" max="30" step="1" value="15">
      </div>

      <div class="control-group">
        <label for="gravity">Percepatan Gravitasi (g): <span id="valGravity">9.8</span> m/s²</label>
        <input type="range" id="gravity" min="1.6" max="20.0" step="0.1" value="9.8">
      </div>
      
      <div class="btn-group">
        <button class="btn-start" id="btnPlay">Mulai / Jeda</button>
        <button class="btn-reset" id="btnReset">Reset</button>
      </div>
    </div>
  </div>

  <!-- Panel Output Data -->
  <div class="data-panel">
    <h3>Hasil Pengamatan (Real-Time)</h3>
    <div class="data-grid">
      <div class="data-card">
        <div>Periode (T)</div>
        <div class="data-value" id="outT">0.00 s</div>
      </div>
      <div class="data-card">
        <div>Frekuensi (f)</div>
        <div class="data-value" id="outF">0.00 Hz</div>
      </div>
      <div class="data-card">
        <div>Waktu (t)</div>
        <div class="data-value" id="outTime">0.00 s</div>
      </div>
    </div>
  </div>
</div>

<script>
  const canvas = document.getElementById('simCanvas');
  const ctx = canvas.getContext('2d');

  // Element Inputs
  const inputL = document.getElementById('length');
  const inputAngle = document.getElementById('angle');
  const inputG = document.getElementById('gravity');
  const btnPlay = document.getElementById('btnPlay');
  const btnReset = document.getElementById('btnReset');

  // Parameters
  let L = parseFloat(inputL.value);
  let g = parseFloat(inputG.value);
  let initialAngle = parseFloat(inputAngle.value) * Math.PI / 180;
  
  let angle = initialAngle;
  let angleVelocity = 0;
  let angleAccel = 0;
  let isRunning = false;
  let time = 0;
  const dt = 0.02; // Time step

  // Anchor Point
  const originX = canvas.width / 2;
  const originY = 40;

  function updatePhysics() {
    if (!isRunning) return;
    
    // Simple pendulum equation: alpha = -(g/L) * sin(theta)
    angleAccel = (-1 * g / (L * 150)) * Math.sin(angle); // 150 is scale pixel factor
    angleVelocity += angleAccel;
    angle += angleVelocity;
    
    time += dt;
    
    // Output calculations
    const T = 2 * Math.PI * Math.sqrt(L / g);
    const f = 1 / T;
    
    document.getElementById('outT').innerText = T.toFixed(2) + " s";
    document.getElementById('outF').innerText = f.toFixed(2) + " Hz";
    document.getElementById('outTime').innerText = time.toFixed(1) + " s";
  }

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Calculate pendulum bob position
    const pixelL = L * 120; // Visual scale
    const x = originX + pixelL * Math.sin(angle);
    const y = originY + pixelL * Math.cos(angle);

    // Draw Support Base
    ctx.fillStyle = '#64748b';
    ctx.fillRect(originX - 40, originY - 10, 80, 10);

    // Draw String
    ctx.beginPath();
    ctx.moveTo(originX, originY);
    ctx.lineTo(x, y);
    ctx.strokeStyle = '#cbd5e1';
    ctx.lineWidth = 2;
    ctx.stroke();

    // Draw Bob
    ctx.beginPath();
    ctx.arc(x, y, 16, 0, Math.PI * 2);
    ctx.fillStyle = '#38bdf8';
    ctx.fill();
    ctx.strokeStyle = '#0284c7';
    ctx.lineWidth = 2;
    ctx.stroke();
  }

  function loop() {
    updatePhysics();
    draw();
    requestAnimationFrame(loop);
  }

  // Listeners
  inputL.addEventListener('input', (e) => {
    L = parseFloat(e.target.value);
    document.getElementById('valLength').innerText = L;
    resetSim();
  });

  inputAngle.addEventListener('input', (e) => {
    initialAngle = parseFloat(e.target.value) * Math.PI / 180;
    document.getElementById('valAngle').innerText = e.target.value;
    resetSim();
  });

  inputG.addEventListener('input', (e) => {
    g = parseFloat(e.target.value);
    document.getElementById('valGravity').innerText = g;
    resetSim();
  });

  btnPlay.addEventListener('click', () => { isRunning = !isRunning; });
  btnReset.addEventListener('click', resetSim);

  function resetSim() {
    angle = initialAngle;
    angleVelocity = 0;
    time = 0;
    isRunning = false;
    document.getElementById('outTime').innerText = "0.00 s";
    
    const T = 2 * Math.PI * Math.sqrt(L / g);
    document.getElementById('outT').innerText = T.toFixed(2) + " s";
    document.getElementById('outF').innerText = (1/T).toFixed(2) + " Hz";
  }

  resetSim();
  loop();
</script>
</body>
</html>
