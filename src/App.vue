<script setup>
import { reactive, ref } from 'vue';
import html2pdf from 'html2pdf.js';

const data = reactive({
  invoiceNumber: '',
  sender: '',
  date: '',
  dueDate: '',
  billTo: '',
  shipTo: '',
  items: [{ description: '', quantity: '', rate: '', amount: '' }],
  notes: '',
  subtotal: '',
  tax: '',
  total: ''
})

// 🎨 Theme settings
const theme = reactive({
  tableHeaderBg: '#212529', 
  tableHeaderText: '#ffffff', 
  tableRowBg: '#ffffff', 
  tableRowText: '#000000', 
  headerBg: '#0d6efd',
  headerText: '#ffffff',
  templateBg: '#ffffff'
})

// Sidebar toggle
const showSidebar = ref(true)

function addMoreItem() {
  data.items.push({ description: '', quantity: '', rate: '', amount: '' })
}

function getSubtotal(){
  let subtotal = 0;
  data.items.forEach(item => subtotal += item.amount)
  data.subtotal = subtotal
  return subtotal
}

function getTotal(){
  const tax = data.subtotal * (data.tax || 0) / 100;
  data.total = data.subtotal + tax;
  return data.total
}

function deleteItem(index) {
  data.items.splice(index, 1);
}

function printInvoice() {
  window.print();
}

function downloadPDF() {
  const element = document.querySelector('.invoice-container');
  
  const now = new Date();
  const date = now.toISOString().split('T')[0]; 
  const time = now.toTimeString().split(' ')[0].replace(/:/g, '-'); 

  const filename = `invoice_${date}_${time}.pdf`;

  html2pdf().from(element).save(filename);
}
</script>

<template>
  <div class="d-flex">
    <!-- Sidebar -->
    <div v-if="showSidebar" class="sidebar p-3 shadow">
      <h5 class="mb-3">🎨 Theme Settings</h5>

      <div class="mb-3">
        <label>Header Background</label>
        <input type="color" v-model="theme.headerBg" class="form-control form-control-color">
      </div>

      <div class="mb-3">
        <label>Header Text</label>
        <input type="color" v-model="theme.headerText" class="form-control form-control-color">
      </div>

      <div class="mb-3">
        <label>Table Header Bg</label>
        <input type="color" v-model="theme.tableHeaderBg" class="form-control form-control-color">
      </div>

      <div class="mb-3">
        <label>Table Header Text</label>
        <input type="color" v-model="theme.tableHeaderText" class="form-control form-control-color">
      </div>

      <div class="mb-3">
        <label>Table Row Bg</label>
        <input type="color" v-model="theme.tableRowBg" class="form-control form-control-color">
      </div>

      <div class="mb-3">
        <label>Table Row Text</label>
        <input type="color" v-model="theme.tableRowText" class="form-control form-control-color">
      </div>

      <div class="mb-3">
        <label>Template Background</label>
        <input type="color" v-model="theme.templateBg" class="form-control form-control-color">
      </div>

      <!-- Hide sidebar button -->
      <button @click="showSidebar = false" class="btn btn-danger btn-sm mt-3">Hide Dashboard</button>
    </div>

    <!-- Main Invoice -->
    <div class="flex-grow-1 p-3" :class="{ 'full-width': !showSidebar }">
      <div class="invoice-container p-3 rounded"
           :style="{ backgroundColor: theme.templateBg }">

        <div class="text-end">
          <h2 class="p-2 rounded"
              :style="{ backgroundColor: theme.headerBg, color: theme.headerText }">
              Invoice
          </h2>
          <label class="form-label">Invoice Number</label><br/>
          <input v-model="data.invoiceNumber" type="text"  class="form-control" placeholder="#">
        </div>

        <div class="row my-3">
          <div class="col-md-6">
              <label class="form-label">Sender</label>
              <input v-model="data.sender" type="text"  class="form-control" placeholder="Sender Name">
          </div>
          <div class="col-md-3">
              <label class="form-label">Date</label>
              <input v-model="data.date" type="date" class="form-control">
          </div>
          <div class="col-md-3">
              <label class="form-label">Due Date</label>
              <input v-model="data.dueDate" type="date"  class="form-control">
          </div>
        </div>
        
        <div class="row mb-3">
          <div class="col-md-6">
              <label class="form-label">Bill To</label>
              <input v-model="data.billTo" type="text"  class="form-control" placeholder="Client Name">
          </div>
          <div class="col-md-6">
              <label class="form-label">Ship To</label>
              <input v-model="data.shipTo" type="text"  class="form-control" placeholder="Shipping Address">
          </div>
        </div>

        <table class="table table-bordered mt-3">
          <thead :style="{ backgroundColor: theme.tableHeaderBg, color: theme.tableHeaderText}">
              <tr>
                  <th>Description</th>
                  <th>Quantity</th>
                  <th>Rate</th>
                  <th>Amount</th>
                  <th>Action</th>
              </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in data.items" :key="index"
                :style="{ backgroundColor: theme.tableRowBg, color: theme.tableRowText}">
                <td><input v-model="item.description" type="text" class="form-control"></td>
                <td><input v-model="item.quantity" type="number" class="form-control"></td>
                <td><input v-model="item.rate" type="number" class="form-control"></td>
                <td class="align-middle fw-bold">{{ item.amount = item.quantity * item.rate }}</td>
                <td>
                  <button @click="deleteItem(index)" type="button" class="btn btn-danger btn-sm">Delete</button>
                </td>
              </tr>
          </tbody>
        </table>
        
        <button @click="addMoreItem()" type="button" class="btn btn-success">Add More</button>
   
        <div class="row mt-4">
          <div class="col-md-6">
              <label class="form-label">Notes</label>
              <textarea v-model="data.notes" class="form-control" rows="3"></textarea>
          </div>
          <div class="col-md-6 text-end">
              <p>Subtotal: <input class="text-end" :value="getSubtotal()" readonly></p>
              <p>Tax: <input class="text-end" type="number" v-model="data.tax" placeholder="Tax"></p>
              <p>Total: <input :value="getTotal()" class="text-end" readonly></p>
          </div>
        </div>

        <div>
          <button @click="printInvoice()" class="btn btn-primary mt-3">Print Invoice</button>
          <button @click="downloadPDF()" class="btn btn-secondary mt-3 ms-2">Download PDF</button>
        </div>

        <!-- Show sidebar again if hidden -->
        <div v-if="!showSidebar" class="mt-3">
          <button @click="showSidebar = true" class="btn btn-outline-secondary">Show Dashboard</button>
        </div>

      </div>
    </div>
  </div>
</template>

<style scoped>
.sidebar {
  width: 250px;
  background: #f8f9fa;
  height: 100vh;
  position: relative;
}
.full-width {
  width: 100%;
}
.invoice-container {
  transition: 0.3s;
}
</style>
