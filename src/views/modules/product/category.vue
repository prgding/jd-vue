<script>
export default {
  data() {
    return {
      dialogVisible: false,
      title: "",
      dialogType: "",
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
      },
      expandedKey: [],
      menus: [{
        label: '一级 1',
        children: []
      }],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    };
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get',
      }).then(res => {
        this.menus = res.data.data
        console.log("成功获取 =", res);
      });
    },
    submitData() {
      if (this.dialogType === "add") {
        this.addCategory();
      } else if (this.dialogType === "edit") {
        this.editCategory();
      }
    },
    append(data) {
      console.log("append", data);
      this.title = "添加菜单"
      this.dialogType = "add"
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },
    addCategory() {
      console.log("提交的数据", this.category);
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(res => {
        this.$message({
          type: 'success',
          message: '提交成功!'
        });
        this.dialogVisible = false;
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      })
    },
    edit: function (data) {
      console.log("要修改的数据", data);
      this.title = "修改菜单"
      this.dialogType = "edit"
      this.dialogVisible = true;

      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get',
      }).then(res => {

        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.menus = res.data.data
        console.log("成功获取 =", res);
      });


    },
    editCategory: function () {
    },
    remove: function (node, data) {
      console.log("remove", node, data);

      var ids = [data.catId];

      this.$confirm(`确认删除【${data.name}】菜单`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(res => {
          this.$message({
            type: 'success',
            message: '删除成功!'
          });
          // 刷新菜单
          this.getMenus();
          // 展开原先节点
          this.expandedKey = [node.parent.data.catId];
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },

  },
  created() {
    this.getMenus();
  }
}
;
</script>

<template>
  <div>
    <el-tree
      :data="menus"
      :default-expanded-keys="expandedKey"
      :expand-on-click-node="false"
      :props="defaultProps"
      node-key="catId"
      show-checkbox>
    <span slot-scope="{ node, data }" class="cu stom-tree-node">
      <span>{{ node.label }}</span>
      <span>
        <el-button
          v-if="node.level <= 2"
          size="mini"
          type="text"
          @click="() => append(data)">
          添加
        </el-button>
        <el-button
          v-if="node.childNodes.length === 0"
          size="mini"
          type="text"
          @click="() => remove(node, data)">
          删除
        </el-button>
        <el-button
          size="mini"
          type="text"
          @click="() => edit(data)">
          编辑
        </el-button>
      </span>
    </span>
    </el-tree>

    <el-dialog
      :visible.sync="dialogVisible"
      :title="this.title"
      width="30%"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<style lang="scss" scoped>

</style>
