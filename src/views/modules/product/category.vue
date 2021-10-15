<template>
  <div>
    <el-tree
      :data="data"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      ><span class="custom-tree-node" slot-scope="{ node, data }">
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
            type="text"
            size="mini"
            @click="() => edit(data)"
          >
            Edit
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span></el-tree
    >
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose"
    >
      <el-form :model="category">
        <el-form-item label="分类名称" >
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitCategory"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
// 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》'

export default {
  // import引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data() {
    return {
      title: '',
      dialogType: '',
      data: [],
      category: {
        catId: null,
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0
      },
      dialogVisible: false,
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  // 计算属性 类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {},
  // 方法集合
  methods: {
    //  获取数据列表
    getDataList() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("获取数据成功", data.data);
        this.data = data.data;
      });
    },
    submitCategory() {
      if (this.dialogType == 'add') {
        this.append();
      }

      if (this.dialogType == 'edit') {
        this.editCategory();
      }
    },
    editCategory() {

    },
    append(data) {
      this.title = '添加';
      this.dialogType = 'add';
      this.dialogVisible = true;
      console.log("data", data);
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },
    edit(data) {
      this.title = '修改';
      this.dialogType = 'edit';
      console.log("要修改的数据", data);
      this.dialogVisible = true;
      this.category.name = data.name;
      this.category.catId = data.catId;
    },
    addCategory() {
      console.log("提交数据", this.category);
      this.$http({
      url:this.$http.adornUrl('/product/category/save'),
      method: 'post',
      data: this.$http.adornData(this.category, false)
      }).then(({ data }) => { 
        this.$message({
              message: "菜单保存成功。",
              type: "success",
            });
        this.dialogVisible = false;
        this.getDataList();
        this.expandedKey = [this.category.parentCid];
      });
    },

    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`此操作将删除[${data.name}]菜单, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              message: "菜单删除成功。",
              type: "success",
            });
            this.getDataList();
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {
          //do nothing
        });
      console.log("delete", node, data);
    },
  },
  // 声明周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getDataList();
  },
  // 声明周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, // 生命周期 - 创建之前
  beforeMount() {}, // 生命周期 - 挂载之前
  beforeUpdate() {}, // 生命周期 - 更新之前
  updated() {}, // 生命周期 - 更新之后
  beforeDestroy() {}, // 生命周期 - 销毁之前
  destroyed() {}, // 生命周期 - 销毁完成
  activated() {}, // 如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>