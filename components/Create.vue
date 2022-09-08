<template >
  <!-- Modal -->
  <div
    class="modal fade"
    id="createModal"
    tabindex="-1"
    role="dialog"
    aria-labelledby="exampleModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="exampleModalLabel">
            <i class="fa fa-plus-circle fa-fw" aria-hidden="true"></i>Create
            Employee
          </h5>
          <button
            type="button"
            class="close"
            data-dismiss="modal"
            aria-label="Close"
          >
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          <div class="form-group">
            <label class="control-label">Name</label>
            <input
              class="form-control"
              type="text"
              v-model="employee.name"
              name="name"
              placeholder="Enter name"
            />
          </div>
          <div class="form-group">
            <label class="control-label">Last Name</label>
            <input
              class="form-control"
              type="text"
              v-model="employee.last_name"
              name="name"
              placeholder="Enter name"
            />
          </div>
          <div class="form-group">
            <label class="control-label">Document</label>
            <input
              class="form-control"
              type="text"
              v-model="employee.documento"
              name="name"
              placeholder="Enter name"
            />
          </div>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-dismiss="modal">
            Close
          </button>
          <button type="button" class="btn btn-primary" @click="save">
            Save
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import VueToast from "vue-toast-notification";

import "vue-toast-notification/dist/theme-sugar.css";

Vue.use(VueToast);

export default {
  data() {
    return {
      endpoint: "http://localhost:3000/api/employees",
      employee: {},
    };
  },
  methods: {
    success() {
      $("#createModal").modal("hide");
      Vue.$toast.success("Suceessfully");
    },
    error() {
      Vue.$toast.error("Error!");
    },
    async save() {
      try {
        const response = await fetch(this.endpoint, {
          method: "POST",
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json",
          },
          credentials: "include",
          body: JSON.stringify(this.employee),
        });
        console.log(response);
      } catch (error) {
        Vue.$toast.error(error.message);
      }
    },
  },
};
</script>

<style>
</style>
