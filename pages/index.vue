<template>
<div>

<v-row>
  <v-col>
        
    <v-card color="#1F7087">
      
    <v-list-item three-line>
      <v-list-item-content>
        <div class="overline mb-4 white--text">Orders</div>
        <v-list-item-title color="pink darken-2" class="white--text headline mb-1">Total </v-list-item-title>
        <v-list-item-subtitle>
          </v-list-item-subtitle>
      </v-list-item-content>

      <v-list-item-avatar
        tile
        size="50"
        color="#385F73"
        >
       <span  class="white--text">{{orders}}</span>
        </v-list-item-avatar>
    </v-list-item>


    
  </v-card>
  </v-col>


  <v-col>
    <v-card color="#1F7087">
    <v-list-item three-line>
      <v-list-item-content>
        <div class="overline mb-4 white--text">Products</div>
        <v-list-item-title class="headline mb-1 white--text">Total </v-list-item-title>
        <v-list-item-subtitle>
          </v-list-item-subtitle>
      </v-list-item-content>

      <v-list-item-avatar
        tile
        size="50"
        color="#385F73"
        >
       <span class="white--text">{{products}}</span>
        </v-list-item-avatar>
    </v-list-item>


    
  </v-card>
  </v-col>

    <v-col>
      <v-card color="#1F7087">
    <v-list-item three-line>
      <v-list-item-content>
        <div class="overline mb-4 white--text">Customers</div>
        <v-list-item-title class="headline mb-1 white--text">Total </v-list-item-title>
        <v-list-item-subtitle>
          </v-list-item-subtitle>
      </v-list-item-content>

      <v-list-item-avatar
        tile
        
        size="50"
        color="#385F73"
        >
       <span  class="white--text">{{customers}}</span>
        </v-list-item-avatar>
    </v-list-item>


    
  </v-card>
  </v-col>
</v-row>


  </div>
</template>

<script>
export default{
  data(){
    return {
      orders:'',
      products:'',
      customers:'',
    }
  },
created () {
      this.initialize()
    },
    methods: {

      async initialize () {

          const counters = await this.$axios.get('counters');
          console.log(counters.data.orders);
          
          this.orders = counters.data.orders;
          this.products = counters.data.products;
          this.customers = counters.data.customers;
          


      },
   }
   }
</script>