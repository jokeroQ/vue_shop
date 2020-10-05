<template>
    <div>
         <!-- 面包屑导航区 -->
       <el-breadcrumb separator-class="el-icon-arrow-right">
  <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>权限管理</el-breadcrumb-item>
  <el-breadcrumb-item>权限列表</el-breadcrumb-item>
</el-breadcrumb>
<!-- 卡片视图区 -->
<el-card>
    <!-- 添加角色按钮区域 -->
    <el-row>
        <el-col>
    <el-button type="primary" @click="addroles">添加角色</el-button>
        </el-col>
    </el-row>
    <!-- 角色列表区域 -->
    <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
            <template slot-scope="scope">
                <el-row :class="['bdbottom',i1===0?'bdtop':'','vcenter']" v-for="(item1,i1) in scope.row.children":key="item1.id">
                    <!-- 渲染一级权限 -->
                    <el-col :span="5">
                        <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                        <i class="el-icon-caret-right"></i>
                    </el-col>
                    <!-- 渲染二级和三级权限 -->
                    <el-col :span="19">
                        <!-- 通过for循环 嵌套渲染二级权限 -->
                        <el-row :class="[i2==0?'':'bdtop','vcenter']" v-for="(item2,i2) in item1.children":key="item2.id">
                            <el-col :span="6">
                        <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                        <i class="el-icon-caret-right"></i>
                            </el-col>
                            <el-col :span="18">
                        <el-tag type="warning" v-for="(item3,i3) in item2.children":key="item3.id" closable @close="removeRightById(scope.row,item3.id)">{{item3.authName}}</el-tag>
                            </el-col>
                        </el-row>
                    </el-col>
                </el-row>
            </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
            <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="editroles(scope.row.id)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteroles(scope.row.id)">删除</el-button>
                <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
            </template>
        </el-table-column>
    </el-table>
</el-card>
<el-dialog title="编辑角色" :visible.sync="editDialogFormVisible" width="50%">
  <el-form :model="editForm">
    <el-form-item label="角色名称">
      <el-input v-model="editForm.roleName"></el-input>
    </el-form-item>
    <el-form-item label="角色描述">
      <el-input v-model="editForm.roleDesc"></el-input>
    </el-form-item>
  </el-form>
  <div slot="footer" class="dialog-footer">
    <el-button @click="editDialogFormVisible = false">取 消</el-button>
    <el-button type="primary" @click="editrolesInfo">确 定</el-button>
  </div>
</el-dialog>
<el-dialog title="添加角色" :visible.sync="addDialogFormVisible" width="50%">
  <el-form :model="addForm">
    <el-form-item label="角色名称">
      <el-input v-model="addForm.roleName"></el-input>
    </el-form-item>
    <el-form-item label="角色描述">
      <el-input v-model="addForm.roleDesc"></el-input>
    </el-form-item>
  </el-form>
  <div slot="footer" class="dialog-footer">
    <el-button @click="addDialogFormVisible = false">取 消</el-button>
    <el-button type="primary" @click="addrolesInfo">确 定</el-button>
  </div>
</el-dialog>
<!-- 分配权限的对话框 -->
<el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
  <!-- 树形控件 -->
  <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
  <div slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </div>
</el-dialog>
    </div>
</template>
<script>
export default {
    data(){
        return{
            rolelist:[],
            editDialogFormVisible:false,
            editForm:{},
            //  editFormRules:{
            //     roleName:[
            //         {required: true, message: '请输入角色名称', trigger: 'blur'},
            //         {validator:checkEmail,trigger: 'blur'}
            //     ]
            // }
            addDialogFormVisible:false,
            addForm:{
                roleName:'',
                roleDesc:''
            },
            // 控制分配权限对话框的显示与隐藏
            setRightDialogVisible:false,
            // 所有权限的数据
            rightslist:[],
            // 树形控件的属性绑定对象
            treeProps:{
                label:'authName',
                children:'children'
            },
            // 默认选中的节点Id数值组
            defKeys:[],
            // 当前即将分配权限的角色Id
            roleId:''
        }
    },
created(){
    this.getRolesList()
},
methods:{
    // 获取所有角色的列表
    async getRolesList(){
const {data:res}=await this.$http.get('roles')
if(res.meta.status!==200){
    return this.$message.error('获取角色列表失败')
}
this.rolelist=res.data
console.log(this.rolelist);
    },
    // 编辑对应的角色信息
    async editroles(id){
        const {data:res}=await this.$http.get('roles/'+id)
        if(res.meta.status!==200){
                return this.$message.error('查询角色信息失败')
            }
        this.editForm=res.data
        this.editDialogFormVisible=true
        // console.log(res);
    },
    async editrolesInfo(){
         const {data:res}=await this.$http.put('roles/'+this.editForm.roleId,{
                    roleName:this.editForm.roleName,
                    roleDesc:this.editForm.roleDesc
                })
                if(res.meta.status!==200){
                    return this.$message.error('更新角色信息失败')
                }
                // 关闭对话框
                this.editDialogFormVisible=false
                // 刷新数据列表
                this.getRolesList()
                // 提示修改成功
                this.$message.success('更新角色信息成功')
    },
    // 添加对应的角色信息
    addroles(){
        this.addDialogFormVisible=true
    },
    async addrolesInfo(){
         const {data:res}=await this.$http.post("roles",this.addForm)
                if(res.meta.status!==201){
                    return this.$message.error('添加用户请求失败')
                }
                this.$message.success('添加用户请求成功')
                // 隐藏添加用户的对话框
                this.addDialogVisible=false
                // 重新获取用户列表
                this.getRolesList()
    },
    // 删除对应的角色信息
   async deleteroles(id){
         // 弹框询问用户是否删除数据
            const confirmResult= await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err);
         if(confirmResult!=='confirm'){
                return this.$message.info('已取消删除')
            }
            const {data:res}=await this.$http.delete('roles/'+id)
            if(res.meta.status!==200){
                return this.$message.error('删除角色失败')
            }
            this.$message.success('删除角色成功')
            this.getRolesList()
    },
    // 根据Id删除对应的权限
   async removeRightById(role,rightId){
        const confirmResult=await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err)
        if(confirmResult!=='confirm'){
            return this.$message.info('取消了删除')
        }
       const {data:res}=await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
       if(res.meta.status!==200){
           return this.$message.error('删除权限失败')
       }
    //    this.getRolesList()
    role.children=res.data
    },
    // 显示分配权限的对话框
   async showSetRightDialog(role){
       this.roleId=role.id;
        // 获取所有权限的数据
        const {data:res}=await this.$http.get('rights/tree')
        if(res.meta.status!==200){
            return this.$message.error('获取权限列表数据失败')
        }
        // 把获取到的权限数据保存到data中
        this.rightslist=res.data
        console.log(this.rightslist);
        // 递归获取三级节点的Id
        this.getLeafKeys(role,this.defKeys)
        this.setRightDialogVisible=true
    },
    // 通过递归的形式，获取角色下所有三级权限的id，并保存到defKeys数组中
    getLeafKeys(node,arr){
        // 如果当前node不包含children属性，则是三级节点
        if(!node.children){
            return arr.push(node.id)
        }
        node.children.forEach(item=>
        this.getLeafKeys(item,arr)
        )
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed(){
        this.defKeys=[]
    },
    // 点击为角色分配权限
   async allotRights(){
        const keys=[
            ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()]
        // console.log(keys);
        const idStr=keys.join(',')
    const {data:res}=await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
    if(res.meta.status!==200){
        return this.$message.error('分配权限失败')
    }
    this.$message.success('分配权限成功')
    this.getRolesList()
    this.setRightDialogVisible=false
    }
}
}
</script>
<style lang="less" scoped>
.el-tag{
    margin: 7px;
}
.bdtop{
    border-top: 1px solid #eee;
}
.bdbottom{
    border-bottom: 1px solid #eee;
}
.vcenter{
    display: flex;
    align-items: center;
}
</style>