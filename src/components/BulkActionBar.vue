<template>
  <div class="bulk-action-bar">
    <span class="checkbox">
      <input type="checkbox"
             :checked="allEmailsSelected"
             :class="[someEmailsSelected ? 'partial-check' : '']"
             @click="bulkSelect"
      >
      </span>
    <span class="buttons">
      <button @click="emailSelection.markRead()"
              :disabled="[...emailSelection.emails].every(e => e.read)"
      >
        Mark Read
      </button>
      <button @click="emailSelection.markUnread()"
              :disabled="[...emailSelection.emails].every(e => !e.read)"
      >
        Mark Unread
      </button>
      <button @click="emailSelection.markArchive()"
              :disabled="numberSelected === 0"
      >
        Archive
      </button>
    </span>
  </div>
</template>

<script>
import useEmailSelection from '../composables/use-email-selection';
import { watch, computed } from 'vue';

export default {
  props: {
    emails: {
      type: Array,
      required: true
    }
  },
  async setup(props) {
    let emailSelection = useEmailSelection();
    let numberSelected = computed(() => emailSelection.emails.size);
    let numberEmails = null;

    watch(() => {
      const { emails } = props;
      numberEmails = emails.length;
    })
    let bulkSelect = function() {
      if (allEmailsSelected.value) {
        emailSelection.clear();
      } else {
        emailSelection.addMultiple(props.emails);
      }
    }
    let allEmailsSelected = computed(() => numberSelected.value === numberEmails);
    let someEmailsSelected = computed(() => numberSelected.value > 0 && numberSelected.value < numberEmails);

    return {
      emailSelection,
      allEmailsSelected,
      someEmailsSelected,
      bulkSelect,
      numberSelected
    }
  }
}
</script>
