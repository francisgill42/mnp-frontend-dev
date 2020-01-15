<template>
<v-app>

    <v-text-field
  v-for="(n,index) in items"
  :key="index"
  v-model="pricing.id[index]"
  @input="getVal(index)"
  color="info"
  outline
  validate-on-blur
/>
</v-app>

</template>
<script>
export default {
auth:false,
data: () => ({
     pricing:{
      id: [],
        },
order_item_id:{
    item:{
        id:''
    }
},
snackbar:false,
action:'',
search:'',
dialog: false,
headers: [
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
text: 'Customer Class',
align: 'left',
sortable: false,
value: 'customer_category_name',
},
{
text: 'Order Total',
align: 'left',
sortable: false,
value: 'order_total',
},

{
text: 'Order Status',
align: 'left',
sortable: false,
value: 'status',
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

 items:[
        {'id':1,'name':'francis'},
        {'id':2,'name':'idrees'},
        {'id':3,'name':'aizaz'},
        {'id':4,'name':'azhar'},

],
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

},  

methods: {

getVal (i) {
    console.log(this.pricing.id[i],i);
},

async initialize () {

// const orders = await this.$axios.get('order');
// this.orders = orders.data;

this.orders = [
        {'id':1,'name':'francis','customer_category_name':'A','status':'pending','order_total':'1590.75'},

]


//const order_items = await this.$axios.get('product');
//this.order_items = order_items.data;

this.order_items = [
    {
        'legacy_code_sku':'BLA002',
        'product_title':'Blackcurrant Sorbet 2.4LC  2X2400ML',
        'product_price':"175.00",
        'product_image':'https:\/\/orangeroomdigital.com\/mnp\/public\/uploads\/product_images\/21 - BLACKCURRANT (SORBET).jpg',
        'product_quantity':'5'
    }
]

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

update_password () {
this.$axios.post('status_change/' + this.editedItem.id, {pwd:this.editedItem.pwd})
.then(res =>{
this.snackbar = res.data.response_status;
this.response.msg = res.data.message;
this.close()
}).catch(error => console.log(err));
},
save () {
const payload = {
name:this.editedItem.name,
email:this.editedItem.email,
// password:this.editedItem.password,
order_category_id:this.editedItem.order_category_id,
ntn:this.editedItem.ntn,
phone_number:this.editedItem.phone_number,
mobile_number:this.editedItem.mobile_number,
address:this.editedItem.address,
state_id:this.editedItem.state_id,
cities_by_state_id:(this.editedItem.cities_by_state_id) ? this.editedItem.cities_by_state_id : this.editedItem.city_id,
IsActive:this.editedItem.IsActive ? 1 : 0,
};


this.$axios.put('order/' + this.editedItem.id,payload)
.then(res => {
if(res.data.response_status){
const index = this.orders.findIndex(item => item.id == this.editedItem.id)
this.orders.splice(index, 1, res.data.updated_record);
this.snackbar = res.data.response_status;
this.response.msg = res.data.message;
this.close()
}
else{


if(res.data.errors){
this.snackbar = true;
this.response.msg = res.data.message;
}
else{
this.errors = res.data.errors;
}

}
})
.catch(error => console.log(err));

},
},
}
</script>