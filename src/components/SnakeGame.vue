<template>
  <div class="game-box">
    <div class="tools">
      <button @click="playGame">
        {{ isPlaying ? "停止" : isLose ? "重新开始" : "开始" }}
      </button>
      <div class="info-bar">
        <p>🐍 的长度: {{ snakeBodyList.length }}</p>
      </div>
      <p class="count">得分: {{ count }}</p>
    </div>
    <div ref="snake" class="snake">
      <div
        :style="{ top: headerPos.y + 'px', left: headerPos.x + 'px' }"
        ref="snakeHeader"
        class="snake-header"
      />
      <div
        :key="uuid"
        :uid="uuid"
        v-for="{ pos: { y, x }, uuid } in snakeBodyList"
        :style="{ top: y + 'px', left: x + 'px' }"
        class="snake-body"
      />
    </div>
    <div
      class="snake-food"
      :style="{ top: foodPos.y + 'px', left: foodPos.x + 'px' }"
    />
  </div>
</template>

<script>
const directionKeys = {
  up: 38,
  down: 40,
  left: 37,
  right: 39,
};
const MAX_X = 500;
const MAX_Y = 500;
const defaultUnit = 10;

// 检查是否在横坐标水平方向上
function checkIsLevel(direction) {
  return direction === directionKeys.right || direction === directionKeys.left;
}
function updatePos(pos, direction) {
  const newPos = { ...pos };
  switch (direction) {
    case directionKeys.up:
      newPos.y -= defaultUnit;
      break;
    case directionKeys.down:
      newPos.y += defaultUnit;
      break;
    case directionKeys.left:
      newPos.x -= defaultUnit;
      break;
    case directionKeys.right:
      newPos.x += defaultUnit;
      break;
    default:
      throw new Error("NotFind");
  }
  return newPos;
}
// 检测是否发生碰撞
function isRepeat(list, pos) {
  return list.some(
    ({ pos: itemPos }) => pos.x === itemPos.x && pos.y === itemPos.y
  );
}
// 检测是否超出边界
function isCrossedLine(x, y) {
  return x >= MAX_X || x < 0 || y >= MAX_Y || y < 0;
}

// 生成随机数
function genRandom(max, start) {
  return start + (((Math.random() * (max - start)) / start) >>> 0) * start;
}

export default {
  data: () => ({
    snakeBodyList: [],
    direction: undefined,
    lastInputDirection: undefined,
    lastPos: {},
    foodPos: {},
    isLose: false,
    headerPos: {},
    isPlaying: false,
    speed: 100,
    id: 0,
  }),
  computed: {
    count() {
      return this.isPlaying ? this.snakeBodyList.length - 10 : 0;
    },
  },
  mounted() {
    window.addEventListener("keydown", this.onKeydown);
  },
  methods: {
    init() {
      const initData = { x: 250, y: 250 };
      this.direction = directionKeys.left;
      this.lastInputDirection = directionKeys.left;
      this.lastPos = { ...initData, direction: this.direction };
      this.headerPos = { ...initData };
      this.snakeBodyList = Array(defaultUnit).fill(0).map(this.createBody);
    },
    createBody() {
      const { x, y } = this.lastPos;
      // 判断是否属于需要 -2 的同类型
      const isLower =
        this.direction === directionKeys.up ||
        this.direction === directionKeys.left;
      const pos = {
        ...updatePos(
          { x, y },
          isLower ? this.direction + 2 : this.direction - 2
        ),
      };
      this.lastPos = pos;
      return {
        uuid: this.id++,
        pos,
      };
    },
    genFoot() {
      const x = genRandom(MAX_X, defaultUnit);
      const y = genRandom(MAX_Y, defaultUnit);
      if (isRepeat(this.snakeBodyList, { x, y }) || isCrossedLine(x, y)) {
        this.genFoot();
      } else {
        this.foodPos = { x, y };
      }
    },
    onKeydown({ keyCode }) {
      if (
        // 检查是否为方向键
        ![38, 40, 37, 39].includes(keyCode) ||
        // 检查是否在同一个水平方向上
        checkIsLevel(keyCode) === checkIsLevel(this.direction)
      ) {
        return;
      }
      this.lastInputDirection = keyCode;
    },
    render() {
      const next = updatePos(this.headerPos, this.lastInputDirection);
      if (isCrossedLine(next.x, next.y) || isRepeat(this.snakeBodyList, next)) {
        this.isLose = true;
        this.isPlaying = false;
        clearTimeout(this._timer);
        return console.log("你输了");
      }
      // let head = this.headerPos;
      // const snakeBodyList = this.snakeBodyList;
      // for (const body of snakeBodyList) {
      //   const nextPos = body.pos;
      //   body.pos = head;
      //   head = nextPos;
      // }
      const snakeBodyList = this.snakeBodyList.slice(
        0,
        this.snakeBodyList.length - 1
      );
      snakeBodyList.unshift({ pos: this.headerPos, uuid: this.id++ });
      this.headerPos = next;
      this.lastPos = snakeBodyList[snakeBodyList.length - 1].pos;
      if (isRepeat(snakeBodyList, this.foodPos)) {
        snakeBodyList.push(this.createBody());
        this.genFoot();
      }
      this.snakeBodyList = snakeBodyList;
      this.direction = this.lastInputDirection;
      this._timer = setTimeout(() => this.render(), this.speed);
    },
    playGame() {
      if (this.isPlaying) {
        clearTimeout(this._timer);
      } else {
        this.isLose = false;
        this.init();
        this.genFoot();
        this.render();
      }
      this.isPlaying = !this.isPlaying;
    },
  },
};
</script>
<style>
body {
  display: flex;
  width: 100vw;
  height: 100vh;
  margin: 0;
}
.game-box {
  position: relative;
  width: 500px;
  height: 500px;
  border: 1px solid #ddd;
  margin: auto;
}
.snake-header,
.snake-food,
.snake-body {
  position: absolute;
  top: -9999px;
  left: -9999px;
  width: 10px;
  height: 10px;
  background-color: #ddd;
  border-radius: 50%;
  transition: all 0.1s;
}
.snake-header {
  background-color: #000;
  z-index: 3;
}
.snake-food {
  background-color: rgb(207, 38, 38);
  z-index: 2;
  transition: unset;
}
.tools {
  width: 500px;
  position: absolute;
  top: -60px;
  left: 0;
  display: flex;
  height: 50px;
}
.info-bar {
  margin-left: 10px;
}
.info-bar p {
  margin: 0;
  margin-left: 10px;
}
.count {
  margin-left: auto;
}
</style>
