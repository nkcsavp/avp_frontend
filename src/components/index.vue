<template>
  <div>
    <el-container>
      <el-menu class="vertical-menu" router unique-opened>
        <el-menu-item index="/interactive">
          <api :strokeWidth="3" fill="#000000" size="24" strokeLinecap="butt" strokeLinejoin="miter" theme="outline"/>
          自定义
        </el-menu-item>
        <el-menu-item index="/share">
          <airplay :strokeWidth="3" fill="#000000" size="24" strokeLinecap="butt" strokeLinejoin="miter" theme="outline"/>
          ShareKey
        </el-menu-item>
        <el-sub-menu index="1">
          <template #title>
            <sort-two :strokeWidth="3" fill="#000000" size="24" strokeLinecap="butt" strokeLinejoin="miter"
                      theme="outline"/>
            <span>排序算法</span></template>
          <el-menu-item v-for="(value,key) in allSortTypes" :index="'/sort/' + key">{{ value }}</el-menu-item>
        </el-sub-menu>
        <el-sub-menu index="2">
          <template #title>
            <christmas-tree :strokeWidth="3" fill="#000000" size="24" strokeLinecap="butt" strokeLinejoin="miter"
                            theme="outline"/>
            <span> 树算法</span></template>
          <el-menu-item v-for="(value,key) in allTreeTypes" :index="'/tree/' + key">{{ value }}</el-menu-item>
        </el-sub-menu>
        <el-sub-menu index="3">
          <template #title>
            <graphic-stitching-three theme="outline" size="24" fill="#000000" :strokeWidth="3" strokeLinejoin="miter" strokeLinecap="butt"/>
            <span> 图算法</span></template>
          <el-menu-item v-for="(value,key) in allGraphTypes" :index="'/graph/' + key">{{ value }}</el-menu-item>
        </el-sub-menu>
      </el-menu>
      <el-main v-loading="store.state.isFrameLoading" style="padding-top: 0">
        <router-view v-slot="{ Component }">
          <transition mode="out-in" name="fade">
            <component :is="Component" :load="false"/>
          </transition>
        </router-view>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import {useStore} from "vuex";
import {provide} from "vue";
import {Api, ChristmasTree, SortTwo, GraphicStitchingThree, Airplay} from "@icon-park/vue-next";
import {getSortTypes} from "./generator/sort.js";
import {getTreeTypes} from "./generator/tree.js";
import {getGraphTypes} from "./generator/graph.js";

export default {
  name: "Index",
  setup() {
    const store = useStore()
    store.dispatch("Finished")
    let allSortTypes = getSortTypes()
    let allTreeTypes = getTreeTypes()
    let allGraphTypes = getGraphTypes()

    provide("dict_sort", allSortTypes)
    provide("dict_tree", allTreeTypes)
    provide("dict_graph", allGraphTypes)

    return {
      store,
      allSortTypes,
      allTreeTypes,
      allGraphTypes
    }
  },
  components: {
    Api,
    ChristmasTree,
    SortTwo,
    GraphicStitchingThree,
    Airplay
  }
}
</script>

<style scoped>
.vertical-menu{
  width: 200px;
  min-height: 600px;
}

.el-menu-item{
  width: 200px;
}
</style>