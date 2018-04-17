<template lang="html">
  <transition name="modal">
    <div class="modal">
        <div v-on:click="$emit('closeModal')" class="modal-background">
          <div class="modal-holder">
            <h4>Under the Umbrella</h4>
            <p class="modal-paragraph">This game was built using a web template developed at NCSU Libraries. Photographs are supplied from camera traps operated by eMammal (emammal.si.edu) and depict animals that live in the same habitat and range as the giant panda. For information on the Smithsonian Institution’s Privacy Statement and Terms of Use, please visit www.si.edu/privacy and www.si.edu/termsofuse.</p>
            <button v-on:click="$emit('closeModal')" class="modal-button">Close</button>
          </div>
        </div>
    </div>
</transition>
</template>

<script>
export default {
  name: 'modal',
  data () {
    return {

    }
  }
}
</script>

<style lang="scss" scoped>
@import "~include-media/dist/include-media";
$breakpoints: (small-phone: 320px, phone: 425px, tablet: 768px, desktop: 1024px);
$card-width: 30vw;
$card-height: 70vh;

.modal {
  transition-duration: 0.3s;
  transition-property: opacity;
}
  .modal-background {
    position: absolute;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: rgba(0, 0, 0, .5);

    .modal-holder {
      width: $card-width * 2;
      height: auto;
      // max-height: 80vh;
      @include media("<=phone") {
        width: calc(5/8 * #{$card-height} * 1.1);
        height: $card-height;
        max-height: 75vmax;
      }
      @include media(">=desktop", "landscape") {
        width: calc(5/8 * #{$card-height} * 2);
        // height: $card-height;
      }
      min-width: 250px;
      min-height: 400px;
      max-width: 500px;
      max-height: 800px;

      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      overflow-y: scroll;

      background-color: #FFFFFF;
      border-style: solid;
      border-width: 1px;
      border-radius: 6px;
      border-color: darken(#FFFFFF, 10%);
      box-shadow: 10px 10px 20px darken(#4156A1, 40%);
      transition-property: transform;
      transition-duration: 0.8s;

      .modal-paragraph {
        margin-left: 1.4em;
        margin-right: 1.4em;
      }

      .modal-button {
        min-height: 1.8em;
        font-size: 1.2em;
        padding-top: 4px;
        padding-right: 10px;
        padding-bottom: 5px;
        padding-left: 10px;
        margin-top: 0.85em;
        margin-bottom: 0.85em;
      }
    }
  }

  .modal-enter, .modal-leave-to {
    opacity: 0;
  }

  .modal-enter .modal-holder,
  .modal-leave-active .modal-holder, {
    transform: translateY(-100vh);
  }
</style>
