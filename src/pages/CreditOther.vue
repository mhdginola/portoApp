<template>
  <div class="container mt-5">
    <h2 class="mb-4">Kalkulator Kredit</h2>
    
    <div class="mb-3">
      <label class="form-label">Harga Barang (Rp)</label>
      <input type="number" class="form-control" v-model.number="price" />
    </div>

    <div class="mb-3">
      <label class="form-label">Uang Muka (Rp)</label>
      <input type="number" class="form-control" v-model.number="downPayment" />
    </div>

    <div class="mb-3">
      <label class="form-label">Tenor (Bulan)</label>
      <input type="number" class="form-control" v-model.number="tenor" />
    </div>

    <div class="mb-3">
      <label class="form-label">Bunga per Tahun (%)</label>
      <input type="number" class="form-control" v-model.number="interestRate" />
    </div>

    <button class="btn btn-primary" @click="calculate">Hitung Cicilan</button>

    <div v-if="monthlyPayment" class="mt-4 alert alert-success">
      <strong>Cicilan per bulan: Rp {{ monthlyPayment.toLocaleString() }}</strong>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const price = ref(0)
const downPayment = ref(0)
const tenor = ref(12)
const interestRate = ref(5)
const monthlyPayment = ref(null)

const calculate = () => {
  const principal = price.value - downPayment.value
  const monthlyInterest = interestRate.value / 100 / 12
  if (monthlyInterest === 0) {
    monthlyPayment.value = principal / tenor.value
  } else {
    monthlyPayment.value = principal * monthlyInterest / (1 - Math.pow(1 + monthlyInterest, -tenor.value))
  }
}
</script>
