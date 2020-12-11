<template>
<div class="q-my-xl">
    <q-card>
        <q-card-title>Create new product</q-card-title>
        <q-card-main>
            <q-field :count="250">
                <q-input float-label="Category" v-model="productCategory" max-length="250" />
            </q-field>
            <q-field :count="250">
                <q-input float-label="Brand" v-model="productBrand" max-length="250" />
            </q-field>
            <q-field :count="250">
                <q-input float-label="Model" v-model="productModel" max-length="250" />
            </q-field>
            <q-field>
                <q-input float-label="Price" v-model.number="productPrice" type="number" />
            </q-field>
            <q-field :count="5000">
                <q-input
                    type="textarea"
                    float-label="Description"
                    v-model="productDescription"
                    :max-height="100"
                    rows="7"
                />
            </q-field>
        </q-card-main>
        <q-card-actions class="q-mt-md">
            <div class="row justify-end full-width docs-btn">
                <q-btn label="Cancel" flat to="/products/index"/>
                <q-btn label="Create" color="primary" @click="createProduct"/>
            </div>
        </q-card-actions>
    </q-card>
</div>
</template>

<style lang="stylus">
.docs-btn .q-btn
    padding 15px 20px
</style>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      productCategory: '',
      productBrand: '',
      productModel: '',
      productPrice: 0,
      productDescription: ''
    }
  },
  methods: {
    createProduct () {
      axios
        .post(`http://127.0.0.1:8000/api/products`, this.productData)
        .then(response => {
          this.$q.notify({ type: 'positive', timeout: 2000, message: 'The product has been created.' })
          this.$router.push({ path: '/products/' + response.data.id + '/edit' })
        })
        .catch(error => {
          this.$q.notify({ type: 'negative', timeout: 2000, message: 'An error has been occured.' })
          console.log(error)
        })
    }
  },
  computed: {
    productData: function () {
      return { category: this.productCategory, brand: this.productBrand, model: this.productModel, price: this.productPrice, description: this.productDescription }
    }
  }
}
</script>
