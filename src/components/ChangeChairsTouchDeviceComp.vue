<template>
  <div class="room">
    <div class="title">
      <p>席がえ</p>
      <p v-show="isEnter">+リカレ</p>
      <p>ント</p>
    </div>
    <div class="inputs">
      <input type="text" class="member input" v-model="memberName">
      <button @touchstart="addMember" class="add__member input">着席</button>
      <button @touchstart="deleteMember" class="input">消す</button>
    </div>
    <div class="shuffle__wrapper">
      <button class="shuffle__btn input" @touchstart="shuffleChairs(chairs)">席替え!!</button>
      <div class="gap__wrapper">
        <p class="gap">間隔</p>
        <div class="btn__wrapper">
          <button class="decrease" @touchstart="decrease">狭く</button>
          <button class="increase" @touchstart="increase">広く</button>
        </div>
      </div>
    </div>
    <transition name="double">
      <p v-show="isDouble.length > 0" class="double">同じ名前があります：{{ isDouble }}</p>
    </transition>
    <p v-show="isStay" class="stay__title">前回と同じ席</p>
    <transition-group tag="ul" mode="out-in" class="stay__chair__wrapper">
      <li v-for="chair in stayChairs" :key="chair" class="stay__chair">
        <p>{{ chair }}</p>
      </li>
    </transition-group><!--stay__chair__wrapper-->
    <p>新しい席</p>
    <transition-group tag="ul" class="chair__wrapper">
      <li v-for="chair in chairs" :key="chair" class="chair" :style="{ 'margin' : '0 ' + gapNumber * 16 + 'px' }">
        <p>{{ chair }}</p>
      </li>
    </transition-group><!--chair__wrapper-->
    <p>今の席</p>
    <transition-group tag="ul" class="current__chair__wrapper">
      <li v-for="chair in currentChairs" :key="chair" class="chair current__chair" :class="{same: chair === chairs[chair] }" :style="{ 'margin' : '0 ' + gapNumber * 16 + 'px' }">
        <p>{{ chair }}</p>
      </li>
    </transition-group><!--chair__wrapper-->
  </div><!--room-->
</template>

<script>
export default {
  data() {
    return {
      chairs: [],
      currentChairs: [],
      stayChairs: [],
      number: 0,
      gapNumber: 0,
      isStay: false,
      memberName: '',
      isDouble: '',
      time: '',
      isEnter: true
    }
  },
  mounted() {
    let count = 2;
    this.time = setInterval(() => {
        if(count > 0) {
          count--;
          if(count === 1) {
            this.isEnter = false;
          }
        } else if(count === 0) {
          clearInterval(this.time);
        }
      }, 1000);
  },
  methods: {
    addMember() {
      console.log(this.memberName);
      if(this.memberName.length > 0) {
        this.chairs.push(this.memberName);
        this.currentChairs.push(this.memberName);
        for(let i = 0; i < this.chairs.length; i++) { 
        const lookChairs = [];
          for(let j = 0; j < this.chairs.length; j++) {
            if(this.chairs[i] === this.chairs[j]) {
              lookChairs.push(this.chairs[j]);
            }
          }
          //同じ名前の入力を禁止する
          if(lookChairs.length === 2) {
            this.chairs.pop(this.memberName);
            this.currentChairs.pop(this.memberName);
            this.isDouble = this.memberName;
            let count = 1;
            this.time = setInterval(() => {
              if(count > 0) {
                count--;
              } else if(count === 0) {
                clearInterval(this.time);
                this.isDouble = '';
              }
            },1000);
          }
        }
      }
      this.memberName = '';
    },
    deleteMember() {
      this.chairs = [];
      this.stayChairs = [];
      this.currentChairs = [];
    },
    decrease() {
      if(this.gapNumber > 0) {
        this.gapNumber--;
      }
    },
    increase() {
      if(this.gapNumber < 3) {
        this.gapNumber++;
      }
      console.log(this.gapNumber)
    },
    shuffleChairs(array) {
      console.log("origin: ",this.chairs);
      this.stayChairs = [];
      this.isStay = false;
      // const copyArray = this.chairs;
      const cloneArray = [...array];
      cloneArray.reduce((prev, current, index) => {
        let rdmIndex = Math.floor(Math.random() * (index + 1));
        //配列の順番を入れ替える
        cloneArray[index] = cloneArray[rdmIndex];
        cloneArray[rdmIndex] = current;
        this.chairs = cloneArray;
      })
      console.log("changed: ",this.chairs);
      console.log(this.currentChairs[0]);
      console.log(this.chairs[0]);
      for(let i = 0; i < this.currentChairs.length; i++) {
        if(this.chairs[i] === this.currentChairs[i]) {
          console.log(this.chairs[i]);
          const stay = this.chairs[i];
          this.stayChairs.push(stay);
          this.isStay = true;
        }
      }
    }
  }
}
</script>

<style scoped>
p {
  text-align: center;
}
.room {
  position: relative;
  width: 100%;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}
.title {
  display: flex;
  justify-content: center;
  width: 100%;
  height: auto;
  background-color: rgba(50, 50, 50, 1);
  margin-bottom: 8px;
}
.title p {
  font-size: 24px;
  text-align: center;
  font-weight: bold;
  color: transparent;
  background: linear-gradient(90deg, rgba(100, 255, 100, 0.9), rgba(255, 100, 100, 0.9));
  background-clip: text;
  animation: enterHeight 2s;
}
@keyframes enterHeight {
  0% {
    font-size: 40px;
    margin-top: 50vh;
    margin-bottom: 50vh;
    min-height: 100vh;
    color: transparent;
    background: linear-gradient(90deg, rgba(255, 100, 100, 0.9), rgba(100, 255, 100, 0.9));
    background-clip: text;
  }
  100% {
    margin-top: 8px;
  }
}
.inputs {
  display: flex;
  width: 100%;
  justify-content: center;
  margin: 0 0 32px;
}
.input {
  border-radius: 8px;
  border: solid 1px;
  width: 64px;
  height: 64px;
  text-align: center;
}
.member {
  width: 128px;
  padding-left: 8px;
  border-radius: 8px 0 0 8px;
  font-size: 24px;
}
.add__member {
  margin-right: 8px;
  border-radius: 0 8px 8px 0;
}
.shuffle__wrapper {
  display: flex;
  justify-content: center;
  width: 100%;
  margin: 0 auto 64px;
}
.shuffle__btn {
  background-color: rgb(250, 150, 150);
  width: 128px;
}
.gap__wrapper {
  display: flex;
  justify-content: center;
}
.gap {
  width: 32px;
  height: 64px;
  line-height: 64px;
  text-align: end;
  margin: 0 0 0 8px;
}
.decrease {
  width: 48px;
  height: 64px;
  border-radius: 8px 0 0 8px;
  border: solid 1px;
}
.increase {
  width: 48px;
  height: 64px;
  border-radius: 0 8px 8px 0;
  border: solid 1px;
}
.double {
  position: absolute;
  top: 112px;
  left: 8px;
  text-align: center;
  width: calc(100% - 16px);
  background-color: rgba(200, 200, 200, 0.5);
  border-radius: 8px;
}
.double-enter-active {
  animation: fadeIn 1s ease;
}
.double-leave-active {
  animation: fadeIn 1s reverse ease;
}
@keyframes fadeIn {
  0% {
    transform: translateX(100%);
    opacity: 0;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}
.chair__wrapper {
  display: flex;
  flex-wrap: wrap;
  margin: 0 0 16px;
  gap: 8px;
  padding: 12px;
  min-height: 60px;
  background-color: rgba(100, 100, 100, 0.5);
  border-radius: 16px;
}
.chair {
  font-size: 12px;
  width: 36px;
  height: 36px;
  list-style: none;
  border-radius: 10px;
  background-color: rgb(250, 250, 250);
  text-align: center;
  line-height: 36px;
}
.v-enter-active, .v-leave-active, .v-move {
  transition: all 1.2s;
}
.v-leave-active {
  position: relative;
}
.v-enter, .v-leave-to {
  opacity: 0;
  transform: translateX(100vw);
}
.stay__chair__wrapper {
  position: absolute;
  top: 220px;
  left: 30vw;
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 16px;
}
.stay__title {
  position: absolute;
  top: 220px;
  left: 8px;
  line-height: 32px;
  text-align: center;
}
.stay__chair {
  font-size: 12px;
  width: 36px;
  height: 36px;
  list-style: none;
  border-radius: 10px;
  color: rgb(250, 250, 250);
  background-color: rgb(250, 150, 150);
  text-align: center;
  line-height: 36px;
}
.current__chair__wrapper {
  display: flex;
  flex-wrap: wrap;
  margin: 0 0;
  min-height: 60px;
  gap: 8px;
  padding: 12px;
  background-color: rgba(200, 200, 200, 0.5);
  border-radius: 16px;
}
.current__chair {
  color: rgb(250, 250, 250);
  background-color: rgb(50, 50, 50);
}
</style>
