<template>
  <div class="hello">
    <vue-easy-tree
      ref="tree"
      node-key="id"
      height="300px"
      :data="tree_data"
      show-checkbox
      :props='props'
      @check="handleCheck"
    ></vue-easy-tree>
    <span v-for="item in right_show_arrays" :key="item.value">
      {{ item.name }}
    </span>
    <house-select-tree v-model="presentHouseList" :type="type" :data="tree_data" :propsName="tree_props_name"></house-select-tree>
  </div>
</template>
 
<script>
import VueEasyTree from "@wchbrad/vue-easy-tree";
import houseSelectTree from './components/HelloWorld.vue'
// 样式文件，可以根据需要自定义样式或主题
import "@wchbrad/vue-easy-tree/src/assets/index.scss";
export default {
  name: "app",
  components: {
    VueEasyTree,
    houseSelectTree
  },
  data() {
    return {
      props: {
        label: "name",
        children: "children",
      },
      right_show_arrays:[],
      tree_data: [],
      presentHouseList:[],
      type:'edit',
      tree_props_name: ['HouseID', 'ResName', 'lsChild', 'haveChild', 'HouseShortName'],
    };
  },
  created() {
    const data = [],
      root = 8,
      children = 3,
      base = 1000;
    for (let i = 0; i < root; i++) {
      data.push({
        HouseID: `${i}`,
        ResName: `test-${i}`,
        haveChild: [],
      });
      for (let j = 0; j < children; j++) {
        data[i].children.push({
          id: `${i}-${j}`,
          name: `test-${i}-${j}`,
          children: [],
        });
        for (let k = 0; k < base; k++) {
          data[i].children[j].children.push({
            id: `${i}-${j}-${k}`,
            name: `test-${i}-${j}-${k}`,
          });
        }
      }
    }
    this.tree_data = data;
  },
  methods: {
    handleCheck() {
      // this.$refs.tree.getCheckedNodes()

            this.right_show_arrays = this.$refs.tree
                .getCheckedNodes()
                .filter((e) => !e.children || e.children.length == 0); //展示树节点最底层的节点
                console.log('%c🚀nodes', 'color: #43bb88;font-size: 24px',this.$refs.tree
                .getCheckedNodes());
                //children
                console.log('%c🚀data', 'color: #43bb88;font-size: 24px', this.right_show_arrays);
        },
  }
};
</script>
 
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>