关于vue+elementUI使用中的备忘记录：

author：薛一帆

1.三目运算符修改对表单传值用法

    <el-table-column align="center" prop="mobile" label="手机号">
        <template slot-scope="scope">
          <div>
            {{scope.row.mobile == null?'无':scope.row.mobile}}
          </div>
        </template>
    </el-table-column>

2.vue for循环列表动态绑定值用法
	
	<span v-for="point in dataList" :style='{bottom:point.y + "px",left:point.x + "px"}'>
          {{point.name}}
    </span>

3.vue websocket的用例

	data() {
      return {
        dataList:[],
        websock: null,
        reconnectData: null,
        lockReconnect: false,    //避免重复连接，因为onerror之后会立即触发 onclose
        timeout: 10000,          //10s一次检测
        timeoutObj: null,
        serverTimeoutObj: null,
      }
    },
	created() {
       this.initWebSocket();
    },
    methods: {
      //初始化weosocket
      initWebSocket() {
        console.log('启动中')
        let ws = 'ws://192.168.1.163:5555/websocket/location';
        this.websock = new WebSocket(ws);
        this.websock.onopen = this.websocketonopen;          //连接成功
        this.websock.onmessage = this.websocketonmessage;    //广播成功
        this.websock.onerror = this.websocketonerror;        //连接断开，失败
        this.websock.onclose = this.websocketclose;          //连接关闭
      },
      //连接成功
      websocketonopen() {
        console.log('连接成功')
      },
      //连接失败
      websocketonerror() {
        console.log('连接失败')
        this.reconnect();
      },
      //各种问题导致的 连接关闭 调用重连
      websocketclose() {
        console.log('断开连接');
        this.reconnect();
      },
      //数据接收 连通后消息处理
      websocketonmessage(strigData) {
        let data = JSON.parse(strigData.data);
        console.log(data);
      },
      //数据发送
      websocketsend(data) {
        this.websock.send(JSON.stringify(data));
      },
      //socket重连
      reconnect() {
        if (this.lockReconnect) {       //这里很关键，因为连接失败之后之后会相继触发 连接关闭，不然会连接上两个 WebSocket
          return
        }
        this.lockReconnect = true;
        this.reconnectData && clearTimeout(this.reconnectData);
        this.reconnectData = setTimeout(() => {
          this.initWebSocket();
          this.lockReconnect = false;
        }, 5000)
      }
    },
    destroyed() {
      this.lockReconnect = true;
      this.websock.close();                   //离开路由之后断开websocket连接
      clearTimeout(this.reconnectData);      //离开清除 timeout
      clearTimeout(this.timeoutObj);         //离开清除 timeout
      clearTimeout(this.serverTimeoutObj);   //离开清除 timeout
    }
    }

4.多语言切换的功能实现demo

	/**
	 * author：xyf
	 语言包文件相关说明：
	 1.需要引入vue-i18n相关的包，用npm命令安装即可 npm install vue-i18n
	 2.需要将要多语言化的相关字段展示放到js文件里面（一种语言写一个，类似script定义全局变量）
	 3.vue-i18n提供一个全局的locale参数，改变locale可以切换语言（修改方式：this.$i18n.locale）
	 */

	import Vue from 'vue'   // 引入Vue
	import VueI18n from 'vue-i18n' // 引入i18n
	import locale from 'element-ui/lib/locale' // 引入element 国际化配置
	
	import CN from './chinese' //自定义中文语言文件
	import EN from './english' // 自定义英文语言文件
	
	Vue.use(VueI18n)  // 混入Vue
	
	// 创建实例并且挂在自定义语言包
	const i18n = new VueI18n({
	  locale: 'cn', // 默认语言为中文
	  messages: {
	    cn: CN,
	    en: EN
	  }
	})
	
	locale.i18n((key, value) => i18n.t(key, value)) // 把element 的语言包挂在到i18n中
	
	export default i18n // 导出实例

对应的china.js做个实例demo说明

	// 引入element 中文包
	import cnLocale from 'element-ui/lib/locale/lang/zh-CN'
	// 导出为meaasges
	export default {
	  common: {
	    homepage: '首页',
	    logout: '登出'
	  },
	  menu:{
	    firstMenu:{
	      people:"人员管理",
	      attendance:"考勤管理",
	      property:"资产管理",
	      area:"区域管理",
	      patrol:"巡更管理",
	      position:"定位消息",
	      warning:"报警管理",
	      system:"系统管理",
	      device:"设备管理"
	    },
	    secondMenu:{
	      people_1:"人员信息",
	      attendance_1:"考勤记录",
	      attendance_2:"考勤规则",
	      property_1:"资产信息",
	      area_1:"区域信息",
	      patrol_1:"巡更记录",
	      patrol_2:"巡更任务",
	      position_1:"实时定位",
	      position_2:"历史轨迹",
	      warning_1:"报警记录",
	      warning_2:"报警规则",
	      system_1:"系统设置",
	      system_2:"参数设置",
	      system_3:"日志管理",
	      device_1:"服务器管理",
	      device_2:"工作站管理",
	      device_3:"基站管理",
	      device_4:"摄像机管理",
	      device_5:"标签管理",
	      device_6:"门禁管理",
	      device_7:"语音屏管理",
	    },
	    thirdMenu:{
	      system_1_1:"机构管理",
	      system_1_2:"用户管理",
	      system_1_3:"角色管理",
	      system_1_4:"菜单管理",
	      system_2_1:"参数管理",
	      system_3_1:"操作日志",
	      system_3_2:"标签日志",
	      device_5_1:"手环型",
	      device_5_2:"安全帽型",
	      device_5_3:"资产型",
	      device_5_4:"工牌型",
	      device_5_5:"车载型",
	    }
		  },
		  audit: {
		    mobile: '手机号：',
		    app: '注册来源：',
		    submit: '提交',
		    error: {
		      empty: '请输入手机号',
		      uncorrect: '请输入正确的手机号'
		    }
		  },
		  ...cnLocale // 混入element 中文包
		}

页面中的调用方式（示例）

	{{$i18n.t("menu.firstMenu.people")}}

需要在main.js文件中引入

	import i18n from './language/i18n' //引入i8n配置
	
	new Vue({
	  el: '#app',
	  router,
	  i18n,
	  components: { App },
	  template: '<App/>'
	})

5.封装请求服务的方法demo
 先写http.js文件（向外开放函数）

	import axios from 'axios'
	import qs  from 'qs'
	
	// post 方法
	export function post(url, params) {
	  return new Promise((resolve, reject) => {
	    axios.post(url, qs.stringify(params))
	      .then(res => {
	        resolve(res.data);
	      })
	      .catch(err =>{
	        reject(err)
	      })
	  });
	}
	// get
	export function get(url, params){
	  return new Promise((resolve, reject) =>{
	    axios.get(url, {
	      params: params
	    }).then(res => {
	      resolve(res.data);
	    }).catch(err =>{
	      reject(err)
	    })
	  });
	}
 
 main.js中引入

	import {get,post} from './components/Service/http'; //导入请求封装的请求方式

	//封装的get post请求
	Vue.prototype.$get = get;
	Vue.prototype.$post = post;

页面里的调用方式demo

	 this.$post("http://localhost:5555/sysUser/selectUserPage",{}).then((res)=>{
          this.tableData = res.data.list;
          this.total = res.data.total;
        })

6.vue关于拦截器的简单设置

	//登录拦截器（可以写在main.js、route.js等文件内）
	router.beforeEach((to, from, next) => {
	  let isLogin = window.sessionStorage.getItem('isLogin');
	  if (to.meta.requireAuth){
	    if (isLogin == "true"){
	      next();
	    }else{
	      next({
	        path: '/Login'  // 重新获取token的页面
	      })
	    }
	  }
	  else{
	    next();
	  }
	});

   说明：在这里设置的beforeEach是在路由跳转的时候触发的函数


	//在路由里面需要配置的内容（meta属性）
	 {
        name: 'AttendanceRecord',
        path: '/',
        meta:{
          requireAuth: true
        },
        component: AttendanceRecord
      },

7.vue中引用子组件以及传值的方法eg：

	import KeepdetailedCom from "@/components/PatrolManage/PatrolRecordCom.vue";
	// 导出的是组件选项
	export default {
	  name: "PropertyType",
	  components: {
	    KeepdetailedCom
	  }...

  调用方法：

	<KeepdetailedCom :detailed_id="table_led_id"></KeepdetailedCom>

  说明：detailed_id传给子组件的字段，table_led_id是父页面传递的值


8.vue中用session存储对象之前需要先转为string的json，在读取的时候再转成json
	
	sessionStorage.setItem("data",JSON.stringify(res.data));
	 //JSON.stringify(res.data) 将data转为字符串

  在相关读取页面转换成json
	
	JSON.parse(sessionStorage.getItem("data"));//转换json后方便调用属性

9.vue打包中相关注意事项

  <1>首先找到config目录下的index.js文件（dev是开发环境，build是构建版本）
	
		build: {
		    // Template for index.html
		    index: path.resolve(__dirname, '../zhgc/index.html'),
		
		    // Paths
		    assetsRoot: path.resolve(__dirname, '../zhgc'),
		    assetsSubDirectory: 'static',
		    assetsPublicPath: './', 
		  }
	
   说明：在assetsPublicPath属性中将‘/’改为‘./’（修改访问的方式，不然会是空白页）

  <2>在css中声明的 background-image 图片不显示的解决办法：找到build目录下的utils.js文件

		if (options.extract) {
	      return ExtractTextPlugin.extract({
	        use: loaders,
	        fallback: 'vue-style-loader',
	        publicPath:'../../' //这里是新增的
	      })
	    } else {
	      return ['vue-style-loader'].concat(loaders)
	    }

   说明：找到如图所示的代码 新增 publicPath:'../../' 重新打包图片即可展示

	