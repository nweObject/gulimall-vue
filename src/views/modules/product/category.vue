<template>
  <div>
    <el-button @click="deleteMenus()" type="danger">批量删除</el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      show-checkbox
      node-key="catId"
      :check-on-click-node="false"
      :default-expanded-keys="expandedKey"
      :expand-on-click-node="false"
      draggable
      ref="tree"
      :allow-drop="allowDrop"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(node, data)">
            edit
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="visible">
      <el-form :model="category">
        <el-form-item label="菜单名称" :label-width="formLabelWidth">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标" :label-width="formLabelWidth">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" :label-width="formLabelWidth">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="visible = false">取 消</el-button>
        <el-button type="primary" @click="save()">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      maxLev: 1,
      menus: [],
      expandedKey: [],
      visible: false,
      formLabelWidth: "120px",
      title: "",
      category: {
        catId: "",
        name: "",
        parentCid: "",
        catLevel: "",
        showStatus: "",
        sort: "",
        icon: "",
        productUnit: ""
      },
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  methods: {
    deleteMenus() {
      let checkMenus = this.$refs.tree.getCheckedNodes();
      let catIds = [];
      for(let i = 0; i< checkMenus.length; i ++) {
        catIds.push(checkMenus[i].catId);
      }
      this.$confirm("是否删除?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl("/product/category/delete"),
          method: "post",
          data: this.$http.adornData(catIds, false)
        }).then(data => {
          this.$message({
                type: "success",
                message: "删除成功!"
              });
          this.initTree();
        })
      }).catch(() => {
        this.$message({
                type: "info",
                message: "取消删除!"
              });
      })
    },
    append(data) {
      this.category.name="";
      this.category.productUnit = "";
      this.category.icon = "";
      this.category.catId = "";
      this.title = "添加菜单";
      console.log(data);
      this.visible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.showStatus = 1;
      this.category.sort = 0;
    },
    edit(node, data) {
      console.log(data)
      this.visible = true;
      this.title="修改菜单";
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get"
      }).then(({ data }) => {
        var data = data.data
        this.category.name = data.name;
        this.category.productUnit = data.productUnit;
        this.category.icon = data.icon;
        this.category.catId = data.catId;
        this.category.parentCid = data.parentCid;
      });
    },
    save() {
      if (this.category.catId == null || this.category.catId == "") {
        //添加
        this.$http({
          url: this.$http.adornUrl("/product/category/save"),
          method: "post",
          data: this.$http.adornData(this.category, false)
        }).then(({ data }) => {
          if (data.code == 0) {
            this.$message({
              showClose: true,
              message: "添加成功",
              type: "success"
            });
            this.visible = false;
            this.initTree();
            this.expandedKey = [this.category.parentCid];
          }
        });
      }else {
        //修改
        var {catId, name, productUnit, icon} = this.category
        this.$http({
          url: this.$http.adornUrl("/product/category/update"),
          method: "post",
          data: this.$http.adornData({catId, name, productUnit, icon}, false)
        }).then(({data}) => {
          this.$message({
              showClose: true,
              message: "修改成功",
              type: "success"
            });
            this.visible = false;
            this.initTree();
            this.expandedKey = [this.category.parentCid];
        })
      }
    },
    remove(node, data) {
      console.log(node);
      let ids = [data.catId];
      this.$confirm("是否删除?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            if (data.code == 0) {
              this.initTree();
              this.expandedKey = [node.parent.data.catId];
              this.$message({
                type: "success",
                message: "删除成功!"
              });
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    /**
     * 判断节点是否能被拖拉
     * 如果节点的层级小于等于3级则可以被拖拉
     * 节点层级等于被拖节点层级加上当前节点总层级
    */
    allowDrop(draggingNode, dropNode, type) {
      console.log(draggingNode)
      var maxLev = this.computeLev(draggingNode.data);
      this.maxLev = 1;
      let deep = maxLev - draggingNode.data.catLevel + 1;
      console.log(deep)
      if(type == "inner") {
        return (deep + dropNode.level) <= 3;
      }
      return (deep + draggingNode.parent.level) <= 3;
    },

    computeLev(node) {
      if(node.children != null && node.children.length > 0) {
        for(let i = 0; i < node.children.length; i++) {
          if (node.children[i].catLevel > this.maxLev) {
            this.maxLev = node.children[i].catLevel;
          }
          this.computeLev(node.children[i]);
        }
        return this.maxLev;
      }
      return node.catLevel;
    },

    initTree() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
      }).then(({ data }) => {
        console.log(data);
        this.menus = data.data;
      });
    }
  },
  created() {
    this.initTree();
  }
};
</script>
<style scoped></style>
