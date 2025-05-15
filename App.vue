<template>
  <div class="container mx-auto p-4">
    <div v-if="!loggedIn">
      <h2 class="text-xl font-bold mb-4">ERPNext Login</h2>
      <frappe-input v-model="email" label="Email" />
      <frappe-input v-model="password" type="password" label="Password" />
      <frappe-button class="mt-4" @click="login">Login</frappe-button>
      <p v-if="error" class="text-red-600 mt-2">{{ error }}</p>
    </div>

    <div v-else>
      <h2 class="text-xl font-bold mb-4">Field Visit Form</h2>
      <frappe-input v-model="form.customer" label="Customer Name" />
      <frappe-input v-model="form.visit_date" type="date" label="Visit Date" />
      <frappe-input v-model="form.location" label="Location" />
      <frappe-text-area v-model="form.purpose" label="Purpose" />
      <frappe-text-area v-model="form.notes" label="Notes" />

      <frappe-button class="mt-4" @click="submitForm">Submit</frappe-button>
      <frappe-button class="mt-4 ml-2" @click="exportPdf">Export PDF</frappe-button>
      <frappe-button class="mt-4 ml-2" @click="exportExcel">Export Excel</frappe-button>

      <p v-if="success" class="text-green-600 mt-2">{{ success }}</p>
      <p v-if="error" class="text-red-600 mt-2">{{ error }}</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import jsPDF from 'jspdf'
import * as XLSX from 'xlsx'

export default {
  data() {
    return {
      email: '',
      password: '',
      loggedIn: false,
      token: '',
      error: '',
      success: '',
      form: {
        customer: '',
        visit_date: '',
        location: '',
        purpose: '',
        notes: ''
      }
    }
  },
  methods: {
    async login() {
      this.error = ''
      try {
        const res = await axios.post('https://hnscrm.frappe.cloud/api/method/login', {
          usr: this.email,
          pwd: this.password
        }, { withCredentials: true })

        if (res.status === 200) {
          this.loggedIn = true
          this.success = 'Login successful!'
        }
      } catch (err) {
        this.error = 'Login failed. Check credentials.'
      }
    },

    async submitForm() {
      this.error = ''
      this.success = ''
      try {
        const data = {
          customer: this.form.customer,
          visit_date: this.form.visit_date,
          location: this.form.location,
          purpose: this.form.purpose,
          notes: this.form.notes
        }

        const res = await axios.post('https://hnscrm.frappe.cloud/api/resource/Field Visit', data, {
          withCredentials: true,
          headers: { 'Content-Type': 'application/json' }
        })

        if (res.status === 200 || res.status === 201) {
          this.success = 'Field Visit saved successfully!'
          this.form = { customer: '', visit_date: '', location: '', purpose: '', notes: '' }
        }
      } catch (err) {
        this.error = 'Failed to save Field Visit. Make sure you are logged in.'
      }
    },

    exportPdf() {
      const doc = new jsPDF()
      doc.text('Field Visit Report', 10, 10)
      doc.text(`Customer: ${this.form.customer}`, 10, 20)
      doc.text(`Visit Date: ${this.form.visit_date}`, 10, 30)
      doc.text(`Location: ${this.form.location}`, 10, 40)
      doc.text(`Purpose: ${this.form.purpose}`, 10, 50)
      doc.text(`Notes: ${this.form.notes}`, 10, 60)
      doc.save('field_visit_report.pdf')
    },

    exportExcel() {
      const ws = XLSX.utils.json_to_sheet([this.form])
      const wb = XLSX.utils.book_new()
      XLSX.utils.book_append_sheet(wb, ws, 'Field Visit')
      XLSX.writeFile(wb, 'field_visit_report.xlsx')
    }
  }
}
</script>

<style>
/* Tailwind CSS या अपनी पसंद की CSS यहां यूज़ करें */
</style>