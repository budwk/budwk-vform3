<template>
  <div id="app">
    <VFormDesigner ref="vfDesignerRef" :global-dsv="globalDsv" :server-dsv="serverDsv" :designerConfig="{
      formTemplates: formTemplates,
      uploadHeaders: uploadHeaders,
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
      //全局数据源变量
      globalDsv: {
        testApiHost: 'http://127.0.0.1:9912/office',
        testPort: 8080,
      },
      //后台服务配置(查询用户/角色/部门)
      serverDsv: {
        base: 'http://127.0.0.1:9912/office',
        user: '/pub/data/user/list',
        unit: '/pub/data/unit/list',
        role: '/pub/data/role/list',
        headers: {
          'X-Token': 'test-token'
        },
        params: {
          'companyId': ''
        }
      },
      uploadHeaders: {
        'X-Token': 'test-token'
      },
      formTemplates: [
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
          }
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
