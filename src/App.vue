<template>
  <div id="app">
    <nav>
      <img class="logo-img" src="https://via.placeholder.com/150x150" alt="">
      <h3 class="app-title">Image Identification Application</h3>
      <h3 class="group-name">Digital Library Initiatives</h3>
    </nav>
    <card></card>
      <div class="loading-icon"></div>
  </div>
</template>

<script>
import Card from './components/Card'

export default {
  name: 'app',
  components: {
    Card
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
  background-color: #F2F2F2;

  .img-responsive {
    max-height: 100%;
    max-width: 100%;
  }

  nav {
    width: 100%;
    height: 4em;
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

  .loading-icon {
    $spinnerSize: 5vw;
    position: absolute;
    width: $spinnerSize;
    height: $spinnerSize;
    left: calc(50% - #{$spinnerSize} / 2);
    top: calc(50% - #{$spinnerSize} / 2);
    border-radius: 100px;
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
