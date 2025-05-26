---
layout: default
title: "Roll-PID Simulator"
permalink: /sim/
---
<link
  href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css"
  rel="stylesheet"
/>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
  /* copy your existing styles here */
  html, body {
    height: 100%;
    margin: 0;
    font-family: sans-serif;
    overflow: hidden;
  }
  .slider-label { font-size: 0.75rem; }
  input[type="range"].form-range { height: 0.9rem; background-color: #dee2e6; }
  input[type="range"].form-range::-webkit-slider-runnable-track,
  input[type="range"].form-range::-moz-range-track {
    height: 0.35rem;
    background: #0d6efd;
    border-radius: 0.25rem;
  }
  #droneCanvas {
    width: auto;
    height: auto;
    max-width: 100%;
    max-height: 40vh;
    aspect-ratio: 1/1;
    border: 1px solid #ccc;
  }
  .container-fluid { flex: 0 0 auto; }
  #chartContainer {
    flex: 1 1 0;
    min-height: 0;
    display: flex;
  }
  #chartCanvas {
    flex: 1;
    width: 100%;
    height: 100%;
    border: 1px solid #ccc;
  }
  .metric { font-size: 0.8rem; }
  .slider-set { max-width: 100%; height: auto; }
</style>

<div class="d-flex flex-column" style="height: 100vh;">
  <div class="container-fluid py-1 px-2">
    <h6 class="text-center mb-2">Quadcopter Roll PID Simulator</h6>
    <div class="row mb-2">
      <div class="col-md-6">
        <!-- sliders & buttons… -->
        <!-- (copy your existing markup from the `<head>` section) -->
      </div>
      <div class="col-md-6 d-flex justify-content-center align-items-center">
        <canvas id="droneCanvas" width="600" height="600"></canvas>
      </div>
    </div>
  </div>

  <div id="chartContainer">
    <canvas id="chartCanvas"></canvas>
  </div>
</div>

  <script>
    // update slider labels in real time
    document.querySelectorAll('input[type=range]').forEach(slider => {
      const span = document.getElementById(slider.id + 'Val');
      slider.addEventListener('input', () => {
        const prec = slider.step < 1 ? 2 : 0;
        span.textContent = parseFloat(slider.value).toFixed(prec);
      });
    });

    // initialize Chart.js
    const chartCtx = document.getElementById('chartCanvas').getContext('2d');
    const chart = new Chart(chartCtx, {
      type: 'line',
      data: {
        datasets: [
          { label: 'Angle (deg)', data: [], borderWidth: 2, fill: false, showLine: true, parsing: false },
          { label: 'Target',      data: [], borderDash: [5, 5], fill: false, showLine: true, parsing: false }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        animation: false,
        scales: {
          x: { type: 'linear', title: { display: true, text: 'Time (s)' } },
          y: { min: -90, max: 90, title: { display: true, text: 'Angle (deg)' } }
        }
      }
    });

    let timer;
    function startSim() {
      // reset
      chart.data.datasets.forEach(ds => ds.data = []);
      ['riseTime','overshoot','settleTime'].forEach(id => document.getElementById(id).textContent = 'N/A');
      document.getElementById('startBtn').disabled = true;
      document.getElementById('stopBtn').disabled  = false;

      let t = 0, angle = 0, omega = 0, errInt = 0, prevErr = 0, riseStart = null, maxAngle = -Infinity, lastTime = 0;
      const dt = 0.01, sphereR = 0.1, scale = 300;

      timer = setInterval(() => {
        // read params
        const Kp   = +document.getElementById('kp').value;
        const Ki   = +document.getElementById('ki').value;
        const Kd   = +document.getElementById('kd').value;
        const mass = +document.getElementById('mass').value;
        const arm  = +document.getElementById('arm').value;
        const tgt  = +document.getElementById('target').value;
        const I    = 0.4 * mass * sphereR * sphereR;

        // PID
        const des = tgt * Math.PI/180;
        const err = des - angle;
        errInt += err * dt;
        const dErr = (err - prevErr) / dt;
        const tau  = Kp*err + Ki*errInt + Kd*dErr;
        omega += (arm * tau / I) * dt;
        angle += omega * dt;
        prevErr = err;

        // performance metrics
        const deg = angle * 180/Math.PI;
        if (riseStart === null && deg >= 0.1 * tgt) riseStart = t;
        if (riseStart !== null && deg >= 0.9 * tgt && document.getElementById('riseTime').textContent === 'N/A') {
          document.getElementById('riseTime').textContent = (t - riseStart).toFixed(2) + 's';
        }
        maxAngle = Math.max(maxAngle, deg);
        if (tgt !== 0) {
          const overs = (maxAngle - tgt) / Math.abs(tgt) * 100;
          document.getElementById('overshoot').textContent = overs.toFixed(1) + '%';
        }
        const band = 0.02 * Math.abs(tgt);
        if (Math.abs(deg - tgt) > band) lastTime = t;
        document.getElementById('settleTime').textContent = lastTime.toFixed(2) + 's';

        // update chart
        chart.data.datasets[0].data.push({ x: t, y: deg });
        chart.data.datasets[1].data = [
          { x: 0, y: tgt },
          { x: Math.max(10, t), y: tgt }
        ];
        chart.options.scales.x.max = Math.max(10, t);
        chart.update();

        // draw the drone
        const dctx = document.getElementById('droneCanvas').getContext('2d');
        dctx.clearRect(0, 0, dctx.canvas.width, dctx.canvas.height);
        dctx.save();
        dctx.translate(dctx.canvas.width/2, dctx.canvas.height/2);
        dctx.rotate(angle);
        dctx.fillStyle = 'black';
        dctx.beginPath();
        dctx.arc(0, 0, sphereR * scale, 0, 2 * Math.PI);
        dctx.fill();
        dctx.strokeStyle = 'black';
        dctx.lineWidth = 4;
        dctx.beginPath();
        dctx.moveTo(-arm * scale / 2, 0);
        dctx.lineTo( arm * scale / 2, 0);
        dctx.stroke();
        const off = -arm * scale * 0.02, pLen = arm * scale * 0.02;
        [-arm * scale / 2, arm * scale / 2].forEach(x => {
          dctx.strokeStyle = 'red';
          dctx.lineWidth = arm * scale * 0.1;
          dctx.beginPath();
          dctx.moveTo(x, off + pLen/2);
          dctx.lineTo(x, off - pLen/2);
          dctx.stroke();
        });
        dctx.restore();

        t += dt;
      }, +document.getElementById('interval').value);
    }

    document.getElementById('startBtn').onclick = startSim;
    document.getElementById('stopBtn').onclick  = () => {
      clearInterval(timer);
      document.getElementById('startBtn').disabled = false;
      document.getElementById('stopBtn').disabled  = true;
    };
  </script>
</body>
</html>
