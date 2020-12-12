<template>
<div class="q-my-xl">
    <q-card>
        <q-card-title>Edit</q-card-title>
        <q-card-main>
            <div>
                <q-select
                    v-model="select"
                    float-label="Category"
                    :options="selectOptions"
                />
            </div>
            <q-field :count="50">
                <q-input float-label="Brand" v-model="productBrand" max-length="50" />
                <p v-if="errors.brand"><span style="color:red">{{ errors.brand[0] }}</span></p>
            </q-field>
            <q-field :count="50">
                <q-input float-label="Model" v-model="productModel" max-length="50" />
                <p v-if="errors.model"><span style="color:red">{{ errors.model[0] }}</span></p>
            </q-field>
            <q-field>
                <q-input float-label="Price" v-model.number="productPrice" type="number" />
                <p v-if="errors.price"><span style="color:red">{{ errors.price[0] }}</span></p>
            </q-field>
            <q-field :count="500">
                <q-input
                    float-label="Description"
                    v-model="productDescription"
                    type="textarea"
                    :max-height="100"
                    rows="5"
                />
                <p v-if="errors.description"><span style="color:red">{{ errors.description[0] }}</span></p>
            </q-field>
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
      productDescription: '',
      selectOptions: [
        {
          label: 'tablets',
          value: 'tablets'
        },
        {
          label: 'mobiles',
          value: 'mobiles'
        },
        {
          label: 'watches',
          value: 'watches'
        },
        {
          label: 'televisions',
          value: 'televisions'
        },
        {
          label: 'computers',
          value: 'computers'
        },
        {
          label: 'laptops',
          value: 'laptops'
        }
      ],
      select: '',
      errors: []
    }
  },
  methods: {
    updateProduct () {
      axios
        .put(`http://127.0.0.1:8000/api/products/` + this.$route.params.id, this.productData)
        .then(response => {
          this.$q.notify({ type: 'positive', timeout: 2000, message: 'The product has been updated.' })
          this.errors = []
        })
        .catch(error => {
          console.error(error)
          this.errors = error.response.data.errors
        })
    }
  },
  mounted () {
    axios
      .get(`http://127.0.0.1:8000/api/products/` + this.$route.params.id + '/edit')
      .then(response => {
        this.select = response.data.name
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
      return { name: this.select, brand: this.productBrand, model: this.productModel, price: this.productPrice, description: this.productDescription }
    }
  }
}
</script>
