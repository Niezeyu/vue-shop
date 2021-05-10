<!--home组件  -->
<template>
  <el-container class="home-container">
    <!-- 头部 -->
    <el-header height="100px">
      <div class="header-left">
        <img src="../assets/login.png" alt="" />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>

    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu background-color="#fff" text-color="black" unique-opened :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath">

          <!-- 一级菜单区域 -->
          <el-submenu   :index="item.id + ''" v-for="item in menulist" :key="item.id" >
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>

            <!-- 二级菜单区域 -->
            <el-menu-item :index="'/' + subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavStatu('/' + subItem.path)">
              <template slot="title">
                <i :class="childrenObj[subItem.id]"></i>
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右边内容 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      menulist: [],
      isCollapse: false,
      iconsObj:{
        "125": "el-icon-user",
        "103": "el-icon-set-up",
        "101": "el-icon-shopping-cart-1",
        "102": "el-icon-tickets",
        "145": "el-icon-data-line",
      },
      childrenObj:{
        "110": "el-icon-s-custom",
        "111": "el-icon-s-custom",
        "112": "el-icon-open",
        "104": "el-icon-s-grid",
        "115": "el-icon-menu",
        "121": "el-icon-s-goods",
        "107": "el-icon-document-copy",
        "146": "el-icon-data-analysis",
      },
      // 被激活的链接地址
      activePath: ""
    }
  },
  created() {
    this.getMenuList(),
    this.activePath = window.sessionStorage.getItem("activePath")
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
    },
    // 展开折叠函数
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    // 保存链接的激活状态
    saveNavStatu(activePath) {
      window.sessionStorage.setItem("activePath", activePath),
      this.activePath = activePath
    },
    
  }
}
</script>

<style scoped>
.home-container {
  height: 100%;
}

.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  color: #fff;
  padding-left: 5px;
  font-size: 20px;
}
.el-header .el-button {
  height: 80%;
  width: 10%;
  margin: 6px;
}

.header-left {
  display: flex;
  align-content: center;
  align-items: center;
}
.header-left img {
  height: 60px;
  width: 60px;
  border-radius: 50%;
}
.header-left span {
  margin-left: 15px;
}

.el-aside {
  background-color: #fff;
}
.toggle-button {
  background-color: #eaedf1;
  font-size: 10px;
  line-height: 24px;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
.el-aside .el-menu{
  border-right: 0px;
}


.el-main {
  background-color: #eaedf1;
}
</style>
