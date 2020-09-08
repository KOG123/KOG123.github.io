# 笔记

## 插件

vuepress ===> vue官方md转html

### 字符串转数字

- 纯数字字符串 ==》 对应数字
- 带有非数字 ==》 0
- boolean ==》 true:1 false:0
- 特殊类型 ==》 转 boolean 为 true:1 false:0 例:~~undefined => 0

### Content-Type

- application/json => json 格式
- application/x-www-form-urlencoded => 键值对格式
- Content-Type: multipart/form-data => 一般用于上传文件

### 面包屑实现

![image-20200814101019394](/Users/zhanggai/Library/Application Support/typora-user-images/image-20200814101019394.png)

 在 route 中配置 meta 属性，并给 meta 属性赋值

```js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from "views/Home.vue";

Vue.use(VueRouter)

/**
 * 使用递归来保证多层children的情况能够被遍历
 */
function recursion(routes) {
  let routeArr = routes.map(route => {
    if (!route.component) {
      let url = route.url;
      route = {
        path: route.path,
        name: route.name,
        component: () => import( /* webpackChunkName: "[request]" */ `views/${url}.vue`),
        meta: route.meta
      };
    }
    if (route.children) {
      route.children = recursion(route.children);
    }
    return route;
  });
  return routeArr;
}

/**
 * @param {String} path: 路由路径
 * @param {String} name: 路由名称
 * @param {Any} component: 非懒加载页面
 * @param {Array} children: 子路由
 * @param {String} url: 懒加载页面地址
 */
let routerConfig = [{
  path: "/",
  name: "home",
  component: Home
}, {
  path: "/breadCrumd",
  name: 'breadCrumd',
  url: 'breadCrumd',
  meta: {
    title: '面包屑',
    breadCrumd: [{
        path: '/',
        name: '首页'
      },
      {
        path: '/breadCrumd',
        name: '面包屑'
      }
    ]
  }
}];

let routes = recursion(routerConfig);

const router = new VueRouter({
  base: process.env.VUE_APP_PUBLICPATH,
  saveScrollPosition: true,
  mode: 'history',
  routes
})

//能避免如蜜传如蜜导致错误
const originalPush = VueRouter.prototype.push
VueRouter.prototype.push = function push(location) {
  return originalPush.call(this, location).catch(err => err)
}

export default router
```

```vue
<template>
  <div class="about">
    <a-breadcrumb>
      <a-breadcrumb-item v-for="(item, index) in pathArr" :key="index"
        ><a :href="item.path">{{ item.name }}</a></a-breadcrumb-item
      >
    </a-breadcrumb>
  </div>
</template>

<script>
  export default {
    data: () => ({
      pathArr: [],
    }),
    created() {
      //取出meta中存储的面包屑信息
      this.pathArr = this.$route.meta.breadCrumd;
    },
  };
</script>
```

### moment.js 时间戳与时间转换

 moment.js 提供 unix 方法来实现时间转时间戳

```js
moment("2020-01-01 00:00:00").unix(); //log:当前时间时间戳 1577808000
```

 但是打印的时间戳为 10 位,而 moment()解析的位数为 13 位时间戳

```js
moment(1597809182).format(); //log:1970-01-19T14:16:48+08:00
```

 直接使用就会导致打印的时间为 1970...导致时间不准确，如果想要正确使用就得先将获取的时间戳乘 1000，会存在一些误差

```js
moment(moment().unix() * 1000).format(); //log:2020-01-01T00:00:00+08:00
```

### Element UI table 组件 空白内容填充 --

 使用 element ui 提供的插槽，并接收返回的值 scope。

```vue
<el-table :data="tableData" :fit="true" style="width: 100%">
	<el-table-column
                   v-for="item in colums"
                   :key="item.prop"
                   :prop="item.prop"
                   :width="item.width"
                   :show-overflow-tooltip="true"
                   :label="item.label"
                   >
    <template slot-scope="scope">
			{{isBlank}}
    </template>
  </el-table-column>
</el-table>
```

```js
isBlank(scope) {
  let key = scope.column.property,
      item = scope.row[key],
      value = '---';
  if (item === 0 || item === false || item) {
    value = item;
  }
  return value;
}
```

 scope.row 接收当前行，scope.cloumn.property 属性对应的是当前单元格的属性名。使用 scope.row[scope.cloumn.property]获取到的是单一单元格的值，然后再做判断并赋值。

### VUE 插槽使用

 插槽相当于占位，在组组件中的固定位置预定一个位置，当我们调用组件的时候好能够在那个位置插入我们想要补充的内容。

*V-Input.vue*

```vue
<div>
  <input v-model="inp"/>
  <slot name="btn" :data="inp"></slot>
</div>
```

```js
export default{
  data:()=>{
    inp: ''
  }
}
```
​	name属性定义具名插槽，一般用于单组件具有多插槽。意思就是这里是某某地，不设置就是默认插槽。还可以通过v-bind绑定属性传递给父组件。

*Father.vue*

```vue
<div>
  <v-input>
  	<template #btn="btnProps">
			<button @click="serch(btnProps.data)">
        serch
      </button>
    </template>
  </v-input>
</div>
```

```js
export default{
  methods:{
    serch(payload){
      console.log(payload);
    }
  }
}
```

​	父组件使用具名插槽`v-slot:name`表示我要到某某地方去。接收插槽参数是使用`v-slot:name=props`然后使用`props.data`,这里`props`是自己任意起名的，`data`也是任意起名，但是要和你在子组建中设置的一致。

### js if语句 判断条件父级不存在报错

​	当判断多层对象的是否为true时，如果判断条件的父级不错在会导致报错`Cannot read property 'XXX' of undefined`。

```javascript
let obj = {a:1}

if(obj.a.s){
   console.log(obj);
}	//正常

if(obj.b.s){
  console.log(obj);
}	//报错 Cannot read property 's' of undefined

if(obj.b && obj.b.s){
  console.log(obj);
}	//正常 
```

​		因为&&语句只有在前面为true的时候才会继续向后判断，当前面语句为false的时候则会被阻断。所以当 obj.b 不存在时就不会继续执行。

### css实现画面居中

```css
.father{
	display:flex;
}
.soon{
	margin:auto
}
```

