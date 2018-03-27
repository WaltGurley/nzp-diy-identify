<template>
  <div id="app" v-on:click="resetOnInactivity">
    <transition name="slide-down">
      <nav v-if="imageInfoLoaded">
        <h1 class="app-title">Under the Umbrella</h1>
        <h3 class="group-name">Smithsonian's National Zoo and Conservation Biology Institute</h3>
        <icon v-on:click.native="showInfo = !showInfo" name="info-circle" class="info-button"></icon>
      </nav>
    </transition>
    <div class="main-container">
      <transition name="slide-left">
        <div v-if="imageInfoLoaded" class="app-containers card-container">
          <transition name="switch-card">
            <card
              v-if="cardSwitch"
              key="card-1"
              :cardData="dataForCurrentState"
              :correctImage="currentImage"
              :stateOfApp="appState"
              v-on:identified="setIdentified"
              v-on:choiceMade="changeNewCardButtonText"
            >
            </card>
            <card
              v-else
              key="card-2"
              :cardData="dataForCurrentState"
              :correctImage="currentImage"
              :stateOfApp="appState"
              v-on:identified="setIdentified"
              v-on:choiceMade="changeNewCardButtonText"
            >
            </card>
          </transition>
        </div>
      </transition>
      <transition name="slide-right">
        <div v-show="!buttonDisabled" class="app-containers score-container">
          <h2 class="score">{{identifiedCount}} / {{imageInfo.length}}</h2>
        </div>
      </transition>
      <transition name="slide-up">
        <div v-show="!buttonDisabled" class="app-containers main-controls">
          <button
            v-on:click="getNewCard"
            id="new-card-button"
            class="main-controls-buttons"
            v-bind:disabled="buttonDisabled"
          >{{newCardButtonText}}</button>
        </div>
      </transition>
    </div>
    <modal
      v-if="showInfo"
      :appInfo="pageInfo"
      v-on:closeModal="showInfo = false"
    >
    </modal>
    <transition name="fade-out">
      <div v-if="!imageInfoLoaded" class="loading-icon"></div>
    </transition>
  </div>
</template>

<script>
import Card from './components/Card'
import Modal from './components/Modal'
import filter from 'lodash/filter'
import sample from 'lodash/sample'
import sampleSize from 'lodash/sampleSize'
import shuffle from 'lodash/shuffle'
import uniqBy from 'lodash/uniqBy'
import 'vue-awesome/icons/info-circle'
import Icon from 'vue-awesome/components/Icon'

import imgData from '../static/imageInfo.csv'

export default {
  name: 'app',
  components: {
    Card,
    Modal,
    Icon
  },
  data () {
    return {
      imageInfo: [],
      dataForCurrentState: [],
      currentImage: {},
      newCardButtonText: 'Start Game',
      identifiedCount: 0,
      pageInfoLoaded: false,
      imageInfoLoaded: false,
      cardSwitch: true,
      showInfo: false,
      appState: 'start',
      buttonDisabled: true,
      startInactiveResetTimer: {}
    }
  },
  methods: {
    startGame: function () {
      this.dataForCurrentState = this.pageInfo
      this.dataForCurrentState.push(this.imageInfo.length)
    },
    startOver: function () {
      this.dataForCurrentState = this.pageInfo
      this.dataForCurrentState.push(this.imageInfo.length)
    },
    getRandomChoicesSetImg: function () {
      const numberOfChoices = 3
      // Filter by entities that have not been identified
      this.currentImage = sample(
        filter(this.imageInfo, entity => !entity.identified)
      )
      // const names = map(this.imageInfo, entity => entity.EntityName)
      const uniqueEntities = uniqBy(this.imageInfo, 'EntityName')
      // const uniqueNames = uniq(names)
      this.dataForCurrentState = sampleSize(
        filter(uniqueEntities, entity =>
          entity.EntityName !== this.currentImage.EntityName),
        numberOfChoices - 1
      )
      this.dataForCurrentState.push(this.currentImage)
      this.dataForCurrentState = shuffle(this.dataForCurrentState)
    },
    setIdentified: function () {
      this.currentImage.identified = true
      this.identifiedCount = filter(this.imageInfo, entity => entity.identified).length
    },
    getNewCard: function () {
      // If this is the start of the game change the app state to in progress
      // Else if this is the end of the game remove correct identified and
      // start over
      if (this.appState === 'start') {
        this.appState = 'in progress'
      } else if (this.appState === 'end') {
        this.resetCards()
      }

      // If all the images have been identified set the game state to 'end',
      // switch cards, and set the appropriate button text
      if (this.identifiedCount === this.imageInfo.length) {
        this.appState = 'end'
        this.cardSwitch = !this.cardSwitch
        this.newCardButtonText = 'Play Again'
        return
      }

      this.getRandomChoicesSetImg()
      this.cardSwitch = !this.cardSwitch
      this.newCardButtonText = 'Skip Card'
    },
    changeNewCardButtonText: function () {
      if (this.appState === 'in progress') {
        this.newCardButtonText = 'Next Card'
      } else if (this.appState === 'start') {
        this.buttonDisabled = false
      }
    },
    resetOnInactivity: function () {
      clearTimeout(this.startInactiveResetTimer)
      console.log('timer started!')
      this.startInactiveResetTimer = setTimeout(() => window.location.reload(), 120000)
    },
    resetCards: function () {
      // At end of game if user selects to play again set all cards to not
      // identified, reset the identified count and set application state
      // to 'in progress'
      this.imageInfo.forEach(function (row) {
        row.identified = false
      })
      this.identifiedCount = 0
      this.appState = 'start'
    }
  },
  beforeMount () {
    // Get the imagage information sheet and extract the data
    // This is locally loaded from '../static/imageInfo.csv'
    // Add 'identified' property to each row of data and set to false
    // as no image has been identified
    imgData.forEach(function (row) {
      row.identified = false
    })
    this.imageInfo = imgData
  },
  mounted () {
    // Remove loading spinner and add card
    this.imageInfoLoaded = true
  }
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Heebo');
@import "~include-media/dist/include-media";
$breakpoints: (small-phone: 320px, phone: 425px, tablet: 768px, desktop: 1024px);

#app {
  height: 100%;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: flex-start;
  flex-wrap: wrap;
  font-family: 'Heebo', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  background-image: url('./assets/RedTile.png');
  background-color: #24383A;

  .important-text {
    font-weight: bold;
    color: #4156A1;
  }

  .img-responsive {
    max-height: 100%;
    max-width: 100%;
  }

  button {
    cursor: pointer;
    background-color: #F2F2F2;
    border-style: solid;
    border-width: 2px;
    border-radius: 6px;
    border-color: #CCCCCC;
    transition: background-color 0.2s ease-in;

    &:hover {
      background-color: #CCCCCC;
    }
  }

  nav {
    width: 100%;
    height: 60px;
    @include media("<=small-phone") {
      height: 40px;
    }
    background-color: #333333;
    color: #F2F2F2;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
    padding-left: 0.85em;
    padding-right: 0.85em;
    box-shadow: 0px 2.5px 5px darken(#4156A1, 30%);

    .app-title {
      font-weight: normal;
      @include media("<=small-phone") {
        font-size: 1em;
      }
    }

    .group-name {
      font-weight: lighter;
      margin-left: auto;
      margin-right: 0.85em;
      @include media("<=phone") {
        display: none;
      }
    }

    .info-button {
      cursor: pointer;
      width: auto;
      height: 40%;
      padding: 2px;
      margin-left: 0.85em;
      @include media("<=tablet") {
        height: 34px;
      }
      @include media("<=phone") {
        margin-left: auto;
      }
      color: #F2F2F2;
      transition: color 0.2s ease-in;

      &:hover {
        color: lighten(#CC0000, 40%);
      }
    }
  }

  .main-container {
    width: 100%;
    // height: fill-available;

    @include media("<=phone", "portrait") {
      height: 80vh;
    }

    @include media("<=small-phone", "portrait") {
      height: calc(100vh - 60px);
    }

    .app-containers {
      width: 100%;
    }

    $card-width: 30vw;
    $card-height: 70vh;
    .card-container {
      position: relative;
      height: 8/5 * $card-width;

      @include media(">=desktop", "landscape") {
        height: $card-height;
      }
      min-height: 400px;
    }

    .score {
      display: flex;
      flex-wrap: nowrap;
      justify-content: center;
      text-align: center;
      color: #FFFFFF;

      @include media("<=small-phone") {
        margin-bottom: 0.1em;
        margin-top: 0.4em;
      }
    }

    .main-controls {
      display: flex;
      flex-wrap: nowrap;
      justify-content: center;

      .main-controls-buttons {
        min-height: 1.8em;
        font-size: 1.2em;
        padding-top: 4px;
        padding-right: 10px;
        padding-bottom: 5px;
        padding-left: 10px;
        color: #FFFFFF;
        background-color: darken(#4156A1, 15%);
        border-color: lighten(#4156A1, 10%);
        box-shadow: 2.5px 2.5px 5px darken(#4156A1, 30%);
        transition: all 0.2s ease-in;

        @include media("<=phone") {
          font-size: 1em;
        }

        &:hover {
          background-color: lighten(#4156A1, 10%);
        }

        &:disabled {
          cursor: default;
          color: #AAAAAA;
          background-color: lighten(#4156A1, 60%);
          box-shadow: none;
        }
      }
    }
  }

  .fade-out-leave-active {
    transition: opacity 0.2s ease;
  }

  .fade-out-leave-to {
    opacity: 0;
  }

  .slide-down-enter-active, .slide-up-enter-active,
  .slide-right-enter-active, .slide-left-enter-active,
  .slide-left-leave-active, .fade-out-leave-active {
    transition-property: all;
    transition-duration: 1.8s;
    transition-timing-function: ease-in-out;
  }

  .slide-down-enter {
    transform: translateY(-100%);
  }

  .slide-up-enter {
    transform: translateY(20vh);
  }

  .slide-right-enter {
    transform: translateX(-100%);
  }

  .slide-left-enter, .slide-left-leave-to {
    transform: translateX(calc(50vw + 50%));
  }

  .switch-card-enter-active, .switch-card-leave-active {
    transition: all 1s cubic-bezier(.75,-0.5,0,1.75);
  }

  .switch-card-enter {
    transform: translateX(calc(50vw + 50%)) translateY(-50%);
  }

  .switch-card-leave-to {
    transform: translateX(calc(-50vw - 105%)) translateY(-50%);
  }

  .loading-icon {
    $spinnerSize: 8vw;
    position: absolute;
    width: $spinnerSize;
    height: $spinnerSize;
    left: calc(50% - #{$spinnerSize} / 2);
    top: calc(50% - #{$spinnerSize} / 2);
    border-radius: 100%;
    border-bottom-color: #F2F2F2;
    border-left-color: #F2F2F2;
    border-bottom-style: solid;
    border-left-style: solid;
    border-width: 1px;
    box-shadow: 0px 2.5px 10px #F2F2F2;
    animation: 1s linear 0s infinite spin;
  }

  @keyframes spin {
    from {transform: rotate(0deg);} to
    {transform: rotate(360deg);}
  }
}
</style>
