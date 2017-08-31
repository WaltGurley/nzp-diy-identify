<template>
  <div v-if="stateOfApp === 'start'" class="card">
    <div class="info-card card-front will-flip" v-bind:class="{ flippedToFront: isFlipped }">
      <h2 class="card-header start-card-header">{{cardData[0]}}</h2>
      <p class="card-paragraph start-card-paragraph">
        Test your ability to identify <span class="important-text">{{cardData[2]}}</span>!
      </p>
      <ol class="card-list card-paragraph start-card-paragraph">
        <li class="card-list-item">You will be presented with an image and three choices.</li>
        <li class="card-list-item">Select the choice that identifies what is in the image.</li>
      </ol>
      <button
        v-on:click="flipped('')"
        class="card-button start-card-button"
      >
        Next
      </button>
    </div>
    <div class="info-card card-back will-flip" v-bind:class="{ flippedToBack: !isFlipped }">
      <h2 class="card-header start-card-header">{{cardData[0]}}</h2>
      <p class="card-paragraph start-card-paragraph">
        Keep track of your progress with the score below. It shows you how many images you have correctly identified out of the total. Don't worry if you get one wrong. Any incorrect cards will be put back into the  deck for you to try again.
      </p>
      <p class="card-paragraph start-card-paragraph">
        Select <span class="important-text">Start Game</span> below to begin!
      </p>
    </div>
  </div>

  <div v-else-if="stateOfApp === 'in progress'" class="card">
    <div class="card-front will-flip" v-bind:class="{ flippedToFront: isFlipped }">
      <div class="img-holder">
        <img
          v-bind:src="imageSource"
          class="img-responsive img-zoomable"
          alt="Can you tell what is in this image?"
          data-action="zoom"
        >
      </div>
      <div class="question-choices">
        <h2 class="card-header question">What is this?</h2>
        <div class="buttons">
          <button
            v-for="entityName in cardData"
            v-on:click="flipped(entityName)"
            class="card-button"
          >
            {{entityName}}
          </button>
        </div>
      </div>
    </div>
    <div class="card-back will-flip" v-bind:class="{ flippedToBack: !isFlipped }">
      <div class="img-holder">
        <img
          v-bind:src="imageSource"
          class="img-responsive img-zoomable"
          alt="Can you tell what is in this image?"
          data-action="zoom"
        >
      </div>
      <div v-show="isCorrect" class="correct-answer">
        <h2 class="card-header">You are correct!</h2>
        <h2 class="card-header">{{correctImage.entityname}}</h2>
        <p class="card-paragraph">{{correctImage.entitydescription}}</p>
      </div>
      <div v-show="!isCorrect" class="incorrect-answer">
        <h2 class="card-header">Sorry, that's incorrect.</h2>
      </div>
    </div>
  </div>

  <div v-else-if="stateOfApp === 'end'" class="card">
    <div class="info-card card-front will-flip" v-bind:class="{ flippedToFront: isFlipped }">
      <h2 class="card-header start-card-header">Congratulations!</h2>
      <h2 class="card-paragraph start-card-paragraph">You have identified all the images!</h2>
      <p class="card-paragraph start-card-paragraph">
        Select <span class="important-text">Play Again</span> below to go through the images again.
      </p>
    </div>
  </div>
</template>

<script>
// import Sheetsy from 'sheetsy'
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
  props: ['cardData', 'correctImage', 'stateOfApp'],
  data () {
    return {
      isFlipped: false,
      isCorrect: false
    }
  },
  methods: {
    // Flip the card
    flipped: function (userChoice) {
      this.isFlipped = !this.isFlipped
      this.$emit('choiceMade')
      this.checkAnswer(userChoice)
      console.log(this.stateOfApp)
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
  $breakpoints: (small-phone: 320px, phone: 425px, tablet: 768px, desktop: 1024px);
  $card-width: 30vw;
  $card-height: 60vh;

  .card {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translateX(-50%) translateY(-50%);
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
      font-weight: bold;
      margin-top: 0.2em;
      margin-bottom: 0.2em;
      text-align: center;
      font-size: 1.4em;
      margin-left: 0.85em;
      margin-right: 0.85em;
    }

    .card-paragraph {
      margin-left: 0.85em;
      margin-right: 0.85em;
    }

    .card-list {
      padding-left: 0.85em;

      .card-list-item {
        margin-top: 0.85em;
      }
    }

    .info-card {
      justify-content: flex-start;
      align-items: flex-start;

      .start-card-header {
        margin-top: 0.85em;
        margin-bottom: 0;
      }

      .start-card-paragraph {
        @include media(">=desktop") {
          font-size: 1.1em;
        }
        margin-top: 0.85em;
        margin-bottom: 0;
        align-self: flex-start;
      }

      .start-card-button {
        min-height: 1.8em;
        font-size: 1.2em;
        padding-top: 4px;
        padding-right: 10px;
        padding-bottom: 5px;
        padding-left: 10px;
        margin-top: 0.85em;
        margin-bottom: 0.85em;
          @include media(">=tablet") {
            margin-top: auto;
            margin-bottom: 1.7em;
          }
      }
    }

    .card-front {
      width: 100%;
      height: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: space-between;
      background-color: #FFFFFF;

      .img-holder {
        width: 100%;
        max-height: 50%;

        img {
          cursor: zoom-in;
          border-top-right-radius: 6px;
          border-top-left-radius: 6px;
        }
      }

      .question-choices {
        width: 100%;
        height: 50%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: space-around;
        @include media(">=desktop") {
          justify-content: center;
        }
        margin-bottom: 20px;

        .buttons {
          width: 100%;
          height: 80%;
          @include media(">=desktop") {
            height: 70%;
          }
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: flex-start;

          .card-button {
            height: 30%;
            width: 75%;
            font-size: 0.9em;
            min-height: 2.4em;
            margin-top: 3%;
            @include media(">=desktop") {
              width: 70%;
              font-size: 1.2em;
            }
            @include media(">=desktop", "height<=760px") {
              width: 70%;
              font-size: 0.9em;
            }
          }
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
      background-color: #FFFFFF;

      .correct-answer, .incorrect-answer {
        overflow-y: scroll;
      }
    }

    .will-flip {
      border-style: solid;
      border-width: 1px;
      border-radius: 6px;
      border-color: darken(#FFFFFF, 10%);
      box-shadow: 10px 10px 20px darken(#4156A1, 30%);
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
