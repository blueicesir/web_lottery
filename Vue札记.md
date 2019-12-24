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
  beforemount:function(){
  },
  mounted:function(){
    // 生命周期钩子的this上下文指向调用它的Vue
  },
  beforeupdate:function(){
  },
  updated:function(){
    // 更新完成
  },
  beforedestroy(){
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


## 模板语法
Vue.js使用了基于HTML的模板语法，允许开发者声明式的将DOM绑定至底层Vue实例的数据，所有Vue.js的模板都是合法的HTML.

在底层的实现上，Vue将模板编译成虚拟DOM的渲染函数，职能的计算减少渲染组件，并减少DOM更新次数。

如果熟悉DOM并想使用Javascript原生的力量，除了可以使用模板之外，还可以直接写渲染函数render，使用可选的JSX语法。


### 插值
文本 - 数据绑定最常见的形式就是使用双大括号的文本插值
<span>Message: {{ msg }} </span>
这样可以直接使用data中的msg属性的值，会自动绑定。
也可以使用v-once指令，进行首次绑定，后续不在跟踪变化和更新
<span v-once>first init value : {{ msg }} </span>

### 原始HTML
双大括号会将数据解释为普通文本，而非HTML代码，当需要输出HTML时需要送v-html指令：
<p>Using mustaches: {{ rawHTML }} </p>
<p>Using v-html directive:<span v-html="rawHtml"></span></p>

v-html指定了需要显示的html代码关联的变量值，注意：v-html不能用来符合局部模板，Vue不是基于字符串的模板引擎，对于用户界面更新，组件更适合用来作为可重用和可组合的基本单元。


Mustache大括号的方法不能用于HTML特性属性上，这时需要使用v-bind指令：
<div v-bind:id="dynamicId"></div>

对于布尔特性，v-bind工作起来略有不同：
<button v-bind:disabled="isButtonDisabled">Button</button>

如果isButtonDisabled值是null\undefined或false，则disabled属性不会被包含在渲染出来的button元素中。
即布尔属性对于v-bind只有输出或不输出对应的属性，而其他特性会成为特性值的一部分。


### 使用Javascript表达式，Vue.js在数据绑定都提供了完整的Javascript表达式支持，例如：
`
{{ number +1 }}
{{ ok ?'YES':'NO' }}
{{ message.split('').reverse().join('') }}
<div v-bind:id="'list-'+id"></div>
`

* 所有的v-bind等属性都必须包含在""双引号中作为一个整体。 *

但有一个箱子，即：绑定只能包含单个表达式，必须是表达式，而不是语句。
{{ var a=1 }} 这种是不行的。
{{ if (ok) {return message} }} 流程控制也不允许，这种情况请使用三元表达式。

模板表达式被放在沙盒中，因此只文芳部分全局变量，例如Math,Date。不应该在模板表达式中访问用户自定义的全局变量。


### 指令 Directives
指令是带有 v- 前缀的特殊特性，指令特性的预期是单个 JavaScript 表达式，v-for是唯一的例外。
指令的职责是但表达式的值改变时，将其产生的两代影响，响应式地作用于DOM.
例如：
<p v-if="seen">now you see me!</p>

这里到的 v-if 将根据表达式 seen的值真假来插入或移除<p>元素，而不仅仅是p元素内的文本。


### 指令的参数，一些指令能接收一个参数，在指令名称之后以冒号表示，例如v-bind可以响应式的更新html的特性：
<a v-bind:href="url">..</a>
之类href就是参数，变量绑定的就是href的值与url变量进行绑定。

同样
<a v-on:click="doSomething">...</a>
这里v-on参数就是click是事件类型。


### 动态参数，从 Vue.js 2.6.0 开始新增了动态参数，可以用方括号括起来的Javascript表达式作为一个指令的参数
<a v-bind:[attributeName]="url">...</a>
[]方括号表示这个参数使用的是动态表达式，attributeName会被当做一个JavaScript表达式动态求职，所得到的值将最终作为参数来使用。例如attributeName变量的值为"href"这等价于v-bind:href

我们可以用这个参数可以动态绑定事件：
<a v-on:[eventName]="doSomething">...</a>
当eventName="focus"等价于直接写<a v-on:focus="doSomething"></a>

### 动态参数的值为null时，用于移除绑定，其它非字符串类型的值将会触发一个警告。

同时动态参数不能使用例如空格和引号，因为这些不是HTML attribute中的有效例如：
<a v-bind:['foo'+ bar] ="value">... </a>  表达式中不应有单引号以及空格，这里我们可以使用计算属性替代这种复杂和不合规的表达式。




在DOM中使用模板时（直接在HTML中撰写模板），需要避免使用大写字符来命名键名，因为浏览器会把attribute名称全部强制转变为小写。

例如：
<a v-bind:[someAttr]="value">...</a>
会自动变为someattr属性。


## 修饰符 modifier 是半角的句号 . 指明的特殊后缀，用于指出指令应该以特殊方式绑定。
例如：.prevent修饰符告诉v-on指令对于触发的事件调用event.preventDefault(
<form v-on:submit.prevent="onSubmit">...</form>

修饰符主要用于v-on和v-for等功能中。

### 缩写
v- 前缀作为一种视觉提示，用来识别模板中的Vue特定的特性。
Vue.js对常用的v-bind和v-on这两个指令提供了简写：
<a v-bind:href="url">...</a>
<a :href="url">...</a>

v-on缩写
<a v-on:click="doSomething">...</a>
<a @click="doSometing">...</a>

v-bind缩写为:开头的内容。 
v-on缩写为@开头的内容。 


### 计算属性和侦听器
计算属性 模板内的表达式非常便利，但是设计他们的初衷用于简单的运算，在模板汇总然如太多的逻辑会让模板过重且难以维护。

例如下面这个例子，逻辑国产，且无法复用。
<div id="example">
{{ message.split('').reverse().join('') }}
</div>

这时我们应该使用计算属性：

计算属性的版本：
var vm=new Vue({
  el:"#example",
  data:{
    message:'Hello'
  },
  computed: {
    reversedMessage:function(){
      return this.message.split('').reverse().join('')
    }
  }
})


## 计算属性封装在computed中。
计算属性在模板中直接使用计算属性的名称即可使用，计算属性依赖于data中的变量做计算。


### 计算属性缓存vs方法

计算属性依赖于我们在data定义的变量，如果data定义的计算属性相关联的变量没有变化，计算属性不会变化，且计算属性的值也不会发生变化。且计算属性的值是有缓存的。

例如如下的计算属性不会发生变化，第一次缓存结果之后，由于计算属性中没有关联有data中的值引起的变化，后续获取的就是第一次计算的记过。
computed : {
  now:function(){
    return Date.now()
  }
}

而如果写成函数，每次触发时都会再次执行函数，重新渲染时计算结果不会缓存。
如果不喜欢有缓存，请使用方法来替代。

## 计算属性 vs 侦听属性

Vue提供了一种跟通用的方法来观察和响应Vue实例上的数据变动：侦听属性。
通常你有一些数据需要随着其它数据变动而变化时，你很容易滥用watch，通常更好的做法是使用了计算属性而不是命令式的watch回调。

<div id="demo">{{ fullName }} </div>

var vm=new Vue({
  el:'#demo',
  data:{
    firstName:'Foo',
    lastName:'Bar',
    fullName:'Foo Bar'
  },
  // 监听属性
  watch:{
    firstName:function(val){
      this.fullName=val+' '+this.lastName
    },
    lastName:function(val){
      this.fullName=this.firstName+' '+val
    }
  }
})

上面代码是命令且重复的，下面使用计算属性的版本来进行比较：

var vm=new Vue({
  el:'#demo',
  data:{
    firstName:'Foo',
    lastName:'Bar'
  },
  computed:{
    fullName:function(){
      return this.firstName+' '+this.lastName
    }
  }
})

## 计算属性比上面的watch监听函数要简洁很多，隐含的只要firstName和lastName两个属性只要发生变化就自动计算fullName的值。



## 计算属性的setter
计算属性默认只有getter，开发者可以在需要时提供一个setter，例如：
computed:{
  fullName:{
    get:function(){
      return this.firstName+' '+this.lastName
    },
    set:function(newValue){
      var names=newValue.split(' ')
      this.firstName=names[0]
      this.lastName=names[names.length-1]
    }
  }
}

# 运行 vm.fullName='BlueICE SecondName'时，setter会被调用，vm.firstName和vm.lastName也会被响应更新。

## 侦听器，虽然计算属性大多数情况下更核实，但有时我们也需要自定义侦听过期，就需要使用Vue的watch提供的更通用的方法来响应数据的变化。在数据变化时执行异步或开销较大的操作时，这个操作方式方式更有效。
<div id="watch-example">
  <p>
  Ask a yes/no question:
  <input v-model="question"> <!-- v-model表示绑定data中的变量  -->
  </p>
  <p>{{ answer }} </p>
</div>

<script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM=new Vue({
  el:'#watch-example',
  data:{
    question:'',
    answer:'I cannot give you an answer until you ask q question!'
  },
  watch:{
    question:function(newQuestion,oldQuestion){
      this.answer='Waiting for you to stop typing...'
      this.debouncedGetAnswer()
    }
  },
  created:function(){
    // _.debounce是一个通过lodash限制操作频率的函数，这个例子汇总我们希望限制yesno.wtf/api的访问频率
    // Ajax请求指导用户输入完毕才发出，在https://lodash.com/docs#debounce
    this.debouncedGetAnswer=_.debounce(this.getAnswer,500) // 这里调用了我们自定义的getAnswer方法。
  },
  methods:{
    getAnswer: function(){
      if(this.question.indexOf('?')===-1){
        this.answer='Questions usually contain a question mark. ;-)'
        return
      }
    this.answer="Thinking..."
    var vm=this
    // 通过axios发起Ajax请求,异步操作，这些
    axios.get('https://yesno.wtf/api')
      .then(function(response){
        vm.answer=_.captialize(response.data.answer)
      })
      .catch(function(error){
        vm.answer='Error!Could not reach the API.'+error
      })
      
    }

  }
})
</script>



## Class 与 Style 绑定


