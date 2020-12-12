<template>
<div class="q-my-xl">
    <q-card>
        <q-card-title>Edit</q-card-title>
        <q-card-main>
            <div class="q-pb-md">
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
            <q-field class="q-pb-md">
                <q-input float-label="Price" v-model.number="productPrice" type="number" />
                <p v-if="errors.price"><span style="color:red">{{ errors.price[0] }}</span></p>
            </q-field>
            <q-field  class="q-pb-md" :count="500">
                <q-input
                    float-label="Description"
                    v-model="productDescription"
                    type="textarea"
                    :max-height="100"
                    rows="5"
                />
                <p v-if="errors.description"><span style="color:red">{{ errors.description[0] }}</span></p>
            </q-field>
            <q-list>
                <q-list-header>Images</q-list-header>
                    <q-item v-for="(image, index) in productImages" :key="image.id">
                        <q-item-side>
                            <!-- <q-item-tile label>{{product.url}}</q-item-tile> -->
                            <img :src= image.url style="max-height: 150px; max-width: 150px;">
                        </q-item-side>
                        <q-item-side right>
                            <q-btn @click="deleteFromList(index)" color="red" size="md" label="x" />
                        </q-item-side>

                    </q-item>
            </q-list>
            <q-field helper="Supported format: JPG, max. file size: 300KiB" class="q-mt-lg">
                <q-uploader
                    float-label="Images"
                    extensions=".jpg,.jpeg,.png"
                    url="http://127.0.0.1:8000/api/products/upload"
                    multiple
                    auto-expand
                />
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
      errors: [],
      productImages: []
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
    },
    deleteFromList (index) {
      this.productImages.splice(index, 1)
      console.log(index)
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
        this.productImages = response.data.images
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
