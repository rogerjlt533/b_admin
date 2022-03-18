<template>
    <div>
        <el-dialog title="发送公告" :visible="notice.show" @close="closeNotice">
            <el-input v-model="notice.title" placeholder="请输入标题"></el-input>
            <el-input type="textarea" :rows="2" v-model="notice.content"></el-input>
            <br><br>
            <el-row>
                <el-select v-model="notice.search_item" style="width: 40%;">
                    <el-option
                            v-for="item in notice.users"
                            :key="item.value"
                            :label="item.label"
                            :value="item.value">
                    </el-option>
                </el-select>
                <el-input v-model="notice.search_value" placeholder="请输入" style="width: 30%;">
                    <i slot="suffix" class="el-input__icon el-icon-search" @click="initNoteUserList"></i>
                    <i slot="suffix" class="el-input__icon el-icon-s-comment" @click="selectedUser"></i>
                </el-input>
            </el-row>
            <br>
            <el-row v-for="sitem in notice.selected" :key="sitem.label">
                <span>{{ sitem.label }}</span><span @click="removeNoteUser(sitem)"> >删除</span>
            </el-row>
            <br>
            <el-row>
                <el-button type="primary" @click="sendNotice">提 交</el-button>
            </el-row>
        </el-dialog>
        <el-tag>用户列表</el-tag>
        <el-form :inline="true" style="padding-top: 5px; padding-bottom: 5px;">
            <el-row>
                <el-col :span="4" :offset="1">
                    <label style="float: left;">用户名:</label>
                    <el-input v-model="userName" placeholder="请输入用户名" style="width: 70%;"></el-input>
                </el-col>
                <el-col :span="3">
                    <label style="float: left;">性别:</label>
                    <el-select v-model="sex" placeholder="请选择" style="width: 70%;">
                        <el-option
                                v-for="item in sexes"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                        </el-option>
                    </el-select>
                </el-col>
                <el-col :span="3">
                    <label style="float: left;">状态:</label>
                    <el-select v-model="useStatus" placeholder="请选择" style="width: 70%;">
                        <el-option
                                v-for="item in useStatusList"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                        </el-option>
                    </el-select>
                </el-col>
                <el-col :span="1">
                    <el-button type="primary" @click="getData(1)">查询</el-button>
                </el-col>
                <el-col :span="2" v-show="false">
                    <span @click="initNoticeOption">发送公告</span>
                </el-col>
            </el-row>
        </el-form>
        <el-table :data="tableData" style="width: 100%">
            <el-table-column prop="id" label="ID" width="60"></el-table-column>
            <el-table-column prop="user_cardno" label="编号" width="80"></el-table-column>
            <el-table-column prop="name" label="用户名" width="80"></el-table-column>
            <el-table-column label="头像" width="60">
                <template slot-scope="scope">
                    <img :src="scope.row.image" width="60" height="60"/>
                </template>
            </el-table-column>
            <el-table-column prop="sex" label="性别"></el-table-column>
            <el-table-column prop="title" label="标题"></el-table-column>
            <el-table-column prop="phone" label="联系电话"></el-table-column>
            <el-table-column prop="email" label="E-mail"></el-table-column>
            <el-table-column prop="area_desc" label="地址"></el-table-column>
            <el-table-column prop="use_status_desc" label="用户状态"></el-table-column>
            <el-table-column prop="created_time" label="创建时间"></el-table-column>
            <el-table-column prop="sort_time" label="活跃时间"></el-table-column>
        </el-table>
        <div class="tabListPage">
            <el-pagination @size-change="handleSizeChange"
                           @current-change="handleCurrentChange"
                           :current-page="currentPage"
                           :page-sizes="pageSizes"
                           :page-size="PageSize" layout="total, sizes, prev, pager, next, jumper"
                           :total="totalCount">
            </el-pagination>
        </div>
    </div>
</template>

<script>
export default {
  name: 'UserList',
  data () {
    return {
      userName: '',
      sex: 3,
      sexes: [{
        label: '全部',
        value: 3
      }, {
        label: '男',
        value: 1
      }, {
        label: '女',
        value: 2
      }, {
        label: '未知',
        value: 0
      }],
      useStatus: 1,
      useStatusList: [{
        label: '全部',
        value: 2
      }, {
        label: '正常',
        value: 1
      }, {
        label: '注销',
        value: 2
      }],
      notice: {
        show: false,
        title: '',
        content: '',
        search_item: '',
        search_value: '',
        selected: []
      },
      // 总数据
      tableData: [],
      // 默认显示第几页
      currentPage: 1,
      // 总条数，根据接口获取数据长度(注意：这里不能为空)
      totalCount: 0,
      // 个数选择器（可修改）
      pageSizes: [10, 20, 30, 40],
      // 默认每页显示的条数（可修改）
      PageSize: 10
    }
  },
  methods: {
    getData (page, size) {
      size = size > 0 ? size : this.PageSize
      const options = {
        method: 'POST',
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
        body: new URLSearchParams({ name: this.userName.trim(), sex: this.sex, use_status: this.useStatus, page, size }).toString()
      }
      fetch('/admin/user/list', options)
        .then(res => res.json())
        .then(json => {
          if (json.code === 200) {
            this.tableData = json.data.list
            this.totalCount = json.data.total
          }
        })
        .catch((err) => {
          console.error(err)
        })
    },
    // 每页显示的条数
    handleSizeChange (val) {
      // 改变每页显示的条数
      this.PageSize = val
      // 点击每页显示的条数时，显示第一页
      // 注意：在改变每页显示的条数时，要将页码显示到第一页
      this.getData(1, this.PageSize)
      this.currentPage = 1
    },
    // 显示第几页
    handleCurrentChange (val) {
      // 改变默认的页数
      this.currentPage = val
      // 切换页码时，要获取每页显示的条数
      this.getData(
        this.getData(val, this.PageSize))
    },
    initNoticeOption () {
      this.notice.title = ''
      this.notice.content = ''
      this.notice.search_value = ''
      this.notice.selected = []
      this.notice.users = []
      this.notice.search_item = ''
      this.notice.show = true
    },
    closeNotice () {
      this.notice.show = false
    },
    removeNoteUser (obj) {
      this.notice.selected.splice(obj, 1)
    },
    initNoteUserList () {
      const options = {
        method: 'POST',
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
        body: new URLSearchParams({ name: this.notice.search_value }).toString()
      }
      fetch('/admin/user/options', options)
        .then(res => res.json())
        .then(json => {
          if (json.code === 200) {
            this.notice.users = json.data.list
            console.log(this.notice.users)
          } else {
            this.notice.users = []
          }
        })
        .catch((err) => {
          console.error(err)
        })
    },
    selectedUser () {
      for (let uindex in this.notice.users) {
        if (this.notice.users[uindex].value === this.notice.search_item) {
          this.notice.selected.push({label: this.notice.users[uindex].label, value: this.notice.users[uindex].value})
          break
        }
      }
    },
    sendNotice () {
      const users = []
      for (let uindex in this.notice.selected) {
        users.push(this.notice.selected[uindex].value)
      }
      const options = {
        method: 'POST',
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
        body: new URLSearchParams({ title: this.notice.title, content: this.notice.content, users: users.join() }).toString()
      }
      fetch('/admin/user/notice/send', options)
        .then(res => res.json())
        .then(json => {
          if (json.code === 200) {
            this.$message({
              message: '发送成功',
              type: 'success',
              center: true
            })
          } else {
            this.$message({
              message: json.message,
              type: 'error',
              center: true
            })
          }
        })
        .catch((err) => {
          console.error(err)
        })
    }
  },
  created: function () {
    this.getData(1, this.PageSize)
  }
}
</script>

<style scoped>

</style>
