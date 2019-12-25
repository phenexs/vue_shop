<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区 -->
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <!-- 渲染一级权限 -->
            <el-row
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
              :class="['btbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
            >
              <el-col :span="5">
                <el-tag
                  type="primary"
                  closable
                  @close="removeRightById(scope.row,item1.id)"
                >{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级权限 -->
              <el-col :span="19">
                <el-row
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row,item2.id)"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row,item3.id)"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSetRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightdialogVisible" width="50%" @close="setRightDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightList" :props="treeProps" show-checkbox node-key="id" :default-expand-all="true" :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rolesList: [],
      setRightdialogVisible: false,
      rightList: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点Id值
      defKeys: [],
      // 即将分配角色的Id
      roleId: ''
    }
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.axios.get('/roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表数据失败！')
      }
      this.rolesList = res.data
      console.log(this.rolesList)
    },
    // 根据Id删除对应的权限
    async removeRightById(role, rightId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该权限, 是否继续?',
        '提示',
        { confirmButtonText: '确定', cancelButtonText: '取消', type: 'warning' }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了此次删除')
      }
      // 调用删除接口
      const { data: res } = await this.axios.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }
      this.$message.success('删除权限成功！')
      // 此处不调用getRolesList方法，因为会折叠页面
      role.children = res.data
    },
    async showSetRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.axios.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败！')
      }
      this.rightList = res.data
      console.log(this.rightList)
      this.getLeafKeys(role, this.defKeys)
      this.setRightdialogVisible = true
    },
    // 通过递归的函数，角色下所有三级权限的id，并保存到defKeys数组中
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件，清空defKeys 数组中的数据
    setRightDialogClosed() {
      this.defKeys = []
    },
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      // console.log(keys)
      const idStr = keys.join(',')
      const { data: res } = await this.axios.post(`roles/${this.roleId}/rights`, { rids: idStr })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightdialogVisible = false
    }
  },
  created() {
    this.getRolesList()
  }
}
</script>

<style lang='less' scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.btbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
