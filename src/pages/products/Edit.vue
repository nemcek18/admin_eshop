<template>
<div class="q-my-xl">
    <q-card>
        <q-card-title>Edit</q-card-title>
        <q-card-main>
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
                    float-label="Description"
                    v-model="productDescription"
                    type="textarea"
                    :max-height="100"
                    rows="7"
                />
            </q-field>
            <!-- <q-field>
                <q-input float-label="Price" v-model.number="productPrice" type="number" />
            </q-field> -->
        </q-card-main>
        <q-card-actions class="q-mt-md">
            <div class="row justify-end full-width docs-btn">
                <q-btn label="Cancel" flat to="/products/index"/>
                <q-btn label="Update" color="primary" @click="updateProduct"/>
            </div>
        </q-card-actions>
    </q-card>
</div>
</template>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      productBrand: '',
      productModel: '',
      productPrice: 0,
      productDescription: ''
    }
  },
  methods: {
    updateProduct () {
      axios
        .put(`http://127.0.0.1:8000/api/products/` + this.$route.params.id, this.productData)
        .then(response => {
          this.$q.notify({ type: 'positive', timeout: 2000, message: 'The product has been updated.' })
        })
        .catch(error => {
          this.$q.notify({ type: 'negative', timeout: 2000, message: 'An error has been occured.' })
          console.log(error)
        })
    }
  },
  mounted () {
    axios
      .get(`http://127.0.0.1:8000/api/products/` + this.$route.params.id + '/edit')
      .then(response => {
        this.productBrand = response.data.brand
        this.productModel = response.data.model
        this.productPrice = response.data.price
        this.productDescription = response.data.description
      })
      .catch(error => {
        this.$q.notify({ type: 'negative', timeout: 2000, message: 'Loading product: an error has been occured.' })
        console.log(error)
      })
  },
  computed: {
    productData: function () {
      return { brand: this.productBrand, model: this.productModel, price: this.productPrice, description: this.productDescription }
    }
  }
}
</script>
