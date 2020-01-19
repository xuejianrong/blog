## 简介和使用场景
官方的说法是：有的情况下，你仍然需要对普通 DOM 元素进行底层操作。
所以当我们不熟悉自定义指令的使用，不知道什么时候需要去使用它的时候，就可以在我们想要操作dom的时候去使用它，例如：输入框在打开页面就自动聚焦
``` js
// 注册全局指令 v-focus
Vue.directive('focus', {
  inserted (el, bindings, vnode) {
    el.focus()
  }
})
// 使用
<input v-focus />
```

考虑到，如果是第一次自定义指令这个知识点，很可能不知道这个东西有什么好处。概念性的东西也枯燥，这里先实现一个比较常用的功能之后，再去说明一些概念性的东西

### **click-outside**: 点击其他元素时触发
应用场景：日期选择组件、下拉选择组件等，点击组件外的内容隐藏选择弹窗。
``` html
<style>
  .comp {
    width: 200px;
  }
  .dialog {
    width: 200px;
    height: 200px;
    background: #666;
  }
</style>
<body>
  <div id="app">
    <div class="comp" v-click-outside>
      <input type="text"><br>
      <div class="dialog" v-show="isShow"></div>
    </div>
  </div>
</body>
```
``` js
// 组件内
new Vue({
  el: '#app',
  data: {
    isShow: false
  },
  directives: {
    clickOutside: {
      // 绑定指令时触发
      bind (el, bindings, vnode) {
        // 给el设置fn方法，为了在元素销毁时解绑
        el.fn = function (e) {
          // 判断点击的是组件内还是组件外
          if (el.contains(e.target)) {
            // vnode.context可以放回当前的vue实例
            vnode.context.show()
          } else {
            vnode.context.hide()
          }
        }
        document.addEventListener('click', el.fn)
      },
      // 指令与元素解绑时调用(可以大概理解为元素销毁时触发)
      unbind (el) {
        // 解除事件绑定，如果不解除，bind中的el.fn、el.contains就会出错，控制台一片红
        document.removeEventListener('click', el.fn)
      }
    }
  },
  methods: {
    show () {
      this.isShow = true
    },
    hide () {
      this.isShow = false
    }
  }
})

```
### 钩子函数
- **bind**: 只调用一次，指令第一次绑定到元素时调用。
- **inserted**: 被绑定元素插入父节点时调用
- **update**: 所在组件的 VNode 更新时调用
- **unbind**: 只调用一次，指令与元素解绑时调用

### 参数
- **el**: 指令绑定的元素，可以用来直接操作dom
- **bindings**: 一个包含很多有用信息的对象
- **vnode**: 虚拟节点，可以通过`vnode.context`来获取当前的Vue实例

### 自定义指令的组成部分
这里只讨论常用部分，例如: `v-myDirective:arg.a.b.c="value"`
- **myDirective**: 自定义指令的名称，使用的时候也可以写成：`v-my-directive`
- **arg**: 参数，可通过`bindings.arg`获取
- **a**: 同b、c，修饰符，可通过`bindings.modifiers`获取
- **value**: 指令绑定的值，可通过`bindings.value`获取，顾名思义也可以传入方法直接调用