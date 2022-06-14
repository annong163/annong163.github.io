---
title: Vue在线接口调试组件
author: AnNong
categories: VUE
summary: 支持Token、接口鉴权、参数自定义的Ant Design Vue在线接口调试组件。
tags:
  - VUE组件
  - Ant Design Vue
abbrlink: 2752da05
date: 2022-04-27 09:41:18
---

## 效果图

![](/img/在线调试组件.png)






## 组件: InterfaceTest.vue

```javascript
<template>
  <a-drawer
    title="在线接口测试"
    width="60%"
    placement="right"
    @close="close"
    :destroy-on-close="true"
    :visible="visible">
    <a-card :bordered="false">
      <a-row style="margin-top: 20px">
        <a-col :md="2" :sm="4">
          <a-select style="width: 90px" size="large" default-value="GET" disabled>
            <a-select-option value="GET">GET</a-select-option>
          </a-select>
        </a-col>
        <a-col :md="22" :sm="20">
          <a-input-search
            placeholder="input send url"
            v-model="tokenUrl"
            :expand-depth="5"
            @search="onSearch"
            enterButton="获 取 token"
            size="large" />
        </a-col>
      </a-row>
      <a-row style="margin-top: 20px">
        <a-col :md="2" :sm="4">
          <a-select style="width: 90px" v-model="requestMethod" size="large">
            <a-select-option value="GET">GET</a-select-option>
            <a-select-option value="POST">POST</a-select-option>
            <a-select-option value="PUT">PUT</a-select-option>
            <a-select-option value="DELETE">DELETE</a-select-option>
          </a-select>
        </a-col>
        <a-col :md="22" :sm="20">
          <a-input-search
            placeholder="input send url"
            v-model="url"
            :expand-depth="5"
            @search="onSearch"
            enterButton="发 送 请 求"
            size="large" />
        </a-col>
      </a-row>
      <a-tabs defaultActiveKey="2" style="margin-top: 10px">
        <a-tab-pane tab="Params" key="2">
          <a-textarea style="width:100%;font-size: 16px;font-weight:500;height: 300px;resize: none;" @blur="changeVal" :placeholder="paramsPlace"></a-textarea>
        </a-tab-pane>
      </a-tabs>
      <a-tabs defaultActiveKey="1" style="margin-top: 10px">
        <a-tab-pane tab="Response" key="1">
          <json-viewer
            style="background-color: #E9EBFE;min-height: 300px;"
            :value="resultJson"
            boxed>
          </json-viewer>
        </a-tab-pane>
      </a-tabs>
    </a-card>
  </a-drawer>
</template>
<script>
  import { axios } from '@/utils/request'
  export default {
    name: 'FlowTest',
    data() {
      return {
        url: '',
        tokenStr: '',
        tokenUrl: '',
        paramJson: '参数转换后对应的JSON',
        paramsPlace: '{\n“key1” : “value1”,\n“key2” : value2\n}',
        visible: false,
        resultJson: {},
        requestMethod: 'GET'
      }
    },
    methods: {
      show(record) {
        this.url = process.env.VUE_APP_API_BASE_URL + record.address
        this.tokenUrl = process.env.VUE_APP_API_BASE_URL + '/openApi/refreshToken/' + record.appId + '/lzjy@2021!'
        this.requestMethod = record.method_dictText
        this.visible = true
      },
      onSearch (value) {
        let that = this
        if (!value) {
          that.$message.error('请填写路径')
          return false
        }
        this.resultJson = {}
        console.log(that.tokenStr)
        axios({
            headers: {
              'Access-Token': that.tokenStr
            },
            url: value,
            method: that.requestMethod,
            data: that.paramJson,
            params: that.paramJson
          }).then((res) => {
            that.resultJson = res
            if (value.indexOf('refreshToken') !== -1) {
              that.tokenStr = res.result
              console.log('***********' + that.tokenStr)
            }
          }).catch((err) => {
            console.log(err)
            that.$message.error('请求异常：' + err)
          })
      },
      changeVal(e) {
        try {
          let json = e.target.value
          if (!json) {
            return false
          }
          this.paramJson = JSON.parse(json)
        } catch (e) {
          console.log(e)
          this.$message.error('非法的JSON字符串')
        }
      },
      close () {
        this.resultJson = {}
        this.paramJson = ''
        this.visible = false
      }
    }
  }
</script>



```


