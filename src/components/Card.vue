<template>
  <div class="card">
    <div class="card-front will-flip" v-bind:class="{ flippedToFront: isFlipped }">
      <div class="img-holder">
        <img
          v-bind:src="imageSource"
          class="img-responsive img-zoomable"
          alt="Can you tell what is in this image?"
          data-action="zoom"
        >
      </div>
      <h4 class="card-header question">What is this?</h4>
      <div class="buttons">
        <button
          v-for="entity in entities"
          v-on:click="flipped(entity.entityname)"
          class="card-button button-front"
        >
          {{entity.entityname}}
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
        <h2 class="card-header">{{correctImage.entityname}}</h2>
        <h2 class="card-header">{{correctImage.entitydescription}}</h2>
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
import Zooming from 'zooming'

const zooming = new Zooming({
  bgOpacity: 0,
  // Add border radius to bottom of image when zoomed
  onBeforeOpen: () => {
    document.querySelector('.img-zoomable').setAttribute(
      'style', 'border-radius: 6px;'
    )
    document.querySelector('#new-card-button').setAttribute('disabled', true)
  },
  // Remove border radius from bottom of image when back to card
  onClose: () => {
    document.querySelector('.img-zoomable').setAttribute(
      'style', 'border-bottom-right-radius: 0; border-bottom-left-radius: 0'
    )
    document.querySelector('#new-card-button').removeAttribute('disabled')
  }
})

export default {
  name: 'card',
  props: ['entities', 'correctImage'],
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
    setCurrentImage: function () {
      this.currentImage = _.find(this.entities, entity => !entity.identified)
    },
    // Flip the card
    flipped: function (userChoice) {
      this.isFlipped = !this.isFlipped
      this.$emit('choiceMade')
      this.checkAnswer(userChoice)
    },
    // Check if the choice matches the image
    checkAnswer: function (name) {
      if (this.correctImage.entityname === name) {
        this.correctImage.identified = true
        this.isCorrect = true
        this.$emit('identified')
      } else this.isCorrect = false
    }
  },
  computed: {
    // Set the source of the current image
    imageSource: function () {
      return './static/images/' + this.correctImage.imagename
    }
  },
  mounted: function () {
    zooming.listen('.img-zoomable')
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  @import "~include-media/dist/include-media";
  $breakpoints: (small-phone: 320px, tablet: 768px, desktop: 1024px);
  $card-width: 30vw;
  $card-height: 70vh;
  .card {
    position: absolute;
    width: $card-width;
    height: calc(8/5 * #{$card-width});

    @include media(">=desktop", "landscape") {
      width: calc(5/8 * #{$card-height});
      height: $card-height;
    }
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
    }

    .will-flip {
      border-style: solid;
      border-width: 1px;
      border-radius: 6px;
      border-color: #CCCCCC;
      box-shadow: 10px 10px 20px #CCCCCC;
      backface-visibility: hidden;
      transition-property: all;
      transition-duration: 1.2s;
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
