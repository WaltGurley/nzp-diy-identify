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
      <div class="buttons">
        <button
          v-for="entity in randomThree"
          v-on:click="flipped(); checkAnswer(entity.name)"
          class="card-button"
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
        <h2>CORRECT!</h2>
      </div>
      <div v-show="!isCorrect" class="incorrect-answer">
        <h2>Sorry, that's incorrect.</h2>
      </div>
      <button v-on:click="getRandomThreeSetImg(); flipped()">NEXT</button>
    </div>
  </div>
</template>

<script>
// import Sheetsy from 'sheetsy'
import _ from 'lodash'

// const spreadsheetKey = Sheetsy.urlToKey('https://docs.google.com/spreadsheets/d/1VXAyesupiwGDHI1akAm6YvqUTTh0_NLemjz7USR9nuw/edit?usp=sharing')

export default {
  name: 'card',
  props: ['entities'],
  data () {
    return {
      allIdentified: false,
      isFlipped: false,
      isCorrect: false,
      randomThree: [],
      currentImage: {}
    }
  },
  methods: {
    getRandomThreeSetImg: function () {
      const numberOfChoices = 4
      // Filter by entities that have not been identified
      let unidentified = _.filter(this.entities, entity => !entity.identified)
      console.log(unidentified.length)
      if (unidentified.length >= numberOfChoices) {
        this.randomThree = _.sampleSize(unidentified, numberOfChoices)
        this.currentImage = _.sample(this.randomThree)
      } else if (unidentified.length >= 1) {
        this.currentImage = _.sample(unidentified)
        this.randomThree = _.sampleSize(
          _.filter(this.entities, entity => entity !== this.currentImage),
          numberOfChoices - 1
        )
        this.randomThree.push(this.currentImage)
        this.randomThree = _.shuffle(this.randomThree)
      } else {
        console.log('COMPLETE')
      }
    },
    checkAnswer: function (name) {
      if (this.currentImage.name === name) {
        this.currentImage.identified = true
        this.isCorrect = true
      } else this.isCorrect = false
    },
    flipped: function (event) {
      this.isFlipped = !this.isFlipped
    }
  },
  computed: {
    // Set the source of the current image
    imageSource: function () {
      if (this.currentImage.name !== undefined) {
        let name = this.currentImage.name.split(' ').join('_')
        // var imgSrc = require('../assets/images/' + name + '.jpg')
        var imgSrc = './static/images/' + name + '.jpg'
      }
      return imgSrc
    }
  },
  mounted: function () {
    this.getRandomThreeSetImg()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  $card-width: 25vw;
  .card {
    width: $card-width;
    height: calc(8/5 * #{$card-width});
    min-width: 250px;
    min-height: 400px;
    perspective: $card-width * 10;

    button {
      cursor: pointer;
      background-color: #F2F2F2;
      border-style: solid;
      border-width: 2px;
      border-radius: 6px;
      border-color: #CCCCCC;
      font-size: 1em;

      &:hover {
        background-color: #CCCCCC;
      }
    }

    .card-front {
      width: 100%;
      height: 100%;
      background-color: #fff;

      .img-holder {
        width: 100%;
        height: 50%;

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

        .card-button {
          width: 75%;
          height: auto;
          min-height: 2.4em;
          margin-top: 0.85em;
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
      background-color: #fff;
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
