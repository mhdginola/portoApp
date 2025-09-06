<script setup>
import { ref, computed } from 'vue'

const Product = ref({
    productName: '',
    productPrice: 0,
    productSize: 1,
    productRepackSize: 1,
    margin: 0,
});

const productPrice1Grams = computed(() => {
    return Product.value.productSize > 0
        ? Product.value.productPrice / Product.value.productSize
        : 0
})

const productRepackQuantity = computed(() => {
    return Product.value.productRepackSize > 0
        ? Product.value.productSize / Product.value.productRepackSize
        : 0
})

const modal = computed(() => {
    return Product.value.productRepackSize * productPrice1Grams.value;
})

const productPriceMargin = computed(() => {
    return (Product.value.margin / 100) * modal.value + modal.value;
})

const total = computed(() => {
    return productPriceMargin.value * productRepackQuantity.value;
})

const profit = computed(() => {
    return total.value - Product.value.productPrice;
})
</script>
<template>
    <CCol :xs="12">
        <CCard class="mb-4">
            <CCardHeader>
                <strong>Margin Form</strong>
            </CCardHeader>
            <CCardBody>
                <CForm>
                    <div class="mb-3">
                        <CFormLabel for="productName">Product Name</CFormLabel>
                        <CFormInput id="productName" v-model="Product.productName" type="text" placeholder="Product Name" />
                    </div>
                    <div class="mb-3">
                        <CFormLabel for="productPrice">Product Price</CFormLabel>
                        <CFormInput id="productPrice" v-model="Product.productPrice" type="number"
                            placeholder="Product Price" />
                    </div>
                    <div class="mb-3">
                        <CFormLabel for="productSize">Product Size (grams)</CFormLabel>
                        <CFormInput id="productSize" v-model="Product.productSize" type="number"
                            placeholder="Product Size" />
                    </div>
                    <div class="mb-3">
                        <CFormLabel for="productPrice1Grams">Product Price (1 grams)</CFormLabel>
                        <CFormInput id="productPrice1Grams" v-model="productPrice1Grams" type="number"
                            placeholder="Product Price 1 (grams)" :disabled="true" />
                    </div>
                    <div class="mb-3">
                        <CFormLabel for="productRepackSize">Product Repack Size (grams)</CFormLabel>
                        <CFormInput id="productRepackSize" v-model="Product.productRepackSize" type="number"
                            placeholder="Product Repack Size" />
                    </div>
                    <div class="mb-3">
                        <CFormLabel for="productRepackQuantity">Product Repack Quantity</CFormLabel>
                        <CFormInput id="productRepackQuantity" v-model="productRepackQuantity" type="number"
                            placeholder="Product Repack Quantity" :disabled="true" />
                    </div>
                    <div class="mb-3">
                        <CFormLabel for="modal">modal</CFormLabel>
                        <CFormInput id="modal" v-model="modal" type="number" placeholder="Modal" :disabled="true" />
                    </div>
                    <div class="mb-3">
                        <CFormLabel for="margin">Margin (%)</CFormLabel>
                        <CFormInput id="margin" v-model="Product.margin" type="number" placeholder="Margin (%)" />
                    </div>
                    <div class="mb-3">
                        <CFormLabel for="productPriceMargin">Product Price</CFormLabel>
                        <CFormInput id="productPriceMargin" v-model="productPriceMargin" type="number" placeholder="Product Price Margin"
                            :disabled="true" />
                    </div>
                    <div class="mb-3">
                        <CFormLabel for="total">Total</CFormLabel>
                        <CFormInput id="total" v-model="total" type="number" placeholder="Total"
                            :disabled="true" />
                    </div>
                    <div class="mb-3">
                        <CFormLabel for="profit">Profit</CFormLabel>
                        <CFormInput id="profit" v-model="profit" type="number" placeholder="Profit"
                            :disabled="true" />
                    </div>
                </CForm>
            </CCardBody>
        </CCard>
    </CCol>
</template>
