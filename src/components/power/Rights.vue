<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 权限表格部分 -->
      <el-table :data="rightsList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="权限名称" prop="authName"></el-table-column>
        <el-table-column label="路径" prop="path"></el-table-column>
        <el-table-column label="权限登记" prop="level">
          <template slot-scope="scope">
            <el-tag type="primary" v-if="scope.row.level === '0'">一级权限</el-tag>
            <el-tag type="success" v-if="scope.row.level === '1'">一级权限</el-tag>
            <el-tag type="warning" v-if="scope.row.level === '2'">一级权限</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rightsList: []
    }
  },
  methods: {
    async getRightsList() {
      const { data: res } = await this.axios.get('/rights/list')
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败！')
      }
      this.rightsList = res.data
      console.log(this.rightsList)
    }
  },
  created() {
    this.getRightsList()
  }
}
</script>

<style lang='less' scoped>

</style>
