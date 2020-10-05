<template>
  <div class="email-display">
    <div class="toolbar">
      <button @click.stop="toggleArchive">
        {{ email.archived ? 'Move to Inbox (e)' : 'Archive (e)' }}
      </button>
      <button @click.stop="toggleRead">
        {{ email.read ? 'Mark Unread (r)' : 'Mark Read (r)' }}
      </button>
      <button @click.stop="goNewer">
        Newer (k)
      </button>
      <button @click.stop="goOlder">
        Older (j)
      </button>
    </div>

    <h2 class="mb-0">
      Subject:
      <strong>{{ email.subject }}</strong>
    </h2>
    <div>
      <em>From {{ email.from }} on {{ format(new Date(email.sentAt), 'MMM do yyyy') }}</em>
    </div>
    <div v-html="marked(email.body)" />
  </div>
</template>

<script>
import { format } from 'date-fns'
import { reactive, toRefs } from 'vue'
import marked from 'marked'
import useKeydown from '../composables/use-keydown'

export default {
  props: {
    email: {
      type: Object,
      required: true
    }
  },
emits: ['update:changeEmail'],
  setup (props, { emit }) {
    const data = reactive({
      email: props.email
    })

    const toggleRead = () => {
      emit('update:changeEmail', { toggleRead: true, save: true })
    }
    const toggleArchive = () => {
      emit('update:changeEmail', { toggleArchive: true, save: true, closeModal: true })
    }
    const goNewer = () => emit('update:changeEmail', { changeIndex: -1 })
    const goOlder = () => emit('update:changeEmail', { changeIndex: 1 })
    const goNewerAndArchive = () => emit('update:changeEmail', { changeIndex: -1, toggleArchive: true })
    const goOlderAndArchive = () => emit('update:changeEmail', { changeIndex: 1, toggleArchive: true })

    useKeydown([
      { key: 'e', fn: toggleArchive },
      { key: 'r', fn: toggleRead },
      { key: 'k', fn: goNewer },
      { key: 'j', fn: goOlder },
      { key: '[', fn: goNewerAndArchive },
      { key: ']', fn: goOlderAndArchive }
    ])

    return {
      format,
      marked,
      toggleRead,
      toggleArchive,
      goNewer,
      goOlder,
      ...toRefs(data)
    }
  }
}
</script>
