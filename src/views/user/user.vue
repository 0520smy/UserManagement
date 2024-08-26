<template>
  <div class="user-container">
    <el-form :model="tableData" label-width="80px" :inline="true" class="form-container">
      <el-form-item>
        <el-input v-model="tableData.name" placeholder="请输入用户名称" clearable="" class="custom-input"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" plain icon="el-icon-search" @click="getUserList">搜索</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="warning" plain icon="el-icon-refresh" @click="resetSearch">重置</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" plain icon="el-icon-plus" @click="addUser">新增</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="danger" plain icon="el-icon-delete" @click="batchDeleteUser">删除</el-button>
      </el-form-item>
    </el-form>

    <el-table
      :data="tableData.list"
      @selection-change="val => tableData.selection = val"
      style="width: 100%; margin-top: 20px;font-size: 20px">
      <el-table-column type="index" width="60"></el-table-column>
      <el-table-column type="selection" width="50"></el-table-column>
      <el-table-column prop="userName" label="用户名"/>
      <el-table-column prop="gender" label="性别"/>
      <el-table-column prop="age" label="年龄"/>
      <el-table-column prop="nativePlace" label="籍贯"/>
      <el-table-column prop="phoneNumber" label="手机号" />
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button type="text" icon="el-icon-edit" size="mini" @click="editUser(scope.row)" style="font-size: 20px">编辑</el-button>
          <el-button type="text" icon="el-icon-delete" size="mini" @click="deleteUser([scope.row.id])" style="font-size: 20px; color:red">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      class="pagination custom-pagination"
      :current-page.sync="tableData.pageNum"
      :page-sizes="[5, 10, 15, 20, 25]"
      :page-size.sync="tableData.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="tableData.total"
      @size-change="getUserList"
      @current-change="getUserList"
    >
    </el-pagination>

    <el-dialog class="user-edit-dialog" :title="userEditForm.id ? '编辑信息' : '新增用户'" :visible.sync="userEditDialogVisible" width="50%" top="8vh">
      <el-form ref="userEditForm" status-icon :model="userEditForm" label-width="80px" :rules="userEditForm.id ? userUpdateRules : userCreateRules">
        <el-form-item prop="userName" label="用户名">
          <el-input v-model="userEditForm.userName"></el-input>
        </el-form-item>
        <el-form-item label="性别">
          <el-radio-group v-model="userEditForm.genderRatio">
            <el-radio :label="0">男</el-radio>
            <el-radio :label="1">女</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item prop="age" label="年龄">
          <el-input v-model="userEditForm.age"></el-input>
        </el-form-item>
        <el-form-item prop="nativePlace" label="籍贯">
          <el-input v-model="userEditForm.nativePlace"></el-input>
        </el-form-item>
        <el-form-item prop="phoneNumber" label="手机号">
          <el-input v-model="userEditForm.phoneNumber"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="userEditDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>

export default {
  name: 'User',
  data() {
    return {
      allUserData: [
        { id: 1, userName: '张怎', gender: '男', age: 34, nativePlace: '北京市朝阳区', phoneNumber: 12345678901 },
        { id: 2, userName: '刘划', gender: '女', age: 24, nativePlace: '北京市海淀区', phoneNumber: 12342434341 },
        { id: 3, userName: '王怎', gender: '女', age: 22, nativePlace: '北京市朝阳区', phoneNumber: 12334222424 },
        { id: 4, userName: '李划', gender: '女', age: 28, nativePlace: '天津市津南区', phoneNumber: 17989834343 },
        { id: 5, userName: '陈怎', gender: '男', age: 23, nativePlace: '江苏省南京市', phoneNumber: 14523223233 },
        { id: 6, userName: '于划', gender: '女', age: 19, nativePlace: '河南省洛阳市', phoneNumber: 12123332232 }
      ],
      tableData: {
        name: '',
        list: [],
        selection: [],
        pageNum: 1,
        pageSize: 10,
        total: 0
      },
      userEditForm: {
        id: '',
        userName: '',
        genderRatio: '',
        age: '',
        nativePlace: '',
        phoneNumber: ''
      },
      userEditDialogVisible: false,
      userCreateRules: {
        userName: [{ required: true, message: '用户名不能为空', trigger: 'blur' }],
        age: [
          { pattern: /^[0-9]+$/, message: '年龄必须是数字', trigger: 'blur' }
        ],
        phoneNumber: [
          { required: true, message: '手机号不能为空', trigger: 'blur' },
          { pattern: /^[0-9]+$/, message: '手机号必须是数字', trigger: 'blur' }
        ]
      },
      userUpdateRules: {
        userName: [{ required: true, message: '用户名不能为空', trigger: 'blur' }],
        age: [
          { type: 'number', message: '年龄必须是数字', trigger: 'blur' }
        ],
        phoneNumber: [
          { required: true, message: '手机号不能为空', trigger: 'blur' },
          { type: 'number', message: '手机号必须是数字', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    getUserList() {
      const filteredData = this.allUserData.filter(user => user.userName.includes(this.tableData.name))
      this.tableData.total = filteredData.length
      this.tableData.list = filteredData.slice(
        (this.tableData.pageNum - 1) * this.tableData.pageSize,
        this.tableData.pageNum * this.tableData.pageSize
      )
    },
    resetSearch() {
      this.tableData.name = ''
      this.tableData.list = this.allUserData.slice(0, this.tableData.pageSize)
      this.tableData.total = this.allUserData.length
      this.tableData.pageNum = 1
    },
    addUser() {
      this.userEditForm = {
        id: '',
        userName: '',
        genderRatio: '',
        age: '',
        nativePlace: '',
        phoneNumber: ''
      }
      this.userEditDialogVisible = true
    },
    batchDeleteUser() {
      const ids = this.tableData.selection.map(item => item.id)
      if (ids.length === 0) {
        this.$message.warning('请选择要删除的用户')
        return
      }
      this.deleteUser(ids)
    },
    addOrUpdateUser() {
      this.$refs.userEditForm.validate(valid => {
        if (valid) {
          const gender = this.userEditForm.genderRatio === 0 ? '男' : '女'

          if (this.userEditForm.id) {
            const index = this.tableData.list.findIndex(user => user.id === this.userEditForm.id)
            const allUserIndex = this.allUserData.findIndex(user => user.id === this.userEditForm.id)
            if (index !== -1) {
              this.tableData.list[index] = { ...this.userEditForm, gender }
            }
            if (allUserIndex !== -1) {
              this.allUserData[allUserIndex] = { ...this.userEditForm, gender }
            }
          } else {
            const newUser = { ...this.userEditForm, gender, id: this.tableData.total + 1 }
            this.allUserData.push(newUser)
            this.tableData.total = this.allUserData.length

            const lastPage = Math.ceil(this.tableData.total / this.tableData.pageSize)
            if (this.tableData.list.length >= this.tableData.pageSize) {
              this.tableData.pageNum = lastPage
              this.tableData.list = this.allUserData.slice(
                (lastPage - 1) * this.tableData.pageSize,
                lastPage * this.tableData.pageSize
              )
            } else {
              this.tableData.list.push(newUser)
            }
          }
          this.userEditDialogVisible = false
        }
      })
    },
    editUser(user) {
      for (const key in this.userEditForm) {
        if (key === 'genderRatio') {
          if (user.gender === '男') {
            this.userEditForm.genderRatio = 0
          } else {
            this.userEditForm.genderRatio = 1
          }
        } else {
          this.userEditForm[key] = user[key]
        }
      }
      this.userEditDialogVisible = true
    },
    deleteUser(ids) {
      if (ids.length === 0) {
        this.$message.warning('请选择要删除的用户')
        return
      }
      this.$confirm('确定要删除选中的用户吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.allUserData = this.allUserData.filter(user => !ids.includes(user.id))
        this.tableData.total = this.allUserData.length
        this.getUserList()
        this.$message.success('删除成功')
      }).catch(() => {})
    }
  },
  mounted() {
    this.tableData.list = this.allUserData.slice(0, this.tableData.pageSize)
    this.tableData.total = this.allUserData.length
    this.getUserList()
  }
}
</script>

<style scoped>

.user-container {
  padding: 20px;
  background-color: #f5f5f5;
  border-radius: 8px;
}

.form-container {
  background-color: #ffffff;
  padding: 10px;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  height: 60px;
}

.el-form-item {
  margin-right: 10px;
}

.el-table {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
}

.table-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
}

.pagination {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
}
.custom-pagination {
  font-size: 18px;
}

.custom-input {
  font-size: 18px;
}
.el-dialog__header {
  background-color: #409EFF;
  color: #ffffff;
  border-radius: 8px 8px 0 0;
}

.dialog-footer {
  text-align: right;
  padding: 10px 20px;
}

.el-dialog__body {
  padding: 20px;
}

.el-button--primary {
  background-color: #409EFF;
  border-color: #409EFF;
  color: #ffffff;
  font-size: 16px;
}

.el-button--danger {
  background-color: #F56C6C;
  border-color: #F56C6C;
  color: #ffffff;
  font-size: 16px;
}

.el-button--warning {
  background-color: #E6A23C;
  border-color: #E6A23C;
  color: #ffffff;
  font-size: 16px;
}

</style>
