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
          <b-button pill variant="info" v-on:click="getEmployees"
            >Search</b-button
          >
        </div>
      </div>
      <div class="float-right ml-auto">
        <b-button pill variant="success" v-on:click="showCreate"
          >Create</b-button
        >
      </div>
    </div>
    <b-container fluid class="text-light text-center">
      <b-table
        striped
        hover
        show-empty
        :items="employees"
        :fields="fields"
        :current-page="currentPage"
        :per-page="0"
      >
        <template #cell(subareas)="data">
          {{ data.item.subareas.name }}
        </template>
        <template #cell(action)="data">
          <b-button variant="danger" @click="removeEmployee(data.item)"
            >Delete</b-button
          >
          <b-button variant="info" @click="showUpdate(data.item)"
            >Edit</b-button
          >
        </template>
      </b-table>
      <b-pagination
        :total-rows="totalItems"
        v-model="currentPage"
        :per-page="perPage"
        @change="handlePageChange"
      >
      </b-pagination>
    </b-container>

    <div id="modal">
      <b-modal id="create" title="Employee">
        <b-form @submit="createEmployee" @reset="resetForm">
          <b-form-group
            id="input-group-1"
            label="First name:"
            label-for="input-1"
          >
            <b-form-input
              id="input-1"
              type="text"
              v-model="employee.name"
              placeholder="Enter employee name"
              required
            ></b-form-input>
          </b-form-group>

          <b-form-group
            id="input-group-2"
            label="Last Name:"
            label-for="input-2"
          >
            <b-form-input
              id="input-2"
              placeholder="Enter lastname"
              v-model="employee.last_name"
              required
            ></b-form-input>
          </b-form-group>
          <b-form-group
            id="input-group-3"
            label="Type document:"
            label-for="input-3"
          >
            <b-form-input
              id="input-3"
              placeholder="Enter type document"
              v-model="employee.type_document"
              required
            ></b-form-input>
          </b-form-group>

          <b-form-group
            id="input-group-4"
            label="Document:"
            label-for="input-4"
          >
            <b-form-input
              id="input-4"
              placeholder="Enter document"
              v-model="employee.document"
              required
            ></b-form-input>
          </b-form-group>
          <b-form-group id="input-group-5" label="Areas:" label-for="input-5">
            <b-form-select
              id="input-5"
              v-model="area"
              :options="areas"
              required
              value-field="id"
              text-field="name"
              @change="getSubAreas()"
            ></b-form-select>
          </b-form-group>

          <b-form-group
            id="input-group-6"
            label="SubAreas:"
            label-for="input-6"
          >
            <b-form-select
              id="input-6"
              v-model="employee.subarea_id"
              :options="subareas"
              required
              value-field="id"
              text-field="name"
              @change="getSubAreas()"
            ></b-form-select>
          </b-form-group>
          <b-button type="submit" variant="primary">Submit</b-button>
          <b-button type="reset" variant="danger">Reset</b-button>
        </b-form>
      </b-modal>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import VueToast from "vue-toast-notification";
import { nextTick } from "vue";
import "vue-toast-notification/dist/theme-sugar.css";

Vue.use(VueToast);

export default Vue.extend({
  data() {
    return {
      employees: [],
      employee: {},
      areas: [],
      subareas: [],
      area: null,
      // search and paginate
      perPage: 10,
      currentPage: 1,
      search: "",
      totalItems: 0,
      // search and paginate
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

  computed: {
    rows() {
      return this.employees.length;
    },
  },
  async created() {
    this.getEmployees();
    this.getAreas();
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
    async handlePageChange(value) {
      this.currentPage = value;
      await this.getEmployees();
    },
    async createEmployee(event) {
      event.preventDefault();
      if (this.employee.id) {
        this.updateEmployee(this.employee);
      } else {
        this.newEmployee(this.employee);
      }
    },
    resetForm(event) {
      event.preventDefault();
      this.employee = {};
    },
    async getEmployees() {
      let url = `http://localhost:3000/api/employees?page=${this.currentPage}`;
      if (this.search) {
        url = `${url}&search=${this.search}`;
      }
      try {
        const response = await fetch(url, {
          headers: {
            "Content-Type": "application/json",
          },
          credentials: "include",
        });

        let data = await response.json();
        this.employees = data.employees.rows;
        this.totalItems = data.employees.count;
      } catch (error) {
        Vue.$toast.error(error.message);
      }
    },
    async getAreas() {
      try {
        const response = await fetch("http://localhost:3000/api/areas", {
          headers: {
            "Content-Type": "application/json",
          },
          credentials: "include",
        });

        this.areas = await response.json();
      } catch (error) {
        Vue.$toast.error(error.message);
      }
    },
    async getSubAreas() {
      const areaId = this.area;
      try {
        const response = await fetch(
          `http://localhost:3000/api/areas/subareas/${areaId}`,
          {
            headers: {
              "Content-Type": "application/json",
            },
            credentials: "include",
          }
        );

        let data = await response.json();

        this.subareas = data.subareas;
      } catch (error) {
        this.subareas = [];
        Vue.$toast.error(error.message);
      }
    },
    async newEmployee(employee) {
      try {
        await fetch("http://localhost:3000/api/employees/", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          credentials: "include",
          body: JSON.stringify({
            name: employee.name,
            last_name: employee.last_name,
            type_document: employee.type_document,
            document: employee.document,
            subarea_id: employee.subarea_id,
          }),
        });
        await this.getEmployees();
        Vue.$toast.success("record created sucessfully");
      } catch (error) {
        Vue.$toast.error(error.message);
      }
    },
    async updateEmployee(employee) {
      try {
        await fetch(`http://localhost:3000/api/employees/${employee.id}`, {
          method: "PUT",
          headers: { "Content-Type": "application/json" },
          credentials: "include",
          body: JSON.stringify({
            name: employee.name,
            last_name: employee.last_name,
            type_document: employee.type_document,
            document: employee.document,
            subarea_id: employee.subarea_id,
          }),
        });
        await this.getEmployees();
        Vue.$toast.success("record updated sucessfully");
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
    async getAreaWithId(id) {
      try {
        const response = await fetch(`http://localhost:3000/api/areas/${id}`, {
          headers: {
            "Content-Type": "application/json",
          },
          credentials: "include",
        });

        let data = await response.json();
        this.area = data.area;
      } catch (error) {
        Vue.$toast.error(error.message);
      }
    },

    async removeEmployee(employee) {
      if (confirm(`Do you want to remove record: ${employee.name}`)) {
        try {
          const response = await fetch(
            `http://localhost:3000/api/employees/${employee.id}`,
            {
              method: "DELETE",
              headers: {
                "Content-Type": "application/json",
              },
              credentials: "include",
            }
          );

          Vue.$toast.success("record deleted sucessfully");
          await this.getEmployees();
        } catch (error) {
          Vue.$toast.error(error.message);
        }
      }
    },
    showCreate: function () {
      this.$bvModal.show("create");
      this.employee = {};
    },

    showUpdate: async function (employee) {
      this.$bvModal.show("create");
      this.employee = employee;
      this.subareas = [this.employee.subareas];

      const area_id = this.employee.subareas.area_id;

      this.area = this.areas.find((item) => {
        return item.id == area_id;
      }).id;
    },
  },
});
</script>
