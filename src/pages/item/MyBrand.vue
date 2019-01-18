<template>
  <v-card>
    <v-card-title>
      <!--新增品牌按钮-->
      <v-btn color="primary" @click="show = true">新增品牌</v-btn>
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
      :items="items"
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
          <v-icon small @click="editBrand(props.item)" style="cursor: pointer">edit</v-icon>
          <v-spacer class="flex xs1"/>
          <v-icon small @click="deleteBrand(props.item)" style="cursor: pointer">delete</v-icon>
        </td>
      </template>
      <!--无信息时的提示-->
      <template slot="no-data">
        <v-alert :value="true" color="error" icon="warning">
          对不起，没有查询到任何数据 :(
        </v-alert>
      </template>
    </v-data-table>
    <!--新增品牌对话框-->
    <v-dialog v-model="show" persistent max-width="600px">
      <v-card>
        <!--对话框顶部工具栏-->
        <v-toolbar dark dense color="primary">
          <v-toolbar-title>{{isEdit ? '修改品牌':'新增品牌'}}</v-toolbar-title>
          <v-spacer/>
          <v-btn icon dark @click="close">
            <v-icon>close</v-icon>
          </v-btn>
        </v-toolbar>
        <!--form表单-->
        <v-card-text class="px-5 py-2">
          <my-brand-form :isEdit="isEdit" :oldBrand="brand" @close="close"></my-brand-form>
        </v-card-text>
      </v-card>
    </v-dialog>
    <!--是否删除品牌对话框-->
    <v-layout row justify-center>
      <v-dialog v-model="deleteDialog" persistent max-width="290">
        <v-card>
          <v-card-title class="headline">是否删除该品牌?</v-card-title>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="error darken-1" flat @click="confirmDelete">删除</v-btn>
            <v-btn color="green darken-1" flat @click="deleteDialog = false">取消</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-layout>
  </v-card>
</template>

<script>
    import VSpec from "./specification/Specification";
    import MyBrandForm from "./MyBrandForm"
    export default {
      name: "MyBrand",
      components: {
        VSpec,
        MyBrandForm
      },
      data(){
        return{
          items:[],//要查询的品牌信息
          totalBrands:0,//品牌总条数
          pagination: {},//分页信息
          loading:false,//是否显示加载进度条
          search:"",//搜索条件
          //表头信息
          headers:[
            {text: '品牌id', align: 'center', sortable: true, value: 'id'},
            {text: '品牌名称', align: 'center', sortable: false, value: 'name'},
            {text: '品牌LOGO', align: 'center', sortable: false, value: 'image'},
            {text: '品牌首字母', align: 'center', sortable: true, value: 'letter'},
            { text: '操作', align: 'center',sortable: false }
          ],
          show:false,//是否显示新增品牌的对话框
          isEdit: false,//是否修改品牌信息
          brand: {},//品牌信息
          deleteDialog:false,//是否显示删除品牌的对话框
          deleteBid:0,//要删除的品牌id
        }
      },
      watch:{
        //监控搜索条件
        search(){
          this.pagination.page = 1;
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
        //关闭对话窗口
        close(){
          //关闭窗口
          this.show = false;
          //修改isEdit为false，即处于新增状态
          this.isEdit = false;
          //重新查询品牌
          this.loadBrands();
        },
        //后台查询品牌信息
        loadBrands(){
          this.$http.get("/item/brand/page",{
            params:{
              key: this.search,//搜索条件
              page: this.pagination.page,//当前页
              rows: this.pagination.rowsPerPage,//每页条数
              sortBy: this.pagination.sortBy,//排序字段
              desc: this.pagination.descending//排序规则
            }
          }).then(resp => {
            //成功回调
            this.items = resp.data.items;
            //从返回的结果中获取总条数
            this.totalBrands = resp.data.total;
          })
        },
        //编辑品牌信息
        editBrand(item){//后台查询品牌的分类信息
          this.$http.get("/item/category/bid/" + item.id).then(resp => {
            // 对brand的赋值必须同步进行，
            // 否则会多次将值赋给<my-brand-form>中的oldBrand属性，
            // 造成watch(val)中的val没有定义
            this.brand = item;
            this.brand.categories = resp.data;
          });
          this.isEdit = true;
          this.show = true;

        },
        //弹出删除品牌窗口
        deleteBrand(item){
          this.deleteDialog = true;
          this.deleteBid = item.id;
        },
        //确认删除品牌
        confirmDelete(){
          this.$http.delete("/item/brand/" + this.deleteBid).then(() => {
            this.$message.success("删除成功！");
            this.loadBrands();
          }).catch(() => {
            this.$message.error("删除失败！")
          });
          this.deleteDialog = false;
        }
      },
      created() {
        //开启加载进度条
        this.loading = true;
        //后台分页查询品牌信息
        this.loadBrands();
        //关闭加载进度条
        this.loading = false;
      }
    }
</script>

<style scoped>

</style>
