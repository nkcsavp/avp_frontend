<template>
  <el-menu :default-active="this.$route.path" mode="horizontal" router>
    <el-menu-item index="/"><img alt="logo" src="/logo.svg">&nbsp;&nbsp;Algorithm Visualization Platform</el-menu-item>
    <el-menu-item v-for="i in navbarItems" :index="i.idx">{{ i.name }}</el-menu-item>
    <el-menu-item v-if="signedIn" :index="'/logout'" @click="signOut">Log Out</el-menu-item>
  </el-menu>
</template>

<script>
import {useStore} from "vuex";
import {useRouter} from "vue-router";
import {computed} from "vue";
import axios from "axios";
import {ElNotification} from "element-plus";

export default {
  name: "navbar",
  setup() {
    const store = useStore();
    const router = useRouter();
    let signedIn = computed(() => store.state.isSignedIn)
    const navbarItems = computed(() => !store.state.isSignedIn ? [{name: "Log In", idx: "/login"}, {name: "Register", idx: "/register"}, {name: "Forget", idx: "/forget"}] : [{name: "Mine", idx: "/mine"},{name: "Update", idx: "/update"}]);
    const signOut = () => {
      store.dispatch("Load")
      axios({
        async: false,
        url: "/logout",
      }).then((res) => {
        store.dispatch("LogOut")
        store.dispatch("Finished")
        ElNotification({
          title: 'Success',
          message: res.data['msg'],
          type: 'success',
        })
        router.push("/")
      })
    }
    return {
      navbarItems,
      signedIn,
      signOut
    }
  }
}
</script>

<style scoped>

</style>