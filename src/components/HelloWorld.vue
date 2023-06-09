<template>
  <div class="compontsContent">
      <div class="selectInput" @click="openSelectTree">
          <div class="referenceBox" @click="openSelectTree" v-if="label_text.length">
              <div class="tagsContent">
                  <el-tag size="mini" class="tags_first">
                      {{ label_text[0].right_show_label || label_text[0].label }}
                  </el-tag>
                  <el-tag size="mini" v-if="label_text.length > 1">
                      +{{ label_text.length - 1 }}
                  </el-tag>
              </div>
          </div>
          <ns-input v-else readonly :placeholder="placeholder" class="ns_input_div"></ns-input>
          <div class="icons_content">
              <ns-icon-svg icon-class="zuzhijigou" class="zuzhijigou_icon"></ns-icon-svg>
          </div>
      </div>
      <ns-dialog width="900px" top="20vh" type="noFooter" append-to-body :visible.sync="visible_select_tree" @close="dialogClose" title="选择">
          <div class="treeContent">
              <div class="leftContent">
                  <div class="headContent">
                      <div class="actionTitle">选择节点</div>
                  </div>
                  <div class="mainContent">
                      <input placeholder="输入关键字进行过滤" v-model="filterText" class="search_input"/>
                      <div class="tree_list">
                          <vue-easy-tree class="filter-tree" :render-after-expand="false" node-key="id" :default-expanded-keys="value" :default-checked-keys="value" show-checkbox :data="tree_data" :props="defaultProps"
                              :filter-node-method="filterNode" ref="tree" @check="handleCheck">
                          </vue-easy-tree>
                      </div>
                  </div>
              </div>
              <div class="rightContent">
                  <div class="select_label">
                      <div class="select_title">已选择的节点</div>
                      <div class="labels" v-if="right_show_arrays.length">
                          <span class="labels_items" v-for="item in right_show_arrays" :key="item.id">
                              {{ item.right_show_label || item.label }}
                              <i class="el-icon-close" @click="deleteCurrentNode(item)" v-show="type != 'check'"></i>
                          </span>
                      </div>
                      <div class="label_empty" v-else>
                        <el-empty description="暂无数据" :image-size="100"></el-empty>
                      </div>
                  </div>
                  <div class="footerContent" v-show="type != 'check'">
                      <ns-button @click="clearAll">清空</ns-button>
                      <ns-button type="primary" class="submitBtn" @click="submit">确定</ns-button>
                  </div>
              </div>
          </div>
      </ns-dialog>
  </div>
</template>

<script>
export default {
  name: 'houseSelectTree',
  data() {
      return {
          visible_select_tree: false,
          filterText: '',
          defaultProps: {
              children: 'children',
              label: 'label',
              disabled: this.disabled_function,
          },
          right_show_arrays: [],
          label_text: [],
          tree_data: [],
      };
  },
  props: {
      //选中id数组
      value: {
          type: Array,
          default() {
              return [];
          },
      },
      //组件类型：edit、check
      type: {
          type: String,
          default: 'edit',
      },
      //数据树 (接口拿到的数据，字段值可改)
      data: {
          type: Array,
          default: ()=>{
            return []
          },
      },
      //字段匹配（id、label、children（子节点））、
      propsName: {
          type: Array,
          default() {
              return ['id', 'label', 'children', 'haveChild', 'right_show_label'];
          },
      },
  },
  computed: {
    placeholder(){
     return this.type == 'edit' ? '请选择交接房产' : '无交接房产'
    },
  },
  watch: {
      data(val) {
          console.log('%c🚀执行', 'color: #43bb88;font-size: 24px', val);
          this.tree_data = this.processTreeData(val);
      },
      value(val) {
          if (val) {
              console.time();
              console.log('%c🚀', 'color: #43bb88;font-size: 24px', val);
              this.right_show_arrays = this.get_treeData_id(val);
              console.log(
                  '%c🚀this.get_treeData_id(val)',
                  'color: #43bb88;font-size: 24px',
                  this.get_treeData_id(val)
              );
              this.label_text = JSON.parse(JSON.stringify(this.right_show_arrays));
              console.timeEnd();
          }
      },
      filterText(val) {
          this.$refs.tree.filter(val);
      },
      immediate: true,
      deep: true,
  },
  created() {},
  mounted() {},

  methods: {
      disabled_function() {
          return this.type == 'check';
      },
      get_treeData_id(id_arr) {
          const resultArray = [];
          function findObjectById(tree, id) {
              if (tree && tree.length > 0) {
                  for (const obj of tree) {
                      if (obj.id === id) {
                          resultArray.push(obj);
                      }
                      if (obj.children) {
                          findObjectById(obj.children, id);
                      }
                  }
              }
          }
          for (const id of id_arr) {
              findObjectById(this.tree_data, id);
          }
          return resultArray;
      },
      submit() {
          const out = this.$refs.tree
              .getCheckedNodes()
              .filter((e) => !e.children || e.children.length == 0)
              .map((e) => {
                  return e.id;
              });
          this.$emit('input', out);
          this.visible_select_tree = false;
      },
      clearAll() {
          this.$refs.tree.setCheckedKeys([]);
          this.right_show_arrays = this.getCheckedOnlyParent(this.$refs['tree']);
      },
      deleteCurrentNode(item) {
          this.right_show_arrays = this.right_show_arrays.filter((e) => e.id != item.id);
          this.set_children(item);
      },
      //取消设置子节点选中
      set_children(item) {
          this.$refs.tree.setChecked(item.id, false);
          if (item.children && item.children.length > 0) {
              item.children.forEach((e) => {
                  this.set_children(e);
              });
          }
      },
      processTreeData(data) {
          //['id','label','children','haveChild']
          if (!data || data.length == 0) return;
          return data.map((item) => {
              const {
                  [this.propsName[0]]: id,
                  [this.propsName[1]]: label,
                  [this.propsName[2]]: children,
                  [this.propsName[3]]: haveChild,
                  [this.propsName[4]]: right_show_label,
              } = item;
              if (haveChild === false) {
                  return { id, label, right_show_label };
              }
              const processedChildren = this.processTreeData(children);
              return { id, label, children: processedChildren, right_show_label };
          });
      },
      //获取勾选的节点（不包括勾选的子节点）
      getCheckedOnlyParent(data) {
          const checkedNodes = [];
          const traverse = function (node) {
              const childNodes = node.root ? node.root.childNodes : node.childNodes;
              childNodes.forEach((child) => {
                  if (child.checked) {
                      checkedNodes.push(child.data);
                  } else {
                      traverse(child);
                  }
              });
          };
          traverse(data);
          return checkedNodes;
      },
      handleCheck(data) {
          this.right_show_arrays = this.$refs.tree
              .getCheckedNodes()
              .filter((e) => !e.children || e.children.length == 0); //展示树节点最底层的节点
          // this.right_show_arrays = this.getCheckedOnlyParent(this.$refs['tree']);
          
          console.log('%c🚀data', 'color: #43bb88;font-size: 24px', data);
      },

      filterNode(value, data) {
          if (!value) return true;
          return data.label.indexOf(value) !== -1;
      },
      openSelectTree() {
          this.visible_select_tree = true;
      },
      dialogClose() {
          this.visible_select_tree = false;
      },
  },
};
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.compontsContent {
  width: 100%;
  .selectInput {
      position: relative;
      width: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      .ns_input_div {
          width: 100%;
      }
      .icons_content {
          position: absolute;
          width: 30px;
          height: 29px;
          right: 1px;
          top: 3px;
          background: #f0f2f5;
          border-top-right-radius: 4px;
          border-bottom-right-radius: 4px;
          display: flex;
          justify-content: center;
          align-items: center;
          .zuzhijigou_icon {
              // color: #bfcbd9;
              color: red;
          }
      }
      .referenceBox {
          border: 1px solid #ccc;
          background: #ffffff;
          margin-top: 2px;
          border-radius: 4px;
          box-sizing: border-box;
          -webkit-box-sizing: border-box;
          color: #666;
          display: inline-block;
          height: 31px;
          line-height: 31px;
          padding: 0 5px;
          -webkit-transition: border-color 0.2s cubic-bezier(0.645, 0.045, 0.355, 1);
          transition: border-color 0.2s cubic-bezier(0.645, 0.045, 0.355, 1);
          width: 100%;
          display: flex;
          align-items: center;
          .tagsContent {
              width: 80%;
          }
      }

      .referenceBox:hover {
          border-color: #0a7af8;
      }
      .referenceBox > div {
          display: flex;
          align-items: center;
      }
      .tags_first {
          text-overflow: ellipsis;
          overflow: hidden;
          max-width: 80%;
      }

      .referenceBox /deep/ .el-tag {
          margin-left: 2.5px;
          margin-right: 2.5px;
          min-width: 10px;
          background-color: #f5f5f5;
          border-color: #ebebeb;
          color: #999;
          padding: 0px 5px;
      }
  }
}
/deep/.el-dialog__body {
  height: 600px !important;
  max-height: 600px !important;
  padding: 0 !important;
}
.treeContent {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  padding: 10px 40px 40px 30px;
  display: flex;
  .leftContent {
      flex: 1;
      height: 100%;
      margin-right: 30px;
      border: 1px solid #d4d4d4;
      .headContent {
          width: 100%;
          height: 40px;
          line-height: 40px;
          background: #f5f5f5;
          .actionTitle {
              width: 100px;
              height: 100%;
              display: inline-flex;
              justify-content: center;
              align-items: center;
              background: #ffffff;
              color: #0a7af8;
              font-weight: 900;
              border-right: 1px solid #d4d4d4;
          }
      }
      .mainContent {
          display: flex;
          width: 100%;
          height: calc(100% - 40px);
          flex-direction: column;
          .tree_list {
              flex: 1;
              overflow-y: scroll;
          }
          .search_input {
              margin: 10px auto;
              width: 95%;
          }
      }
  }
  .rightContent {
      flex: 1;
      height: 100%;
      display: flex;
      flex-direction: column;
      .select_label {
          height: calc(100% - 50px);
          border: 1px solid #d4d4d4;
          display: flex;
          flex-direction: column;
          .select_title {
              height: 50px;
              background: #f5f7fa;
              font-weight: 900;
              line-height: 50px;
              padding-left: 20px;
          }
          .label_empty{
            height: calc(100% - 50px);
            padding: 15px;
            display: flex;
            justify-content: center;
            align-items: center;
            /deep/.el-empty{
              display: flex;
              flex-direction: column;
              align-items: center;
            }
          }
          .labels {
              height: calc(100% - 50px);
              padding: 15px;
              overflow-y: scroll;

              .labels_items {
                  background-color: #e7f2fe;
                  display: inline-block;
                  height: 32px;
                  padding: 0 10px;
                  line-height: 30px;
                  font-size: 12px;
                  color: #0a7af8;
                  border: 1px solid #cee4fe;
                  border-radius: 4px;
                  -webkit-box-sizing: border-box;
                  box-sizing: border-box;
                  white-space: nowrap;
                  margin: 5px;

                  .el-icon-close {
                      border-radius: 50%;
                      text-align: center;
                      position: relative;
                      cursor: pointer;
                      font-size: 12px;
                      height: 16px;
                      width: 16px;
                      line-height: 16px;
                      vertical-align: middle;
                      top: -1px;
                      right: -5px;
                  }
                  .el-icon-close:hover {
                      color: #fff;
                      background-color: #0a7af8;
                  }
              }
              .active_label {
                  background: #e7f2fe;
                  color: #87affa;
                  border: 1px solid #d6e8fe;
              }
          }
      }
      .footerContent {
          height: 50px;
          line-height: 60px;
          position: relative;
          .submitBtn {
              float: right;
              margin-top: 30px;
              transform: translate(0%, -50%);
          }
      }
  }
}
</style>

