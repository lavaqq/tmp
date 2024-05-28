<template>
  <div class="components component-documents" :class="{ loading: loading }">
    <Loading />

    <header
      class="main-data"
      :style="{
        background: `linear-gradient(45deg, ${color}, ${color}b8)`,
      }"
    >
      <span class="title">Documents à échéance</span>

      <ul class="actions">
        <li>
          <multiselect
            :hide-selected="true"
            v-model="documentTypeSelected"
            selectLabel=""
            selectedLabel=""
            deselectLabel=""
            placeholder="Sélectionner un type"
            label="name"
            track-by="id"
            :options="documentTypes"
            @input="loadDocuments"
          >
            <span slot="noResult" class="no-result">Aucun résultat</span>
          </multiselect>
        </li>
        <li>
          <button class="reset" @click="resetFilters">Réinitialiser</button>
        </li>
      </ul>
    </header>

    <div class="content">
      <table class="uk-table" v-if="documents.length > 0">
        <thead>
          <tr>
            <th>Résident</th>
            <th>Type</th>
            <th>Echéance</th>
            <th>Commentaires</th>
            <th class="right">Actions</th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="(document, index) in documents"
            :key="index"
            :class="document.urgent"
          >
            <td>
              <template v-if="document.resident !== null">
                {{ document.resident.lastname }}
                {{ document.resident.firstname }}
              </template>
              <template v-else>
                <span class="relation-bug">Résident supprimé</span>
              </template>
            </td>
            <td>{{ document.document_type.name }}</td>
            <td>{{ moment(document.valid_to).format("DD/MM/YYYY") }}</td>
            <td>{{ document.comments.length }}</td>
            <td>
              <ul class="actions">
                <li
                  v-if="
                    $auth.user.permissions.includes('documents.id.forceRemove')
                  "
                >
                  <button @click="forceRemove(document)">
                    <span uk-icon="trash"></span>
                  </button>
                </li>
                <li
                  v-if="$auth.user.permissions.includes('documents.id.index')"
                >
                  <NuxtLink :to="`/documents/${document.id}`">
                    <span uk-icon="search"></span>
                  </NuxtLink>
                </li>
              </ul>
            </td>
          </tr>
        </tbody>
      </table>
      <span class="no-data" v-else>Aucun document à échéance</span>
      <div style="padding: 4px">
        <sliding-pagination
          v-if="documents.length > 0"
          :current="currentPage"
          :total="totalPages"
          @page-change="loadDocuments"
        ></sliding-pagination>
      </div>
    </div>
  </div>
</template>

<script>
import moment from "moment";
import Multiselect from "vue-multiselect";
import SlidingPagination from "vue-sliding-pagination";

export default {
  auth: true,
  layout: "app",
  components: {
    Multiselect,
    SlidingPagination,
  },
  props: ["color"],
  data() {
    return {
      currentPage: 0,
      totalPages: 0,
      loading: true,
      documents: [],
      documentTypes: [],
      documentTypeSelected: "",
    };
  },
  mounted() {
    this.fetchDocumentTypes();
    this.loadDocuments();
  },
  methods: {
    moment(date) {
      return moment(date);
    },
    async fetchDocumentTypes() {
      try {
        const response = await this.$axios.get("/api/documentTypes");
        this.documentTypes = response.data.data.documentTypes;
      } catch (error) {
        this.showNotification(
          "Une erreur est survenue, impossible de récupérer les types de documents !",
          "danger",
        );
      }
    },
    async loadDocuments(page = 1) {
      this.documents = [];
      try {
        const response = await this.$axios.get("/api/documents", {
          params: {
            term: true,
            document_type_id: this.documentTypeSelected.id,
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
        const now = moment();

        this.documents = response.data.data.documents.data.map((document) => {
          document.urgent = this.getUrgencyClass(document, now);
          return document;
        });
        this.totalPages = response.data.data.documents.last_page;
        this.currentPage = response.data.data.documents.current_page;
        this.loading = false;
      } catch (error) {
        this.showNotification(
          "Une erreur est survenue, impossible de récupérer les documents !",
          "danger",
        );
        this.loading = false;
      }
    },
    getUrgencyClass(document, now) {
      if (moment(document.valid_to).isBefore(now)) {
        return "red";
      }
      if (
        moment(document.valid_to)
          .subtract(document.document_type.term2, "days")
          .isBefore(now)
      ) {
        return "orange";
      }
      return "";
    },
    async forceRemove(document) {
      this.loading = true;
      try {
        await this.$axios.get(`/api/documents/${document.id}/forceRemove`);
        this.loadDocuments();
      } catch (error) {
        this.showNotification(
          "Une erreur est survenue, impossible de supprimer le document !",
          "danger",
        );
        this.loading = false;
      }
    },
    resetFilters() {
      this.documentTypeSelected = "";
      this.loadDocuments();
    },
    showNotification(message, status) {
      UIkit.notification({
        message: `<span uk-icon='icon: check'></span> ${message}`,
        status,
        timeout: 5000,
      });
    },
  },
};
</script>