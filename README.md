# index.html


<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<meta name="theme-color" content="#080d0a">
<title>KGF LUDO | بازی منچ</title>
<script src="https://telegram.org/js/telegram-web-app.js"></script>

<style>
:root{--bg:#080d0a;--panel:#111a14;--green:#39ff88;--muted:#91a398}
*{box-sizing:border-box}
body{margin:0;min-height:100vh;color:#fff;background:radial-gradient(ellipse at top,#153323,transparent 55%),var(--bg);font-family:system-ui,Tahoma,sans-serif;text-align:center}
header{padding:20px 12px 12px}
.logo{font-size:30px;font-weight:1000;font-style:italic;letter-spacing:3px;color:var(--green);text-shadow:0 0 20px #39ff8860}
.subtitle{color:var(--muted);font-size:12px;margin-top:4px}
.wrap{width:min(94vw,560px);margin:auto}
.panel{background:#111a14;border:1px solid #254532;border-radius:18px;padding:13px;margin:10px 0;box-shadow:0 8px 35px #0005}
.row{display:flex;align-items:center;justify-content:space-between;gap:8px;flex-wrap:wrap}
label{color:#a9b9ae;font-size:12px}
select,button{font:inherit;border-radius:11px;border:1px solid #355541;background:#18251c;color:white;padding:10px 12px}
button{cursor:pointer}
button:active{transform:scale(.97)}
.primary{background:var(--green);color:#07140b;font-weight:900;border:none}
#status{font-size:16px;font-weight:900;margin:2px 0 7px}
#hint{color:var(--muted);font-size:12px;min-height:20px}
.dice-zone{height:120px;display:flex;align-items:flex-end;justify-content:center;padding-bottom:7px;overflow:hidden}
.dice{width:76px;height:76px;border-radius:18px;background:linear-gradient(145deg,#fff,#dce8df);color:#111;font-size:43px;font-weight:900;border:3px solid #fff;box-shadow:0 5px 0 #87968b,0 0 20px #39ff8830;transform:translateY(0)}
.dice.rolling{animation:throwDice .65s cubic-bezier(.2,.8,.3,1)}
@keyframes throwDice{
  0%{transform:translateY(35px) rotate(-20deg) scale(.65);opacity:.25}
  45%{transform:translateY(-22px) rotate(180deg) scale(1.12);opacity:1}
  75%{transform:translateY(5px) rotate(330deg) scale(.96)}
  100%{transform:translateY(0) rotate(360deg) scale(1)}
}
.dice:disabled{opacity:.7;cursor:default}
.board-box{padding:8px;background:linear-gradient(145deg,#17231a,#0b100d);border:1px solid #34513d;border-radius:20px;box-shadow:0 0 35px #39ff8810}
canvas{display:block;width:100%;height:auto;border-radius:12px;touch-action:manipulation}
.players{display:grid;grid-template-columns:repeat(4,1fr);gap:6px}
.player{padding:9px 3px;border-radius:12px;background:#0c130e;border:1px solid #263b2c;font-size:11px;color:#c4d2c8}
.player.active{border-color:var(--green);box-shadow:inset 0 0 12px #39ff8815;color:white}
.dot{display:block;width:17px;height:17px;border-radius:50%;margin:0 auto 5px;border:2px solid #ffffffa0}
.footer{color:#637568;font-size:10px;padding:16px}
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
    <div id="hint">برای شروع، تاس را از پایین به بالا پرتاب کن 🎲</div>
    <div class="dice-zone">
      <button id="rollBtn" class="dice" aria-label="انداختن تاس">⚄</button>
    </div>
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

const tg=window.Telegram?.WebApp;
if(tg){tg.ready();tg.expand();try{tg.setHeaderColor("#080d0a");tg.setBackgroundColor("#080d0a")}catch(e){}}

const canvas=document.getElementById("board"),ctx=canvas.getContext("2d");
const rollBtn=document.getElementById("rollBtn"),hint=document.getElementById("hint");
const status=document.getElementById("status"),countSelect=document.getElementById("playerCount");
const playersBox=document.getElementById("players"),soundBtn=document.getElementById("soundBtn");
const SIZE=40;

const COLORS=[
{name:"قرمز",hex:"#ff4b55",start:0},
{name:"سبز",hex:"#30df79",start:13},
{name:"زرد",hex:"#ffd447",start:26},
{name:"آبی",hex:"#439bff",start:39}
];

const path=[
[6,1],[6,2],[6,3],[6,4],[6,5],[5,6],[4,6],[3,6],[2,6],[1,6],[0,6],[0,7],[0,8],
[1,8],[2,8],[3,8],[4,8],[5,8],[6,9],[6,10],[6,11],[6,12],[6,13],[6,14],[7,14],[8,14],
[8,13],[8,12],[8,11],[8,10],[8,9],[9,8],[10,8],[11,8],[12,8],[13,8],[14,8],[14,7],[14,6],
[13,6],[12,6],[11,6],[10,6],[9,6],[8,5],[8,4],[8,3],[8,2],[8,1],[8,0],[7,0],[6,0]
];

const lanes=[
[[7,1],[7,2],[7,3],[7,4],[7,5],[7,6]],
[[13,7],[12,7],[11,7],[10,7],[9,7],[8,7]],
[[7,13],[7,12],[7,11],[7,10],[7,9],[7,8]],
[[1,7],[2,7],[3,7],[4,7],[5,7],[6,7]]
];

const bases=[{x:0,y:0},{x:9,y:0},{x:9,y:9},{x:0,y:9}];
let game,audio,soundOn=true,animating=false;

function newGame(){
  const n=Number(countSelect.value);
  game={players:COLORS.slice(0,n).map((c,i)=>({color:c,tokens:Array.from({length:4},()=>({steps:-1})),index:i})),turn:0,dice:0,rolled:false,over:false,winner:null};
  animating=false;rollBtn.disabled=false;rollBtn.textContent="⚄";
  hint.textContent="برای شروع تاس را بزن 🎲";
  updateUI();draw();
}
function playSound(freq=440,duration=.07){
  if(!soundOn)return;
  try{
    audio=audio||new(window.AudioContext||window.webkitAudioContext)();
    if(audio.state==="suspended")audio.resume();
    const osc=audio.createOscillator(),gain=audio.createGain();
    osc.type="sine";osc.frequency.value=freq;
    gain.gain.setValueAtTime(.08,audio.currentTime);
    gain.gain.exponentialRampToValueAtTime(.001,audio.currentTime+duration);
    osc.connect(gain);gain.connect(audio.destination);osc.start();osc.stop(audio.currentTime+duration);
  }catch(e){}
}
function currentPlayer(){return game.players[game.turn]}
function updateUI(){
  status.textContent=game.over?"🏆 برنده: "+game.winner.color.name:"نوبت "+currentPlayer().color.name;
  playersBox.replaceChildren();
  COLORS.forEach((c,i)=>{
    const p=game.players.find(x=>x.index===i),div=document.createElement("div"),dot=document.createElement("span");
    div.className="player"+(p&&p.index===game.turn&&!game.over?" active":"");
    dot.className="dot";dot.style.background=c.hex;
    div.append(dot,document.createTextNode(c.name),document.createElement("br"),document.createTextNode(p?p.tokens.filter(t=>t.steps>=57).length+"/4 خانه آخر":"غیرفعال"));
    playersBox.appendChild(div);
  });
}
function center(p){return{x:(p[0]+.5)*SIZE,y:(p[1]+.5)*SIZE}}
function roundRect(x,y,w,h,r,fill,stroke){
  ctx.beginPath();ctx.roundRect(x,y,w,h,r);
  if(fill){ctx.fillStyle=fill;ctx.fill()}
  if(stroke){ctx.strokeStyle=stroke;ctx.lineWidth=1.5;ctx.stroke()}
}
function drawCell(x,y,color="#f7f9f7"){
  ctx.fillStyle=color;ctx.fillRect(x*SIZE+1,y*SIZE+1,SIZE-2,SIZE-2);
  ctx.strokeStyle="#a8b6ab";ctx.lineWidth=1;ctx.strokeRect(x*SIZE+1,y*SIZE+1,SIZE-2,SIZE-2);
}
function drawBase(i){
  const b=bases[i],c=COLORS[i];
  ctx.fillStyle=c.hex;ctx.fillRect(b.x*SIZE,b.y*SIZE,240,240);
  roundRect(b.x*SIZE+16,b.y*SIZE+16,208,208,22,"#f8faf8","#fff");
  const spots=[[b.x+1.5,b.y+1.5],[b.x+4.5,b.y+1.5],[b.x+1.5,b.y+4.5],[b.x+4.5,b.y+4.5]];
  spots.forEach((p,j)=>{
    const t=game.players[i]?.tokens[j],occupied=t&&t.steps===-1,x=p[0]*SIZE,y=p[1]*SIZE;
    ctx.beginPath();ctx.arc(x,y,13,0,Math.PI*2);ctx.fillStyle=occupied?c.hex:"#e3e9e4";ctx.fill();
    ctx.strokeStyle=occupied?"#fff":"#9aa79d";ctx.lineWidth=2;ctx.stroke();
    if(occupied){ctx.beginPath();ctx.arc(x-4,y-4,4,0,Math.PI*2);ctx.fillStyle="#ffffffa0";ctx.fill()}
  });
}
function drawHome(){
  const x=240,y=240,s=120;
  [[0,0,COLORS[0].hex],[1,0,COLORS[1].hex],[1,1,COLORS[2].hex],[0,1,COLORS[3].hex]].forEach(([cx,cy,col])=>{
    ctx.fillStyle=col;ctx.fillRect(x+cx*s/2,y+cy*s/2,s/2,s/2);
  });
  ctx.beginPath();ctx.moveTo(240,240);ctx.lineTo(360,240);ctx.lineTo(300,300);ctx.closePath();ctx.fillStyle=COLORS[0].hex;ctx.fill();
  ctx.beginPath();ctx.moveTo(360,240);ctx.lineTo(360,360);ctx.lineTo(300,300);ctx.closePath();ctx.fillStyle=COLORS[1].hex;ctx.fill();
  ctx.beginPath();ctx.moveTo(360,360);ctx.lineTo(240,360);ctx.lineTo(300,300);ctx.closePath();ctx.fillStyle=COLORS[2].hex;ctx.fill();
  ctx.beginPath();ctx.moveTo(240,360);ctx.lineTo(240,240);ctx.lineTo(300,300);ctx.closePath();ctx.fillStyle=COLORS[3].hex;ctx.fill();
  ctx.beginPath();ctx.arc(300,300,20,0,Math.PI*2);ctx.fillStyle="#fff";ctx.fill();
  ctx.fillStyle="#101810";ctx.font="bold 12px system-ui";ctx.textAlign="center";ctx.textBaseline="middle";ctx.fillText("KGF",300,300);
}
function drawToken(x,y,color,selectable){
  ctx.save();if(selectable){ctx.shadowColor=color.hex;ctx.shadowBlur=16}
  ctx.beginPath();ctx.arc(x,y,12,0,Math.PI*2);ctx.fillStyle=color.hex;ctx.fill();
  ctx.strokeStyle="#fff";ctx.lineWidth=2;ctx.stroke();ctx.shadowBlur=0;
  ctx.beginPath();ctx.arc(x-3,y-4,4,0,Math.PI*2);ctx.fillStyle="#ffffffb0";ctx.fill();ctx.restore();
}
function tokenPos(player,t,j){
  if(t.steps===-1){
    const b=bases[player.index],spots=[[b.x+1.5,b.y+1.5],[b.x+4.5,b.y+1.5],[b.x+1.5,b.y+4.5],[b.x+4.5,b.y+4.5]];
    return spots[j];
  }
  if(t.steps>=52)return lanes[player.index][Math.min(5,t.steps-52)];
  return path[(player.color.start+t.steps)%52];
}
function draw(){
  ctx.clearRect(0,0,600,600);ctx.fillStyle="#f1f5f1";ctx.fillRect(0,0,600,600);
  for(let i=0;i<4;i++)drawBase(i);
  path.forEach((p,i)=>drawCell(p[0],p[1],COLORS.find(c=>c.start===i)?.hex||"#f7f9f7"));
  lanes.forEach((lane,i)=>lane.forEach(p=>drawCell(p[0],p[1],COLORS[i].hex)));
  drawHome();
  game.players.forEach(p=>p.tokens.forEach((t,j)=>{
    if(t.steps<0)return;
    const pos=tokenPos(p,t,j),c=center(pos);
    const same=p.tokens.filter(a=>a.steps>=0&&tokenPos(p,a,p.tokens.indexOf(a)).join(",")===pos.join(",")).length;
    drawToken(c.x+(same>1?(j%2?7:-7):0),c.y,p.color,p.index===game.turn&&canMove(p,t));
  }));
}
function canMove(p,t){
  if(!game.rolled||game.over||animating)return false;
  if(t.steps===-1)return game.dice===6;
  return t.steps<57&&t.steps+game.dice<=57;
}
function nextTurn(extra=false){
  if(game.over)return;
  game.rolled=false;game.dice=0;
  if(!extra)game.turn=(game.turn+1)%game.players.length;
  rollBtn.textContent="⚄";rollBtn.disabled=false;
  hint.textContent=extra?"۶ آوردی؛ دوباره تاس بنداز!":"نوبت بازیکن بعدی است.";
  updateUI();draw();
}
function rollDice(){
  if(game.over||game.rolled||animating)return;
  animating=true;rollBtn.disabled=true;
  rollBtn.classList.remove("rolling");
  void rollBtn.offsetWidth;
  rollBtn.classList.add("rolling");
  let count=0;
  const timer=setInterval(()=>{
    rollBtn.textContent=["⚀","⚁","⚂","⚃","⚄","⚅"][Math.floor(Math.random()*6)];
    playSound(300+Math.random()*500,.04);count++;
    if(count>=9){
      clearInterval(timer);rollBtn.classList.remove("rolling");
      game.dice=Math.floor(Math.random()*6)+1;game.rolled=true;animating=false;rollBtn.disabled=false;
      rollBtn.textContent=["","⚀","⚁","⚂","⚃","⚄","⚅"][game.dice];playSound(650,.12);
      const p=currentPlayer(),moves=p.tokens.filter(t=>canMove(p,t));
      if(!moves.length){
        hint.textContent="حرکت ممکن نیست؛ نوبت بعدی.";
        draw();setTimeout(()=>nextTurn(game.dice===6),900);
      }else{
        hint.textContent=game.dice===6?"۶ آوردی! یکی از مهره‌ها را انتخاب کن.":"یکی از مهره‌های قابل حرکت را لمس کن.";
        draw();
      }
    }
  },65);
}
function moveToken(token){
  const p=currentPlayer();if(!canMove(p,token))return;
  if(token.steps===-1)token.steps=0;else token.steps+=game.dice;
  const six=game.dice===6;game.rolled=false;playSound(520,.09);
  if(token.steps<52){
    const idx=(p.color.start+token.steps)%52;
    if(!COLORS.some(c=>c.start===idx)){
      game.players.forEach(other=>{
        if(other===p)return;
        other.tokens.forEach(enemy=>{
          if(enemy.steps>=0&&enemy.steps<52&&(other.color.start+enemy.steps)%52===idx){
            enemy.steps=-1;playSound(220,.18);
          }
        });
      });
    }
  }
  if(p.tokens.every(t=>t.steps>=57)){
    game.over=true;game.winner=p;hint.textContent="🏆 "+p.color.name+" برنده شد!";
    rollBtn.disabled=true;updateUI();draw();return;
  }
  hint.textContent="مهره حرکت کرد.";updateUI();draw();nextTurn(six);
}
function getTokenAt(x,y){
  let best=null,dist=Infinity;const p=currentPlayer();
  p.tokens.forEach((t,j)=>{
    if(!canMove(p,t))return;
    const c=center(tokenPos(p,t,j)),d=Math.hypot(x-c.x,y-c.y);
    if(d<25&&d<dist){best=t;dist=d}
  });
  return best;
}
canvas.addEventListener("pointerdown",e=>{
  if(!game.rolled||game.over||animating)return;
  const r=canvas.getBoundingClientRect(),x=(e.clientX-r.left)*600/r.width,y=(e.clientY-r.top)*600/r.height;
  const t=getTokenAt(x,y);if(t)moveToken(t);
});
rollBtn.addEventListener("click",rollDice);
document.getElementById("resetBtn").addEventListener("click",newGame);
countSelect.addEventListener("change",newGame);
soundBtn.addEventListener("click",()=>{
  soundOn=!soundOn;soundBtn.textContent=soundOn?"🔊 صدا روشن":"🔇 صدا خاموش";
  if(soundOn)playSound(600,.1);
});
newGame();
})();
</script>
</body>
</html>
