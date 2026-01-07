<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Plateforme Pixel Mobile</title>

<style>
body {
  margin: 0;
  overflow: hidden;
  background: #87ceeb;
  font-family: Arial, sans-serif;
  touch-action: none;
}

canvas {
  display: none;
  background: #87ceeb;
  image-rendering: pixelated;
}

/* ===== MENU ===== */
.menu {
  position: absolute;
  inset: 0;
  background: #87ceeb;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.menu h1 { font-size: 42px; }
.menu button {
  font-size: 26px;
  padding: 18px 40px;
  margin-top: 20px;
}

/* ===== CONTROLES TABLETTE ===== */
.control {
  position: fixed;
  bottom: 30px;
  width: 140px;
  height: 140px;
  background: rgba(0,0,0,0.55);
  color: white;
  font-size: 64px;
  text-align: center;
  line-height: 140px;
  border-radius: 30px;
  user-select: none;
}

#leftBtn { left: 30px; }
#rightBtn { left: 200px; }

/* ===== BOUTONS FIN ===== */
#replayBtn, #menuBtn {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  padding: 18px 40px;
  font-size: 24px;
  display: none;
}

#replayBtn { top: 60%; background: red; color: white; }
#menuBtn { top: 72%; background: green; color: white; }
</style>
</head>

<body>

<div class="menu" id="menu">
  <h1>Plateforme Pixel</h1>
  <p>Meilleur score : <span id="bestScore">0</span></p>
  <button id="startBtn">Jouer</button>
</div>

<canvas id="game"></canvas>

<div id="leftBtn" class="control">←</div>
<div id="rightBtn" class="control">→</div>

<button id="replayBtn">Rejouer</button>
<button id="menuBtn">Menu</button>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
canvas.width = innerWidth;
canvas.height = innerHeight;

let bestScore = localStorage.getItem("bestScore") || 0;
bestScore = Number(bestScore);
document.getElementById("bestScore").textContent = bestScore;

let player, platforms, score, gameOver, left, right;

/* ===== INIT ===== */
function initGame() {

  platforms = [{
    x: canvas.width/2 - 140,
    y: canvas.height - 120,
    w: 280,
    h: 25
  }];

  for (let i = 1; i < 10; i++) {
    platforms.push({
      x: Math.random() * (canvas.width - 160),
      y: canvas.height - 120 - i * 140,
      w: 160,
      h: 22
    });
  }

  player = {
    x: platforms[0].x + platforms[0].w/2 - 16,
    y: platforms[0].y - 60,
    dy: 0
  };

  score = 0;
  gameOver = false;
  left = right = false;

  replayBtn.style.display = "none";
  menuBtn.style.display = "none";
}

/* ===== PERSONNAGE ===== */
function drawPlayer(x, y) {
  const p = 4;
  ctx.fillStyle="#000"; ctx.fillRect(x+4*p,y,6*p,2*p);
  ctx.fillStyle="#f1c27d"; ctx.fillRect(x+4*p,y+2*p,6*p,4*p);
  ctx.fillStyle="#8b4513"; ctx.fillRect(x+4*p,y+4*p,6*p,3*p);
  ctx.fillStyle="#1e90ff"; ctx.fillRect(x+3*p,y+7*p,8*p,4*p);
  ctx.fillStyle="#000";
  ctx.fillRect(x+3*p,y+11*p,3*p,4*p);
  ctx.fillRect(x+8*p,y+11*p,3*p,4*p);
}

/* ===== JEU ===== */
function update() {
  if (gameOver) {
    ctx.fillStyle="black";
    ctx.font="40px Arial";
    ctx.textAlign="center";
    ctx.fillText("MORT",canvas.width/2,canvas.height/2-40);
    ctx.fillText("Score : "+Math.floor(score),canvas.width/2,canvas.height/2);
    replayBtn.style.display="block";
    menuBtn.style.display="block";

    if (score > bestScore) {
      bestScore = Math.floor(score);
      localStorage.setItem("bestScore", bestScore);
      document.getElementById("bestScore").textContent = bestScore;
    }
    return;
  }

  ctx.clearRect(0,0,canvas.width,canvas.height);

  if (left) player.x -= 6;
  if (right) player.x += 6;

  player.dy += 0.7;
  player.y += player.dy;

  platforms.forEach(p=>{
    if(player.dy>0 &&
       player.x+32>p.x &&
       player.x<p.x+p.w &&
       player.y+60>p.y &&
       player.y+60<p.y+p.h){
      player.dy = -16;
    }
  });

  if (player.y < canvas.height/2) {
    let d = canvas.height/2 - player.y;
    player.y = canvas.height/2;
    platforms.forEach(p=>p.y+=d);
    score += d;
  }

  platforms = platforms.filter(p=>p.y<canvas.height);
  while(platforms.length<10){
    let h=Math.min(...platforms.map(p=>p.y));
    platforms.push({
      x:Math.random()*(canvas.width-160),
      y:h-140,
      w:160,
      h:22
    });
  }

  ctx.fillStyle="green";
  platforms.forEach(p=>ctx.fillRect(p.x,p.y,p.w,p.h));
  drawPlayer(player.x,player.y);

  ctx.fillStyle="black";
  ctx.font="22px Arial";
  ctx.fillText("Score : "+Math.floor(score),20,40);

  if (player.y > canvas.height) gameOver = true;
  requestAnimationFrame(update);
}

/* ===== CONTROLES TACTILES ===== */
leftBtn.ontouchstart = ()=>left=true;
leftBtn.ontouchend = ()=>left=false;
rightBtn.ontouchstart = ()=>right=true;
rightBtn.ontouchend = ()=>right=false;

startBtn.onclick = ()=>{
  menu.style.display="none";
  canvas.style.display="block";
  initGame();
  update();
};

replayBtn.onclick = ()=>{initGame();update();};
menuBtn.onclick = ()=>{
  canvas.style.display="none";
  menu.style.display="flex";
};
</script>

</body>
</html>
