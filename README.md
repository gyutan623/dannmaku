<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>弾幕ゲームEX</title>
<style>
body {
  margin: 0;
  overflow: hidden;
  background: radial-gradient(circle, #020024, #000000);
  color: white;
  font-family: sans-serif;
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
  y: canvas.height-150,
  size: 4, // 当たり判定小さく
  lives: 3,
  invincible: 0,
  bombs: 3
};

// ===== 状態 =====
let bullets = [];
let frame = 0;
let score = 0;
let graze = 0;
let gameOver = false;

// ===== マウス操作 =====
canvas.addEventListener("mousemove", e=>{
  player.x = e.clientX;
  player.y = e.clientY;
});

// ===== 右クリックでボム =====
canvas.addEventListener("contextmenu", e=>{
  e.preventDefault();
  if(player.bombs > 0){
    bullets = [];
    player.bombs--;
  }
});

// ===== 弾幕① 回転 =====
let angle = 0;
function patternCircle(){
  for(let i=0;i<20;i++){
    let a = angle + i*(Math.PI*2/20);
    bullets.push({
      x: canvas.width/2,
      y: 120,
      dx: Math.cos(a)*2,
      dy: Math.sin(a)*2,
      size: 5,
      color: `hsl(${frame%360},80%,60%)`
    });
  }
  angle += 0.15;
}

// ===== 弾幕② 狙い =====
function patternAim(){
  let dx = player.x - canvas.width/2;
  let dy = player.y - 120;
  let d = Math.sqrt(dx*dx+dy*dy);

  bullets.push({
    x: canvas.width/2,
    y: 120,
    dx: dx/d*5,
    dy: dy/d*5,
    size: 6,
    color: "#ff4d6d"
  });
}

// ===== 弾幕③ 波 =====
function patternWave(){
  for(let i=0;i<10;i++){
    bullets.push({
      x: i*canvas.width/10,
      y: 0,
      dx: Math.sin(frame*0.1+i)*2,
      dy: 3,
      size: 5,
      color: "#4cc9f0"
    });
  }
}

// ===== 更新 =====
function update(){
  bullets.forEach(b=>{
    b.x += b.dx;
    b.y += b.dy;
  });

  if(player.invincible > 0) player.invincible--;
}

// ===== 描画 =====
function drawPlayer(){
  ctx.beginPath();
  ctx.arc(player.x, player.y, 6, 0, Math.PI*2);
  ctx.fillStyle = player.invincible ? "#ffff00" : "#ffffff";
  ctx.fill();

  // 当たり判定表示
  ctx.beginPath();
  ctx.arc(player.x, player.y, player.size, 0, Math.PI*2);
  ctx.fillStyle = "#ff0000";
  ctx.fill();
}

function drawBullets(){
  bullets.forEach(b=>{
    ctx.beginPath();
    ctx.arc(b.x,b.y,b.size,0,Math.PI*2);
    ctx.fillStyle = b.color;
    ctx.fill();
  });
}

// ===== 判定 =====
function collision(){
  for(let b of bullets){
    let dx = player.x - b.x;
    let dy = player.y - b.y;
    let dist = Math.sqrt(dx*dx+dy*dy);

    // かすり
    if(dist < player.size + b.size + 15){
      graze++;
      score += 2;
    }

    // 被弾
    if(dist < player.size + b.size && player.invincible === 0){
      player.lives--;
      player.invincible = 120;
      bullets = [];
      if(player.lives <= 0){
        gameOver = true;
      }
      break;
    }
  }
}

// ===== UI =====
function drawUI(){
  ui.innerHTML = `
    残機:${player.lives} 
    ボム:${player.bombs} <br>
    スコア:${score} 
    かすり:${graze} <br>
    ${gameOver ? "Enterで復活" : ""}
  `;
}

// ===== コンティニュー =====
document.addEventListener("keydown", e=>{
  if(gameOver && e.key==="Enter"){
    player.lives=3;
    player.bombs=3;
    score=0;
    graze=0;
    bullets=[];
    gameOver=false;
  }
});

// ===== ループ =====
function loop(){
  ctx.fillStyle="rgba(0,0,0,0.2)";
  ctx.fillRect(0,0,canvas.width,canvas.height);

  if(!gameOver){
    if(frame%40===0) patternCircle();
    if(frame%70===0) patternAim();
    if(frame%100===0) patternWave();

    update();
    collision();
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
