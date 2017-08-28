<template>
  <div id="app">
    <transition name="slide-down">
      <nav v-if="pageInfoLoaded">
        <h2 class="app-title">{{pageInfo.appname}}</h2>
        <h3 class="group-name">Digital Library Initiatives</h3>
      </nav>
    </transition>
    <div class="main-container">
      <transition name="slide-left">
        <div v-if="imageInfoLoaded" class="app-containers card-container">
          <transition name="switch-card">
            <card
              v-if="cardSwitch"
              key="card-1"
              :entities="randomThree"
              :correctImage="currentImage"
              v-on:identified="setIdentified"
              v-on:choiceMade="changeNewCardButtonText"
            >
            </card>
            <card
              v-else
              key="card-2"
              :entities="randomThree"
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
          >{{newCardButtonText}}</button>
        </div>
      </transition>
    </div>
    <div v-if="!imageInfoLoaded" class="loading-icon"></div>
  </div>
</template>

<script>
import Card from './components/Card'
import Sheetsy from 'sheetsy'
import _ from 'lodash'

const spreadsheetKey = Sheetsy.urlToKey('https://docs.google.com/spreadsheets/d/1Ew_tsLL-TxHdKuFHkeq807o9gEOekfPvKjFiecznDqc/edit#gid=0')

export default {
  name: 'app',
  components: {
    Card
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
      cardSwitch: true
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
      const uniqueNames = _.uniqBy(this.imageInfo, 'entityname')
      this.randomThree = _.sampleSize(
        _.filter(uniqueNames, entity => entity !== this.currentImage),
        numberOfChoices - 1
      )
      this.randomThree.push(this.currentImage)
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
    // Get the Google Sheets workbook and get the first sheet ID on initial load
    Sheetsy.getWorkbook(spreadsheetKey).then(
      workbook => {
        const imageInfoSheetId = workbook.sheets[0].id
        const appInfoSheetId = workbook.sheets[1].id
        // Get the first sheet and extract the data
        Sheetsy.getSheet(spreadsheetKey, appInfoSheetId).then(
          appInfoSheet => {
            this.pageInfo = appInfoSheet.rows[0]
            this.pageInfoLoaded = true

            Sheetsy.getSheet(spreadsheetKey, imageInfoSheetId).then(
              imageInfoSheet => {
                // Add 'identified' property to each row of data and set to false
                // as no image has been identified
                imageInfoSheet.rows.forEach(function (row) {
                  row.identified = false
                })
                this.imageInfo = imageInfoSheet.rows

                console.log(this.imageInfo)
                // Remove loading spinner and add card
                this.imageInfoLoaded = true
                this.getRandomThreeSetImg()
              }
            )
          }
        )
      }
    )
  }
}
</script>

<style lang="scss">
@import "~include-media/dist/include-media";
$breakpoints: (small-phone: 320px, tablet: 768px, desktop: 1024px);

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
  background-color: #FFFFFF;

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
    font-size: 1em;
    transition: background-color 0.2s ease-in;

    &:hover {
      background-color: #CCCCCC;
    }
  }

  nav {
    width: 100%;
    height: 60px;
    background-color: #CC0000;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
    padding-left: 0.85em;
    padding-right: 0.85em;

    .app-title {
      font-weight: normal;
    }

    .group-name {
      font-weight: normal;
      margin-left: auto;
    }
  }

  .main-container {
    width: 100%;

    .app-containers {
      width: 100%;
      display: flex;
      flex-wrap: nowrap;
      justify-content: center;
    }

    $card-width: 30vw;
    $card-height: 70vh;
    .card-container {
      height: 8/5 * $card-width;

      @include media(">=desktop", "landscape") {
        height: $card-height;
      }
      min-height: 400px;
    }

    .score {
      text-align: center;

      @include media("<=small-phone") {
        margin-bottom: 0.1em;
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

  $card-width: 30vw;
  .slide-left-enter, .slide-left-leave-to {
    transform: translateX(50vw + ($card-width / 2));
  }

  .switch-card-enter-active, .switch-card-leave-active {
    transition: all 0.6s cubic-bezier(.75,-0.5,0,1.75);
  }

  $card-width: 30vw;
  .switch-card-enter, .switch-card-leave-to {
    transform: translateX(50vw + ($card-width / 2));
    opacity: 0;
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
