<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    >
    </el-switch>
    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="data"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="menuTree"
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
          <el-button type="text" size="mini" @click="() => edit(data)">
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
    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitCategory">确 定</el-button>
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
      pCid: [],
      draggable: false,
      updateNodes: [],
      maxLevel: 0,
      title: "",
      dialogType: "",
      data: [],
      category: {
        catId: null,
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: "",
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
      if (this.dialogType == "add") {
        this.addCategory();
      }

      if (this.dialogType == "edit") {
        this.editCategory();
      }
    },
    editCategory(data) {
      var { catId, name, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单修改成功。",
          type: "success",
        });
        this.dialogVisible = false;
        this.getDataList();
        this.expandedKey = [this.category.parentCid];
      });
    },
    append(data) {
      this.title = "添加";
      this.dialogType = "add";
      this.dialogVisible = true;
      console.log("data", data);
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.catId = null;
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
    },
    edit(data) {
      this.title = "修改";
      this.dialogType = "edit";
      console.log("要修改的数据", data);
      this.dialogVisible = true;
      //发送请求获取当前节点最新数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        console.log("要回显的数据", data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      });
    },
    addCategory() {
      console.log("提交数据", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
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

    allowDrop(draggingNode, dropNode, type) {
      console.log("allowDrop参数", draggingNode, dropNode, type);
      this.countNodelLevels(draggingNode);
      console.log("maxlevel", this.maxLevel);
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;
      console.log("deep", deep);
      if (type == "inner") {
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },
    countNodelLevels(node) {
      //遍历所有子节点，求出最大深度
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          this.countNodelLevels(node.childNodes[i]);
        }
      } else {
        this.maxLevel = node.level;
      }
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("tree drop: ", draggingNode, dropNode, dropType);
      //拖拽节点新的父节点id
      //拖拽节点新的兄弟节点
      let pCid = 0;
      let siblings = null;
      if (dropType == "inner") {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      } else {
        pCid = dropNode.data.parentCid;
        siblings = dropNode.parent.childNodes;
      }
      this.pCid.push(pCid);
      //更新包括拖拽节点在内的子节点的顺序
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId == draggingNode.data.catId) {
          let catLevel = 0;
          if (siblings[i].level != draggingNode.level) {
            catLevel = siblings[i].level;
            this.updateChildNodesLevel(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentId: pCid,
            catLevel: catLevel,
          });
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }
      console.log("updateNodes", this.updateNodes);
    },
    batchSave() {
      //更新拖拽节点及其子节点的层级
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单更新成功。",
          type: "success",
        });
        //刷新菜单
        this.getDataList();
        //展开被拖拽节点
        this.expandedKey = this.pCid;
        this.updateNodes = [];
        this.maxLevel = 0;
      });
    },
    batchDelete() {
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      console.log("批量删除", checkedNodes);
      let catIds = [];
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId);
      }
      this.$confirm(`此操作将批量删除【${catIds}】菜单, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false),
          }).then(({ data }) => {
            this.$message({
              message: "菜单批量删除成功。",
              type: "success",
            });
            //刷新菜单
            this.getDataList();
            //展开被拖拽节点
            // this.expandedKey = this.pCid;
          });
        })
        .catch(() => {
          //do nothing
        });
    },
    updateChildNodesLevel(node) {
      if (node != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var CNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: CNode.catId,
            catLevel: node.childNodes[i].level,
          });
          this.updateChildNodesLevel(node.childNodes[i]);
        }
      }
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