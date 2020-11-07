<template>
<div>
  <v-card>
    <!-- 卡片的头部 -->
    <v-card-title>
      <v-btn color="primary" @click="addGoods">新增商品</v-btn>
      <v-spacer></v-spacer>
      状态：
      <v-btn-toggle v-model="fielter.saleable">
        <v-btn flat>全部</v-btn>
        <v-btn flat :value="true">上架</v-btn>
        <v-btn flat :value="false">下架</v-btn>
      </v-btn-toggle>
      &nbsp; &nbsp; &nbsp; &nbsp;
      <v-text-field 
        label="请输入关键字搜索" 
        v-model="fielter.search" 
        append-icon="search"
        hide-details
        @keyup.enter="find">
      </v-text-field>
    </v-card-title>
    <!-- 分割线 -->
    <v-divider/>
    <v-data-table
      :headers="headers"
      :items="goodsList"
      :pagination.sync="pagination"
      :total-items="totalGoods"
      :loading="loading"
      class="elevation-1"
    >
      <template slot="items" slot-scope="props">
        <td>{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.title }}</td>
        <td class="text-xs-center">
          {{props.item.cname}}
        </td>
        <td class="text-xs-center">{{ props.item.bname }}</td>
        <td class="text-xs-center">
          <v-btn flat icon color="blue"><v-icon middle @click="editGoods(props.item)">edit</v-icon></v-btn>
          <v-btn flat icon color="red"><v-icon middle @click="deleteGoods(props.item)">delete</v-icon></v-btn>
          <v-btn flat icon ><span>下架</span></v-btn>
        </td>
      </template>
    </v-data-table>
  </v-card>
  <v-dialog v-model="show" @click="addGoods" max-width="700" persistent scrollable>
    <v-card>
      <v-toolbar dark color="blue" >
        <v-toolbar-title>{{isEdit ? '修改' : '新增'}}品牌</v-toolbar-title>
        <v-spacer></v-spacer>
        <v-btn icon @click="show=false"><v-icon>close</v-icon></v-btn>
      </v-toolbar>
      <v-card-text class="px-7" style="height:600px">
          <GoodsForm :step="step"></GoodsForm>
        </v-card-text>
        <v-card-actions class="elevation-10">
          <v-spacer></v-spacer>
          <v-btn  color="primary" @click="prev" :disabled="step === 1">上一步</v-btn>
          <v-btn  color="primary" @click="next" :disabled="step === 4">下一步</v-btn>
          <v-spacer></v-spacer>
        </v-card-actions>
    </v-card>
  </v-dialog>
  </div>
</template>

<script>
 import GoodsForm from './GoodsForm'
  export default {
    data() {
      return {
        headers: [
          // text: 文本信息 value: 字段名 sortable: 是否允许排序, 
          {text: 'id', value: 'id', align: 'center'},
          {text: '标题', value: 'title', align: 'center'},
          {text: '商品分类', value: 'cname', align: 'center'},
          {text: '品牌', value: 'bname', align: 'center'},
          {text: '操作', sortable: false, align: 'center'}
        ],
        goodsList: [], // 数据
        fielter: {
          search:'',
          saleable:true
        }, //搜索的关键字
        pagination: {}, // 分页排序信息
        totalGoods: 0, // 总记录数
        loading: true, // 显示进度条
        show:false, // 弹出框
        oldGoods:{}, // 保存旧的数据
        isEdit:false, // 是否为修改的状态
        step:1,
      }
    },
    methods: {
      prev(){
        if(this.step > 1) this.step--;
      },
      next(){
        if(this.step < 4) this.step++;
      },
      // add
      addGoods(){
        this.isEdit = false;
        this.show = true;
        this.oldGoods = null;
      },
      // edit
      editGoods(item){
        this.$http({
          url:`/item/brand/cates/${item.id}`,
          method:'get'
        }).then(back=>{
          this.oldGoods = item;
          this.oldGoods.categories = [];
          this.oldGoods.categories.push(back.data[0]);
        });
        this.isEdit = true;
        this.show = true;
      },
      deleteGoods(item){
         this.$message.confirm('确认删除该品牌吗').then(()=>{
           this.$http({
             url:'/item/brand',
             method:'delete',
             params:{
               id:item.id,
             }
           }).then(back=>{
              if(back.data.status === 200){
                 this.$message.success("删除成功");
                 this.getDataFromServer();
              }
           })
         });
      },
      closeWindow(){
        // 关闭窗口
        this.show = false;
        // 重新获取数据
        this.getDataFromServer();
      },
      getDataFromServer() {
        //1. axios.get('url').then(回调函数)
        //2. axios.get('url', {请求参数}).then(回调函数)
        this.$http.get('/item/spu/page', {
            params:{
              key: this.fielter.search, // 搜索条件
              saleable:this.fielter.saleable, // 上下架
              page: this.pagination.page,// 当前页
              rows: this.pagination.rowsPerPage,// 每页大小
              saleable:this.fielter.saleable
          }
        }).then(resp => {
          console.log(resp)
          //const {status, data} = resp
          this.goodsList = resp.data.item
          this.totalgoods = resp.data.total
          this.loading = false
        })
      },
      find() {
        this.getDataFromServer()
      }
    },
    mounted() {
      // 查询数据
      this.getDataFromServer()
    },
    watch: {
      pagination: {
        deep: true,
        handler() {
          this.getDataFromServer()
        }
      },
      fielter:{
        deep:true,
        handler(){
          this.getDataFromServer()
        }
      }
    },
    components:{
      GoodsForm,
    }
  }
</script>

<style scoped>

</style>