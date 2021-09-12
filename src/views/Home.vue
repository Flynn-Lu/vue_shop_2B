<template>
  
<el-container class="home_container">
  <el-header>
      <span>电商后台管理系统</span>
      <el-button type='info' @click="logout">退出</el-button>
  </el-header>
  
  <el-container>
    <el-aside width="200px">
    <el-menu
    
    :default-active="activePath"
    class="el-menu-vertical-demo"
      background-color="#545c64"
      text-color="#fff"
      active-text-color="#ffd04b"
      unique-opened
      router
      >
      <el-submenu  :index="item.id+''" v-for="item in menulist" :key="item.id">
        <template slot="title">
          <i :class="iconsObj[item.id]"></i>
          <span>{{item.authName}}</span>
        </template>
       <!-- 记得加/ -->
            <el-menu-item :index="'/' + subItem.path" 
            v-for="subItem in item.children" 
            :key="subItem.id"
            @click="saveNavState('/'+subItem.path)">
            {{subItem.authName}}
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
    name:'Home',
    methods:{
        logout(){
            window.sessionStorage.clear()
            this.$router.push('login')
        },
        async getMenuList(){
            const{data:res} = await this.$http.get('menus')
            if(res.meta.status!==200) return this.$message.error(res.meta.msg)
            this.menulist=res.data
            console.log(this.menulist)
        },
        saveNavState(activePath){
            window.sessionStorage.setItem('activeItem',activePath)
            this.activePath=activePath
        }
    },
    data(){
        return{
            menulist:[],
            iconsObj:{
               '125': 'iconfont icon-user',
                '103': 'iconfont icon-tijikongjian',
                '101': 'iconfont icon-shangpin',
                '102': 'iconfont icon-danju',
                '145': 'iconfont icon-baobiao' 
            },
            activePath:''
        }
    },
    created(){
        this.getMenuList()
        this.activePath = window.sessionStorage.getItem('activePath')
    }

}
</script>

<style lang="less" scoped>
.home_container{
    height: 100%;
}
.el-header{
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 20px;
    > span{
        display: block;
        color:white;
        // font-size:26px;
        // text-align: center;
    }
    
}

.el-aside {
  background-color: #333744;
  .el-menu {
    border-right: none;
  }
  .el-submenu {
      background-color: inherit;
  }
}

.el-main {
  background-color: #eaedf1;
}
</style>