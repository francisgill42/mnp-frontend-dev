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
:items="orders"
:search="search"
class="elevation-1"
:options.sync="options"
:server-items-length="data.total"
@pagination="paginate"
:items-per-page=10
:footer-props="{
    itemsPerPageOptions:[5,10,15]
}"
>
<template  v-slot:item.status="{ item }">
<v-chip small class="white--text" :class="myBtnClass(item.status)">
{{item.status}}
</v-chip>
</template>

<template v-slot:top>
<v-toolbar flat color="">
<v-toolbar-title dark>Orders</v-toolbar-title>
<v-divider
class="mx-4"
inset
vertical
></v-divider>



<v-text-field
v-model.lazy="search"
@input="searchIt"
label="Search"
single-line
hide-details
></v-text-field>
<v-divider
class="mx-4"
inset
vertical
></v-divider>

<v-dialog v-model="dialog" max-width="1200px">
<!-- <template v-slot:activator="{ on }">
<v-btn color="primary" to="order/create" class="black--text mb-2">New Item</v-btn>
</template> -->
<v-card>
<!-- <v-card-title>

<span class="headline">{{ formTitle }}</span> -->




<v-card-text>
<v-container>
<v-row>
<v-col>
<v-toolbar
class="primary mb-2 black--text"
dark
flat
>
<v-toolbar-title>{{ editedItem.name }}</v-toolbar-title>
</v-toolbar>
</v-col>


</v-row>


<v-row>
<v-col> 
<template>
<v-simple-table dense>
<template v-slot:default>
<thead>
<tr>
<th class="text-left">Id</th>
<th class="text-left">Item Code</th>
<th class="text-left">Title</th>
<th class="text-left">Image</th>
<th class="text-center">Quantity</th>
<th class="text-left">Unit Price</th>
<th class="text-left">Price</th>
<th class="text-left" v-if="!isReadOnly">Change Product</th>
<th class="text-left" v-if="!isReadOnly">Change Quantity</th>
<th v-if="!isReadOnly">Action</th>
</tr>
</thead>
<tbody>
        
<tr v-for="(item,index) in editedItem.products" :key="index">

<td>{{ item.id }}</td>
<td>{{ item.legacy_code_sku }}</td>
<td>{{ item.product_title }}</td>
<td height="25" width="15%"><img style="padding-top:5px;" width="30%" :src="item.product_image" alt="Orange Room Digital"/></td>
<td width="14%" class="text-center">{{ item.product_quantity }} 
    <span 
    v-if="editedItem.order_status_id == 1" 
    :class="get_stock_info(item.stock).class"
    >
    {{get_stock_info(item.stock).text}}
    {{item.stock > 0 ? '(' + item.stock + ')' : '' }} 
     
    </span>  
    
</td>
    
<td>AED {{ item.product_price }}</td>
<!-- <td>AED {{ item.product_quantity * item.product_price }}</td> -->

<td>AED {{sum_of_product_price(item.product_quantity , item.product_price)}}</td>

<td v-if="!isReadOnly">

<v-select
class="mb-2"
v-model="order_item.id[index]"

:items="order_items"
item-value="id"
item-text="product_title" 
label="Products"
></v-select>

</td>
<td v-if="!isReadOnly"><v-text-field  type="number"  v-model="order_item_quantity[index]"></v-text-field>
</td>
<td v-if="!isReadOnly">
     <v-btn class="primary black--text" @click="get_product(index,item.order_item_id)" fab x-small dark>
              <v-icon>mdi-content-save</v-icon>
    </v-btn>

</td>
    
</tr>



</tbody>

</template>
</v-simple-table>
</template>
</v-col>

</v-row>
<v-divider></v-divider>
<v-row>
    <v-form ref="form" lazy-validation>
<v-col>
<v-autocomplete

class="mb-2"
:rules="Rules"
v-if="!isReadOnly"
v-model="editedItem.driver_id" 
:items="drivers"
item-value="id"
item-text="name" 
label="Driver"
></v-autocomplete>

<v-btn class="primary black--text" text @click="close">Cancel</v-btn>
&nbsp;
<v-btn v-if="!isReadOnly" class="primary black--text" text @click="save">Save</v-btn>

</v-col>
</v-form>

<v-col>
    <v-spacer></v-spacer>
    </v-col>
    <v-col> 
<template>
<v-simple-table dense>
<template v-slot:default>
<tbody>
<tr>
<th style="padding-left:75px; width:60%;">Gross Amount</th>
<td style="border:none;">AED {{ get_decimal_value(editedItem.order_gross) }}</td>
</tr>    
<tr>
<th style="padding-left:75px; width:60%;">Tax (VAT %5)</th>
<td style="border:;">AED {{ get_decimal_value(editedItem.order_tax) }}</td>
</tr>
<tr>
<th style="padding-left:75px; width:60%;">Grand Total</th>
<th>AED {{get_decimal_value(editedItem.order_total)}}  </th>
</tr>


</tbody>

</template>
</v-simple-table>
</template>
</v-col>
</v-row>



</v-container>
</v-card-text>

</v-card>
</v-dialog>


</v-toolbar>
</template>
<template v-slot:item.indicator="{ item }">
    
    <!-- <v-icon small  v-if="item.order_status_id == 1 && 
     (item.products[0].stock == 0 || 
     item.products[0].stock == 'Stock does not exist')" 
     class="error"
     >mdi-checkbox-blank-circle</v-icon> -->

</template>
<template v-slot:item.order_total="{ item }">
    {{get_decimal_value(item.order_total)}}  
</template>

<template v-slot:item.action="{ item }">
<v-icon
small
class="mr-2"
@click="viewItem(item)"
>
mdi-eye
</v-icon>
<v-icon
v-if="item.order_status_id == 1 || item.order_status_id == 2"
small
class="mr-2"
@click="editItem(item)"
>
mdi-pencil
</v-icon>
<!-- {{item}} -->

<!-- <v-icon
small
@click="deleteItem(item)"
>
mdi-delete
</v-icon> -->
</template>

</v-data-table>
</v-app>
</template>
<script>
export default {

data: () => ({
order_item:{
    id:[]
},
order_item_quantity:[],
options:{
    sortBy:['id','order_total','created_at'],
    sortDesc:[true]
},
snackbar:false,
action:'',
search:'',
dialog: false,
headers: [

{
align: 'left',
value: 'indicator',
sortable: false,
},

{
text: 'Order #',
align: 'left',
value: 'id',
},

{
text: 'Customer Name',
align: 'left',
sortable: false,
value: 'name',
},
{
text: 'Mobile Number',
align: 'left',
sortable: false,
value: 'mobile_number',
},

{
text: 'Customer Class',
align: 'left',
sortable: false,
value: 'customer_category_name',   
},
{
text: 'Order Total',
align: 'left',
value: 'order_total',
},

{
text: 'Order Status',
align: 'left',
sortable: false,
value: 'status',
},
{
text: 'Order DateTime',
align: 'left',
value: 'created_at',
},
{ text: 'Actions', value: 'action', sortable: false },
],
emailRules: [
v => /.+@.+/.test(v) || 'E-mail must be valid',
],
msg:"",
errors:[
{password:''},
{email:''}
],

snackbar:false,
Rules : [
v => !!v || 'This field is required',
],

 drivers:[],
order_items:[],
editedIndex: -1,
editedItem: {
    products:[],
    driver_id:'',
    product_id:'',

},
defaultItem: {
    products:[],
    driver_id:'',
    product_id:'',
},
response : {
msg:''
},
orders:[],
data:[],
change_product:[],



}),

computed: {

isReadOnly(){
return this.action == 'View Item'
},
formTitle () {
return (this.action) ? this.action :this.editedIndex === -1 ? 'New Item' : 'Edit Item'
},

},

watch: {
dialog (val) {
val || this.close()
},
},

created () {
this.initialize()
this.myBtnClass()

},  


methods: {
    
searchIt(){

        if(this.search.length){
            this.$axios.get('search_order/'+this.search)
            .then(res => {
                this.orders = res.data.orders;            
                this.data = res.data;
            });
        }
        else{

             this.$axios.get('order')
            .then(res => {
                this.orders = res.data.orders;            
                this.data = res.data;
            });

        }





},    
get_stock_info (e) {
   switch(e) {
    case 'Stock does not exist':
      return {class:'stock_does_not_exist',text:'Stock does not exist'}
    case 0:
      return {class:'out_of_stock',text:'out of stock'}
    default:
      return {class:'in_stock',text:'in stock'}
  
} 
},    
myBtnClass(name) {
  switch(name) {

  case 'pending':
      return 'warning darken-3'
  case 'processing':
      return 'primary'
  case 'loaded':
      return 'teal'
  case 'on the way':
      return 'blue lighten-1'
  case 'delivered':
      return 'green lighten-1'
  default:
      return 'error'
  
}
    
}, 
    

get_product (i,order_item_id) {


var payload = {
    order_id : this.editedItem.id,
    order_item_id : order_item_id,
    product_id : this.order_item.id[i],
    item_quantity : this.order_item_quantity[i]
};
//console.log(payload)

this.$axios.post('change_order_item',payload)
.then(res =>{

console.log(res.data);

this.editedItem.order_gross = this.get_decimal_value(res.data.updated_record.order_gross)
this.editedItem.order_tax = this.get_decimal_value(res.data.updated_record.order_tax)
this.editedItem.order_total = this.orders[this.editedIndex].order_total = this.get_decimal_value(res.data.updated_record.order_total)
Object.assign(this.editedItem.products[i],res.data.updated_record.products)

this.snackbar = res.data.response_status;
this.response.msg = res.data.message;
this.order_item.id[i] = ''
this.order_item_quantity[i] = ''

}).catch(error => console.log(error));

},

sum_of_product_price (a,b) {
    return this.get_decimal_value(a * b)    
},
get_decimal_value (value) {
return (Math.round(value * 100) / 100).toFixed(2);
},


async paginate (e) {
const orderBy =  this.options.sortBy.length == 0 ? 'id' : this.options.sortBy[0];
const sortBy =  this.options.sortDesc.length > 0 || this.options.sortDesc[0]    
                                                            ? 'asc' : 'desc';

let  data = await this.$axios.get('order?page='+e.page,{params:{
    'per_page':e.itemsPerPage,
    'sort_by':sortBy,
    'order_by':orderBy}});
this.orders = data.data.orders;
this.data = data.data;


},
async initialize () {




const drivers = await this.$axios.get('driver');
this.drivers = drivers.data;

const order_items = await this.$axios.get('product');
this.order_items = order_items.data;


},

editItem (item) {

this.editedIndex = this.orders.indexOf(item)
this.editedItem = Object.assign({}, item)
this.dialog = true
this.action = 'Edit Item';
},

viewItem (item) {
this.editedIndex = this.orders.indexOf(item)
this.editedItem = Object.assign({}, item)
this.dialog = true
this.action = 'View Item';
},


deleteItem (item) {
confirm('Are you sure you want to delete this item?') &&
this.$axios.delete('order/'+item.id)
.then((res) => {
if(res.data.response_status){
const index = this.orders.indexOf(item)
this.orders.splice(index, 1)
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

const stock_check = this.editedItem.products.filter((v => v.stock == 0 || v.stock == 'Stock does not exist'));
console.log(stock_check ? true : false );


if(this.$refs.form.validate() && stock_check == false){

const payload = {
driver_id : this.editedItem.driver_id,
order_id : this.editedItem.id,
status_id : 2
};

this.$axios.post('status_change',payload)
.then(res => {

this.orders[this.editedIndex].status = res.data.updated_record.status

this.snackbar = res.data.response_status;
this.response.msg = res.data.message;
this.close()

})
.catch(error => console.log(error));
}
else{
this.snackbar = true;
this.response.msg = 'Kindly adjust your stock';
}
},
},
}
</script>
<style>
.indicator{
    margin-left: -24%; padding:0;
    background: red !important;
    width: 3px;
}
.stock_does_not_exist{
    font-size: 11px;
    display: block;
    background: #fbdede;
    color: #cc0000;
    border-radius: 20px;
    /* width: 120px; */
    text-align: center;
}
.out_of_stock{
    font-size: 11px;
    display: block;
    background: #fbdede;
    color: #cc0000;
    border-radius: 20px;
    /* width: 120px; */
    text-align: center;
}
.in_stock{
    font-size: 11px;
    display: block;
    background: #ddf9be;
    color: #207910;
    border-radius: 20px;
    text-align: center;
}
</style>