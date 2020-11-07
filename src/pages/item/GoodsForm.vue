<template>
  <div>
    <v-stepper v-model="step">
      <v-stepper-header>
        <v-stepper-step :complete="step > 1" step="1">基本信息</v-stepper-step>
        <v-divider></v-divider>
        <v-stepper-step :complete="step > 2" step="2">商品描述</v-stepper-step>
        <v-divider></v-divider>
        <v-stepper-step :complete="step > 3" step="3">规格参数</v-stepper-step>
        <v-divider></v-divider>
        <v-stepper-step step="4">sku属性</v-stepper-step>
      </v-stepper-header>
      <v-stepper-item>
        <v-stepper-content step="1">
          <!-- showAllLevels 显示三级分类 -->
          <v-cascader
            url="/item/category/list"
            showAllLevels
            v-model="goods.categories"
            label="请选择商品分类"
            required
          />
          <!-- v-model="goods.brand_id"  将seelct的value赋值给brand_id -->
          <v-select
            label="所属品牌"
            :items="brandOptions"
            item-text="name"
            item-value="id"
            v-model="goods.brand_id"
            required
            autocomplete
            clearable
            dense
            chips
          ></v-select>
          <v-text-field label="商品标题" v-model="goods.title" :counter="200"></v-text-field>
          <v-text-field label="商品卖点" v-model="goods.sub_title" :counter="200"></v-text-field>
          <!-- multi-line 多行的 -->
          <v-text-field
            label="包装清单"
            v-model="spu_detail.packing_list"
            :counter="1000"
            multi-line
            :row="3"
          ></v-text-field>
          <v-text-field
            label="售后服务"
            v-model="spu_detail.after_service"
            :counter="1000"
            multi-line
            :row="3"
          ></v-text-field>
        </v-stepper-content>
        <v-stepper-content step="2">
          <v-editor
            v-model="spu_detail.description"
            upload-url="/upload/static/upload"
            fileName="file"
          />
        </v-stepper-content>
        <v-stepper-content step="3">
          <!-- :key="" 唯一的标识下标 -->
          <!-- 
                mx-auto == margin: 0 auto
                px-3    == padding:0 10px;
                subheading == font-weight:25px
          -->
          <v-flex xs10 class="mx-auto px-3">
            <v-card v-for="spec in specification" :key="spec.group" class="my-2">
              <!-- 组名称 -->
              <v-card-title class="subheading">{{spec.group}}</v-card-title>
              <!-- 遍历组中的每个属性 -->
              <v-card-text v-for="param in spec.params" :key="param.k">
                <!-- 
                      v-text-field == <input type="text">
                -->
                <v-text-field
                  v-model="param.v"
                  v-if="param.options.length <= 0"
                  :label="param.k"
                  :suffix="param.unit || ''"
                ></v-text-field>
                <v-select 
                  v-else 
                  :label="param.k" 
                  :items="param.options"
                  v-model="param.v"
                ></v-select>
              </v-card-text>
            </v-card>
          </v-flex>
        </v-stepper-content>
        <v-stepper-content step="4">sku属性</v-stepper-content>
      </v-stepper-item>
    </v-stepper>
  </div>
</template>

<script>
export default {
  data() {
    return {
      goods: {
        categories: [], // 分类信息
        brand_id: "", // 品牌ID
        title: "", // 标题
        sub_title: "", // 热卖标题
      },
      brandOptions: [], // 可选的品牌选项
      specification: [], // 规格参数模板
      spu_detail: {
        description: "",
        packing_list: "",
        after_service: "",
      },
    };
  },
  props: {
    step: {
      type: Number,
      default: 1,
    },
  },
  watch: {
    "goods.categories": {
      deep: true,
      handler() {
        // 根据最后一级的cid，获取该分类下的品牌信息
        this.$http({
          url: `/item/cate_brand/${this.goods.categories[2].id}`,
          method: "get",
        }).then((back) => {
          const { status, data } = back;
          if (status === 200) {
            this.brandOptions = data;
          }
        });

        // 根据最后一级的cid，得到对应的模板信息
        this.$http({
          url: `/item/spec/${this.goods.categories[2].id}`,
          method: "get",
        }).then((back) => {
          console.log(back);
          const { data, status } = back;
          if (status === 200) {
            this.specification = data;
          }
        });
      },
    },
  },
};
</script>

<style>
</style>