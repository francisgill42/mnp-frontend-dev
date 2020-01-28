<template>
<v-app>
  <div class="text-center ma-2">

    <v-snackbar
      v-model="snackbar"
      :top="'top'"
    >
      {{response.msg}}
      <v-btn      
        text
        @click="snackbar = false"
      >
        Close
      </v-btn>
    </v-snackbar>
  </div>
  <v-data-table
    :headers="headers"
    :items="discounts"
    :search="search"
    class="elevation-1"
  >
  <template  v-slot:item.categories="{ item }">
  <v-col style="margin-left:-15px !important;" sm="6">
  <v-chip
  v-for="(category,index) in item.categories" :key="index"
  small  class="ma-1 black--text primary">
  {{category.customer_category_name}}
  </v-chip>
  </v-col>
  </template>
 
    <!-- <template v-slot:item.discount_type="{ item }">
      {{item.discount_type == 'percentage' ? '%': item.discount_type}}
    </template> -->
    <template v-slot:top>
      <v-toolbar flat color="">
        <v-toolbar-title>Discounts</v-toolbar-title>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
      
      <v-text-field
        v-model="search"
        label="Search"
        single-line
        hide-details
      ></v-text-field>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
       <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on }">
            <v-btn color="primary"  class="mb-2 black--text" v-on="on">add discount</v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-form ref="form" lazy-validation>
                <v-row>
                  <v-col>
                    <v-text-field :rules="Rules"  v-model="editedItem.discount_title" label="Discount Name"></v-text-field>
                  </v-col>
                 
                   <v-col>
                    <v-text-field :rules="Rules"  type="number" min="0" v-model.number="editedItem.discount_amount" label="Discount Amount"></v-text-field>
                  </v-col>
                 
                 
                  </v-row>
                   <v-row>
                  <v-col>
                  <v-select
                  :rules="Rules" 
                  v-model="editedItem.discount_type" 
                  :items="['percentage','amount']"
                  label="Discount Type"
                  ></v-select>
                  </v-col>
                   </v-row>
                   <v-row>
                  <v-col>
                  <v-select
                  :rules="Rules" 
                  v-model="editedItem.customer_category_ids" 
                  :items="cat_to_upload"
                  attach
                  chips
                  multiple
                  item-value="id"
                  item-text="customer_category_name" 
                  label="Customer Group"
                  ></v-select>
                  
                  </v-col>
                  </v-row>
                  </v-form>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="save">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.action="{ item }">
      <!-- <v-icon
        small
        class="mr-2"
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon> -->
      <v-icon
        small
        @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>
    </template>
    <template v-slot:no-data>
      <!-- <v-btn color="primary" @click="initialize">Reset</v-btn> -->
    </template>
  </v-data-table>
  </v-app>
</template>
<script>
  export default {
    // async fetch({store}){
    //     await store.dispatch('getRoles')
    //   },

    data: () => ({
      search:'',
      snackbar:false,
      dialog: false,
      headers: [
        {
          text: '#',
          align: 'left',
          sortable: false,
          value: 'id',
        },
         {
          text: 'Discount Name',
          align: 'left',
          sortable: false,
          value: 'discount_title',
        },
        {
          text: 'Discount',
          align: 'left',
          sortable: false,
          value: 'discount_amount',
        },
          {
          text: 'Discount Type',
          align: 'left',
          sortable: false,
          value: 'discount_type',
        },
          {
          text: 'Category',
          align: 'left',
          sortable: false,
          value: 'categories',
        },
        { text: 'Actions', value: 'action', sortable: false },
      ],
      editedIndex: -1,
      editedItem: {
        discount_title: '',
        discount_amount:'',
        discount_type:'',
        customer_category_ids:[],  
       
      },
      defaultItem: {
        discount_title: '',
        discount_amount:'',
        discount_type:'',
        customer_category_ids:[],
      
      },
      Rules : [
      v => !!v || 'This field is required',
      ],
      response : {
        msg:''
      },
      discounts:[],
      cat_to_upload:[]
    }),

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      },
      // discounts(){
        
      //   return this.$store.state.discounts

      // }
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
    },

    async created () {
     const discounts = await this.$axios.get('discount');
     this.discounts = discounts.data;

     console.log(this.discounts);

     const customer_categories = await this.$axios.get('customer_group');
     this.cat_to_upload = customer_categories.data;
            
    },

    methods: {

      editItem (item) {
          this.editedIndex = this.discounts.indexOf(item)
          this.editedItem = Object.assign({}, item)
          this.dialog = true
          
      },

      deleteItem (item) {
        confirm('Are you sure you want to delete this item?') && 
         this.$axios.delete('discount/'+item.id)
            .then((res) => {

              

              if(res.data.response_status){ 

              const index = this.discounts.indexOf(item)
              this.discounts.splice(index, 1)
              this.snackbar = res.data.response_status;             
              this.response.msg = res.data.message;

              }
            });
      },

      close () {
        this.dialog = false
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        }, 300)
      },

      save () {


//        console.log(this.editedItem.customer_category_ids);
        const payload = {
            discount_title: this.editedItem.discount_title,
            discount_amount: this.editedItem.discount_amount,
            discount_type: this.editedItem.discount_type,
            customer_category_ids: this.editedItem.customer_category_ids
           
            }

           if (this.editedIndex > -1) {

            this.$axios.put('discount/' + this.editedItem.id,payload)
            .then(res => {
              if(res.data.response_status){ 
              const index = this.discounts.findIndex(item => item.id == this.editedItem.id)
              Object.assign(this.discounts[index],res.data.updated_record);
              this.snackbar = res.data.response_status;
              this.response.msg = res.data.message;
              this.close()
              }
            })
            .catch(error => console.log(err));
           }
           else{
          if(this.$refs.form.validate()){              
            this.$axios.post('discount',payload)
              .then((res) => {

              if(res.data.response_status){ 

              this.discounts.push(res.data.new_record)
              this.snackbar = res.data.response_status;
              this.response.msg = res.data.message;

              this.close()
              
              }

            });
            }
            }
        
      },
    },
  }
</script>