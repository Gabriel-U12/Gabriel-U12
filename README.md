<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <title>L'Évasion Scolaire</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    body {
      background-color: #e0f7fa;
      font-family: sans-serif;
      overflow: hidden;
    }
    canvas {
      display: block;
      margin: auto;
      background: #fff;
      border: 4px solid #000;
    }
  </style>
</head>
<body>
  <canvas id="game" width="800" height="400"></canvas>
  <script>
    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");

    // Joueur
    const player = {
      x: 50,
      y: 300,
      width: 40,
      height: 40,
      vy: 0,
      jumpForce: 15,
      grounded: true,
      color: "#4caf50"
    };

    // Variables globales
    let gravity = 0.8;
    let obstacles = [];
    let enemies = [];
    let projectiles = [];
    let score = 0;
    let gameOver = false;

    // Génération des obstacles
    function spawnObstacle() {
      const height = 40;
      obstacles.push({
        x: canvas.width,
        y: canvas.height - height,
        width: 40,
        height: height,
        color: "#795548"
      });
    }

    // Génération des profs
    function spawnEnemy() {
      enemies.push({
        x: canvas.width,
        y: Math.random() * 200 + 50,
        width: 40,
        height: 40,
        color: "#f44336"
      });
    }

    // Contrôles
    document.addEventListener("keydown", (e) => {
      if (e.code === "Space" && player.grounded) {
        player.vy = -player.jumpForce;
        player.grounded = false;
      }
      if (e.code === "KeyF") {
        // Lancer un fardeau
        projectiles.push({
          x: player.x + player.width,
          y: player.y + player.height / 2 - 5,
          width: 10,
          height: 10,
          color: "#000",
          speed: 6
        });
      }
    });

    // Boucle de jeu
    function update() {
      if (gameOver) return;

      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Gravité
      player.vy += gravity;
      player.y += player.vy;
      if (player.y + player.height >= canvas.height) {
        player.y = canvas.height - player.height;
        player.vy = 0;
        player.grounded = true;
      }

      // Affichage joueur
      ctx.fillStyle = player.color;
      ctx.fillRect(player.x, player.y, player.width, player.height);

      // Obstacles
      for (let i = 0; i < obstacles.length; i++) {
        const obs = obstacles[i];
        obs.x -= 5;
        ctx.fillStyle = obs.color;
        ctx.fillRect(obs.x, obs.y, obs.width, obs.height);

        // Collision joueur
        if (
          player.x < obs.x + obs.width &&
          player.x + player.width > obs.x &&
          player.y < obs.y + obs.height &&
          player.y + player.height > obs.y
        ) {
          gameOver = true;
        }
      }

      // Profs (ennemis)
      for (let i = 0; i < enemies.length; i++) {
        const enemy = enemies[i];
        enemy.x -= 3;
        ctx.fillStyle = enemy.color;
        ctx.fillRect(enemy.x, enemy.y, enemy.width, enemy.height);

        // Collision joueur
        if (
          player.x < enemy.x + enemy.width &&
          player.x + player.width > enemy.x &&
          player.y < enemy.y + enemy.height &&
          player.y + player.height > enemy.y
        ) {
          gameOver = true;
        }

        // Collision projectile
        for (let j = 0; j < projectiles.length; j++) {
          const p = projectiles[j];
          if (
            p.x < enemy.x + enemy.width &&
            p.x + p.width > enemy.x &&
            p.y < enemy.y + enemy.height &&
            p.y + p.height > enemy.y
          ) {
            enemies.splice(i, 1);
            projectiles.splice(j, 1);
            score += 10;
            break;
          }
        }
      }

      // Projeter les fardeaux
      for (let i = 0; i < projectiles.length; i++) {
        const p = projectiles[i];
        p.x += p.speed;
        ctx.fillStyle = p.color;
        ctx.fillRect(p.x, p.y, p.width, p.height);

        // Supprimer si hors écran
        if (p.x > canvas.width) {
          projectiles.splice(i, 1);
          i--;
        }
      }

      // Score
      score += 0.1;
      ctx.fillStyle = "#000";
      ctx.font = "20px Arial";
      ctx.fillText("Score: " + Math.floor(score), 10, 30);

      // Game Over
      if (gameOver) {
        ctx.fillStyle = "red";
        ctx.font = "40px Arial";
        ctx.fillText("GAME OVER", canvas.width / 2 - 120, canvas.height / 2);
      }

      requestAnimationFrame(update);
    }

    // Génération aléatoire
    setInterval(spawnObstacle, 2000);
    setInterval(spawnEnemy, 4000);

    update();
  </script>
</body>
</html>
