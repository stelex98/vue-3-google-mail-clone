<template>
  <div class="modal">
    <div @click.stop="emit('update:closeModal')">
      <div
        class="modal-card"
        style="border: 1px solid black;"
      >
        <slot />
      </div>
    </div>
  </div>
</template>

<script>
import useKeydown from '../composables/use-keydown'

export default {
emits: ['update:closeModal'],
  setup (props, { emit }) {
    useKeydown([
      { key: 'Escape', fn: () => { emit('update:closeModal') } }
    ])

    return {
      emit
    }
  }
}
</script>
