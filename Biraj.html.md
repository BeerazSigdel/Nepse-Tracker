<!DOCTYPE html>  
<html lang="en">  
<head>  
<meta charset="UTF-8">  
<meta name="viewport" content="width=device-width, initial-scale=1.0">  
<title>NEPSE Live Tracker</title>  
<link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=DM+Sans:wght@300;400;500;600;700&display=swap" rel="stylesheet">  
<style>  
  :root {  
    --navy:    #0D1B2A;  
    --navy2:   #1B2E45;  
    --navy3:   #243B55;  
    --gold:    #C9A84C;  
    --gold2:   #F0D080;  
    --green:   #27AE60;  
    --green2:  #1A6B3C;  
    --green-bg:#0D2B1A;  
    --red:     #E74C3C;  
    --red-bg:  #2B0D0D;  
    --orange:  #E67E22;  
    --orange-bg:#2B1A0D;  
    --grey:    #8899AA;  
    --light:   #D0DCE8;  
    --white:   #F0F4F8;  
    --mono:    'Space Mono', monospace;  
    --sans:    'DM Sans', sans-serif;  
  }  
  * { margin:0; padding:0; box-sizing:border-box; }  
  body {  
    background:var(--navy); color:var(--white);  
    font-family:var(--sans); min-height:100vh; overflow-x:hidden;  
  }  
  body::before {  
    content:''; position:fixed; inset:0;  
    background-image:  
      linear-gradient(rgba(201,168,76,0.04) 1px, transparent 1px),  
      linear-gradient(90deg, rgba(201,168,76,0.04) 1px, transparent 1px);  
    background-size:40px 40px; pointer-events:none; z-index:0;  
  }  
  .app { position:relative; z-index:1; max-width:1200px; margin:0 auto; padding:12px; }  
  
  /* HEADER */  
  .header {  
    display:flex; align-items:center; justify-content:space-between; flex-wrap:wrap; gap:10px;  
    padding:16px 20px;  
    background:linear-gradient(135deg,var(--navy2),var(--navy3));  
    border:1px solid rgba(201,168,76,0.3); border-radius:16px; margin-bottom:12px;  
    box-shadow:0 8px 32px rgba(0,0,0,0.4);  
  }  
  .header-left h1 { font-family:var(--mono); font-size:1.2rem; color:var(--gold); letter-spacing:2px; }  
  .header-left p  { color:var(--grey); font-size:0.75rem; margin-top:3px; }  
  .header-right   { display:flex; align-items:center; gap:10px; flex-wrap:wrap; }  
  
  .api-status {  
    display:flex; align-items:center; gap:6px; padding:6px 12px;  
    border-radius:20px; font-family:var(--mono); font-size:0.7rem;  
    border:1px solid; transition:all 0.3s;  
  }  
  .api-status.connected   { background:rgba(39,174,96,0.15); color:var(--green); border-color:var(--green); }  
  .api-status.connecting  { background:rgba(230,126,34,0.15); color:var(--orange); border-color:var(--orange); }  
  .api-status.offline     { background:rgba(231,76,60,0.15); color:var(--red); border-color:var(--red); }  
  .api-dot { width:7px; height:7px; border-radius:50%; background:currentColor; }  
  .api-dot.pulse { animation:pulse 1.5s infinite; }  
  @keyframes pulse { 0%,100%{opacity:1;transform:scale(1)} 50%{opacity:0.3;transform:scale(0.7)} }  
  
  .btn {  
    background:rgba(201,168,76,0.15); border:1px solid var(--gold);  
    color:var(--gold); border-radius:10px; padding:7px 14px;  
    font-family:var(--mono); font-size:0.72rem; cursor:pointer; transition:all 0.2s;  
  }  
  .btn:hover { background:rgba(201,168,76,0.3); }  
  .btn.spinning { animation:spin 1s linear infinite; }  
  @keyframes spin { to{transform:rotate(360deg)} }  
  .btn.danger { border-color:var(--red); color:var(--red); background:rgba(231,76,60,0.1); }  
  
  /* API INFO BANNER */  
  .api-banner {  
    padding:10px 16px; border-radius:10px; margin-bottom:12px;  
    font-size:0.78rem; display:flex; align-items:flex-start; gap:10px;  
    border:1px solid;  
  }  
  .api-banner.info    { background:rgba(36,107,163,0.15); border-color:rgba(36,107,163,0.4); color:var(--light); }  
  .api-banner.success { background:rgba(39,174,96,0.1); border-color:rgba(39,174,96,0.4); color:var(--green); }  
  .api-banner.warning { background:rgba(230,126,34,0.1); border-color:rgba(230,126,34,0.4); color:var(--orange); }  
  .api-banner.error   { background:rgba(231,76,60,0.1); border-color:rgba(231,76,60,0.4); color:var(--red); }  
  
  /* MARKET BAR */  
  .market-bar {  
    display:flex; align-items:center; justify-content:space-between; flex-wrap:wrap; gap:6px;  
    padding:9px 16px; background:rgba(27,46,69,0.8);  
    border:1px solid rgba(201,168,76,0.15); border-radius:10px; margin-bottom:12px;  
    font-family:var(--mono); font-size:0.72rem;  
  }  
  .mkt-open   { color:var(--green); }  
  .mkt-closed { color:var(--red); }  
  
  /* INDEX BAR */  
  .index-bar { display:flex; gap:8px; margin-bottom:12px; overflow-x:auto; padding-bottom:4px; }  
  .idx-pill {  
    flex-shrink:0; background:var(--navy2); border:1px solid rgba(255,255,255,0.08);  
    border-radius:10px; padding:10px 14px; min-width:150px;  
  }  
  .idx-label { font-size:0.65rem; color:var(--grey); text-transform:uppercase; letter-spacing:1px; }  
  .idx-value { font-family:var(--mono); font-size:0.95rem; font-weight:700; margin:4px 0; }  
  .idx-chg   { font-family:var(--mono); font-size:0.68rem; }  
  .green { color:var(--green); } .red { color:var(--red); } .gold { color:var(--gold); } .grey { color:var(--grey); }  
  
  /* SUMMARY */  
  .summary-grid { display:grid; grid-template-columns:repeat(auto-fit,minmax(165px,1fr)); gap:10px; margin-bottom:12px; }  
  .card {  
    background:linear-gradient(135deg,var(--navy2),var(--navy3));  
    border:1px solid rgba(255,255,255,0.08); border-radius:14px; padding:15px 18px;  
    transition:transform 0.2s,border-color 0.2s; position:relative; overflow:hidden;  
  }  
  .card:hover { transform:translateY(-2px); border-color:rgba(201,168,76,0.3); }  
  .card-label { font-size:0.68rem; color:var(--grey); text-transform:uppercase; letter-spacing:1.2px; margin-bottom:6px; }  
  .card-value { font-family:var(--mono); font-size:1.2rem; font-weight:700; }  
  .card-sub   { font-size:0.7rem; margin-top:3px; color:var(--grey); }  
  
  /* TABS */  
  .tabs {  
    display:flex; gap:4px; margin-bottom:12px;  
    background:rgba(13,27,42,0.6); padding:4px; border-radius:12px;  
    border:1px solid rgba(255,255,255,0.06); overflow-x:auto;  
  }  
  .tab {  
    flex-shrink:0; flex:1; min-width:80px; padding:9px 12px; text-align:center;  
    cursor:pointer; border-radius:9px; font-size:0.78rem; font-weight:600;  
    transition:all 0.2s; color:var(--grey); white-space:nowrap;  
  }  
  .tab.active { background:var(--navy2); color:var(--gold); border:1px solid rgba(201,168,76,0.3); }  
  .tab:hover:not(.active) { color:var(--white); }  
  
  .panel { display:none; }  
  .panel.active { display:block; }  
  
  /* TABLE */  
  .table-wrap {  
    background:var(--navy2); border:1px solid rgba(255,255,255,0.08);  
    border-radius:14px; overflow:hidden; margin-bottom:12px;  
  }  
  .table-header {  
    display:flex; align-items:center; justify-content:space-between; flex-wrap:wrap; gap:8px;  
    padding:12px 18px; border-bottom:1px solid rgba(255,255,255,0.06);  
  }  
  .table-title { font-weight:700; font-size:0.9rem; color:var(--gold); font-family:var(--mono); }  
  .badge {  
    font-size:0.68rem; padding:3px 10px; border-radius:20px;  
    background:rgba(201,168,76,0.1); color:var(--gold); border:1px solid rgba(201,168,76,0.3);  
  }  
  table { width:100%; border-collapse:collapse; }  
  thead tr { background:rgba(13,27,42,0.8); }  
  thead th {  
    padding:9px 12px; font-size:0.65rem; color:var(--grey);  
    text-transform:uppercase; letter-spacing:1px; text-align:right;  
    font-family:var(--mono); border-bottom:1px solid rgba(255,255,255,0.06);  
  }  
  thead th:first-child { text-align:left; }  
  tbody tr { border-bottom:1px solid rgba(255,255,255,0.04); transition:background 0.15s; }  
  tbody tr:hover { background:rgba(201,168,76,0.04); }  
  tbody tr.section-row td {  
    background:rgba(201,168,76,0.06); color:var(--gold);  
    font-size:0.68rem; font-weight:700; letter-spacing:1.5px;  
    padding:7px 12px; text-transform:uppercase; font-family:var(--mono);  
  }  
  td { padding:11px 12px; font-size:0.78rem; text-align:right; font-family:var(--mono); }  
  td:first-child { text-align:left; }  
  td.sname { font-weight:700; color:var(--white); }  
  td.ssector { color:var(--grey); font-size:0.68rem; font-family:var(--sans); }  
  
  .status-badge {  
    display:inline-block; padding:2px 9px; border-radius:20px;  
    font-size:0.62rem; font-weight:700; font-family:var(--sans);  
  }  
  .s-hold   { background:rgba(39,174,96,0.15); color:var(--green); border:1px solid rgba(39,174,96,0.3); }  
  .s-watch  { background:rgba(230,126,34,0.15); color:var(--orange); border:1px solid rgba(230,126,34,0.3); }  
  .s-free   { background:rgba(201,168,76,0.15); color:var(--gold); border:1px solid rgba(201,168,76,0.3); }  
  .s-review { background:rgba(231,76,60,0.15); color:var(--red); border:1px solid rgba(231,76,60,0.3); }  
  .s-alert  { background:rgba(231,76,60,0.35); color:#fff; border:1px solid var(--red); animation:aPulse 1s infinite; }  
  .s-target { background:rgba(39,174,96,0.35); color:#fff; border:1px solid var(--green); animation:aPulse 1s infinite; }  
  @keyframes aPulse { 0%,100%{opacity:1} 50%{opacity:0.6} }  
  
  tr.row-stop   td { background:rgba(231,76,60,0.08); }  
  tr.row-near   td { background:rgba(230,126,34,0.05); }  
  tr.row-target td { background:rgba(39,174,96,0.06); }  
  
  /* LOADING SKELETON */  
  .skeleton {  
    background:linear-gradient(90deg,rgba(255,255,255,0.04) 25%,rgba(255,255,255,0.08) 50%,rgba(255,255,255,0.04) 75%);  
    background-size:200% 100%; animation:shimmer 1.5s infinite; border-radius:4px; height:14px;  
  }  
  @keyframes shimmer { 0%{background-position:200%} 100%{background-position:-200%} }  
  
  /* ALERTS */  
  .alerts-list { display:flex; flex-direction:column; gap:10px; }  
  .alert-card {  
    display:flex; align-items:flex-start; gap:12px;  
    padding:14px 16px; border-radius:12px; border:1px solid;  
  }  
  .alert-card.danger  { background:var(--red-bg); border-color:var(--red); }  
  .alert-card.warning { background:var(--orange-bg); border-color:var(--orange); }  
  .alert-card.success { background:var(--green-bg); border-color:var(--green); }  
  .alert-card.info    { background:rgba(27,46,69,0.8); border-color:rgba(201,168,76,0.3); }  
  .a-icon  { font-size:1.3rem; flex-shrink:0; margin-top:2px; }  
  .a-body  { flex:1; }  
  .a-title { font-weight:700; font-size:0.82rem; margin-bottom:3px; }  
  .a-desc  { font-size:0.73rem; color:var(--grey); }  
  .a-price { font-family:var(--mono); font-size:0.88rem; font-weight:700; text-align:right; flex-shrink:0; }  
  
  /* MARKET GRID */  
  .mkt-grid { display:grid; grid-template-columns:repeat(auto-fill,minmax(155px,1fr)); gap:8px; padding:12px; }  
  .mkt-card {  
    background:rgba(13,27,42,0.6); border:1px solid rgba(255,255,255,0.06);  
    border-radius:10px; padding:12px 14px; transition:all 0.2s; cursor:default;  
  }  
  .mkt-card:hover { border-color:rgba(201,168,76,0.3); transform:translateY(-1px); }  
  .mkt-card.owned { border-color:rgba(201,168,76,0.25); }  
  .mc-sym   { font-family:var(--mono); font-weight:700; font-size:0.82rem; color:var(--white); }  
  .mc-price { font-family:var(--mono); font-size:1rem; font-weight:700; margin:5px 0; }  
  .mc-chg   { font-family:var(--mono); font-size:0.7rem; }  
  .mc-sec   { font-size:0.65rem; color:var(--grey); margin-top:3px; }  
  .mc-bar   { height:3px; border-radius:2px; margin-top:7px; background:rgba(255,255,255,0.08); overflow:hidden; }  
  .mc-fill  { height:100%; border-radius:2px; transition:width 0.8s; }  
  
  /* LIVE INDICATOR */  
  .live-badge {  
    display:flex; align-items:center; gap:5px;  
    background:rgba(39,174,96,0.12); border:1px solid rgba(39,174,96,0.4);  
    border-radius:20px; padding:5px 12px; font-size:0.7rem; color:var(--green); font-family:var(--mono);  
  }  
  .live-dot { width:7px; height:7px; border-radius:50%; background:var(--green); animation:pulse 1.5s infinite; }  
  
  /* TOAST */  
  #toasts { position:fixed; top:16px; right:16px; z-index:9999; display:flex; flex-direction:column; gap:8px; pointer-events:none; }  
  .toast {  
    background:var(--navy2); border:1px solid; border-radius:12px;  
    padding:12px 16px; min-width:260px; max-width:320px;  
    box-shadow:0 8px 32px rgba(0,0,0,0.5); pointer-events:auto;  
    animation:slideIn 0.3s ease;  
  }  
  @keyframes slideIn { from{transform:translateX(120%);opacity:0} to{transform:translateX(0);opacity:1} }  
  .toast.danger  { border-color:var(--red); }  
  .toast.success { border-color:var(--green); }  
  .toast.warning { border-color:var(--orange); }  
  .toast.info    { border-color:rgba(201,168,76,0.4); }  
  .t-title { font-weight:700; font-size:0.82rem; margin-bottom:3px; }  
  .t-msg   { font-size:0.74rem; color:var(--grey); }  
  .toast.danger .t-title  { color:var(--red); }  
  .toast.success .t-title { color:var(--green); }  
  .toast.warning .t-title { color:var(--orange); }  
  .toast.info .t-title    { color:var(--gold); }  
  
  /* SETTINGS */  
  .settings-grid { display:grid; grid-template-columns:repeat(auto-fit,minmax(280px,1fr)); gap:10px; }  
  .s-card { background:var(--navy2); border:1px solid rgba(255,255,255,0.08); border-radius:14px; padding:16px; }  
  .s-card h3 { color:var(--gold); font-size:0.82rem; margin-bottom:12px; font-family:var(--mono); letter-spacing:1px; }  
  .s-row {  
    display:flex; align-items:center; justify-content:space-between;  
    padding:9px 0; border-bottom:1px solid rgba(255,255,255,0.05);  
  }  
  .s-row:last-child { border-bottom:none; }  
  .s-label { font-size:0.78rem; color:var(--light); }  
  .s-val   { font-family:var(--mono); font-size:0.78rem; color:var(--gold); }  
  input[type=number] {  
    background:rgba(13,27,42,0.9); border:1px solid rgba(201,168,76,0.3);  
    color:var(--gold); border-radius:6px; padding:4px 8px;  
    font-family:var(--mono); font-size:0.78rem; width:85px; text-align:right;  
  }  
  input[type=number]:focus { outline:none; border-color:var(--gold); }  
  
  /* TICKER */  
  .ticker-wrap { overflow:hidden; background:rgba(13,27,42,0.9); border:1px solid rgba(201,168,76,0.12); border-radius:8px; padding:7px 0; margin-bottom:12px; }  
  .ticker-track { display:flex; white-space:nowrap; animation:ticker 50s linear infinite; }  
  .ticker-track:hover { animation-play-state:paused; }  
  @keyframes ticker { 0%{transform:translateX(0)} 100%{transform:translateX(-50%)} }  
  .t-item {  
    display:inline-flex; align-items:center; gap:7px;  
    padding:0 20px; font-family:var(--mono); font-size:0.72rem;  
    border-right:1px solid rgba(255,255,255,0.07);  
  }  
  .t-sym { color:var(--light); font-weight:700; }  
  .t-prc { color:var(--white); }  
  
  /* API SOURCE TOGGLE */  
  .source-toggle { display:flex; gap:6px; align-items:center; font-size:0.72rem; color:var(--grey); font-family:var(--mono); }  
  .src-btn { padding:4px 10px; border-radius:6px; border:1px solid rgba(255,255,255,0.15); background:transparent; color:var(--grey); cursor:pointer; font-family:var(--mono); font-size:0.68rem; transition:all 0.2s; }  
  .src-btn.active { border-color:var(--gold); color:var(--gold); background:rgba(201,168,76,0.1); }  
  
  /* FOOTER */  
  .footer { text-align:center; padding:16px; color:var(--grey); font-size:0.68rem; border-top:1px solid rgba(255,255,255,0.05); margin-top:16px; line-height:1.8; }  
  
  @media(max-width:600px) {  
    .header { flex-direction:column; gap:10px; text-align:center; }  
    .header-right { justify-content:center; }  
    .summary-grid { grid-template-columns:1fr 1fr; }  
    td { padding:8px 7px; font-size:0.72rem; }  
    .header-left h1 { font-size:1rem; }  
  }  
</style>  
</head>  
<body>  
<div id="toasts"></div>  
<div class="app">  
  
  <!-- HEADER -->  
  <div class="header">  
    <div class="header-left">  
      <h1>🇳🇵 NEPSE LIVE TRACKER</h1>  
      <p>Real-time data · Your personal portfolio · May 2026</p>  
    </div>  
    <div class="header-right">  
      <div class="live-badge"><div class="live-dot"></div>LIVE API</div>  
      <div class="api-status connecting" id="api-status">  
        <div class="api-dot pulse"></div>  
        <span id="api-status-text">Connecting...</span>  
      </div>  
      <button class="btn" id="refresh-btn" onclick="fetchAllPrices()">⟳ REFRESH</button>  
    </div>  
  </div>  
  
  <!-- API BANNER -->  
  <div class="api-banner info" id="api-banner">  
    <span>ℹ️</span>  
    <span id="api-banner-text">Connecting to free NEPSE API (ShareBazaar)... Data sourced from official NEPSE public channels. For personal/educational use only.</span>  
  </div>  
  
  <!-- MARKET BAR -->  
  <div class="market-bar">  
    <span id="mkt-status" class="mkt-closed">● MARKET CLOSED</span>  
    <span id="nst-time" style="color:var(--grey)">Loading NST...</span>  
    <div class="source-toggle">  
      API:  
      <button class="src-btn active" id="src1" onclick="setSource('sharebazaar')">ShareBazaar</button>  
      <button class="src-btn" id="src2" onclick="setSource('nepseapi')">NepseAPI</button>  
      <button class="src-btn" id="src3" onclick="setSource('manual')">Manual</button>  
    </div>  
    <span id="last-upd" style="color:var(--gold);font-family:var(--mono);font-size:0.7rem">Not updated yet</span>  
  </div>  
  
  <!-- INDEX PILLS -->  
  <div class="index-bar" id="index-bar">  
    <div class="idx-pill"><div class="idx-label">NEPSE Index</div><div class="idx-value gold skeleton" id="idx-nepse" style="width:100px;height:22px"></div><div class="idx-chg" id="idx-nepse-chg"></div></div>  
    <div class="idx-pill"><div class="idx-label">Total Turnover</div><div class="idx-value gold" id="idx-turnover">—</div><div class="idx-chg grey">Today</div></div>  
    <div class="idx-pill"><div class="idx-label">Total Volume</div><div class="idx-value" id="idx-volume">—</div><div class="idx-chg grey">Shares traded</div></div>  
    <div class="idx-pill"><div class="idx-label">Traded Scripts</div><div class="idx-value" id="idx-scripts">—</div><div class="idx-chg grey">Companies active</div></div>  
    <div class="idx-pill"><div class="idx-label">Market Status</div><div class="idx-value" id="idx-mkt-stat">—</div><div class="idx-chg grey">NST hours</div></div>  
  </div>  
  
  <!-- TICKER -->  
  <div class="ticker-wrap"><div class="ticker-track" id="ticker"></div></div>  
  
  <!-- SUMMARY CARDS -->  
  <div class="summary-grid">  
    <div class="card"><div class="card-label">Total Invested</div><div class="card-value" id="c-invested" style="color:var(--white)">Calculating...</div><div class="card-sub">Core positions</div></div>  
    <div class="card"><div class="card-label">Market Value</div><div class="card-value gold" id="c-value">—</div><div class="card-sub">Live portfolio worth</div></div>  
    <div class="card"><div class="card-label">Unrealised P&L</div><div class="card-value" id="c-pnl">—</div><div class="card-sub" id="c-pnl-pct">—</div></div>  
    <div class="card"><div class="card-label">Locked Profit</div><div class="card-value green">NPR 1,17,770</div><div class="card-sub">RSML+NRN sold ✅</div></div>  
    <div class="card"><div class="card-label">Cash Available</div><div class="card-value gold">NPR 1,12,720</div><div class="card-sub">Ready to deploy</div></div>  
    <div class="card"><div class="card-label">Active Alerts</div><div class="card-value" id="c-alerts" style="color:var(--orange)">—</div><div class="card-sub">Stop loss / Target</div></div>  
  </div>  
  
  <!-- TABS -->  
  <div class="tabs">  
    <div class="tab active" onclick="showTab('portfolio',this)">📊 Portfolio</div>  
    <div class="tab" onclick="showTab('alerts',this)">🔔 Alerts</div>  
    <div class="tab" onclick="showTab('market',this)">📈 Market</div>  
    <div class="tab" onclick="showTab('settings',this)">⚙️ Settings</div>  
  </div>  
  
  <!-- PORTFOLIO -->  
  <div class="panel active" id="panel-portfolio">  
    <div class="table-wrap">  
      <div class="table-header">  
        <span class="table-title">YOUR HOLDINGS</span>  
        <span class="badge" id="portfolio-badge">Loading...</span>  
      </div>  
      <div style="overflow-x:auto">  
        <table><thead><tr>  
          <th style="text-align:left">STOCK</th>  
          <th>SECTOR</th><th>QTY</th><th>COST</th>  
          <th>LTP</th><th>CHG%</th><th>VALUE</th>  
          <th>P&L</th><th>P&L%</th><th>STOP</th><th>TARGET</th><th>STATUS</th>  
        </tr></thead>  
        <tbody id="ptable"></tbody></table>  
      </div>  
    </div>  
  </div>  
  
  <!-- ALERTS -->  
  <div class="panel" id="panel-alerts">  
    <div class="alerts-list" id="alerts-list">  
      <div class="alert-card info"><div class="a-icon">⏳</div><div class="a-body"><div class="a-title">Loading alerts...</div><div class="a-desc">Fetching live prices to generate alerts.</div></div></div>  
    </div>  
  </div>  
  
  <!-- MARKET -->  
  <div class="panel" id="panel-market">  
    <div class="table-wrap">  
      <div class="table-header">  
        <span class="table-title">MARKET WATCHLIST</span>  
        <span class="badge">⭐ = In Your Portfolio</span>  
      </div>  
      <div class="mkt-grid" id="mkt-grid"></div>  
    </div>  
  </div>  
  
  <!-- SETTINGS -->  
  <div class="panel" id="panel-settings">  
    <div class="settings-grid" id="settings-grid"></div>  
    <div style="display:flex;gap:10px;justify-content:center;margin-top:14px;flex-wrap:wrap">  
      <button class="btn" onclick="saveManualPrices()" style="padding:10px 28px">💾 SAVE MANUAL PRICES</button>  
      <button class="btn danger" onclick="clearCache()" style="padding:10px 20px">🗑 CLEAR CACHE</button>  
    </div>  
  </div>  
  
  <div class="footer">  
    🇳🇵 NEPSE Live Tracker · API: ShareBazaar (nepsetty.kokomo.workers.dev) · NepseAPI (nepseapi.surajrimal.dev)<br>  
    Data sourced from unofficial public channels · For personal/educational use only · Not financial advice<br>  
    Always verify prices on merolagani.com before trading  
  </div>  
</div>  
  
<script>  
// ════════════════════════════════════════════════════════════  
// CONFIGURATION  
// ════════════════════════════════════════════════════════════  
const API_SOURCES = {  
  sharebazaar: {  
    name: "ShareBazaar",  
    url: sym => `https://nepsetty.kokomo.workers.dev/api?symbol=${sym}`,  
    parse: data => ({  
      price:  parseFloat(data.lastTradedPrice || data.ltp || data.price || 0),  
      change: parseFloat(data.percentageChange || data.change || data.pChange || 0),  
      volume: parseInt(data.totalTradeQuantity || data.volume || 0),  
      high:   parseFloat(data.highPrice || data.high || 0),  
      low:    parseFloat(data.lowPrice || data.low || 0),  
    })  
  },  
  nepseapi: {  
    name: "NepseAPI",  
    url: sym => `https://nepseapi.surajrimal.dev/stock/${sym}`,  
    parse: data => ({  
      price:  parseFloat(data.lastTradedPrice || data.ltp || 0),  
      change: parseFloat(data.percentageChange || data.pChange || 0),  
      volume: parseInt(data.totalTradeQuantity || data.volume || 0),  
      high:   parseFloat(data.highPrice || data.high || 0),  
      low:    parseFloat(data.lowPrice || data.low || 0),  
    })  
  }  
};  
  
let currentSource = 'sharebazaar';  
let priceCache = {};  
let lastFetch = null;  
let alertsShown = new Set();  
let autoRefreshTimer = null;  
  
// ════════════════════════════════════════════════════════════  
// YOUR PORTFOLIO DATA  
// ════════════════════════════════════════════════════════════  
const PORTFOLIO = [  
  { section: "CORE HOLDINGS" },  
  { sym:"NABIL",   name:"Nabil Bank",            sector:"Commercial Bank",     qty:null, cost:528,  stop:468,  t1:608,  t2:686,  status:"hold",  note:"Confirm qty" },  
  { sym:"KBL",     name:"Kumari Bank",            sector:"Commercial Bank",     qty:70,   cost:214,  stop:185,  t1:250,  t2:285,  status:"hold"  },  
  { sym:"NMB",     name:"NMB Bank",               sector:"Commercial Bank",     qty:20,   cost:240,  stop:210,  t1:276,  t2:312,  status:"hold"  },  
  { sym:"NIFRA",   name:"Nepal Infrastructure",   sector:"Infrastructure Bank", qty:54,   cost:261,  stop:228,  t1:300,  t2:340,  status:"hold"  },  
  { section: "SECTOR HOLDINGS" },  
  { sym:"UMRH",    name:"Upper Marsyangdi",       sector:"Hydropower",          qty:11,   cost:532,  stop:465,  t1:612,  t2:692,  status:"hold"  },  
  { sym:"RHPL",    name:"Rasuwagadhi Hydro",      sector:"Hydropower",          qty:20,   cost:275,  stop:240,  t1:316,  t2:358,  status:"hold"  },  
  { sym:"SJLIC",   name:"SuryaJyoti Life Ins.",   sector:"Life Insurance",      qty:10,   cost:429,  stop:375,  t1:493,  t2:558,  status:"hold"  },  
  { sym:"SRLI",    name:"Reliable Life Ins.",     sector:"Life Insurance",      qty:11,   cost:390,  stop:340,  t1:448,  t2:507,  status:"hold"  },  
  { sym:"SJCL",    name:"Sanjen Jalavidhyut",     sector:"Hydropower",          qty:10,   cost:298,  stop:260,  t1:343,  t2:387,  status:"hold"  },  
  { sym:"SHEL",    name:"Sanima Hydropower",      sector:"Hydropower",          qty:10,   cost:307,  stop:268,  t1:353,  t2:399,  status:"hold"  },  
  { sym:"NICLBSL", name:"NIC Asia Laghubitta",    sector:"Microfinance",        qty:10,   cost:576,  stop:490,  t1:662,  t2:748,  status:"watch" },  
  { section: "FREE RIDE (Zero Cost)" },  
  { sym:"RSML",    name:"Reliance Spinning",      sector:"Manufacturing",       qty:15,   cost:820,  stop:3000, t1:null, t2:null, status:"free"  },  
  { sym:"NRN",     name:"NRN Infrastructure",     sector:"Infrastructure",      qty:5,    cost:300,  stop:1100, t1:null, t2:null, status:"free"  },  
  { section: "NEEDS REVIEW" },  
  { sym:"CSY",     name:"CSY",                    sector:"Unknown",             qty:100,  cost:null, stop:null, t1:null, t2:null, status:"review"},  
  { sym:"LUK",     name:"LUK",                    sector:"Unknown",             qty:100,  cost:null, stop:null, t1:null, t2:null, status:"review"},  
  { sym:"LEC",     name:"LEC",                    sector:"Unknown",             qty:10,   cost:null, stop:null, t1:null, t2:null, status:"review"},  
];  
  
const WATCHLIST = ["NABIL","KBL","NMB","NIFRA","EBL","NICA","SBI","HIDCL","UMRH","RHPL","SJCL","CHCL","SJLIC","ADBL","NTDC","RSML"];  
  
// Fallback prices (updated from your screenshots)  
const FALLBACK = {  
  NABIL:528, KBL:215, NMB:240.4, NIFRA:261.2, UMRH:532, RHPL:275,  
  SJLIC:429.6, SRLI:390, SJCL:298.7, SHEL:307, NICLBSL:576.5,  
  RSML:3850.2, NRN:1443, CSY:9.32, LUK:9.94, LEC:222,  
  EBL:714, NICA:398, SBI:427, HIDCL:301, CHCL:342, ADBL:293, NTDC:880  
};  
  
// Manual override prices (user editable)  
let manualPrices = JSON.parse(localStorage.getItem('nepse_manual') || '{}');  
  
// ════════════════════════════════════════════════════════════  
// HELPERS  
// ════════════════════════════════════════════════════════════  
const fmt  = n => n == null ? "—" : n >= 1000 ? n.toLocaleString('en-IN',{maximumFractionDigits:1}) : n.toFixed(2);  
const fmtK = n => n >= 1e9 ? (n/1e9).toFixed(2)+'B' : n >= 1e6 ? (n/1e6).toFixed(1)+'M' : n >= 1e3 ? (n/1e3).toFixed(0)+'K' : n;  
const clr  = n => n > 0 ? 'green' : n < 0 ? 'red' : '';  
const arr  = n => n > 0 ? '▲' : n < 0 ? '▼' : '—';  
  
function getPrice(sym) {  
  if (manualPrices[sym]) return manualPrices[sym];  
  if (priceCache[sym])   return priceCache[sym].price;  
  return FALLBACK[sym] || null;  
}  
function getChange(sym) {  
  return priceCache[sym]?.change || 0;  
}  
  
// ════════════════════════════════════════════════════════════  
// NEPAL TIME & MARKET STATUS  
// ════════════════════════════════════════════════════════════  
function updateTime() {  
  const now  = new Date();  
  const nst  = new Date(now.toLocaleString("en-US",{timeZone:"Asia/Kathmandu"}));  
  const h=nst.getHours(), m=nst.getMinutes(), d=nst.getDay();  
  const tStr = nst.toLocaleTimeString("en-US",{hour:"2-digit",minute:"2-digit",second:"2-digit"});  
  const dStr = nst.toLocaleDateString("en-US",{weekday:"short",month:"short",day:"numeric"});  
  document.getElementById("nst-time").textContent = `🕐 NST ${tStr} · ${dStr}`;  
  const open = d>=0&&d<=4 && (h>11||(h===11&&m>=0)) && h<15;  
  const el = document.getElementById("mkt-status");  
  el.textContent = open ? "● MARKET OPEN" : "● MARKET CLOSED";  
  el.className = open ? "mkt-open" : "mkt-closed";  
  document.getElementById("idx-mkt-stat").textContent = open ? "OPEN 🟢" : "CLOSED 🔴";  
  document.getElementById("idx-mkt-stat").className = "idx-value " + (open?"green":"red");  
}  
setInterval(updateTime,1000); updateTime();  
  
// ════════════════════════════════════════════════════════════  
// FETCH PRICES  
// ════════════════════════════════════════════════════════════  
function setApiStatus(state, text) {  
  const el = document.getElementById("api-status");  
  const dot = el.querySelector(".api-dot");  
  el.className = "api-status " + state;  
  document.getElementById("api-status-text").textContent = text;  
  if(state==="connecting") dot.classList.add("pulse"); else dot.classList.remove("pulse");  
}  
  
function setBanner(type, text) {  
  const b = document.getElementById("api-banner");  
  b.className = "api-banner " + type;  
  document.getElementById("api-banner-text").textContent = text;  
}  
  
async function fetchSinglePrice(sym, source) {  
  const src = API_SOURCES[source];  
  if (!src) return null;  
  try {  
    const resp = await fetch(src.url(sym), { signal: AbortSignal.timeout(8000) });  
    if (!resp.ok) throw new Error("HTTP " + resp.status);  
    const data = await resp.json();  
    const parsed = src.parse(data);  
    if (parsed.price > 0) return parsed;  
    return null;  
  } catch(e) {  
    return null;  
  }  
}  
  
async function fetchAllPrices() {  
  if (currentSource === 'manual') { buildPortfolio(); buildAlerts(); buildMarket(); buildTicker(); return; }  
  
  const btn = document.getElementById("refresh-btn");  
  btn.textContent = "⟳";  
  btn.classList.add("spinning");  
  setApiStatus("connecting", "Fetching...");  
  setBanner("info", `Fetching live prices from ${API_SOURCES[currentSource]?.name || currentSource}...`);  
  
  const allSyms = [...new Set([  
    ...PORTFOLIO.filter(s=>s.sym).map(s=>s.sym),  
    ...WATCHLIST  
  ])];  
  
  let successCount = 0;  
  let failCount = 0;  
  
  // Fetch in batches to avoid rate limits  
  const batchSize = 3;  
  for (let i = 0; i < allSyms.length; i += batchSize) {  
    const batch = allSyms.slice(i, i+batchSize);  
    await Promise.all(batch.map(async sym => {  
      const result = await fetchSinglePrice(sym, currentSource);  
      if (result && result.price > 0) {  
        priceCache[sym] = result;  
        successCount++;  
      } else {  
        failCount++;  
      }  
    }));  
    // Small delay between batches to respect rate limits  
    if (i + batchSize < allSyms.length) await new Promise(r => setTimeout(r, 400));  
  }  
  
  lastFetch = new Date();  
  document.getElementById("last-upd").textContent = "Updated: " + lastFetch.toLocaleTimeString();  
  
  if (successCount > 0) {  
    setApiStatus("connected", `Live · ${successCount} stocks`);  
    setBanner("success", `✅ Live prices loaded for ${successCount} stocks via ${API_SOURCES[currentSource].name}. ${failCount > 0 ? failCount + ' used fallback.' : ''} Data from official NEPSE public channels.`);  
  } else {  
    setApiStatus("offline", "API Offline");  
    setBanner("error", `❌ API unreachable. Using fallback prices. Try switching API source or use Manual mode. Verify prices on merolagani.com before trading.`);  
  }  
  
  buildPortfolio();  
  buildAlerts();  
  buildMarket();  
  buildTicker();  
  
  btn.textContent = "⟳ REFRESH";  
  btn.classList.remove("spinning");  
  
  // Alert on stop loss hits  
  PORTFOLIO.filter(s=>s.sym).forEach(row => {  
    const p = getPrice(row.sym);  
    if (row.stop && p && p <= row.stop && !alertsShown.has(row.sym+"_stop")) {  
      alertsShown.add(row.sym+"_stop");  
      toast(`🚨 ${row.sym} STOP LOSS HIT!`, `Price NPR ${fmt(p)} ≤ Stop NPR ${fmt(row.stop)}. SELL NOW!`, "danger");  
      playAlert();  
    }  
    if (row.t1 && p && p >= row.t1 && !alertsShown.has(row.sym+"_t1")) {  
      alertsShown.add(row.sym+"_t1");  
      toast(`🎯 ${row.sym} TARGET REACHED!`, `Price NPR ${fmt(p)} ≥ Target NPR ${fmt(row.t1)}. Consider selling 25%.`, "success");  
      playAlert();  
    }  
  });  
}  
  
function playAlert() {  
  try {  
    const ctx = new (window.AudioContext || window.webkitAudioContext)();  
    [440,550,660].forEach((f,i) => {  
      const o = ctx.createOscillator(); const g = ctx.createGain();  
      o.connect(g); g.connect(ctx.destination);  
      o.frequency.value = f; g.gain.value = 0.15;  
      o.start(ctx.currentTime + i*0.15);  
      o.stop(ctx.currentTime + i*0.15 + 0.12);  
    });  
  } catch(e) {}  
}  
  
// ════════════════════════════════════════════════════════════  
// SOURCE TOGGLE  
// ════════════════════════════════════════════════════════════  
function setSource(src) {  
  currentSource = src;  
  document.getElementById("src1").className = "src-btn" + (src==="sharebazaar"?" active":"");  
  document.getElementById("src2").className = "src-btn" + (src==="nepseapi"?" active":"");  
  document.getElementById("src3").className = "src-btn" + (src==="manual"?" active":"");  
  if (src === "manual") {  
    setBanner("warning", "⚠️ Manual mode: Prices shown are your last entered values. Go to Settings to update prices manually from merolagani.com.");  
    setApiStatus("offline", "Manual Mode");  
  } else {  
    fetchAllPrices();  
  }  
}  
  
// ════════════════════════════════════════════════════════════  
// TICKER  
// ════════════════════════════════════════════════════════════  
function buildTicker() {  
  const syms = WATCHLIST.slice(0,12);  
  const items = syms.map(sym => {  
    const p = getPrice(sym); const c = getChange(sym);  
    const cls = c >= 0 ? "green" : "red";  
    return `<span class="t-item"><span class="t-sym">${sym}</span><span class="t-prc">NPR ${fmt(p)}</span><span class="${cls}">${arr(c)}${Math.abs(c).toFixed(2)}%</span></span>`;  
  }).join("");  
  document.getElementById("ticker").innerHTML = items + items;  
}  
  
// ════════════════════════════════════════════════════════════  
// PORTFOLIO TABLE  
// ════════════════════════════════════════════════════════════  
function buildPortfolio() {  
  const tbody = document.getElementById("ptable");  
  tbody.innerHTML = "";  
  let totalCost=0, totalValue=0, alertCount=0;  
  
  PORTFOLIO.forEach(row => {  
    if (row.section) {  
      const tr = document.createElement("tr");  
      tr.className = "section-row";  
      tr.innerHTML = `<td colspan="12">${row.section}</td>`;  
      tbody.appendChild(tr); return;  
    }  
  
    const ltp = getPrice(row.sym);  
    const dayChg = getChange(row.sym);  
    const qty  = row.qty;  
    const cost = row.cost;  
    const value = qty && ltp ? qty * ltp : null;  
    const costTotal = qty && cost ? qty * cost : null;  
    const pnl  = value && costTotal ? value - costTotal : null;  
    const pnlP = pnl && costTotal ? (pnl/costTotal)*100 : null;  
  
    if (costTotal) totalCost  += costTotal;  
    if (value)     totalValue += value;  
  
    // Status logic  
    let rowClass="", statusKey=row.status;  
    if (row.stop && ltp && ltp <= row.stop)           { rowClass="row-stop";   statusKey="alert";  alertCount++; }  
    else if (row.stop && ltp && ltp <= row.stop*1.06) { rowClass="row-near";                        alertCount++; }  
    if (row.t1 && ltp && ltp >= row.t1)               { rowClass="row-target"; statusKey="target"; alertCount++; }  
  
    const statusLabels = { hold:"HOLD ✅", watch:"WATCH ⚠️", free:"FREE RIDE 🎯", review:"REVIEW ❓", alert:"🔴 STOP HIT", target:"🎯 TARGET!" };  
    const statusCls    = { hold:"s-hold", watch:"s-watch", free:"s-free", review:"s-review", alert:"s-alert", target:"s-target" };  
  
    const dayClr = clr(dayChg);  
    const pnlClr = pnl != null ? clr(pnl) : "";  
    const srcTag = priceCache[row.sym] ? `<span style="font-size:0.55rem;color:var(--green);margin-left:3px">●LIVE</span>` : `<span style="font-size:0.55rem;color:var(--grey);margin-left:3px">●LAST</span>`;  
  
    const tr = document.createElement("tr");  
    tr.className = rowClass;  
    tr.innerHTML = `  
      <td class="sname">${row.sym}${srcTag}${row.note?`<br><span style="font-size:0.6rem;color:var(--orange)">${row.note}</span>`:""}</td>  
      <td class="ssector">${row.sector}</td>  
      <td>${qty ?? "?"}</td>  
      <td style="color:var(--grey)">${cost?"NPR "+fmt(cost):"—"}</td>  
      <td class="${dayClr}" style="font-weight:700">${ltp?"NPR "+fmt(ltp):"—"}</td>  
      <td class="${dayClr}">${dayChg?(arr(dayChg)+" "+Math.abs(dayChg).toFixed(2)+"%"):"—"}</td>  
      <td>${value?"NPR "+fmt(value):"—"}</td>  
      <td class="${pnlClr}">${pnl!=null?(pnl>=0?"+":"")+fmt(pnl):"—"}</td>  
      <td class="${pnlClr}">${pnlP!=null?(pnlP>=0?"+":"")+pnlP.toFixed(1)+"%":"—"}</td>  
      <td style="color:var(--red)">${row.stop?fmt(row.stop):"—"}</td>  
      <td style="color:var(--green)">${row.t1?fmt(row.t1):"—"}</td>  
      <td><span class="status-badge ${statusCls[statusKey]}">${statusLabels[statusKey]}</span></td>  
    `;  
    tbody.appendChild(tr);  
  });  
  
  // Summary  
  const pnl = totalValue - totalCost;  
  const pnlP = totalCost ? (pnl/totalCost)*100 : 0;  
  document.getElementById("c-invested").textContent = "NPR " + fmt(totalCost);  
  document.getElementById("c-value").textContent    = "NPR " + fmt(totalValue);  
  const pe = document.getElementById("c-pnl");  
  pe.textContent   = (pnl>=0?"+":"") + "NPR " + fmt(Math.abs(pnl));  
  pe.className     = "card-value " + (pnl>=0?"green":"red");  
  document.getElementById("c-pnl-pct").textContent  = (pnlP>=0?"+":"") + pnlP.toFixed(1) + "% unrealised";  
  document.getElementById("c-alerts").textContent   = alertCount + " active";  
  document.getElementById("portfolio-badge").textContent = `${priceCache&&Object.keys(priceCache).length>0?"🟢 Live":"⚪ Fallback"} · Updated ${lastFetch?lastFetch.toLocaleTimeString():"never"}`;  
}  
  
// ════════════════════════════════════════════════════════════  
// ALERTS  
// ════════════════════════════════════════════════════════════  
function buildAlerts() {  
  const list = document.getElementById("alerts-list");  
  list.innerHTML = "";  
  let found = false;  
  
  PORTFOLIO.filter(s=>s.sym).forEach(row => {  
    const p = getPrice(row.sym); if (!p) return;  
    const c = getChange(row.sym);  
  
    if (row.stop && p <= row.stop) {  
      found = true;  
      list.innerHTML += alertHTML("danger","🚨",`${row.sym} — STOP LOSS HIT`,`Price NPR ${fmt(p)} has hit your stop loss of NPR ${fmt(row.stop)}. SELL NOW per your rules. No hoping.`,`NPR ${fmt(p)}`,"var(--red)");  
    } else if (row.stop && p <= row.stop*1.05) {  
      found = true;  
      const dist = (((p-row.stop)/row.stop)*100).toFixed(1);  
      list.innerHTML += alertHTML("warning","⚠️",`${row.sym} — Near Stop Loss (${dist}% away)`,`Price is dangerously close to stop loss NPR ${fmt(row.stop)}. Monitor closely. Be ready to sell immediately.`,`NPR ${fmt(p)}`,"var(--orange)");  
    }  
    if (row.t1 && p >= row.t1) {  
      found = true;  
      list.innerHTML += alertHTML("success","🎯",`${row.sym} — TARGET 1 REACHED!`,`Sell 25% of your position now. Move stop loss to your entry price (breakeven). Let rest ride.`,`NPR ${fmt(p)}`,"var(--green)");  
    }  
    if (row.cost && p <= row.cost*0.97 && (!row.stop || p > row.stop)) {  
      found = true;  
      list.innerHTML += alertHTML("info","📉",`${row.sym} — Below Your Cost`,`Price dipped ${((p-row.cost)/row.cost*100).toFixed(1)}% below your entry. Check RSI + support before adding. Don't average down blindly.`,`NPR ${fmt(p)}`,"var(--gold)");  
    }  
  });  
  
  // Buy zone alerts  
  const kbl = getPrice("KBL");  
  if (kbl && kbl <= 207) { found=true; list.innerHTML += alertHTML("success","💰","KBL — Tranche 2 Buy Zone!",`KBL at NPR ${fmt(kbl)} is in your ideal buy zone NPR 200-207. Consider buying ~75 more shares.`,`NPR ${fmt(kbl)}`,"var(--green)"); }  
  const nabil = getPrice("NABIL");  
  if (nabil && nabil <= 515) { found=true; list.innerHTML += alertHTML("success","💰","NABIL — Add More Opportunity",`NABIL at NPR ${fmt(nabil)} is in your add zone NPR 505-515. Consider buying ~78 more shares.`,`NPR ${fmt(nabil)}`,"var(--green)"); }  
  const nmb = getPrice("NMB");  
  if (nmb && nmb <= 232) { found=true; list.innerHTML += alertHTML("success","💰","NMB — Add More Opportunity",`NMB at NPR ${fmt(nmb)} is in your buy zone NPR 225-230. Consider buying ~108 more shares.`,`NPR ${fmt(nmb)}`,"var(--green)"); }  
  
  if (!found) {  
    list.innerHTML = alertHTML("info","✅","All Clear — No Urgent Alerts","All positions within normal ranges. No stop losses triggered. No targets hit. Keep monitoring weekly.","","") +  
    alertHTML("info","📋","Reminder: Verify with merolagani.com","Always confirm prices on merolagani.com before making any buy or sell decision. ●LIVE tag = API price, ●LAST = fallback price.","","");  
  }  
}  
  
function alertHTML(type,icon,title,desc,price,color) {  
  return `<div class="alert-card ${type}">  
    <div class="a-icon">${icon}</div>  
    <div class="a-body"><div class="a-title">${title}</div><div class="a-desc">${desc}</div></div>  
    ${price?`<div class="a-price" style="color:${color}">${price}</div>`:""}  
  </div>`;  
}  
  
// ════════════════════════════════════════════════════════════  
// MARKET GRID  
// ════════════════════════════════════════════════════════════  
function buildMarket() {  
  const grid = document.getElementById("mkt-grid");  
  grid.innerHTML = "";  
  const portSyms = new Set(PORTFOLIO.filter(s=>s.sym).map(s=>s.sym));  
  WATCHLIST.forEach(sym => {  
    const p = getPrice(sym); const c = getChange(sym);  
    const cls = c >= 0 ? "green" : "red";  
    const owned = portSyms.has(sym);  
    const fillPct = Math.min(100, Math.max(5, 50 + c*8));  
    const fillColor = c>=0 ? "var(--green)" : "var(--red)";  
    const live = priceCache[sym] ? `<span style="font-size:0.55rem;color:var(--green)">●LIVE</span>` : `<span style="font-size:0.55rem;color:var(--grey)">●LAST</span>`;  
    grid.innerHTML += `<div class="mkt-card${owned?" owned":""}">  
      <div class="mc-sym">${sym} ${owned?"⭐":""} ${live}</div>  
      <div class="mc-price ${cls}">NPR ${fmt(p)}</div>  
      <div class="mc-chg ${cls}">${arr(c)} ${Math.abs(c).toFixed(2)}%</div>  
      <div class="mc-sec">${priceCache[sym]?"Live via API":"Fallback price"}</div>  
      <div class="mc-bar"><div class="mc-fill" style="width:${fillPct}%;background:${fillColor}"></div></div>  
    </div>`;  
  });  
}  
  
// ════════════════════════════════════════════════════════════  
// SETTINGS  
// ════════════════════════════════════════════════════════════  
function buildSettings() {  
  const grid = document.getElementById("settings-grid");  
  const portStocks = PORTFOLIO.filter(s=>s.sym);  
  
  let html = `<div class="s-card">  
    <h3>✏️ MANUAL PRICE OVERRIDE</h3>  
    <p style="font-size:0.7rem;color:var(--grey);margin-bottom:12px">Override API with your own prices from merolagani.com. These take priority over API data.</p>`;  
  portStocks.forEach(s => {  
    const v = manualPrices[s.sym] || "";  
    html += `<div class="s-row">  
      <span class="s-label"><strong>${s.sym}</strong><br><span style="font-size:0.65rem;color:var(--grey)">${s.name}</span></span>  
      <input type="number" id="mp-${s.sym}" value="${v}" placeholder="${FALLBACK[s.sym]||""}" step="0.1" min="0">  
    </div>`;  
  });  
  html += `</div>`;  
  
  html += `<div class="s-card">  
    <h3>🌐 API SOURCES</h3>  
    <div class="s-row"><span class="s-label">ShareBazaar</span><span class="s-val" style="color:var(--green)">Free · No key needed</span></div>  
    <div class="s-row"><span class="s-label">URL</span><span class="s-val" style="font-size:0.65rem;color:var(--grey)">nepsetty.kokomo.workers.dev</span></div>  
    <div style="height:10px"></div>  
    <div class="s-row"><span class="s-label">NepseAPI Unofficial</span><span class="s-val" style="color:var(--green)">Free · Educational</span></div>  
    <div class="s-row"><span class="s-label">URL</span><span class="s-val" style="font-size:0.65rem;color:var(--grey)">nepseapi.surajrimal.dev</span></div>  
    <div style="height:10px"></div>  
    <div class="s-row"><span class="s-label">Data Reliability</span><span class="s-val" style="color:var(--orange)">ALWAYS verify on merolagani</span></div>  
    <div class="s-row"><span class="s-label">Rate Limit</span><span class="s-val">~60 req/min max</span></div>  
    <div class="s-row"><span class="s-label">Auto Refresh</span><span class="s-val">Every 5 minutes</span></div>  
  </div>`;  
  
  html += `<div class="s-card">  
    <h3>📋 YOUR RULES (Read Before Trading)</h3>  
    <div class="s-row"><span class="s-label" style="color:var(--red)">Stop Loss</span><span class="s-val">10-13% below entry</span></div>  
    <div class="s-row"><span class="s-label" style="color:var(--green)">Take Profit</span><span class="s-val">25% at each +10%</span></div>  
    <div class="s-row"><span class="s-label">Cash Reserve</span><span class="s-val">NPR 32,720 — sacred</span></div>  
    <div class="s-row"><span class="s-label">Max Stocks</span><span class="s-val">3-4 core</span></div>  
    <div class="s-row"><span class="s-label">Price Check</span><span class="s-val">Once per day max</span></div>  
    <div class="s-row"><span class="s-label">Borrowed Money</span><span class="s-val" style="color:var(--red)">NEVER</span></div>  
    <div class="s-row"><span class="s-label">Monthly Review</span><span class="s-val">Last Sunday</span></div>  
  </div>`;  
  
  grid.innerHTML = html;  
}  
  
function saveManualPrices() {  
  PORTFOLIO.filter(s=>s.sym).forEach(s => {  
    const el = document.getElementById("mp-"+s.sym);  
    if (el && el.value) manualPrices[s.sym] = parseFloat(el.value);  
    else if (el && !el.value) delete manualPrices[s.sym];  
  });  
  localStorage.setItem('nepse_manual', JSON.stringify(manualPrices));  
  buildPortfolio(); buildAlerts(); buildMarket(); buildTicker();  
  toast("Manual Prices Saved ✅","Your price overrides are saved and applied.", "success");  
}  
  
function clearCache() {  
  priceCache = {}; manualPrices = {};  
  localStorage.removeItem('nepse_manual');  
  buildSettings(); buildPortfolio(); buildMarket(); buildTicker();  
  toast("Cache Cleared","All overrides cleared. API prices will be used.", "warning");  
}  
  
// ════════════════════════════════════════════════════════════  
// TABS  
// ════════════════════════════════════════════════════════════  
function showTab(name, el) {  
  document.querySelectorAll(".panel").forEach(p=>p.classList.remove("active"));  
  document.querySelectorAll(".tab").forEach(t=>t.classList.remove("active"));  
  document.getElementById("panel-"+name).classList.add("active");  
  el.classList.add("active");  
  if (name==="settings") buildSettings();  
  if (name==="market")   buildMarket();  
}  
  
// ════════════════════════════════════════════════════════════  
// TOAST  
// ════════════════════════════════════════════════════════════  
function toast(title, msg, type="info") {  
  const c = document.getElementById("toasts");  
  const t = document.createElement("div");  
  t.className = "toast " + type;  
  t.innerHTML = `<div class="t-title">${title}</div><div class="t-msg">${msg}</div>`;  
  c.appendChild(t);  
  setTimeout(() => { t.style.opacity="0"; t.style.transform="translateX(120%)"; t.style.transition="all 0.3s"; setTimeout(()=>t.remove(),300); }, 5000);  
}  
  
// ════════════════════════════════════════════════════════════  
// INIT  
// ════════════════════════════════════════════════════════════  
// Build with fallback immediately  
buildPortfolio(); buildAlerts(); buildMarket(); buildTicker(); buildSettings();  
  
// Then fetch live  
setTimeout(fetchAllPrices, 800);  
  
// Auto refresh every 5 minutes  
autoRefreshTimer = setInterval(() => {  
  if (currentSource !== "manual") fetchAllPrices();  
}, 5 * 60 * 1000);  
  
// Welcome  
setTimeout(() => toast("🇳🇵 NEPSE Live Tracker", "Connecting to free NEPSE API. Prices load in ~10 seconds.", "info"), 500);  
</script>  
</body>  
</html>  
