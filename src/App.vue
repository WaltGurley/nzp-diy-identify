<template>
  <div id="app">
    <transition name="slide-down">
      <nav v-if="pageInfoLoaded">
        <img class="logo-img" src="https://via.placeholder.com/150x150" alt="">
        <h2 class="app-title">{{pageInfo.appname}}</h2>
        <h3 class="group-name">Digital Library Initiatives</h3>
      </nav>
    </transition>
    <transition name="slide-up">
      <card v-if="imageInfoLoaded" :entities="imageInfo" class="slide-delay"></card>
    </transition>
    <div v-if="!imageInfoLoaded" class="loading-icon"></div>
  </div>
</template>

<script>
import Card from './components/Card'
import Sheetsy from 'sheetsy'

const spreadsheetKey = Sheetsy.urlToKey('https://docs.google.com/spreadsheets/d/1Ew_tsLL-TxHdKuFHkeq807o9gEOekfPvKjFiecznDqc/edit#gid=0')

export default {
  name: 'app',
  components: {
    Card
  },
  data () {
    return {
      imageInfo: [],
      pageInfo: [],
      pageInfoLoaded: false,
      imageInfoLoaded: false
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
            console.log(this.pageInfo)
            this.pageInfoLoaded = true

            Sheetsy.getSheet(spreadsheetKey, imageInfoSheetId).then(
              imageInfoSheet => {
                // Add 'identified' property to each row of data and set to false
                // as no image has been identified
                imageInfoSheet.rows.forEach(function (row) {
                  row.identified = false
                })
                this.imageInfo = imageInfoSheet.rows

                // Remove loading spinner and add card
                this.imageInfoLoaded = true
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

  nav {
    width: 100%;
    height: 8%;
    min-height: 4em;
    background-color: #CC0000;
    color: white;
    display: flex;
    align-items: center;
    justify-content: flex-start;
    flex-wrap: wrap;
    padding-left: 0.85em;
    padding-right: 0.85em;

    .logo-img {
      height: 3em;
    }

    .app-title {
      font-weight: normal;
      margin-left: 0.85em;
    }

    .group-name {
      font-weight: normal;
      margin-left: auto;
    }
  }

  .slide-down-enter-active, .slide-up-enter-active, .fade-leave-active {
    transition-property: all;
    transition-timing-function: ease-in-out;
    transition-duration: 0.6s;
  }

  .slide-delay {
    transition-delay: 0.2s;
  }

  .slide-down-enter {
    transform: translateY(-100%);
  }

  .slide-up-enter {
    transform: translateY(calc(50vh + 50%));
  }

  .fade-leave {
    transform: translateY(100vh);
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
