<template>
  <div>
    <canvas ref="canvas" class="game-canvas" :width="background.width" :height="background.height"></canvas>
    <div v-if="game.lost || game.win" class="game-over-message">
      {{ game.lost ? 'GAMEOVER！Score: ' + game.score : 'CLEAR!' }}
    </div>
  </div>
  <div>
    <h1>Vue Breakout</h1>
    <h2>Aで左<br>Dで右<br>Rで最初から</h2>
  </div>
</template>

<script>
export default {
  data() {
    return {
      started: false,
      game: {
        score: 0,
        win: 0,
        lost: 0
      },
      background: {
        width: 800,
        height: 600
      },
      ball: {
        radius: 5,
        coordinates: { x: 400, y: 530 },
        speed: { x: 8, y: -8 }
      },
      paddle: {
        body: { width: 160, height: 15 },
        coordinates: { x: 320, y: 570 }
      },
      bricks: []
    };
  },
  methods: {
    constrain(number, low, high) {
      return number < low ? low : number > high ? high : number;
    },
    //二点間の距離でやっていきしたらうまいこといった
    distanceBetweenPoints(p1, p2) {
      return Math.sqrt((p2.x - p1.x) ** 2 + (p2.y - p1.y) ** 2);
    },
    updatePaddle(key) {
      if (!(this.game.win || this.game.lost)) {
        if (key === 'a') {
          this.paddle.coordinates.x = this.constrain(this.paddle.coordinates.x - 15, 0, this.background.width);
        } else if (key === 'd') {
          this.paddle.coordinates.x = this.constrain(this.paddle.coordinates.x + 15, 0, this.background.width - this.paddle.body.width);
        }
      }
    },
    checkWin() {
      if (this.bricks.length === 0) {
        this.game.win = true;
      }
    },
    moveBall() {
      this.ball.coordinates.x += this.ball.speed.x;
      this.ball.coordinates.y += this.ball.speed.y;

      if (
          this.ball.coordinates.x + this.ball.radius > this.paddle.coordinates.x &&
          this.ball.coordinates.x - this.ball.radius < this.paddle.coordinates.x + this.paddle.body.width &&
          this.ball.coordinates.y + this.ball.radius > this.paddle.coordinates.y &&
          this.ball.coordinates.y - this.ball.radius < this.paddle.coordinates.y + this.paddle.body.height
      ) {
        this.ball.speed.y = -this.ball.speed.y;
      }

      for (let i = 0; i < this.bricks.length; i++) {
        const brick = this.bricks[i];
        if (
            this.ball.coordinates.x + this.ball.radius > brick.coordinates.x &&
            this.ball.coordinates.x - this.ball.radius < brick.coordinates.x + brick.body.width &&
            this.ball.coordinates.y + this.ball.radius > brick.coordinates.y &&
            this.ball.coordinates.y - this.ball.radius < brick.coordinates.y + brick.body.height
        ) {

          this.bricks.splice(i, 1);
          this.game.score += 10;
          this.ball.speed.y = -this.ball.speed.y;
          this.checkWin();
          break;
        }
      }

      //上下の跳ね返りをするやつ
      if (this.ball.coordinates.y - this.ball.radius < 0) {
        this.ball.speed.y = -this.ball.speed.y;
      }

      //左右の跳ね返りをするやつ
      if (
          this.ball.coordinates.x - this.ball.radius < 0 ||
          this.ball.coordinates.x + this.ball.radius > this.background.width
      ) {
        this.ball.speed.x = -this.ball.speed.x;
      }

      //落下
      if (this.ball.coordinates.y + this.ball.radius > this.background.height) {
        this.game.lost = 1;
      }
    },
    movePaddle() {
    },
    draw() {
      const canvas = this.$refs.canvas;
      if (!canvas) return;

      const ctx = canvas.getContext('2d');
      if (!ctx) return;

      ctx.fillStyle = '#041a1a';
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      ctx.fillStyle = '#041a1a';
      ctx.fillRect(0, 0, this.background.width, this.background.height);

      this.bricks.forEach((bri) => {
        ctx.fillStyle = bri.color;
        ctx.fillRect(bri.coordinates.x, bri.coordinates.y, bri.body.width, bri.body.height);
        ctx.lineWidth = 2;
        ctx.strokeStyle = '#00d192';
        ctx.strokeRect(bri.coordinates.x, bri.coordinates.y, bri.body.width, bri.body.height);
      });

      ctx.beginPath();
      ctx.arc(this.ball.coordinates.x, this.ball.coordinates.y, this.ball.radius, 0, 2 * Math.PI);
      ctx.fillStyle = '#F00';
      ctx.fill();
      ctx.lineWidth = 2;
      ctx.strokeStyle = '#F00';
      ctx.stroke();

      ctx.fillStyle = '#70c49e';
      ctx.fillRect(this.paddle.coordinates.x, this.paddle.coordinates.y, this.paddle.body.width, this.paddle.body.height);

      if (!(this.game.win || this.game.lost)) {
        ctx.font = '20px Arial';
        ctx.fillStyle = '#00d192';
        ctx.fillText('Score: ' + this.game.score, 10, 20);
        ctx.stroke();
        requestAnimationFrame(this.draw);
      }
    }
  },
  mounted() {
    window.addEventListener('keydown', (e) => {
      if (!(this.game.win || this.game.lost)) {
        this.updatePaddle(e.key.toLowerCase());
      } else if (e.key.toLowerCase() === 'r') {
        location.reload();
      }
    });

    for (let i = 40; i < this.background.width - 50; i += 45) {
      for (let j = 50; j <= this.background.height - 350; j += 15) {
        this.bricks.push({
          coordinates: { x: i, y: j },
          body: { width: 40, height: 10 },
          color: '#00d192'
        });
      }
    }

    let lastTime = 0;

    const gameLoop = (timestamp) => {
      const deltaTime = timestamp - lastTime;
      this.moveBall(deltaTime);
      this.movePaddle(deltaTime);
      this.draw();

      lastTime = timestamp;
      requestAnimationFrame(gameLoop);
    };

    requestAnimationFrame(gameLoop);
  }
};
</script>

<style scoped>
.game-canvas {
  display: block;
  margin: auto;
  border: 2px solid #FFF;
}
.game-over-message {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 30px;
  text-align: center;
}
</style>
