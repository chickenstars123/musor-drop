<!doctype html>
<html lang="nl">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Mango Klikker</title>
  <style>
    body {
      margin: 0;
      min-height: 100vh;
      display: grid;
      place-items: center;
      font-family: system-ui, sans-serif;
      background: #fff6d8;
      color: #3f3218;
      text-align: center;
    }
    main {
      width: min(90%, 420px);
      padding: 24px;
      border-radius: 24px;
      background: #fffdf5;
      box-shadow: 0 8px 30px #68460022;
    }
    h1 { margin-top: 0; }
    #mango {
      display: block;
      margin: 12px auto;
      border: 0;
      background: transparent;
      font-size: 110px;
      cursor: pointer;
      transition: transform 0.08s;
    }
    #mango:active { transform: scale(.9); }
    .score { font-size: 1.3rem; }
    .shop button {
      width: 100%;
      margin: 7px 0;
      padding: 12px;
      border: 0;
      border-radius: 12px;
      background: #ffcc4d;
      color: #3f3218;
      font-size: 1rem;
      cursor: pointer;
    }
    .shop button:disabled {
      opacity: .5;
      cursor: not-allowed;
    }
    small { color: #70664f; }
  </style>
</head>
<body>
  <main>
    <h1>🥭 Mango Klikker</h1>
    <p>Klik op de mango en verzamel punten!</p>

    <button id="mango" aria-label="Klik op de mango">🥭</button>

    <p class="score">Punten: <strong id="points">0</strong></p>
    <p>Per klik: <strong id="perClick">1</strong> punt</p>
    <p>Automatisch: <strong id="perSecond">0</strong> punten per seconde</p>

    <section class="shop">
      <h2>Upgrades</h2>
      <button id="multiplier"></button>
      <button id="auto"></button>
    </section>
    <small>Je punten blijven bewaard op dit apparaat.</small>
  </main>

  <script>
    let points = Number(localStorage.getItem("mangoPoints")) || 0;
    let perClick = Number(localStorage.getItem("mangoPerClick")) || 1;
    let perSecond = Number(localStorage.getItem("mangoPerSecond")) || 0;
    let multiplierCost = Number(localStorage.getItem("mangoMultiplierCost")) || 10;
    let autoCost = Number(localStorage.getItem("mangoAutoCost")) || 25;

    const pointsEl = document.querySelector("#points");
    const perClickEl = document.querySelector("#perClick");
    const perSecondEl = document.querySelector("#perSecond");
    const multiplierButton = document.querySelector("#multiplier");
    const autoButton = document.querySelector("#auto");

    function save() {
      localStorage.setItem("mangoPoints", points);
      localStorage.setItem("mangoPerClick", perClick);
      localStorage.setItem("mangoPerSecond", perSecond);
      localStorage.setItem("mangoMultiplierCost", multiplierCost);
      localStorage.setItem("mangoAutoCost", autoCost);
    }

    function update() {
      pointsEl.textContent = Math.floor(points);
      perClickEl.textContent = perClick;
      perSecondEl.textContent = perSecond;

      multiplierButton.textContent =
        `Klik-multiplier: +1 punt per klik — ${multiplierCost} punten`;
      autoButton.textContent =
        `Mango-machine: +1 punt per seconde — ${autoCost} punten`;

      multiplierButton.disabled = points < multiplierCost;
      autoButton.disabled = points < autoCost;
      save();
    }

    document.querySelector("#mango").addEventListener("click", () => {
      points += perClick;
      update();
    });

    multiplierButton.addEventListener("click", () => {
      if (points < multiplierCost) return;
      points -= multiplierCost;
      perClick += 1;
      multiplierCost = Math.ceil(multiplierCost * 1.7);
      update();
    });

    autoButton.addEventListener("click", () => {
      if (points < autoCost) return;
      points -= autoCost;
      perSecond += 1;
      autoCost = Math.ceil(autoCost * 1.8);
      update();
    });

    setInterval(() => {
      points += perSecond;
      update();
    }, 1000);

    update();
  </script>
</body>
</html>
