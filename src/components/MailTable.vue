<template>
  <table class="mail-table">
    <tbody>
      <tr
        v-for="email in unarchivedEmails"
        :key="email.id"
        :class="['clickable', email.read ? 'read' : '']"
        @click="openEmail(email)"
      >
        <td>
          <input type="checkbox" />
        </td>
        <td>{{email.from}}</td>
        <td>
          <p>
            <strong>{{email.subject}}</strong>
            - {{email.body}}
          </p>
        </td>
        <td class="date">{{format(new Date(email.sentAt), 'MMM do yyyy')}}</td>
        <td>
          <button @click="archiveEmail(email)">Archive</button>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script>
import { format } from "date-fns";
import axios from "axios";
import { ref, computed } from "vue";

import MailView from '@/components/MailView.vue';

export default {
  async setup() {
    const emails = ref([])
    const openedEmail = ref(null)
    const response = await axios.get("http://localhost:3000/emails");
    emails.value = response.data;

    const sortedEmails = computed(() => {
      return emails.value.sort((e1, e2) => {
        return e1.sentAt < e2.sentAt ? 1 : -1;
      });
    });

    const unarchivedEmails = computed(() => {
      return sortedEmails.value.filter((e) => !e.archived);
    });

    const updateEmail = (email) => {
      axios.put(`http://localhost:3000/emails/${email.id}`, email);
    };

    const openEmail = (email) => {
      email.read = true;
      openedEmail.value = email
      updateEmail(email);
    };

    const archiveEmail = (email) => {
      email.archived = true;
      updateEmail(email);
    };

    return {
      format,
      emails,
      openedEmail,
      unarchivedEmails,
      openEmail,
      archiveEmail,
    };
  },
};
</script>

<style scoped>
</style>