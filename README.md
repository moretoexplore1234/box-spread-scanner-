# box-spread-scanner-index.html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>BoxScan — Options Arbitrage Terminal</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Syne:wght@400;700;800&display=swap" rel="stylesheet">
<style>
  :root {
    --bg:#070711; --surface:#0c0c1a; --panel:#101022;
    --border:#1a1a30; --border-bright:#252545;
    --amber:#f0a500; --amber-dim:#7a5200; --amber-faint:#1f1400;
    --green:#00e676; --green-dim:#00502a; --green-faint:#001a0e;
    --red:#ff4060; --red-dim:#600018; --red-faint:#1a0008;
    --text:#d8d8f0; --text-dim:#6060a0; --text-muted:#303060;
    --mono:'Space Mono',monospace; --sans:'Segoe UI',system-ui,sans-serif;
  }
  *{box-sizing:border-box;margin:0;padding:0}
  html{scroll-behavior:smooth}
  body{
    background:var(--bg);color:var(--text);font-family:var(--sans);min-height:100vh;
    background-image:linear-gradient(var(--border) 1px,transparent 1px),
                     linear-gradient(90deg,var(--border) 1px,transparent 1px);
    background-size:40px 40px;
  }
  input,button,select{font-family:inherit}
  ::-webkit-scrollbar{width:6px;height:6px}
  ::-webkit-scrollbar-track{background:var(--bg)}
  ::-webkit-scrollbar-thumb{background:var(--border);border-radius:3px}
  input[type=number]::-webkit-inner-spin-button{-webkit-appearance:none}

  /* ── Layout ── */
  #app{max-width:1400px;margin:0 auto;padding:20px 24px}

  /* ── Header ── */
  #header{
    background:var(--surface);border-bottom:1px solid var(--border-bright);
    padding:0 24px;display:flex;align-items:center;justify-content:space-between;
    height:56px;position:sticky;top:0;z-index:50;backdrop-filter:blur(12px);
  }
  .logo-name{font-family:'Syne',sans-serif;font-weight:800;font-size:15px;color:var(--amber);letter-spacing:3px;text-transform:uppercase}
  .logo-sub{font-family:var(--mono);font-size:9px;color:var(--text-dim);letter-spacing:2px;margin-top:2px}
  .divider{width:1px;height:28px;background:var(--border);margin:0 4px}
  .badge{
    background:rgba(0,0,0,.2);border:1px solid;border-radius:3px;
    padding:1px 7px;font-family:var(--mono);font-size:9px;letter-spacing:1px;
    white-space:nowrap;
  }
  .badge-amber{color:var(--amber);border-color:rgba(240,165,0,.3);background:rgba(240,165,0,.1)}
  .badge-green{color:var(--green);border-color:rgba(0,230,118,.3);background:rgba(0,230,118,.1)}
  .header-right{display:flex;gap:12px;align-items:center}
  #last-scan{font-family:var(--mono);font-size:10px;color:var(--text-dim)}
  #config-toggle{
    background:var(--amber-faint);border:1px solid var(--amber);color:var(--amber);
    padding:5px 12px;border-radius:4px;cursor:pointer;
    font-family:var(--mono);font-size:10px;letter-spacing:1px;transition:all .2s;
  }

  /* ── Config Panel ── */
  #config-panel{
    background:var(--surface);border:1px solid var(--border-bright);
    border-radius:8px;padding:20px 24px;margin-bottom:20px;
    animation:fadeIn .2s ease;
  }
  .config-row{display:flex;flex-wrap:wrap;gap:20px;align-items:flex-end;margin-top:16px}
  .field{display:flex;flex-direction:column;gap:4px}
  .field label{font-size:10px;color:var(--text-dim);letter-spacing:1px;text-transform:uppercase}
  .field input{
    background:var(--bg);border:1px solid var(--border);border-radius:4px;
    color:var(--text);font-family:var(--mono);font-size:13px;padding:6px 10px;
    outline:none;transition:border-color .2s;width:100%;
  }
  .field input:focus{border-color:var(--amber)}
  .field .hint{font-size:9px;color:var(--text-muted)}
  .mode-btns,.env-btns{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
  .mode-label{font-family:var(--mono);font-size:10px;color:var(--text-dim);letter-spacing:2px;text-transform:uppercase;margin-right:4px}
  .toggle-btn{
    border:1px solid var(--border);border-radius:4px;padding:4px 12px;cursor:pointer;
    font-family:var(--mono);font-size:10px;letter-spacing:1px;text-transform:uppercase;
    background:transparent;color:var(--text-dim);transition:all .15s;
  }
  .toggle-btn.active-amber{border-color:var(--amber);background:var(--amber-faint);color:var(--amber)}
  .toggle-btn.active-green{border-color:var(--green);background:var(--green-faint);color:var(--green)}
  .tradier-note{
    margin-top:14px;padding:10px 14px;background:var(--amber-faint);
    border:1px solid var(--amber-dim);border-radius:4px;
    font-size:11px;color:var(--amber);line-height:1.7;
  }
  #scan-btn{
    background:var(--green-faint);border:1px solid var(--green);color:var(--green);
    padding:8px 28px;border-radius:4px;cursor:pointer;
    font-family:var(--mono);font-size:11px;letter-spacing:2px;font-weight:bold;
    box-shadow:0 0 20px rgba(0,230,118,.2);transition:box-shadow .2s;white-space:nowrap;
  }
  #scan-btn:hover{box-shadow:0 0 40px rgba(0,230,118,.4)}
  #stop-btn{
    background:var(--red-faint);border:1px solid var(--red);color:var(--red);
    padding:8px 20px;border-radius:4px;cursor:pointer;
    font-family:var(--mono);font-size:11px;letter-spacing:2px;white-space:nowrap;
  }

  /* ── Progress ── */
  #progress-wrap{
    background:var(--surface);border:1px solid var(--border);
    border-radius:8px;padding:16px 20px;margin-bottom:20px;display:none;
  }
  .prog-row{display:flex;justify-content:space-between;margin-bottom:10px}
  .prog-label{font-family:var(--mono);font-size:11px;color:var(--amber)}
  .prog-count{font-family:var(--mono);font-size:11px;color:var(--text-dim)}
  .prog-track{height:4px;background:var(--border);border-radius:2px;overflow:hidden}
  #prog-fill{height:100%;background:linear-gradient(90deg,var(--amber),var(--green));border-radius:2px;width:0%;transition:width .3s;box-shadow:0 0 8px rgba(0,230,118,.5)}
  #prog-found{margin-top:8px;font-family:var(--mono);font-size:10px;color:var(--green)}

  /* ── Error ── */
  #error-wrap{
    background:var(--red-faint);border:1px solid var(--red-dim);
    border-radius:6px;padding:12px 16px;margin-bottom:20px;display:none;
    font-family:var(--mono);font-size:11px;color:var(--red);
  }

  /* ── Stats ── */
  #stats-bar{
    display:flex;gap:1px;margin-bottom:16px;
    background:var(--border);border-radius:8px;overflow:hidden;display:none;
  }
  .stat-cell{flex:1;background:var(--panel);padding:12px 16px;min-width:140px}
  .stat-label{font-size:9px;color:var(--text-dim);text-transform:uppercase;letter-spacing:1px;margin-bottom:4px}
  .stat-value{font-family:var(--mono);font-size:14px;color:var(--amber)}

  /* ── Results Table ── */
  #results-wrap{
    background:var(--surface);border:1px solid var(--border);
    border-radius:8px;overflow:hidden;display:none;
  }
  .results-header{
    padding:12px 16px;border-bottom:1px solid var(--border);
    display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:8px;
  }
  .results-title{font-family:var(--mono);font-size:11px;color:var(--text-dim);letter-spacing:2px}
  .sort-group{display:flex;gap:6px;align-items:center}
  .sort-label{font-family:var(--mono);font-size:9px;color:var(--text-muted)}
  .sort-btn{
    border:1px solid var(--border);border-radius:3px;padding:3px 8px;cursor:pointer;
    font-family:var(--mono);font-size:9px;letter-spacing:1px;
    background:transparent;color:var(--text-dim);transition:all .15s;
  }
  .sort-btn.active{border-color:var(--amber);background:var(--amber-faint);color:var(--amber)}
  .table-scroll{overflow-x:auto}
  table{width:100%;border-collapse:collapse}
  thead th{
    padding:8px 10px;font-family:var(--mono);font-size:9px;color:var(--text-dim);
    letter-spacing:1px;text-transform:uppercase;background:var(--bg);
    position:sticky;top:0;user-select:none;white-space:nowrap;
  }
  thead th.sortable{cursor:pointer}
  thead th.sortable.active{color:var(--amber)}
  thead th.r,td.r{text-align:right}
  tbody tr{border-bottom:1px solid var(--border);cursor:pointer;transition:background .15s}
  tbody tr:hover{background:rgba(255,255,255,.04)}
  tbody tr.sel{background:var(--green-faint)}
  td{padding:7px 10px;font-family:var(--mono);font-size:12px;color:var(--text);white-space:nowrap}
  td.col-net{color:var(--green)}
  td.col-roi{color:var(--amber)}

  /* ── Empty State ── */
  #empty{text-align:center;padding:80px 20px}
  .empty-icon{font-size:40px;color:var(--border);margin-bottom:16px}
  .empty-title{font-family:'Syne',sans-serif;font-weight:800;font-size:22px;color:var(--text-dim);margin-bottom:8px}
  .empty-body{font-size:13px;color:var(--text-muted);max-width:440px;margin:0 auto;line-height:1.7}
  .sym-chips{display:flex;gap:16px;justify-content:center;flex-wrap:wrap;margin-top:24px}
  .sym-chip{
    border:1px solid var(--border);border-radius:4px;padding:6px 16px;cursor:pointer;
    font-family:var(--mono);font-size:11px;color:var(--text-dim);background:transparent;transition:all .15s;
  }
  .sym-chip:hover,.sym-chip.active{border-color:var(--amber);background:var(--amber-faint);color:var(--amber)}

  /* ── Footer ── */
  #footer{margin-top:32px;padding:16px 0;border-top:1px solid var(--border);font-size:10px;color:var(--text-muted);line-height:1.8}

  /* ── Modal ── */
  #modal-overlay{
    position:fixed;inset:0;background:rgba(0,0,0,.55);
    display:flex;align-items:center;justify-content:center;z-index:100;display:none;
  }
  #modal{
    background:var(--panel);border:1px solid var(--border-bright);
    border-radius:8px;width:600px;max-width:96vw;max-height:90vh;
    overflow:hidden;display:flex;flex-direction:column;
    box-shadow:0 24px 80px rgba(0,0,0,.8);
  }
  #modal-head{
    background:var(--green-faint);border-bottom:1px solid var(--border-bright);
    padding:14px 20px;display:flex;justify-content:space-between;align-items:center;flex-shrink:0;
  }
  #modal-title{font-family:var(--mono);font-size:16px;color:var(--green);letter-spacing:2px}
  #modal-sub{font-family:var(--mono);font-size:9px;color:var(--text-dim);letter-spacing:1px;margin-top:3px}
  #modal-close{background:none;border:none;color:var(--text-dim);cursor:pointer;font-size:20px;line-height:1;padding:2px 6px}
  #modal-close:hover{color:var(--text)}
  #modal-body{overflow-y:auto;flex:1}
  .metric-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:1px;background:var(--border);margin-bottom:1px}
  .metric-cell{background:var(--panel);padding:12px 14px}
  .metric-label{font-size:9px;color:var(--text-dim);text-transform:uppercase;letter-spacing:1px;margin-bottom:4px}
  .metric-value{font-family:var(--mono);font-size:15px;color:var(--text)}
  .metric-value.green{color:var(--green)}
  .metric-value.amber{color:var(--amber)}
  .metric-value.red{color:var(--red)}
  .legs-section{padding:14px 20px}
  .legs-title{font-size:10px;color:var(--text-dim);text-transform:uppercase;letter-spacing:1px;margin-bottom:10px}
  .leg-row{
    display:flex;justify-content:space-between;align-items:center;
    padding:8px 12px;margin-bottom:4px;border-radius:4px;border:1px solid;gap:12px;flex-wrap:wrap;
  }
  .leg-row.ask{border-color:var(--red-dim);background:var(--red-faint)}
  .leg-row.bid{border-color:var(--green-dim);background:var(--green-faint)}
  .leg-label{font-family:var(--mono);font-size:11px;font-weight:700}
  .leg-row.ask .leg-label{color:var(--red)}
  .leg-row.bid .leg-label{color:var(--green)}
  .leg-meta{font-family:var(--mono);font-size:11px;color:var(--text-dim)}
  .modal-disclaimer{padding:10px 20px 16px;border-top:1px solid var(--border);font-size:10px;color:var(--text-muted);line-height:1.7}

  @keyframes fadeIn{from{opacity:0;transform:translateY(-8px)}to{opacity:1;transform:none}}

  /* ── Responsive ── */
  @media(max-width:700px){
    #app{padding:12px 12px}
    #header{padding:0 12px}
    .config-row{flex-direction:column;gap:12px}
    .field input{width:100%!important}
    .metric-grid{grid-template-columns:repeat(2,1fr)}
    thead th:nth-child(4),td:nth-child(4),
    thead th:nth-child(5),td:nth-child(5){display:none}
  }
</style>
</head>
<body>

<!-- ── Header ── -->
<div id="header">
  <div style="display:flex;align-items:center;gap:12px">
    <div>
      <div class="logo-name">BoxScan</div>
      <div class="logo-sub">OPTIONS ARBITRAGE TERMINAL</div>
    </div>
    <div class="divider"></div>
    <span id="mode-badge" class="badge badge-amber">DEMO MODE</span>
    <span id="count-badge" class="badge badge-green" style="display:none"></span>
  </div>
  <div class="header-right">
    <span id="last-scan" style="display:none"></span>
    <button id="config-toggle">⚙ CONFIG</button>
  </div>
</div>

<!-- ── Main ── -->
<div id="app">

  <!-- Config Panel -->
  <div id="config-panel">
    <!-- Mode row -->
    <div class="mode-btns">
      <span class="mode-label">Data Source</span>
      <button class="toggle-btn active-amber" data-mode="demo">Demo / Synthetic</button>
      <button class="toggle-btn" data-mode="tradier">Tradier API</button>
    </div>

    <!-- Inputs row -->
    <div class="config-row">
      <div class="field" style="width:90px">
        <label>Symbol</label>
        <input id="inp-symbol" value="SPY" maxlength="6">
      </div>
      <div class="field" style="width:110px">
        <label>Min Open Interest</label>
        <input id="inp-minoi" type="number" value="100">
        <span class="hint">Per leg</span>
      </div>
      <div class="field" style="width:90px">
        <label>Min Volume</label>
        <input id="inp-minvol" type="number" value="10">
        <span class="hint">Per leg</span>
      </div>
      <div class="field" style="width:100px">
        <label>Fee / Leg ($)</label>
        <input id="inp-fee" type="number" step="0.01" value="0.65">
        <span class="hint">Per contract</span>
      </div>
      <div class="field" style="width:105px">
        <label>Max Contracts</label>
        <input id="inp-maxc" type="number" value="10">
        <span class="hint">Per spread</span>
      </div>
      <div class="field" style="width:110px">
        <label>Risk-Free Rate %</label>
        <input id="inp-rf" type="number" step="0.01" value="4.50">
        <span class="hint">Annualized</span>
      </div>
      <div class="field" style="width:90px">
        <label>Max DTE</label>
        <input id="inp-dte" type="number" value="90">
        <span class="hint">Days</span>
      </div>

      <!-- Tradier fields (hidden by default) -->
      <div id="tradier-fields" style="display:none;display:flex;gap:16px;flex-wrap:wrap;align-items:flex-end">
        <div class="field" style="width:240px">
          <label>Tradier API Key</label>
          <input id="inp-apikey" type="password" placeholder="Bearer token">
        </div>
        <div class="field">
          <label>Environment</label>
          <div class="env-btns">
            <button class="toggle-btn active-amber" data-env="sandbox">Sandbox</button>
            <button class="toggle-btn" data-env="live">Live</button>
          </div>
        </div>
      </div>

      <div style="margin-left:auto;display:flex;gap:10px">
        <button id="stop-btn" style="display:none">■ STOP</button>
        <button id="scan-btn">▶ SCAN SPY</button>
      </div>
    </div>

    <div id="tradier-note" class="tradier-note" style="display:none">
      <strong>Tradier Setup:</strong> Get a free API key at
      <code style="background:rgba(0,0,0,.3);padding:0 4px;border-radius:2px">tradier.com/settings/api</code>.
      Use Sandbox for testing. For live NBBO, select Live and use your production bearer token.
    </div>
  </div>

  <!-- Progress -->
  <div id="progress-wrap">
    <div class="prog-row">
      <span class="prog-label" id="prog-label">SCANNING...</span>
      <span class="prog-count" id="prog-count">0 / 0</span>
    </div>
    <div class="prog-track"><div id="prog-fill"></div></div>
    <div id="prog-found" style="display:none"></div>
  </div>

  <!-- Error -->
  <div id="error-wrap"></div>

  <!-- Stats Bar -->
  <div id="stats-bar">
    <div class="stat-cell"><div class="stat-label">Opportunities</div><div class="stat-value" id="stat-opps">0</div></div>
    <div class="stat-cell"><div class="stat-label">Best Net Profit</div><div class="stat-value" id="stat-best">—</div></div>
    <div class="stat-cell"><div class="stat-label">Best Ann. ROI</div><div class="stat-value" id="stat-roi">—</div></div>
    <div class="stat-cell"><div class="stat-label">Expirations Scanned</div><div class="stat-value" id="stat-exp">0</div></div>
    <div class="stat-cell"><div class="stat-label">Total Net Profit</div><div class="stat-value" id="stat-total">—</div></div>
  </div>

  <!-- Results Table -->
  <div id="results-wrap">
    <div class="results-header">
      <span class="results-title">PROFITABLE BOX SPREADS — CLICK ROW FOR LEG DETAIL</span>
      <div class="sort-group">
        <span class="sort-label">SORT:</span>
        <button class="sort-btn active" data-sort="netProfit">Net $</button>
        <button class="sort-btn" data-sort="annualizedRoi">Ann ROI</button>
        <button class="sort-btn" data-sort="dte">DTE</button>
        <button class="sort-btn" data-sort="width">Width</button>
      </div>
    </div>
    <div class="table-scroll">
      <table>
        <thead>
          <tr>
            <th>Expiry</th>
            <th>Strikes</th>
            <th class="r sortable active" data-sort="dte">DTE ▼</th>
            <th class="r">Cost/Ctrct</th>
            <th class="r">Theor. Value</th>
            <th class="r sortable active" data-sort="netProfit">Net Profit</th>
            <th class="r">Contracts</th>
            <th class="r sortable" data-sort="annualizedRoi">Ann. ROI</th>
          </tr>
        </thead>
        <tbody id="results-tbody"></tbody>
      </table>
    </div>
  </div>

  <!-- Empty -->
  <div id="empty">
    <div class="empty-icon">⬛</div>
    <div class="empty-title">No scan results yet</div>
    <div class="empty-body">
      Configure your parameters above and press
      <span style="color:var(--green);font-family:var(--mono)">▶ SCAN</span>
      to search for profitable box spreads. Every strike pair across all expirations is checked.
    </div>
    <div class="sym-chips">
      <button class="sym-chip active" onclick="setSymbol('SPY')">SPY</button>
      <button class="sym-chip" onclick="setSymbol('QQQ')">QQQ</button>
      <button class="sym-chip" onclick="setSymbol('SPX')">SPX</button>
      <button class="sym-chip" onclick="setSymbol('NVDA')">NVDA</button>
      <button class="sym-chip" onclick="setSymbol('AAPL')">AAPL</button>
      <button class="sym-chip" onclick="setSymbol('IWM')">IWM</button>
    </div>
  </div>

  <!-- Footer -->
  <div id="footer">
    <strong style="color:var(--text-dim)">Risk Notice:</strong>
    Box spreads in <strong style="color:var(--text-dim)">European-style index options</strong> (SPX, XSP) are preferred
    — they cannot be early-assigned, making the position truly risk-free at expiry.
    American-style options (SPY, QQQ, single stocks) carry early-assignment risk.
    Profitable opportunities after fees are rare in liquid markets and close quickly.
    Always verify NBBO prices with your broker before entry. This tool is for research only, not investment advice.
  </div>
</div>

<!-- ── Detail Modal ── -->
<div id="modal-overlay" onclick="closeModal()">
  <div id="modal" onclick="event.stopPropagation()">
    <div id="modal-head">
      <div>
        <div id="modal-title"></div>
        <div id="modal-sub"></div>
      </div>
      <button id="modal-close" onclick="closeModal()">✕</button>
    </div>
    <div id="modal-body">
      <div class="metric-grid" id="modal-metrics"></div>
      <div class="legs-section">
        <div class="legs-title">Execution Legs</div>
        <div id="modal-legs"></div>
      </div>
      <div class="modal-disclaimer">
        ⚠ Prefer European-style index options (SPX, XSP) to avoid early-assignment risk on short legs.
        Verify all prices with your broker before entry. Box spread P&L