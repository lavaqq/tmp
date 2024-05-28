<template>
  <div class="components">
    <header
      class="main-data"
      :style="
        'background: linear-gradient(45deg, ' + color + ', ' + color + 'b8);'
      "
    >
      <span class="title">Therapylink</span>
    </header>

    <div
      style="
        padding: 16px;
        display: flex;
        flex-direction: column;
        gap: 16px;
        background-color: white;
      "
    >
      <VueCtkDateTimePicker
        format="YYYY-MM-DD"
        formatted="DD/MM/YYYY"
        :only-date="true"
        label="Sélectionnez une date de début"
        button-now-translation="Maintenant"
        locale="fr"
        :overlay="true"
        v-model="date_from"
      />
      <input
        type="text"
        class="uk-input"
        v-model="days"
        placeholder="Nombre de jours"
        @input="date_to = ''"
        @keydown='validateInteger'
      />
      <VueCtkDateTimePicker
        format="YYYY-MM-DD"
        formatted="DD/MM/YYYY"
        :only-date="true"
        label="Sélectionnez une date de fin"
        button-now-translation="Maintenant"
        locale="fr"
        v-model="date_to"
        :overlay="true"
        :min-date="moment(date_from).add(1, 'days').format('YYYY-MM-DD')"
        :max-date="'2999-12-24'"
        :disabled="days != ''"
      />
      <button
        class="uk-button uk-button-primary"
        @click="send"
        :disabled="sendDisabled"
      >
        Envoyer le therapylink
      </button>
    </div>
  </div>
</template>

<script>
import moment from "moment";
import VueCtkDateTimePicker from "vue-ctk-date-time-picker";

export default {
  auth: true,
  layout: "app",
  props: ["color"],
  components: {
    VueCtkDateTimePicker,
  },
  data() {
    return {
      date_from: moment().format('YYYY-MM-DD'),
      date_to: "",
      days: "",
      sendDisabled: false,
    };
  },
  methods: {
    validateInteger(event) {
      if (event.key.length == 1 && !(/^\d$/.test(event.key))) {
        event.preventDefault();
      }
    },
    moment(date) {
      return moment(date);
    },
    async send() {
      this.sendDisabled = true;
      try {
        await this.$axios.get("/api/therapylink/send", {
          params: {
            date_from: this.date_from,
            date_to: this.date_to,
            days: this.days,
          },
        });
        this.showNotification("Therapylink envoyé !", "success");
        this.sendDisabled = false;
      } catch (error) {
        this.showNotification(
          "Une erreur est survenue, impossible d'envoyé le therapylink !",
          "danger",
        );
        this.sendDisabled = false;
      }
    },
    showNotification(message, status) {
      UIkit.notification({
        message: `<span uk-icon='icon: check'></span> ${message}`,
        status: status,
        timeout: 5000,
      });
    },
  },
};
</script>