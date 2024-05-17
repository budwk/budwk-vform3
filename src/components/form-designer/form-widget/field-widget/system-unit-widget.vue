<template>
  <form-item-wrapper :designer="designer" :field="field" :rules="rules" :design-state="designState"
                     :parent-widget="parentWidget" :parent-list="parentList" :index-of-parent-list="indexOfParentList"
                     :sub-form-row-index="subFormRowIndex" :sub-form-col-index="subFormColIndex" :sub-form-row-id="subFormRowId">
    <div v-if="previewDetail" class="form-render-content">
       {{ fieldModel.map(obj => obj.name).join('、') }}
    </div>
    <el-tag v-else v-for="(obj,idx) in fieldModel" :key="'role_'+idx" style="margin: 0 5px;" type="primary" :closable="field.options.clearable && !field.options.disabled" @close="removeRole(obj)">
      {{ obj.name }}
    </el-tag> 
    <el-button style="margin: 0 5px;"  v-if="!previewDetail && !field.options.disabled &&(field.options.multiple || (!field.options.multiple && fieldModel.length<1))" type="info" text @click="showDialogSelect">
      <el-icon color="#409efc" class="no-inherit">
        <CirclePlus />
      </el-icon>
    </el-button>
    <el-dialog v-model="showDialog" width="50%">
      
          <el-row>
            <el-input v-model="searchValue" placeholder="请输入单位名称" clearable>
              <template #append>
                <el-button @click="searchUnit" icon="el-icon-search"></el-button>
              </template>
            </el-input>
          </el-row>
          <el-table :data="unitTree" row-key="id"
            :tree-props="{ children: 'children', hasChildren: 'hasChildren' }"
            stripe border @selection-change="handleSelectionChange">
            <el-table-column v-if="field.options.multiple==true" type="selection" width="55"></el-table-column>
            <el-table-column prop="name" label="单位名称">
              <template #default="{ row }">
                <el-icon v-if="row.type && row.type.value === 'GROUP'" class="no-inherit">
                              <HomeFilled />
                   </el-icon>
                <el-icon v-if="row.type && row.type.value === 'COMPANY'" class="no-inherit">
                  <HomeFilled />
                </el-icon>
                {{ row.name }}
              </template>
            </el-table-column>
            <el-table-column prop="type" label="单位类型">
              <template #default="{ row }">
                {{ row.type?.text }}
              </template>
            </el-table-column>
            <el-table-column v-if="field.options.multiple==false" label="操作" align="center">
              <template #default="{ row }">
                <el-button text type="primary" @click="selectOne(row)">选中</el-button>
              </template>
            </el-table-column>
          </el-table>
          <el-row style="text-align: right;" v-if="field.options.multiple==true" >
            <el-button style="margin: 5px 0;" type="primary" @click="selectMore">选择</el-button>
          </el-row>
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
    name: "system-unit-widget",
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
        oldFieldValue: null, //field组件change之前的值
        fieldModel: [],
        rules: [],
        showDialog: false,
        expandedKeys: [],
        unitList: [],
        unitTree: [],
        roleList: [],
        searchValue: '',
        selectUnits: [],
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
        this.selectUnits = data
      },
      removeRole(role) {
        // 从fieldModel删除
        this.fieldModel = this.fieldModel.filter(obj => obj.id !== role.id)
      },
      showDialogSelect() {
        this.showDialog = true
        // 获取单位部门数据
        this.searchUnit()
      }, 
      searchUnit() {
        const serverDsv = this.getServerDsv()
        axios({
          url: serverDsv.base + serverDsv.unit,
          method: 'get',
          headers: serverDsv.headers,
          params: {
            companyId: serverDsv.params.companyId,
            name: this.searchValue
          }
        }).then(res => {
          if (!!res.data.code) {
            this.$message.error('没有查询到单位数据')
            return
          }
          this.unitList = res.data.data
          this.unitTree = handleTree(this.unitList)
        }).catch(error => {
          this.$message.error('查询部门单位出错')
        })
      },
      selectOne(unit) {
        this.fieldModel = [{
          id: unit.id,
          name: unit.name,
          path: unit.path
        }]
        this.syncUpdateFormModel(this.fieldModel)
        this.showDialog = false
      },
      selectMore(){
        if(this.selectUnits.length<1){
          this.$message.error('请选择单位')
          return
        }
        this.fieldModel.push(...this.selectUnits.map(obj => {
          return {
            id: obj.id,
            name: obj.name,
            path: obj.path,
          }
        }))
        this.syncUpdateFormModel(this.fieldModel)
        this.showDialog = false
      },
    }
  }
</script>

<style lang="scss" scoped>
  @import "../../../../styles/global.scss"; /* form-item-wrapper已引入，还需要重复引入吗？ */

</style>
