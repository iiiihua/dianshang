<template>
    <div>
      <!--面包屑导航区域-->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
      </el-breadcrumb>
      <!--卡片视图-->
      <el-card>
        <!--添加角色区域-->
        <el-row>
          <el-col>
            <el-button type="primary" @click="rolesLogVisble = true">添加角色</el-button>
          </el-col>
        </el-row>
        <!--角色列表区域-->
        <el-table :data="roleslist" border stripe>
          <!--展开列-->
          <el-table-column type="expand">
            <template slot-scope="scope">
              <!--珊格-->
              <el-row :class="['bdbottom', i1 ===0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                <!--渲染一级权限-->
                <el-col :span="5">
                  <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag><i class="el-icon-caret-right"></i>
                </el-col>
                <!--渲染二级跟三级权限-->
                <el-col :span="19">
                  <!--通过for循环嵌套渲染二级权限-->
                  <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                    <!--二级权限-->
                    <el-col :span="6">
                      <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag><i class="el-icon-caret-right"></i>
                    </el-col>
                    <!--三级权限-->
                    <el-col :span="18">
                      <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">
                        {{item3.authName}}
                      </el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </template>
          </el-table-column>
          <!--索引列-->
          <el-table-column type="index"></el-table-column>
          <el-table-column label="角色名称" prop="roleName"></el-table-column>
          <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
          <el-table-column label="操作" width="300px">
            <template slot-scope="scope">
              <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
              <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
              <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-card>
      <!--添加角色区域-->
      <el-dialog
        title="提示"
        :visible.sync="rolesLogVisble"
        width="50%" @close="rolesSetUserClose">
        <!--添加角色form表单-->
        <el-form :model="rolesSetUser" :rules="rolesSetUserRules" ref="rolesSetUserRules" label-width="100px" class="demo-ruleForm">
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="rolesSetUser.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="rolesSetUser.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer">
          <el-button @click="rolesLogVisble = false">取 消</el-button>
          <el-button type="primary" @click="getRolesUser">确 定</el-button>
        </span>
      </el-dialog>
      <!--分配权限的对话框-->
      <el-dialog
        title="分配权限"
        :visible.sync="setRightDialogVisible"
        width="50%" @close="setRightDialogClosed">
        <!--树形控件-->
        <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys='defKeys' ref="treeRef"></el-tree>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRightDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
      </el-dialog>
    </div>
</template>
<script>
export default {
  name: 'Roles',
  data () {
    return {
      // 所有角色列表数据
      roleslist: [],
      // 添加角色窗口控制
      rolesLogVisble: false,
      // 添加角色时的表单对象
      rolesSetUser: {
        roleName: '',
        roleDesc: ''
      },
      rolesSetUserRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' },
          { min: 5, max: 11, message: '用户名长度在5～11个字符之间', trigger: 'blur' }],
        roleDesc: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 50, message: '角色描述长度在6～50个字符之间', trigger: 'blur' }
        ]
      },
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 所有权限的数组
      rightslist: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点id值数组
      defKeys: [],
      // 即将分配权限的角色的id
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色的列表
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败')
      this.roleslist = res.data
    },
    // 关闭表单时重置表单验证数据
    rolesSetUserClose () {
      this.$refs.rolesSetUserRules.resetFields()
    },
    // 添加角色提交
    getRolesUser () {
      // 表单预验证
      this.$refs.rolesSetUserRules.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.rolesSetUser)
        if (res.meta.status !== 201) return this.$message.error('添加角色失败')
        this.$message.success('添加角色成功')
        this.rolesLogVisble = false
        this.getRolesList()
      })
    },
    // 根据ID删除对应的权限
    async removeRightById (role, rightId) {
      // 弹框提示用户是否要删除
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除权限失败')
      // 为了防止重新渲染页面导致的下拉箭头回弹，可以用新的返回的数据赋值给原来的数据，而不是调用getRolesList来重新渲染页面
      this.$message.success('已删除该权限')
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      // 获取所有权限的数据
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限数据失败')
      this.rightslist = res.data
      // 递归获取三级节点的id
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式获取角色下所有三级权限的id，然后保存到defKeys数组中
    getLeafKeys (node, arr) {
      // 如果当前node节点不包含children属性，则是三级节点
      if (!node.children) return arr.push(node.id)
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed () {
      // 每次关闭对话框都重新赋值清空缓存
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) return this.$message.error('分配权限失败')
      this.$message.success('已修改权限')
      this.setRightDialogVisible = false
      this.getRolesList()
    }
  }
}
</script>

<style scoped>
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
