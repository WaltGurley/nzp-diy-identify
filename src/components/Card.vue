<template>
  <div v-if="stateOfApp === 'start'" class="card">
    <div class="info-card card-front will-flip" v-bind:class="{ flippedToFront: isFlipped }">
      <h1 class="card-header start-card-header">Under the Umbrella</h1>
      <p class="card-paragraph start-card-paragraph">
        Test your ability to identify <span class="important-text">Animals in China</span>!
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
        <aside class="img-zoom-callout top-left-border-curved">
          <i class="fas fa-search-plus img-zoom-icon"></i>
          <div class="img-zoom-text">
            <h5 class="line">tap image to zoom</h5>
          </div>
        </aside>
        <img
          v-bind:src="imageSource.url"
          v-bind:alt="imageSource.altText"
          class="img-responsive img-zoomable top-border-radius"
          data-action="zoom"
        >
      </div>
      <div class="question-choices">
        <h2 class="card-header question">Can you identify this animal?</h2>
        <div class="buttons">
          <button
            v-for="entity in cardData"
            v-on:click="flipped(entity)"
            class="card-button"
          >
            {{entity.EntityName}}
          </button>
        </div>
      </div>
    </div>

    <div class="card-back will-flip" v-bind:class="{ flippedToBack: !isFlipped }">
        <div class="img-holder">
          <aside class="img-zoom-callout top-left-border-curved">
            <i class="fas fa-search-plus img-zoom-icon"></i>
            <div class="img-zoom-text">
              <h5 class="line">tap image to zoom</h5>
            </div>
          </aside>
          <img
            v-bind:src="imageSource.url"
            v-bind:alt="imageSource.altText"
            class="img-responsive img-zoomable top-border-radius"
            data-action="zoom"
          >
        </div>

        <div v-show="isCorrect" class="correct-answer">
          <h2 class="card-header">You are correct!</h2>
          <h2 class="card-header">{{correctImage.EntityName}}</h2>
          <p class="card-paragraph">{{correctImage.EntityDescription}}</p>
        </div>
        <div v-show="!isCorrect">
          <div class="incorrect-answer">
            <h2 class="card-header">Sorry, that's incorrect.</h2>
            <p class="card-paragraph">This is a {{correctImage.EntityName}}. Below is an example of a {{userChoice.EntityName}}</p>
          </div>
          <div class="img-holder">
            <aside class="img-zoom-callout">
              <i class="fas fa-search-plus img-zoom-icon"></i>
              <div class="img-zoom-text">
                <h5 class="line">tap image to zoom</h5>
              </div>
            </aside>
            <img
              v-bind:src="wrongImageSource.url"
              v-bind:alt="wrongImageSource.altText"
              class="img-responsive img-zoomable bottom-border-radius"
              alt="Can you tell what is in this image?"
              data-action="zoom"
            >
          </div>
        </div>
    </div>
  </div>

  <div v-else-if="stateOfApp === 'end'" class="card">
    <div class="info-card card-front will-flip">
      <h2 class="card-header start-card-header">Congratulations!</h2>
      <h2 class="card-paragraph start-card-paragraph">You have identified all the images!</h2>
      <p class="card-paragraph start-card-paragraph">
        Select <span class="important-text">Play Again</span> below to go through the images again.
      </p>
    </div>
  </div>
</template>

<script>
// import Zooming for image zoom
import Zooming from 'zooming'

const zooming = new Zooming({
  bgOpacity: 0,
  // Add border radius to bottom of image when zoomed
  onBeforeOpen: () => {
    document.querySelectorAll('.img-zoomable').forEach(img =>
      img.classList.add('full-border-radius')
    )
    document.querySelectorAll('#new-card-button').forEach(d =>
      d.setAttribute('disabled', true)
    )
  },
  // Remove border radius from bottom of image when back to card
  onClose: () => {
    document.querySelectorAll('.img-zoomable').forEach(img =>
      img.classList.remove('full-border-radius')
    )
    document.querySelectorAll('#new-card-button').forEach(d =>
      d.removeAttribute('disabled')
    )
  }
})

export default {
  name: 'card',
  props: ['cardData', 'correctImage', 'stateOfApp'],
  data () {
    return {
      isFlipped: false,
      isCorrect: false,
      wrongImageSource: '',
      userChoice: ''
    }
  },
  methods: {
    // Flip the card
    flipped: function (userChoice) {
      this.isFlipped = !this.isFlipped
      this.$emit('choiceMade')
      this.checkAnswer(userChoice)
    },
    // Check if the choice matches the image
    checkAnswer: function (userChoice) {
      this.userChoice = userChoice
      if (this.correctImage.EntityName === userChoice.EntityName) {
        this.correctImage.identified = true
        this.isCorrect = true
        this.$emit('identified')
      } else {
        this.isCorrect = false
        this.wrongImageSource = {
          'url': userChoice.url,
          'altText': userChoice.AltText
        }
      }
    }
  },
  computed: {
    // Set the source of the current image
    imageSource: function () {
      return {
        'url': this.correctImage.url,
        'altText': this.correctImage.AltText
      }
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
      justify-content: flex-start !important;
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

    .img-holder {
      width: 100%;
      max-height: 50%;
      position: relative;

      .img-zoom-callout {
        position: absolute;
        display: flex;
        align-items: flex-start;
        pointer-events: none;
        top: 0;
        left: 50%;
        transform: translateX(-50%);
        padding: 0.4em;
        // border-top-right-radius: 0.5rem;
        border-bottom-right-radius: 1rem;
        border-bottom-left-radius: 1rem;
        color: white;
        background-color: rgba(0, 0, 0, 0.6);

        .img-zoom-icon {
          font-size: 1.4rem;
          padding-right: 0.2em;
        }

        .img-zoom-text {
          display: flex;
          flex-direction: column;
          font-size: 1.2rem;
          margin: 0;
          line-height: 1.2rem;

          .line {
            margin: 0;
          }
        }
      }

      .full-border-radius {
        border-radius: 1rem;
      }

      .bottom-border-radius {
        border-bottom-right-radius: 1rem;
        border-bottom-left-radius: 1rem;
      }

      .top-border-radius {
        border-top-right-radius: 1rem;
        border-top-left-radius: 1rem;
      }

      img {
        cursor: zoom-in;
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
      background-color: white;

      .correct-answer, .incorrect-answer {
        overflow-y: scroll;
      }
    }

    .will-flip {
      border-style: solid;
      border-width: 1px;
      border-radius: 1em;
      border-color: darken(#FFFFFF, 10%);
      box-shadow: 10px 20px 40px #24383A;
      backface-visibility: hidden;
      transition-property: all;
      transition-duration: 1.2s;
      transtition-timing-function: ease-in-out;
      transition-timing-function: cubic-bezier(0.25, -0.5, 0.25, 1.5);
    }

    .flippedToFront {
      transform: rotateY(-180deg);
      box-shadow: -10px 10px 20px #24383A;
    }

    .flippedToBack {
      transform: rotateY(180deg);
      box-shadow: 10px 10px 20px #24383A;
    }
  }
</style>
