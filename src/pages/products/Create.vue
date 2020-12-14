<template>
<div class="q-my-xl">
    <q-card>
        <q-card-title>Create new product</q-card-title>
        <q-card-main>
            <div class="q-pb-md">
                <q-select
                    v-model="select"
                    float-label="Category"
                    :options="selectOptions"
                />
                <p v-if="errors.name"><span style="color:red">{{ errors.name[0] }}</span></p>
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
            <q-field class="q-pb-md" :count="500">
                <q-input
                    type="textarea"
                    float-label="Description"
                    v-model="productDescription"
                    :max-height="100"
                    rows="5"
                />
                <p v-if="errors.description"><span style="color:red">{{ errors.description[0] }}</span></p>
            </q-field>
                        <q-list>
                <q-list-header>Main image</q-list-header>
                    <q-item v-for="(image, index) in uploadedImages" :key="image">
                        <q-item-side>
                            <img :src= image style="max-height: 150px; max-width: 150px;">
                        </q-item-side>
                        <q-item-main />
                        <q-item-side right>
                            <q-btn @click="deleteMain(index)" color="red" size="md" label="x" />
                        </q-item-side>

                    </q-item>
            </q-list>
            <p v-if="errors.images"><span style="color:red">{{ errors.images[0] }}</span></p>
            <div id="separator"></div>
            <q-list>
                <q-list-header>Gallery images</q-list-header>
                    <q-item v-for="(image, index) in galleryUpload" :key="image">
                        <q-item-side>
                            <img :src= image style="max-height: 150px; max-width: 150px;">
                        </q-item-side>
                        <q-item-main />
                        <q-item-side right>
                            <q-btn @click="deleteGallery(index)" color="red" size="md" label="x" />
                        </q-item-side>

                    </q-item>
            </q-list>
            <p v-if="errors.gallery_images"><span style="color:red">{{ errors.gallery_images[0] }}</span></p>
            <p class="caption" id=first_caption>toggle OFF: image will be uploaded as main image</p>
            <p class="caption">toggle ON: image will be uploaded as gallery image</p>
            <q-toggle v-model="type_gallery" color="dark" />
            <q-field helper="Supported format: JPG, max. file size: 300KiB" class="q-mt-lg">
                <q-uploader
                    float-label="Images"
                    extensions=".jpg,.jpeg,.png"
                    :upload-factory="uploadFile"
                    ref="uploader"
                    url=""
                    multiple
                    auto-expand
                    stack-label="upload image"
                />
            </q-field>
        </q-card-main>
        <q-card-actions class="q-mt-md">
            <div class="row justify-end full-width docs-btn">
                <q-btn label="Cancel" flat to="/products/index" @click="cancel"/>
                <q-btn label="Create" color="primary" @click="createProduct"/>
            </div>
        </q-card-actions>
    </q-card>
</div>
</template>

<style scoped>
#separator {
  padding-top: 16px;
  padding-bottom: 16px;
}
#first_caption {
  padding-top: 40px;
}
.caption {
  margin: 0;
}
</style>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      productBrand: '',
      productModel: '',
      productPrice: '',
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
      productImages: [],
      removedImages: [],
      uploadedImages: [],
      galleryImages: [],
      galleryRemove: [],
      galleryUpload: [],
      type_gallery: true,
      type_main: false
    }
  },
  watch: {
    type_gallery (val) {
      if (val) {
        this.type_main = false
      }
    },
    type_main (val) {
      if (val) {
        this.type_gallery = false
      }
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
          this.errors = error.response.data.errors
          this.$q.notify({ type: 'negative', timeout: 2000, message: 'The product has not been created.' })
        })
    },
    uploadFile (file, updateProgress) {
      const fd = new FormData()
      fd.append('file', file)
      fd.append('type_gallery', this.type_gallery)
      updateProgress(0)
      return new Promise((resolve, reject) => {
        this.$axios.post('http://127.0.0.1:8000/api/products/upload',
          fd,
          {
            headers: { 'Content-Type': 'multipart/form-data' },
            onUploadProgress: (progressEvent) => {
              updateProgress(Math.round((progressEvent.loaded / progressEvent.total) * 100) / 100)
            }
          })
          .then(response => {
            if (response.data.type_gallery === 'true') {
              this.galleryUpload.push(response.data.path)
            } else {
              this.uploadedImages.push(response.data.path)
            }
            resolve(file)
            this.$refs.uploader.reset()
          })
          .catch(error => reject(error))
      })
    },
    deleteGallery (index) {
      var dataToRemove = { images: [this.galleryUpload[index]] }
      this.galleryUpload.splice(index, 1)
      axios
        .post(`http://127.0.0.1:8000/api/products/remove`, dataToRemove)
        .then(response => {
          this.$q.notify({ type: 'positive', timeout: 2000, message: 'Image was removed from gallery folder' })
        })
        .catch(error => {
          console.error(error)
        })
    },
    deleteMain (index) {
      var dataToRemove = { images: [this.uploadedImages[index]] }
      this.uploadedImages.splice(index, 1)
      axios
        .post(`http://127.0.0.1:8000/api/products/remove`, dataToRemove)
        .then(response => {
          this.$q.notify({ type: 'positive', timeout: 2000, message: 'Image was removed main folder' })
        })
        .catch(error => {
          console.error(error)
        })
    },
    cancel () {
      if (this.uploadedImages.length > 0 || this.galleryUpload.length > 0) {
        var temp = this.uploadedImages.concat(this.galleryUpload)
        var dataToRemove = { images: temp }
        axios
          .post(`http://127.0.0.1:8000/api/products/remove`, dataToRemove)
          .then(response => {
            // this.$q.notify({ type: 'negative', timeout: 2000, message: 'Product has not been updated.' })
          })
          .catch(error => {
            console.error(error)
          })
      }
      this.$q.notify({ type: 'warning', timeout: 2000, message: 'Product has not been created.' })
    }
  },
  beforeRouteLeave (to, from, next) {
    this.cancel()
    next()
  },
  computed: {
    productData: function () {
      return { name: this.select,
        brand: this.productBrand,
        model: this.productModel,
        price: this.productPrice,
        description: this.productDescription,
        uploadedImages: this.uploadedImages,
        images: this.productImages.length + this.uploadedImages.length,
        galleryUpload: this.galleryUpload,
        gallery_images: this.galleryImages.length + this.galleryUpload.length
      }
    }
  }
}
</script>
