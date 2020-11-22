<template>
  <el-tree
    :data="menus"
    :props="defaultProps"
    show-checkbox
    node-key="cid"
    default-expand-all
    :expand-on-click-node="false"
  >
    <span class="custom-tree-node" slot-scope="{ node, data }">
      <span>{{ node.label }}</span>
      <span>
        <el-button type="text" size="mini" @click="() => append(data)">
          Append
        </el-button>
        <el-button type="text" size="mini" @click="() => remove(node, data)">
          Delete
        </el-button>
      </span>
    </span>
  </el-tree>
</template>
<script>
export default {
  data() {
    return {
      menus: [],
      defaultProps: {
          children: 'children',
          label: 'name'
        }
    };
  },
  methods: {
    append(data) {
      
    },

    remove(node, data) {
      
    },
    initTree() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({ data }) => {
        console.log(data)
        this.menus = data.data
      })
    }
  },
  created() {
    this.initTree();
  }
};
</script>
<style scoped></style>
