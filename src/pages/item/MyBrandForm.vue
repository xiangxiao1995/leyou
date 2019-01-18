<template>
  <v-form ref="brandForm" v-model="valid">
    <!--品牌名称-->
    <v-text-field
      v-model="brand.name"
      :counter="10"
      label="请输入品牌名称"
      :rules="[v => !!v || '品牌名称不能为空']"
      required
    />
    <!--品牌首字母-->
    <v-text-field
      v-model="brand.letter"
      label="请输入品牌首字母"
      :rules="[v => v.length === 1 || '首字母只能是1位']"
      required
      mask="A"
    />
    <!--商品分类，自定义级联组件-->
    <v-cascader
      url="/item/category/list" required
      v-model="brand.categories"
      multiple label="商品分类"
    />
    <!--品牌LOGO，自定义上传组件-->
    <v-layout row>
      <v-flex xs3>
        <span style="font-size: 16px; color: #444">品牌LOGO：</span>
      </v-flex>
      <v-flex>
        <v-upload
          v-model="brand.image"
          url="/upload/image"
          :multiple="false"
          :pic-width="250"
          :pic-height="250"
        />
      </v-flex>
    </v-layout>
    <!--提交和重置按钮-->
    <v-layout class="my-4 justify-center">
      <v-btn color="primary" @click="submit">提 交</v-btn>
      <v-btn color="warning" @click="clear">重 置</v-btn>
    </v-layout>
  </v-form>
</template>

<script>
    export default {
      name: "MyBrandForm",
      data(){
        return{
          valid: true,
          brand:{
            name: "",
            image: "",
            letter: "",
            categories: []
          }
        }
      },
      props:{
        isEdit: false,
        oldBrand: Object
      },
      watch:{
        oldBrand:{
          deep: true,
          handler: function(val){
            if(val != null){
              this.brand = Object.deepCopy(val);
            }
          }
        }
      },
      methods:{
        //提交表单
        submit(){
          //校验表单
          if(this.$refs.brandForm.validate()){
            this.brand.letter = this.brand.letter.toUpperCase();
            //将categories数组变成字符串，否则传参时会报错
            const {...params} = this.brand;
            params.categories = params.categories.map(c => c.id).join(",");
            //将数据请求到后台
            this.$http({
              method: this.isEdit ? 'put':'post',
              url: '/item/brand',
              data: this.$qs.stringify(params)
            }).then(() => {
              this.$message.success("保存成功！");
              this.$emit("close");
            }).catch(() => {
              this.$message.error("保存失败！")
            })
          }
        },
        //重置表单
        clear(){
          this.$refs.brandForm.reset();
          this.categories = [];
        }
      }
    }
</script>

<style scoped>

</style>
