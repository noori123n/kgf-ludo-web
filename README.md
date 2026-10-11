# index.html

<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,viewport-fit=cover">
<meta name="theme-color" content="#080b09">
<title>KGF LUDO</title>
<script src="https://telegram.org/js/telegram-web-app.js"></script>

<style>
*{box-sizing:border-box}
:root{
 --bg:#080b09;--green:#39ff78;
 --red:#f43d4c;--blue:#3197ff;
 --yellow:#ffd83d;--lime:#31d96b;
}
body{
 margin:0;background:radial-gradient(ellipse at top,#15241a,#070907 70%);
 color:white;font-family:Arial,Tahoma,sans-serif;
 min-height:100vh;min-height:100dvh;
}
.app{
 width:100%;max-width:570px;margin:auto;padding:12px 10px 0;
 min-height:100vh;min-height:100dvh;
 display:flex;flex-direction:column;gap:10px;
}
header{text-align:center;padding:3px}
.logo{font-weight:1000;font-size:35px;letter-spacing:5px;color:var(--green);
 text-shadow:0 0 18px #39ff7866}
.sub{font-size:10px;letter-spacing:3px;color:#a6b8aa}
.status{
 display:flex;align-items:center;justify-content:space-between;gap:8px;
 background:#111813;border:1px solid #2b4632;border-radius:12px;padding:10px;
 font-size:13px;
}
#status{font-weight:bold}
#playerCount{
 color:var(--green);background:#19251c;border:1px solid #34543c;
 padding:6px;border-radius:8px;
}
.board-box{
 width:100%;padding:5px;border-radius:15px;
 border:1px solid #35563c;background:#0c110d;
 box-shadow:0 0 24px #39ff7810;
}
#board{width:100%;height:auto;display:block;touch-action:manipulation}
.hint{text-align:center;min-height:32px;color:#d8e8dc;font-size:12px;line-height:1.8}
.dock{
 position:sticky;bottom:0;margin-top:auto;z-index:5;
 padding:12px 10px calc(12px + env(safe-area-inset-bottom));
 background:linear-gradient(180deg,#101712f7,#080b09);
 border-top:1px solid #2e4b35;border-radius:16px 16px 0 0;
 box-shadow:0 -7px 22px #0008;
}
.dice-row{display:flex;align-items:center;justify-content:center;gap:14px}
#dice{
 width:72px;height:72px;flex:0 0 72px;border-radius:17px;
 background:#142018;border:2px solid var(--green);color:white;
 font-size:42px;box-shadow:0 0 18px #39ff7833;cursor:pointer;
}
#dice:active{transform:scale(.94)}
.roll-title{color:var(--green);font-weight:bold;font-size:16px}
.roll-note{color:#a9b9ad;font-size:11px;margin-top:6px;line-height:1.7}
.buttons{display:flex;gap:8px;margin-top:10px}
button.action{
 flex:1;border:1px solid #304735;border-radius:10px;
 background:#172019;color:white;padding:10px 5px;font-size:12px;
}
button.primary{background:var(--green);color:#071109;font-weight:bold;border-color:var(--green)}
footer{text-align:center;color:#627666;font-size:9px;padding:4px 0 8px}
@media(max-height:700px){
 .app{gap:6px;padding-top:5px}
 .logo{font-size:28px}
 #dice{width:60px;height:60px;flex-basis:60px;font-size:35px}
 .dock{padding-top:8px}
}
</style>
</head>
<body>
<div class="app">
 <header>
  <div class="logo">KGF</div>
  <div class="sub">L U D O&nbsp; G A M E</div>
 </header>

 <section class="status">
  <span id="status">نوبت بازیکن قرمز</span>
  <select id="playerCount" aria-label="تعداد بازیکنان">
   <option value="2">۲ بازیکن</option>
   <option value="3">۳ بازیکن</option>
   <option value="4" selected>۴ بازیکن</option>
  </select>
 </section>

 <section class="board-box">
  <canvas id="board" width="600" height="600"></canvas>
 </section>

 <div class="hint" id="hint">برای شروع، تاس پایین صفحه را بزن 🎲</div>

 <section class="dock">
  <div class="dice-row">
   <button id="dice" aria-label="انداختن تاس">⚄</button>
   <div>
    <div class="roll-title">تاس منچ</div>
    <div class="roll-note" id="diceNote">تاس را بزن و مهره‌ات را حرکت بده</div>
   </div>
  </div>
  <div class="buttons">
   <button class="action primary" id="newGame">بازی جدید</button>
   <button class="action" id="sound">🔊 صدا روشن</button>
  </div>
 </section>
 <footer>KGF LUDO · CLASSIC BOARD</footer>
</div>

<script>
(() => {
 const tg=window.Telegram?.WebApp;
 if(tg){tg.ready();tg.expand()}

 const canvas=document.getElementById('board');
 const ctx=canvas.getContext('2d');
 const statusEl=document.getElementById('status');
 const hint=document.getElementById('hint');
 const note=document.getElementById('diceNote');
 const diceBtn=document.getElementById('dice');
 const playerSelect=document.getElementById('playerCount');

 const C=[
  {name:'قرمز',color:'#f43d4c',dark:'#a91e30',pale:'#ffd9dd'},
  {name:'سبز',color:'#31d96b',dark:'#14833c',pale:'#d4ffe1'},
  {name:'زرد',color:'#ffd83d',dark:'#b88b00',pale:'#fff4b9'},
  {name:'آبی',color:'#3197ff',dark:'#155da9',pale:'#d8edff'}
 ];

 /*
  * مسیر استاندارد حلقه‌ای ۵۲ خانه‌ای در صفحه ۱۵×۱۵.
  * خانه‌های ورودی هر رنگ و خانه‌های امن با ستاره مشخص شده‌اند.
  */
 const path=[];
 for(let x=6;x<=8;x++)path.push([x,0]);
 path.push([8,1],[8,2],[8,3],[8,4],[8,5]);
 path.push([9,6],[10,6],[11,6],[12,6],[13,6],[14,6]);
 path.push([14,7],[14,8]);
 for(let x=13;x>=9;x--)path.push([x,8]);
 path.push([8,9],[8,10],[8,11],[8,12],[8,13],[8,14]);
 path.push([7,14],[6,14]);
 for(let x=5;x>=0;x--)path.push([x,13]);
 path.push([0,12],[0,11],[0,10],[0,9],[0,8]);
 path.push([1,8],[2,8],[3,8],[4,8],[5,8]);
 path.push([6,7],[6,6],[5,6],[4,6],[3,6],[2,6],[1,6],[0,6]);
 path.push([0,5],[0,4],[0,3],[0,2],[0,1]);
 // اگر ترتیب بالا بیشتر از ۵۲ خانه تولید کرد، مسیر را به ۵۲ خانه محدود می‌کنیم.
 const TRACK=path.slice(0,52);

 const homes=[
  [[1,1],[4,1],[1,4],[4,4]],
  [[10,1],[13,1],[10,4],[13,4]],
  [[10,10],[13,10],[10,13],[13,13]],
  [[1,10],[4,10],[1,13],[4,13]]
 ];
 const starts=[0,13,26,39];
 const safeStarts=new Set(starts);

 let players=4,turn=0,rolled=0,needMove=false,sound=true,locked=false;
 let pawns=[];

 function reset(){
  players=+playerSelect.value;turn=0;rolled=0;needMove=false;locked=false;
  pawns=C.map(()=>Array.from({length:4},()=>({pos:-1})));
  diceBtn.textContent='⚄';note.textContent='تاس را بزن و مهره‌ات را حرکت بده';
  hint.textContent='برای شروع، تاس پایین صفحه را بزن 🎲';
  updateStatus();draw();
 }

 function updateStatus(){
  statusEl.textContent='نوبت بازیکن '+C[turn].name;
  statusEl.style.color=C[turn].color;
 }

 function cell(x,y,fill,stroke='#d5ddd6'){
  const s=600/15,px=x*s,py=y*s;
  ctx.fillStyle=fill;ctx.fillRect(px,py,s,s);
  ctx.strokeStyle=stroke;ctx.lineWidth=1.5;ctx.strokeRect(px,py,s,s);
 }

 function star(cx,cy,r,fill){
  ctx.beginPath();
  for(let i=0;i<10;i++){
   const a=-Math.PI/2+i*Math.PI/5;
   const rr=i%2===0?r:r*.45;
   const x=cx+Math.cos(a)*rr,y=cy+Math.sin(a)*rr;
   if(i===0)ctx.moveTo(x,y);else ctx.lineTo(x,y);
  }
  ctx.closePath();ctx.fillStyle=fill;ctx.fill();
 }

 function drawHome(index,x,y,color){
  const s=40;
  ctx.fillStyle=color.dark;ctx.fillRect(x*s,y*s,6*s,6*s);
  ctx.strokeStyle=color.pale;ctx.lineWidth=3;
  ctx.strokeRect(x*s+3,y*s+3,6*s-6,6*s-6);
  ctx.fillStyle='#f7f7ef';ctx.fillRect(x*s+0.85*s,y*s+0.85*s,4.3*s,4.3*s);
  homes[index].forEach(([hx,hy],i)=>{
   const px=(hx+.5)*s,py=(hy+.5)*s;
   ctx.beginPath();ctx.arc(px,py,13,0,Math.PI*2);
   ctx.fillStyle=color.color;ctx.fill();
   ctx.strokeStyle=color.dark;ctx.lineWidth=2;ctx.stroke();
   star(px,py,7,'#fff');
  });
 }

 function draw(){
  const s=40;
  ctx.clearRect(0,0,600,600);
  ctx.fillStyle='#fffdf4';ctx.fillRect(0,0,600,600);

  // چهار خانه بزرگ منچ
  drawHome(0,0,0,C[0]);
  drawHome(1,9,0,C[1]);
  drawHome(2,9,9,C[2]);
  drawHome(3,0,9,C[3]);

  // مسیر اصلی، خانه به خانه
  TRACK.forEach(([x,y],i)=>{
   let fill='#fffdf4';
   if(i===starts[0])fill=C[0].color;
   if(i===starts[1])fill=C[1].color;
   if(i===starts[2])fill=C[2].color;
   if(i===starts[3])fill=C[3].color;
   cell(x,y,fill,'#9ca99e');
  });

  // مسیرهای نهایی رنگی تا مرکز
  const lanes=[
   [[7,1],[7,2],[7,3],[7,4],[7,5],[7,6]],
   [[13,7],[12,7],[11,7],[10,7],[9,7],[8,7]],
   [[7,13],[7,12],[7,11],[7,10],[7,9],[7,8]],
   [[1,7],[2,7],[3,7],[4,7],[5,7],[6,7]]
  ];
  lanes.forEach((lane,i)=>{
   lane.forEach(([x,y])=>cell(x,y,C[i].pale,'#a3aaa3'));
   lane.slice(0,5).forEach(([x,y])=>star((x+.5)*s,(y+.5)*s,8,C[i].color));
  });

  // مرکز چهاررنگ
  const center=[
   [[6,6],[7,6],[6,7],[7,7]],
   [[8,6],[9,6],[8,7],[9,7]],
   [[8,8],[9,8],[8,9],[9,9]],
   [[6,8],[7,8],[6,9],[7,9]]
  ];
  const triangles=[
   {pts:[[240,240],[360,240],[300,300]],c:C[0].color},
   {pts:[[360,240],[360,360],[300,300]],c:C[1].color},
   {pts:[[360,360],[240,360],[300,300]],c:C[2].color},
   {pts:[[240,360],[240,240],[300,300]],c:C[3].color}
  ];
  triangles.forEach(t=>{
   ctx.beginPath();ctx.moveTo(...t.pts[0]);ctx.lineTo(...t.pts[1]);
   ctx.lineTo(...t.pts[2]);ctx.closePath();ctx.fillStyle=t.c;ctx.fill();
   ctx.strokeStyle='#fff';ctx.lineWidth=2;ctx.stroke();
  });
  star(300,300,15,'#fff');

  // خانه‌های امن ستاره‌دار روی مسیر
  [[6,2],[12,6],[8,12],[2,8]].forEach(([x,y],i)=>{
   star((x+.5)*s,(y+.5)*s,9,C[i].dark);
  });

  // مهره‌های داخل مسیر
  pawns.forEach((group,pi)=>{
   group.forEach((pawn,mi)=>{
    if(pawn.pos<0)return;
    const idx=(starts[pi]+pawn.pos)%TRACK.length;
    const [x,y]=TRACK[idx];
    const offsets=[[-7,-7],[7,-7],[-7,7],[7,7]][mi];
    const px=(x+.5)*s+offsets[0],py=(y+.5)*s+offsets[1];
    ctx.beginPath();ctx.arc(px,py,10,0,Math.PI*2);
    ctx.fillStyle=C[pi].color;ctx.fill();
    ctx.strokeStyle='#fff';ctx.lineWidth=2;ctx.stroke();
    star(px,py,5,'#fff');
   });
  });

  // قاب نهایی
  ctx.strokeStyle='#263c2b';ctx.lineWidth=4;ctx.strokeRect(2,2,596,596);
 }

 function playSound(freq=550){
  if(!sound)return;
  try{
   const A=window.AudioContext||window.webkitAudioContext;
   if(!A)return;
   const ac=new A(),o=ac.createOscillator(),g=ac.createGain();
   o.frequency.value=freq;g.gain.value=.04;
   o.connect(g);g.connect(ac.destination);o.start();
   o.stop(ac.currentTime+.08);o.onended=()=>ac.close();
  }catch(e){}
 }

 function nextTurn(){
  rolled=0;needMove=false;turn=(turn+1)%players;
  diceBtn.textContent='⚄';updateStatus();draw();
  note.textContent='تاس را بزن تا نوبتت شروع شود';
 }

 function roll(){
  if(locked||needMove)return;
  locked=true;
  diceBtn.textContent='🎲';
  diceBtn.style.transform='rotate(15deg)';
  playSound(650);
  setTimeout(()=>{
   rolled=1+Math.floor(Math.random()*6);
   locked=false;diceBtn.style.transform='';
   diceBtn.textContent=['⚀','⚁','⚂','⚃','⚄','⚅'][rolled-1];
   const movable=pawns[turn].some(p=>p.pos>=0||rolled===6);
   if(!movable){
    hint.textContent='حرکت ممکن نیست؛ نوبت بعدی';
    note.textContent='عدد تاس: '+rolled;
    setTimeout(nextTurn,800);return;
   }
   needMove=true;
   hint.textContent=rolled===6?'۶ آوردی! مهره‌ات را لمس کن':'یکی از مهره‌های خودت را لمس کن';
   note.textContent='عدد تاس: '+rolled+' — انتخاب مهره';
   playSound(450+rolled*60);
  },400);
 }

 function boardClick(e){
  if(!needMove)return;
  const rect=canvas.getBoundingClientRect();
  const x=(e.clientX-rect.left)*600/rect.width;
  const y=(e.clientY-rect.top)*600/rect.height;
  const group=pawns[turn],s=40;
  let chosen=-1,best=Infinity;

  group.forEach((p,i)=>{
   let px,py;
   if(p.pos<0){
    const [hx,hy]=homes[turn][i];px=(hx+.5)*s;py=(hy+.5)*s;
   }else{
    const [tx,ty]=TRACK[(starts[turn]+p.pos)%TRACK.length];
    const off=[[-7,-7],[7,-7],[-7,7],[7,7]][i];
    px=(tx+.5)*s+off[0];py=(ty+.5)*s+off[1];
   }
   const d=Math.hypot(x-px,y-py);
   if(d<30&&d<best){best=d;chosen=i;}
  });

  if(chosen<0){hint.textContent='یکی از مهره‌های خودت را انتخاب کن';return}
  const p=group[chosen];
  if(p.pos<0){
   if(rolled!==6){hint.textContent='برای بیرون آوردن مهره باید ۶ بیاوری';return}
   p.pos=0;
  }else{
   p.pos+=rolled;
   if(p.pos>=51)p.pos=51;
  }
  playSound(760);draw();needMove=false;

  if(rolled===6){
   rolled=0;diceBtn.textContent='⚄';
   hint.textContent='۶ آوردی! دوباره تاس بزن';
   note.textContent='نوبت اضافه';
  }else{
   hint.textContent='حرکت انجام شد';
   setTimeout(nextTurn,650);
  }
 }

 diceBtn.addEventListener('click',roll);
 canvas.addEventListener('click',boardClick);
 document.getElementById('newGame').addEventListener('click',reset);
 playerSelect.addEventListener('change',reset);
 document.getElementById('sound').addEventListener('click',e=>{
  sound=!sound;e.currentTarget.textContent=sound?'🔊 صدا روشن':'🔇 صدا خاموش';
  if(sound)playSound();
 });

 reset();
})();
</script>
</body>
</html>
