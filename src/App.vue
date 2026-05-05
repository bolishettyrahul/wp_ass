<template>
  <div class="container py-4">
    <!-- Header -->
    <div class="text-center mb-4">
      <h2 class="fw-bold text-primary">Employee Management System</h2>
      <p class="text-muted">CBIT - Web Programming Assignment II</p>
    </div>

    <!-- Alert -->
    <div v-if="alert.message" :class="`alert alert-${alert.type} alert-dismissible`" role="alert">
      {{ alert.message }}
      <button type="button" class="btn-close" @click="alert.message = ''"></button>
    </div>

    <!-- Form Card -->
    <div class="card shadow-sm mb-4">
      <div class="card-header bg-primary text-white fw-semibold">
        {{ isEditing ? 'Edit Employee' : 'Add New Employee' }}
      </div>
      <div class="card-body">
        <form @submit.prevent="isEditing ? updateEmployee() : addEmployee()">
          <div class="row g-3">
            <div class="col-md-6">
              <label class="form-label">Employee ID</label>
              <input v-model="form.employeeId" type="text" class="form-control" placeholder="e.g. EMP001" required :disabled="isEditing" />
            </div>
            <div class="col-md-6">
              <label class="form-label">Name</label>
              <input v-model="form.name" type="text" class="form-control" placeholder="Full Name" required />
            </div>
            <div class="col-md-4">
              <label class="form-label">Designation</label>
              <input v-model="form.designation" type="text" class="form-control" placeholder="e.g. Software Engineer" required />
            </div>
            <div class="col-md-4">
              <label class="form-label">Department</label>
              <select v-model="form.department" class="form-select" required>
                <option value="">Select Department</option>
                <option>IT</option>
                <option>HR</option>
                <option>Finance</option>
                <option>Marketing</option>
                <option>Operations</option>
              </select>
            </div>
            <div class="col-md-4">
              <label class="form-label">Salary (₹)</label>
              <input v-model.number="form.salary" type="number" class="form-control" placeholder="e.g. 50000" required min="0" />
            </div>
          </div>
          <div class="mt-3 d-flex gap-2">
            <button type="submit" class="btn btn-primary px-4">
              <span v-if="loading" class="spinner-border spinner-border-sm me-1"></span>
              {{ isEditing ? 'Update' : 'Add Employee' }}
            </button>
            <button v-if="isEditing" type="button" class="btn btn-secondary" @click="cancelEdit">Cancel</button>
          </div>
        </form>
      </div>
    </div>

    <!-- Employee Table -->
    <div class="card shadow-sm">
      <div class="card-header bg-primary text-white d-flex justify-content-between align-items-center">
        <span class="fw-semibold">Employee Records</span>
        <span class="badge bg-light text-dark">{{ employees.length }} employees</span>
      </div>
      <div class="card-body p-0">
        <div v-if="fetching" class="text-center py-4">
          <div class="spinner-border text-primary"></div>
          <p class="mt-2 text-muted">Loading employees...</p>
        </div>
        <div v-else-if="employees.length === 0" class="text-center py-5 text-muted">
          No employees found. Add one above.
        </div>
        <div v-else class="table-responsive">
          <table class="table table-hover table-striped mb-0 align-middle">
            <thead class="table-dark">
              <tr>
                <th>#</th>
                <th>Employee ID</th>
                <th>Name</th>
                <th>Designation</th>
                <th>Department</th>
                <th>Salary</th>
                <th class="text-center">Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(emp, index) in employees" :key="emp.id">
                <td>{{ index + 1 }}</td>
                <td><span class="badge bg-secondary">{{ emp.employeeId }}</span></td>
                <td class="fw-semibold">{{ emp.name }}</td>
                <td>{{ emp.designation }}</td>
                <td><span class="badge bg-info text-dark">{{ emp.department }}</span></td>
                <td>₹{{ emp.salary?.toLocaleString('en-IN') }}</td>
                <td class="text-center">
                  <button class="btn btn-sm btn-outline-warning me-1" @click="editEmployee(emp)">Edit</button>
                  <button class="btn btn-sm btn-outline-danger" @click="deleteEmployee(emp.id)">Delete</button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

// MockAPI endpoint
const API_URL = 'https://69f07f3cc1533dbedc9d145c.mockapi.io/api/employees'

export default {
  name: 'App',
  data() {
    return {
      employees: [],
      form: {
        employeeId: '',
        name: '',
        designation: '',
        department: '',
        salary: ''
      },
      isEditing: false,
      editId: null,
      loading: false,
      fetching: false,
      alert: { message: '', type: 'success' }
    }
  },
  mounted() {
    this.fetchEmployees()
  },
  methods: {
    showAlert(message, type = 'success') {
      this.alert = { message, type }
      setTimeout(() => { this.alert.message = '' }, 3000)
    },

    async fetchEmployees() {
      this.fetching = true
      try {
        const res = await axios.get(API_URL)
        this.employees = res.data
      } catch {
        this.showAlert('Failed to fetch employees.', 'danger')
      } finally {
        this.fetching = false
      }
    },

    async addEmployee() {
      this.loading = true
      try {
        const res = await axios.post(API_URL, this.form)
        this.employees.push(res.data)
        this.showAlert(`Employee ${this.form.name} added successfully!`)
        this.resetForm()
      } catch {
        this.showAlert('Failed to add employee.', 'danger')
      } finally {
        this.loading = false
      }
    },

    editEmployee(emp) {
      this.isEditing = true
      this.editId = emp.id
      this.form = {
        employeeId: emp.employeeId,
        name: emp.name,
        designation: emp.designation,
        department: emp.department,
        salary: emp.salary
      }
      window.scrollTo({ top: 0, behavior: 'smooth' })
    },

    async updateEmployee() {
      this.loading = true
      try {
        const res = await axios.put(`${API_URL}/${this.editId}`, this.form)
        const idx = this.employees.findIndex(e => e.id === this.editId)
        this.employees.splice(idx, 1, res.data)
        this.showAlert(`Employee ${this.form.name} updated successfully!`)
        this.cancelEdit()
      } catch {
        this.showAlert('Failed to update employee.', 'danger')
      } finally {
        this.loading = false
      }
    },

    async deleteEmployee(id) {
      if (!confirm('Are you sure you want to delete this employee?')) return
      try {
        await axios.delete(`${API_URL}/${id}`)
        this.employees = this.employees.filter(e => e.id !== id)
        this.showAlert('Employee deleted successfully!', 'warning')
      } catch {
        this.showAlert('Failed to delete employee.', 'danger')
      }
    },

    cancelEdit() {
      this.isEditing = false
      this.editId = null
      this.resetForm()
    },

    resetForm() {
      this.form = { employeeId: '', name: '', designation: '', department: '', salary: '' }
    }
  }
}
</script>
