
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="theme-color" content="#090b18">
<title>KGF LUDO | بازی منچ</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#090b18;color:#f5f7ff;font-family:Tahoma,Arial,sans-serif}
.app{max-width:950px;margin:auto;padding:18px}
header{text-align:center;padding:24px 8px}
.logo{font-size:38px;font-weight:900;letter-spacing:4px;color:#54ffb0;text-shadow:0 0 12px #00ff9d,0 0 35px #00ff9d66}
.subtitle{color:#9ba8c7;font-size:13px;margin-top:12px}
.panel{background:#12172a;border:1px solid #293453;border-radius:22px;padding:16px;margin-bottom:16px;box-shadow:0 0 30px #050812}
.status{display:flex;justify-content:space-between;gap:8px;flex-wrap:wrap;color:#b7c5e5;font-size:12px}
.pill{border:1px solid #354361;border-radius:20px;padding:8px 12px}
.board-wrap{max-width:540px;margin:18px auto;position:relative}
canvas{display:block;width:100%;height:auto;border-radius:18px;border:1px solid #3b4667;box-shadow:0 0 28px #6b42ff26}
.players{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:10px}
.player{background:#0c1121;border:1px solid #283451;border-radius:15px;padding:12px;display:flex;align-items:center;gap:10px}
.dot{width:15px;height:15px;border-radius:50%;box-shadow:0 0 12px currentColor}
.player strong{font-size:13px}
.player small{display:block;color:#929fbd;margin-top:5px;font-size:11px}
.active{border-color:#54ffb0;box-shadow:0 0 16px #54ffb022}
.actions{display:flex;justify-content:center;gap:12px;flex-wrap:wrap;margin:18px 0 8px}
button{font-family:inherit;font-weight:bold;cursor:pointer;border:0;border-radius:14px;padding:14px 20px;font-size:14px}
.primary{background:linear-gradient(120deg,#54ffb0,#1ed8c0);color:#07120e;box-shadow:0 0 20px #54ffb033}
.secondary{background:#202945;color:#e8edff;border:1px solid #354361}
button:disabled{opacity:.45;cursor:not-allowed}
.dice{font-size:42px;text-align:center;min-height:65px;margin:12px 0;text-shadow:0 0 18px #9a73ff}
.dice.rolling{animation:shake .45s ease-in-out}
@keyframes shake{0%,100%{transform:rotate(0) scale(1)}25%{transform:rotate(-18deg) scale(1.15)}75%{transform:rotate(18deg) scale(1.15)}}
.note{text-align:center;color:#8997b8;font-size:11px;line-height:2;margin:18px 4px}
.toast{text-align:center;min-height:24px;color:#54ffb0;font-size:13px}
footer{text-align:center;padding:18px;color:#566483;font-size:11px}
@media(max-width:420px){.app{padding:10px}.logo{font-size:31px}.panel{padding:11px}.player{padding:9px}.player strong{font-size:12px}}
</style>
</head>
<body>
<div class="app">
<header>
  <div class="logo">KGF LUDO</div>
  <div class="subtitle">بازی منچ مدرن • رقابت کن و برنده شو</div>
</header>

<section class="panel">
  <div class="status">
    <span class="pill">🎮 حالت نمایشی</span>
    <span class="pill" id="turnLabel">نوبت: قرمز</span>
    <span class="pill">⚡ KGF ARENA</span>
  </div>

  <div class="board-wrap">
    <canvas id="board" width="600" height="600"></canvas>
  </div>

  <div class="players" id="players">
    <div class="player active"><span class="dot" style="color:#ff4b68;background:#ff4b68"></span><div><strong>بازیکن قرمز</strong><small id="score0">۴ مهره در خانه</small></div></div>
    <div class="player"><span class="dot" style="color:#38e9a4;background:#38e9a4"></span><div><strong>بازیکن سبز</strong><small id="score1">۴ مهره در خانه</small></div></div>
    <div class="player"><span class="dot" style="color:#ffd34e;background:#ffd34e"></span><div><strong>بازیکن زرد</strong><small id="score2">۴ مهره در خانه</small></div></div>
    <div class="player"><span class="dot" style="color:#5b91ff;background:#5b91ff"></span><div><strong>بازیکن آبی</strong><small id="score3">۴ مهره در خانه</small></div></div>
  </div>

  <div class="dice" id="dice">🎲</div>
  <div class="toast" id="message">برای شروع، روی «پرتاب تاس» بزن.</div>
  <div class="actions">
    <button class="primary" id="roll">🎲 پرتاب تاس</button>
    <button class="secondary" id="reset">↻ بازی جدید</button>
    <button class="secondary" id="sound">🔊 صدا روشن</button>
  </div>
</section>

<div class="note">
این نسخه، پیش‌نمایش قابل‌بازی برای آزمایش ظاهر و تاس است.<br>
بازی آنلاین چندنفره و اتصال به ربات تلگرام در مرحله بعد اضافه می‌شود.
</div>
<footer>KGF © LUDO ARENA</footer>
</div>

<script>
const canvas=document.getElementById("board");
const ctx=canvas.getContext("2d");
const colors=["#ff4b68","#38e9a4","#ffd34e","#5b91ff"];
const names=["قرمز","سبز","زرد","آبی"];
const diceFaces=["⚀","⚁","⚂","⚃","⚄","⚅"];
const path=[];
let turn=0, dice=0, soundOn=true, busy=false;
let pawns=colors.map(()=>[-1,-1,-1,-1]);
const center={x:300,y:300};
const safe=[0,8,13,21,26,34,39,47];

for(let i=0;i<52;i++){
  const angle=(-Math.PI/2)+(i/52)*Math.PI*2;
  const r=206;
  path.push({x:center.x+Math.cos(angle)*r,y:center.y+Math.sin(angle)*r});
}
function rounded(x,y,w,h,r,color){
  ctx.fillStyle=color;ctx.beginPath();ctx.roundRect(x,y,w,h,r);ctx.fill();
}
function draw(){
  ctx.clearRect(0,0,600,600);
  rounded(0,0,600,600,24,"#10162a");
  const bases=[
    {x:22,y:22,c:colors[0]},
    {x:378,y:22,c:colors[1]},
    {x:22,y:378,c:colors[2]},
    {x:378,y:378,c:colors[3]}
  ];
  bases.forEach((b,i)=>{
    rounded(b.x,b.y,200,200,22,b.c+"20");
    ctx.strokeStyle=b.c;ctx.lineWidth=2;
    ctx.beginPath();ctx.roundRect(b.x,b.y,200,200,22);ctx.stroke();
    for(let j=0;j<4;j++){
      const px=b.x+58+(j%2)*78,py=b.y+58+Math.floor(j/2)*78;
      ctx.beginPath();ctx.arc(px,py,22,0,Math.PI*2);
      ctx.fillStyle="#0b1020";ctx.fill();
      ctx.strokeStyle=b.c;ctx.lineWidth=3;ctx.stroke();
    }
  });
  path.forEach((p,i)=>{
    ctx.beginPath();ctx.arc(p.x,p.y,13,0,Math.PI*2);
    ctx.fillStyle=safe.includes(i)?"#dce6ff": "#202b45";
    ctx.fill();
    ctx.strokeStyle=safe.includes(i)?"#fff":"#475573";
    ctx.lineWidth=1.5;ctx.stroke();
  });
  // مرکز رنگی
  const triangles=[
    {c:colors[0],a:-Math.PI/2},
    {c:colors[1],a:0},
    {c:colors[3],a:Math.PI},
    {c:colors[2],a:Math.PI/2}
  ];
  triangles.forEach(t=>{
    ctx.beginPath();ctx.moveTo(300,300);
    ctx.lineTo(300+Math.cos(t.a-.52)*85,300+Math.sin(t.a-.52)*85);
    ctx.lineTo(300+Math.cos(t.a+.52)*85,300+Math.sin(t.a+.52)*85);
    ctx.closePath();ctx.fillStyle=t.c;ctx.fill();
  });
  ctx.beginPath();ctx.arc(300,300,22,0,Math.PI*2);
  ctx.fillStyle="#fff";ctx.shadowColor="#fff";ctx.shadowBlur=20;ctx.fill();ctx.shadowBlur=0;
  // مهره‌های بازیکن‌ها
  pawns.forEach((arr,player)=>{
    arr.forEach((pos,piece)=>{
      let x,y;
      if(pos<0){
        const b=bases[player];
        x=b.x+58+(piece%2)*78;
        y=b.y+58+Math.floor(piece/2)*78;
      }else{
        const p=path[pos%52];x=p.x;y=p.y;
      }
      ctx.beginPath();ctx.arc(x,y,12,0,Math.PI*2);
      ctx.fillStyle=colors[player];ctx.shadowColor=colors[player];ctx.shadowBlur=14;ctx.fill();ctx.shadowBlur=0;
      ctx.strokeStyle="#fff";ctx.lineWidth=2;ctx.stroke();
      ctx.beginPath();ctx.arc(x-3,y-4,3,0,Math.PI*2);
      ctx.fillStyle="#fff";ctx.fill();
    });
  });
}
function sound(freq=440,duration=.12){
  if(!soundOn)return;
  try{
    const AC=window.AudioContext||window.webkitAudioContext;
    const ac=new AC(),o=ac.createOscillator(),g=ac.createGain();
    o.connect(g);g.connect(ac.destination);o.frequency.value=freq;
    g.gain.setValueAtTime(.07,ac.currentTime);
    g.gain.exponentialRampToValueAtTime(.001,ac.currentTime+duration);
    o.start();o.stop(ac.currentTime+duration);
    o.onended=()=>ac.close();
  }catch(e){}
}
function updateUI(){
  document.getElementById("turnLabel").textContent="نوبت: "+names[turn];
  document.querySelectorAll(".player").forEach((el,i)=>el.classList.toggle("active",i===turn));
  pawns.forEach((arr,i)=>{
    const home=arr.filter(p=>p<0).length;
    document.getElementById("score"+i).textContent=home+" مهره در خانه";
  });
}
function message(t){document.getElementById("message").textContent=t;}
document.getElementById("roll").addEventListener("click",()=>{
  if(busy)return;
  busy=true;
  const btn=document.getElementById("roll");
  const die=document.getElementById("dice");
  btn.disabled=true;die.classList.add("rolling");
  sound(300,.08);
  let count=0;
  const interval=setInterval(()=>{
    dice=1+Math.floor(Math.random()*6);
    die.textContent=diceFaces[dice-1];
    count++;
    if(count>8){
      clearInterval(interval);
      die.classList.remove("rolling");
      sound(520,.15);
      let options=pawns[turn].map((p,i)=>({p,i})).filter(o=>o.p>=0&&o.p<57);
      if(dice===6){
        const home=pawns[turn].findIndex(p=>p<0);
        if(home!==-1){
          pawns[turn][home]=0;
          message("🎉 یک مهره وارد بازی شد!");
        }else if(options.length){
          pawns[turn][options[0].i]=(options[0].p+dice)%52;
          message("عالی! مهره حرکت کرد.");
        }
      }else if(options.length){
        pawns[turn][options[0].i]=(options[0].p+dice)%52;
        message("مهره "+names[turn]+" حرکت کرد.");
      }else{
        message(dice===6?"تاس ۶ آمد! دوباره نوبت توست.":"مهره‌ای برای حرکت نیست.");
      }
      draw();updateUI();
      if(dice!==6)turn=(turn+1)%4;
      updateUI();
      busy=false;btn.disabled=false;
    }
  },75);
});
document.getElementById("reset").addEventListener("click",()=>{
  pawns=colors.map(()=>[-1,-1,-1,-1]);turn=0;dice=0;
  document.getElementById("dice").textContent="🎲";
  message("بازی جدید آماده است!");
  draw();updateUI();sound(620,.18);
});
document.getElementById("sound").addEventListener("click",()=>{
  soundOn=!soundOn;
  document.getElementById("sound").textContent=soundOn?"🔊 صدا روشن":"🔇 صدا خاموش";
  if(soundOn)sound(660,.15);
});
draw();updateUI();
</script>
</body>
</html>

