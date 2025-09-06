<script setup>
import { ref, computed, watch } from "vue"
import {
  CTable, CTableHead, CTableRow, CTableHeaderCell,
  CTableBody, CTableDataCell,
  CPagination, CPaginationItem,
  CFormInput, CFormLabel, CFormSelect, CButton, CListGroup, CListGroupItem
} from '@coreui/vue'
import * as XLSX from "xlsx"
import jsPDF from "jspdf"
import "jspdf-autotable"

const principal = ref(1000000000)
const tenor = ref(240)

// Fixed rate
const fixedRate = ref(5)
const fixedPeriod = ref(2)
const fixedUnit = ref("year")

// Floating rate
const floatingRates = ref([6])
const floatingUnit = ref("year")
const newFloatingRate = ref(null)

// Reset floatingRates saat unit berubah
watch(floatingUnit, () => {
  floatingRates.value = []
})

// Pagination
const currentPage = ref(1)
const perPage = ref(12)

function formatNumber(num) {
  return new Intl.NumberFormat("id-ID").format(num)
}

function addFloatingRate() {
  if (newFloatingRate.value !== null && newFloatingRate.value !== "") {
    floatingRates.value.push(parseFloat(newFloatingRate.value))
    newFloatingRate.value = null
  }
}

function removeFloatingRate(index) {
  floatingRates.value.splice(index, 1)
}

// DP
const dpNominal = ref(100000000)
const dpPercent = ref(10)

// Sinkronisasi DP
watch(dpNominal, (val) => {
  dpPercent.value = ((val / principal.value) * 100).toFixed(0)
})

watch(dpPercent, (val) => {
  dpNominal.value = ((val / 100) * principal.value).toFixed(0)
})

const effectivePrincipal = computed(() => principal.value - Number(dpNominal.value))

// Saat menghitung monthlyPayments, gunakan nilai efektif
const monthlyPayments = computed(() => {
  let payments = []
  let remaining = principal.value - dpNominal.value // Kurangi DP
  const fixedMonths = fixedUnit.value === "year" ? fixedPeriod.value * 12 : fixedPeriod.value

  for (let i = 0; i < tenor.value; i++) {
    let rate
    if (i < fixedMonths) {
      rate = fixedRate.value
    } else {
      if (floatingUnit.value === "year") {
        const yearIndex = Math.floor((i - fixedMonths) / 12)
        rate = floatingRates.value[yearIndex] ?? floatingRates.value[floatingRates.value.length - 1]
      } else {
        const monthIndex = i - fixedMonths
        rate = floatingRates.value[monthIndex] ?? floatingRates.value[floatingRates.value.length - 1]
      }
    }

    let monthlyRate = rate / 100 / 12
    let interest = remaining * monthlyRate
    let principalPay = (principal.value - dpNominal.value) / tenor.value
    let installment = interest + principalPay
    remaining -= principalPay

    payments.push({
      bulan: i + 1,
      bunga: rate,
      interest,
      angsuranPokok: principalPay,
      totalBayar: installment,
      sisaPinjaman: remaining
    })
  }
  return payments
})

const paginatedPayments = computed(() => {
  const start = (currentPage.value - 1) * perPage.value
  return monthlyPayments.value.slice(start, start + perPage.value)
})

const totalPages = computed(() => Math.ceil(monthlyPayments.value.length / perPage.value))

function goToPage(page) {
  if (page >= 1 && page <= totalPages.value) currentPage.value = page
}

const paginationPages = computed(() => {
  const total = totalPages.value
  const current = currentPage.value
  const pages = []
  if (total <= 10) {
    for (let i = 1; i <= total; i++) pages.push(i)
  } else {
    pages.push(1, 2, 3)
    if (current > 5) pages.push("...")
    const start = Math.max(4, current - 1)
    const end = Math.min(total - 3, current + 1)
    for (let i = start; i <= end; i++) pages.push(i)
    if (current < total - 4) pages.push("...")
    pages.push(total - 2, total - 1, total)
  }
  return pages
})

// ----- EXPORT FUNCTIONS -----
function exportCSV() {
  const header = ["Bulan","Bunga (%)","Bunga (Rp)","Pokok (Rp)","Total Bayar (Rp)","Sisa Pinjaman"]
  const data = monthlyPayments.value.map(p => [
    p.bulan,
    p.bunga,
    Math.round(p.interest),
    Math.round(p.angsuranPokok),
    Math.round(p.totalBayar),
    Math.round(p.sisaPinjaman)
  ])
  const csvContent = [header, ...data].map(e => e.join(",")).join("\n")
  const blob = new Blob([csvContent], { type: "text/csv;charset=utf-8;" })
  const link = document.createElement("a")
  link.href = URL.createObjectURL(blob)
  link.setAttribute("download", "simulasi_krp.csv")
  link.click()
}

function exportXLSX() {
  const ws = XLSX.utils.json_to_sheet(monthlyPayments.value.map(p => ({
    Bulan: p.bulan,
    "Bunga (%)": p.bunga,
    "Bunga (Rp)": Math.round(p.interest),
    "Pokok (Rp)": Math.round(p.angsuranPokok),
    "Total Bayar (Rp)": Math.round(p.totalBayar),
    "Sisa Pinjaman": Math.round(p.sisaPinjaman)
  })))
  const wb = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(wb, ws, "Simulasi KRP")
  XLSX.writeFile(wb, "simulasi_krp.xlsx")
}

function exportPDF() {
  const doc = new jsPDF()
  const columns = ["Bulan","Bunga (%)","Bunga (Rp)","Pokok (Rp)","Total Bayar (Rp)","Sisa Pinjaman"]
  const rows = monthlyPayments.value.map(p => [
    p.bulan,
    p.bunga,
    Math.round(p.interest),
    Math.round(p.angsuranPokok),
    Math.round(p.totalBayar),
    Math.round(p.sisaPinjaman)
  ])
  doc.autoTable({
    head: [columns],
    body: rows
  })
  doc.save("simulasi_krp.pdf")
}
</script>

<template>
  <div class="container my-4">
    <h1 class="h4 fw-bold mb-4">Simulasi KRP Fixed + Floating</h1>

    <!-- Pinjaman -->
    <div class="mb-3">
      <CFormLabel>Plafon Pinjaman</CFormLabel>
      <CFormInput v-model.number="principal" type="number" />
    </div>
    <div class="mb-3">
      <CFormLabel>Tenor (bulan)</CFormLabel>
      <CFormInput v-model.number="tenor" type="number" />
    </div>

    <!-- Tambahkan di bagian Input Pinjaman -->
    <div class="row mb-3">
    <div class="col-md-6">
      <CFormLabel>Uang Muka (DP Nominal)</CFormLabel>
      <CFormInput v-model.number="dpNominal" type="number" placeholder="Rp" />
    </div>
    <div class="col-md-6">
      <CFormLabel>Uang Muka (DP Persentase)</CFormLabel>
      <CFormInput v-model.number="dpPercent" type="number" placeholder="%" />
    </div>
  </div>

  <div class="mb-3">
    <p>Plafon efektif setelah DP: Rp {{ new Intl.NumberFormat("id-ID").format(effectivePrincipal) }}</p>
  </div>

    <!-- Fixed Rate -->
    <div class="row mb-3">
      <div class="col-md-6">
        <CFormLabel>Fixed Rate (%)</CFormLabel>
        <CFormInput v-model.number="fixedRate" type="number" />
      </div>
      <div class="col-md-3">
        <CFormLabel>Lama Fixed</CFormLabel>
        <CFormInput v-model.number="fixedPeriod" type="number" />
      </div>
      <div class="col-md-3">
        <CFormLabel>Satuan</CFormLabel>
        <CFormSelect v-model="fixedUnit">
          <option value="year">Tahun</option>
          <option value="month">Bulan</option>
        </CFormSelect>
      </div>
    </div>

    <!-- Floating Rate -->
    <div class="row mb-3">
      <div class="col-md-9">
        <CFormLabel>Floating Rates (%)</CFormLabel>
        <div class="d-flex gap-2 mb-2">
          <CFormInput
            v-model.number="newFloatingRate"
            type="number"
            placeholder="Masukkan bunga"
          />
          <CButton color="primary" @click="addFloatingRate">+</CButton>
        </div>

        <CListGroup>
          <CListGroupItem
            v-for="(rate, index) in floatingRates"
            :key="index"
            class="d-flex justify-content-between align-items-center"
          >
            <span>{{ floatingUnit === 'year' ? `Tahun ${index+1}` : `Bulan ${index+1}` }}:</span>
            <CFormInput
              v-model.number="floatingRates[index]"
              type="number"
              style="width:80px"
            />
            <CButton color="danger" size="sm" @click="removeFloatingRate(index)">❌</CButton>
          </CListGroupItem>
        </CListGroup>
      </div>

      <div class="col-md-3">
        <CFormLabel>Satuan Floating</CFormLabel>
        <CFormSelect v-model="floatingUnit">
          <option value="year">Tahun</option>
          <option value="month">Bulan</option>
        </CFormSelect>
      </div>
    </div>
    
    <!-- EXPORT BUTTONS -->
    <div class="mb-3 d-flex gap-2">
      <CButton color="success" @click="exportCSV">Export CSV</CButton>
      <CButton color="primary" @click="exportXLSX">Export XLSX</CButton>
      <CButton color="danger" @click="exportPDF">Export PDF</CButton>
    </div>

    <!-- Tabel -->
    <CTable bordered striped hover small responsive>
      <CTableHead color="light">
        <CTableRow>
          <CTableHeaderCell>Bulan</CTableHeaderCell>
          <CTableHeaderCell>Bunga (%)</CTableHeaderCell>
          <CTableHeaderCell>Bunga (Rp)</CTableHeaderCell>
          <CTableHeaderCell>Pokok (Rp)</CTableHeaderCell>
          <CTableHeaderCell>Total Bayar (Rp)</CTableHeaderCell>
          <CTableHeaderCell>Sisa Pinjaman</CTableHeaderCell>
        </CTableRow>
      </CTableHead>
      <CTableBody>
        <CTableRow v-for="p in paginatedPayments" :key="p.bulan">
          <CTableDataCell class="text-center">{{ p.bulan }}</CTableDataCell>
          <CTableDataCell class="text-center">{{ p.bunga }}</CTableDataCell>
          <CTableDataCell>{{ formatNumber(p.interest.toFixed(0)) }}</CTableDataCell>
          <CTableDataCell>{{ formatNumber(p.angsuranPokok.toFixed(0)) }}</CTableDataCell>
          <CTableDataCell>{{ formatNumber(p.totalBayar.toFixed(0)) }}</CTableDataCell>
          <CTableDataCell>{{ formatNumber(p.sisaPinjaman.toFixed(0)) }}</CTableDataCell>
        </CTableRow>
      </CTableBody>
    </CTable>

    <!-- Pagination -->
    <div class="d-flex justify-content-between align-items-center mt-3">
      <CPagination aria-label="Page navigation">
        <CPaginationItem :disabled="currentPage === 1" @click="goToPage(1)">First</CPaginationItem>
        <CPaginationItem :disabled="currentPage === 1" @click="goToPage(currentPage - 1)">Prev</CPaginationItem>

        <CPaginationItem
          v-for="page in paginationPages"
          :key="page"
          :disabled="page === '...'"
          :active="page === currentPage"
          @click="page !== '...' && goToPage(page)"
          :class="{ 'fw-bold': page === currentPage }"
        >
          {{ page }}
        </CPaginationItem>

        <CPaginationItem :disabled="currentPage === totalPages" @click="goToPage(currentPage + 1)">Next</CPaginationItem>
        <CPaginationItem :disabled="currentPage === totalPages" @click="goToPage(totalPages)">Last</CPaginationItem>
      </CPagination>

      <div class="ms-3">
        <CFormLabel class="me-2">Baris per halaman:</CFormLabel>
        <CFormSelect v-model.number="perPage" class="d-inline-block w-auto">
          <option :value="5">5</option>
          <option :value="10">10</option>
          <option :value="12">12</option>
          <option :value="20">20</option>
        </CFormSelect>
      </div>
    </div>
  </div>
</template>
