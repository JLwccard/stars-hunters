<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Catch the Stars ✨ - Shadowling</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');
    :root{
      --bg:#0f1226; --panel:#171a35; --text:#eaf0ff; --accent:#7aa2ff; --muted:#9aa4c7;
      --good:#5ce1a6; --bad:#ff6b6b; --gold:#ffd166;
    }
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;font-family:'Press Start 2P',cursive}
    body{
      display:grid;place-items:center;
      background:linear-gradient(180deg,#000010 0%,#0a0a25 40%,#020212 100%);
      color:var(--text);
    }
    .wrap{width:min(1100px, 96vw);display:grid;grid-template-columns:1fr 320px;gap:16px;align-items:start}
    canvas{width:100%;height:600px;background:radial-gradient(circle at center, #0f0f25 0%, #000010 100%);border:1px solid rgba(255,255,255,.1);border-radius:18px;box-shadow:0 20px 40px rgba(0,0,0,.35);image-rendering:pixelated;}
    aside{background:var(--panel);border:1px solid rgba(255,255,255,.08);padding:14px 14px 8px;border-radius:18px;box-shadow:0 12px 30px rgba(0,0,0,.25);position:relative}
    h1{font-weight:800;letter-spacing:0.3px;font-size:16px;margin:0 0 6px;text-transform:uppercase}
    .sub{font-size:10px;color:var(--muted);margin-bottom:10px;line-height:1.4}
    .stat{background:rgba(255,255,255,.04);padding:8px 10px;border-radius:12px;border:1px solid rgba(255,255,255,.07);margin-bottom:6px}
    .stat h3{font-size:9px;color:var(--muted);margin:0 0 4px}
    .stat .v{font-size:14px;font-weight:700}
    button{cursor:pointer;border-radius:12px;padding:9px 12px;border:1px solid rgba(255,255,255,.12);background:rgba(255,255,255,.06);color:var(--text);font-weight:600;margin:4px;font-family:'Press Start 2P',cursive;font-size:10px;transition:all .2s ease}
    button:hover{transform:translateY(-2px) scale(1.05);} 
    button.primary{background:linear-gradient(180deg, var(--accent), #4b7cff);border-color:rgba(0,0,0,.2);color:#fff}
    footer{font-size:9px;color:var(--muted);margin-top:6px;text-align:center;line-height:1.4}
  </style>
</head>
<body>
  <div class="wrap">
    <canvas id="gameCanvas" width="720" height="600"></canvas>
    <aside>
      <h1>Catch the Stars ✨</h1>
      <div class="sub">Objetivo: Pegue 20 estrelas sem ser pego pelo <b>Shadowling</b> 👾! Ao atingir certas pontuações, ferramentas especiais aparecerão para ajudar você! Após 50 estrelas, obstáculos perigosos surgirão e o jogo ficará cada vez mais difícil!</div>

      <div class="stat"><h3>Pontuação</h3><div class="v" id="score">0</div></div>
      <div class="stat"><h3>Status</h3><div class="v" id="status">Aguardando...</div></div>

      <button class="primary" onclick="startGame()">Iniciar</button>
      <button onclick="resetGame()">Resetar</button>

      <footer>
        Controles: <b>WASD</b> ou <b>Setas</b>. Fuja do <b>Shadowling</b> 👾, colete as estrelas amarelas ✨ e sobreviva aos desafios!
      </footer>
    </aside>
  </div>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

let player = { x: 360, y: 300, size: 24, speed: 4, frame:0, anim:0 };
let stars = [];
let shadowling = { x: 50, y: 50, size: 32, speed: 2, eyeBlink: 0, frame:0, anim:0 };
let obstacles = [];
let tools = [];
let score = 0;
let gameRunning = false;
let keys = {};
let objective = 20;

// sons
let starSound = new Audio("https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg");
let deathSound = new Audio("https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg");

// fundo estrelado animado
let nightStars = [];
for (let i=0;i<100;i++){
  nightStars.push({x:Math.random()*canvas.width,y:Math.random()*canvas.height,r:Math.random()*1.5, twinkle:Math.random()*100});
}

document.addEventListener("keydown", e => keys[e.key] = true);
document.addEventListener("keyup", e => keys[e.key] = false);

function spawnStar() {
  let star = { x: Math.random()*(canvas.width-20), y: Math.random()*(canvas.height-20), size: 8 };
  stars.push(star);
}

function spawnObstacle() {
  let obstacle = {
    x: Math.random() * (canvas.width - 40),
    y: Math.random() * (canvas.height - 40),
    size: 30,
    dx: (Math.random() < 0.5 ? -1 : 1) * (1 + Math.random()*1.5),
    dy: (Math.random() < 0.5 ? -1 : 1) * (1 + Math.random()*1.5)
  };
  obstacles.push(obstacle);
}

function spawnTool(type){
  let tool = { x: Math.random()*(canvas.width-30), y: Math.random()*(canvas.height-30), size: 15, type: type };
  tools.push(tool);
}

function movePlayer(){
  if (keys["ArrowUp"]||keys["w"]) player.y -= player.speed;
  if (keys["ArrowDown"]||keys["s"]) player.y += player.speed;
  if (keys["ArrowLeft"]||keys["a"]) player.x -= player.speed;
  if (keys["ArrowRight"]||keys["d"]) player.x += player.speed;
  if (player.x<0) player.x=0;
  if (player.y<0) player.y=0;
  if (player.x+player.size>canvas.width) player.x=canvas.width-player.size;
  if (player.y+player.size>canvas.height) player.y=canvas.height-player.size;
  player.frame++;
  player.anim = Math.sin(player.frame*0.2)*3; // animação de flutuar
}

function moveShadowling(){
  if (shadowling.x<player.x) shadowling.x+=shadowling.speed;
  if (shadowling.x>player.x) shadowling.x-=shadowling.speed;
  if (shadowling.y<player.y) shadowling.y+=shadowling.speed;
  if (shadowling.y>player.y) shadowling.y-=shadowling.speed;
  shadowling.eyeBlink=(shadowling.eyeBlink+1)%120;
  shadowling.frame++;
  shadowling.anim = Math.sin(shadowling.frame*0.25)*2; // animação de ondulação
}

function moveObstacles(){
  obstacles.forEach(o=>{
    o.x+=o.dx;
    o.y+=o.dy;
    if(o.x<=0||o.x+o.size>=canvas.width) o.dx*=-1;
    if(o.y<=0||o.y+o.size>=canvas.height) o.dy*=-1;
  });
}

function checkCollisions(){
  for(let i=stars.length-1;i>=0;i--){
    let s=stars[i];
    if(player.x<s.x+s.size&&player.x+player.size>s.x&&player.y<s.y+s.size&&player.y+player.size>s.y){
      stars.splice(i,1);
      score++;
      starSound.currentTime=0; starSound.play();
      document.getElementById("score").innerText=score;
      if(score===5) spawnTool("shield");
      if(score===10) spawnTool("speed");
      if(score===15) spawnTool("freeze");
      if(score===50){for(let i=0;i<4;i++) spawnObstacle();}
      if(score%20===0){shadowling.speed+=0.5;player.speed+=0.2;spawnObstacle();}
      if(score>=objective){gameOver(true);}
    }
  }
  if(player.x<shadowling.x+shadowling.size&&player.x+player.size>shadowling.x&&player.y<shadowling.y+shadowling.size&&player.y+player.size>shadowling.y){gameOver(false);}
  obstacles.forEach(o=>{if(player.x<o.x+o.size&&player.x+player.size>o.x&&player.y<o.y+o.size&&player.y+player.size>o.y){gameOver(false);}});
  for(let i=tools.length-1;i>=0;i--){let t=tools[i];if(player.x<t.x+t.size&&player.x+player.size>t.x&&player.y<t.y+t.size&&player.y+player.size>t.y){applyTool(t.type);tools.splice(i,1);}}
}

function drawPlayer(){
  ctx.save();
  ctx.translate(player.x+player.size/2, player.y+player.size/2 + player.anim);
  ctx.fillStyle="#4ba3ff";
  ctx.beginPath();
  ctx.moveTo(-player.size/2, player.size/2);
  ctx.lineTo(0, -player.size/2);
  ctx.lineTo(player.size/2, player.size/2);
  ctx.closePath();
  ctx.fill();
  // efeito de propulsão
  ctx.fillStyle="#ffdd55";
  ctx.beginPath();
  ctx.moveTo(-5, player.size/2);
  ctx.lineTo(0, player.size/2+8+Math.sin(player.frame*0.5)*3);
  ctx.lineTo(5, player.size/2);
  ctx.closePath();
  ctx.fill();
  ctx.restore();
}

function drawStars(){
  ctx.fillStyle="#ffd166";
  stars.forEach(s=>{
    ctx.save();
    ctx.translate(s.x+5, s.y+5);
    ctx.rotate((Date.now()/200)% (2*Math.PI));
    ctx.beginPath();
    for(let i=0;i<5;i++){
      let angle=(i*72-90)*Math.PI/180;
      let x=Math.cos(angle)*s.size;
      let y=Math.sin(angle)*s.size;
      ctx.lineTo(x,y);
    }
    ctx.closePath();
    ctx.fill();
    ctx.restore();
  });
}

function drawShadowling(){
  ctx.save();
  ctx.translate(shadowling.x+shadowling.size/2, shadowling.y+shadowling.size/2 + shadowling.anim);
  ctx.fillStyle="#aa2222";
  ctx.fillRect(-shadowling.size/2, -shadowling.size/2, shadowling.size, shadowling.size);
  // olhos
  if(shadowling.eyeBlink<100){
    ctx.fillStyle="#fff";
    ctx.beginPath();ctx.arc(-6,-4,4,0,Math.PI*2);ctx.fill();
    ctx.beginPath();ctx.arc(6,-4,4,0,Math.PI*2);ctx.fill();
    ctx.fillStyle="#000";
    ctx.beginPath();ctx.arc(-6,-4,2,0,Math.PI*2);ctx.fill();
    ctx.beginPath();ctx.arc(6,-4,2,0,Math.PI*2);ctx.fill();
  }
  // boca assustadora
  ctx.strokeStyle="#000";
  ctx.lineWidth=3;
  ctx.beginPath();
  ctx.moveTo(-8,8);
  ctx.lineTo(8,8);
  ctx.stroke();
  ctx.restore();
}

function drawObstacles(){
  ctx.fillStyle="#c0392b";
  obstacles.forEach(o=>{ctx.fillRect(o.x,o.y,o.size,o.size);});
}

function drawTools(){
  tools.forEach(t=>{
    if(t.type==="shield") ctx.fillStyle="#5ce1a6";
    if(t.type==="speed") ctx.fillStyle="#7aa2ff";
    if(t.type==="freeze") ctx.fillStyle="#ffd166";
    ctx.beginPath();
    ctx.arc(t.x+t.size/2, t.y+t.size/2, t.size/2, 0, Math.PI*2);
    ctx.fill();
  });
}

function drawBackground(){
  ctx.fillStyle="#000010";
  ctx.fillRect(0,0,canvas.width,canvas.height);
  ctx.fillStyle="white";
  nightStars.forEach(ns=>{
    ns.twinkle+=0.05;
    let r=ns.r*(1+0.3*Math.sin(ns.twinkle));
    ctx.beginPath();ctx.arc(ns.x,ns.y,r,0,Math.PI*2);ctx.fill();
  });
}

function applyTool(type){
  if(type==="shield"){
    document.getElementById("status").innerText="🛡 Escudo ativado!";
    let oldGameOver=gameOver;
    gameOver=function(win){if(!win){document.getElementById("status").innerText="Escudo quebrou!";gameOver=oldGameOver;} else {oldGameOver(true);}};
  }
  if(type==="speed"){
    document.getElementById("status").innerText="⚡ Velocidade!";
    player.speed+=2;setTimeout(()=>{player.speed-=2;},5000);
  }
  if(type==="freeze"){
    document.getElementById("status").innerText="❄ Shadowling congelado!";
    let oldSpeed=shadowling.speed;shadowling.speed=0;
    setTimeout(()=>{shadowling.speed=oldSpeed;},4000);
  }
}

function update(){
  if(!gameRunning) return;
  ctx.clearRect(0,0,canvas.width,canvas.height);
  drawBackground();
  movePlayer();moveShadowling();moveObstacles();checkCollisions();
  drawPlayer();drawStars();drawShadowling();drawObstacles();drawTools();
  if(Math.random()<0.02) spawnStar();
  requestAnimationFrame(update);
}

function startGame(){
  resetGame();gameRunning=true;
  document.getElementById("status").innerText="Jogando...";
  update();
}

function resetGame(){
  score=0;stars=[];obstacles=[];tools=[];player.x=360;player.y=300;shadowling.x=50;shadowling.y=50;shadowling.speed=2;gameRunning=false;
  document.getElementById("score").innerText="0";
  document.getElementById("status").innerText="Aguardando...";
}

function gameOver(win){
  gameRunning=false;
  if(win){document.getElementById("status").innerText="🎉 Você venceu! Pegou 20 estrelas!";}
  else{document.getElementById("status").innerText="💀 Você perdeu!"; deathSound.currentTime=0; deathSound.play();}
}
</script>
</body>
</html>
