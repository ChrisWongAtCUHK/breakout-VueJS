<template>
  <div>
    <div class="game-notifications clearfix">
        <div class="instr">
        Score : {{ setting.currentScore }}
        </div>
        <ul class="life">
        <li class="heart" :class="{ 'heart-o': setting.deathNumber > 0 }"></li>
        <li class="heart" :class="{ 'heart-o': setting.deathNumber > 1 }"></li>
        <li class="heart"></li>
        </ul>
    </div>

    <!-- Game Container -->
    <div class="container">
      <canvas ref="breakout-game-canvas" height="430" width="460"></canvas>
    </div>
    <div class="container">
      <div class="instr-footer">
        <small v-html="footer">
        </small>
      </div>
    </div>

    <!-- Modal for game over -->
    <div class="modal-overlay" v-if="showOverlay">
    </div>
    <div class="modal-box">
      <h3>{{ modalHeader }}</h3>
      <p class="modal-score" v-if="won">Score : {{ this.setting.currentScore }}</p>
      <p class="modal-message">{{ modalMessage }}</p>
      <a href="#" class="btn-play-again">Play breakout again!</a>
    </div>
  </div>
</template>

<script>
export default {
  name: 'BreakoutGame',
  data () {
    return {
      canvas: undefined,
      context: undefined,
      canvasWidth: undefined,
      canvasHeight: undefined,
      brick: undefined,
      bricks: {
        row: 4,
        column: 6,
        color: '#ffc500',
        height: 20,
        width: undefined,
        padding: 2
      },
      setting: {
        ballColor: '#7f8c8d',
        ballRadius: 10,
        paddleHeight: 10,
        paddleWidth: 120,
        frameSpeed: 10,
        paddleMoveLeft: false,
        paddleMoveRight: false,
        paddleMoveRate: 4,
        paddleX: 170,
        paddleY: 420,
        gameState: true,
        deathNumber: 0,
        deathTotal: 2,
        lostMessage: 'You\'ve lost the game. Would you like to play again?',
        wonMessage: 'You\'ve beaten the game. Would you like to improve on your score?',
        currentScore: 0,
        brickScore: 20,
        paddleScore: 10
      },
      x: undefined,
      y: 200,
      dx: 0.5,
      dy: 4,
      gameStart: undefined,
      totalBricks: 0,
      footer: `          
          Use left and right keys to move paddle.
          <br>
          Press spacebar to start game.`,
      modalHeader: '',
      modalMessage: '',
      showOverlay: false,
      won: false
    };
  },
  methods: {
    drawBall: function () {
      this.context.clearRect(0, 0, this.canvasWidth, this.canvasHeight);
      this.context.beginPath();
      this.context.arc(this.x, this.y, this.setting.ballRadius, 0, 2 * Math.PI, true);
      this.context.closePath();
      this.context.fillStyle = this.setting.ballColor;
      this.context.fill();

      this.x += this.dx;
      this.y += this.dy;
    },
    drawPaddle: function () {
      if (this.setting.paddleMoveLeft && this.setting.paddleX >= 0) {
        this.setting.paddleX -= this.setting.paddleMoveRate;
      }
      if (this.setting.paddleMoveRight && this.setting.paddleX <= this.canvasWidth - this.setting.paddleWidth) {
        this.setting.paddleX += this.setting.paddleMoveRate;
      }

      this.context.fillStyle = '#2c3e50';
      this.context.fillRect(this.setting.paddleX, this.setting.paddleY, this.setting.paddleWidth, this.setting.paddleHeight);
    },
    movePaddle: function () {
      let $vm = this;
      window.addEventListener('keydown', function (e) {
        if (e.keyCode === 39) {
          $vm.setting.paddleMoveRight = true;
        } else if (e.keyCode === 37) {
          $vm.setting.paddleMoveLeft = true;
        }
      });
      window.addEventListener('keyup', function (e) {
        if (e.keyCode === 39) {
          $vm.setting.paddleMoveRight = false;
        } else if (e.keyCode === 37) {
          $vm.setting.paddleMoveLeft = false;
        }
      });
    },
    initBricks: function () {
      this.totalBricks = this.bricks.row * this.bricks.column
      this.brick = new Array(this.bricks.column);
      for (let i = 0; i < this.bricks.column; i++) {
        this.brick[i] = new Array(this.bricks.row);
        for (let j = 0; j < this.bricks.row; j++) {
          this.brick[i][j] = { x: 0, y: 0, state: true };
        }
      }
    },
    drawBricks: function () {
      this.bricks.width = this.canvasWidth / this.bricks.column - 2;
      for (let i = 0; i < this.bricks.column; i++) {
        for (let j = 0; j < this.bricks.row; j++) {
          if (this.brick[i][j].state) {
            let brickX = i * (this.bricks.width + this.bricks.padding);
            let brickY = j * (this.bricks.height + this.bricks.padding);

            this.brick[i][j].x = brickX;
            this.brick[i][j].y = brickY;

            this.context.beginPath();
            this.context.rect(
              brickX,
              brickY,
              this.bricks.width,
              this.bricks.height
            );
            if (j === 0) {
              this.context.fillStyle = '#2980b9';
            } else if (j === 1) {
              this.context.fillStyle = '#27ae60';
            } else if (j === 2) {
              this.context.fillStyle = '#ED8F03';
            } else {
              this.context.fillStyle = '#e74c3c';
            }
            this.context.fill();
            this.context.closePath();
          }
        }
      }
    },
    brickCollision: function () {
      for (let n = 0; n < this.bricks.column; n++) {
        for (let k = 0; k < this.bricks.row; k++) {
          let currentBrick = this.brick[n][k];

          if (currentBrick.state) {
            if (this.x > currentBrick.x && this.x < currentBrick.x + this.bricks.width && this.y > currentBrick.y && this.y < currentBrick.y + this.bricks.height) {
              this.dy = -this.dy;
              currentBrick.state = false;
              this.setting.currentScore += this.setting.brickScore;
              this.totalBricks -= 1;
            }
          }
        }
      }
    },
    boundaryCollision: function () {
      if (this.y - this.setting.ballRadius <= 0) {
        this.dy = -this.dy;
      } else if (this.y + this.setting.ballRadius >= this.canvasHeight) {
        let fullBallLeft = this.x + this.setting.ballRadius;
        let fullBallRight = this.x - this.setting.ballRadius;

        if (fullBallLeft >= this.setting.paddleX && fullBallRight < this.setting.paddleX + this.setting.paddleWidth) {
          this.dx = 8 * ((this.x - (this.setting.paddleX + this.setting.paddleWidth / 2)) / this.setting.paddleWidth);
          if (this.dx === undefined) {
            console.log(this.x);
          }
          this.dy = Math.sqrt(20 - (this.dx * this.dx));
          this.dy = -this.dy;

          if (this.setting.currentScore > 0) {
            this.setting.currentScore -= this.setting.paddleScore;
          }
        } else {
          clearInterval(this.gameStart);
          this.setting.gameState = false;
        }
      }

      if (this.y + this.setting.ballRadius > this.canvasHeight + 15) {
        clearInterval(this.gameStart);
        this.setting.gameState = false;
      }

      if (this.x + this.setting.ballRadius >= this.canvasWidth || this.x - this.setting.ballRadius <= 0) {
        this.dx = -this.dx;
      }
    },
    loseTurn: function () {
      let $vm = this;
      setTimeout(function (e) {
        $vm.footer = 'Press any keys to restart the game.';

        $vm.setting.deathNumber += 1;

        $vm.x = $vm.canvasWidth / 2;
        $vm.y = 200;
        $vm.setting.paddleX = 170;

        $vm.drawBall();
        $vm.drawBricks();
        $vm.drawPaddle();
        $vm.setting.gameState = true;

        window.addEventListener('keyup', function (e) {
          $vm.gameStart = setInterval($vm.init, $vm.setting.frameSpeed);
        }, { once: true });
      }, 400);
    },
    loseGame: function () {
      this.modalPopup(this.setting.lostMessage, 'Game Over !');
    },
    initSpacebar: function () {
      let $vm = this;
      window.addEventListener('keyup', function (e) {
        if (e.keyCode === 32 && $vm.setting.deathNumber < $vm.setting.deathTotal) {
          $vm.gameStart = setInterval($vm.init, $vm.setting.frameSpeed);
          $vm.footer = '';
        }
      }, { once: true });
    },
    gameWon: function () {
      clearInterval(this.gameStart);
      this.won = true;
      this.modalPopup(this.setting.wonMessage, 'Well done !');
      this.drawBall();
      this.drawPaddle();
    },
    modalPopup: function (message, header) {
      this.modalHeader = header;
      this.modalHeader = message;

      let $vm = this;

      setTimeout(function () {
        $vm.showOverlay = true;
        document.getElementsByTagName('BODY')[0].classList.add('game-over');
      }, 100);

      document.getElementsByClassName('btn-play-again')[0].addEventListener('click', function (e) {
        e.preventDefault();
        location.reload();
      });
    },
    init: function () {
      if (this.setting.gameState === true) {
        this.drawBall();
      }
      this.boundaryCollision();
      this.drawPaddle();
      this.drawBricks();
      this.movePaddle();
      this.brickCollision();

      if (this.setting.gameState === false && this.setting.deathNumber < this.setting.deathTotal) {
        this.loseTurn();
      } else if (this.setting.gameState === false && this.setting.deathNumber === this.setting.deathTotal) {
        this.loseGame();
      }

      if (this.totalBricks === 0) {
        this.gameWon();
      }
    }
  },
  mounted () {
    this.canvas = this.$refs['breakout-game-canvas'];
    this.context = this.canvas.getContext('2d');
    this.canvasWidth = this.canvas.width;
    this.canvasHeight = this.canvas.height;
    this.x = this.canvasWidth / 2;
    this.initBricks();
    this.init();
    this.initSpacebar();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
canvas {
  background: #eee;
}
@media screen and (max-width: 460px) {
  canvas {
    max-width: 100% !important;
    width: 100% !important;
  }
}
* {
  margin: 0; padding: 0;
  box-sizing: border-box;
}
a, a:visited, a:hover, a:active, a:focus {
  text-decoration:  none; outline: none;
}
body {
  font: 16px/1.4 "Roboto", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  color: #888;
}

.container {
  width: 460px;
  margin: 10px auto;
}

.modal-overlay {
  position: fixed;
  top: 0; left: 0; bottom: 0; right: 0;
  background: rgba( 17, 67, 87, 0.95 );
}
.modal-box {
  position: fixed;
  top: 50%; left: 50%;
  -webkit-transform: translateX(-50%) translateY(-50%) scale(0);
          transform: translateX(-50%) translateY(-50%) scale(0);
  width: 560px;
  background: rgba(255,255,255,1);
  padding: 40px 20px;
  border-radius: 10px;
  opacity: 0;
  -webkit-transition: opacity 0.2s ease-out 0.2s, -webkit-transform 0.4s ease-out;
  transition: opacity 0.2s ease-out 0.2s, transform 0.4s ease-out;
}
.game-over .modal-box {
  opacity: 1;
  -webkit-transform: translateX(-50%) translateY(-50%) scale(1);
          transform: translateX(-50%) translateY(-50%) scale(1);
}
.modal-box {
  text-align: center;
}
.modal-box h3 {
  font-size: 44px;
  font-weight: normal;
  color: #111;
  margin-bottom: 0.15em;
  letter-spacing: -2px;
}
.modal-score {
  margin-bottom: 15px;
  font-size: 24px;
  color: #444;
}
.btn-play-again {
  background: #114357;
  display: inline-block;
  color: #fff;
  padding: 0 1.4em;
  height: 42px;
  line-height: 42px;
  border-radius: 4px;
}
.btn-play-again:hover {
  background: #19556D;
}
.modal-message {
  margin-bottom: 20px;
}

.game-notifications {
  margin: 20px auto 0;
  width: 460px;
}
.life {
  list-style-type: none;
  float: right;
}
.instr {
  float: left;
}
.heart {
  display: inline-block;
  width: 24px;
  height: 19px;
  background: #fff url('img/heart.png') no-repeat top left;
  background-size: 24px auto;
  transition: all 0.4s ease;
  -webkit-transition: all 0.4s ease;
}
.heart-o {
  background: #fff url('img/heart.png') no-repeat 0 -23px;
  background-size: 24px auto;
  position: relative; top: 1px; /* Cheating haha */
}

.clearfix:after {
  visibility: hidden;
  display: block;
  font-size: 0;
  content: " ";
  clear: both;
  height: 0;
}
.instr-footer {
  text-align:  center;
  text-transform: uppercase;
  font-size: 13px;
  letter-spacing: 1px;
  line-height: 1.8;
  margin-top: 30px;
}
</style>
