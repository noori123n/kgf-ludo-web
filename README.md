# index.html

<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<meta name="theme-color" content="#080d0a">
<meta name="mobile-web-app-capable" content="yes">
<title>KGF LUDO | بازی منچ</title>
<script src="https://telegram.org/js/telegram-web-app.js"></script>

<style>
:root {
  --bg: #080d0a;
  --panel: #111a14;
  --green: #39ff88;
  --muted: #91a398;
}
* { box-sizing: border-box; }
body {
  margin: 0;
  min-height: 100vh;
  color: white;
  background:
    radial-gradient(ellipse at top, #153323 0%, transparent 55%),
    var(--bg);
  font-family: system-ui, Tahoma, sans-serif;
  text-align: center;
}
header { padding: 20px 12px 12px; }
.logo {
  font-size: 30px;
  font-weight: 1000;
  font-style: italic;
  letter-spacing: 3px;
  color: var(--green);
  text-shadow: 0 0 20px #39ff8860;
}
.subtitle { color: var(--muted); font-size: 12px; margin-top: 4px; }
.wrap { width: min(94vw, 560px); margin: auto; }
.panel {
  background: #111a14eF;
  border: 1px solid #254532;
  border-radius: 18px;
  padding: 13px;
  margin: 10px 0;
  box-shadow: 0 8px 35px #0005;
}
.row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  flex-wrap: wrap;
}
label { color: #a9b9ae; font-size: 12px; }
select, button {
  font: inherit;
  border-radius: 11px;
  border: 1px solid #355541;
  background: #18251c;
  color: white;
  padding: 10px 12px;
}
button { cursor: pointer; }
button:active { transform: scale(.97); }
.primary {
  background: var(--green);
  color: #07140b;
  font-weight: 900;
  border: none;
  box-shadow: 0 0 18px #39ff8830;
}
#status {
  font-size: 15px;
  font-weight: 800;
  margin-bottom: 8px;
}
#hint { color: var(--muted); font-size: 12px; min-height: 20px; }
.dice {
  width: 76px;
  height: 76px;
  border-radius: 18px;
  background: linear-gradient(145deg,#fff,#dce8df);
  color: #111;
  font-size: 43px;
  font-weight: 900;
  border: 3px solid #fff;
  box-shadow: 0 5px 0 #87968b, 0 0 20px #39ff8830;
}
.dice:disabled { opacity: .65; cursor: default; }
.board-box {
  padding: 8px;
  background: linear-gradient(145deg,#17231a,#0b100d);
  border: 1px solid #34513d;
  border-radius: 20px;
  box-shadow: 0 0 35px #39ff8810;
}
canvas {
  display: block;
  width: 100%;
  height: auto;
  border-radius: 12px;
  touch-action: manipulation;
}
.players {
  display: grid;
  grid-template-columns: repeat(4,1fr);
  gap: 6px;
}
.player {
  padding: 9px 3px;
  border-radius: 12px;
  background: #0c130e;
  border: 1px solid #263b2c;
  font-size: 11px;
  color: #c4d2c8;
}
.player.active {
  border-color: var(--green);
  box-shadow: inset 0 0 12px #39ff8815;
  color: white;
}
.dot {
  display: block;
  width: 17px;
  height: 17px;
  border-radius: 50%;
  margin: 0 auto 5px;
  border: 2px solid #ffffffa0;
}
.footer { color: #637568; font-size: 10px; padding: 16px; }

/* تنها تغییر: پنل تاس در پایین صفحه گوشی ثابت می‌ماند */
main.wrap > section.panel:has(#rollBtn) {
  position: fixed;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: min(94vw, 560px);
  margin: 0;
  z-index: 9999;
  border-radius: 18px 18px 0 0;
  padding: 13px 13px calc(13px + env(safe-area-inset-bottom));
  background: #111a14f5;
  box-shadow: 0 -8px 25px #0008;
}

/* فضای پایین صفحه برای اینکه محتوا زیر پنل تاس نرود */
main.wrap {
  padding-bottom: 180px;
}
</style>
</head>

<body>
<header>
  <div class="logo">KGF LUDO</div>
  <div class="subtitle">YOUR GAME · YOUR RULES</div>
</header>

<main class="wrap">
  <section class="panel">
    <div class="row">
      <div style="text-align:right">
        <label for="playerCount">تعداد بازیکن‌ها</label><br>
        <select id="playerCount">
          <option value="2">۲ بازیکن</option>
          <option value="3">۳ بازیکن</option>
          <option value="4" selected>۴ بازیکن</option>
        </select>
      </div>
      <button id="soundBtn">🔊 صدا روشن</button>
      <button id="resetBtn">↻ شروع مجدد</button>
    </div>
  </section>

  <section class="panel">
    <div id="status">نوبت قرمز</div>
    <div class="row" style="justify-content:center">
      <button id="rollBtn" class="dice" aria-label="انداختن تاس">⚄</button>
    </div>
    <div id="hint">تاس را بزن تا بازی شروع شود 🎲</div>
  </section>

  <section class="board-box">
    <canvas id="board" width="600" height="600"></canvas>
  </section>

  <section class="panel">
    <div class="players" id="players"></div>
  </section>

  <div class="footer">KGF LUDO · نسخه آزمایشی محلی</div>
</main>

<script>
(() => {
  "use strict";

  const tg = window.Telegram?.WebApp;
  if (tg) {
    tg.ready();
    tg.expand();
    try { tg.setHeaderColor("#080d0a"); } catch (_) {}
    try { tg.setBackgroundColor("#080d0a"); } catch (_) {}
  }

  const canvas = document.getElementById("board");
  const ctx = canvas.getContext("2d");
  const rollBtn = document.getElementById("rollBtn");
  const hint = document.getElementById("hint");
  const status = document.getElementById("status");
  const countSelect = document.getElementById("playerCount");
  const playersBox = document.getElementById("players");
  const soundBtn = document.getElementById("soundBtn");

  const SIZE = 40;
  const COLORS = [
    { name:"قرمز", hex:"#ff4b55", light:"#ff9a9f", start:0 },
    { name:"سبز", hex:"#30df79", light:"#9affbb", start:13 },
    { name:"زرد", hex:"#ffd447", light:"#fff0a0", start:26 },
    { name:"آبی", hex:"#439bff", light:"#a3d0ff", start:39 }
  ];

  // مسیر حلقه‌ای ۵۲ خانه‌ای صفحه‌ی منچ
  const path = [
    [6,1],[6,2],[6,3],[6,4],[6,5],
    [5,6],[4,6],[3,6],[2,6],[1,6],[0,6],
    [0,7],[0,8],
    [1,8],[2,8],[3,8],[4,8],[5,8],
    [6,9],[6,10],[6,11],[6,12],[6,13],[6,14],
    [7,14],[8,14],
    [8,13],[8,12],[8,11],[8,10],[8,9],
    [9,8],[10,8],[11,8],[12,8],[13,8],[14,8],
    [14,7],[14,6],
    [13,6],[12,6],[11,6],[10,6],[9,6],
    [8,5],[8,4],[8,3],[8,2],[8,1],[8,0],
    [7,0],[6,0]
  ];

  // خانه‌های پایانی اختصاصی هر رنگ
  const lanes = [
    [[7,1],[7,2],[7,3],[7,4],[7,5],[7,6]],
    [[13,7],[12,7],[11,7],[10,7],[9,7],[8,7]],
    [[7,13],[7,12],[7,11],[7,10],[7,9],[7,8]],
    [[1,7],[2,7],[3,7],[4,7],[5,7],[6,7]]
  ];

  // جای چهار مهره داخل خانه‌ی شروع هر رنگ
  const bases = [
    {x:0,y:0},
    {x:9,y:0},
    {x:9,y:9},
    {x:0,y:9}
  ];

  let game, audio, soundOn = true, animating = false;

  function newGame() {
    const n = Number(countSelect.value);
    game = {
      players: COLORS.slice(0,n).map((c,i) => ({
        color: c,
        tokens: Array.from({length:4}, () => ({steps:-1})),
        index:i
      })),
      turn:0, dice:0, rolled:false, over:false, winner:null
    };
    animating = false;
    rollBtn.disabled = false;
    rollBtn.textContent = "⚄";
    hint.textContent = "تاس را بزن تا بازی شروع شود 🎲";
    updateUI();
    draw();
  }

  function playSound(freq=440, duration=0.07) {
    if (!soundOn) return;
    try {
      audio = audio || new (window.AudioContext || window.webkitAudioContext)();
      if (audio.state === "suspended") audio.resume();
      const osc = audio.createOscillator();
      const gain = audio.createGain();
      osc.type = "sine";
      osc.frequency.value = freq;
      gain.gain.setValueAtTime(0.08, audio.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.001, audio.currentTime + duration);
      osc.connect(gain);
      gain.connect(audio.destination);
      osc.start();
      osc.stop(audio.currentTime + duration);
    } catch (_) {}
  }

  function currentPlayer() {
    return game.players[game.turn];
  }

  function updateUI() {
    if (!game) return;
    if (game.over) {
      status.textContent = "🏆 برنده: " + game.winner.color.name;
    } else {
      status.textContent = "نوبت " + currentPlayer().color.name;
    }

    playersBox.innerHTML = "";
    COLORS.forEach((c,i) => {
      const p = game.players.find(x => x.index === i);
      const div = document.createElement("div");
      div.className = "player" + (p && p.index === game.turn && !game.over ? " active" : "");
      const dot = document.createElement("span");
      dot.className = "dot";
      dot.style.background = c.hex;
      div.appendChild(dot);
      div.appendChild(document.createTextNode(c.name));
      div.appendChild(document.createElement("br"));
      div.appendChild(document.createTextNode(p ? p.tokens.filter(t => t.steps >= 57).length + "/4 خانه آخر" : "غیرفعال"));
      playersBox.appendChild(div);
    });
  }

  function cellCenter(pos) {
    return {x:(pos[0]+0.5)*SIZE, y:(pos[1]+0.5)*SIZE};
  }

  function roundRect(x,y,w,h,r,fill,stroke) {
    ctx.beginPath();
    ctx.roundRect(x,y,w,h,r);
    if (fill) { ctx.fillStyle=fill; ctx.fill(); }
    if (stroke) { ctx.strokeStyle=stroke; ctx.lineWidth=1.5; ctx.stroke(); }
  }

  function drawCell(x,y,color="#f4f6f4") {
    ctx.fillStyle = color;
    ctx.fillRect(x*SIZE+1,y*SIZE+1,SIZE-2,SIZE-2);
    ctx.strokeStyle = "#a8b6ab";
    ctx.lineWidth = 1;
    ctx.strokeRect(x*SIZE+1,y*SIZE+1,SIZE-2,SIZE-2);
  }

  function drawBase(i) {
    const b = bases[i], c = COLORS[i];
    ctx.fillStyle = c.hex;
    ctx.fillRect(b.x*SIZE,b.y*SIZE,6*SIZE,6*SIZE);

    roundRect(b.x*SIZE+16,b.y*SIZE+16,208,208,22,
      "#f8faf8","#ffffff");

    const spots = [
      [b.x+1.5,b.y+1.5],[b.x+4.5,b.y+1.5],
      [b.x+1.5,b.y+4.5],[b.x+4.5,b.y+4.5]
    ];
    spots.forEach((p,j) => {
      const token = game.players[i]?.tokens[j];
      const occupied = token && token.steps === -1;
      const x = p[0]*SIZE, y = p[1]*SIZE;
      ctx.beginPath();
      ctx.arc(x,y,13,0,Math.PI*2);
      ctx.fillStyle = occupied ? c.hex : "#e3e9e4";
      ctx.fill();
      ctx.strokeStyle = occupied ? "#fff" : "#9aa79d";
      ctx.lineWidth = 2;
      ctx.stroke();
      if (occupied) {
        ctx.beginPath();
        ctx.arc(x-4,y-4,4,0,Math.PI*2);
        ctx.fillStyle = "#ffffffa0";
        ctx.fill();
      }
    });
  }

  function drawHome() {
    // مرکز چهاررنگ
    const x=6*SIZE, y=6*SIZE, s=3*SIZE;
    ctx.fillStyle=COLORS[0].hex;
    ctx.beginPath(); ctx.moveTo(x,y); ctx.lineTo(x+s,y); ctx.lineTo(x+s/2,y+s/2); ctx.closePath(); ctx.fill();
    ctx.fillStyle=COLORS[1].hex;
    ctx.beginPath(); ctx.moveTo(x+s,y); ctx.lineTo(x+s,y+s); ctx.lineTo(x+s/2,y+s/2); ctx.closePath(); ctx.fill();
    ctx.fillStyle=COLORS[2].hex;
    ctx.beginPath(); ctx.moveTo(x+s,y+s); ctx.lineTo(x,y+s); ctx.lineTo(x+s/2,y+s/2); ctx.closePath(); ctx.fill();
    ctx.fillStyle=COLORS[3].hex;
    ctx.beginPath(); ctx.moveTo(x,y+s); ctx.lineTo(x,y); ctx.lineTo(x+s/2,y+s/2); ctx.closePath(); ctx.fill();
    ctx.beginPath();
    ctx.arc(x+s/2,y+s/2,21,0,Math.PI*2);
    ctx.fillStyle="#ffffff";
    ctx.fill();
    ctx.strokeStyle="#17231a";
    ctx.lineWidth=3;
    ctx.stroke();
    ctx.fillStyle="#101810";
    ctx.font="bold 12px system-ui";
    ctx.textAlign="center";
    ctx.textBaseline="middle";
    ctx.fillText("KGF",x+s/2,y+s/2);
  }

  function draw() {
    ctx.clearRect(0,0,600,600);
    ctx.fillStyle="#f1f5f1";
    ctx.fillRect(0,0,600,600);

    drawBase(0);
    drawBase(1);
    drawBase(2);
    drawBase(3);

    path.forEach((p,i) => {
      const safe = COLORS.some(c => c.start === i);
      drawCell(p[0],p[1], safe ? COLORS.find(c=>c.start===i).hex : "#f7f9f7");
    });

    lanes.forEach((lane,i) => {
      lane.forEach(p => drawCell(p[0],p[1],COLORS[i].hex));
    });

    // خانه‌های ورودی به مسیر
    COLORS.forEach((c,i) => {
      const p = path[c.start];
      drawCell(p[0],p[1],c.hex);
    });

    drawHome();

    // مهره‌های خارج‌شده از خانه‌ی شروع
    game.players.forEach(player => {
      player.tokens.forEach((token,j) => {
        if (token.steps < 0) return;
        let pos;
        if (token.steps >= 52) {
          pos = lanes[player.index][Math.min(5,token.steps-52)];
        } else {
          const idx = (player.color.start + token.steps) % 52;
          pos = path[idx];
        }
        const center = cellCenter(pos);
        const same = player.tokens.filter(t => {
          if (t.steps < 0) return false;
          let q = t.steps >= 52 ? lanes[player.index][Math.min(5,t.steps-52)] : path[(player.color.start+t.steps)%52];
          return q[0]===pos[0] && q[1]===pos[1];
        }).length;
        const offset = same > 1 ? (j%2 ? 7 : -7) : 0;
        drawToken(center.x+offset,center.y,player.color,j,
          game.turn===player.index && game.rolled && canMove(player,token));
      });
    });
  }

  function drawToken(x,y,color,j,selectable) {
    ctx.save();
    if (selectable) {
      ctx.shadowColor=color.hex;
      ctx.shadowBlur=16;
    }
    ctx.beginPath();
    ctx.arc(x,y,12,0,Math.PI*2);
    ctx.fillStyle=color.hex;
    ctx.fill();
    ctx.strokeStyle="#ffffff";
    ctx.lineWidth=2;
    ctx.stroke();
    ctx.shadowBlur=0;
    ctx.beginPath();
    ctx.arc(x-3,y-4,4,0,Math.PI*2);
    ctx.fillStyle="#ffffffb0";
    ctx.fill();
    ctx.restore();
  }

  function canMove(player,token) {
    if (!game.rolled || game.over || animating) return false;
    if (token.steps === -1) return game.dice === 6;
    return token.steps < 57 && token.steps + game.dice <= 57;
  }

  function rollDice() {
    if (game.over || game.rolled || animating) return;
    animating = true;
    rollBtn.disabled = true;
    let count = 0;
    const timer = setInterval(() => {
      rollBtn.textContent = ["⚀","⚁","⚂","⚃","⚄","⚅"][Math.floor(Math.random()*6)];
      playSound(300+Math.random()*500,0.04);
      count++;
      if (count >= 9) {
        clearInterval(timer);
        game.dice = Math.floor(Math.random()*6)+1;
        game.rolled = true;
        animating = false;
        rollBtn.disabled = false;
        rollBtn.textContent = ["","⚀","⚁","⚂","⚃","⚄","⚅"][game.dice];
        playSound(650,0.12);

        const p = currentPlayer();
        const moves = p.tokens.filter(t=>canMove(p,t));
        if (moves.length === 0) {
          hint.textContent = game.dice === 6
            ? "مهره‌ی قابل حرکت پیدا نشد؛ نوبت بعدی."
            : "حرکت ممکن نیست؛ نوبت بعدی.";
          draw();
          setTimeout(nextTurn,900);
        } else {
          hint.textContent = game.dice===6
            ? "۶ آوردی! یکی از مهره‌های روشن را انتخاب کن."
            : "یکی از مهره‌های قابل حرکت را لمس کن.";
          draw();
        }
      }
    },65);
  }

  function nextTurn(extra=false) {
    if (game.over) return;
    game.rolled = false;
    game.dice = 0;
    if (!extra) {
      game.turn = (game.turn+1) % game.players.length;
    }
    rollBtn.textContent = "⚄";
    rollBtn.disabled = false;
    hint.textContent = extra ? "۶ آوردی؛ دوباره تاس بنداز!" : "نوبت بازیکن بعدی است.";
    updateUI();
    draw();
  }

  function moveToken(token) {
    const player = currentPlayer();
    if (!canMove(player,token)) return;

    if (token.steps === -1) token.steps = 0;
    else token.steps += game.dice;

    game.rolled = false;
    const rolledSix = game.dice === 6;
    playSound(520,0.09);

    // زدن مهره‌های حریف در خانه‌های غیرامن
    if (token.steps < 52) {
      const posIndex = (player.color.start + token.steps) % 52;
      const safe = COLORS.some(c => c.start === posIndex);
      if (!safe) {
        game.players.forEach(other => {
          if (other === player) return;
          other.tokens.forEach(enemy => {
            if (enemy.steps < 0 || enemy.steps >= 52) return;
            const enemyIndex = (other.color.start + enemy.steps) % 52;
            if (enemyIndex === posIndex) {
              enemy.steps = -1;
              playSound(220,0.18);
            }
          });
        });
      }
    }

    const won = player.tokens.every(t=>t.steps >= 57);
    if (won) {
      game.over = true;
      game.winner = player;
      hint.textContent = "🏆 " + player.color.name + " برنده‌ی بازی شد!";
      rollBtn.disabled = true;
      updateUI();
      draw();
      return;
    }

    hint.textContent = token.steps >= 57 ? "مهره به خانه‌ی پایانی رسید!" : "مهره حرکت کرد.";
    updateUI();
    draw();

    if (rolledSix) nextTurn(true);
    else nextTurn(false);
  }

  function getTokenAt(x,y) {
    let best = null, bestDist = Infinity;
    currentPlayer().tokens.forEach((t,j) => {
      if (!canMove(currentPlayer(),t)) return;
      let pos;
      if (t.steps === -1) {
        const b=bases[currentPlayer().index];
        const spots=[[b.x+1.5,b.y+1.5],[b.x+4.5,b.y+1.5],[b.x+1.5,b.y+4.5],[b.x+4.5,b.y+4.5]];
        pos=[spots[j][0],spots[j][1]];
      } else if (t.steps >= 52) {
        pos=lanes[currentPlayer().index][Math.min(5,t.steps-52)];
      } else {
        pos=path[(currentPlayer().color.start+t.steps)%52];
      }
      const c=cellCenter(pos);
      const d=Math.hypot(x-c.x,y-c.y);
      if (d<25 && d<bestDist) { best=t; bestDist=d; }
    });
    return best;
  }

  canvas.addEventListener("pointerdown", e => {
    if (!game.rolled || game.over || animating) return;
    const rect=canvas.getBoundingClientRect();
    const x=(e.clientX-rect.left)*600/rect.width;
    const y=(e.clientY-rect.top)*600/rect.height;
    const token=getTokenAt(x,y);
    if (token) moveToken(token);
  });

  rollBtn.addEventListener("click",rollDice);
  document.getElementById("resetBtn").addEventListener("click",newGame);
  countSelect.addEventListener("change",newGame);
  soundBtn.addEventListener("click",() => {
    soundOn=!soundOn;
    soundBtn.textContent=soundOn ? "🔊 صدا روشن" : "🔇 صدا خاموش";
    if (soundOn) playSound(600,0.1);
  });

  newGame();
})();
</script>
</body>
</html>
