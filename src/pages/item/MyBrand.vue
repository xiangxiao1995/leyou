<template>
    <div>
      <v-card>
        <v-card-title>
          <!--新增品牌按钮-->
          <v-btn color="primary">新增品牌</v-btn>
          <v-spacer/>
          <!--搜索框-->
          <v-text-field
            v-model="search"
            append-icon="search"
            label="搜索"
            hide-details
          ></v-text-field>
        </v-card-title>
        <v-divider/>
        <!--数据表-->
        <v-data-table
          :headers="headers"
          :items="brands"
          :pagination.sync="pagination"
          :total-items="totalBrands"
          :loading="loading"
          class="elevation-1"
        >
          <!--遍历行数据-->
          <template slot="items" slot-scope="props">
            <td class="text-xs-center">{{ props.item.id }}</td>
            <td class="text-xs-center">{{ props.item.name }}</td>
            <td class="text-xs-center"><img :src="props.item.image" alt=""></td>
            <td class="text-xs-center">{{ props.item.letter }}</td>
            <td class="justify-center layout px-0 py-3">
              <v-icon small @click="editItem(props.item)" style="cursor: pointer">edit</v-icon>
              <v-spacer class="flex xs1"/>
              <v-icon small @click="deleteItem(props.item)" style="cursor: pointer">delete</v-icon>
            </td>
          </template>
        </v-data-table>
      </v-card>
    </div>
</template>

<script>
    import VSpec from "./specification/Specification";
    export default {
      name: "MyBrand",
      components: {VSpec},
      data(){
        return{
          //表头信息
          headers:[
            {text: '品牌id', align: 'center', sortable: true, value: 'id'},
            {text: '品牌名称', align: 'center', sortable: false, value: 'name'},
            {text: '品牌LOGO', align: 'center', sortable: false, value: 'image'},
            {text: '品牌首字母', align: 'center', sortable: true, value: 'letter'},
            { text: '操作', align: 'center',sortable: false }
          ],
          brands:[],//要查询的品牌信息
          totalBrands:0,//品牌总条数
          pagination: {},//分页信息
          loading:false,//是否显示加载进度条
          dialog:false,//是否显示新增品牌的对话框
          search:"",//搜索条件
        }
      },
      watch:{
        //监控搜索条件
        search(){
          this.loadBrands();
        },
        //监控分页信息
        pagination:{
          deep: true,
          handler(){
            this.loadBrands();
          }
        }
      },
      methods:{
        //后台查询品牌信息
        loadBrands(){
          this.$http.get("/brand/page",{
            params:{
              key: this.search,//搜索条件
              page: this.pagination.page,//当前页
              rows: this.pagination.rowsPerPage,//每页条数
              sortBy: this.pagination.sortBy,//排序字段
              desc: this.pagination.descending//排序规则
            }
          })
        },
        //编辑品牌信息
        editItem(item){
          alert(item);
        },
        //删除品牌信息
        deleteItem(item){
          alert(item);
        }
      },
      created() {
        //开启加载进度条
        this.loading = true;
        //后台分页查询品牌信息
        this.loadBrands();
        //从返回的结果中获取总条数
        this.totalBrands = 15;
        //关闭加载进度条
        this.loading = false;
      }
    }
</script>

<style scoped>

</style>
