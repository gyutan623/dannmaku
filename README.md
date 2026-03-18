<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>弾幕ゲーム改</title>
<style>
body {
  margin: 0;
  overflow: hidden;
  background: radial-gradient(circle, #000010, #000000);
  color: white;
  font-family: sans-serif;
}
canvas {
  display: block;
}
#ui {
  position: absolute;
  top: 10px;
  left: 10px;
}
</style>
</head>
<body>

<canvas id="game"></canvas>
<div id="ui"></div>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
const ui = document.getElementById("ui");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

// ===== プレイヤー =====
let player = {
  x: canvas.width/2,
  y: canvas.height-100,
  size: 8,
  speed: 5,
  lives: 3
};

// ===== 状態 =====
let bullets = [];
let keys = {};
let frame = 0;
let score = 0;
let gameOver = false;

// ===== 入力 =====
document.addEventListener("keydown", e => keys[e.key] = true);
document.addEventListener("keyup", e => keys[e.key] = false);

// ===== プレイヤー移動 =====
function movePlayer() {
  let sp = keys["Shift"] ? 2 : player.speed;

  if (keys["ArrowLeft"]) player.x -= sp;
  if (keys["ArrowRight"]) player.x += sp;
  if (keys["ArrowUp"]) player.y -= sp;
  if (keys["ArrowDown"]) player.y += sp;

  player.x = Math.max(0, Math.min(canvas.width, player.x));
  player.y = Math.max(0, Math.min(canvas.height, player.y));
}

// ===== 弾幕① 回転弾 =====
let angle = 0;
function patternCircle() {
  for (let i=0;i<16;i++){
    let a = angle + (Math.PI*2/16)*i;
    bullets.push({
      x: canvas.width/2,
      y: 120,
      dx: Math.cos(a)*2,
      dy: Math.sin(a)*2,
      size: 5,
      color: `hsl(${frame%360},80%,60%)`
    });
  }
  angle += 0.2;
}

// ===== 弾幕② 狙い弾 =====
function patternAim() {
  let dx = player.x - canvas.width/2;
  let dy = player.y - 120;
  let dist = Math.sqrt(dx*dx+dy*dy);

  bullets.push({
    x: canvas.width/2,
    y: 120,
    dx: dx/dist*4,
    dy: dy/dist*4,
    size: 6,
    color: "#ff4d6d"
  });
}

// ===== 更新 =====
function updateBullets() {
  bullets.forEach(b=>{
    b.x += b.dx;
    b.y += b.dy;
  });
}

// ===== 描画 =====
function drawPlayer() {
  ctx.beginPath();
  ctx.arc(player.x, player.y, player.size, 0, Math.PI*2);
  ctx.fillStyle = "#ffffff";
  ctx.fill();
}

function drawBullets() {
  bullets.forEach(b=>{
    ctx.beginPath();
    ctx.arc(b.x,b.y,b.size,0,Math.PI*2);
    ctx.fillStyle = b.color;
    ctx.fill();
  });
}

// ===== 当たり判定 =====
function checkCollision() {
  for (let b of bullets) {
    let dx = player.x - b.x;
    let dy = player.y - b.y;
    let dist = Math.sqrt(dx*dx + dy*dy);

    if (dist < player.size + b.size) {
      player.lives--;
      bullets = []; // リセット
      if (player.lives <= 0) {
        gameOver = true;
      }
      break;
    }
  }
}

// ===== UI =====
function drawUI() {
  ui.innerHTML = `
    残機: ${player.lives} <br>
    スコア: ${score} <br>
    ${gameOver ? "Enterでコンティニュー" : ""}
  `;
}

// ===== コンティニュー =====
document.addEventListener("keydown", e=>{
  if (gameOver && e.key === "Enter") {
    player.lives = 3;
    score = 0;
    bullets = [];
    gameOver = false;
  }
});

// ===== ループ =====
function loop() {
  ctx.fillStyle = "rgba(0,0,0,0.25)";
  ctx.fillRect(0,0,canvas.width,canvas.height);

  if (!gameOver) {
    movePlayer();

    if (frame % 40 === 0) patternCircle();
    if (frame % 60 === 0) patternAim();

    updateBullets();
    checkCollision();

    score++;
  }

  drawPlayer();
  drawBullets();
  drawUI();

  frame++;
  requestAnimationFrame(loop);
}

loop();
</script>

</body>
</html>
