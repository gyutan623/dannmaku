<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>弾幕ゲーム</title>
<style>
  body {
    margin: 0;
    overflow: hidden;
    background: radial-gradient(circle, #000010, #000000);
  }
  canvas {
    display: block;
  }
</style>
</head>
<body>

<canvas id="game"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

// プレイヤー
let player = {
  x: canvas.width / 2,
  y: canvas.height - 100,
  size: 10,
  speed: 5
};

// 弾
let bullets = [];

// 入力
let keys = {};
document.addEventListener("keydown", e => keys[e.key] = true);
document.addEventListener("keyup", e => keys[e.key] = false);

// プレイヤー移動
function movePlayer() {
  if (keys["ArrowLeft"]) player.x -= player.speed;
  if (keys["ArrowRight"]) player.x += player.speed;
  if (keys["ArrowUp"]) player.y -= player.speed;
  if (keys["ArrowDown"]) player.y += player.speed;

  // 画面外防止
  player.x = Math.max(0, Math.min(canvas.width, player.x));
  player.y = Math.max(0, Math.min(canvas.height, player.y));
}

// 弾生成（円形弾幕）
let angle = 0;
function spawnBullets() {
  for (let i = 0; i < 12; i++) {
    let a = angle + (Math.PI * 2 / 12) * i;
    bullets.push({
      x: canvas.width / 2,
      y: 100,
      dx: Math.cos(a) * 3,
      dy: Math.sin(a) * 3,
      size: 5,
      color: `hsl(${Math.random()*360}, 80%, 60%)`
    });
  }
  angle += 0.1;
}

// 弾更新
function updateBullets() {
  bullets.forEach(b => {
    b.x += b.dx;
    b.y += b.dy;
  });
}

// 描画
function drawPlayer() {
  ctx.beginPath();
  ctx.arc(player.x, player.y, player.size, 0, Math.PI*2);
  ctx.fillStyle = "#ffffff";
  ctx.fill();
}

function drawBullets() {
  bullets.forEach(b => {
    ctx.beginPath();
    ctx.arc(b.x, b.y, b.size, 0, Math.PI*2);
    ctx.fillStyle = b.color;
    ctx.fill();
  });
}

// 当たり判定
function checkCollision() {
  for (let b of bullets) {
    let dx = player.x - b.x;
    let dy = player.y - b.y;
    let dist = Math.sqrt(dx*dx + dy*dy);

    if (dist < player.size + b.size) {
      alert("被弾した…");
      document.location.reload();
    }
  }
}

// ループ
let frame = 0;
function loop() {
  ctx.fillStyle = "rgba(0,0,0,0.2)";
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  movePlayer();

  if (frame % 30 === 0) spawnBullets();

  updateBullets();
  drawPlayer();
  drawBullets();
  checkCollision();

  frame++;
  requestAnimationFrame(loop);
}

loop();
</script>

</body>
</html>
