<template>
  <div id="app">
    <VFormDesigner ref="vfDesignerRef" :server-dsv="serverDsv" :designerConfig="{
      formTemplates: formTemplates,
    }">
      <!--
      <template #customToolButtons>
        <el-button type="text" @click="doTest">测试btn</el-button>
      </template>
      -->
    </VFormDesigner>

  </div>
</template>

<script>
import VFormDesigner from './components/form-designer/index.vue'

export default {
  name: 'App',
  components: {
    VFormDesigner,
  },
  data() {
    return {
      //后台服务配置(查询用户/角色/部门等)
      serverDsv: {
        base: 'http://127.0.0.1:9912/office',
        user: '/pub/data/user/list',  //查询用户
        unit: '/pub/data/unit/list',  //查询单位部门
        role: '/pub/data/role/list',  //查询角色
        file: '/pub/upload/file',     //文件上传
        image: '/pub/upload/image',   //图片上传
        field: '/pub/data/field/1',    //查询表字段+表单id
        headers: {
          'X-Token': ''
        },
        params: {
          'companyId': ''
        }
      },
      /* filed 返回数据格式
          {
          "code": 0,
          "msg": "操作成功",
          "time": 1714448941456,
          "data": [
              {
                  "code": "name",
                  "name": "姓名"
              },
              {
                  "code": "age",
                  "name": "年龄"
              }
          ]
      }
      */
      formTemplates:  [
            {
              title: '单列表单',
              imgUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/t1.png',
              jsonUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/json1.txt',
              description: '表单模板详细说明...'
            },

            {
              title: '多列表单',
              imgUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/t2.png',
              jsonUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/json2.txt',
              description: '表单模板详细说明...'
            },

            {
              title: '分组表单',
              imgUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/t3.png',
              jsonUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/json3.txt',
              description: '表单模板详细说明...'
            },

            {
              title: '标签页表单',
              imgUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/t4.png',
              jsonUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/json4.txt',
              description: '表单模板详细说明...'
            },

            {
              title: '主从表单',
              imgUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/t5.png',
              jsonUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/json5.txt',
              description: '表单模板详细说明...'
            },

            {
              title: '响应式表单',
              imgUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/t6.png',
              jsonUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/json6.txt',
              description: '表单模板详细说明...'
            },

            {
              title: '问卷调查表',
              imgUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/t7.png',
              jsonUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/json7.txt',
              description: '表单模板详细说明...'
            },

            {
              title: '固定表格表单',
              imgUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/t8.png',
              jsonUrl: 'https://ks3-cn-beijing.ksyuncs.com/vform-static/form-samples/json8.txt',
              description: '表单模板详细说明...'
            },

      ]


    }
  },
  computed: {
    //
  },
  methods: {
    doTest() {
      let fieldList = this.$refs.vfDesignerRef.getFieldWidgets(null, true)
      console.log('test', fieldList)
    }

  }
}
</script>

<style lang="scss">
  #app {
    height: 100%;
  }
</style>
