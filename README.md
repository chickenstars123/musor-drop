<!doctype html>
<html lang="nl">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Mango Klikker — Upgrade Edition</title>
  <style>
    :root{--bg:#fff6d8;--card:#fffdf5;--accent:#ffcc4d;--text:#3f3218}
    *{box-sizing:border-box}
    body {
      margin: 0;
      min-height: 100vh;
      display: grid;
      place-items: center;
      font-family: system-ui, sans-serif;
      background: var(--bg);
      color: var(--text);
      text-align: center;
      padding: 20px;
    }
    .container{
      width: min(96%, 980px);
      display: grid;
      grid-template-columns: 1fr 340px;
      gap: 20px;
      align-items: start;
    }
    main {
      padding: 20px;
      border-radius: 16px;
      background: var(--card);
      box-shadow: 0 8px 30px #68460022;
    }
    header{display:flex;align-items:center;gap:16px}
    h1{margin:0;font-size:1.6rem}
    #mango {
      display: inline-block;
      border: 0;
      background: transparent;
      font-size: 110px;
      cursor: pointer;
      transition: transform 0.08s;
      line-height:1;
    }
    #mango:active { transform: scale(.9); }
    .score { font-size: 1.3rem; margin:8px 0 }
    .panel{background:var(--card);padding:16px;border-radius:12px}
    .shop button{
      width: 100%;
      margin: 7px 0;
      padding: 10px;
      border: 0;
      border-radius: 10px;
      background: var(--accent);
      color: var(--text);
      font-size: 0.95rem;
      cursor: pointer;
    }
    .shop button:disabled{opacity:.5;cursor:not-allowed}
    .row{display:flex;gap:10px}
    .col{display:flex;flex-direction:column;gap:8px}
    small{color:#70664f}
    .right {position:sticky;top:20px}
    .muted{color:#6b5b44;font-size:.95rem}
    .stat{display:flex;justify-content:space-between;align-items:center;padding:6px 0}
    .separator{height:1px;background:#efe5c7;margin:12px 0;border-radius:2px}
    .buy-inline{display:flex;gap:8px}
    .tiny{font-size:.9rem}
  </style>
</head>
<body>
  <div class="container">
    <main>
      <header>
        <button id="mango" aria-label="Klik op de mango">🥭</button>
        <div>
          <h1>🥭 Mango Klikker — Upgrade Edition</h1>
          <div class="muted">Klik, bouw farms, en ontdek nieuwe valuta &amp; upgrades</div>
        </div>
      </header>

      <section class="panel" style="margin-top:12px">
        <p class="score">Punten: <strong id="points">0</strong></p>
        <p class="score">Zaden: <strong id="seeds">0</strong> <small>(alternatieve valuta)</small></p>
        <div class="stat"><span>Per klik</span><strong id="perClick">1</strong></div>
        <div class="stat"><span>Per seconde (punten)</span><strong id="perSecond">0</strong></div>
        <div class="stat"><span>Per seconde (zaden)</span><strong id="seedPerSecond">0</strong></div>
        <div class="separator"></div>
        <div class="muted tiny">Tips: Farms produceren zaden. Gebruik zaden voor speciale upgrades of ruil ze voor punten.</div>
      </section>

      <section class="panel" style="margin-top:12px">
        <h2 style="margin:0 0 8px 0">Acties</h2>
        <div class="row">
          <button id="sellSeeds">Ruil 10 zaden → 50 punten</button>
          <button id="prestige">Prestige (reset voor bonus)</button>
        </div>
        <div class="muted tiny" style="margin-top:8px">Prestige: zet punten terug naar 0 maar ontvang een permanente +0.5% punten boost per 1000 punten (example)</div>
      </section>

      <section class="panel shop" style="margin-top:12px">
        <h2 style="margin:0 0 8px 0">Upgrades (Punten)</h2>
        <button id="multiplier"></button>
        <button id="auto"></button>
        <button id="factory"></button>
        <button id="booster"></button>
      </section>

      <section class="panel shop" style="margin-top:12px">
        <h2 style="margin:0 0 8px 0">Upgrades (Zaden / Farm)</h2>
        <button id="farm"></button>
        <button id="greenhouse"></button>
        <button id="seedTrader"></button>
      </section>

    </main>

    <aside class="right">
      <div class="panel">
        <h3 style="margin:0 0 8px 0">Status</h3>
        <div class="stat"><span>Points</span><span id="pointsSmall">0</span></div>
        <div class="stat"><span>Seeds</span><span id="seedsSmall">0</span></div>
        <div class="stat"><span>Prestige lvl</span><span id="prestigeLevel">0</span></div>
        <div class="separator"></div>
        <div class="muted tiny">Auto-buy: <label><input type="checkbox" id="autobuy"> Aan</label></div>
      </div>
      <div class="panel" style="margin-top:12px">
        <h3 style="margin:0 0 8px 0">Legenda</h3>
        <div class="tiny">Factory: converteert zaden naar punten automatisch.<br>Booster: tijdelijke x2 opbrengst.</div>
      </div>
    </aside>
  </div>

  <script>
    // State variables (persisted)
    let points = Number(localStorage.getItem('mangoPoints')) || 0;
    let seeds = Number(localStorage.getItem('mangoSeeds')) || 0;
    let perClick = Number(localStorage.getItem('mangoPerClick')) || 1;
    let perSecond = Number(localStorage.getItem('mangoPerSecond')) || 0; // points/sec
    let seedPerSecond = Number(localStorage.getItem('mangoSeedPerSecond')) || 0; // seeds/sec

    // Upgrade costs and counts
    let multiplierCost = Number(localStorage.getItem('mangoMultiplierCost')) || 10;
    let autoCost = Number(localStorage.getItem('mangoAutoCost')) || 25;
    let factoryCost = Number(localStorage.getItem('mangoFactoryCost')) || 100;
    let boosterCost = Number(localStorage.getItem('mangoBoosterCost')) || 200;

    let farmCost = Number(localStorage.getItem('mangoFarmCost')) || 50;
    let greenhouseCost = Number(localStorage.getItem('mangoGreenhouseCost')) || 300;
    let seedTraderCost = Number(localStorage.getItem('mangoSeedTraderCost')) || 150;

    let factoryLevel = Number(localStorage.getItem('mangoFactoryLevel')) || 0;
    let prestigeLevel = Number(localStorage.getItem('mangoPrestige')) || 0;

    // special
    let boosterActive = false;
    let boosterEndsAt = 0;
    let autobuy = (localStorage.getItem('mangoAutobuy') === 'true');

    // DOM refs
    const pointsEl = document.querySelector('#points');
    const seedsEl = document.querySelector('#seeds');
    const perClickEl = document.querySelector('#perClick');
    const perSecondEl = document.querySelector('#perSecond');
    const seedPerSecondEl = document.querySelector('#seedPerSecond');
    const multiplierButton = document.querySelector('#multiplier');
    const autoButton = document.querySelector('#auto');
    const factoryButton = document.querySelector('#factory');
    const boosterButton = document.querySelector('#booster');
    const farmButton = document.querySelector('#farm');
    const greenhouseButton = document.querySelector('#greenhouse');
    const seedTraderButton = document.querySelector('#seedTrader');
    const sellSeedsBtn = document.querySelector('#sellSeeds');
    const prestigeBtn = document.querySelector('#prestige');
    const pointsSmall = document.querySelector('#pointsSmall');
    const seedsSmall = document.querySelector('#seedsSmall');
    const prestigeLevelEl = document.querySelector('#prestigeLevel');
    const autobuyCheckbox = document.querySelector('#autobuy');

    // Save state
    function save(){
      localStorage.setItem('mangoPoints', points);
      localStorage.setItem('mangoSeeds', seeds);
      localStorage.setItem('mangoPerClick', perClick);
      localStorage.setItem('mangoPerSecond', perSecond);
      localStorage.setItem('mangoSeedPerSecond', seedPerSecond);
      localStorage.setItem('mangoMultiplierCost', multiplierCost);
      localStorage.setItem('mangoAutoCost', autoCost);
      localStorage.setItem('mangoFactoryCost', factoryCost);
      localStorage.setItem('mangoBoosterCost', boosterCost);
      localStorage.setItem('mangoFarmCost', farmCost);
      localStorage.setItem('mangoGreenhouseCost', greenhouseCost);
      localStorage.setItem('mangoSeedTraderCost', seedTraderCost);
      localStorage.setItem('mangoFactoryLevel', factoryLevel);
      localStorage.setItem('mangoPrestige', prestigeLevel);
      localStorage.setItem('mangoAutobuy', autobuy);
      localStorage.setItem('mangoBoosterEndsAt', boosterEndsAt);
    }

    // UI update
    function format(n){return Math.floor(n)}
    function update(){
      pointsEl.textContent = format(points);
      seedsEl.textContent = format(seeds);
      perClickEl.textContent = perClick.toFixed(1).replace(/\.0$/,'');
      perSecondEl.textContent = perSecond;
      seedPerSecondEl.textContent = seedPerSecond;

      multiplierButton.textContent = `Klik-multiplier: +1 per klik — ${multiplierCost} punten`;
      autoButton.textContent = `Mango-machine: +1 pt/s — ${autoCost} punten`;
      factoryButton.textContent = `Factory: converteer zaden → punten, lvl ${factoryLevel} — ${factoryCost} punten`;
      boosterButton.textContent = `Booster: x2 opbrengst (30s) — ${boosterCost} punten`;

      farmButton.textContent = `Farm: +1 zaad/s — ${farmCost} punten`;
      greenhouseButton.textContent = `Greenhouse: +5 zaad/s — ${greenhouseCost} punten`;
      seedTraderButton.textContent = `Seed Trader: ruil 20 zaden → 150 punten — ${seedTraderCost} zaden`;

      multiplierButton.disabled = points < multiplierCost;
      autoButton.disabled = points < autoCost;
      factoryButton.disabled = points < factoryCost;
      boosterButton.disabled = points < boosterCost || boosterActive;
      farmButton.disabled = points < farmCost;
      greenhouseButton.disabled = points < greenhouseCost;
      seedTraderButton.disabled = seeds < seedTraderCost;
      sellSeedsBtn.disabled = seeds < 10;

      pointsSmall.textContent = format(points);
      seedsSmall.textContent = format(seeds);
      prestigeLevelEl.textContent = prestigeLevel;

      autobuyCheckbox.checked = autobuy;
      save();
    }

    // Core interactions
    document.querySelector('#mango').addEventListener('click', ()=>{
      let gain = perClick;
      if (boosterActive) gain *= 2;
      // prestige bonus (small percent per level)
      gain *= (1 + prestigeLevel * 0.005);
      points += gain;
      update();
    });

    multiplierButton.addEventListener('click', ()=>{
      if (points < multiplierCost) return;
      points -= multiplierCost;
      perClick += 1;
      multiplierCost = Math.ceil(multiplierCost * 1.6);
      update();
    });

    autoButton.addEventListener('click', ()=>{
      if (points < autoCost) return;
      points -= autoCost;
      perSecond += 1;
      autoCost = Math.ceil(autoCost * 1.7);
      update();
    });

    factoryButton.addEventListener('click', ()=>{
      if (points < factoryCost) return;
      points -= factoryCost;
      factoryLevel += 1;
      // each factory level converts seeds -> points passively
      factoryCost = Math.ceil(factoryCost * 2);
      update();
    });

    boosterButton.addEventListener('click', ()=>{
      if (points < boosterCost || boosterActive) return;
      points -= boosterCost;
      boosterActive = true;
      boosterEndsAt = Date.now() + 30_000; // 30s booster
      setTimeout(()=>{ boosterActive=false; update(); }, 30_000);
      boosterCost = Math.ceil(boosterCost * 2.5);
      update();
    });

    farmButton.addEventListener('click', ()=>{
      if (points < farmCost) return;
      points -= farmCost;
      seedPerSecond += 1;
      farmCost = Math.ceil(farmCost * 1.9);
      update();
    });

    greenhouseButton.addEventListener('click', ()=>{
      if (points < greenhouseCost) return;
      points -= greenhouseCost;
      seedPerSecond += 5;
      greenhouseCost = Math.ceil(greenhouseCost * 2.4);
      update();
    });

    seedTraderButton.addEventListener('click', ()=>{
      if (seeds < seedTraderCost) return;
      seeds -= seedTraderCost;
      points += 150;
      seedTraderCost = Math.ceil(seedTraderCost * 1.6);
      update();
    });

    sellSeedsBtn.addEventListener('click', ()=>{
      if (seeds < 10) return;
      seeds -= 10;
      points += 50;
      update();
    });

    prestigeBtn.addEventListener('click', ()=>{
      // simple prestige: require 1000 points to prestige once
      if (points < 1000) return alert('Je hebt minstens 1000 punten nodig om te prestigeren.');
      // calculate bonus levels
      const gained = Math.floor(points / 1000);
      prestigeLevel += gained;
      // reset soft
      points = 0;
      seeds = 0;
      perClick = 1;
      perSecond = 0;
      seedPerSecond = 0;
      multiplierCost = 10; autoCost = 25; factoryCost = 100; boosterCost = 200;
      farmCost = 50; greenhouseCost = 300; seedTraderCost = 150; factoryLevel = 0;
      update();
      alert(`Je hebt ${gained} prestige level(s) verdiend!`);
    });

    // autobuy
    autobuyCheckbox.addEventListener('change', ()=>{ autobuy = autobuyCheckbox.checked; update(); });

    // Passive loops
    // every second: add perSecond points and seedPerSecond seeds, plus factory conversions
    setInterval(()=>{
      let pGain = perSecond;
      if (boosterActive) pGain *= 2;
      pGain *= (1 + prestigeLevel * 0.005);
      points += pGain;

      let sGain = seedPerSecond;
      seeds += sGain;

      // factory converts seeds -> points passively: each level converts 0.5*level seeds per sec into points at rate 10 points per seed
      if (factoryLevel > 0){
        const seedsToConvert = Math.min(seeds, 0.5 * factoryLevel);
        if (seedsToConvert > 0){
          seeds -= seedsToConvert;
          points += seedsToConvert * 10;
        }
      }

      // autobuy: prioritize cheap upgrades
      if (autobuy){
        tryAutobuy();
      }

      update();
    }, 1000);

    function tryAutobuy(){
      // order: farm, auto, multiplier, factory, booster if available, green house
      if (points >= farmCost) { points -= farmCost; seedPerSecond += 1; farmCost = Math.ceil(farmCost * 1.9); }
      else if (points >= autoCost) { points -= autoCost; perSecond += 1; autoCost = Math.ceil(autoCost * 1.7); }
      else if (points >= multiplierCost) { points -= multiplierCost; perClick += 1; multiplierCost = Math.ceil(multiplierCost * 1.6); }
      else if (points >= factoryCost) { points -= factoryCost; factoryLevel += 1; factoryCost = Math.ceil(factoryCost * 2); }
      else if (!boosterActive && points >= boosterCost) { points -= boosterCost; boosterActive=true; boosterEndsAt = Date.now()+30000; setTimeout(()=>{boosterActive=false;update();},30000); boosterCost = Math.ceil(boosterCost * 2.5); }
      else if (points >= greenhouseCost) { points -= greenhouseCost; seedPerSecond += 5; greenhouseCost = Math.ceil(greenhouseCost * 2.4); }
    }

    // load booster state
    (function restoreBooster(){
      const savedEnd = Number(localStorage.getItem('mangoBoosterEndsAt')) || 0;
      if (savedEnd > Date.now()){
        boosterActive = true; boosterEndsAt = savedEnd;
        const remaining = savedEnd - Date.now();
        setTimeout(()=>{ boosterActive=false; update(); }, remaining);
      }
    })();

    // initial update
    update();

    // expose quick reset (for testing)
    window.__mangoReset = function(){
      localStorage.clear(); location.reload();
    }
  </script>
</body>
</html>
