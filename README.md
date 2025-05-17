# N
Game
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8" />
<title>Прототип эволюционной игры с эволюцией</title>
<style>
  body { margin: 0; overflow: hidden; background: #222; }
  canvas { display: block; margin: 0 auto; background: #333; }
  #info { color: #eee; text-align: center; font-family: sans-serif; margin-top: 10px; }
</style>
</head>
<body>
<canvas id="game" width="500" height="500"></canvas>
<div id="info">Собери пищу, чтобы развиваться!</div>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

const WIDTH = canvas.width;
const HEIGHT = canvas.height;

// Игрок - существо
const player = {
  x: WIDTH / 2,
  y: HEIGHT / 2,
  radius: 15,
  color: 'lime',
  foodCollected: 0,
  level: 1,
  limbs: 0  // Количество дополнительных частей тела
};

// Пища
const foods = [];
const FOOD_COUNT = 20;

function randomPosition() {
  return {
    x: Math.random() * (WIDTH - 20) + 10,
    y: Math.random() * (HEIGHT - 20) + 10
  };
}

for (let i = 0; i < FOOD_COUNT; i++) {
  foods.push({
    x: randomPosition().x,
    y: randomPosition().y,
    radius: 7,
    color: 'orange'
  });
}

// Обновление положения игрока по движению мыши
canvas.addEventListener('mousemove', (e) => {
  const rect = canvas.getBoundingClientRect();
  player.x = e.clientX - rect.left;
  player.y = e.clientY - rect.top;
});

// Проверка столкновения
function isColliding(a, b) {
  const dx = a.x - b.x;
  const dy = a.y - b.y;
  const distance = Math.sqrt(dx * dx + dy * dy);
  return distance < a.radius + b.radius;
}

// Обновление состояния игры
function update() {
  // Проверяем, съел ли игрок пищу
  for (let i = foods.length - 1; i >= 0; i--) {
    if (isColliding(player, foods[i])) {
      foods.splice(i, 1);
      player.foodCollected++;
      if (player.foodCollected % 5 === 0) {
        player.level++;
        player.radius += 3;
        player.limbs++;  // Добавляем новую часть тела при каждом уровне
      }
    }
  }

  // Если пища закончилась, добавляем новую
  if (foods.length < FOOD_COUNT) {
    foods.push({
      x: randomPosition().x,
      y: randomPosition().y,
      radius: 7,
      color: 'orange'
    });
  }
}

// Рисуем объект на канвасе
function drawCircle(obj) {
  ctx.beginPath();
  ctx.arc(obj.x, obj.y, obj.radius, 0, Math.PI * 2);
  ctx.fillStyle = obj.color;
  ctx.fill();
  ctx.closePath();
}

// Рисуем щупальца (или дополнительные части тела)
function drawLimbs(player) {
  const limbsCount = player.limbs;
  const angleStep = (Math.PI * 2) / limbsCount;

  for (let i = 0; i < limbsCount; i++) {
    const angle = i * angleStep;
    const limbLength = player.radius + 10;
    const limbX = player.x + Math.cos(angle) * limbLength;
    const limbY = player.y + Math.sin(angle) * limbLength;

    ctx.beginPath();
    ctx.strokeStyle = 'lime';
    ctx.lineWidth = 4;
    ctx.moveTo(player.x, player.y);
    ctx.lineTo(limbX, limbY);
    ctx.stroke();
    ctx.closePath();

    // Кончики
