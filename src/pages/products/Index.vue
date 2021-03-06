<template>
<div class="q-my-xl">
<q-table
      :data="serverData"
      row-key="name"
      :pagination.sync="serverPagination"
      :loading="loading"
      @request="request"
      :columns="columns"
      title="List of products"
      binary-state-sort
      >
      <q-tr slot="body" slot-scope="props" :props="props">
        <q-td key="product_id" :props="props">
          <span>{{ props.row.product_id }}</span>
        </q-td>
        <q-td key="name" :props="props">
          <span>{{ props.row.name }}</span>
        </q-td>
        <q-td key="brand" :props="props">
          <span>{{ props.row.brand }}</span>
        </q-td>
        <q-td key="model" :props="props">
          <span>{{ props.row.model }}</span>
        </q-td>
        <q-td key="price" :props="props">
          <span>{{ props.row.price }}</span>
        </q-td>
        <q-td class="text-right">
          <div v-if="props.row.product_id == 'DELETED'">DELETED</div>
          <div v-else>
            <q-btn round icon="edit" class="q-mr-xs" @click="$router.push('/products/' + props.row.product_id + '/edit')" />
            <q-btn round icon="delete" @click="destroy(props.row.product_id, props.row.brand, props.row.name, props.row.model, props.row.__index)"/>
          </div>
        </q-td>
      </q-tr>
    </q-table>
</div>
</template>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      columns: [
        { name: 'product_id', label: 'ID', field: 'product_id', sortable: true, align: 'left' },
        { name: 'name', label: 'Category', field: 'name', sortable: true, align: 'left' },
        { name: 'brand', label: 'Brand', field: 'brand', sortable: true, align: 'left' },
        { name: 'model', label: 'Model', field: 'model', sortable: true, align: 'left' },
        { name: 'price', label: 'Price', field: 'price', sortable: true, align: 'left' },
        { name: 'actions', label: 'Actions', sortable: false, align: 'right' }
      ],
      selected: [],
      loading: false,
      serverPagination: {
        page: 1,
        rowsNumber: 1000, // the number of total rows in DB
        rowsPerPage: 10,
        sortBy: 'product_id',
        descending: false
      },
      serverData: []
    }
  },
  methods: {
    request ({ pagination }) {
      // QTable to "loading" state
      this.loading = true
      // fetch data
      axios
        .get(`http://127.0.0.1:8000/api/products/list/${pagination.page}?rowsPerPage=${pagination.rowsPerPage}&sortBy=${pagination.sortBy}&descending=${pagination.descending}`)
        .then(({ data }) => {
          // updating pagination to reflect in the UI
          this.serverPagination = pagination

          // we also set (or update) rowsNumber
          this.serverPagination.rowsNumber = data.rowsNumber

          // then we update the rows with the fetched ones
          this.serverData = data.rows

          // finally we tell QTable to exit the "loading" state
          this.loading = false
        })
        .catch(error => {
          // there's an error... do SOMETHING
          console.log(error)

          // we tell QTable to exit the "loading" state
          this.loading = false
        })
    },

    destroy (productId, name, brand, model, rowIndex) {
      this.$q.dialog({
        title: 'Delete',
        message: 'Are you sure to delete ' + brand + ' ' + model + '?',
        color: 'primary',
        ok: true,
        cancel: true
      }).then(() => {
        axios
          .delete(`http://127.0.0.1:8000/api/products/${productId}`)
          .then(() => {
            // this.serverData[rowIndex].id = 'DELETED'
            this.request({
              pagination: this.serverPagination,
              filter: this.filter
            })
            this.$q.notify({ type: 'positive', timeout: 2000, message: 'The product has been deleted.' })
          })
          .catch(error => {
            this.$q.notify({ type: 'negative', timeout: 2000, message: 'An error has been occured.' })
            console.log(error)
          })
      }).catch(() => {
        // cancel
      })
    }
  },
  mounted () {
    // once mounted, we need to trigger the initial server data fetch
    this.request({
      pagination: this.serverPagination,
      filter: this.filter
    })
  }
}
</script>
