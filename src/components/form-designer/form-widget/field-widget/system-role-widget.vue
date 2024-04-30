<template>
  <form-item-wrapper :designer="designer" :field="field" :rules="rules" :design-state="designState"
                     :parent-widget="parentWidget" :parent-list="parentList" :index-of-parent-list="indexOfParentList"
                     :sub-form-row-index="subFormRowIndex" :sub-form-col-index="subFormColIndex" :sub-form-row-id="subFormRowId">
    
    <el-tag v-for="(obj,idx) in fieldModel" :key="'role_'+idx" style="margin: 0 5px;" type="primary" :closable="field.options.clearable && !field.options.disabled" @close="removeRole(obj)">
      {{ obj.name }}
    </el-tag> 
    <el-button style="margin: 0 5px;"  v-if="!field.options.disabled && (field.options.multiple || (!field.options.multiple && fieldModel.length<1))" type="info" text @click="showDialogSelect">
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
            <el-input v-model="searchValue" placeholder="请输入角色名称" clearable>
              <template #append>
                <el-button @click="searchRole" icon="el-icon-search"></el-button>
              </template>
            </el-input>
          </el-row>
          <el-table :data="roleList" stripe border @selection-change="handleSelectionChange">
            <el-table-column v-if="field.options.multiple==true" type="selection" width="55"></el-table-column>
            <el-table-column prop="name" label="角色名称"></el-table-column>
            <el-table-column v-if="field.options.multiple==false" label="操作" align="center">
              <template #default="{ row }">
                <el-button text type="primary" @click="selectOne(row)">选中</el-button>
              </template>
            </el-table-column>
          </el-table>
          <el-row style="text-align: right;" v-if="field.options.multiple==true" >
            <el-button style="margin: 5px 0;" type="primary" @click="selectMore">选择</el-button>
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
    name: "system-role-widget",
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
        roleList: [],
        searchValue: '',
        selectRoles: [],
        companyId: ''
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
        this.selectRoles = data
      },
      removeRole(role) {
        // 从fieldModel删除
        this.fieldModel = this.fieldModel.filter(obj => obj.id !== role.id)
      },
      showDialogSelect() {
        this.showDialog = true
        const serverDsv = this.getServerDsv()
        // 获取单位部门数据
        axios({
          url: serverDsv.base + serverDsv.unit,
          method: 'get',
          headers: serverDsv.headers,
          params: {
            companyId: serverDsv.params.companyId,
          }
        }).then(res => {
          if (!!res.data.code) {
            this.$message.error('没有查询到单位数据')
            return
          }
          this.unitList = res.data.data
          this.unitTree = handleTree(this.unitList)
          this.expandedKeys = [this.unitList[0].id]
          this.companyId = serverDsv.params.companyId
          this.getRole('') // 获取用户数据
        }).catch(error => {
          this.$message.error('查询部门单位出错')
        })
      }, 
      getRole(){
        const serverDsv = this.getServerDsv()
        axios({
          url: serverDsv.base + serverDsv.role,
          method: 'get',
          headers: serverDsv.headers,
          params: {
            companyId: this.companyId,
            name: this.searchValue,
          }
        }).then(res => {
          if (!!res.data.code) {
            this.$message.error('没有查询到角色数据')
            return
          }
          this.roleList = res.data.data
        }).catch(error => {
          this.$message.error('查询角色数据出错')
        })
      },
      searchRole() {
        this.pageNo = 1
        this.getRole()
      },
      selectOne(role) {
        this.fieldModel = [{
          id: role.id,
          name: role.name,
          code: role.code
        }]
        this.showDialog = false
      },
      selectMore(){
        if(this.selectRoles.length<1){
          this.$message.error('请选择角色')
          return
        }
        this.fieldModel.push(...this.selectRoles.map(obj => {
          return {
            id: obj.id,
            name: obj.name,
            code: obj.code,
          }
        }))
        this.showDialog = false
      },
      handleNodeClick(data) {
        this.companyId = data.id
        this.getRole()
      },
    }
  }
</script>

<style lang="scss" scoped>
  @import "../../../../styles/global.scss"; /* form-item-wrapper已引入，还需要重复引入吗？ */

</style>
