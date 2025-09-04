# -storage-emulated-0-Download-pen-export-JoYwxer-1-1-pen-export-JoYwxer-dist-script.js
لعبه الحيه ممتعه للاطفال ولا تحتاج الا الانترنت 
<!doctype html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>majed alsaheb — لعبة الحية</title>
  <link rel="stylesheet" href="style.css"> 
</head>
<body>
  <div class="wrap">
    <header>
      <h1>majed alsaheb</h1>
      <div class="muted">النقاط: <span class="score" id="score">0</span></div>
    </header>

    <div class="board-wrap" style="position:relative">
      <canvas id="game" width="600" height="600" aria-label="لعبة الحية"></canvas>
      <div id="overlay" class="overlay" style="display:none">
        <div class="panel">
          <h2 id="overlayTitle">تمت الهزيمة</h2>
          <p id="overlayText">نقاطك: <strong id="finalScore">0</strong></p>
          <div style="display:flex;gap:8px;justify-content:center;margin-top:8px">
            <button class="btn" id="restart">إعادة اللعب</button>
            <button class="btn" id="newGame">لعبة جديدة</button>
          </div>
        </div>
      </div>
    </div>

    <div class="controls">
      <button class="btn" id="btnUp">↑</button>
      <button class="btn" id="btnLeft">←</button>
      <button class="btn" id="btnDown">↓</button>
      <button class="btn" id="btnRight">→</button>
      <div class="muted">استخدم الأسهم أو WASD للتحكم. أو اللمس على الموبايل.</div>
    </div>

    <!-- أزرار التحكم بالموسيقى -->
    <div style="margin-top:10px;display:flex;gap:8px;justify-content:center;">
      <button class="btn" id="playMusic">تشغيل الموسيقى</button>
      <button class="btn" id="pauseMusic">إيقاف الموسيقى</button>
    </div>

    <!-- موسيقى خلفية جاهزة -->
    <audio id="bg-music" src="https://assets.mixkit.co/music/preview/mixkit-retro-arcade-game-239.mp3" loop autoplay></audio>
  </div>

  <script src="app.js"></script>
  <script>
    const audio = document.getElementById('bg-music');

    document.getElementById('playMusic').addEventListener('click', () => {
      audio.play();
    });

    document.getElementById('pauseMusic').addEventListener('click', () => {
      audio.pause();
    });
  </script>
</body>
</html>
:root{--bg:#000;--panel:#0b0b0b;--text:#e6e6e6;--accent1:#ff2d2d;--accent2:#2d8bff}
*{box-sizing:border-box}
html,body{height:100%;margin:0;font-family:system-ui,-apple-system,Segoe UI,Roboto,'Noto Sans',Arial}
body{display:flex;align-items:center;justify-content:center;background:var(--bg);color:var(--text)}
.wrap{width:100%;max-width:820px;padding:20px}
header{display:flex;align-items:center;justify-content:space-between;gap:12px;margin-bottom:14px}
h1{margin:0;font-size:20px;color:var(--text);letter-spacing:1px}
.score{font-weight:700;background:linear-gradient(90deg,var(--accent1),var(--accent2));-webkit-background-clip:text;background-clip:text;color:transparent}
.board-wrap{background:linear-gradient(180deg,rgba(255,255,255,0.02),rgba(255,255,255,0.01));border-radius:12px;padding:12px;border:1px solid rgba(255,255,255,0.06)}
canvas{display:block;background:#000;border-radius:8px;max-width:100%;height:auto}
.controls{display:flex;gap:8px;flex-wrap:wrap;margin-top:10px}
.btn{background:#111;padding:8px 12px;border-radius:10px;color:var(--text);border:1px solid rgba(255,255,255,0.04);cursor:pointer}
.muted{color:#9aa0a6;font-size:13px}
.overlay{position:absolute;inset:0;display:flex;align-items:center;justify-content:center}
.panel{background:rgba(0,0,0,0.75);backdrop-filter:blur(4px);padding:18px;border-radius:12px;border:1px solid rgba(255,255,255,0.05);text-align:center}
.panel h2{color:var(--text);margin:0 0 8px}
.panel p{color:#bfc7cc;margin:0 0 10px}
.big{font-size:18px;font-weight:800}
@media (max-width:520px){h1{font-size:18px}}
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const scoreEl = document.getElementById('score');
const overlay = document.getElementById('overlay');
const finalScore = document.getElementById('finalScore');

const CELL = 20;
const COLS = Math.floor(canvas.width / CELL);
const ROWS = Math.floor(canvas.height / CELL);

const COLORS = { bg:'#000000', snakeHead:'#ff2d2d', snakeBody1:'#2d8bff', apple:'#ff2d2d' };

let snake = [];
let dir = {x:1,y:0};
let nextDir = {x:1,y:0};
let apple = {x:0,y:0};
let score = 0;
let speed = 8;
let lastTime = 0;
let gameOver = false;

function reset(){
  const startX = Math.floor(COLS/2);
  const startY = Math.floor(ROWS/2);
  snake = [ {x:startX, y:startY}, {x:startX-1, y:startY}, {x:startX-2, y:startY} ];
  dir = {x:1,y:0};
  nextDir = {x:1,y:0};
  score = 0;
  speed = 8;
  placeApple();
  gameOver = false;
  overlay.style.display = 'none';
  updateScore();
}

function placeApple(){
  while(true){
    const ax = Math.floor(Math.random()*COLS);
    const ay = Math.floor(Math.random()*ROWS);
    if(!snake.some(s=>s.x===ax && s.y===ay)){
      apple.x = ax; apple.y = ay; break;
    }
  }
}

function updateScore(){ scoreEl.textContent = score; }

function draw(){
  ctx.fillStyle = COLORS.bg;
  ctx.fillRect(0,0,canvas.width,canvas.height);

  drawAppleShape(apple.x, apple.y);

  for(let i=0;i<snake.length;i++){
    const s = snake[i];
    ctx.fillStyle = (i===0)? COLORS.snakeHead : (i%2===0? COLORS.snakeBody1 : '#ff2d2d');
    drawCell(s.x,s.y, ctx.fillStyle);
  }
}

function drawCell(cx,cy,color){
  const px = cx*CELL;
  const py = cy*CELL;
  ctx.fillStyle = color;
  ctx.fillRect(px+1,py+1,CELL-2,CELL-2);
}

function drawAppleShape(cx,cy){
  const px = cx*CELL + CELL/2;
  const py = cy*CELL + CELL/2;
  ctx.fillStyle = '#ff4646';
  ctx.beginPath();
  ctx.arc(px,py, CELL*0.38, 0, Math.PI*2);
  ctx.fill();
}

function step(timestamp){
  if((nextDir.x !== -dir.x || nextDir.y !== -dir.y)){
    dir = {...nextDir};
  }

  if(timestamp - lastTime > 1000/speed){
    lastTime = timestamp;
    const head = {x:snake[0].x+dir.x, y:snake[0].y+dir.y};

    if(head.x<0||head.y<0||head.x>=COLS||head.y>=ROWS||snake.some(s=>s.x===head.x&&s.y===head.y)){
      gameOver = true;
    }

    if(!gameOver){
      snake.unshift(head);
      if(head.x===apple.x&&head.y===apple.y){
        score++;
        updateScore();
        placeApple();
        speed+=0.3;
      } else {
        snake.pop();
      }
    } else {
      finalScore.textContent = score;
      overlay.style.display = 'flex';
    }
  }

  draw();
  requestAnimationFrame(step);
}

function handleKey(e){
  if(e.key==='ArrowUp'||e.key==='w') nextDir={x:0,y:-1};
  if(e.key==='ArrowDown'||e.key==='s') nextDir={x:0,y:1};
  if(e.key==='ArrowLeft'||e.key==='a') nextDir={x:-1,y:0};
  if(e.key==='ArrowRight'||e.key==='d') nextDir={x:1,y:0};
}
![1000003907](https://github.com/user-attachments/assets/0a66d890-0793-4010-89d5-ead07379b5b1)

document.addEventListener('keydown',handleKey);

document.getElementById('btnUp').onclick=()=>nextDir={x:0,y:-1};
document.getElementById('btnDown').onclick=()=>nextDir={x:0,y:1};
document.getElementById('btnLeft').onclick=()=>nextDir={x:-1,y:0};
document.getElementById('btnRight').onclick=()=>nextDir={x:1,y:0};

document.getElementById('restart').onclick=()=>reset();
document.getElementById('newGame').onclick=()=>reset();

reset();
requestAnimationFrame(step);
