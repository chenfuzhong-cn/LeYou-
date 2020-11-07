<template>
  <div>
    <v-form ref="myBrandForm">
      <v-text-field v-model="brand.name" label="请输入品牌名" required clearable :rules="brandNameRules" />
      <v-text-field
        v-model="brand.letter"
        label="请输入品牌首字母"
        required
        clearable
        :rules="brandFirstRules"
      />
      <v-cascader
        url="/item/category/list"
        multiple
        requierd
        :rules="brandProduct"
        v-model="brand.categories"
        label="请选择商品"
      />
      <v-layout>
        <v-flex xs3>
          <span class="subheading">品牌LOGO</span>
        </v-flex>
        <v-flex xs9>
          <v-upload
            v-model="brand.image"
            url="/upload"
            :multiple="false"
            :pic-height="90"
            :pic-width="250"
          />
        </v-flex>
      </v-layout>
      <v-layout class="pt-4">
        <v-spacer></v-spacer>
        <v-btn dark color="blue" @click="submit">提交</v-btn>
        <v-btn dark color="blue" @click="reset">重置</v-btn>
      </v-layout>
    </v-form>
  </div>
</template>
<script>
export default {
  // 父组件给子组件传的值

  data() {
    return {
      brand: {
        name: "", // 品牌名
        image: "", // LOGO
        letter: "", // 品牌首字母
        categories: [], // 分类信息
      },
      brandNameRules: [
        (v) => !!v || "品牌名不能为空",
        (v) => v.length >= 2 || "至少两位名称品牌名",
      ],

      brandFirstRules: [
        (v) => !!v || "首字母不能为空",
        (v) => /^[A-Z]$/.test(v) || "品牌字母只能是一个A~Z的大写字母",
      ],

      brandProduct: [(v) => !!v || "请选择商品"],
      /*  */
      
    }
  },
  props: {
        oldBrand: {
          type: Object,
        },
        isEdit:{
          type:Boolean,
          default:false
        }
      },
  methods: {
    submit() {
      if (this.$refs.myBrandForm.validate()) {
        // 发送axios请求，新增，post /item/brand, 请求参数
        // 解构brands数组，提取出categories，剩下的放到params对象中
        const { categories, ...params } = this.brand;
        params.cids = categories.map((c) => c.id).join(",");
        this.$http({
          url: "/item/brand",
          method: this.isEdit ? 'put' : 'post',
          data: params,
        }).then((back) => {
          console.log(back);
          const { data, status } = back.data;
          if (status === 201) {
            this.$message.success("保存成功");
            this.$emit("close");
          } else {
            this.$message.error("保存失败");
          }
        })
      } else {
        this.$message({
          type: "error",
          $message: "表单验证失败",
        });
      }
    },
    reset() {
      // 重置表单
      this.$refs.myBrandForm.reset();
      this.brand.categories = [];
    },
  },
  /* 监听 */
  watch: {
    // 监听oldBrand 的变化
    oldBrand: {
      deep: true,
      // val 是 oldBrand的值
      handler(val) {
        if (val) {
          // 修改操作
          // 注意不要直接复制，否则这边的修改会直接影响到父组件的数据，copy属性即可
          this.brand = Object.deepCopy(val);
        } else {
          // 新增操作
          this.brand = {
            name: "", // 品牌名
            image: "", // LOGO
            letter: "", // 品牌首字母
            categories: [], // 分类信息
          }
        }
      },
    },
  },
};
</script>

<style>
</style>