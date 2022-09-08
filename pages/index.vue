<template>
  <div class="container">
    <h2 class="text-center mb-5">Employees App</h2>
    <div class="d-flex mb-5">
      <div class="row justify-content-around ml-auto flex-grow-1" style="100%">
        <div class="col-9">
          <input
            name="search"
            class="form-control"
            type="text"
            placeholder="Enter name or document"
            aria-label="Search"
            v-model="search"
          />
        </div>
        <div class="col-3">
          <button class="btn btn-primary btn-flat">Buscar</button>
        </div>
      </div>
      <div class="float-right ml-auto">
        <button class="btn btn-outline-success">
          <i
            class="fa fa-user-plus icon-expe text-sucess"
            v-on:click.prevent="showCreate()"
          ></i>
          Crear Empleado
        </button>
      </div>
    </div>
    <b-container fluid class="text-light text-center">
      <b-row>
        <b-table striped hover :items="employees" :fields="fields">
          <template #cell(subareas)="data">
            {{ data.item.subareas.name }}
          </template>
          <!-- <template #cell(action)="data">
            <b-button variant="info">Edit</b-button>
            <b-button variant="danger">Delete</b-button>
          </template> -->
        </b-table>
        <b-pagination
          v-model="currentPage"
          :total-rows="rows"
          :per-page="perPage"
          aria-controls="my-table"
        ></b-pagination>
      </b-row>
    </b-container>
    <create />
  </div>
</template>

<script>
import Vue from "vue";
import VueToast from "vue-toast-notification";

import "vue-toast-notification/dist/theme-sugar.css";

Vue.use(VueToast);

//components
import Create from "./../components/Create.vue";

export default Vue.extend({
  data() {
    return {
      employees: [],
      employee: {},
      perPage: 3,
      currentPage: 1,
      search: "",
      fields: [
        {
          key: "name",
          label: "Name",
          sortable: true,
        },
        {
          key: "last_name",
          label: "Last Name",
          sortable: false,
        },
        {
          key: "type_document",
          label: "Document Type",
          sortable: false,
        },
        {
          key: "document",
          label: "Document",
          sortable: false,
        },
        { key: "subareas", label: "SubArea" },
        {
          key: "action",
          label: "Actions",
        },
      ],
    };
  },
  components: { Create },
  computed: {
    rows() {
      return this.employees.length;
    },
  },
  async created() {
    this.getEmployees();
  },
  async mounted() {
    try {
      this.$nuxt.$emit("auth", true);
    } catch (e) {
      this.$router.push("/signin");
      this.$nuxt.$emit("auth", false);
    }
  },
  methods: {
    async getEmployees() {
      try {
        const response = await fetch("http://localhost:3000/api/employees", {
          headers: {
            "Content-Type": "application/json",
          },
          credentials: "include",
        });

        let data = await response.json();
        this.employees = data.rows;
        this.perPage = data.count;
      } catch (error) {
        Vue.$toast.error(error.message);
      }
    },
    async getEmployee(id) {
      try {
        const response = await fetch(
          `http://localhost:3000/api/employees/${id}`,
          {
            headers: {
              "Content-Type": "application/json",
            },
            credentials: "include",
          }
        );

        let data = await response.json();
        this.employee = data.employee;
      } catch (error) {
        Vue.$toast.error(error.message);
      }
    },
    showCreate: function () {
      $("#createModal").modal("show");
    },
  },
});
</script>
