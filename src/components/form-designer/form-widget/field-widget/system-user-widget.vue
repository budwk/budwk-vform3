<template>
  <form-item-wrapper :designer="designer" :field="field" :rules="rules" :design-state="designState"
                     :parent-widget="parentWidget" :parent-list="parentList" :index-of-parent-list="indexOfParentList"
                     :sub-form-row-index="subFormRowIndex" :sub-form-col-index="subFormColIndex" :sub-form-row-id="subFormRowId">
    
    <el-tag v-for="(obj,idx) in fieldModel" :key="'user_'+idx" style="margin: 0 5px;" type="primary" :closable="field.options.clearable" @close="removeUser(obj)">
      {{ obj.username }}
    </el-tag> 
    <el-button style="margin: 0 5px;"  v-if="field.options.multiple || (!field.options.multiple && fieldModel.length<1)" type="info" text @click="showDialogSelect">
      <el-icon color="#409efc" class="no-inherit">
        <CirclePlus />
      </el-icon>
    </el-button>
    <el-dialog v-model="showDialog" width="60%">
      <el-container>
        <el-aside width="220px" style="border-right: 1px solid #dcdfe6;">
          <el-tree style="padding: 5px;" :default-expanded-keys="expandedKeys"
          node-key="id"
:data="unitTree" :props="{ label: 'name', children: 'children' }"
                        :expand-on-click-node="false" ref="unitTreeRef"
                        highlight-current @node-click="handleNodeClick">
                        <template #default="{ node, data }">
                            <el-icon v-if="data.type && data.type.value === 'GROUP'" class="no-inherit">
                              <HomeFilled />
                            </el-icon>
                            <el-icon v-if="data.type && data.type.value === 'COMPANY'" class="no-inherit">
                              <HomeFilled />
                            </el-icon>
                            {{ data.name }}
                        </template>
                    </el-tree>
        </el-aside>
        <el-main>
          <el-row>
            <el-input v-model="searchValue" placeholder="请输入名称或用户名" clearable>
              <template #append>
                <el-button @click="searchUser" icon="el-icon-search"></el-button>
              </template>
            </el-input>
          </el-row>
          <el-table :data="userList" stripe border @selection-change="handleSelectionChange">
            <el-table-column v-if="field.options.multiple==true" type="selection" width="55"></el-table-column>
            <el-table-column prop="username" label="名称"></el-table-column>
            <el-table-column prop="loginname" label="用户名"></el-table-column>
            <el-table-column v-if="field.options.multiple==false" label="操作" align="center">
              <template #default="{ row }">
                <el-button text type="primary" @click="selectUserOne(row)">选中</el-button>
              </template>
            </el-table-column>
          </el-table>
          <el-row style="text-align: right;">
            <el-pagination style="padding: 5px 0;margin-right: 20px;" v-model:current-page="pageNo" 
          v-model:page-size="pageSize" :total="userTotal" layout="total, prev, pager, next"
          @change="getUser"
          ></el-pagination>
            <el-button style="margin: 5px 0;" v-if="field.options.multiple==true" type="primary" @click="selectUserMore">选择</el-button>
          </el-row>
    
        </el-main>
      </el-container>
    </el-dialog>
  </form-item-wrapper>
</template>

<script>
  import FormItemWrapper from './form-item-wrapper'
  import emitter from '@/utils/emitter'
  import {handleTree} from '@/utils/util'
  import i18n, {translate} from "@/utils/i18n";
  import fieldMixin from "@/components/form-designer/form-widget/field-widget/fieldMixin";
  import SvgIcon from "@/components/svg-icon/index";
  import axios from 'axios'

  export default {
    name: "system-user-widget",
    componentName: 'FieldWidget',  //必须固定为FieldWidget，用于接收父级组件的broadcast事件
    mixins: [emitter, fieldMixin, i18n],
    props: {
      field: Object,
      parentWidget: Object,
      parentList: Array,
      indexOfParentList: Number,
      designer: Object,

      designState: {
        type: Boolean,
        default: false
      },

      subFormRowIndex: { /* 子表单组件行索引，从0开始计数 */
        type: Number,
        default: -1
      },
      subFormColIndex: { /* 子表单组件列索引，从0开始计数 */
        type: Number,
        default: -1
      },
      subFormRowId: { /* 子表单组件行Id，唯一id且不可变 */
        type: String,
        default: ''
      },

    },
    components: {
      FormItemWrapper,
      SvgIcon,
    },
    data() {
      return {
        oldFieldValue: [], //field组件change之前的值
        fieldModel: [],
        rules: [],
        showDialog: false,
        expandedKeys: [],
        unitList: [],
        unitTree: [],
        userList: [],
        userTotal: 0,
        pageNo: 1,
        pageSize: 10,
        searchValue: '',
        path: '',
        selectUsers: []
      }
    },
    computed: {
      inputType() {
        if (this.field.options.type === 'number') {
          return 'text'  //当input的type设置为number时，如果输入非数字字符，则v-model拿到的值为空字符串，无法实现输入校验！故屏蔽之！！
        }

        return this.field.options.type
      },

    },
    beforeCreate() {
      /* 这里不能访问方法和属性！！ */
    },

    created() {
      /* 注意：子组件mounted在父组件created之后、父组件mounted之前触发，故子组件mounted需要用到的prop
         需要在父组件created中初始化！！ */
      this.initFieldModel()

      this.registerToRefList()
      this.initEventHandler()
      this.buildFieldRules()

      this.handleOnCreated()

    },

    mounted() {
      this.handleOnMounted()
    },

    beforeUnmount() {
      this.unregisterFromRefList()
    },

    methods: {
      handleSelectionChange(data) {
        this.selectUsers = data
      },
      removeUser(user) {
        // 从fieldModel删除user
        this.fieldModel = this.fieldModel.filter(obj => obj.id !== user.id)
      },
      showDialogSelect() {
        this.showDialog = true
        const serverDsv = this.getServerDsv()
        // 获取单位部门数据
        axios({
          url: serverDsv.base + serverDsv.unit,
          method: 'get',
          headers: serverDsv.headers,
          params: serverDsv.params
        }).then(res => {
          if (!!res.data.code) {
            this.$message.error('没有查询到单位数据')
            return
          }
          this.unitList = res.data.data
          this.unitTree = handleTree(this.unitList)
          this.expandedKeys = [this.unitList[0].id]
          this.getUser('') // 获取用户数据
        }).catch(error => {
          this.$message.error('查询单位数据出错')
        })
      }, 
      getUser(){
        const serverDsv = this.getServerDsv()
        axios({
          url: serverDsv.base + serverDsv.user,
          method: 'get',
          headers: serverDsv.headers,
          params: {
            companyId: serverDsv.params.companyId,
            path: this.path,
            name: this.searchValue,
            pageNo: this.pageNo,
            pageSize: this.pageSize
          }
        }).then(res => {
          if (!!res.data.code) {
            this.$message.error('没有查询到用户数据')
            return
          }
          this.userList = res.data.data.list
          this.userTotal = res.data.data.totalCount
        }).catch(error => {
          this.$message.error('查询用户数据出错')
        })
      },
      searchUser() {
        this.pageNo = 1
        this.getUser()
      },
      selectUserOne(user) {
        this.fieldModel = [{
          id: user.id,
          username: user.username,
          loginname: user.loginname
        }]
        this.showDialog = false
      },
      selectUserMore(){
        if(this.selectUsers.length<1){
          this.$message.error('请选择用户')
          return
        }
        this.fieldModel.push(...this.selectUsers.map(obj => {
          return {
            id: obj.id,
            username: obj.username,
            loginname: obj.loginname
          }
        }))
        this.showDialog = false
      },
      handleNodeClick(data) {
        this.path = data.path
        this.getUser()
      },
    }
  }
</script>

<style lang="scss" scoped>
  @import "../../../../styles/global.scss"; /* form-item-wrapper已引入，还需要重复引入吗？ */

</style>
