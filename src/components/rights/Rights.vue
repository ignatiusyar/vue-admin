<template>
  <div class="rights">
    <!-- <h1>haha</h1> -->
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item to="/rights">权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 表格 -->
    <el-table :data="rightList">
      <el-table-column type="index"></el-table-column>
      <el-table-column label="权限名称" prop="authName"></el-table-column>
      <el-table-column label="路径" prop="path"></el-table-column>
      <el-table-column label="层级" prop="level">
        <template slot-scope="{row}">
          <span v-if="row.level === '0'">一级</span>
          <span v-else-if="row.level === '1'">二级</span>
          <span v-else>三级</span>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rightList: []
    }
  },
  beforeMount() {
    this.getRightList()
  },
  methods: {
    async getRightList() {
      // this.axios({
      //   methods: 'get',
      //   url: 'rights/list'
      // }).then(res => {
      //   console.log(res.data)
      // })
      let res = await this.axios.get(`rights/list`)
      if (res.data.meta.status === 200) {
        // console.log('2222')
        this.rightList = res.data.data
        // console.log(this.rightList)
      }
    }
  }
}
</script>

<style>
</style>
