# index.html

<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, viewport-fit=cover">
<meta name="theme-color" content="#080b09">
<title>KGF LUDO</title>

<script src="https://telegram.org/js/telegram-web-app.js"></script>

<style>
:root {
  --bg: #080b09;
  --panel: #111713;
  --green: #39ff78;
  --muted: #a5b5a9;
  --red: #ff414b;
  --blue: #339cff;
  --yellow: #ffda42;
  --greenPlayer: #35ed78;
}

* { box-sizing: border-box; }

html {
  min-height: 100%;
  background: var(--bg);
}

body {
  margin: 0;
  min-height: 100vh;
  min-height: 100dvh;
  color: white;
  background:
    radial-gradient(ellipse at top, #14251a 0%, #080b09 58%);
  font-family: Tahoma, Arial, sans-serif;
}

button, select { font: inherit; }

.app {
  width: 100%;
  max-width: 620px;
  min-height: 100vh;
  min-height: 100dvh;
  margin: auto;
  padding: 12px 12px 0;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.header {
  text-align: center;
  padding: 8px 0 2px;
}

.logo {
  font-size: clamp(30px, 8vw, 42px);
  font-weight: 1000;
  letter-spacing: 5px;
  color: var(--green);
  text-shadow: 0 0 10px #39ff7866, 0 0 25px #39ff7833;
}

.subtitle {
  color: #b9c8bd;
  font-size: 11px;
  letter-spacing: 3px;
  margin-top: 3px;
}

.status-panel {
  background: #101612;
  border: 1px solid #263b2b;
  border-radius: 15px;
  padding: 10px 12px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  box-shadow: 0 5px 18px #0005;
}

.status-text {
  font-size: 13px;
  font-weight: bold;
  line-height: 1.8;
}

.status-text small {
  display: block;
  color: var(--muted);
  font-size: 10px;
  font-weight: normal;
}

.turn-dot {
  width: 11px;
  height: 11px;
  display: inline-block;
  border-radius: 50%;
  background: var(--green);
  margin-left: 6px;
  box-shadow: 0 0 12px currentColor;
}

.settings {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 11px;
  color: #cbd8ce;
  white-space: nowrap;
}

select {
  background: #19241c;
  color: var(--green);
  border: 1px solid #35563d;
  border-radius: 9px;
  padding: 7px 4px;
  outline: none;
}

.board-wrap {
  width: 100%;
  position: relative;
  border: 2px solid #2e5037;
  border-radius: 18px;
  padding: 5px;
  background: #0d120e;
  box-shadow: 0 0 25px #39ff7810;
}

canvas {
  display: block;
  width: 100%;
  height: auto;
  aspect-ratio: 1 / 1;
  border-radius: 11px;
  touch-action: manipulation;
}

.hint {
  min-height: 35px;
  text-align: center;
  color: #d8e8dc;
  font-size: 12px;
  padding: 7px 4px 2px;
  line-height: 1.8;
}

/* تاس همیشه بعد از صفحه بازی قرار دارد */
.dice-dock {
  position: sticky;
  bottom: 0;
  z-index: 20;
  margin-top: auto;
  padding: 12px 10px calc(12px + env(safe-area-inset-bottom));
  background: linear-gradient(180deg, #101712f5, #080b09 35%);
  border-top: 1px solid #2b4832;
  border-radius: 17px 17px 0 0;
  box-shadow: 0 -8px 24px #0007;
}

.dice-row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 14px;
}

.dice {
  width: 76px;
  height: 76px;
  flex: 0 0 76px;
  border-radius: 18px;
  border: 2px solid var(--green);
  background: #101b13;
  color: white;
  font-size: 44px;
  font-weight: 900;
  display: grid;
  place-items: center;
  box-shadow: 0 0 18px #39ff7830, inset 0 0 12px #39ff7810;
  cursor: pointer;
  user-select: none;
  -webkit-tap-highlight-color: transparent;
}

.dice:active { transform: scale(.94); }

.dice.rolling {
  animation: shake .42s ease-in-out;
}

@keyframes shake {
  0% { transform: rotate(0) scale(1); }
  20% { transform: rotate(-14deg) scale(1.08); }
  40% { transform: rotate(14deg) scale(1.08); }
  60% { transform: rotate(-10deg) scale(1.05); }
  80% { transform: rotate(8deg) scale(1.03); }
  100% { transform: rotate(0) scale(1); }
}

.roll-info {
  text-align: right;
  flex: 1;
  min-width: 0;
}

.roll-title {
  color: var(--green);
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 5px;
}

.roll-subtitle {
  color: #a9b9ad;
  font-size: 11px;
  line-height: 1.8;
}

.action-buttons {
  display: flex;
  gap: 8px;
  margin-top: 11px;
}

.action {
  flex: 1;
  padding: 10px 5px;
  border-radius: 10px;
  border: 1px solid #2c4432;
  color: #e7f4e9;
  background: #172019;
  font-size: 12px;
  cursor: pointer;
}

.action.primary {
  color: #061008;
  font-weight: bold;
  background: var(--green);
  border-color: var(--green);
}

.action:active { opacity: .8; }

.footer {
  text-align: center;
  color: #5f7765;
  font-size: 9px;
  padding: 5px 0 9px;
}

@media (max-height: 720px) {
  .app { gap: 8px; padding-top: 6px; }
  .header { padding-top: 2px; }
  .logo { font-size: 29px; }
  .dice { width: 62px; height: 62px; flex-basis: 62px; font-size: 36px; }
  .dice-dock { padding-top: 8px; }
}

@media (min-width: 600px) {
  .app { padding-left: 18px; padding-right: 18px; }
}
</style>
</head>

<body>
<div class="app">

  <header class="header">
    <div class="logo">KGF</div>
    <div class="subtitle">L U D O&nbsp; G A M E</div>
  </header>

  <section class="status-panel">
    <div class="status-text">
      <span class="turn-dot" id="turnDot"></span>
      <span id="status">نوبت بازیکن قرمز</span>
      <small id="subStatus">تاس را از پایین صفحه بزن</small>
    </div>

    <div class="settings">
      بازیکن
      <select id="players">
        <option value="2">۲ نفر</option>
        <option value="3">۳ نفر</option>
        <option value="4" selected>۴ نفر</option>
      </select>
    </div>
  </section>

  <!-- صفحه بازی -->
  <section class="board-wrap">
    <canvas id="board" width="600" height="600"></canvas>
  </section>

  <div class="hint" id="hint">
    برای شروع، تاس پایین صفحه را لمس کن 🎲
  </div>

  <!-- تاس در پایین گوشی -->
  <section class="dice-dock">
    <div class="dice-row">
      <button class="dice" id="rollBtn" aria-label="انداختن تاس">🎲</button>
      <div class="roll-info">
        <div class="roll-title">نوبت توئه!</div>
        <div class="roll-subtitle" id="diceInfo">
          تاس را بزن و بازی را شروع کن
        </div>
      </div>
    </div>

    <div class="action-buttons">
      <button class="action primary" id="newGameBtn">بازی جدید</button>
      <button class="action" id="soundBtn">🔊 صدا روشن</button>
    </div>
  </section>

  <footer class="footer">KGF LUDO · PLAY WITH FRIENDS</footer>
</div>

<script>
(() => {
  const tg = window.Telegram?.WebApp;
  if (tg) {
    tg.ready();
    tg.expand();
  }

  const canvas = document.getElementById("board");
  const ctx = canvas.getContext("2d");
  const rollBtn = document.getElementById("rollBtn");
  const statusEl = document.getElementById("status");
  const subStatusEl = document.getElementById("subStatus");
  const hintEl = document.getElementById("hint");
  const diceInfo = document.getElementById("diceInfo");
  const playersSelect = document.getElementById("players");
  const turnDot = document.getElementById("turnDot");
  const soundBtn = document.getElementById("soundBtn");

  const SIZE = 600;
  const COLORS = [
    { name: "قرمز", hex: "#ff414b", light: "#ff9ba0", base: "#561c25" },
    { name: "سبز", hex: "#35ed78", light: "#a4ffc2", base: "#16472b" },
    { name: "زرد", hex: "#ffda42", light: "#fff0a0", base: "#594817" },
    { name: "آبی", hex: "#339cff", light: "#a5d5ff", base: "#153b62" }
  ];

  /*
   * مسیر حلقه‌ای ۵۲ خانه‌ای برای نسخه نمایشی.
   * این نسخه برای آزمایش رابط و حرکت پایه‌ای است؛
   * هنوز تمام قوانین رسمی منچ را پیاده‌سازی نمی‌کند.
   */
  const TRACK = [];
  const cx = 300, cy = 300, rx = 207, ry = 207;

  for (let i = 0; i < 52; i++) {
    const angle = (-Math.PI / 2) + (i / 52) * Math.PI * 2;
    TRACK.push({
      x: cx + Math.cos(angle) * rx,
      y: cy + Math.sin(angle) * ry
    });
  }

  const BASES = [
    { x: 22, y: 22, w: 205, h: 205 },
    { x: 373, y: 22, w: 205, h: 205 },
    { x: 373, y: 373, w: 205, h: 205 },
    { x: 22, y: 373, w: 205, h: 205 }
  ];

  let playerCount = 4;
  let currentPlayer = 0;
  let diceValue = 0;
  let awaitingMove = false;
  let soundOn = true;
  let gameOver = false;
  let rolling = false;

  // position=-1 یعنی مهره هنوز وارد مسیر نشده است.
  let pawns = [];

  function initGame() {
    playerCount = Number(playersSelect.value);
    currentPlayer = 0;
    diceValue = 0;
    awaitingMove = false;
    gameOver = false;
    rolling = false;

    pawns = COLORS.map(() =>
      Array.from({ length: 4 }, () => ({ position: -1 }))
    );

    updateStatus();
    diceInfo.textContent = "تاس را بزن و بازی را شروع کن";
    hintEl.textContent = "برای شروع، تاس پایین صفحه را لمس کن 🎲";
    rollBtn.textContent = "🎲";
    drawBoard();
  }

  function beep(freq = 500, duration = 70) {
    if (!soundOn) return;
    try {
      const AudioCtx = window.AudioContext || window.webkitAudioContext;
      if (!AudioCtx) return;
      const audio = new AudioCtx();
      const oscillator = audio.createOscillator();
      const gain = audio.createGain();
      oscillator.frequency.value = freq;
      oscillator.type = "sine";
      gain.gain.value = 0.05;
      oscillator.connect(gain);
      gain.connect(audio.destination);
      oscillator.start();
      oscillator.stop(audio.currentTime + duration / 1000);
      oscillator.onended = () => audio.close();
    } catch (_) {}
  }

  function updateStatus() {
    const color = COLORS[currentPlayer];
    statusEl.textContent = "نوبت بازیکن " + color.name;
    turnDot.style.background = color.hex;
    turnDot.style.color = color.hex;
    turnDot.style.boxShadow = "0 0 12px " + color.hex;
    subStatusEl.textContent = gameOver
      ? "بازی تمام شد"
      : awaitingMove
        ? "یکی از مهره‌های خودت را انتخاب کن"
        : "تاس را از پایین صفحه بزن";
  }

  function roundedRect(x, y, w, h, r, fill, stroke) {
    ctx.beginPath();
    ctx.roundRect(x, y, w, h, r);
    if (fill) {
      ctx.fillStyle = fill;
      ctx.fill();
    }
    if (stroke) {
      ctx.strokeStyle = stroke;
      ctx.lineWidth = 2;
      ctx.stroke();
    }
  }

  function drawPawn(x, y, color, active = false) {
    ctx.save();

    ctx.beginPath();
    ctx.arc(x, y + 2, 13, 0, Math.PI * 2);
    ctx.fillStyle = "#0007";
    ctx.fill();

    ctx.beginPath();
    ctx.arc(x, y, 13, 0, Math.PI * 2);
    ctx.fillStyle = color.hex;
    ctx.fill();

    ctx.beginPath();
    ctx.arc(x - 4, y - 4, 4, 0, Math.PI * 2);
    ctx.fillStyle = color.light;
    ctx.fill();

    ctx.strokeStyle = active ? "#ffffff" : "#ffffffaa";
    ctx.lineWidth = active ? 3 : 1.2;
    ctx.stroke();

    if (active) {
      ctx.beginPath();
      ctx.arc(x, y, 18, 0, Math.PI * 2);
      ctx.strokeStyle = "#ffffff";
      ctx.lineWidth = 2;
      ctx.stroke();
    }

    ctx.restore();
  }

  function drawBoard() {
    ctx.clearRect(0, 0, SIZE, SIZE);

    // پس‌زمینه
    const bg = ctx.createLinearGradient(0, 0, 600, 600);
    bg.addColorStop(0, "#17251a");
    bg.addColorStop(1, "#090e0b");
    ctx.fillStyle = bg;
    ctx.fillRect(0, 0, SIZE, SIZE);

    // مسیر ۵۲ خانه‌ای
    TRACK.forEach((p, i) => {
      ctx.beginPath();
      ctx.arc(p.x, p.y, 14, 0, Math.PI * 2);
      ctx.fillStyle = "#dfe9df";
      ctx.fill();
      ctx.strokeStyle = "#8a9d8d";
      ctx.lineWidth = 1.5;
      ctx.stroke();

      if (i % 13 === 0) {
        ctx.beginPath();
        ctx.arc(p.x, p.y, 9, 0, Math.PI * 2);
        ctx.fillStyle = COLORS[Math.floor(i / 13)].hex;
        ctx.fill();
      }
    });

    // چهار پایگاه رنگی
    BASES.forEach((b, i) => {
      const color = COLORS[i];
      roundedRect(b.x, b.y, b.w, b.h, 18, color.base, color.hex);

      roundedRect(
        b.x + 25, b.y + 25, b.w - 50, b.h - 50,
        15, "#101611", color.hex
      );

      ctx.fillStyle = color.hex;
      ctx.font = "bold 17px Tahoma";
      ctx.textAlign = "center";
      ctx.fillText(color.name, b.x + b.w / 2, b.y + 17);

      const spots = [
        [b.x + 66, b.y + 68],
        [b.x + b.w - 66, b.y + 68],
        [b.x + 66, b.y + b.h - 66],
        [b.x + b.w - 66, b.y + b.h - 66]
      ];

      spots.forEach((spot, pawnIndex) => {
        const pawn = pawns[i][pawnIndex];
        if (pawn.position === -1) {
          drawPawn(spot[0], spot[1], color);
        }
      });
    });

    // مرکز صفحه
    ctx.beginPath();
    ctx.arc(300, 300, 46, 0, Math.PI * 2);
    ctx.fillStyle = "#101b13";
    ctx.fill();
    ctx.strokeStyle = "#39ff78";
    ctx.lineWidth = 3;
    ctx.stroke();

    ctx.textAlign = "center";
    ctx.textBaseline = "middle";
    ctx.font = "1000 20px Tahoma";
    ctx.fillStyle = "#39ff78";
    ctx.fillText("KGF", 300, 297);
    ctx.font = "bold 9px Tahoma";
    ctx.fillStyle = "#dcefe0";
    ctx.fillText("LUDO", 300, 318);

    // مهره‌های واردشده به مسیر
    pawns.forEach((group, playerIndex) => {
      group.forEach((pawn, pawnIndex) => {
        if (pawn.position >= 0) {
          const p = TRACK[pawn.position % TRACK.length];
          const offset = [
            [-8, -8], [8, -8], [-8, 8], [8, 8]
          ][pawnIndex];

          drawPawn(
            p.x + offset[0],
            p.y + offset[1],
            COLORS[playerIndex],
            awaitingMove && playerIndex === currentPlayer
          );
        }
      });
    });

    ctx.textBaseline = "alphabetic";
  }

  function rollDice() {
    if (rolling || awaitingMove || gameOver) return;

    rolling = true;
    rollBtn.classList.add("rolling");
    rollBtn.textContent = "🎲";
    beep(650, 70);

    setTimeout(() => {
      diceValue = Math.floor(Math.random() * 6) + 1;
      rolling = false;
      rollBtn.classList.remove("rolling");
      rollBtn.textContent = ["⚀", "⚁", "⚂", "⚃", "⚄", "⚅"][diceValue - 1];

      const movable = pawns[currentPlayer].some(p =>
        p.position >= 0 || diceValue === 6
      );

      diceInfo.textContent = "عدد تاس: " + diceValue;
      beep(420 + diceValue * 70, 100);

      if (!movable) {
        hintEl.textContent = diceValue === 6
          ? "یک مهره را انتخاب کن تا وارد مسیر شود"
          : "حرکت ممکن نیست؛ نوبت بعدی";
        awaitingMove = false;
        setTimeout(nextTurn, 850);
        return;
      }

      awaitingMove = true;
      updateStatus();
      hintEl.textContent = diceValue === 6
        ? "عدد ۶ آمد! یک مهره را لمس کن"
        : "یکی از مهره‌های خودت را انتخاب کن";
      drawBoard();
    }, 430);
  }

  function nextTurn() {
    if (gameOver) return;
    awaitingMove = false;
    diceValue = 0;
    currentPlayer = (currentPlayer + 1) % playerCount;
    updateStatus();
    diceInfo.textContent = "نوبت " + COLORS[currentPlayer].name;
    hintEl.textContent = "تاس را بزن 🎲";
    rollBtn.textContent = "🎲";
    drawBoard();
  }

  function handleBoardTap(event) {
    if (!awaitingMove || gameOver) return;

    const rect = canvas.getBoundingClientRect();
    const scaleX = SIZE / rect.width;
    const scaleY = SIZE / rect.height;
    const x = (event.clientX - rect.left) * scaleX;
    const y = (event.clientY - rect.top) * scaleY;

    const group = pawns[currentPlayer];
    let chosen = -1;
    let bestDistance = Infinity;

    group.forEach((pawn, i) => {
      let px, py;

      if (pawn.position === -1) {
        const b = BASES[currentPlayer];
        const spots = [
          [b.x + 66, b.y + 68],
          [b.x + b.w - 66, b.y + 68],
          [b.x + 66, b.y + b.h - 66],
          [b.x + b.w - 66, b.y + b.h - 66]
        ];
        [px, py] = spots[i];
      } else {
        const p = TRACK[pawn.position % TRACK.length];
        const offset = [[-8,-8],[8,-8],[-8,8],[8,8]][i];
        px = p.x + offset[0];
        py = p.y + offset[1];
      }

      const d = Math.hypot(x - px, y - py);
      if (d < 32 && d < bestDistance) {
        bestDistance = d;
        chosen = i;
      }
    });

    if (chosen === -1) {
      hintEl.textContent = "یکی از مهره‌های خودت را لمس کن";
      return;
    }

    const pawn = group[chosen];

    if (pawn.position === -1) {
      if (diceValue !== 6) {
        hintEl.textContent = "برای وارد کردن مهره به مسیر باید ۶ بیاوری";
        return;
      }
      pawn.position = currentPlayer * 13;
    } else {
      pawn.position = (pawn.position + diceValue) % TRACK.length;
    }

    beep(720, 90);
    awaitingMove = false;
    drawBoard();

    hintEl.textContent = "حرکت انجام شد!";
    updateStatus();

    if (diceValue === 6) {
      diceValue = 0;
      rollBtn.textContent = "🎲";
      diceInfo.textContent = "۶ آوردی! دوباره تاس بزن";
      hintEl.textContent = "۶ آوردی؛ دوباره نوبت توست!";
    } else {
      setTimeout(nextTurn, 600);
    }
  }

  rollBtn.addEventListener("click", rollDice);
  canvas.addEventListener("click", handleBoardTap);

  document.getElementById("newGameBtn").addEventListener("click", () => {
    initGame();
    beep(550, 90);
  });

  playersSelect.addEventListener("change", initGame);

  soundBtn.addEventListener("click", () => {
    soundOn = !soundOn;
    soundBtn.textContent = soundOn ? "🔊 صدا روشن" : "🔇 صدا خاموش";
    if (soundOn) beep();
  });

  initGame();
})();
</script>
</body>
</html>
