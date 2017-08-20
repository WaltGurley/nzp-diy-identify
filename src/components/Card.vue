<template>
  <div class="card">
    <div class="img-holder">
      <img  class="img-responsive" src="https://via.placeholder.com/150x150" alt="">
    </div>
    <button
      v-for="entity in randomThree"
      v-on:click="generateRandomThree"
      class=""
    >
      {{entity.name}}
    </button>
  </div>
</template>

<script>
import Sheetsy from 'sheetsy'
import _ from 'lodash'

const spreadsheetKey = Sheetsy.urlToKey('https://docs.google.com/spreadsheets/d/1VXAyesupiwGDHI1akAm6YvqUTTh0_NLemjz7USR9nuw/edit?usp=sharing')

export default {
  name: 'card',
  beforeCreate () {
    Sheetsy.getWorkbook(spreadsheetKey).then(
      workbook => {
        const sheetId = workbook.sheets[0].id
        Sheetsy.getSheet(spreadsheetKey, sheetId).then(
          sheet => {
            this.entities = sheet.rows
            this.randomThree = _.sampleSize(this.entities, 3)
            document.getElementsByClassName('loading-icon')[0]
              .style.display = 'none'
            console.log(sheet.rows)
          }
        )
      }
    )
  },
  data () {
    return {
      entities: [],
      randomThree: []
    }
  },
  methods: {
    generateRandomThree: function () {
      this.randomThree = _.sampleSize(this.entities, 3)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  .card {
    width: 10vw;
    height: 20vw;
    background-color: #ddd;

    .img-holder {
      width: 100%;
      height: 50%;
    }
  }
</style>
