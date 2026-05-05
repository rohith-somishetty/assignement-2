<template>
  <div class="container py-5">
    <!-- Header Section -->
    <header class="row mb-5">
      <div class="col text-center">
        <h1 class="display-4 text-primary fw-bold">Employee Portal</h1>
        <p class="lead text-secondary">Manage your workforce efficiently</p>
      </div>
    </header>

    <div class="row g-4">
      <!-- Form Section -->
      <section class="col-lg-4">
        <div class="card shadow-sm border-0 h-100">
          <div class="card-body p-4">
            <h2 class="h4 mb-4 text-dark border-bottom pb-2">
              {{ formTitle }}
            </h2>
            
            <form @submit.prevent="handleSubmit">
              <div class="mb-3">
                <label for="empName" class="form-label fw-semibold text-secondary">Full Name</label>
                <input 
                  id="empName"
                  v-model.trim="formData.Name" 
                  type="text" 
                  class="form-control" 
                  required 
                  placeholder="Name" 
                />
              </div>

              <div class="mb-3">
                <label for="empEmail" class="form-label fw-semibold text-secondary">Email Address</label>
                <input 
                  id="empEmail"
                  v-model.trim="formData.email" 
                  type="email" 
                  class="form-control" 
                  required 
                  placeholder="Email" 
                />
              </div>

              <div class="mb-3">
                <label for="empSalary" class="form-label fw-semibold text-secondary">Salary</label>
                <input 
                  id="empSalary"
                  v-model.number="formData.salary" 
                  type="number" 
                  class="form-control" 
                  required 
                  placeholder="Salary" 
                />
              </div>

              <div class="mb-3">
                <label for="empDepartment" class="form-label fw-semibold text-secondary">Department</label>
                <input 
                  id="empDepartment"
                  v-model.trim="formData.department" 
                  type="text" 
                  class="form-control" 
                  required 
                  placeholder="Department" 
                />
              </div>

              <div class="mb-4">
                <label for="empDesignation" class="form-label fw-semibold text-secondary">Designation</label>
                <input 
                  id="empDesignation"
                  v-model.trim="formData.designation" 
                  type="text" 
                  class="form-control" 
                  required 
                  placeholder="Designation" 
                />
              </div>

              <div class="d-grid gap-2">
                <button type="submit" class="btn" :class="submitButtonClass">
                  {{ submitButtonText }}
                </button>
                <button 
                  v-if="isEditing" 
                  type="button" 
                  @click="cancelEdit" 
                  class="btn btn-outline-secondary"
                >
                  Cancel
                </button>
              </div>
            </form>
          </div>
        </div>
      </section>

      <!-- Table Section -->
      <section class="col-lg-8">
        <div class="card shadow-sm border-0 h-100">
          <div class="card-body p-4">
            <h2 class="h4 mb-4 text-dark border-bottom pb-2">Employee Directory</h2>
            
            <div class="table-responsive">
              <table v-if="hasEmployees" class="table table-hover align-middle">
                <thead class="table-light">
                  <tr>
                    <th scope="col">Name</th>
                    <th scope="col">Email</th>
                    <th scope="col">Salary</th>
                    <th scope="col">Department</th>
                    <th scope="col">Designation</th>
                    <th scope="col" class="text-end">Actions</th>
                  </tr>
                </thead>
                <transition-group name="list" tag="tbody">
                  <tr v-for="emp in employees" :key="emp.id">
                    <td class="fw-semibold">{{ emp.Name }}</td>
                    <td class="text-muted">{{ emp.email }}</td>
                    <td class="text-muted">₹{{ emp.salary }}</td>
                    <td>{{ emp.department }}</td>
                    <td><span class="badge bg-info text-dark">{{ emp.designation }}</span></td>
                    <td class="text-end">
                      <button 
                        @click="editEmployee(emp)" 
                        class="btn btn-sm btn-outline-primary me-2" 
                        title="Edit Employee"
                        aria-label="Edit Employee"
                      >
                        Edit
                      </button>
                      <button 
                        @click="deleteEmployee(emp.id)" 
                        class="btn btn-sm btn-outline-danger" 
                        title="Delete Employee"
                        aria-label="Delete Employee"
                      >
                        Delete
                      </button>
                    </td>
                  </tr>
                </transition-group>
              </table>
              
              <!-- Empty State -->
              <div v-else class="text-center py-5 text-muted">
                <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" class="mb-3 opacity-50"><circle cx="12" cy="12" r="10"></circle><path d="M16 16s-1.5-2-4-2-4 2-4 2"></path><line x1="9" y1="9" x2="9.01" y2="9"></line><line x1="15" y1="9" x2="15.01" y2="9"></line></svg>
                <p class="mb-0">No employees found. Add some team members to get started!</p>
              </div>
            </div>
          </div>
        </div>
      </section>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

// IMPORTANT: Replace this with your actual MockAPI Endpoint
const API_URL = 'https://69f9d2b9c509a40d3aa36200.mockapi.io/employee'; 

export default {
  name: 'EmployeeManager',
  
  data() {
    return {
      employees: [],
      formData: this.getInitialFormData(),
      isEditing: false,
      isLoading: false
    }
  },

  mounted() {
    this.fetchEmployees();
  },

  computed: {
    hasEmployees() {
      return this.employees.length > 0;
    },
    formTitle() {
      return this.isEditing ? 'Edit Employee' : 'Add New Employee';
    },
    submitButtonText() {
      return this.isEditing ? 'Update Employee' : 'Add Employee';
    },
    submitButtonClass() {
      return this.isEditing ? 'btn-success' : 'btn-primary';
    }
  },

  methods: {
    getInitialFormData() {
      return {
        id: null,
        Name: '',
        email: '',
        salary: null,
        department: '',
        designation: ''
      };
    },

    async fetchEmployees() {
      this.isLoading = true;
      try {
        const response = await axios.get(API_URL);
        this.employees = response.data;
      } catch (error) {
        console.error("Error fetching employees:", error);
        alert("Please make sure to set your real MockAPI URL at the top of the script!");
      } finally {
        this.isLoading = false;
      }
    },

    async handleSubmit() {
      if (this.isEditing) {
        await this.updateEmployee();
      } else {
        await this.addEmployee();
      }
      this.resetForm();
    },

    async addEmployee() {
      try {
        const response = await axios.post(API_URL, this.formData);
        this.employees.push(response.data);
      } catch (error) {
        console.error("Error adding employee:", error);
      }
    },

    async updateEmployee() {
      try {
        const response = await axios.put(`${API_URL}/${this.formData.id}`, this.formData);
        const index = this.employees.findIndex(e => e.id === this.formData.id);
        if (index !== -1) {
          this.employees.splice(index, 1, response.data);
        }
      } catch (error) {
        console.error("Error updating employee:", error);
      }
    },

    editEmployee(emp) {
      this.formData = { ...emp };
      this.isEditing = true;
    },

    async deleteEmployee(id) {
      if (confirm('Are you sure you want to remove this employee?')) {
        try {
          await axios.delete(`${API_URL}/${id}`);
          this.employees = this.employees.filter(e => e.id !== id);
          
          if (this.isEditing && this.formData.id === id) {
            this.cancelEdit();
          }
        } catch (error) {
          console.error("Error deleting employee:", error);
        }
      }
    },

    cancelEdit() {
      this.resetForm();
    },

    resetForm() {
      this.formData = this.getInitialFormData();
      this.isEditing = false;
    }
  }
}
</script>

<style scoped>
@import url('https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css');
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');

.container {
  font-family: 'Inter', sans-serif;
}

/* Animations for adding/removing table rows */
.list-enter-active,
.list-leave-active {
  transition: all 0.4s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}
</style>
