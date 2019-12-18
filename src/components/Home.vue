<template>
  <el-container>
    <el-header>
        <div class="left">
          <img src="../assets/logo.png" alt="">
          <span>广告素材管理</span>
        </div>
        <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
        <el-aside :width="isCollapse?'64px' : '200px'">
            <div class="toggle-button" @click="toggleCollapse">|||</div>
            <el-menu
              default-active="2"
              class="el-menu-vertical-demo"
              background-color="#545c64"
              text-color="#fff"
              active-text-color="#409EFF"
              :unique-opened=true
              :collapse="isCollapse"
              :collapse-transition="false"
              router>
              <el-submenu :index="item.id + ' '" v-for="item in menulist" :key="item.id">
                <template slot="title">
                  <i :class="iconsObj[item.id]"></i>
                  <span>{{item.authName}}</span>
                </template>
                <el-menu-item
                  :index="'/' + subItem.path"
                  v-for="subItem in item.children"
                  :key="subItem.id"
                  @click="saveNavState('/' + subItem.path)">
                  <i class="el-icon-menu"></i>
                  <span>{{subItem.authName}}</span>
                </el-menu-item>
              </el-submenu>
            </el-menu>
        </el-aside>
        <el-main>
          <router-view></router-view>
        </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      menulist: [],
      iconsObj: {
        '125': 'iconfont icon-user',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-danju',
        '145': 'iocnfont icon-baobiao'
      },
      isCollapse: false
    }
  },
  created () {
    this._getMenuList()
    console.log('home')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async _getMenuList() {
      const { data: res } = await this.$http.get('/menus')
        if (res.meta.status !== 200) {
          return
        }
        this.menulist = res.data
        console.log(res)
    },
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    }
  }
}
</script>

<style lang="less" scoped>
  .el-container {
    height: 100%;
  }
  .el-header {
    background: #373d41;
    display: flex;
    justify-content: space-between;
    align-items: center;
    // padding: 0;

    .left {
      display: flex;
      align-items: center;
      font-size: 25px;
    }

    img {
      width : 50px;
      height: 50px;
      margin-right: 20px;
    }
  }

  .el-aside {
    background: #333744;
  }

  .el-menu {
    border-right: 0px;
  }

  .el-main {
    background:#EAEDF1;
  }

  .iconfont{
    margin-right: 10px;
  }

  .toggle-button {
    background: #4a5;
    font-size: 20px;
    line-height: 24px;
    text-align: center;
    letter-spacing: 0.2rem;
  }
</style>