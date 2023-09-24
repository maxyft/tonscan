<template>
  <div class="touch-panel" :class="{ 'touch-panel--active': value }">
    <transition name="fade-in">
      <div
          v-if="value"
          class="touch-panel__overlay"
          v-on:click="handleOverlayClick"
      ></div>
    </transition>

    <transition
        v-on:enter="handleTransitionEnter"
        v-on:leave="handleTransitionLeave"
    >
      <div
          v-if="value"
          class="touch-panel__main-area"
          v-bind:class="{ 'touch-panel__main-area--with-transition': !touchStartY }"
          v-bind:style="mainAreaTransformStyle"
      >
        <div
            class="touch-panel__header"
            v-on:touchstart="handleTouchStart"
            v-on:touchmove="handleTouchMove"
            v-on:touchend="handleTouchEnd"
        ></div>
        <slot></slot>
      </div>
    </transition>
  </div>
</template>

<script>
export default {

  props: {
    value: Boolean
  },

  data () {
    return {
      touchStartY: null,
      previousTouchY: null,
      maxDistance: 100,
      animationOptions: {
        duration: 200, direction: "normal", iterations: 1
      }
    }
  },

  methods: {
    handleTouchStart (evt) {
      const targetTouch = evt.targetTouches[0]
      this.touchStartY = targetTouch.clientY
    },
    handleTouchMove (evt) {
      const targetTouch = evt.targetTouches[0]
      this.previousTouchY = targetTouch.clientY
    },
    handleTouchEnd () {
      if (this.touchDistance > this.maxDistance) {
        this.$emit('close')
      }

      this.touchStartY = null
      this.previousTouchY = null
    },
    handleOverlayClick () {
      this.$emit('close')
    },
    handleTransitionEnter (el, done) {
      const transformOptions = [
        { transform: 'translateY(100%)' },
        { transform: 'translateY(0)'}
      ]

      const animation = el.animate(transformOptions, this.animationOptions)
      animation.onfinish = done
    },
    handleTransitionLeave (el, done) {
      const currentTransform = el.style.transform
      const transformOptions = [
        { transform: currentTransform },
        { transform: 'translateY(100%)'}
      ]

      const animation = el.animate(transformOptions, this.animationOptions)
      animation.onfinish = done
    }
  },

  computed: {
    touchDistance () {
      return this.previousTouchY - this.touchStartY
    },
    mainAreaTransformStyle () {
      if (this.touchStartY !== null && this.touchDistance > 0) {
        return `transform: translateY(${this.touchDistance}px)`
      }

      return 'transform: translateY(0px)'
    }
  }

}
</script>

<style lang="scss">
  .touch-panel {
    pointer-events: none;
    position: fixed;
    display: flex;
    flex-direction: column;
    justify-content: end;
    align-items: end;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    &--active {
      z-index: 9999;
      pointer-events: auto;
    }
  }

  .touch-panel__overlay {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    z-index: 1;
    background: rgba(0,0,0, .2);
  }

  .touch-panel__main-area {
    height: 500px;
    width: 100%;
    z-index: 2;
    background: rgba(0,0,0, .3);
    &--with-transition {
      transition: transform .15s ease;
    }
  }

  .touch-panel__header {
    height: 50px;
    width: 100%;
    background: rgba(0,0,0, .5);
  }

  .fade-in-enter-active {
    animation: fade .25s;
  }

  .fade-in-leave-active {
    animation: fade .3s reverse;
  }

  @keyframes fade {
    0% {
      opacity: 0;
    }
    33% {
      opacity: 0.5;
    }
    100% {
      opacity: 1;
    }
  }
</style>