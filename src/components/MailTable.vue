<template>
  <button @click="selectScreen('inbox')"
          :disabled="selectedScreen === 'inbox'"
  >
    Inbox
  </button>
  <button @click="selectScreen('archive')"
          :disabled="selectedScreen === 'archive'">
    Archived
  </button>
  <BulkActionBar :emails="filteredEmails" />
  <table class="mail-table">
    <tbody>
      <tr
        v-for="email in filteredEmails"
        :key="email.id"
        :class="['clickable', email.read ? 'read' : '']"
      >
        <td>
          <input
            type="checkbox"
            @click="emailSelection.toggle(email)"
            :checked="emailSelection.emails.has(email)"
          >
        </td>
        <td @click="openEmail(email)">{{ email .from }}</td>
        <td @click="openEmail(email)">
          <p>{{ email.subject }}</p>
        </td>
        <td @click="openEmail(email)">
          <p>
            <strong>{{ email.subject }}</strong> - {{ email.body }}
          </p>
        </td>
        <td class="date" @click="openEmail(email)">
          {{ format(new Date(email.sentAt), 'MMM do yyyy') }}
        </td>
        <td>
          <button @click="archiveEmail(email)">
            Archive
          </button>
        </td>
      </tr>
    </tbody>
  </table>
  <ModalView
    v-if="openedEmail"
    @update:closeModal="openedEmail = null"
  >
    <MailView
      :email="openedEmail"
      @update:changeEmail="changeEmail"
    />
  </ModalView>
</template>

<script>
  import { format } from 'date-fns'
  import axios from 'axios'
  import MailView from './MailView'
  import ModalView from './ModalView'
  import BulkActionBar from "./BulkActionBar";
  import useEmailSelection from '../composables/use-email-selection'
  import { reactive, toRefs, ref } from "vue";

  export default {
    components: {
      MailView,
      ModalView,
      BulkActionBar
    },
    async setup() {
      const result = await axios.get('http://localhost:3000/emails');
      let emailsData = reactive({
        emails: result.data,
        openedEmail: null
      })

      return {
        emailSelection: useEmailSelection(),
        ...toRefs(emailsData),
        selectedScreen: ref('inbox'),
        format
      }
    },
    computed: {
      sortedEmail() {
        console.log(this.emails)
        return this.emails.slice().sort((e1, e2) => {
          return e1.sentAt - e2.sentAt ? 1 : -1
        })
      },
      filteredEmails() {
        if(this.selectedScreen === 'inbox') {
          return this.sortedEmail.filter(e => !e.archived)
        } else {
          return this.sortedEmail.filter(e => e.archived)
        }
      }
    },
    methods: {
      selectScreen(newScreen) {
        this.selectedScreen = newScreen;
        this.emailSelection.clear();
      },
      changeEmail({toggleRead, toggleArchive, save, closeModal, changeIndex}) {
        const email = this.openedEmail
        if (toggleRead) {
          email.read = !email.read
        }
        if (toggleArchive) {
          email.archived = !email.archived
        }
        if (save) {
          this.updateEmail(email)
        }
        if (closeModal) {
          this.openedEmail = null
        }

        if (changeIndex) {
          const emails = this.filteredEmails
          const currentIndex = emails.indexOf(this.openedEmail)
          const newEmail = emails[currentIndex + changeIndex]
          this.openEmail(newEmail)
        }
      },
      openEmail(email) {
        this.openedEmail = email

        if (email) {
          email.read = true
          axios.put(`http://localhost:3000/emails/${email.id}`, email)
        }
      },
      archiveEmail(email) {
        email.archived = true
        this.updateEmail(email)
      },
      updateEmail(email) {
        axios.put(`http://localhost:3000/emails/${email.id}`, email)
      }
    }
  }
</script>

<style scoped>

</style>
