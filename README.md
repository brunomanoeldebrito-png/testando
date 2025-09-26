<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Soccer</title>
<style>
  body { margin:0; font-family: Arial; background:#222; color:#fff; text-align:center; }
  #menu, #game, #shop { display:none; padding-top:30px; }
  .active { display:block; }
  button { padding:10px 20px; margin:5px; cursor:pointer; }
  canvas { background:green; display:block; margin:20px auto; border:2px solid #fff; }
  input { padding:5px; margin:5px; }
</style>
</head>
<body>

<div id="menu" class="active">
  <h1>Soccer</h1>
  <input type="text" id="teamName" placeholder="Nome do seu time" />
  <button onclick="startGame()">Criar Time e Jogar</button>
  <button onclick="openShop()">Loja de Jogadores</button>
</div>

<div id="game">
  <h2 id="teamDisplay"></h2>
  <p>Moedas: <span id="coinsDisplay">0</span></p>
  <canvas id="field" width="600" height="400"></canvas>
  <div>
    <button onclick="goMenu()">Voltar ao Menu</button>
    <button onclick="winMatch()">Vencer Partida (+10 moedas)</button>
  </div>
</div>

<div id="shop">
  <h2>Loja de Jogadores</h2>
  <p>Moedas: <span id="coinsDisplayShop">0</span></p>
  <div id="playerList"></div>
  <button onclick="closeShop()">Voltar</button>
</div>

<script>
// --- Variáveis do Jogo ---
const menu = document.getElementById('menu');
const game = document.getElementById('game');
const shop = document.getElementById('shop');
const teamDisplay = document.getElementById('teamDisplay');
const coinsDisplay = document.getElementById('coinsDisplay');
const coinsDisplayShop = document.getElementById('coinsDisplayShop');

const canvas = document.getElementById('field');
const ctx = canvas.getContext('2d');

let coins = Number(localStorage.getItem('soccerCoins')) || 0;
let teamName = localStorage.getItem('soccerTeam') || "Meu Time";

let player = { x: 300, y: 200, size: 15, color: 'blue' };
let ball = { x: 300, y: 200, size: 8, color: 'white' };

// Jogadores disponíveis na loja
const availablePlayers = [
  { name: "Jogador A", price: 10 },
  { name: "Jogador B", price: 20 },
  { name: "Jogador C", price: 30 }
];

// --- Funções ---
function startGame(){
  teamName = document.getElementById('teamName').value || teamName;
  localStorage.setItem('soccerTeam', teamName);
  teamDisplay.textContent = "Time: " + teamName;
  coinsDisplay.textContent = coins;
  coinsDisplayShop.textContent = coins;
  menu.classList.remove('active');
  game.classList.add('active');
  drawField();
}

function goMenu(){
  menu.classList.add('active');
  game.classList.remove('active');
}

function drawField(){
  ctx.clearRect(0,0,canvas.width,canvas.height);
  ctx.fillStyle = 'green';
  ctx.fillRect(0,0,canvas.width,canvas.height);
  // jogador
  ctx.fillStyle = player.color;
  ctx.beginPath();
  ctx.arc(player.x, player.y, player.size, 0, Math.PI*2);
  ctx.fill();
  // bola
  ctx.fillStyle = ball.color;
  ctx.beginPath();
  ctx.arc(ball.x, ball.y, ball.size, 0, Math.PI*2);
  ctx.fill();
}

document.addEventListener('keydown', e => {
  const speed = 5;
  if(e.key === 'ArrowUp') player.y -= speed;
  if(e.key === 'ArrowDown') player.y += speed;
  if(e.key === 'ArrowLeft') player.x -= speed;
  if(e.key === 'ArrowRight') player.x += speed;
  drawField();
});

function winMatch(){
  coins += 10;
  localStorage.setItem('soccerCoins', coins);
  coinsDisplay.textContent = coins;
  coinsDisplayShop.textContent = coins;
  alert("Você ganhou 10 moedas!");
}

// --- Loja ---
function openShop(){
  menu.classList.remove('active');
  shop.classList.add('active');
  renderShop();
}

function closeShop(){
  shop.classList.remove('active');
  menu.classList.add('active');
}

function renderShop(){
  coinsDisplayShop.textContent = coins;
  const list = document.getElementById('playerList');
  list.innerHTML = '';
  availablePlayers.forEach((p,i)=>{
    const btn = document.createElement('button');
    btn.textContent = `${p.name} - ${p.price} moedas`;
    btn.onclick = ()=>{
      if(coins >= p.price){
        coins -= p.price;
        localStorage.setItem('soccerCoins', coins);
        coinsDisplay.textContent = coins;
        coinsDisplayShop.textContent = coins;
        alert(`${p.name} comprado!`);
      } else {
        alert("Moedas insuficientes!");
      }
    }
    list.appendChild(btn);
  });
}
</script>

</body>
</html>
