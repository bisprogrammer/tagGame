<template>
  <div class="game">
    <div class="game__header">
      <div class="container">
        <div class="row">
          <div class="col-4">
            <div class="game__header-digit">15</div>
          </div>
          <div class="col-8">
            <div class="game__header-items">
                <div class="game__header-item">
                  <h3>Ходов</h3>
                  <p>{{movesNumber}}</p>
                </div>
                <div class="game__header-item">
                  <h3>Время</h3>
                  <p>{{time}}</p>
                </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="game__content">
      <draggable class="game__items"
                group="digits"
                direction="vertical"
                :animation="200"
                v-model="digits"
                @start="onStart"
                @end="onEnd"
                :move="onMove">
        <div class="game__item"
            :class="{ 'game__item_empty': !digit.value }"
            v-for="(digit, index) in digits"
            :key="index">{{digit.value}}
        </div>
      </draggable>
    </div>
    <div class="game__footer">
      <button class="button bg-primary text-white" @click="startNewGame">Начать новую игру</button>
    </div>
  </div>
</template>

<script>
import draggable from 'vuedraggable'

export default {
  components: { draggable },
  data() {
    return {
      digits: [],
      drag: false,
      movingIndex: 0,
      futureIndex: 0,
      relatedValues: [],
      movesNumber: 0,
      time: "00:00",
      timeBegan: null,
      stopWatchInterval: null
    }
  },
  methods: {
    shuffle(array) {
      return array.sort(() => Math.random() - 0.5);
    },
    getShiffleDigits() {
      let digits = [];
      for (let i = 1; i < 16; i++) {
        digits.push({
          value: i
        });
      }
      digits.push({ value: null });
      digits = this.shuffle(digits);
      return digits;
    },
    onStart() {
      this.relatedValues = [];
    },
    onEnd() {
      if (this.drag && !this.relatedValues.length) {
        this.futureItem = this.digits[this.futureIndex];
        this.movingItem = this.digits[this.movingIndex];
        const _items = Object.assign([], this.digits);
        _items[this.futureIndex] = this.movingItem;
        _items[this.movingIndex] = this.futureItem;

        this.digits = _items;
        this.movesNumber++;

        let isWin = this.isWin();
        if (isWin) {
          this.stopWatchStop();
          this.digits[this.digits.length - 1].value = 16;
        }
      }
    },
    onMove(evt) {
      if (evt.relatedContext.element.value) {
        if (!this.relatedValues.includes(evt.relatedContext.element.value)) {
          this.relatedValues.push(evt.relatedContext.element.value);
        }
        this.drag = false;
        return false;
      }
      const { index, futureIndex } = evt.draggedContext;
      this.movingIndex = index;
      this.futureIndex = futureIndex;
      this.drag = true;
      return false;
    },

    zeroPrefix(num, digit) {
      var zero = '';
      for(var i = 0; i < digit; i++) {
        zero += '0';
      }
      return (zero + num).slice(-digit);
    },
    stopWatchInit() {
      this.timeBegan = new Date();
      this.stopWatchInterval = setInterval(this.stopWatchRun, 1000);
    },
    stopWatchRun(){
      let currentTime = new Date(),
        elapsedTime = new Date(currentTime - this.timeBegan),
        min = elapsedTime.getUTCMinutes(),
        sec = elapsedTime.getUTCSeconds();

      this.time = this.zeroPrefix(min, 2) + ":" + this.zeroPrefix(sec, 2);
    },
    stopWatchStop() {
      clearInterval(this.stopWatchInterval);
    },
    stopWatchReset() {
      clearInterval(this.stopWatchInterval);
      this.time = "00:00";
    },

    isWin() {
      for (let i = 1; i < this.digits.length; i++) {
        if (this.digits[i - 1].value !== i) {
          return false;
        }
      }
      return true;
    },

    startNewGame() {
      this.digits = this.getShiffleDigits();
      this.stopWatchReset();
      this.stopWatchInit();
    }
  },
  mounted() {
    this.startNewGame();
  }
}
</script>

<style lang="scss" scoped>
.game {
  width: 460px;
  margin: auto;
  background: #ccc;
  &__items {
    display: flex;
    flex-wrap: wrap;
  }
  &__item {
    width: 100px;
    height: 100px;
    background: #cccccc;
    display: flex !important;
    justify-content: center;
    align-items: center;
    border-radius: 5px;
    margin-bottom: 10px;
    &:not(:nth-child(4n)) {
      margin-right: 10px;
    }
    &_empty {
      background: transparent;
    }
    span {
      font-size: 42px;
      font-weight: bold;
    }
  }
  &__header {
    padding: 20px;
    &-digit {
      font-size: 80px;
      font-weight: bold;
      display: flex;
      align-items: center;
      height: 100%;
      line-height: 1;
    }
    &-items {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100%;
    }
    &-item {
      padding: 10px 15px;
      background: gray;
      text-align: center;
      height: 100%;
      margin-right: 10px;
      display: flex;
      justify-content: center;
      flex-direction: column;
      width: 80px;
      height: 60px;
      h3 {
        color: #ddd;
        font-size: 16px;
      }
      p {
        color: #fff;
        margin: 0;
      }
    }
  }
  &__content {
    padding: 15px 15px 5px;
    background: gray;
  }
  &__footer {
    overflow: hidden;
  }
  .button {
    display: block;
    width: 300px;
    padding: 10px 15px;
    border-radius: 5px;
    text-align: center;
    border: none;
    outline: none;
    margin: 40px auto;
  }
}
</style>
