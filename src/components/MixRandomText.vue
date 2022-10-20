<template>
  <div class="mix-random-text">
    <div class="mix-random-text__area" ref="workArea">
      <div class="mix-random-text__title" ref="title">
        <span v-for="(char, index) in textChanged"
              :key="'title-' + char + '-' + index"
              class="mix-random-text__title-hidden"
        >{{ char }}</span>
      </div>

      <TransitionGroup tag="span" name="fade" mode="out-in">
              <span v-for="(char, index) in targetChars"
                    class="mix-random-text__char"
                    :class="{ '_active': textVisible }"
                    :key="char + '-' + index"
                    :style="{ 'top': char.top, 'left': char.left, 'font-size': char.fontSize }"
              >{{ char.value }}</span>
      </TransitionGroup>
      <TransitionGroup tag="span" name="fade" mode="out-in">
        <span v-for="(char, index) in basicChars"
              class="mix-random-text__char"
              :key="char.value + '-' + index"
              :style="{ 'top': char.top, 'left': char.left, 'font-size': char.fontSize }"
        >{{ char.value }}</span>
      </TransitionGroup>
    </div>
    <div class="mix-random-text__toolbar">
      <input type="text" v-model="textChanged">
      <button type="button" @click.prevent="magicClick()">{{ textVisible ? 'Mix' : 'Join' }}</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "MixRandomText",
  data() {
    return {
      text: "hello world!",
      textVisible: false,
      basicChars: [],
      targetChars: [],
      areaWidth: null,
      areaHeight: null,
    }
  },
  computed: {
    textChanged: {
      get() {
        return this.text;
      },
      set(newValue) {
        this.targetChars = this.initCharsArray([...newValue], true);
        this.basicChars.forEach((char) => {
          const { top, left, fontSize } = this.generateRandomStyles();
          char.top = top;
          char.left = left;
          char.fontSize = fontSize;
        })
        if (this.textVisible) {
          this.magicClick();
        }
        this.text = newValue;
      }
    },
  },
  methods: {
    magicClick() {
      this.textVisible = !this.textVisible;
      console.log(this.$refs.title.getBoundingClientRect());

      if (!this.textVisible) {
        this.targetChars.forEach((char) => {
          const { top, left, fontSize } = this.generateRandomStyles();
          char.top = top;
          char.left = left;
          char.fontSize = fontSize;
        });
      } else {
        [...this.$refs.title.getElementsByTagName("span")].forEach((el, index) => {
          const elClientRect = el.getBoundingClientRect();
          const targetChar = this.targetChars[index];
          targetChar.top = `${elClientRect.top}px`;
          targetChar.left = `${elClientRect.left}px`;
          targetChar.fontSize = "inherit";
        })
      }
      this.basicChars.forEach((char) => {
        const { top, left, fontSize } = this.generateRandomStyles();
        char.top = top;
        char.left = left;
        char.fontSize = fontSize;
      })
    },
    getRandomInt(min, max) {
      min = Math.ceil(min);
      max = Math.floor(max);
      return Math.floor(Math.random() * (max - min) + min); // The maximum is exclusive and the minimum is inclusive
    },
    generateAlphabet(capital) {
      return [...Array(26)].map((_, i) => String.fromCharCode(i + (capital ? 65 : 97)));
    },
    generateRandomStyles() {
      return {
        top: `${this.getRandomInt(0, this.areaHeight)}px`,
        left: `${this.getRandomInt(0, this.areaWidth)}px`,
        fontSize: `${this.getRandomInt(10, 40)}px`,
      }
    },
    initCharsArray(array, target) {
      return array?.map((char) => ({
          value: char,
          target,
          ...this.generateRandomStyles(),
        })
      );
    }
  },
  mounted() {
    this.areaWidth = this.$refs.workArea.clientWidth;
    this.areaHeight = this.$refs.workArea.clientHeight;

    this.basicChars = this.initCharsArray([...this.generateAlphabet(), ...this.generateAlphabet(true)]);
    this.targetChars = this.initCharsArray([...this.text], true);
  }
}
</script>

<style lang="less">
.mix-random-text {
  display: block;
  position: absolute;
  width: 100%;
  height: 100%;
  background: aliceblue;

  &__title {
    font-size: x-large;
    justify-content: center;
    align-self: center;
    position: relative;
    overflow-wrap: anywhere;
    text-align: center;
    width: 80%;

    &-hidden {
      visibility: hidden;
    }
  }

  &__char {
    position: absolute;
    opacity: 0.1;
    transition: all 2s;

    &._active {
      opacity: 1;
    }
  }

  &__area {
    display: flex;
    justify-content: center;
    height: 80%;
    transition: all 2s;
  }

  &__toolbar {
    display: flex;
    justify-content: center;

    button, input {
      margin: 0 10px;
    }
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 1s ease;
}

.fade-enter-from,
.fade-leave-to {
  transition: opacity 1s ease;
  opacity: 0;
}
</style>