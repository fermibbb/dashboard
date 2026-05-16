<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="description" content="Sample multi-site dashboard for UK grower groups — yield, LEAF compliance, and spray records across member farms.">
<title>Kent Valley Growers · Member-farm dashboard</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Instrument+Serif:ital@0;1&family=Geist:wght@400;500;600&family=Geist+Mono:wght@400;500&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@tabler/icons-webfont@3.5.0/dist/tabler-icons.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
<style>
  :root {
    --bg: #FAF7F0;
    --surface: #FFFFFF;
    --ink: #1A1814;
    --ink-2: #5C564B;
    --ink-3: #8B8474;
    --line: #E8E2D3;
    --line-2: #D4CCB8;
    --green: #2D5F3F;
    --green-bg: #E8F0EA;
    --amber: #9C6B1F;
    --amber-bg: #F5EBD6;
    --terra: #A04B3A;
    --terra-bg: #F3E0DA;
    --chart-bar: #4A6B5C;
    --chart-bar-2: #7A8C7E;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  html, body { background: var(--bg); color: var(--ink); font-family: 'Geist', system-ui, sans-serif; font-size: 15px; line-height: 1.5; -webkit-font-smoothing: antialiased; }
  .num { font-family: 'Geist Mono', ui-monospace, monospace; font-feature-settings: 'tnum'; letter-spacing: -0.01em; }
  main { max-width: 1120px; margin: 0 auto; padding: 40px 32px 64px; }
  header.brand { display: flex; justify-content: space-between; align-items: flex-end; padding-bottom: 24px; border-bottom: 1px solid var(--line); margin-bottom: 28px; }
  header.brand .title { font-family: 'Instrument Serif', serif; font-size: 44px; line-height: 1; font-weight: 400; letter-spacing: -0.01em; }
  header.brand .title em { font-style: italic; color: var(--green); }
  header.brand .sub { color: var(--ink-2); font-size: 14px; margin-top: 8px; }
  header.brand .filters { display: flex; gap: 8px; }
  .demo-badge { display: inline-flex; align-items: center; gap: 6px; background: var(--amber-bg); color: var(--amber); font-size: 12px; padding: 4px 10px; border-radius: 999px; font-weight: 500; margin-left: 12px; vertical-align: 6px; }
  select { font-family: inherit; font-size: 13px; padding: 8px 14px; background: var(--surface); border: 1px solid var(--line-2); border-radius: 6px; color: var(--ink); cursor: pointer; }
  select:hover { border-color: var(--ink-3); }
  .metrics { display: grid; grid-template-columns: repeat(4, 1fr); gap: 16px; margin-bottom: 28px; }
  .metric { background: var(--surface); border: 1px solid var(--line); border-radius: 10px; padding: 20px 22px; }
  .metric .label { font-size: 12px; color: var(--ink-2); text-transform: uppercase; letter-spacing: 0.06em; margin-bottom: 10px; }
  .metric .value { font-family: 'Geist Mono', ui-monospace, monospace; font-size: 32px; font-weight: 500; line-height: 1; letter-spacing: -0.02em; }
  .metric .unit { font-size: 18px; color: var(--ink-2); font-weight: 400; margin-left: 2px; }
  .metric .delta { font-size: 12px; margin-top: 10px; display: flex; align-items: center; gap: 4px; }
  .delta.up { color: var(--green); }
  .delta.warn { color: var(--amber); }
  .delta.flat { color: var(--ink-3); }
  .charts { display: grid; grid-template-columns: 1.55fr 1fr; gap: 16px; margin-bottom: 28px; }
  .card { background: var(--surface); border: 1px solid var(--line); border-radius: 10px; padding: 22px 24px; }
  .card h3 { font-family: 'Instrument Serif', serif; font-size: 22px; font-weight: 400; margin-bottom: 4px; letter-spacing: -0.01em; }
  .card .card-sub { font-size: 13px; color: var(--ink-2); margin-bottom: 18px; }
  .chart-wrap { position: relative; height: 240px; }
  .donut-wrap { position: relative; height: 200px; }
  .legend { display: flex; flex-direction: column; gap: 10px; font-size: 13px; margin-top: 16px; }
  .legend-row { display: flex; justify-content: space-between; align-items: center; }
  .legend-label { display: flex; align-items: center; gap: 8px; color: var(--ink-2); }
  .legend-dot { width: 10px; height: 10px; border-radius: 2px; }
  .legend-val { font-family: 'Geist Mono', monospace; color: var(--ink); }
  .table-card { background: var(--surface); border: 1px solid var(--line); border-radius: 10px; padding: 22px 24px; margin-bottom: 28px; }
  .table-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 16px; }
  .table-head h3 { font-family: 'Instrument Serif', serif; font-size: 22px; font-weight: 400; letter-spacing: -0.01em; }
  button.action { font-family: inherit; font-size: 13px; padding: 8px 14px; background: var(--ink); color: var(--bg); border: none; border-radius: 6px; cursor: pointer; display: inline-flex; align-items: center; gap: 6px; }
  button.action:hover { background: var(--green); }
  button.ghost { background: transparent; color: var(--ink); border: 1px solid var(--line-2); }
  button.ghost:hover { background: var(--bg); border-color: var(--ink-3); }
  table { width: 100%; border-collapse: collapse; font-size: 14px; }
  thead th { text-align: left; font-weight: 500; color: var(--ink-2); font-size: 12px; text-transform: uppercase; letter-spacing: 0.06em; padding: 8px 12px 12px; border-bottom: 1px solid var(--line); }
  thead th.r { text-align: right; }
  tbody td { padding: 14px 12px; border-bottom: 1px solid var(--line); }
  tbody tr:last-child td { border-bottom: none; }
  tbody tr:hover { background: var(--bg); }
  td.r { text-align: right; font-family: 'Geist Mono', monospace; }
  td .crop { color: var(--ink-2); font-size: 13px; }
  .pill { display: inline-flex; align-items: center; gap: 4px; font-size: 12px; padding: 4px 10px; border-radius: 999px; font-weight: 500; }
  .pill.ok { background: var(--green-bg); color: var(--green); }
  .pill.warn { background: var(--amber-bg); color: var(--amber); }
  .pill.bad { background: var(--terra-bg); color: var(--terra); }
  .actions { display: flex; gap: 10px; flex-wrap: wrap; }
  footer { margin-top: 32px; padding-top: 24px; border-top: 1px solid var(--line); display: flex; justify-content: space-between; align-items: center; font-size: 13px; color: var(--ink-3); }
  footer em { font-family: 'Instrument Serif', serif; font-style: italic; color: var(--ink-2); }
  @media (max-width: 880px) {
    main { padding: 24px 18px; }
    header.brand { flex-direction: column; align-items: flex-start; gap: 12px; }
    header.brand .title { font-size: 32px; }
    .metrics { grid-template-columns: repeat(2, 1fr); }
    .charts { grid-template-columns: 1fr; }
  }
</style>
</head>
<body>
<main>
  <header class="brand">
    <div>
      <h1 class="title">Kent Valley <em>Growers</em><span class="demo-badge"><i class="ti ti-flask" aria-hidden="true"></i>Demo data</span></h1>
      <p class="sub">Member-farm overview · 2025 season · 6 sites · Last refresh 12 Aug, 06:00</p>
    </div>
    <div class="filters">
      <select><option>All crops</option><option>Apple</option><option>Cherry</option><option>Pear</option><option>Plum</option></select>
      <select><option>2025 season</option><option>2024 season</option><option>YoY compare</option></select>
    </div>
  </header>

  <section class="metrics">
    <div class="metric">
      <p class="label">Group yield</p>
      <p class="value num">4,287<span class="unit"> t</span></p>
      <p class="delta up"><i class="ti ti-trending-up" aria-hidden="true"></i> 8.4% vs 2024</p>
    </div>
    <div class="metric">
      <p class="label">Avg yield / hectare</p>
      <p class="value num">38.2<span class="unit"> t</span></p>
      <p class="delta up"><i class="ti ti-trending-up" aria-hidden="true"></i> 2.1 t vs 2024</p>
    </div>
    <div class="metric">
      <p class="label">LEAF audit-ready</p>
      <p class="value num">83<span class="unit">%</span></p>
      <p class="delta warn"><i class="ti ti-alert-triangle" aria-hidden="true"></i> 1 site at risk</p>
    </div>
    <div class="metric">
      <p class="label">Open actions</p>
      <p class="value num">7</p>
      <p class="delta flat">across 4 sites</p>
    </div>
  </section>

  <section class="charts">
    <div class="card">
      <h3>Yield by site</h3>
      <p class="card-sub">2025 season totals, sorted high to low — tonnes harvested</p>
      <div class="chart-wrap"><canvas id="yieldChart" role="img" aria-label="Yield bar chart by site"></canvas></div>
    </div>
    <div class="card">
      <h3>LEAF readiness</h3>
      <p class="card-sub">Status across the six member sites</p>
      <div class="donut-wrap"><canvas id="leafChart" role="img" aria-label="Doughnut of LEAF audit readiness"></canvas></div>
      <div class="legend">
        <div class="legend-row"><span class="legend-label"><span class="legend-dot" style="background:#2D5F3F"></span>Audit ready</span><span class="legend-val">4</span></div>
        <div class="legend-row"><span class="legend-label"><span class="legend-dot" style="background:#9C6B1F"></span>Action needed</span><span class="legend-val">1</span></div>
        <div class="legend-row"><span class="legend-label"><span class="legend-dot" style="background:#A04B3A"></span>At risk</span><span class="legend-val">1</span></div>
      </div>
    </div>
  </section>

  <section class="table-card">
    <div class="table-head">
      <h3>Member sites</h3>
      <button class="action"><i class="ti ti-file-export" aria-hidden="true"></i> Export audit pack</button>
    </div>
    <table>
      <thead>
        <tr>
          <th>Site</th>
          <th>Crops</th>
          <th class="r">Yield (t)</th>
          <th class="r">t / ha</th>
          <th>Last spray</th>
          <th>LEAF</th>
        </tr>
      </thead>
      <tbody>
        <tr><td>Bruce-Lockhart</td><td class="crop">Apple, Pear</td><td class="r">982</td><td class="r">41.3</td><td class="crop">3 Aug</td><td><span class="pill ok"><i class="ti ti-check" aria-hidden="true"></i>Ready</span></td></tr>
        <tr><td>GH Chambers</td><td class="crop">Apple, Plum</td><td class="r">814</td><td class="r">39.8</td><td class="crop">5 Aug</td><td><span class="pill ok"><i class="ti ti-check" aria-hidden="true"></i>Ready</span></td></tr>
        <tr><td>Hares Fruit</td><td class="crop">Cherry, Apple</td><td class="r">760</td><td class="r">42.1</td><td class="crop">2 Aug</td><td><span class="pill warn"><i class="ti ti-alert-triangle" aria-hidden="true"></i>Action × 2</span></td></tr>
        <tr><td>Nicholls</td><td class="crop">Apple</td><td class="r">650</td><td class="r">36.1</td><td class="crop">7 Aug</td><td><span class="pill ok"><i class="ti ti-check" aria-hidden="true"></i>Ready</span></td></tr>
        <tr><td>Ware Farms</td><td class="crop">Pear, Apple</td><td class="r">581</td><td class="r">35.2</td><td class="crop">4 Aug</td><td><span class="pill ok"><i class="ti ti-check" aria-hidden="true"></i>Ready</span></td></tr>
        <tr><td>Skinner &amp; Son</td><td class="crop">Apple, Plum</td><td class="r">500</td><td class="r">31.4</td><td class="crop">1 Aug</td><td><span class="pill bad"><i class="ti ti-alert-octagon" aria-hidden="true"></i>At risk</span></td></tr>
      </tbody>
    </table>
  </section>

  <div class="actions">
    <button class="ghost"><i class="ti ti-zoom-in" aria-hidden="true"></i> Drill into Hares Fruit</button>
    <button class="ghost"><i class="ti ti-versions" aria-hidden="true"></i> Compare cherry yield YoY</button>
    <button class="ghost"><i class="ti ti-cloud-rain" aria-hidden="true"></i> Spray windows next 7 days</button>
  </div>

  <footer>
    <span>Demo dashboard built for grower-group illustration. Numbers are <em>illustrative</em>, not from real farms.</span>
    <span>Built by <em>[Your name]</em></span>
  </footer>
</main>

<script>
  const ink2 = '#5C564B';
  const line = '#E8E2D3';
  Chart.defaults.font.family = "'Geist', system-ui, sans-serif";
  Chart.defaults.color = ink2;

  new Chart(document.getElementById('yieldChart'), {
    type: 'bar',
    data: {
      labels: ['Bruce-Lockhart', 'GH Chambers', 'Hares Fruit', 'Nicholls', 'Ware Farms', 'Skinner'],
      datasets: [{
        data: [982, 814, 760, 650, 581, 500],
        backgroundColor: '#4A6B5C',
        borderRadius: 4,
        barPercentage: 0.65
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: { legend: { display: false }, tooltip: { backgroundColor: '#1A1814', padding: 10, titleFont: { size: 12, weight: 500 }, bodyFont: { family: "'Geist Mono', monospace" } } },
      scales: {
        x: { ticks: { font: { size: 12 } }, grid: { display: false }, border: { color: line } },
        y: { beginAtZero: true, ticks: { font: { family: "'Geist Mono', monospace", size: 11 } }, grid: { color: line, drawTicks: false }, border: { display: false } }
      }
    }
  });

  new Chart(document.getElementById('leafChart'), {
    type: 'doughnut',
    data: {
      labels: ['Ready', 'Action', 'At risk'],
      datasets: [{ data: [4, 1, 1], backgroundColor: ['#2D5F3F', '#9C6B1F', '#A04B3A'], borderWidth: 0 }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      cutout: '68%',
      plugins: { legend: { display: false }, tooltip: { backgroundColor: '#1A1814', padding: 10 } }
    }
  });
</script>
</body>
</html>
