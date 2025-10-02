<script setup>
import { reactive } from 'vue';
import html2pdf from 'html2pdf.js';


const data = reactive({
  invoiceNumber: '',
  sender: '',
  date: '',
  dueDate: '',
  billTo: '',
  shipTo: '',
  items: [
    {
      description: '',
      quantity: '',
      rate: '',
      amount: '',
    }
  ],
  
  notes: '',
  subtotal: '',
  tax: '',
  total: ''
  
})

function addMoreItem() {
  data.items.push({
    description: '',
    quantity: '',
    rate: '',
    amount: '',
  })
}

function getSubtotal(){
  let subtotal = 0;
  data.items.forEach(item => {
    subtotal += item.amount
  })
  data.subtotal = subtotal
  return subtotal
}

function getTotal(){
  const tax = data.subtotal * data.tax / 100;
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
  const element = document.querySelector('.container');
  html2pdf().from(element).save('invoice.pdf');
}
</script>

<template>
  <div class="container mt-5">
    <div class="text-end">
      <h2 class=" mb-1">Invoice</h2>
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
      <thead class="table-dark">
          <tr>
              <th>Description</th>
              <th>Quantity</th>
              <th>Rate</th>
              <th>Amount</th>
              <th>Action</th>
          </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in data.items" :key="index">
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
          <p>Subtotal: <input class="text-end" :value="getSubtotal()" readonly placeholder="Subtotal"></p>
          <p>Tax: <input class="text-end" type="number" v-model="data.tax" placeholder="Tax"></p>
          <p>Total: <input :value="getTotal()"  class="text-end" readonly placeholder="Total"></p>
      </div>
    </div>

    <div>
      <button @click="printInvoice()" class="btn btn-primary mt-3">Print Invoice</button>
      <button @click="downloadPDF()" class="btn btn-secondary mt-3 ms-2">Download PDF</button>
    </div>

  </div>
</template>




        
        
        
 
<style scoped>
input{
  border: 1px solid #ddd;
}
</style>




