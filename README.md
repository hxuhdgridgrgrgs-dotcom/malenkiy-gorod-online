<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>Маленький город — Online</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#17212b;color:white;font-family:Arial;overflow:hidden}
header{height:54px;background:#101820;display:flex;justify-content:space-between;align-items:center;padding:12px}
canvas{display:block;width:100vw;height:calc(100dvh - 166px);background:#6eaa62;touch-action:none}
footer{height:112px;background:#101820;display:flex;align-items:center;justify-content:space-around}
.pad{display:grid;grid-template-columns:52px 52px 52px;grid-template-rows:40px 40px;gap:4px}
button{border:0;border-radius:11px;background:#2b3b49;color:white;font-size:22px}
.up{grid-column:2}
.left{grid-column:1;grid-row:2}
.down{grid-column:2;grid-row:2}
.right{grid-column:3;grid-row:2}
#turbo{width:80px;height:80px;border-radius:50%;background:#cf5148;font-size:14px}
</style>
</head>
<body>
<header>
  <b>🏙️ Маленький город — Online</b>
  <span id="status">Подключение...</span>
</header>

<canvas id="game"></canvas>

<footer>
  <div class="pad">
    <button class="up" data-key="up">▲</button>
    <button class="left" data-key="left">◀</button>
    <button class="down" data-key="down">▼</button>
    <button class="right" data-key="right">▶</button>
  </div>
  <span id="players">Игроков: 1</span>
  <button id="turbo">ТУРБО</button>
</footer>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

let W, H;
function resize(){
  const r = canvas.getBoundingClientRect();
  W = r.width;
  H = r.height;
  canvas.width = W * devicePixelRatio;
  canvas.height = H * devicePixelRatio;
  ctx.setTransform(devicePixelRatio,0,0,devicePixelRatio,0,0);
}
resize();
addEventListener("resize",resize);

const keys = {};
const me = {x:W/2,y:H/2,angle:0};
const others = new Map();
let myId = null;
let turbo = false;

const protocol = location.protocol === "https:" ? "wss" : "ws";
const socket = new WebSocket(`${protocol}://${location.host}`);

socket.onopen = () => {
  document.getElementById("status").textContent = "🟢 Онлайн";
};

socket.onclose = () => {
  document.getElementById("status").textContent = "🔴 Нет связи";
};

socket.onmessage = e => {
  const msg = JSON.parse(e.data);

  if(msg.type === "welcome"){
    myId = msg.id;
    me.x = msg.player.x;
    me.y = msg.player.y;
  }

  if(msg.type === "state"){
    others.clear();
    msg.players.forEach(p => others.set(p.id,p));
    document.getElementById("players").textContent =
      "Игроков: " + msg.players.length;
  }
};

document.querySelectorAll("[data-key]").forEach(button => {
  const key = button.dataset.key;

  button.addEventListener("pointerdown",e=>{
    e.preventDefault();
    keys[key] = true;
  });

  ["pointerup","pointercancel","pointerleave"].forEach(event=>{
    button.addEventListener(event,()=>keys[key]=false);
  });
});

addEventListener("keydown",e=>{
  const map = {
    ArrowUp:"up",w:"up",
    ArrowDown:"down",s:"down",
    ArrowLeft:"left",a:"left",
    ArrowRight:"right",d:"right"
  };
  if(map[e.key]) keys[map[e.key]]=true;
});

addEventListener("keyup",e=>{
  const map = {
    ArrowUp:"up",w:"up",
    ArrowDown:"down",s:"down",
    ArrowLeft:"left",a:"left",
    ArrowRight:"right",d:"right"
  };
  if(map[e.key]) keys[map[e.key]]=false;
});

const turboButton = document.getElementById("turbo");

turboButton.addEventListener("pointerdown",e=>{
  e.preventDefault();
  turbo=true;
});

["pointerup","pointercancel","pointerleave"].forEach(event=>{
  turboButton.addEventListener(event,()=>turbo=false);
});

function drawRoad(){
  ctx.fillStyle="#6eaa62";
  ctx.fillRect(0,0,W,H);

  ctx.fillStyle="#454d52";
  ctx.fillRect(W*.18,0,W*.64,H);
  ctx.fillRect(0,H*.36,W,H*.28);

  ctx.strokeStyle="#e6dfb6";
  ctx.lineWidth=3;
  ctx.setLineDash([20,18]);

  ctx.beginPath();
  ctx.moveTo(W/2,0);
  ctx.lineTo(W/2,H);
  ctx.moveTo(0,H/2);
  ctx.lineTo(W,H/2);
  ctx.stroke();

  ctx.setLineDash([]);

  for(let i=0;i<10;i++){
    const x=(i*127+35)%W;
    const y=(i*83+25)%H;

    if(!(x>W*.16&&x<W*.84)&&!(y>H*.34&&y<H*.66)){
      ctx.fillStyle="#4d9149";
      ctx.beginPath();
      ctx.arc(x,y,13,0,7);
      ctx.fill();
    }
  }
}

function drawCar(p,isMe){
  const x=p.x-me.x+W/2;
  const y=p.y-me.y+H/2;

  ctx.save();
  ctx.translate(x,y);
  ctx.rotate(p.angle||0);

  ctx.fillStyle="#20262b";
  ctx.fillRect(-17,-11,34,22);

  ctx.fillStyle=isMe?"#f0b63f":p.color;
  ctx.fillRect(-14,-9,28,18);

  ctx.fillStyle="#bce4e9";
  ctx.fillRect(-4,-7,10,14);

  ctx.restore();

  ctx.fillStyle="white";
  ctx.font="12px Arial";
  ctx.textAlign="center";
  ctx.fillText(isMe?"Ты":p.name||"Игрок",x,y-19);
}

let lastSend=0;

function frame(t){
  let dx=(keys.right?1:0)-(keys.left?1:0);
  let dy=(keys.down?1:0)-(keys.up?1:0);

  if(dx||dy){
    const length=Math.hypot(dx,dy);
    dx/=length;
    dy/=length;

    me.angle=Math.atan2(dy,dx);

    const speed=turbo?5:3;
    me.x+=dx*speed;
    me.y+=dy*speed;
  }

  drawRoad();

  for(const p of others.values()){
    if(p.id!==myId) drawCar(p,false);
  }

  drawCar(me,true);

  if(socket.readyState===WebSocket.OPEN&&t-lastSend>50){
    socket.send(JSON.stringify({
      type:"move",
      x:me.x,
      y:me.y,
      angle:me.angle
    }));
    lastSend=t;
  }

  requestAnimationFrame(frame);
}

requestAnimationFrame(frame);
</script>
</body>
</html>

