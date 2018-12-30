<template>
  <div class="roles">
    <!-- <h1>哈哈哈</h1> -->
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item to="/rights">权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-button type="primary" plain>添加角色</el-button>
    <el-table :data="roleList">
      <el-table-column type="expand">
        <template slot-scope="{row}">
          <div v-show="row.children.length  === 0">暂无权限</div>
          <!-- {{ row }} -->
          <!-- 展示一级权限 -->
          <el-row v-for="l1 in row.children" :key="l1.id" class="l1">
            <el-col :span="4">
              <el-tag closable type="primary" @close="delRight(row,l1.id)">{{l1.authName}}</el-tag>
              <i class="el-icon-arrow-right"></i>
            </el-col>
            <el-col :span="20">
              <!-- 展示二级权限 -->
              <el-row class="l2" v-for="l2 in l1.children" :key="l2.id">
                <el-col :span="4">
                  <el-tag closable type="success" @close="delRight(row,l2.id)">{{ l2.authName }}</el-tag>
                  <i class="el-icon-d-arrow-right"></i>
                </el-col>
                <el-col :span="20">
                  <!-- 展示三级权限 -->
                  <el-tag
                    v-for="l3 in l2.children"
                    :key="l3.id"
                    class="l3"
                    closable
                    type="danger"
                    @close="delRight(row,l3.id)"
                  >{{ l3.authName }}</el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </template>
      </el-table-column>
      <el-table-column type="index"></el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="{row}">
          <el-button type="primary" plain icon="el-icon-edit" circle size="mini"></el-button>
          <el-button
            type="danger"
            plain
            icon="el-icon-delete"
            circle
            size="mini"
            @click="delRole(row.id)"
          ></el-button>
          <el-button
            type="success"
            icon="el-icon-check"
            plain
            round
            size="mini"
            @click="showAssign(row)"
          >分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 角色分配模态框 -->
    <el-dialog title="分配权限" :visible.sync="assignDialogVisible" width="30%">
      <!--
        树形菜单
        data: 树形菜单需要显示的数据
        props: 指定显示的属性名（？？？？）
      -->
      <el-tree
        :data="rightList"
        :props="defaultProps"
        node-key="id"
        ref="tree"
        show-checkbox
        default-expand-all
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="assignDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="assignRight()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      roleList: [],
      rightList: [],
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      assignDialogVisible: false,
      roleId: ''
    }
  },
  beforeMount() {
    this.getRoleList()
  },
  methods: {
    // 获取权限列表
    async getRoleList() {
      let res = await this.axios.get(`roles`)
      // console.log(res)
      if (res.data.meta.status === 200) {
        // console.log('222')
        this.roleList = res.data.data
      }
    },
    // 删除权限列表
    async delRight(role, rightId) {
      try {
        let res = await this.axios.delete(`roles/${role.id}/rights/${rightId}`)
        if (res.data.meta.status === 200) {
          role.children = res.data.data
          // console.log(res.data.data)
          this.$message.success('删除权限成功')
        }
      } catch (e) {
        this.$message.error('网络错误,删除失败')
      }
    },
    // 显示添加模态框(并且展示数据)
    async showAssign(role) {
      this.roleId = role.id // 暂存ID
      this.assignDialogVisible = true
      let res = await this.axios.get(`rights/tree`)
      if (res.data.meta.status === 200) {
        // console.log(res.data)
        this.rightList = res.data.data
      }
      let ids = []
      role.children.forEach(l1 => {
        l1.children.forEach(l2 => {
          l2.children.forEach(l3 => {
            ids.push(l3.id)
            // console.log(l3.authName)
          })
        })
        // ids.push(l1.id)
        // console.log(l1)
      })
      this.$refs.tree.setCheckedKeys(ids)
    },
    // 添加/删除权限 更新权限
    async assignRight() {
      let checkedKeys = this.$refs.tree.getCheckedKeys()
      // console.log(checkedKeys)
      let halfCheckedKeys = this.$refs.tree.getHalfCheckedKeys()
      let rids = [...checkedKeys, ...halfCheckedKeys].join()
      // let rids = '...checkedKeys,...halfCheckedKeys'
      // console.log(rids)
      let res = await this.axios.post(`roles/${this.roleId}/rights`, { rids })
      // console.log(res)
      if (res.data.meta.status === 200) {
        // console.log('2222')
        this.assignDialogVisible = false
        this.getRoleList()
        this.$message.success('权限分配成功')
      }
    },
    async delRole(id) {
      // console.log('222')
      // console.log(id)
      try {
        await this.$confirm('你确定要删除该角色吗?', '温馨提示', {
          type: 'warning'
        })
        let res = await this.axios.delete(`roles/${id}`)
        if (res.data.meta.status === 200) {
          this.getRoleList()
          this.$message.success('删除角色成功')
        }
      } catch (e) {
        this.$message.error('取消删除')
      }
    }
  }
}
</script>

<style lang="less" scoped>
.el-button {
  margin-bottom: 5px;
}
.l1 {
  padding: 5px 0;
}
.l3 {
  margin-right: 5px;
  margin-bottom: 5px;
}
</style>
