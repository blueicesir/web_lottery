### Vue札记，根据lottery的项目学习Vue

v-for="(val,key) in dicts"
@click="onClick(val)"
class="pure-button"
:id="getBtnID(index)"
:class="{'button-error':getBtnID(key)==selected_id}"

{{应用值}}

### 创建一个Vue实例 vm ViewModel
var vm=new Vue({})

var vm = new Vue({
  data: data
})


访问data中的数据
vm.data // 无需使用

只有在创建时就已经存在于data中的属性才支持响应式，如果在运行时新添加一个属性是不会触发视图中绑定的值的更新

data : {
  newTodoText: '',
  visitCount: 0,
  hideCompletedTodos: false,
  todos: [],
  error: null
}

如果调用了
OBject.freeze()
会阻止修改现有的属性，系统无法最终变化。


数据可以放在Vue初始化函数之外
var obj ={
  foo:'bar'
}

Object.freeze(obj)

new Vue({el:'#app',data:obj})

<div id="app">
<p>{{foo}}</p>
<!-- 这里的 foo 不会更新 -->
<button v-on:click="foo =='baz'">Change it</button>
</div>


除了数据属性之外，Vue还暴露了一些实例属性与方法。他们都有前缀$，以便于用户属性区分开来。
例如：
var data={a:1}
var vm=new Vue({el:'#example',data:data})

vm.$data===data
vm.$el===document.getElementById('example')

// $watch是一个实例方法
vm.$watch('a',function(newValue,oldValue){
// 这个回调将在`vm.a`改变后调用
})


// 实例生命周期钩子，每个Vue实例在被创建时都要经过一系列过程
// 设置数据监听、编译模板、将实例挂载到DOM并在数据变化时更新DOM。

还有一些生命周期钩子的函数，可以在不同阶段添加自己代码的机会。
例如：created钩子可以用来在一个实例被创建之后执行代码
new Vue({
  data: {
    a:1
  },
  created:function(){
    // `this`
    console.log('a is:'+this.a)
  },
  mounted:function(){
    // 生命周期钩子的this上下文指向调用它的Vue
  },
  updated:function(){
    // 更新完成
  },
  destroyed:function(){
    // 销毁对象
  }
})


** 不要在选项属性或回调函数上使用箭头函数 **
`
created:()=>console.log(this.a)

`
或
`
vm.$watch('a',newValue=>this.myMethod())
`
因为箭头函数没有this，会导致变量一直向上级词法作用域查找。


![Vue初始化流程](https://cn.vuejs.org/images/lifecycle.png ''Vue初始化流程'')



