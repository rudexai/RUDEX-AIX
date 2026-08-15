# RUDEX-AIX
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flappy Bird HTML5</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #70c5ce;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            font-family: Arial, sans-serif;
            overflow: hidden;
        }
        canvas {
            border: 2px solid #000;
            background-color: #70c5ce;
            box-shadow: 0 10px 20px rgba(0,0,0,0.2);
        }
    </style>
</head>
<body>

<canvas id="gameCanvas" width="320" height="480"></canvas>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

// Estado do Jogo
let frames = 0;
let score = 0;
let highScore = 0;
let gameState = "START"; // START, PLAY, GAMEOVER

// Passarinho
const bird = {
    x: 50,
    y: 150,
    radius: 12,
    gravity: 0.25,
    jump: 4.6,
    velocity: 0,
    
    draw() {
        ctx.fillStyle = "#FFD700";
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
        ctx.fill();
        ctx.strokeStyle = "#000";
        ctx.stroke();

        // Olho
        ctx.fillStyle = "#FFF";
        ctx.beginPath();
        ctx.arc(this.x + 4, this.y - 4, 4, 0, Math.PI * 2);
        ctx.fill();
        ctx.fillStyle = "#000";
        ctx.beginPath();
        ctx.arc(this.x + 5, this.y - 4, 1.5, 0, Math.PI * 2);
        ctx.fill();

        // Bico
        ctx.fillStyle = "#FF4500";
        ctx.beginPath();
        ctx.arc(this.x + 8, this.y + 2, 4, 0, Math.PI * 2);
        ctx.fill();
    },

    update() {
        if (gameState === "PLAY") {
            this.velocity += this.gravity;
            this.y += this.velocity;

            // Colisão com o chão
            if (this.y + this.radius >= canvas.height - 50) {
                this.y = canvas.height - 50 - this.radius;
                gameOver();
            }

            // Colisão com o teto
            if (this.y - this.radius <= 0) {
                this.y = this.radius;
                this.velocity = 0;
            }
        } else if (gameState === "START") {
            // Efeito flutuante na tela inicial
            this.y = 150 + Math.sin(frames / 10) * 5;
        }
    },

    flap() {
        this.velocity = -this.jump;
    },

    reset() {
        this.y = 150;
        this.velocity = 0;
    }
};

// Canos
const pipes = {
    position: [],
    width: 50,
    gap: 120,
    dx: 2,

    draw() {
        ctx.fillStyle = "#2ecc71";
        ctx.strokeStyle = "#27ae60";
        ctx.lineWidth = 2;

        for (let i = 0; i < this.position.length; i++) {
            let p = this.position[i];
            let topPipeY = p.y;
            let bottomPipeY = p.y + this.gap;

            // Cano Superior
            ctx.fillRect(p.x, 0, this.width, topPipeY);
            ctx.strokeRect(p.x, 0, this.width, topPipeY);

            // Cano Inferior
            ctx.fillRect(p.x, bottomPipeY, this.width, canvas.height - bottomPipeY - 50);
            ctx.strokeRect(p.x, bottomPipeY, this.width, canvas.height - bottomPipeY - 50);
        }
    },

    update() {
        if (gameState !== "PLAY") return;

        // Adiciona novos canos a cada 100 frames
        if (frames % 100 === 0) {
            this.position.push({
                x: canvas.width,
                y: Math.floor(Math.random() * (canvas.height - 200 - this.gap)) + 30
            });
        }

        for (let i = 0; i < this.position.length; i++) {
            let p = this.position[i];
            p.x -= this.dx;

            // Checa colisão com o passarinho
            let bottomPipeY = p.y + this.gap;
            
            // Colisão no eixo X
            if (bird.x + bird.radius > p.x && bird.x - bird.radius < p.x + this.width) {
                // Colisão no eixo Y (superior ou inferior)
                if (bird.y - bird.radius < p.y || bird.y + bird.radius > bottomPipeY) {
                    gameOver();
                }
            }

            // Aumenta pontuação
            if (p.x + this.width < bird.x && !p.passed) {
                score++;
                p.passed = true;
            }

            // Remove canos que saíram da tela
            if (p.x + this.width <= 0) {
                this.position.shift();
                i--;
            }
        }
    },

    reset() {
        this.position = [];
    }
};

// Chão
const ground = {
    draw() {
        ctx.fillStyle = "#e0c97b";
        ctx.fillRect(0, canvas.height - 50, canvas.width, 50);
        ctx.fillStyle = "#00b140";
        ctx.fillRect(0, canvas.height - 50, canvas.width, 10);
    }
};

// Controles
function action() {
    switch (gameState) {
        case "START":
            gameState = "PLAY";
            break;
        case "PLAY":
            bird.flap();
            break;
        case "GAMEOVER":
            pipes.reset();
            bird.reset();
            score = 0;
            gameState = "PLAY";
            break;
    }
}

document.addEventListener("keydown", (e) => {
    if (e.code === "Space") action();
});
canvas.addEventListener("click", action);

function gameOver() {
    if (score > highScore) highScore = score;
    gameState = "GAMEOVER";
}

// Interface (Textos)
function drawUI() {
    ctx.fillStyle = "#FFF";
    ctx.strokeStyle = "#000";
    ctx.lineWidth = 2;

    if (gameState === "START") {
        ctx.font = "24px Arial";
        ctx.fillText("FLAPPY BIRD", 85, 230);
        ctx.font = "16px Arial";
        ctx.fillText("Clique/Espaço para jogar", 70, 260);
    } else if (gameState === "PLAY") {
        ctx.font = "35px Arial";
        ctx.fillText(score, canvas.width / 2 - 10, 50);
        ctx.strokeText(score, canvas.width / 2 - 10, 50);
    } else if (gameState === "GAMEOVER") {
        ctx.font = "30px Arial";
        ctx.fillText("GAME OVER", 70, 200);
        
        ctx.font = "18px Arial";
        ctx.fillText(`Pontos: ${score}`, 115, 240);
        ctx.fillText(`Recorde: ${highScore}`, 110, 270);
        
        ctx.font = "14px Arial";
        ctx.fillText("Clique para reiniciar", 95, 310);
    }
}

// Loop Principal
function loop() {
    // Limpa a tela
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Atualiza objetos
    bird.update();
    pipes.update();

    // Desenha objetos
    pipes.draw();
    ground.draw();
    bird.draw();
    drawUI();

    frames++;
    requestAnimationFrame(loop);
}

loop();
</script>

</body>
</html>

