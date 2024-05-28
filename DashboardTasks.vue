<template>
  <div class="components component-tasks" :class="{ loading: loading }">
    <Loading />

    <header
      class="main-data"
      :style="{
        background: `linear-gradient(45deg, ${color}, ${color}b8)`,
      }"
    >
      <span class="title">Tâches du jour et en retard</span>

      <ul class="actions">
        <li>
          <NuxtLink to="/tasks"><span uk-icon="search"></span></NuxtLink>
        </li>
      </ul>
    </header>

    <div class="content">
      <table class="uk-table" v-if="tasks.length > 0">
        <thead>
          <tr>
            <th>Résident</th>
            <th>Type</th>
            <th>Deadline</th>
            <th class="right">Actions</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(task, index) in tasks" :key="index">
            <td>
              <img
                v-if="task.resident.picture"
                :src="`${$axios.defaults.baseURL}/storage/residents/pictures/${task.resident.picture}`"
              />
              <img v-else src="~/assets/img/default-resident-picture.png" />
              {{ task.resident.lastname }} {{ task.resident.firstname }}
            </td>
            <td>{{ task.task_type.name }}</td>
            <td>{{ moment(task.deadline_end).format("HH:mm") }}</td>
            <td>
              <ul class="actions">
                <li>
                  <NuxtLink :to="`/tasks/${task.id}`"
                    ><span uk-icon="search"></span
                  ></NuxtLink>
                </li>
              </ul>
            </td>
          </tr>
        </tbody>
      </table>
      <span class="no-data" v-else>Aucune tâche</span>
      <div style="padding: 4px">
        <sliding-pagination
          v-if="tasks.length > 0"
          :current="currentPage"
          :total="totalPages"
          @page-change="get"
        ></sliding-pagination>
      </div>
    </div>
  </div>
</template>

<script>
import moment from "moment";
import SlidingPagination from "vue-sliding-pagination";

export default {
  auth: true,
  layout: "app",
  props: ["color"],
  components: {
    SlidingPagination,
  },
  data() {
    return {
      currentPage: 0,
      totalPages: 0,
      tasks: [],
    };
  },
  mounted() {
    this.get();
  },
  methods: {
    moment(date) {
      return moment(date);
    },
    async get(page = 1) {
      try {
        this.$store.commit("loader/set", true);
        const response = await this.$axios.get("/api/tasks", {
          params: {
            date_from: moment().format("YYYY-MM-DD 00:00:00"),
            mine: true,
            status: [0],
            establishment_id:
              JSON.parse(localStorage.getItem("establishmentSelected"))?.id !==
              "all"
                ? JSON.parse(localStorage.getItem("establishmentSelected")).id
                : "",
            paginate: true,
            display: 5,
            page: page,
          },
        });
        this.tasks = response.data.data.tasks.data;
        this.totalPages = response.data.data.tasks.last_page;
        this.currentPage = response.data.data.tasks.current_page;
        this.$store.commit("loader/set", false);
      } catch (error) {
        this.showNotification(
          "Une erreur est survenue, impossible de récupérer les tâches !",
          "danger",
        );
        this.$store.commit("loader/set", false);
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