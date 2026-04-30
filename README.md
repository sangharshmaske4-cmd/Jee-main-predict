<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JEE Predictor Pro | Sangharsh Maske</title>
    <style>
        :root { --primary: #0f172a; --accent: #2563eb; --success: #10b981; --warning: #f59e0b; --bg: #f8fafc; }
        body { font-family: 'Inter', sans-serif; background: var(--bg); display: flex; justify-content: center; padding: 20px; }
        .container { background: white; padding: 30px; border-radius: 20px; width: 100%; max-width: 550px; box-shadow: 0 10px 30px rgba(0,0,0,0.08); }
        .brand { text-align: center; font-family: monospace; letter-spacing: 5px; font-weight: 900; margin-bottom: 10px; }
        .header { text-align: center; margin-bottom: 25px; }
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 20px; }
        label { font-size: 0.7rem; font-weight: 800; color: #64748b; text-transform: uppercase; }
        input, select { width: 100%; padding: 12px; border-radius: 10px; border: 2px solid #e2e8f0; box-sizing: border-box; }
        button { width: 100%; padding: 16px; background: var(--primary); color: white; border: none; border-radius: 12px; font-weight: 700; cursor: pointer; margin-top: 10px; }
        .stats { display: flex; gap: 10px; margin-top: 25px; display: none; }
        .stat-card { flex: 1; background: var(--primary); color: white; padding: 15px; border-radius: 12px; text-align: center; }
        .college-card { border: 1px solid #e2e8f0; padding: 15px; border-radius: 12px; margin-top: 10px; border-left: 5px solid #cbd5e1; }
        .tag { font-size: 0.6rem; font-weight: 900; padding: 3px 7px; border-radius: 4px; text-transform: uppercase; margin-bottom: 5px; display: inline-block; }
        .vsafe { background: #dcfce7; color: #15803d; }
        .csab { background: #fef3c7; color: #b45309; }
    </style>
</head>
<body>

<div class="container">
    <div class="brand">SANGHARSH MASKE</div>
    <div class="header">
        <h1 style="margin:0; font-size: 1.5rem;">Predictor Pro v4.5</h1>
        <p style="color:#64748b; margin:5px 0;">Strategic Seat Simulation 2026</p>
    </div>

    <div class="grid">
        <div style="grid-column: span 2;">
            <label>Marks (0-300)</label>
            <input type="number" id="marks" placeholder="Enter NTA Score">
        </div>
        <div>
            <label>Category</label>
            <select id="cat">
                <option value="GEN">General</option>
                <option value="OBC">OBC-NCL</option>
                <option value="SC">SC</option>
                <option value="ST">ST</option>
            </select>
        </div>
        <div>
            <label>Home State</label>
            <select id="state">
                <option value="Maharashtra">Maharashtra</option>
                <option value="Delhi">Delhi</option>
                <option value="Other">Other State</option>
            </select>
        </div>
    </div>

    <button onclick="calculate()">Simulate Allocation</button>

    <div class="stats" id="statPanel">
        <div class="stat-card">
            <span style="font-size:0.6rem; opacity:0.8;">PERCENTILE</span>
            <div id="resP" style="font-size:1.2rem; font-weight:900;">-</div>
        </div>
        <div class="stat-card">
            <span style="font-size:0.6rem; opacity:0.8;">CRL RANK</span>
            <div id="resR" style="font-size:1.2rem; font-weight:900;">-</div>
        </div>
    </div>

    <div id="results" style="margin-top:20px;"></div>
</div>

<script>
    // Expanded DB for High Rank Coverage (GFTIs included)
    const db = [
        { n: "NIT Trichy", b: "CSE", s: "Tamil Nadu", c: { GEN: {os: 1500, hs: 5000}, SC: {os: 450, hs: 1100} } },
        { n: "NIT Nagpur (VNIT)", b: "Electronics", s: "Maharashtra", c: { GEN: {os: 9500, hs: 12500}, SC: {os: 1600, hs: 3200} } },
        { n: "NIT Nagaland", b: "Civil", s: "Nagaland", c: { GEN: {os: 55000, hs: 450000}, SC: {os: 8000, hs: 95000} } },
        { n: "GFTI Mizoram", b: "ECE", s: "Mizoram", c: { GEN: {os: 85000, hs: 750000}, SC: {os: 15000, hs: 175000} } },
        { n: "RCOEM Nagpur", b: "CSE/IT", s: "Maharashtra", c: { GEN: {os: 40000, hs: 55000}, SC: {os: 45000, hs: 320000} } },
        { n: "GKV Haridwar", b: "CSE", s: "Uttarakhand", c: { GEN: {os: 95000, hs: 95000}, SC: {os: 18000, hs: 18000} } }
    ];

    function calculate() {
        const m = parseFloat(document.getElementById('marks').value);
        const cat = document.getElementById('cat').value;
        const hState = document.getElementById('state').value;

        if(!m && m !== 0) return;

        // Corrected 90 Marks Anchor
        let p = (m >= 45) ? (73.5 + (m - 45) * 0.4).toFixed(2) : (m * 1.63).toFixed(2);
        if(m >= 100) p = 95.8; 
        
        let crl = Math.round(((100 - p) * 1600000) / 100);

        document.getElementById('statPanel').style.display = 'flex';
        document.getElementById('resP').innerText = p + "%";
        document.getElementById('resR').innerText = crl.toLocaleString();

        let catRank = (cat === "SC") ? crl * 0.055 : (cat === "OBC") ? crl * 0.28 : crl;
        
        // BUFFER INCREASED for high ranks to fix blank results
        let buffer = (crl > 300000) ? 2.8 : 1.5;

        let html = "";
        db.forEach(col => {
            let q = (col.s === hState) ? 'hs' : 'os';
            let cut = col.c[cat]?.[q] || col.c['GEN'][q];
            let ratio = catRank / cut;

            if(ratio < buffer) {
                let safe = ratio <= 1.0;
                html += `
                    <div class="college-card" style="border-left-color:${safe ? 'var(--success)' : 'var(--warning)'}">
                        <span class="tag ${safe ? 'vsafe' : 'csab'}">${safe ? 'JoSAA Probable' : 'CSAB Targeted'}</span>
                        <div style="font-weight:800;">${col.n}</div>
                        <div style="font-size:0.8rem; color:#64748b;">${col.b} | Quota: ${q.toUpperCase()}</div>
                        <div style="font-size:0.75rem; margin-top:5px; font-weight:700; color:var(--accent)">Cutoff: ${cut.toLocaleString()}</div>
                    </div>`;
            }
        });

        document.getElementById('results').innerHTML = html || `
            <div style="text-align:center; padding:20px; border:2px dashed #e2e8f0; border-radius:12px; color:#64748b;">
                <b>No direct NIT matches.</b><br>Focus on MHT-CET or CSAB Special Rounds for GFTIs.
            </div>`;
    }
</script>
</body>
</html>

