<template>
  <div class="outer">
    <p class="catched__chair" v-show="ghostName" :style="{ 'top' : catchedY +'vh' , 'left': catchedX +'vw', 'background-color': chairColors[chairColorNumber].color}">{{ catchedName }}</p>
    <div class="header">
      <p class="wf-roundedmplus1c">せきふる</p>
      <div class="colors__wrapper">
        <button class="chair__color__btn" @touchstart="lookChairColor" :style="{'background-color': chairColors[chairColorNumber].color}"></button>
        <button class="room__color__btn" @touchstart="lookRoomColor" :style="{'background-color': roomColors[roomColorNumber].color}"></button>
        <transition-group tag="ul" name="color" class="chair__colors">
          <li v-for="color in chairColors" :key="color.id" v-show="isChairColorSelect">
            <p class="chair__color" :style="{'background-color': color.color, 'transform': 'rotate('+ color.id *-35 +'deg)', 'transform-origin': '-35px 0px'}" @touchstart="selectChairColor(color.id)"></p>
          </li>
        </transition-group>
        <transition-group tag="ul" name="color" class="room__colors">
          <li v-for="color in roomColors" :key="color.id" v-show="isRoomColorSelect">
            <p class="room__color" :style="{'background-color': color.color, 'transform': 'rotate('+ color.id *-35 +'deg)', 'transform-origin': '-35px 0px'}" @touchstart="selectRoomColor(color.id)"></p>
          </li>
        </transition-group>
      </div>
    </div><!--header-->
    <div class="room" :style="{'background-color': roomColors[roomColorNumber].color}">
      <div class="shuffle__room">
        <transition-group tag="ul" class="shuffle__chairs__wrapper" :style="{'width': (64 * spreadNumber) +( 8 * (spreadNumber + 1)) + 'px', 'gap': '8px '+ ((72 * gapNumber) + 8 ) + 'px'}">
          <li v-for="chair in shuffleChairs" :key="chair.id" class="chair" :class="{ghost:catchedName && chair.name === catchedName, alert__anim:isAlert}" @touchstart="catchedChair(chair.name)" @touchmove="moveCatchedChair">
            <p :class="{empty:!chair.stay, catched: chair.name === catchedName}">{{ chair.name }}</p>
          </li>
        </transition-group>
        <ul class="all__chairs__wrapper" :style="{'width': (64 * spreadNumber) +( 8 * (spreadNumber + 1)) + 'px'}">
          <li v-for="chairs in roomChairs" :key="chairs" class="room__chair" :class="{room__chair__hide:isSettingOpen}">
            <p></p>
          </li>
        </ul>
        <div class="room__x__y">
          <p>{{ roomY }}</p><!--縦の数-->
          <p>{{ spreadNumber }}</p><!--横の数-->
        </div>
      </div><!--shuffle__room-->
      <transition name="current">
        <div v-if="isCheck" class="current__wrapper" :class="{current__hide:!isCurrentOpen}" :style="{'bottom': currentY +'px', 'right': currentX +'px'}" @touchmove="moveCurrent">
          <transition-group tag="ul" class="current__chairs__wrapper" :style="{'width': (36 * spreadNumber) +( 6 * (spreadNumber + 1)) + 'px', 'gap': '6px '+ ((42 * gapNumber) + 6 ) +'px'}">
            <li v-for="chair in currentChairs" :key="chair.id" class="chair current__chair" :class="{no__move:chair.move === false && chair.stay === true, no__stay:chair.stay === false}">
              <p>{{ chair.name }}</p>
            </li>
          </transition-group><!--current__chairs__wrapper-->
          <ul class="all__chairs__wrapper__current" :style="{'width': (36 * spreadNumber) +( 6 * (spreadNumber + 1)) + 'px'}">
            <li v-for="chairs in roomChairs" :key="chairs" class="room__chair__current" :class="{room__chair__hide:isSettingOpen}">
              <p></p>
            </li>
          </ul>
        </div><!--current__wrapper-->
      </transition>
      <button class="return__btn" @touchstart="returnCurrent" v-if="isCheck"></button>
      <button class="current__btn" :class="{current__close:!isCurrentOpen}" @touchstart="lookOrHide" v-if="isCheck">
        <p></p>
      </button>
    </div><!--room-->
    <transition name="setting" mode="out-in">
      <div class="setting" v-show="isSettingOpen">
        <div class="add">
          <div class="base__set" v-show="isBaseSet">
            <div class="chairs">
              <p>シート</p>
              <button class="decrease" :class="{no__decrease:shuffleChairs.length === 0 || useChairsNumber === 0}" @touchstart="decrease">
                <p></p>
              </button>
              <div class="chair__number">
                <button class="plus" @touchstart="chairPlus" @touchend="plusEnd">+</button>
                <input type="number" min="0" v-model="useChairsNumber">
                <button class="minus" @touchstart="chairMinus" @touchend="minusEnd">-</button>
              </div><!--chair__number-->
              <button class="increase" :class="{no__increase:useChairsNumber === 0}" @touchstart="increase">
                <p></p>
              </button>
            </div><!--chairs-->
            <div class="wide">
              <p>ライン</p>
              <button class="decrease" :class="{no__decrease:spreadNumber === 1}" @touchstart="narrowWide">
                <p></p>
              </button>
              <p>{{ spreadNumber }}</p>
              <button class="increase" @touchstart="spreadWide">
                <p></p>
              </button>
            </div><!--wide-->
            <div class="chairs__gap">
              <p>スペース</p>
              <button class="decrease" :class="{no__decrease:gapNumber === 0}" @touchstart="narrowGap">
                <p></p>
              </button>
              <p>{{ gapNumber }}</p>
              <button class="increase" :class="{no__increase: gapNumber === spreadNumber - 1}" @touchstart="spreadGap">
                <p></p>
              </button>
            </div><!--gap-->
          </div><!--base__set-->
          <button class="change" :class="{to__base:!isBaseSet}" @touchstart="changeSet">
            <p>{{ isBaseSet ? "メンバー" : "ルーム" }}</p>
          </button>
          <div class="member__set" v-show="!isBaseSet">
            <ul class="inputs__wrapper">
              <li v-for="(chair, index) in shuffleChairs" :key="index">
                <input type="text" maxlength="4" class="member input" :placeholder="index +1" :class="{grey:typeof shuffleChairs[index].name !== 'string', duplicate:shuffleChairs[index].name === duplicateName}" v-model="shuffleChairs[index].name" @change="checkName(chair.name)" @touchstart="clearName(index)" @blur="resetName(index)">
              </li>
            </ul><!--member__set-->
            <button @touchstart="addMember" class="add__member input" :class="{off:shuffleChairs.length === 0}">すわる</button>
          </div>
        </div><!--add-->
        <button @touchstart="deleteMember" class="input reset" :class="{off:shuffleChairs.length === 0}">リセット</button>
      </div><!--setting-->
    </transition>
    <div class="footer">
      <div class="inputs">
        <button class="roomset__btn input" :class="{roomset__touch:isTouch}" @touchstart="openCloseSetting">{{ !isSettingOpen ? "ルーム＆メンバー" : "とじる" }}</button>
        <button class="shuffle__btn input" @touchstart="shuffle(shuffleChairs)" :class="{shuffle__start:isActive || stayChairNumber === 0}" :style="{'background-color': chairColors[chairColorNumber].color}">シャッフル</button>
      </div>
    </div><!--footer-->
  </div><!--outer-->
</template>

<script>
export default {
  data() {
    return {
      roomColors: [
        {
          id: 1,
          color: 'rgba(250, 250, 250, 0.5)'
        },
        {
          id: 2,
          color: 'rgba(250, 0, 0, 0.5)'
        },
        {
          id: 3,
          color: 'rgba(250, 150, 150, 0.5)'
        },
        {
          id: 4,
          color: 'rgba(250, 250, 150, 0.5)'
        },
        {
          id: 5,
          color: 'rgba(0, 250, 150, 0.5)'
        },
        {
          id: 6,
          color: 'rgba(150, 250, 250, 0.5)'
        },
        {
          id: 7,
          color: 'rgba(0, 150, 150, 0.5)'
        },
        {
          id: 8,
          color: 'rgba(0, 150, 250, 0.5)'
        },
        {
          id: 9,
          color: 'rgba(0, 0, 250, 0.5)'
        },
        {
          id: 10,
          color: 'rgba(0, 0, 0, 0.5)'
        }
      ],
      chairColors: [
        {
          id: 1,
          color: 'rgba(250, 150, 0, 0.8)'
        },
        {
          id: 2,
          color: 'rgba(250, 0, 0, 0.8)'
        },
        {
          id: 3,
          color: 'rgba(250, 250, 150, 0.8)'
        },
        {
          id: 4,
          color: 'rgba(0, 250, 150, 0.8)'
        },
        {
          id: 5,
          color: 'rgba(150, 250, 250, 0.8)'
        },
        {
          id: 6,
          color: 'rgba(0, 150, 150, 0.8)'
        },
        {
          id: 7,
          color: 'rgba(0, 150, 250, 0.8)'
        },
        {
          id: 8,
          color: 'rgba(0, 0, 250, 0.8)'
        }
      ],
      roomColorNumber: 0,
      chairColorNumber: 0,
      isRoomColorSelect: false,
      isChairColorSelect: false,
      shuffleChairs: [ //メインで動かす配列
        // {
        //   id: 1,
        //   name: 'りか',
        //   stay: true
        // },
        // {
        //   id: 2,
        //   name: '',
        //   stay: false
        // }
      ],
      currentChairs: [], //現在の席を保存するための配列。画面の左上に表示される。
      roomChairs: [], //背景に使用する配列。
      useChairsNumber: 0, //設定画面で増減させる席数
      duplicateName: '', //名前がダブっているときに表示させる
      isDouble: '',
      time: '',
      isActive: false,
      shuffleCount: 0,
      catchIndex: null,
      dropedIndex: null,
      catchedName: '',
      ghostName: '',
      catchedX: 0,
      catchedY: 0,
      isCheck: false,
      spreadNumber: 4, //列
      isSettingOpen: false,
      isTouch: false,
      isBaseSet: true,
      gapNumber: 0,
      shuffleHeight: 0,
      isCurrentOpen: true,
      numberName: null,
      numberIndex: null,
      currentX: 8,
      currentY: 120,
      isAlert: false
    }
  },
  computed: {
    //席に名前が１つでも存在すればシャッフル可能であることを視覚的に表示する
    stayChairNumber() {
      let stayChairNumber = 0;
      for (let i = 0; i < this.shuffleChairs.length; i++) {
        if(this.shuffleChairs[i].stay === true) {
          stayChairNumber++;
        }
      }
      return stayChairNumber;
    },
    roomY() {
      return this.shuffleHeight > 0 ? (this.shuffleHeight - 8) /72 : 0;
    },
  },
  methods: {
    //color__setting//
    lookRoomColor() {
      if(!this.isChairColorSelect && !this.isSettingOpen) {
        this.isRoomColorSelect = !this.isRoomColorSelect;
      }
    },
    lookChairColor() {
      if(!this.isRoomColorSelect && !this.isSettingOpen) {
        this.isChairColorSelect = !this.isChairColorSelect;
      }
    },
    selectRoomColor(id) {
      if(!this.isSettingOpen) {
        this.roomColorNumber = id -1;
        this.isRoomColorSelect = !this.isRoomColorSelect;
      }
    },
    selectChairColor(id) {
      if(!this.isSettingOpen) {
        this.chairColorNumber = id -1;
        this.isChairColorSelect = !this.isChairColorSelect;
      }
    },

    //setting//
    openCloseSetting() {
      if(!this.isRoomColorSelect && !this.isChairColorSelect) {
        this.isSettingOpen = !this.isSettingOpen;
        this.isTouch = true;
        this.useChairsNumber = 0;
        let count = 1;
        let time = setInterval(() => {
          if(count > 0) {
            count--;
          } else if(count === 0) {
            clearInterval(time);
            this.isTouch = false;
          }
        }, 150);
        if(!this.isSettingOpen) {
          //空席も含めたすべての席を配置する。
          this.allChairsLineUp();
        }
      }
    },
    allChairsLineUp() {
      this.shuffleHeight = document.getElementsByClassName('shuffle__chairs__wrapper')[0].clientHeight;
      console.log(this.shuffleHeight)
      const roomChairsNumber = ((this.shuffleHeight - 8) /72)*this.spreadNumber;
      this.roomChairs = [];
      for(let i = 0; i < roomChairsNumber; i++) {
        let chair = i;
        this.roomChairs.push(chair);
      }
    },
    //base__set//
    //増減させる椅子の数を調整する処理
    chairPlus() {
      this.useChairsNumber++;
      this.plusContinue();
    },
    plusContinue() {
      this.time = setInterval(() => {
        this.useChairsNumber++;
      }, 150);
    },
    plusEnd() {
      clearInterval(this.time);
    },
    chairMinus() {
      if(this.useChairsNumber > 0) {
        this.useChairsNumber--;
        this.minusContinue();
      }
    },
    minusContinue() {
      this.time = setInterval(() => {
        if(this.useChairsNumber > 0) {
          this.useChairsNumber--;
        }
      }, 150);
    },
    minusEnd() {
      clearInterval(this.time);
    },
    increase() {
      console.log("increase");
      let maxId = 0;
      if(this.shuffleChairs.length === 0) {
        maxId = 0;
        console.log(maxId);
      }else if(this.shuffleChairs.length > 0) {
        for(let i = 0; i < this.shuffleChairs.length; i++) {
          if(maxId < this.shuffleChairs[i].id) {
            maxId = this.shuffleChairs[i].id;
            console.log(maxId);
          }
        }
      }
      console.log(maxId);
      for(let i = 1; i < this.useChairsNumber*1 +1; i++) {
        let newChair = {
        id: i + maxId,
        name: i + maxId,
        stay: false
        }
        this.shuffleChairs.push(newChair);
      }
      console.log(this.shuffleChairs);
      this.currentChairs = this.shuffleChairs;
      this.checkStayChairs();
    },
    decrease() {
      console.log("decrease");
      for(let i = 0; i < this.useChairsNumber; i++) {
        if(this.shuffleChairs.length > 0) {
          if(this.shuffleChairs[this.shuffleChairs.length - 1].stay === false) {
            console.log(this.shuffleChairs[this.shuffleChairs.length - 1].stay);
            this.shuffleChairs.pop();
          }
        }
      }
      console.log(this.shuffleChairs);
      this.currentChairs = this.shuffleChairs;
    },
    //列の幅を１から２０の間に制限して調整できる処理
    spreadWide() {
      if(this.spreadNumber < 20) {
        this.spreadNumber++;
      }
    },
    narrowWide() {
      if(this.spreadNumber > 1) {
        this.spreadNumber--;
      }
    },
    narrowGap() {
      if(this.gapNumber > 0) {
        this.gapNumber--;
      }
    },
    spreadGap() {
      if(this.gapNumber < (this.spreadNumber - 1)) {
        this.gapNumber++;
        console.log(this.gapNumber);
      }
    },
    changeSet() {
      this.isBaseSet = !this.isBaseSet;
    },
    //member__set//
    clearName(index) {
      this.numberName = this.shuffleChairs[index].name;
      console.log("numberName:",this.numberName);
      this.shuffleChairs[index].name = '';
      this.numberIndex = index;
    },
    resetName(index) {
      if(this.shuffleChairs[index].name === '') {
        this.shuffleChairs[index].name = this.numberName;
      }
    },
    checkName(value) {
      this.checkduplicateName(value);
      if(value === '') {
        this.shuffleChairs[this.numberIndex].name = this.numberName;
      }
      console.log(this.shuffleChairs);
    },
    checkduplicateName(value) {
      this.duplicateName = '';
      console.log(value);
      let duplicateCount = 0;
      for(let i = 0; i < this.shuffleChairs.length; i++) {
        if(this.shuffleChairs[i].name === value) {
          duplicateCount++;
          if(duplicateCount === 2) {
            console.log("duplicate!!");
            this.duplicateName = value;
            let count = 1;
            this.time = setInterval(() => {
              if(count > 0) {
                count--;
              } else if(count === 0) {
                clearInterval(this.time);
                this.shuffleChairs[i].name = '';
                this.duplicateName = '';
              }
            }, 500);
          }
        }
      }
      duplicateCount = 0;
      console.log(duplicateCount);
    },
    addMember() {
      if(this.shuffleChairs.length > 0) {
        console.log(this.shuffleChairs);
        for(let i = 0; i < this.shuffleChairs.length; i++) {
          //名前が存在する場合に"stay"の状態を"true"にする
          if(typeof this.shuffleChairs[i].name === "string") {
          this.shuffleChairs[i].stay = true;
          }
        }
      }
    },
    deleteMember() {
      this.shuffleChairs = [];
      this.currentChairs = [];
      this.shuffleCount = 0;
      this.isCheck = false;
    },

    //shuffle__room//
    shuffle(array) {
      if(!this.isSettingOpen) {
          // isActiveがfalseのときのみ席替えボタンを押せるようにしている
        if(this.shuffleChairs.length > 0 && !this.isActive) {
          // シャッフルされる前の席順を記録する
          if(this.shuffleCount === 0) {
            this.currentChairs = this.shuffleChairs;
          }
          this.shuffleCount++;
          this.isActive = true;
          const cloneArray = [...array];
          cloneArray.reduce((prev, current, index) => {
            let rdmIndex = Math.floor(Math.random() * (index + 1));
            //配列の順番を入れ替える
            cloneArray[index] = cloneArray[rdmIndex];
            cloneArray[rdmIndex] = current;
            this.shuffleChairs = cloneArray;
          })
          console.log("changed: ",this.shuffleChairs);
          this.checkStayChairs();
          this.isCheck = true;
        }
      }
    },
    returnCurrent() {
      this.shuffleChairs = this.currentChairs;
      this.checkStayChairs();
    },
    // 前回とcurrentChairsとshuffleChairsを調べてダブっている席を見つける
    checkStayChairs() {
      for(let i = 0; i < this.currentChairs.length; i++) {
        if(this.shuffleChairs[i] === this.currentChairs[i]) {
          this.currentChairs[i].move = false;
        } else if(this.shuffleChairs[i] !== this.currentChairs[i]) {
          this.currentChairs[i].move = true;
        }
      }
      // 席替え中に誤って席替えを行わないようにしている
      let count = 1;
      let time = setInterval(() => {
        if(count > 0) {
          count--;
        } else if(count === 0) {
          clearInterval(time);
          this.isActive = false;
        }
      }, 500);
    },
    // 席をドラッグで個別に入れ替えるメソッド //
    catchedChair(name) {
      if(this.catchIndex === null) {
        console.log("未選択です");
        console.log("catched:",name);
        for(let i = 0; i < this.shuffleChairs.length; i++) {
          if(this.shuffleChairs[i].name === name) {
            this.catchIndex = i;
            console.log("catchindex:",i);
            if(this.shuffleChairs[i].stay === true) {
              this.catchedName = name;
            }
            if(this.shuffleChairs[i].stay === false) {
              this.catchedName = '空席';
            }
          }
        }
      } else if(this.catchIndex !== null) {
        this.dropedChair(name);
      }
    },
    moveCatchedChair(e) {
      this.catchedX = e.changedTouches[0].clientX / window.innerWidth *100;
      this.catchedY = e.changedTouches[0].clientY / window.innerHeight *100;
      if(this.ghostName === '') {
        this.ghostName = this.catchedName;
      }
    },
    dropedChair(name) {
      this.catchedName = '';
      this.ghostName = '';
      this.catchedX = 0;
      this.catchedY = 0;
      console.log("droped:",name);
      for(let i = 0; i < this.shuffleChairs.length; i++) {
        if(this.shuffleChairs[i].name === name) {
          console.log("dropedindex:",i);
          this.dropedIndex = i;
        }
      }
      if(this.catchIndex === this.dropedIndex) {
        this.alertAnim();
      } else if(this.catchIndex !== this.dropedIndex) {
        let catchCopy = this.shuffleChairs[this.catchIndex];
        let dropedCopy = this.shuffleChairs[this.dropedIndex];
        //どちらの椅子も空席の場合は入れ替えない
        if(catchCopy.stay === false && dropedCopy.stay === false) {
          this.alertAnim();
        //それ以外の場合、席を入れ替える
        } else {
          this.shuffleChairs.splice(this.catchIndex, 1, dropedCopy);
          this.shuffleChairs.splice(this.dropedIndex, 1, catchCopy);
        }
      }
      this.catchIndex = null;
      this.dropedIndex = null;
      this.checkStayChairs();
    },
    alertAnim() {
      let count = 1;
      this.time = setInterval(() => {
        if(count > 0) {
          this.isAlert = true;
          count--;
        } else if(count === 0) {
          clearInterval(this.time);
          this.isAlert = false;
        }
      })
    },
    //current//
    moveCurrent(e) {
      const currentWidth = document.getElementsByClassName('current__wrapper')[0].clientWidth;
      const currentHeight = document.getElementsByClassName('current__wrapper')[0].clientHeight;
      const currentTop = document.getElementsByClassName('current__wrapper')[0].offsetTop;
      const currentLeft = document.getElementsByClassName('current__wrapper')[0].offsetLeft;
      this.currentX = this.currentX - ((e.changedTouches[0].clientX - currentLeft) - currentWidth / 2);
      this.currentY = this.currentY - ((e.changedTouches[0].clientY - currentTop) - currentHeight / 2);
    },
    lookOrHide() {
      this.isCurrentOpen = !this.isCurrentOpen;
      this.currentX = 8;
      this.currentY = 120;
    }
  }
}
</script>

<style scoped>
.header {
  width: 100%;
  height: 68px;
  line-height: 68px;
  font-size: 28px;
  color: rgba(250, 250, 250, 0.7);
  background: linear-gradient(rgba(50, 50, 50, 1), rgba(50, 50, 50, 0.5));
  backdrop-filter: blur(2px);
  margin: 0 auto;
  z-index: 10;
}

.room {
  position: relative;
  width: 100%;
  min-height: 100vh;
  margin: 0 auto;
  transition: 1s ease;
  z-index: 1;
}
.shuffle__room {
  position: relative;
  margin: 0 8px;
  display: flex;
  align-items: center;
  min-height: calc(100vh - 74px);
  overflow: scroll;
  z-index: 1;
}

/* setting */
.setting {
  width: 336px;
  height: 366px;
}

.base__set {
  width: 244px;
  height: 290px;
}
.change {
  width: 76px;
  height: 36px;
  background-color: rgba(0, 0, 0, 0);
  border: none;
}
.change p {
  position: relative;
  width: 76px;
  height: 36px;
  text-align: center;
  line-height: 36px;
  color: rgba(250, 250, 250, 1);
}
.change p::before {
  content: '';
  position: absolute;
  top: 4px;
  right: 0;
  width: 36px;
  height: 6px;
  background-color: rgba(250, 250, 250, 1);
  border-radius: 3px;
  transform: rotate(45deg);
  transition: 1s ease;
}
.change p::after {
  content: '';
  position: absolute;
  bottom: 4px;
  right: 0;
  width: 36px;
  height: 6px;
  background-color: rgba(250, 250, 250, 1);
  border-radius: 3px;
  transform: rotate(-45deg);
  transition: 1s ease;
}
.to__base p {
  text-align: center;
}
.to__base p::before {
  content: '';
  position: absolute;
  top: 4px;
  left: 0;
  width: 36px;
  height: 6px;
  background-color: rgba(250, 250, 250, 1);
  border-radius: 3px;
  transform: rotate(-45deg);
  transition: 1s ease;
}
.to__base p::after {
  content: '';
  position: absolute;
  bottom: 4px;
  left: 0;
  width: 36px;
  height: 6px;
  background-color: rgba(250, 250, 250, 1);
  border-radius: 3px;
  transform: rotate(45deg);
  transition: 1s ease;
}

.member__set {
  width: 244px;
  height: 290px;
}
.inputs__wrapper {
  height: 290px;
}


.catched {
  width: 64px;
  height: 64px;
  box-shadow: inset rgb(50, 50, 50) 3px 3px 6px;
  border-radius: 8px;
}
.current__wrapper {
  position: relative;
  position: fixed;
  transition: 1s;
  z-index: 1000;
}
.current__hide {
  opacity: 0;
  transition: 1s;
}
.return__btn {
  position: fixed;
  bottom: 76px;
  right: 52px;
  width: 36px;
  height: 36px;
  border: none;
  background-color: rgba(50, 50, 50, 0.5);
  backdrop-filter: blur(1px);
  border-radius: 6px;
  z-index: 1000;
}
.return__btn::before {
  content: '';
  position: absolute;
  top: 6px;
  right: 6px;
  width: 12px;
  height: 24px;
  border-top: solid rgb(250, 250, 250) 3px;
  border-right: solid rgb(250, 250, 250) 3px;
  border-bottom: solid rgb(250, 250, 250) 3px;
  border-radius: 3px 3px 3px 0;
}
.return__btn::after {
  content: '';
  position: absolute;
  bottom: 6px;
  left: 6px;
  width: 12px;
  height: 12px;
  border-bottom: 12px solid rgb(250, 250, 250);
  border-left: 12px solid transparent;
}
.current__btn {
  position: fixed;
  bottom: 76px;
  right: 8px;
  width: 36px;
  height: 36px;
  border: none;
  background-color: rgba(50, 50, 50, 0.5);
  backdrop-filter: blur(1px);
  border-radius: 6px;
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
}
.current__btn p {
  background-color: rgb(250, 250, 250);
  width: 12px;
  height: 12px;
  border-radius: 50%;
  transition: 0.5s ease;
}
.current__btn::before {
  content: '';
  position: absolute;
  top: 10px;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
  border-top: solid rgb(250, 250, 250) 2px;
  border-radius: 50%;
  width: 30px;
  height: 18px;
  transition: 0.5s ease;
  transform: translateY(-5px);
}
.current__btn::after {
  content: '';
  position: absolute;
  top: 0;
  bottom: 10px;
  left: 0;
  right: 0;
  margin: auto;
  border-bottom: solid rgb(250, 250, 250) 2px;
  border-radius: 50%;
  width: 30px;
  height: 18px;
  transition: 0.5s ease;
  transform: translateY(5px);
}
.current__close p {
  width: 0;
  height: 0;
  transition: 0.5s ease;
}
.current__close::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  margin: auto;
  background-color: rgb(250, 250, 250);
  border-radius: 2px;
  width: 24px;
  height: 1px;
  transition: 0.5s ease;
  transform: translateY(0);
}
.current__close::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  margin: auto;
  background-color: rgb(250, 250, 250);
  border-radius: 2px;
  width: 24px;
  height: 1px;
  transition: 0.5s ease;
  transform: translateY(0);
}
.current__chairs__wrapper {
  display: flex;
  flex-wrap: wrap;
  min-height: 48px;
  gap: 6px;
  padding: 6px;
  background-color: rgba(50, 50, 50, 0.5);
  backdrop-filter: blur(2px);
  border-radius: 12px;
  z-index: 10;
}
.current__chair {
  line-height: 36px;
  width: 36px;
  height: 36px;
  font-size: 12px;
  color: rgb(50, 50, 50);
  background-color: rgba(250, 250, 250, 0.5);
  border-radius: 6px;
}
.current__chair p {
  width: 36px;
  height: 36px;
  overflow: hidden;
}
.no__move {
  color: rgb(250, 250, 250);
  background-color: rgba(50, 50, 50, 0.5);
}
.no__stay {
  color: rgba(50, 50, 50, 0);
}
.shuffle__chairs__wrapper {
  position: absolute;
  top: 8px;
  left: 0px;
  display: flex;
  flex-wrap: wrap;
  min-height: 80px;
  padding: 8px;
  background-color: rgba(250, 250, 250, 0.5);
  box-shadow: inset rgba(50, 50, 50, 0.5) 2px 2px 4px;
  border-radius: 16px;
  animation: appear 2s;
  z-index: 1;
}
.all__chairs__wrapper {
  position: absolute;
  top: 8px;
  left: 0px;
  display: flex;
  flex-wrap: wrap;
  align-content: center;
  justify-content: center;
  width: 100%;
  min-height: 80px;
  padding: 8px;
  border-radius: 16px;
  gap: 8px;
  z-index: 0;
}
.all__chairs__wrapper__current {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin: auto;
  display: flex;
  flex-wrap: wrap;
  align-content: center;
  justify-content: center;
  width: 100%;
  min-height: 48px;
  padding: 6px;
  border-radius: 12px;
  gap: 6px;
  z-index: 0;
}
.ghost {
  color: rgba(200, 200, 200, 0.5);
}


</style>
