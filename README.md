<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no" />
<title>Rana Runner Móvil 🐸</title>
<style>
  body {
    margin: 0; background: #a0d8f7; overflow: hidden; font-family: Arial, sans-serif;
    touch-action: manipulation;
  }
  canvas {
    background: #70c5ce;
    display: block;
    margin: 0 auto;
    border: 2px solid #333;
    max-width: 100vw;
    height: auto;
  }
  #info {
    text-align: center;
    color: #333;
    margin-top: 5px;
  }
  .btn {
    position: fixed;
    background: rgba(50, 150, 50, 0.7);
    border-radius: 10px;
    color: white;
    font-weight: bold;
    font-size: 18px;
    user-select: none;
    z-index: 10;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  #btnJump {
    left: 10px; top: 10px;
    width: 80px; height: 80px;
  }
  #btnCrouch {
    left: 10px; bottom: 10px;
    width: 80px; height: 80px;
  }
  #btnShoot {
    right: 10px; bottom: 10px;
    width: 80px; height: 80px;
    background: rgba(200, 50, 50, 0.7);
  }
  #gameOver {
    color: red;
    font-size: 24px;
    display: none;
    margin-top: 10px;
  }
</style>
</head>
<body>

<canvas id="gameCanvas" width="800" height="300"></canvas>
<div id="info">
  <p><b>Controles táctiles:</b></p>
  <p>Saltar (arriba izquierda) | Agacharse (abajo izquierda) | Disparar (abajo derecha)</p>
  <p id="score">Puntuación: 0</p>
  <p id="gameOver">¡Juego terminado! Recarga para jugar de nuevo.</p>
</div>

<button id="btnJump" class="btn">Saltar</button>
<button id="btnCrouch" class="btn">Agacharse</button>
<button id="btnShoot" class="btn">Disparar</button>

<script>
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');

  const GRAVEDAD = 0.6;
  const SUELO_Y = canvas.height - 50;
  let gameOver = false;
  let score = 0;

  const rana = {
    x: 50,
    y: SUELO_Y,
    width: 50,
    height: 50,
    vy: 0,
    jumping: false,
    crouching: false,
    canShoot: true,
    shootCooldown: 0,
    draw() {
      ctx.fillStyle = '#2e8b57';
      if (this.crouching) {
        ctx.fillRect(this.x, this.y + 25, this.width, this.height - 25);
        ctx.beginPath();
        ctx.arc(this.x + 25, this.y + 25, 15, 0, Math.PI * 2);
        ctx.fill();
      } else {
        ctx.fillRect(this.x, this.y, this.width, this.height);
        ctx.fillStyle = 'white';
        ctx.beginPath();
        ctx.arc(this.x + 15, this.y + 15, 8, 0, Math.PI * 2);
        ctx.arc(this.x + 35, this.y + 15, 8, 0, Math.PI * 2);
        ctx.fill();
        ctx.fillStyle = 'black';
        ctx.beginPath();
        ctx.arc(this.x + 15, this.y + 15, 4, 0, Math.PI * 2);
        ctx.arc(this.x + 35, this.y + 15, 4, 0, Math.PI * 2);
        ctx.fill();
      }
    },
    update() {
      this.vy += GRAVEDAD;
      this.y += this.vy;
      if (this.y > SUELO_Y) {
        this.y = SUELO_Y;
        this.vy = 0;
        this.jumping = false;
      }
      if (this.shootCooldown > 0) this.shootCooldown--;
      else this.canShoot = true;
    },
    jump() {
      if (!this.jumping && !this.crouching) {
        this.vy = -12;
        this.jumping = true;
      }
    },
    crouch(state) {
      if (!this.jumping) this.crouching = state;
    },
    shoot() {
      if (this.canShoot) {
        this.canShoot = false;
        this.shootCooldown = 30;
        return true;
      }
      return false;
    }
  };

  class Proyectil {
    constructor(x, y) {
      this.x = x;
      this.y = y;
      this.radius = 5;
      this.speed = 10;
      this.active = true;
    }
    update() {
      this.x += this.speed;
      if (this.x > canvas.width) this.active = false;
    }
    draw() {
      ctx.fillStyle = '#ff4500';
      ctx.beginPath();
      ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
      ctx.fill();
    }
  }

  class Enemigo {
    constructor(x, y) {
      this.x = x;
      this.y = y;
      this.width = 40;
      this.height = 40;
      this.speed = 6;
      this.active = true;
      this.health = 1;
    }
    update() {
      this.x -= this.speed;
      if (this.x + this.width < 0) this.active = false;
    }
    draw() {
      ctx.fillStyle = '#8b0000';
      ctx.fillRect(this.x, this.y, this.width, this.height);
      ctx.fillStyle = 'yellow';
      ctx.beginPath();
      ctx.arc(this.x + 10, this.y + 15, 7, 0, Math.PI * 2);
      ctx.arc(this.x + 30, this.y + 15, 7, 0, Math.PI * 2);
      ctx.fill();
      ctx.fillStyle = 'black';
      ctx.beginPath();
      ctx.arc(this.x + 10, this.y + 15, 3, 0, Math.PI * 2);
      ctx.arc(this.x + 30, this.y + 15, 3, 0, Math.PI * 2);
      ctx.fill();
    }
  }

  class Obstaculo {
    constructor(x, y, width, height) {
      this.x = x;
      this.y = y;
      this.width = width;
      this.height = height;
      this.speed = 6;
      this.active = true;
    }
    update() {
      this.x -= this.speed;
      if (this.x + this.width < 0) this.active = false;
    }
    draw() {
      ctx.fillStyle = '#654321';
      ctx.fillRect(this.x, this.y, this.width, this.height);
    }
  }

  let proyectiles = [];
  let enemigos = [];
  let obstaculos = [];
  let frameCount = 0;

  function generarElementos() {
    if (frameCount % 150 === 0) {
      if (Math.random() < 0.6) {
        enemigos.push(new Enemigo(canvas.width, SUELO_Y + 10));
      } else {
        let altura = 30 + Math.random() * 20;
        obstaculos.push(new Obstaculo(canvas.width, SUELO_Y + 50 - altura, 30, altura));
      }
    }
  }

  function colisionRect(a, b) {
    return (
      a.x < b.x + b.width &&
      a.x + a.width > b.x &&
      a.y < b.y + b.height &&
      a.y + a.height > b.y
    );
  }

  function colisionProyectilEnemigo(proy, enem) {
    let distX = Math.abs(proy.x - (enem.x + enem.width / 2));
    let distY = Math.abs(proy.y - (enem.y + enem.height / 2));
    if (distX > (enem.width / 2 + proy.radius)) return false;
    if (distY > (enem.height / 2 + proy.radius)) return false;
    if (distX <= (enem.width / 2)) return true;
    if (distY <= (enem.height / 2)) return true;
    let dx = distX - enem.width / 2;
    let dy = distY - enem.height / 2;
    return (dx * dx + dy * dy <= (proy.radius * proy.radius));
  }

  function dibujarSuelo() {
    ctx.fillStyle = '#654321';
    ctx.fillRect(0, SUELO_Y + 50, canvas.width, 10);
  }

  function actualizarPuntuacion() {
    document.getElementById('score').textContent = 'Puntuación: ' + score;
  }

  function finJuego() {
    gameOver = true;
    document.getElementById('gameOver').style.display = 'block';
  }

  function gameLoop() {
    if (gameOver) return;

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    ctx.fillStyle = '#87ceeb';
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.fillStyle = 'white';
    ctx.beginPath();
    ctx.ellipse(100 + (frameCount % 800), 50, 40, 20, 0, 0, Math.PI * 2);
    ctx.ellipse(140 + (frameCount % 800), 50, 40, 20, 0, 0, Math.PI * 2);
    ctx.fill();

    rana.update();
    rana.draw();

    proyectiles.forEach(p => p.update());
    proyectiles = proyectiles.filter(p => p.active);

    generarElementos();

    enemigos.forEach(e => e.update());
    enemigos = enemigos.filter(e => e.active);

    obstaculos.forEach(o => o.update());
    obstaculos = obstaculos.filter(o => o.active);

    proyectiles.forEach(p => p.draw());

    enemigos.forEach(enem => {
      enem.draw();
      proyectiles.forEach(proy => {
        if (colisionProyectilEnemigo(proy, enem)) {
          enem.health--;
          proy.active = false;
          if (enem.health <= 0) {
            enem.active = false;
            score += 10;
            actualizarPuntuacion();
          }
        }
      });
      if (colisionRect(rana, enem)) finJuego();
    });

    obstaculos.forEach(obs => {
      obs.draw();
      if (colisionRect(rana, obs)) finJuego();
    });

    dibujarSuelo();

    frameCount++;
    score++;
    if (frameCount % 60 === 0) actualizarPuntuacion();

    requestAnimationFrame(gameLoop);
  }

  // Botones táctiles
  document.getElementById('btnJump').addEventListener('touchstart', e => {
    e.preventDefault();
    if (!gameOver) rana.jump();
  });
  document.getElementById('btnCrouch').addEventListener('touchstart', e => {
    e.preventDefault();
    if (!gameOver) rana.crouch(true);
  });
  document.getElementById('btnCrouch').addEventListener('touchend', e => {
    e.preventDefault();
    rana.crouch(false);
  });
  document.getElementById('btnShoot').addEventListener('touchstart', e => {
    e.preventDefault();
    if (!gameOver && rana.shoot()) {
      proyectiles.push(new Proyectil(rana.x + rana.width, rana.y + (rana.crouching ? 50 : 25)));
    }
  });

  actualizarPuntuacion();
  gameLoop();
</script>

</body>
</html>
