<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <HelloWorld v-if="showHelloWorld" msg="Welcome to Your Vue.js + TypeScript App"/>
    <SnakeGame v-else/>
    <button id="toggle-game" v-on:click="HandlePlaySnakeGame('ok', $event)">{{ buttonText }}</button>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import HelloWorld from './components/HelloWorld.vue';
import SnakeGame from './components/SnakeGame.vue';


export default Vue.extend({
  name: 'App',
  components: {
    HelloWorld,
    SnakeGame
  },
  methods:{    
    HandlePlaySnakeGame(message:string, event:Event) {
      // 这里是你的事件处理逻辑
      console.log(message); // 输出: 'ok'
      console.log(event.target); // 输出: 点击的按钮元素
      console.dir(event); // 输出：event内容
      console.log(event instanceof Event); // true
      this.showHelloWorld = !this.showHelloWorld;
      if (this.buttonText === 'PlaySnakeGame') {
        this.buttonText = 'GoBack';
      } else {
        this.buttonText = 'PlaySnakeGame';
      }
    }
  },
  data(){
    return {
      showHelloWorld: true,
      buttonText: 'PlaySnakeGame'
      }
  }
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  margin-left: auto;
  margin-right: auto;
  width: 50%; /* 或者其他固定宽度 */
}
</style>
