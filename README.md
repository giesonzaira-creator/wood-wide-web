<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Wood Wide Web</title>
  <style>
    :root {
      --bg: #0f1a14;
      --panel: #16261d;
      --accent: #7ccf9a;
      --accent-soft: #4fa87a;
      --text: #e7f5ec;
      --muted: #9fbfb0;
    }

    * { box-sizing: border-box; }

    body {
      margin: 0;
      min-height: 100vh;
      font-family: system-ui, -apple-system, Segoe UI, Roboto, sans-serif;
      background: radial-gradient(circle at top, #173326, var(--bg));
      color: var(--text);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      overflow-x: hidden;
    }

    .hyphae {
      position: fixed;
      inset: 0;
      pointer-events: none;
      background-image:
        radial-gradient(circle at 20% 30%, rgba(124,207,154,0.08) 0 2px, transparent 3px),
        radial-gradient(circle at 70% 60%, rgba(124,207,154,0.06) 0 2px, transparent 3px);
      animation: drift 20s linear infinite;
    }

    @keyframes drift {
      from { transform: translateY(0); }
      to { transform: translateY(-40px); }
    }

    h1 {
      font-size: clamp(2.2rem, 5vw, 3.2rem);
      margin-bottom: 0.2em;
      letter-spacing: 0.04em;
    }

    .tagline {
      color: var(--muted);
      margin-bottom: 2rem;
    }

    .search-box {
      background: var(--panel);
      border-radius: 18px;
      padding: 1rem;
      width: min(90%, 520px);
      box-shadow: 0 10px 30px rgba(0,0,0,0.4);
    }

    input {
      width: 100%;
      padding: 0.9rem 1rem;
      border-radius: 12px;
      border: 1px solid #284437;
      background: #0e1b14;
      color: var(--text);
      font-size: 1rem;
      outline: none;
    }

    input::placeholder { color: #6f9986; }

    button {
      margin-top: 0.8rem;
      width: 100%;
      padding: 0.8rem;
      border-radius: 12px;
      border: none;
      background: linear-gradient(135deg, var(--accent), var(--accent-soft));
      color: #0b1a13;
      font-weight: 600;
      cursor: pointer;
    }

    button:hover { filter: brightness(1.05); }

    .results {
      margin-top: 2rem;
      width: min(90%, 720px);
      display: none;
    }

    .node {
      background: rgba(22,38,29,0.9);
      border-left: 3px solid var(--accent);
      padding: 1rem;
      border-radius: 10px;
      margin-bottom: 1rem;
      position: relative;
    }

    .node::before {
      content: "";
      position: absolute;
      left: -14px;
      top: 50%;
      width: 12px;
      height: 2px;
      background: var(--accent);
    }

    .node h3 {
      margin: 0 0 0.3rem 0;
      font-size: 1.1rem;
    }

    .node p {
      margin: 0;
      color: var(--muted);
      font-size: 0.95rem;
    }

    footer {
      position: fixed;
      bottom: 10px;
      font-size: 0.8rem;
      color: #6f9986;
    }

    canvas {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      pointer-events: none;
    }

    #mycelio {
      position: fixed;
      right: 16px;
      bottom: 48px;
      background: rgba(22,38,29,0.95);
      border: 1px solid #284437;
      border-radius: 14px;
      padding: 10px 12px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.4);
      max-width: 220px;
      font-size: 0.9rem;
    }

    #mycelio-text { color: var(--muted); margin-top: 6px; }
  </style>
</head>
<body>
  <div class="hyphae"></div>

  <h1>Wood Wide Web</h1>
  <div class="tagline">Search the forest. Knowledge grows underground.</div>

  <div class="search-box">
    <input id="query" placeholder="Search spores, ideas, connections…" />
    <button onclick="sprout()">🌱 Sprout</button>
  </div>

  <canvas id="hyphaCanvas"></canvas>
  <div class="results" id="results"></div>

  <div id="mycelio">🍄 <strong>Mycelio</strong><div id="mycelio-text">Type a query and let the network grow.</div></div>
  <footer>© Wood Wide Web · Mycelium-powered</footer>

  <script>
    function sprout() {
      const q = document.getElementById('query').value.trim();
      const results = document.getElementById('results');
      const canvas = document.getElementById('hyphaCanvas');
      const ctx = canvas.getContext('2d');

      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      if (!q) return;

      results.style.display = 'block';
      results.innerHTML = '';

      fetch(`https://en.wikipedia.org/w/api.php?action=query&list=search&srsearch=${encodeURIComponent(q)}&utf8=&format=json&origin=*`)
        .then(res => res.json())
        .then(data => {
          if (!data.query.search.length) {
            results.innerHTML = '<div class="node"><h3>No spores found</h3><p>The mycelium could not locate related knowledge.</p></div>';
            return;
          }

          const nodes = [];

          document.getElementById('mycelio-text').textContent = `I found ${data.query.search.length} nutrient-rich connections. Showing the strongest mycelial links.`;

          data.query.search.slice(0, 5).forEach(item => {
            const node = document.createElement('div');
            node.className = 'node';
            node.innerHTML = `
              <h3><a href="https://en.wikipedia.org/wiki/${encodeURIComponent(item.title.replace(/ /g, '_'))}" target="_blank" style="color: var(--accent); text-decoration: none;">${item.title}</a></h3>
              <p>${item.snippet.replace(/<[^>]+>/g, '')}...</p>
            `;
            results.appendChild(node);
            nodes.push(node);
          });

          requestAnimationFrame(() => drawHyphae(nodes, canvas, ctx));
        });
    }

    function drawHyphae(nodes, canvas, ctx) {
      ctx.strokeStyle = 'rgba(124,207,154,0.45)';
      ctx.lineWidth = 1.2;

      const centerX = canvas.width / 2;
      const centerY = canvas.height / 2;

      const points = nodes.map(node => {
        const r = node.getBoundingClientRect();
        return { x: r.left + r.width/2, y: r.top + r.height/2 };
      });

      let t = 0;
      function animate() {
        t += 0.02;
        ctx.clearRect(0,0,canvas.width,canvas.height);

        // center to nodes (growth animation)
        points.forEach(p => {
          ctx.beginPath();
          ctx.moveTo(centerX, centerY);
          const cx = centerX + (p.x-centerX)/2;
          const cy = centerY;
          const ix = centerX + (p.x-centerX)*Math.min(t,1);
          const iy = centerY + (p.y-centerY)*Math.min(t,1);
          ctx.bezierCurveTo(cx, cy, cx, iy, ix, iy);
          ctx.stroke();
        });

        // node-to-node connections
        for (let i=0;i<points.length-1;i++){
          ctx.beginPath();
          ctx.moveTo(points[i].x, points[i].y);
          ctx.lineTo(points[i+1].x, points[i+1].y);
          ctx.stroke();
        }

        if (t < 1.05) requestAnimationFrame(animate);
      }
      animate();
    });
    }

    document.getElementById('query').addEventListener('keydown', e => {
      if (e.key === 'Enter') sprout();
    });
  </script>
</body>
</html>
