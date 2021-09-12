<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card >
        <el-row :gutter="20">
            <el-col>
               <el-button type="primary">添加角色</el-button> 
            </el-col>
        </el-row>
        <el-table
            :data="roleList"
            stripe
            style="width: 100%"
            border
            >
            <!-- 展开列 -->
            <el-table-column type="expand">
                <template slot-scope="scope">
                    <el-row v-for="(item1,i1) in scope.row.children" :key='item1.id' :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']">
                        <el-col :span='5'>
                            <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>
                        </el-col>
                        <el-col :span='19'>
                            <el-row v-for="(item2,i2) in item1.children" :key="item2.id" :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']">
                                <el-col :span='8'>
                            <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)" >{{item2.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>
                            </el-col>
                            <el-col :span='16'>
                                <el-tag type="warning" closable @close="removeRightById(scope.row,item3.id)" v-for="(item3) in item2.children" :key="item3.id">{{item3.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            </el-row>
                            
                        </el-col>
                        

                    </el-row>
                  <!-- <pre>{{scope.row}}</pre>   -->
                </template>
                
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            
            <el-table-column label="角色操作">
                <template slot-scope="scope">
                    <el-button type="primary" icon="el-icon-edit" size='mini'></el-button>
                
                <el-button type="danger" icon="el-icon-delete" size='mini'></el-button>
                <el-tooltip
              effect="dark"
              content="分配角色"
              placement="top"
              :enterable="false"
                    >
                <el-button type="warning" icon="el-icon-setting" size='mini' @click='showSetRightDialog(scope.row)'></el-button>
                </el-tooltip>
                </template>
                
            </el-table-column>
        </el-table>
        
    </el-card>

    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" @closed='setRightDialogClosed'>
    <el-tree
    :data="rightsList"
    show-checkbox
    node-key="id"
    
    :props="treeProps"
    default-expand-all 
    :default-checked-keys="defKeys" 
    ref="treeRef">
    </el-tree>

    <div slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
    </div>
</el-dialog>
</div>
</template>

<script>
export default {
    name:'Roles',
    data(){
        return{
            roleList:[],
            setRightDialogVisible:false,
            rightsList:[],
            roleId:'',
            defKeys:[],
            treeProps: {
        label: 'authName',
        children: 'children'
      },
        }
    },
    created(){
        this.getRoleList()
    },
    methods:{
        async getRoleList(){
            const {data:res}=await this.$http.get('roles')
            if(res.meta.status!=200){
                this.$message.error('获取用户列表失败');
            }
            this.$message.success('获取用户列表成功')
            this.roleList=res.data
            console.log(this.roleList)
        },
        async removeRightById(role,rightId){
                  // 弹框提示用户是否要删除
      const confirmResult = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除！')
      }
      //传的是role的id  没有删掉右尖号
      const{data:res}=await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
            if(res.meta.status!=200){
                this.$message.error('删除失败')
            }
            // this.getRolesList()!!!!!!!!更新

      role.children = res.data
      console.log(role.children)
        },
        async showSetRightDialog(role){
            this.roleId=role.id
            const {data:res}=await this.$http.get('rights/tree')
            if(res.meta.status!==200){
                this.$message.error('获取权限列表失败')
            }
            this.rightsList=res.data
            this.getLeafKeys(role,this.defKeys)
            this.setRightDialogVisible=true
        },
        /////////!!!!!!!!!!!!!!!!!!!!!!!
        getLeafKeys(node,arr){
            if(!node.children)
            return this.defKeys.push(node.id)

            node.children.forEach(item => 
            this.getLeafKeys(item,arr)
            );

        },
        async allotRights(){
            const keys = [...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()]
        const idStr = keys.join(',')
        const {data:res}= await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
        if(res.meta.status!==200){
            this.$message.error('分配错误')
        }
        
        this.getRoleList()
        this.setRightDialogVisible=false
        },
        setRightDialogClosed(){
            this.defKeys=[]
        }

    }

}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>