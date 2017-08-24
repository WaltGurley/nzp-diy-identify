<template>
  <div class="card">
    <div class="card-front will-flip" v-bind:class="{ flippedToFront: isFlipped }">
      <div class="img-holder">
        <img
          v-bind:src="imageSource"
          class="img-responsive"
          alt="Can you tell what is in this image?"
        >
      </div>
      <h4 class="card-header question">What is this?</h4>
      <div class="buttons">
        <button
          v-for="entity in entities"
          v-on:click="checkAnswer(entity.name); flipped()"
          class="card-button button-front"
        >
          {{entity.name}}
        </button>
      </div>
    </div>
    <div class="card-back will-flip" v-bind:class="{ flippedToBack: !isFlipped }">
      <div class="img-holder">
        <img
          v-bind:src="imageSource"
          class="img-responsive"
          alt="Can you tell what is in this image?"
        >
      </div>
      <div v-show="isCorrect" class="correct-answer">
        <h2 class="card-header">You are correct!</h2>
        <h2 class="card-header">{{correctImage.name}}</h2>
        <h2 class="card-header">{{correctImage.info}}</h2>
      </div>
      <div v-show="!isCorrect" class="incorrect-answer">
        <h2 class="card-header">Sorry, that's incorrect.</h2>
      </div>
    </div>
  </div>
</template>

<script>
// import Sheetsy from 'sheetsy'
import _ from 'lodash'

// const spreadsheetKey = Sheetsy.urlToKey('https://docs.google.com/spreadsheets/d/1VXAyesupiwGDHI1akAm6YvqUTTh0_NLemjz7USR9nuw/edit?usp=sharing')

export default {
  name: 'card',
  props: ['entities', 'correctImage'],
  data () {
    return {
      isFlipped: false,
      isCorrect: false,
      randomThree: [],
      currentImage: {}
    }
  },
  methods: {
    setCurrentImage: function () {
      this.currentImage = _.find(this.entities, entity => !entity.identified)
    },
    // Check if the choice matches the image
    checkAnswer: function (name) {
      if (this.correctImage.name === name) {
        this.correctImage.identified = true
        this.isCorrect = true
        this.$emit('identified')
      } else this.isCorrect = false
    },
    // Flip the card
    flipped: function () {
      this.isFlipped = !this.isFlipped
    }
  },
  computed: {
    // Set the source of the current image
    imageSource: function () {
      if (this.correctImage.name !== undefined) {
        let name = this.correctImage.name.split(' ').join('_')
        var imgSrc = './static/images/' + name + '.jpg'
      }
      return imgSrc
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  $card-width: 30vw;
  .card {
    position: absolute;
    width: $card-width;
    height: 8/5 * $card-width;
    min-width: 250px;
    min-height: 400px;
    max-width: 500px;
    max-height: 800px;
    perspective: $card-width * 10;

    .card-header {
      margin-top: 0.2em;
      margin-bottom: 0.2em;
      text-align: center;
      font-size: 1.2em;
    }

    .card-button {
      height: auto;
      min-height: 2.4em;
      margin-top: 0.85em;
    }

    .card-front {
      width: 100%;
      height: 100%;
      background-color: #fff;

      .img-holder {
        width: 100%;
        // height: 50%;

        img {
          border-top-right-radius: 6px;
          border-top-left-radius: 6px;
        }
      }

      .buttons {
        height: 40%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;

        .button-front {
          width: 75%;
        }
      }
    }

    .card-back {
      position: relative;
      top: -100%;
      left: 0;
      width: 100%;
      height: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
      background-color: #fff;

      .button-back {
        padding-left: 1em;
        padding-right: 1em;
      }
    }

    .will-flip {
      border-style: solid;
      border-width: 1px;
      border-radius: 6px;
      border-color: #CCCCCC;
      box-shadow: 10px 10px 20px #CCCCCC;
      backface-visibility: hidden;
      transition-property: all;
      transition-duration: 0.6s;
      transtition-timing-function: ease-in-out;
      transition-timing-function: cubic-bezier(0.25, -0.5, 0.25, 1.5);
    }

    .flippedToFront {
      transform: rotateY(-180deg);
      box-shadow: -10px 10px 20px #CCCCCC;
    }

    .flippedToBack {
      transform: rotateY(180deg);
      box-shadow: 10px 10px 20px #CCCCCC;
    }
  }
</style>
