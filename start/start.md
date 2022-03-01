# 起步

## 1.全局配置

### 1. iview配置

1.1.下载iview

```cmd
npm install view-design@4.5.0 
npm install ElementUI
npm install moment 
```

1.2.在main.js全局引入
```js
import iview from 'view-design' // 引入iview
import ElementUI from 'element-ui' // 引入 ElementUI
import moment from "moment" // 引入 moment
import '@/assets/js/jquery.min.js' // 全局引入jquery
Vue.use(iview).use(ElementUI)
Vue.prototype.$moment = moment
```

### 2.base.js配置


2.1.在public文件夹下新建并配置base.js

```js
const ImsServerConfig = {
    zbgUrl:"http://192.168.37.73:28090",  //执法平台地址
    downloadUrl:  'http://119.23.25.131:8080', //'http://10.89.12.32:8080',  //前端图
    uploadUrl: 'http://119.23.25.131:8080',
    //linux环境 国产影源扫描仪 license
    winMageScanLicense: 'gE/rN4vZATMg49y/OCFlZA==',
    PDF_CONVERT_RATE: 'ws://10.89.12.32:1899', // 获取PDF上传转换进度
    uploadMaxCount:500, // 文件上传 一次上传的文件数量，
    uploadMaxSize: 1024*500, // 单张图片大小最大值
    dmsZrajRule: 'imsRule_zraj' , //电子卷宗昨日案件权限规则
    websocketUrl: 'ws://10.89.12.32:18896', //dms websocket地址
    uploadType: '1',  //文件上传方式  浪潮云:'2' gofast:'1'
    dmsUrl:'http://10.89.12.32:28899', //dms服务地址
    showZnbmSjtb: false,
    bmzjShowPDFUpload:true, // 编目组卷本地上传，true：图片、pdf，false：图片
    isShowDmsSmsc: true, //DMS是否显示扫描上传
    winMageScanUrl: 'http://localhost:18989/WebScan/webScan.html',
    //硬件相关配置
    dwtUrl:'http://192.168.37.73:8830/dwt-service/#/common/scanV2',//扫描服务器地址
    zbgType: '02',  //对接执法模块标识 01海关; 02深圳市局; 03山东; 04广州省厅; 05对接华资; 06mysql中间库
}
```

2.2.在index.html中全局使用base.js
```html
<script type="text/javascript" src="base.js"></script>
```

## 3.vuex数据存储

**3.1 sessionUser（用户信息）**
> vuex需抛出用户信息，getter 方法中，需要sessionUser(用户信息)。组件取值是this.$store.getter.sessionUser 

**3.2 统一请求方式**
> vuex设置通用请求方法和基本信息，store文件夹>mobule>user.js,内容如下：

```js
state: {
    accessUser: '',
    token: getToken(),
},
mutations: {
    setAccessUser (state, data) {
      state.accessUser = data
      state.accessUser.token = state.token
    },
    setToken (state, token) {
      state.token = token
      setToken(token)
    },
},
getters: {
    sessionUser: state => {
      let obj = state.accessUser
      obj.token = state.token
      return obj
    }
}
actions: {
    axiosPostRequest ({ state }, { url, params, options }) {
      let _option = {
        method: 'post',
        url: `${url}`,
        data: Qs.stringify(params)
      }
      if (options) {
        Object.assign(_option, options)
      }
      return axios.request(_option)
    },
    axiosGetRequest ({ state }, { url, params }) {
      return axios.request({
        method: 'get',
        url: `${url}`,
        params
      })
    },
    postRequest ({ state, rootState, dispatch }, { url, params, options }) {
      let _param = {
        access_token: Cookies.get('token'),
        centerId: rootState.centers.currentCenter.id
      }
      if (params) {
        Object.assign(_param, params)
      }
      return new Promise((resolve, reject) => {
        dispatch('axiosPostRequest', { url: url, params: _param, options: options }).then(resp => {
          if (resp.status === 200) {
            resolve(resp.data)
          } else {
            reject(resp)
          }
        })
      })
    },
    getRequest ({ state, rootState, dispatch }, { url, params }) {
      let _param = {
        access_token: Cookies.get('token'),
        centerId: rootState.centers.currentCenter.id
      }
      if (params) {
        Object.assign(_param, params)
      }
      return new Promise((resolve, reject) => {
        dispatch('axiosGetRequest', { url: url, params: _param }).then(resp => {
          if (resp.status === 200) {
            resolve(resp.data)
          } else {
            reject(resp)
          }
        })
      })
    },
    noTokenGetRequest ({ state, dispatch }, { url, params }) {
      let _param = {
      }
      if (params) {
        Object.assign(_param, params)
      }
      return new Promise((resolve, reject) => {
        dispatch('axiosGetRequest', { url: url, params: _param }).then(resp => {
          if (resp.status === 200) {
            resolve(resp.data)
          }
        })
      })
    },
      // 认证中心post请求方法，不以表单形式提交
    authPostRequest({ state, rootState }, { url, params }) {
        let _param = {
          access_token: Cookies.get('token'),
          centerId: rootState.centers.currentCenter.id
        }
        return new Promise((resolve, reject) => {
          axios.request({ method: 'post', url: `${url}`, data: params, params: _param }).then(resp => {
            if (resp.status === 200) {
              resolve(resp.data)
            }
          })
        })
    },
}

```

**3.3 请求示例**

```js

// 请求示例
this.$store.dispatch("postRequest", {
    url: API.DmsApi.DMS_MARKERSLIST,
    params
}).then(res=>{
    // console.log(res)
})

```

### 4 dev代理配置

```js

proxy: {
        '/sundun-ims': {
            target: 'http://10.89.12.32:18096', // 3.0.0
            changeOrigin: true, 
            pathRewrite: {
                '^/sundun-ims': '/'
            }
        },
        '/sundun-dms': {
            target: 'http://10.89.12.32:18896', // 3.0.0
            changeOrigin: true, 
            pathRewrite: {
                '^/sundun-dms': '/'
            }
        },
        '/dms-convert': {
            target: 'http://10.89.12.32:1900', // 2.4
            changeOrigin: true, 
            pathRewrite: {
                '^/dms-convert': '/'
            }
        },
        '/sundun-zfpt': {
            target: 'http://10.89.12.32:28097', // 2.4.2
            changeOrigin: true, //是否跨域,
            pathRewrite: {
                '^/sundun-zfpt': '/'
            }
        },
        '/sundun-upload': {
            target: 'http://10.89.12.32:8080',
            changeOrigin: true, //是否跨域,
            pathRewrite: {
                '^/sundun-upload': '/'
            }
        }
    }
```


















