### Vue札记，根据lottery的项目学习Vue  
```
v-for="(val,key) in dicts"  
@click="onClick(val)"  
class="pure-button"  
:id="getBtnID(index)"  
:class="{'button-error':getBtnID(key)==selected_id}"  
```  
{{data中的变量}}  

### 创建一个Vue实例 vm ViewModel  
(```)
var vm=new Vue({})  

var vm = new Vue({  
  data: data  
})  
(```)

访问data中的数据  
vm.data // 无需使用  

只有在创建时就已经存在于data中的属性才支持响应式，如果在运行时新添加一个属性是不会触发视图中绑定的值的更新  
(```)
data : {  
  newTodoText: '',  
  visitCount: 0,  
  hideCompletedTodos: false,  
  todos: [],  
  error: null  
}  
(```)
如果调用了  
`OBject.freeze()`  
会阻止修改现有的属性，系统无法最终变化。  


数据可以放在Vue初始化函数之外  
(```)
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
(```)

除了数据属性之外，Vue还暴露了一些实例属性与方法。他们都有前缀$，以便于用户属性区分开来。  
例如：  
(```)
var data={a:1}  
var vm=new Vue({el:'#example',data:data})  

vm.$data===data  
vm.$el===document.getElementById('example')  

// $watch是一个实例方法  
vm.$watch('a',function(newValue,oldValue){  
// 这个回调将在`vm.a`改变后调用  
})  
(```)

// 实例生命周期钩子，每个Vue实例在被创建时都要经过一系列过程  
// 设置数据监听、编译模板、将实例挂载到DOM并在数据变化时更新DOM。  

还有一些生命周期钩子的函数，可以在不同阶段添加自己代码的机会。  
例如：created钩子可以用来在一个实例被创建之后执行代码  
(```)
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
(```)

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
(```)
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
(```)


## Class 与 Style 绑定  

操作元素的class列表和内联样式是数据绑定的一个常见需求，因为他们都是属性，我们可以使用v-bind处理他们，
只需要通过表达式计算出字符串结果即可，不过由于字符串拼接麻烦且易出错，因此在将v-bind用于class和style时，Vue.js做了专门增强，表达式结果类型除了字符串之外还可以是对象或数组。

## 绑定 HTML Class  

# 对象语法  
我们可以传递一个v-bind:class 一个对象，动态切换class  
`<div v-bind:class="{active:isActive }"></div>`  
v-bind:class 指令也可以与普通的class属性共存：  
`<div class="static" v-bind:class="{ active:isActive , 'text-danger':hasError }"></div>`  

class 可以和v-bind:class进行复核操作，即会操作增加多个属性

和data如下属性进行配合：  
(```)
data:{
  isActive:true,
  hasError:false
}
(```)

### 绑定到计算属性中:  
<div v-bind:class="classObject"></div>

data:{
  isActive:true,
  error:null
},
computed:{
  classObject:function(){
    return {
      active:this.isActive && !this.error,'text-danger':this.error&&this.error.type==='fatal'
    }
  }
}



# 数组语法  
我们可以把一个数组传递给 v-bind:class 以应用一个class列表：
(```)
<div v-bind:class="[activeClass,errorClass]"></div>  

data:{  
  activeClass:'active',  
  errorClass:'text-danger'  
}  
(```)
渲染为：  
<div class="active text-danger"></div>  

也可以使用三元表达式：  
`<div v-bind:class="[isActive ? activeClass :'',errorClass]"></div>  `

# 用在组件上  
当一个自定义组件上使用class属性时，这些class 将被添加到改组件的根元素上面。这个元素上已经存在的class不会被覆盖。  

例如，声明了一个组件，定义组件：Vue.component('组件名称',{组件属性，例如模板等})  
(```)
Vue.component('my-component',{
  template:'<p class="foo bar">Hi</p>  
})  
(```)

然后在使用它的时候添加一些class：  
`<my-component class="baz boo"></my-component>`  

HTML将被渲染为：  
`<p class="foo bar baz boo">Hi</p>`  

对于带数据绑定的class也同样适用（组件也可以使用v-bind:class定义样式）：  
`<my-component v-bind:class="{ active: isActive }"></my-component>`  

### 绑定内联样式  
## 对象语法  
v-bind:style的对象语法十分直观，非常像css,但其实是一个javascript对象。CSS属性名可以用驼峰式命名或短线分隔来命名：  
`<div v-bind:style="{ color:activeColor,fontSize:fontSize+'px'}"></div>`  
(```)
data: {
  activeColor:'red',
  fontSize:30
}  
(```)
直接绑定到一个样式对象通常会更好，会让模板更清晰：  

`<div v-bind:style="styleObject"></div>`  
(```)
data : {
  styleObject:{
    color:'red',
    fontSize:'13px'  
  }  
}  
(```)

同样的，对象语法尝尝结果返回对象的计算属性使用  

## 数组语法  
**v-bind:style** 的数组的语法可以将多个样式对象应用到同一个元素上：  
<div v-bind:style="[baseStyles,overridingStyles]"></div>  

## 自动添加前缀  
当 **v-bind:style** 使用需要添加浏览器引擎前缀的css属性时，如transform，Vue.js最自动检测并添加相应的前缀。  



### 多重值  
从2.3.0开始，你可以为 **style** 绑定中的属性提供一个包含多个值的数组，常用语提供多个带有前缀的值，例如：  ` <div :style="{display:['-webkit-box','-ms-flexbox','flex' }"></div> `  
这样写智慧渲染数组中最有一个被浏览器支持的值，例如您的浏览器不支持flexbox，只会渲染为display:flex。  



# 条件渲染  
*** v-if ***  
v-if 指令用语条件性的渲染一块内容，这块内容只会在指令的表达返回truthy值的时候会被渲染。  
` <h1 v-if="awesome"> Vue is awesome!</h1> `  
也可以用v-else 添加一个else块:  
<h1 v-if='awesome">Vue is awesome!</h1>  
<h1 v-else>Oh no</h1>  

### 在 <template> 元素上使用 v-if 条件渲染分组  
因为 v-if 是一个指令，所以当必须将他添加到一个元素上。但是当多个元素切换呢，此事可以把<template>元素当做不可见的包裹元素，并在上面使用v-if。最终的渲染结果将不包含<template>元素。  
  
(```)
<template v-if="ok">
  <h1>Title</h1>
  <p>page1</p>
  <p>page2</p>
</template>
(```)  

### v-else，可以使用 v-else 指令表示 v-if的else块  

(```)
<div v-if="Math.random() > 0.5">
Now you see me
</div>
<div v-else>
Now you don't
</div>
(```)

**v-else** 元素必须紧跟在 **v-if** 或者  **v-else-if** 的元素后面，否则它将不会被识别。   

### v-else-if 在2.1.0后新增，v-else-if可以连续使用  
(```)
<div v-if="type==='A'">
A
</div>
<div v-else-if="type==='B'">
B
</div>
<div v-else-if="type==='C'">
C
</div>
<div v-else>
Not A/B/C
</div>
(```)
          

### 用key管理可复用的元素  
Vue会尽可能的高效渲染元素，通常会复用已有元素而不是从头开始渲染。
(```)
<template v-if="loginType==='username'">
  <label>UserName</label>
  <inpout placeholder="Enter your username">
    </template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address">
</template>  
(```)  

上面的代码中切换loginType将不会清楚用户已经输入的内容，因为模板中使用了相同的元素<input>不会被替换，仅仅替换它的placeholder属性。  
这样有时不符合我们的需求，因此Vue提供了一种方式来表达这两个元素是完全独立的，不要复用他们，只需添加一个唯一的key属性即可。

(```)

<template v-if="loginType==='username'">
  <label>Uesrname</label>
  <input placeholder="Enter your username" key="username-input">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input">
</template>

(```)

由于增加了不同的 **key** ，输入框在每次切换时都会重新渲染，也不会保留之前的另外一个key的值到新的输入框。  
注意：<label>元素仍旧会被高效的复用，因为他们没有添加 **key** 属性。  


### v-show 另外一个根据条件展示y8uansu的选项是 v-show 指令，用法 如下：  
` <h1 v-show="ok">Hello</h1> `  
不用的是 **v-show** 元素时钟会被渲染并保留在DOM中，v-show只是简单的切换元素的CSS属性 **display** .  

**注意：** ***v-show*** 不支持 *<template>* 元素，也不支持 **v-else**  


### v-if vs v-show  
v-if是真正的条件渲染，因为它会确保在切换过程中条件块内的时间监听器和子组件适当的被销毁和重建。  
v-if也是惰性的，如果在初始渲染时条件为假，则什么也不做，知道条件第一次变为真时，才会开始渲染条件块。  

相比之下 **v-show** 就简单得多，不管初始条件是什么，元素中是会被渲染，并且只是简单基于CSS进行切换display属性。  
一般而言,v-if有更高的切换开销，而v-show 有更高的初始渲染开销，如果是非常频繁的切换建议使用v-show，如果运行时条件变化少，则使用v-if较好。  


### v-if 与 v-for 一起使用  
** 不推荐同时使用 v-if和v-for。**  

当v-if与v-for一起使用时，v-for具有比v-if更高的优先级。  


  

### 列表渲染  
用 v-for 把一个数组对应为一组元素  
我们可以用v-for指令基于一个数组来渲染一个列表。v-for指令需要使用item in items形式的特殊语法，其中items是源数组，item则是被迭代的数组元素的别名。  
(```)
<ul id="example-1">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>

var example1=new Vue({
  el:'#example-1',
  data:{
    items:[
      {message:'Foo'},
      {message:'Bar'}
    ]
  }
})

(```)


在 v-for 块中，我们可以访问所有父作用域的属性。v-for还支持一个可选的第二参数，即当前项的索引。  
(```)

<ul id="example-2">
  <li v-for="(item,index) in items">
    {{ parentMessage}} - {{index}} - {{item.message}}
  </li>
</ul>

var example2=new Vue({
  el:'#example-2',
  data:{
    parentMessage:'Parent',
    items:[
      {message:'Foo'},
      {mesasge:'Bar'},
    ]
  }
})

(```)

** 你也可以使用 of 替代 in 作为分隔符，它更接近Javascript迭代器的语法 **
` <div v-for="item of items"></div> `  


### 在 v-for里使用对象  
你可以使用v-for 来遍历一个对象的属性  
(```)
<ul id="v-for-object" class="demo">
  <li v-for="value in object">
    {{ value }}
  </li>
</ul>

new Vue({
  el:'#v-for-object',
  data:{
    object:{
      title:'How to do lists in Vue',
      author:'Jane Doe',
      pulishedAt:'2016-04-10'
    }
  }
})

(```)

** 你可以提供第二个参数作为 property 名称（键名） **  
(```)
<div v-for="(value,name) in object">
{{name}}:{{value}}
</div>
(```)

** 还可以使用第三个参数作为索引***
(```)
<div v-for="(value,name,index) in object">

</div>
(```)


** 在遍历对象时，会按Object.keys()的结果遍历，但不保证结果在不同的Javascript引擎下都一致 **  

### 维护状态  
当Vue正在更新使用 v-for 渲染的元素列表时，它默认使用就地更新的策略，如果数据项的顺序被改变，Vue将不会移动DOM元素来匹配数据项的顺序，而是就地更新每个元素，并确保他们在每个索引位置正确的渲染。

需要默认修改上述行为特性，需要给Vue一个提示，以便它能跟踪每个节点的省分，从而重用和重新排序现有的元素，你需要为每个项提供一个唯一的Key属性。  

<div v-for="item in items" v-bind:key="item.id">  
</div>  

** 建议尽可能的使用 v-for 时提供的 key attribute，除非遍历输出的DOM内容非常简单，或者是刻意依赖默认行为上获得性能上的提升。**  


不要使用对象或数组之类的非基础类型作为 v-for 的 key。请使用字符串或数字类型的值。  

### 数组更新检测  
#### 编译方法(mutation method)  
Vue将被侦听的数组的变异方法进行包装，所以他们也将会触发视图的更新，这些被包裹和包装的方法如下：  
+ push()
+ pop()
+ shift()
+ unshift()
+ splice()
+ sort()
+ reverse()
  
你可以在浏览器控制台，对前面例子的 items 数组尝试调用变异方法。比如：
` example1.items.push({mesasge:'Baz'}) `  
更新之后会导致前端的list发生变化。  


### 替换数组  
变异方法，顾名思义，会改变调用了这些方法的原始数组，相比之下也有非变异(non-mutating method)方法。
例如:filter(),concat()和slice(),他们不会给变元素数组，而总是反馈一个新数组，当使用非变异方法时，可以用新数组提花那就数组。  
(```)
example1.items=example1.items.filter(function(item){return item.message.match(/Foo/)})
(```)


### 注意事项  
由于javascript的限制，Vue不能检测一下数组的变动：  
+1 但你那你用索引设置一个数组时，例如：vm.items[indexOfItem]=newValue
+2 但你你修改数组的长度时，例如：vm.items.length=newLength


例如：  
var vm=new Vue({
  data:{
    items:['a','b','c']
  }
})

vm.items[1]='x' // 这种方式不是响应式的，Vue无法检测到对应的数据变化
vm.items.length=2 // 这个也不是响应式的。  

为了解决上述问题，可以送如下方式触发响应式状态更新：  
Vue.set(vm.items,indexOfItem,newValue)

或者
vm.items.splice(indexOfItem,1,newValue)

也就是你可以使用实例方法 vm.$set()或者全局伏安法Vue.set()的一个别名  
vm.$set(vm.items.indexOfItem,newValue)
为了解决第二类问题你可以使用splice:   
vm.items.splice(newLength)  
  
  
### 对象变更检测注意事项  
由于Javascript的限制，Vue不能检测到对象属性的添加或删除  
var vm=new Vue({
  data:{
    a:1
  }
})  

vm.a 是响应式的
vm.b=2 不是响应式的，因为在Vue生成实例时它不存在，如果实例已经创建，Vue不允许动态的添加根级别的响应式属性，但是我们可以使用Vue.set(object,propertyname,value)方法向嵌套对象添加行吟诗属性。  


var vm=new Vue({
  data:{
    userProfile:{
      name:'Anika'
    }
  }
})

你可以添加一个新的age属性嵌套到userProfile中。  
Vue.set(vm.userProfile,'age',27)


你还可以使用实例方法：
vm.$set()它只是全局Vue.set()的别名。  
vm.$set(vm.userProfile,'age',27)  

一次添加多个后期响应式属性，可以使用如下方法：
vm.userProfile=Object.assign({},vm.userProfile,{age:27,favoriteColor:'Vue Green'})  


### 显示过滤、排序后的结果  
有时我们需要显示一个数组经过过滤或排序后的版本，而不实际改变或重置原始数据，在这种情况下我们可以创建一个计算属性，来返回过滤或排序后的数组。  
<li v-for="n in evenNumbers">{{n}}</li>

data:{
  numbers:[1,2,3,4,5]
},
computed:{
  evenNumbers:function(){
    return this.numbers.filter(function(number){return number%2===0})
  }
}  

在计算属性不使用的情况下，可以使用一个方法：  
data:{
  numbers:[1,2,3,4,5,6]
},
methods:{
  event:function(numbers){\
    return numbers.filter(function(number){
      return number%2===0
      })
  }
}  


## 在 v-for 里使用值范围  
v-for 也可以接受整数，在这种情况下，它会把模板重复对应次数：  
<div>
  <span v-for="n in 10">{{n}}</span>
</div>



### 在 <template>上使用 v-for  
  
类似于 v-if 你也可以利用带有 v-for 的<template>来循环渲染一段包含多个元素的内容：  
这样template中使用v-for重复li
<ul>
    <template v-for="item in items">
      <li>{{item.msg}}</li>
      <li class="divider" role="presentation"></li>
    </template>
</ul>


**注意我们不推荐在同一元素上使用v-if和v-for**

当v-if和v-for处于同一节点时，v-for拥有更高的的优先级，这意味着v-if将分别重复运行与每个v-for循环中。

<li v-for="todo in todos" v-if"!todo.isComplete">
  {{ todo }}
</li>

上述代码只渲染未完成的todo  
如果你的目的是有条件的跳过循环的执行，那么可以将v-if置于元素（或<template>）上。例如：  
<ul v-if"todos.length">  
  <li v-for="todo in todos">
    {{todo}}
  </li>
</ul>
<p v-else>No todos left!</p>  

### 在组件上使用v-for  
在主定义组件上，你可以像在任何普通元素上一样使用v-for  
<my-component v-for="item in items" :key="item.id"></my-component>  


** 在2.2.0+版本你，但组件上使用 v-for时，key现在是必须的 **
然而，任何数据都不会主动传递到组件中，因为组件有自己独立的作用域。为了把迭代数据传递到组件中，我们需要使用prop：  
<my-component
v-for="(item,index) in items"
v-bind:item="item"
v-bind:index="index"
v-bind:key="item.id"
></my-component>

不自动将item注入到组件里的原因是，这回使得组件与v-for的运行紧密耦合。明确组件数据的来源能够使组件在其它场合重复使用。

----

下面是一个简单todo列表的完整例子：  
<div id="todo-list-example">
   <form v-on:submit.prevent="addNewTodo">
     <label for="new-todo">Add a todo</label>
     <input 
            v-modle="newTodoText"
            id="new-todo"
            placeholder="E.g.Fee the cat"
      >
     <button>Add</button>
  </form>
  <ul>
     <li
         **is="todo-item"**
         v-for="(todo,index) in todos"
         v-bind:key="todo.id"
         v-bind:title="todo.title"
         v-on:remove="todos.splice(index,1)"
       ></li>
  </ul>
</div>   


### 定义组件  
Vue.component('todo-item',{
  template:'\
  <li>\
  {{title}}\
  <button v-on:click="$emit(\'remove\')">Remove</button>\
  </li>\
  ',
  props:['title'] // 这里定义了传入组件的属性
})


new Vue({
  el:'#todo-list-example',
  data:{
    newTodoText:'',
    todos:[
      {id,1,title:'Do the dishes'},
      {id:2,title'take out the trash},
    ],
    nextTodoId:3
  },
  methods:{
    addNewTodo:function(){
      this.todos.push({id:this.nextTodoId++,title:this.newTodoText})
      this.newTodoText=''
    }
  }
})


### 事件处理  
可以使用v-on指令监听DOM事件，并在触发时运行一些Javascript代码。  
<div id="example-1">
  <button v-on:click="counter+=1">Add 1</button>
  <p>The button above has been clicked {{ counter}} times.</p>
</div>

var example1=new Vue({
  el:'#example-1',
  data:{
    counter:0
  }
})

### 事件处理方法
许多事件处理方法更为复杂，卸载v-on指令中是不可行的，因此v-on还可以接受一个需要调用放的名称：  
<div id="example-2">
  <button v-on:click="greet">Greet</button>
</div>

var example2=new Vue({
  el:'#example-2',
  data:{
    name:'Vue.js'
  },
  methods:{
    greet:function(event){
      alert("Hello "+this.name+"!")
      if(event){
        alert(event.target.tagName)
      }
    }
  }
})  


也可以直接调用
example2.greet()  


### 内联处理器中的方法，并传递参数  
<div id="example-3">
  <button v-on:click="say('hi')">Sai hi</button>
  <button v-on:click="say('what'">Say what</button>
</div>

new Vue({
  el:'#example-3',
  methods:{
    say:function(message){
      alert(message)
    }
  }
})  


**有时我们需要在内联语句中原始访问DOM事件，可以使用特殊的变量$event***  
<button v-on:click=warn('FFF'),$event)">Submit</button>  

methods:{
  warn:function(mesasge,event){
    if(event){
      event.preventDefault()
    }
    alert(message)
  }
}



### 事件修饰符  
在事件处理程序中调用 event.preventDefault() 或 event.stopPropagation() 是非常常见的。
建议方法只是纯粹的数据逻辑，而不是去处理DOM事件细节。  
为了解决这个问题，Vue.js为v-on提供了事件修饰符。  
-.stop # 阻止单击事件继续传播 <a v-on:click.stop="doThis"></a>
-.prevent # 提交事件不再重载页面 <form v-on:submit.prevent="OnSubmit"></form>

修饰符可以串联 <a v-on:click.stop.prevent="doThat"></a>  
也可以只有修饰符 例如:<form v-on:click.stop.prevent></form>

-.capture # 添加事件监听器使用事件捕获模式，即内部元素触发的事件先处理，然后才交由内部元素进行处理。
-.self # 只有event.target是当前元素时触发处理函数，即内部元素触发的事件不触发。<div v-on:click.self="doThat">..</div>
-.once # 只会触发一次
-.passive  # Vue 2.3.0新增了passive修饰符，滚动事件会立即触发，而不等待onScroll完成。提高了移动端的性能。

**修饰符顺序很重要，会影响相应的代码产生顺序**
v-on:click.prevent.self 会阻止所有的点击  
v-on:click.self.prevent 智慧阻止对元素自身的点击。  


** 不要把.passive和.prevent一起使用，因为.prevent将会被忽略，同时浏览器可能会提示一个警告 **

**.passive会告诉浏览器你不想组织事件的默认行为**  



### 按键修饰符  
在监听假盘事件时，我们需要详细检查按键，Vue允许为v-on在监听键盘事件时添加按键修饰符：  
<input v-on:keyup.enter="submit">表示只有按回车时响应vm.submit()函数。  
你也可以直接将KeyboardEvent.key暴露的任意有效按键转换为kebab-case来作为修饰符。  
<iniput v-on:keyup.page-down="onPageDown">  
  
### 按键码  
KeyCode的时间已经被废弃了，并可能会被最新的浏览器支持，使用KeyCode特性也是允许的：  
<input v-on:keyup.13="submit">

按键别名：  
+ .enter
+ .tab
+ .delete
+ .space
+ .esc
+ .up
+ .left
+ .right


你还可以通过全局 config.KeyCodes 对象的自定义按键修饰符别名：  
Vue.config.keyCodes.f1=112  

### 系统修饰键  
2.1.0新增可以用如下修饰符来实现仅在按下响应按键时才出发鼠标后键盘事件的监听器。  
.ctrl
.alt
.shift
.meta (OSX 系统对应 command按键，Windows对应视窗按键)



<input @keyup.alt.67="claer"> 表示Alt+C
<div @click.ctrl="doSomething">Do something</div> Ctrl+Click


### .exact 修饰符  
.exact 修饰符允许你控制由精确的系统修饰符组合出发的事件。  
<button @click.ctrl="onClick">A</button> Alt或Shift被同时按下会触发。

<button @click.ctrl.exact="onCtrlClick">A</button> 只有Ctrl被按下时才触发。  

<button @click.exact="onClick">A</button> 没有任何系统修饰符被按下时才触发。


### 鼠标修饰符  
2.2.0新增  
+ .left
+ .right
+ .middle
  
**当一个ViewModel被销毁时，所有的事件处理器都会自动被删除，你无需担心如何清理他们**

### 表单输入绑定  

你可以用 v-model 指定在表单 <input>、<textarea>、<select>元素上创建双向数据绑定。
它会根据空间类型自动选择正确的方法来更新元素。v-model本质上不过是语法糖。  


v-model会忽略所有表单元素的value、checked、selected特性的初始值而总是将Vue实例的数据作为数据来源。
你应该在Javascript在组件中的data选项中声明初始化值。  


v-model在内部为不同的输入元素使用不同的属性并抛出不同的事件:  
+ text和textarea元素使用value属性和input事件。  
+ checkbox和radio使用checked属性和change事件。  
+ select字段将value作为prop并将change作为事件。  

**对于中日韩等输入法，v-model默认不会在输入过程中获得更新，如果需要在输入过程中获取请使用input事件**  

### 文本  
<input v-model="message" placeholder="edit me">  
<p>Message is:{{message}}</p>  

### 多行文本  
<textarea v-model="mesasge" placeholder="add multiple lines"></textarea>  
**在文本区域插值<textarea>{{text}}</textarea>并不会生效，应使用v-model替代。  **

**多选框绑定到同一个数组**

data:{
  checkedNames:[]
}


**多选框应该绑定到 data:{ select:''}字符串类型**

**select多选时绑定到一个数组**  
<div id="example-6">
  <select v-model="selected" multiple style="width:50px;">
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <br>
</div>

new Vue({
  el:"#example-6",
  data:{
    selected:[]
  }
})


用v-for渲染的动态选项：  
<select v-model="selected">
  <option v-for="option in options" v-bind:value="option.value">
    {{option.text}}
  </option>
</select>
<span>Selected:{{selected}}</span>


new Vue({
  el:'...',
  data:{
    selected:'A',
    options:[
      {text:'One',value:'A'},
      {text:'Two',value:'B'},
      {text:'Three',value:'C'}
    ]
  }
})

**v-model="selected"会使用绑定的初始化变量，options选择会绑定选项数组**  

### 复选框  
<input type="checkbox" v-model="toggle" true-value="yes" false-value="no">  
复选框，指定了true-value和false-value，否则默认是布尔值。  

### 单选按钮  
<input type="radio" v-model="pick" v-bind:value="a">  

### 下拉选择框使用字面量，而不是data中的数据选项时  
<select v-model="selected">
  <option v-bind:value="{ number:123}">123</option>
</select>


## 修饰符  
### .lazy  
默认情况下v-model在每次input时间处罚后将输入框的值与数据进行同步，你可以添加lazy修饰符，从而转变为change事件进行同步：  
<input v-model.lazy="msg"> <!-- 使用change事件更新msg变量 -->  

### .number
如果需要将用户输入的值转换为数值类型可以添加number修饰符：  
<input v-model.number="age" type="number">  

### .trim修饰符  
顾名思义，这个修饰符会把输入的前后空格丢弃  
<input v-model.trim="msg">  



## 组件基础  
定义一个组件样例：  
Vue.component('button-counter',{
  data:function(){
    return {
      count:0
    }
  },
  template:'<button v-on:click="count++">Your clicked me {{ count }} times.</button>"
})  
在这个案例中button-counter是自定义组件的名字。  


我们可以在一个通过new Vue创建的Vue根实例中，把这个组件作为自定义元素来使用：  
<div id="componenets-demo">
   <button-counter></button-counter>
</div>
  
new Vue({el:'#components-demo'}

**因为组件是可以复用的Vue实例，所以他们与new Vue接收先用的选项，如data、computed、watch、methods以及生命周期钩子等，仅偶遇一个例外是el这样的根实例特有的选项。** 

**组件内的变量在组件实例中是相互隔离的**  

**组件中的data必须是一个函数，原因是组件的data选项必须是一个函数，因为这样才能返回对象的独立拷贝，如果不是，后续生成的相同组件的实例会共享这个变量。**  

### 组件的组织  
为了能在模板中使用组件，必须先注册以便Vue能够识别，注册分为***全局注册***和***局部注册***。
通过Vue.component('my-component-global',{})注册的组件是全局组件。  


### 通过 Prop 向子组件传递数据  
Prop是你可以在组件上注册的一些自定义特性，但一个值传递给一个prop特性的时候，它就编程了一个组件实例的一个属性，例如给博文组件传递一个标题，我们可以使用props选项将其包含在该组件可接受的prop列表中：  

Vue.component('blog-post',{props:['title'],template:'<h3>{{title}}</h3>})  
一个组件默认可以拥有任意数量的prop，任何值都可以传递给任何prop，在上述模板汇总，你会发现我们能在组件实例中访问这个值，就如访问data中的值一样。  
在一个prop被注册之后，我们可以把数据大工作一个自定义特性传递进来：
<blog-post title="My Name is BlueICE"></blog-post>  



然而在一个典型的应用中，你可能在data里有一个博文的数组：  
new Vue({
  el:'#blog-post-demo',
  data:{
    posts:[
      {id:1,title:'first title'},
      {id:2,title:'second  tilte'},
      {id:3,title:'three title'}
    ]
  }
})

我们为每个博文渲染一个组件：  
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
  ></blog-post>

我们可以使用v-bind来动态传递prop.  


从一个url获取博文的样例：  
<script src="https://unpkg.com/vue"></script>
<div id="blog-post-demo" class="demo">
    <blog-post
       v-for="post in posts"
       v-bind:key="post.id"
       v-bind:title="post.title"
     ></blog-post>
</div>

new Vue({
  el:'#blog-post-demo',
  data:{
    posts:[]
  },
  created:function(){
    var vm=this
    fetch('https://jsonplaceholder.typeicode.com/posts')
      .then(function(response){
        return response.json() // 这个应该是json编码。
      })
      .then(function(data){
        vm.posts=data // 这里更新了vm中的posts数组。
      })
  },
})


**声明组件时把完整post属性传递给组件**

<blog-post 
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:posts="post"
  ></blog-post>  

Vue.component('blog-post',{
  props:['post'],
  template:`
    <div class="blog-post">
      <h3>{{post.title}}</h3>
      <div v-html="post.content"></div>
    </div>
  `


在组件使用时，直接把属性post通过v-bind进行绑定，在组件中通过posts:['post']，传递进来，
这样组件的模板汇总可以使用行政的post中新增的属性。  



### 监听子组件的事件  
在开发<blog-post>组件时，他们的一些功能可能要求我们和父组件进行沟通，例如我们增加一个按钮单独放大博文中的字体。

在其父组件中，我们可以同构添加一个postFontSize数据属性来支撑这个功能：  
new Vue({
  el:'#blog-posts-events-demo',
  data:{
    posts:[/*...*/],
    postFontSize:1
  }
})


在模板汇总用postFontSize控制字体大小
<div id="blog-posts-events-demo">
  <div :style="{fontSize:postFontSize+'em'}">
    <blog-post
       v-for="post in posts"
       v-bind:key="post.id"
       v-bind:post="post"
     ></blog-post>
  </div>
</div>

组件中我们在模板中增加设置修改字体大小的按钮  
Vue.component('blog-post',{
  prosp:['post'],
  template:`
    <div class="blog-post">
      <h3>{{post.title}}</h3>
      <button>
        Enlarge Text
      </button>
      <div v-html=post.content"></div>
    </div>
  `
})

**Vue提供了一个自定义事件系统来解决这个问题，父组件可以像处理natvie DOM事件一样通过v-on监听子组件实例的任意事件**  

<blog-post
  v-on:enlarge-txt="postFontSize+=01" <!-- 在组件使用时，直接响应v-on自定义事件，并执行对应的函数或直接写js代码 -->
></blog-post>

同时子组件可以通过调用内建的 $emit 方法并传入事件名称来触发一个事件：  
<!-- 子组件代码声明时，可以绑定事件v-on:click，需要使用$emit('传递事件的名称') -->
<button v-on:click="$emit('enlarge-text')">
  Enlarge text
</button>


### 使用事件抛出一个值  
例如我们要指定字体具体的大小，然后子组件的事件携带一个参数
<button v-on:click="$emit('enlarge-text',0.1)">Enlarge Text</button>  

在父组件监听这个事件的时候，我们可以通过$event访问到被抛出的这个值：  
<blog-post
  v-on:enlarge-text="postFontSize+=$event"
></blog-post>
 


### 在组件上使用 v-model  
自定义事件也可以用于创建支持 v-model 的自定义输入组件
<input v-model="searchText">

等价于：
<input 
  v-bind:value="searchText"
  v-on:input="searchText = $event.target.value"
>


**在组件上使用v-model也就是在在输入控件上同时使用了v-bind和v-on:input**  


组件中的Input中绑定模型需要设置v-bind和v-on。  
Vue.component('custom-input',{
  props:['value'],
  template:`
    <input v-bind:value="value"
    v-on:input="$emit('input',$event.target.value)"
  `
})

现在v-mode就可在这个组件上正常工作了：  
<custom-input v-model="searchText"></custom-input>  


### 通过插槽分发内容  (<slot></slot>)
和HTML元素一样，我们经常需要向一个组件传递内容，像这样：  
<alert-box>
  Something bad happened.
</alert-box>

Vue自定义的<slot>元素可以简化这件事：  
  Vue.component('alert-box',{
    template:`
      <div class="demo-alert-box">
        <strong>Error!</strong>
        <slot></slot>
      </div>
    `
  })



### 动态组件  
有时我们不同的组件之前切换是非常必要的，例如在一个多标签界面中：  
我们可以通过Vue<component>元素加一个特殊逇is特性来实现。

<component v-bind:is="currenttabComponent"></component>
<!-- 组件会在currenttableComponent改变时改变 -->



------
**通过切换组件达到显示不同组件的功能**
Vue.component('tab-home',{template:'<div>Home component</div>'})
Vue.component('tab-posts',{tempalte:'<div>Posts component</div>})
Vue.component('tab-archive',{tempalte:'<div>Archive component</div>'})

new Vue({
  el:'#dynamic-component-demo',
  data:{
    currentTab:'Home',
    tabls:['Home','Posts','Archive']
  },
  computed:{
    // 计算属性
    currentTabComponent:function(){
      return 'tab-'+this.currentTab.toLowerCase()
    }
  }
})



**HTML代码**  
<script src="https://unpkg.com/vue"></script>
<div id="dynamic-component-demo" class="demo">
  <button
          v-for="tab in tabls"
          v-bin:key="tab"
          v-bind:class="['tab-button',{active:currentTab===tab}]"
          v-on:click="current=tab"
   >{{tab}}</button>
  
  <component
    v-bind:is="currentTabComponent" <!-- 这里绑定时使用了is -->
    class="tab"
  ></component>
</div>


组件注册  
Vue.component('component-a',{})

new Vue({el:'#app'})

<div id="app">
   <compoent-a></component-a>
   <compoent-b></component-b>
   <compoent-c></component-c>
</div>

1）首先注册组件
2）生成Vue实例
3）确保在html中的Vue注册的根id中包含了组件使用代码。


### 组件的局部注册  

var ComponentA={}
var ComponentB={}

new Vue({
  el:'#app',
  components:{
    'component-a':ComponentA,
    'component-b':ComponentB
  }
})

**注意局部注册的组件在其子组件中是不可用的，流入你希望在ComponentA在ComponentB中使用需要另外一种写法**  
var ComponenetA={...}

var ComponentB={
  componenets:{
    'component-a':ComponentA
  }
}



### 模块系统  
如果把模块放在components目录中，加载文件使用.js或.vue文件

CompoenetB.vue文件中：
import ComponenetA from './ComponenetA'
import ComponenetC from './ComponenetC'

export default {
  components:{
    ComponentA,
    ComponenetC
  }
}

这样导入模块之后就可以在ComponentB的模板中使用了。  


### 基础组件的自动化全局注册  
有时我们在项目中输入框或按钮之类的是相对通用的，欧式我们会把他们称作基础组件，会在各组件中频繁的使用。  

如果使用webpack那么使用require.content只全局注册这些非常通用的基础组件，例如通过src/main.js导入。  


import Vue from 'vue'
import upperFirst from 'lodash/upperFirst'
import camelCase from 'lodash/camelCase'

const requireComponent=require.content(
  './components',
  false,
  /Base[A-Z]\w+\.(vue|js)$/
)

requireComponent.Keys().forEach(fileName=>{
  const componentConfig=requireComponenet(fileName)
  const componentName=upperFirst(
    camelCase(
      fileName
        .split('/')
        .pop()
        .replace(/\.\w+$/,'')
    )
  )
})

// 注册全局组件
Vue.component(
  componentName,
  componentConfig.default||componentConfig
)


**全局注册的行文必须在根Vue实例（new Vue）创建之前发生**  



### Prop类型  
通常我们会通过数组的方式列出prop。
props:['title','likes','isPublished','commmentIds','author']

但是有时希望指定props中的属性类型，我们采用如下的方式：  
props:{
  title:String,
  likes:Number,
  ispublished:Boolean,
  commentIds:Array,
  author:Object,
  callback:Function,
  contactsPromise:Promise
}


### 传递静态或动态Prop  
我们之前已经可以像属性一样传递prop传入一个静态的值：  
<blog-post title="My Name is BlueICE"></blog-post>

我们可以可以通过v-bind动态绑定prop:
<blog-post v-bind:title="post.title"></blog-post>

<blog-post v-bind:title="post.title+'by'+post.author.name"></blog-post>

**任何类型都可以传递给prop**  


### prop传入一个数字  
<blog-post v-bind:likes="42"></blog-post>
<blog-post v-bind:likes="post.likes"></blog-post>


### 传入一个布尔值
<blog-post is-published></blog-post>
<blog-post v-bind:is-published="false"></blog-post>
<blog-post v-bind:is-published="post.isPublished"></blog-post>


### 传入一个数组  
<blog-post v-bind:comment-ids="[1,3,5]"></blog-psot>
  
  
### 单向数据流  
所有的prop都使父子prop之间形成了一个单向下行的绑定，父级prop的更新会向下流动到子组件中，但反之则不行。


### prop验证  
Vue.component('my-component',{
  props:{
    propA:Number,
    propB:[String,Number],
    PropC:{
      type:String,
      required:true
    },
    propD:{
      type:Number,
      default:100
    },
    propF:{
      validator:function(value){
        return ['success','warning','danger'].indexOf(value)!==-1
      }
    }
  }
})



