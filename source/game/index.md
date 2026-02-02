---
title: 摸鱼时间
date: 2026-02-02 12:00:00
---

<style>
.game-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  font-family: Arial, sans-serif;
}
#gameCanvas {
  border: 2px solid #535353;
  background: #f7f7f7;
  display: block;
  margin: 20px auto;
}
.score-board {
  font-size: 20px;
  margin: 10px 0;
  color: #535353;
}
.game-info {
  color: #888;
  margin: 10px 0;
  text-align: center;
}
</style>

<div class="game-wrapper">
  <div class="score-board">得分: <span id="score">0</span></div>
  <canvas id="gameCanvas" width="600" height="200"></canvas>
  <div class="game-info">跳跃: 空格 / ↑ / W / J | 下蹲: ↓ / S / K</div>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const scoreElement = document.getElementById('score');

let score = 0;
let gameSpeed = 3;
let gravity = 0.6;
let isGameOver = false;

// 恐龙
const dino = {
  x: 50,
  y: 150,
  width: 20,
  height: 40,
  dy: 0,
  jumpPower: -12,
  grounded: false,
  ducking: false
};

// 障碍物数组
let obstacles = [];
let frameCount = 0;
let obstacleFrequency = 100;

// 键盘控制
const keys = {};
document.addEventListener('keydown', (e) => {
  keys[e.code] = true;
  // 跳跃键: 空格, ↑, W, J
  if ((e.code === 'Space' || e.code === 'ArrowUp' || e.code === 'KeyW' || e.code === 'KeyJ') && dino.grounded && !isGameOver) {
    dino.dy = dino.jumpPower;
    dino.grounded = false;
  }
  // 防止空格键滚动页面
  if (e.code === 'Space' || e.code === 'ArrowUp' || e.code === 'ArrowDown') {
    e.preventDefault();
  }
});

document.addEventListener('keyup', (e) => {
  keys[e.code] = false;
});

// 创建障碍物
function createObstacle() {
  const types = ['cactus', 'bird'];
  const type = types[Math.floor(Math.random() * types.length)];
  
  obstacles.push({
    x: canvas.width,
    y: type === 'bird' ? 120 : 150,
    width: 20,
    height: type === 'bird' ? 20 : 40,
    type: type
  });
}

// 绘制恐龙
function drawDino() {
  ctx.fillStyle = '#535353';
  
  // 下蹲键: ↓, S, K
  if ((keys['ArrowDown'] || keys['KeyS'] || keys['KeyK']) && dino.grounded) {
    // 下蹲姿势
    ctx.fillRect(dino.x, dino.y + 20, dino.width + 10, dino.height - 20);
    dino.ducking = true;
  } else {
    // 正常姿势
    ctx.fillRect(dino.x, dino.y, dino.width, dino.height);
    dino.ducking = false;
  }
  
  // 眼睛
  ctx.fillStyle = '#fff';
  ctx.fillRect(dino.x + 12, dino.y + 5, 3, 3);
}

// 绘制障碍物
function drawObstacles() {
  ctx.fillStyle = '#535353';
  obstacles.forEach(obs => {
    if (obs.type === 'cactus') {
      ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
    } else {
      // 鸟
      ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
      ctx.fillRect(obs.x - 5, obs.y + 5, 10, 10);
    }
  });
}

// 更新游戏
function update() {
  if (isGameOver) return;
  
  frameCount++;
  
  // 更新恐龙位置
  if (!dino.grounded) {
    dino.dy += gravity;
    dino.y += dino.dy;
    
    if (dino.y >= 150) {
      dino.y = 150;
      dino.dy = 0;
      dino.grounded = true;
    }
  }
  
  // 创建新障碍物
  if (frameCount % obstacleFrequency === 0) {
    createObstacle();
    if (obstacleFrequency > 50) obstacleFrequency -= 2;
  }
  
  // 更新障碍物
  obstacles.forEach((obs, index) => {
    obs.x -= gameSpeed;
    
    // 移除屏幕外的障碍物
    if (obs.x + obs.width < 0) {
      obstacles.splice(index, 1);
      score++;
      scoreElement.textContent = score;
      
      // 增加难度
      if (score % 10 === 0 && gameSpeed < 8) {
        gameSpeed += 0.5;
      }
    }
    
    // 碰撞检测
    const dinoHeight = dino.ducking ? dino.height - 20 : dino.height;
    const dinoY = dino.ducking ? dino.y + 20 : dino.y;
    const dinoWidth = dino.ducking ? dino.width + 10 : dino.width;
    
    if (
      dino.x < obs.x + obs.width &&
      dino.x + dinoWidth > obs.x &&
      dinoY < obs.y + obs.height &&
      dinoY + dinoHeight > obs.y
    ) {
      gameOver();
    }
  });
}

// 绘制游戏
function draw() {
  // 清空画布
  ctx.fillStyle = '#f7f7f7';
  ctx.fillRect(0, 0, canvas.width, canvas.height);
  
  // 绘制地面
  ctx.strokeStyle = '#535353';
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.moveTo(0, 190);
  ctx.lineTo(canvas.width, 190);
  ctx.stroke();
  
  drawDino();
  drawObstacles();
  
  if (isGameOver) {
    ctx.fillStyle = '#535353';
    ctx.font = '30px Arial';
    ctx.textAlign = 'center';
    ctx.fillText('游戏结束!', canvas.width / 2, 80);
    ctx.font = '16px Arial';
    ctx.fillText('按空格键重新开始', canvas.width / 2, 110);
  }
}

// 游戏结束
function gameOver() {
  isGameOver = true;
  document.addEventListener('keydown', restart);
}

// 重新开始
function restart(e) {
  if (e.code === 'Space' && isGameOver) {
    isGameOver = false;
    score = 0;
    gameSpeed = 3;
    obstacleFrequency = 100;
    obstacles = [];
    frameCount = 0;
    dino.y = 150;
    dino.dy = 0;
    dino.grounded = true;
    scoreElement.textContent = score;
    document.removeEventListener('keydown', restart);
  }
}

// 游戏循环
function gameLoop() {
  update();
  draw();
  requestAnimationFrame(gameLoop);
}

gameLoop();
</script>

> 🦖 经典的 Chrome 小恐龙跳跃游戏！躲避仙人掌和飞鸟，看你能坚持多久！
