<template>
  <div id="app">
    <transition name="slide-down">
      <nav v-if="pageInfoLoaded">
        <h2 class="app-title">{{pageInfo.appname}}</h2>
        <h3 class="group-name">Digital Library Initiatives</h3>
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
              :entityNames="randomThree"
              :correctImage="currentImage"
              v-on:identified="setIdentified"
              v-on:choiceMade="changeNewCardButtonText"
            >
            </card>
            <card
              v-else
              key="card-2"
              :entityNames="randomThree"
              :correctImage="currentImage"
              v-on:identified="setIdentified"
              v-on:choiceMade="changeNewCardButtonText"
            >
            </card>
          </transition>
        </div>
      </transition>
      <transition name="slide-right">
        <div v-if="imageInfoLoaded" class="app-containers score-container">
          <h2 class="score">{{identifiedCount}} / {{imageInfo.length}}</h2>
        </div>
      </transition>
      <transition name="slide-up">
        <div v-if="imageInfoLoaded" class="app-containers main-controls">
          <button
            v-on:click="getRandomThreeSetImg(); newCard()"
            id="new-card-button"
            class="main-controls-buttons"
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
    <div v-if="!imageInfoLoaded" class="loading-icon"></div>
  </div>
</template>

<script>
import Card from './components/Card'
import Modal from './components/Modal'
import Sheetsy from 'sheetsy'
import _ from 'lodash'
import 'vue-awesome/icons/info-circle'
import Icon from 'vue-awesome/components/Icon'

export default {
  name: 'app',
  components: {
    Card,
    Modal,
    Icon
  },
  data () {
    return {
      pageInfo: [],
      imageInfo: [],
      randomThree: [],
      currentImage: {},
      newCardButtonText: 'Skip Card',
      identifiedCount: 0,
      pageInfoLoaded: false,
      imageInfoLoaded: false,
      allIdentified: false,
      cardSwitch: true,
      showInfo: false
    }
  },
  methods: {
    getRandomThreeSetImg: function () {
      // Return if all images have been identified, GAME OVER
      if (_.every(this.imageInfo, ['identified', true])) {
        this.allIdentified = true
        return
      }
      const numberOfChoices = 3
      // Filter by entities that have not been identified
      this.currentImage = _.sample(
        _.filter(this.imageInfo, entity => !entity.identified)
      )
      const names = _.map(this.imageInfo, entity => entity.entityname)
      const uniqueNames = _.uniq(names)
      this.randomThree = _.sampleSize(
        _.filter(uniqueNames, entity =>
          entity !== this.currentImage.entityname),
        numberOfChoices - 1
      )
      this.randomThree.push(this.currentImage.entityname)
    },
    setIdentified: function () {
      this.currentImage.identified = true
      this.identifiedCount = _.filter(this.imageInfo, entity => entity.identified).length
    },
    newCard: function () {
      this.cardSwitch = !this.cardSwitch
      this.newCardButtonText = 'Skip Card'
    },
    changeNewCardButtonText: function () {
      this.newCardButtonText = 'Next Card'
    }
  },
  beforeMount () {
    // Start by fetching the Google Sheets URL from the local config file
    fetch('./static/config.json').then(response => {
      // Return the response JSON
      return response.json()
    }).then(config => {
      // Convert the URL to a Key
      return Sheetsy.urlToKey(config.GoogleSheetsURL)
    }).then(key => {
      // Get the Google Sheets workbook, get the sheet IDs, and return an
      // array of the key, entity info sheet ID, and app info sheet ID
      return Sheetsy.getWorkbook(key).then(
        workbook => {
          const entitySheetIDIndex = _.findIndex(workbook.sheets, sheet =>
              sheet.name === 'Image Info'
            )
          const appSheetIDIndex = _.findIndex(workbook.sheets, sheet =>
              sheet.name === 'App Info'
            )
          const keyAndIDs = {
            spreadsheetKey: key,
            entitySheet: workbook.sheets[entitySheetIDIndex].id,
            appSheet: workbook.sheets[appSheetIDIndex].id
          }
          return keyAndIDs
        })
    }).then(keyAndIDs => {
      // Get the App Info sheet and extract the data
      Sheetsy.getSheet(keyAndIDs.spreadsheetKey, keyAndIDs.appSheet)
        .then(appInfoSheet => {
          this.pageInfo = appInfoSheet.rows[0]
          this.pageInfoLoaded = true
        })

      // Get the Entity Info sheet and extract the data
      Sheetsy.getSheet(keyAndIDs.spreadsheetKey, keyAndIDs.entitySheet)
        .then(imageInfoSheet => {
          // Add 'identified' property to each row of data and set to false
          // as no image has been identified
          imageInfoSheet.rows.forEach(function (row) {
            row.identified = false
          })
          this.imageInfo = imageInfoSheet.rows

          // Remove loading spinner and add card
          this.imageInfoLoaded = true
          this.getRandomThreeSetImg()
        })
    })
  }
}
</script>

<style lang="scss">
@import "~include-media/dist/include-media";
$breakpoints: (small-phone: 320px, phone: 425px, tablet: 768px, desktop: 1024px);

#app {
  height: 100%;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: flex-start;
  flex-wrap: wrap;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  background-color: #4156A1;

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
      font-weight: normal;
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
        font-size: 1.2em;
        padding-top: 4px;
        padding-right: 10px;
        padding-bottom: 5px;
        padding-left: 10px;
        color: #FFFFFF;
        background-color: darken(#4156A1, 15%);
        border-color: lighten(#4156A1, 10%);
        box-shadow: 2.5px 2.5px 5px darken(#4156A1, 30%);
        transition: background-color 0.2s ease-in;

        @include media("<=phone") {
          font-size: 1em;
        }

        &:hover {
          background-color: lighten(#4156A1, 10%);
        }
      }
    }
  }

  .slide-down-enter-active, .slide-up-enter-active, .slide-right-enter-active, .slide-left-enter-active, .slide-left-leave-active {
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
    transition: all 0.8s cubic-bezier(.75,-0.5,0,1.75);
  }

  .switch-card-enter {
    transform: translateX(calc(50vw + 50%));
  }

  .switch-card-leave-to {
    transform: translateX(calc(-50vw - 100%));
  }

  .loading-icon {
    $spinnerSize: 5vw;
    position: absolute;
    width: $spinnerSize;
    height: $spinnerSize;
    left: calc(50% - #{$spinnerSize} / 2);
    top: calc(50% - #{$spinnerSize} / 2);
    border-radius: 50%;
    border-bottom-color: #CC0000;
    border-left-color: #000;
    border-bottom-style: dashed;
    border-left-style: dashed;
    animation: 1s linear 0s infinite spin;
  }

  @keyframes spin {
    from {transform: rotate(0deg);} to
    {transform: rotate(360deg);}
  }
}
</style>
