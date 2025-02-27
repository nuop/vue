<template>
  <el-card>
    <div slot="header">
      <el-form :inline="true" :model="filterParams" ref="filter-form">
        <el-form-item label="手机号" prop="phone">
          <el-input v-model="filterParams.phone"></el-input>
        </el-form-item>
        <el-form-item label="注册时间" prop="rangeDate">
          <el-date-picker
            v-model="filterParams.rangeDate"
            type="datetimerange"
            range-separator="至"
            start-placeholder="开始时间"
            end-placeholder="结束时间"
            value-format="yyyy-MM-dd"
          />
        </el-form-item>
        <el-form-item>
          <el-button :disabled="isLoading" @click="handleReset">重置</el-button>
          <el-button type="primary" @click="handleQuery" :disabled="isLoading"
            >查询</el-button
          >
        </el-form-item>
      </el-form>
    </div>
    <el-table :data="users" style="width: 100%">
      <el-table-column prop="id" label="用户ID" width="150"> </el-table-column>
      <el-table-column prop="name" label="头像" width="100">
        <template slot-scope="scope">
          <img
            width="30px"
            :src="
              scope.row.portrait ||
              'https://cube.elemecdn.com/9/c2/f0ee8a3c7c9638a54940382568c9dpng.png'
            "
          />
        </template>
      </el-table-column>
      <el-table-column prop="name" label="用户名"> </el-table-column>
      <el-table-column prop="phone" label="手机号"> </el-table-column>
      <el-table-column prop="createTime" label="注册时间">
        <template slot-scope="scope">
          <span>{{ scope.row.createTime | dateFormat }}</span>
        </template>
        <!-- 用户状态操作（服务端没有开放权限，只能演示） -->
      </el-table-column>
      <el-table-column prop="name" label="状态" width="80">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.status"
            active-value="ENABLE"
            inactive-value="DISABLE"
            active-color="#13ce66"
            inactive-color="#ff4949"
            @change="handleForbidUser(scope.row)"
          >
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column prop="address" label="操作">
        <template slot-scope="scope">
          <el-button type="text" @click="handleSelectRole(scope.row)"
            >分配角色</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <el-dialog title="分配角色" :visible.sync="dialogVisible" width="50%">
      <span>这是一段信息</span>
      <el-select v-model="roleIdList" multiple placeholder="请选择">
        <!-- 根据 options 遍历生成选项列表 -->
        <el-option
          v-for="item in roles"
          :key="item.value"
          :label="item.name"
          :value="item.id"
        >
        </el-option>
      </el-select>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="handelAllocRole">确 定</el-button>
      </span>
    </el-dialog>
  </el-card>
</template>

<script>
import { getUserPages, forbidUser } from '@/services/user'
import dateFormat from '@/utils/dateFormat'
import { getAllRoles, allocateUserRoles, getUserRoles } from '@/services/role'
export default {
  name: 'UserList',
  data () {
    return {
      users: [],
      filterParams: {
        currentPage: 1,
        pageSize: 100,
        phone: '',
        startCreateTime: '',
        endCreateTime: '',
        rangeDate: []
      },
      isLoading: true,
      dialogVisible: false,
      roles: [],
      roleIdList: [],
      currentRoleID: null
    }
  },
  created () {
    this.loadUsers()
  },
  methods: {
    async loadUsers () {
      this.isLoading = true
      const { rangeDate } = this.filterParams
      console.log(rangeDate)
      if (rangeDate && rangeDate.length) {
        this.filterParams.startCreateTime = rangeDate[0]
        this.filterParams.endCreateTime = rangeDate[1]
      } else {
        this.filterParams.startCreateTime = ''
        this.filterParams.endCreateTime = ''
      }
      const { data } = await getUserPages(this.filterParams)
      this.users = data.data.records
      this.isLoadin = false
    },
    async handleForbidUser (user) {
      const { data } = await forbidUser(user.id)
      // 接口禁用，这里没法使用
      console.log(data)
    },
    handleQuery () {
      this.filterParams.currentPage = 1
      this.loadUsers()
    },
    handleReset () {
      this.$refs['filter-form'].resetFields()
      this.loadUsers()
    },
    async handleSelectRole (userData) {
      this.dialogVisible = true
      this.currentRoleID = userData.id
      const { data } = await getAllRoles()
      console.log(data)
      if (data.code === '000000') {
        this.roles = data.data
      }
      const data2 = (await getUserRoles(userData.id)).data
      console.log(data2)
      this.roleIdList = data2.data.map((item) => item.id)
    },
    // 提交角色分配
    async handelAllocRole () {
      const { data } = await allocateUserRoles({
        userId: this.currentRoleID,
        roleIdList: this.roleIdList
      })
      if (data.code === '000000') {
        this.$message.success('分配角色成功')
        this.dialogVisible = false
      }
    }
  },
  filters: {
    dateFormat
  }
}
</script>
