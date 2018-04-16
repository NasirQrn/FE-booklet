# Vue实例对象的数据选项

　　一般地，当模板内容较简单时，使用data选项配合表达式即可。涉及到复杂逻辑时，则需要用到methods、computed、watch等方法。本文将详细介绍Vue实例对象的数据选项

&nbsp;

### data

　　data是Vue实例的数据对象。Vue将会递归将data的属性转换为getter/setter，从而让data属性能响应数据变化

　　[注意]不应该对`data`属性使用箭头函数

<div>
<pre>&lt;div id="app"&gt;
  {{ message }}
&lt;/div&gt;</pre>
</div>
<div>
<div>
<pre>&lt;script&gt;
var values = {message: 'Hello Vue!'}
var vm = new Vue({
  el: '#app',
  data: values
})
console.log(vm);
&lt;/script&gt;</pre>
</div>
</div>

![vue_base_dataOption1](https://pic.xiaohuochai.site/blog/vue_base_dataOptions1.png)


　　Vue实例创建之后，可以通过`vm.$data`访问原始数据对象

<div>
<pre>console.log(vm.$data);</pre>
</div>

![vue_base_dataOption2](https://pic.xiaohuochai.site/blog/vue_base_dataOptions2.png)


　　Vue实例也代理了data对象上所有的属性

<div>
<pre>&lt;script&gt;
var values = {message: 'Hello Vue!'}
var vm = new Vue({
  el: '#app',
  data: values
})
console.log(vm.$data === values);//true
console.log(vm.message);//'Hello Vue!'
console.log(vm.$data.message);//'Hello Vue!'
&lt;/script&gt;</pre>
</div>

　　被代理的属性是响应的，也就是说值的任何改变都是触发视图的重新渲染。设置属性也会影响到原始数据，反之亦然


![vue_base_dataOption3](https://pic.xiaohuochai.site/blog/vue_base_dataOption3.gif)


　　但是，以`_`或`$`开头的属性不会被Vue实例代理，因为它们可能和Vue内置的属性或方法冲突。可以使用例如`vm.$data._property`的方式访问这些属性

<div>
<pre>&lt;script&gt;
var values = {
  message: 'Hello Vue!',
  _name: '小火柴'
}
var vm = new Vue({
  el: '#app',
  data: values
})
console.log(vm._name);//undefined
console.log(vm.$data._name);//'小火柴'
&lt;/script&gt;</pre>
</div>

&nbsp;

### computed

　　计算属性函数computed将被混入到Vue实例中。所有getter和setter的this上下文自动地绑定为Vue实例

　　[注意]不应该使用箭头函数来定义计算属性函数

　　下面是关于computed的一个例子

<div>
<pre>&lt;div id="example"&gt;
  &lt;p&gt;原始字符串: "{{ message }}"&lt;/p&gt;
  &lt;p&gt;反向字符串: "{{ reversedMessage }}"&lt;/p&gt;
&lt;/div&gt;</pre>
</div>
<div>
<pre>&lt;script&gt;
var vm = new Vue({
  el: '#example',
  data: {
    message: '小火柴'
  },
  computed: {
    reversedMessage: function () {
      return this.message.split('').reverse().join('')
    }
  }
})
&lt;/script&gt;</pre>
</div>

　　结果如下


![vue_base_dataOption4](https://pic.xiaohuochai.site/blog/vue_base_dataOptions4.png)


　　这里声明了一个计算属性&nbsp;`reversedMessage`&nbsp;。提供的函数将用作属性&nbsp;`vm.reversedMessage`&nbsp;的 getter&nbsp;

<div>
<pre>console.log(vm.reversedMessage) // -&gt; '柴火小'
vm.message = 'Goodbye'
console.log(vm.reversedMessage) // -&gt; 'eybdooG'</pre>
</div>

`　　vm.reversedMessage`&nbsp;的值始终取决于&nbsp;`vm.message`&nbsp;的值。可以像绑定普通属性一样在模板中绑定计算属性。当&nbsp;`vm.message`&nbsp;发生改变时，所有依赖于&nbsp;`vm.reversedMessage`&nbsp;的绑定也会更新

　　结果如下图所示，vm.reversedMessage依赖于vm.message的值，vm.reversedMessage本身并不能被赋值


![vue_base_dataOption5](https://pic.xiaohuochai.site/blog/vue_base_dataOption5.gif)


【setter】

　　计算属性默认只有 getter ，不过在需要时也可以提供一个 setter

<div>
<pre>&lt;script&gt;
var vm = new Vue({
  data: { a: 1 },
  computed: {
    // 仅读取，值只须为函数
    aDouble: function () {
      return this.a * 2
    },
    // 读取和设置
    aPlus: {
      get: function () {
        return this.a + 1
      },
      set: function (v) {
        this.a = v - 1
      }
    }
  }
})
console.log(vm.aPlus);//2
vm.aPlus = 3
console.log(vm.a);//2
console.log(vm.aDouble);//4
&lt;/script&gt;</pre>
</div>

&nbsp;

### methods

　　通过调用表达式中的 methods 也可以达到同样的效果

　　[注意]不应该使用箭头函数来定义methods函数

<div>
<pre>&lt;div id="example"&gt;
  &lt;p&gt;原始字符串: "{{ message }}"&lt;/p&gt;
  &lt;p&gt;反向字符串: "{{ reversedMessage() }}"&lt;/p&gt;
&lt;/div&gt;</pre>
</div>
<div>
<pre>&lt;script&gt;
var vm = new Vue({
  el: '#example',
  data: {
    message: '小火柴'
  },
  methods: {
    reversedMessage: function () {
      return this.message.split('').reverse().join('')
    }
  }    
})
&lt;/script&gt;</pre>
</div>

【缓存】

　　对于最终的结果，两种方式确实是相同的

　　然而，不同的是计算属性是基于它们的依赖进行缓存的。计算属性只有在它的相关依赖发生改变时才会重新求值。这就意味着只要&nbsp;`message`&nbsp;还没有发生改变，多次访问&nbsp;`reversedMessage`&nbsp;计算属性会立即返回之前的计算结果，而不必再次执行函数

　　相比而言，只要发生重新渲染，method 调用总会执行该函数。如下所示

<div>
<pre>&lt;div id="example"&gt;
  &lt;p&gt;计算属性: "{{ time1 }}"&lt;/p&gt;
  &lt;p&gt;methods方法: "{{ time2() }}"&lt;/p&gt;
&lt;/div&gt;</pre>
</div>
<div>
<pre>&lt;script&gt;
var vm = new Vue({
  el: '#example',
  computed:{
    time1: function () {
        return (new Date()).toLocaleTimeString()
    }
  },
  methods: {
    time2: function () {
      return (new Date()).toLocaleTimeString()
    }
  }    
})
&lt;/script&gt;</pre>
</div>

![vue_base_dataOption6](https://pic.xiaohuochai.site/blog/vue_base_dataOption6.gif)


　　假设有一个性能开销比较大的的计算属性A，它需要遍历一个极大的数组和做大量的计算。可能有其他的计算属性依赖于&nbsp;A&nbsp;。如果没有缓存，将不可避免的多次执行A的getter！如果不希望有缓存，则用 method 替代

&nbsp;

### watch

　　Vue提供了一种通用的方式来观察和响应Vue实例上的数据变动：watch属性。watch属性是一个对象，键是需要观察的表达式，值是对应回调函数，回调函数得到的参数为新值和旧值。值也可以是方法名，或者包含选项的对象。Vue实例将会在实例化时调用`$watch()`，遍历watch对象的每一个属性

　　[注意]不应该使用箭头函数来定义 watch 函数

<div>
<pre>&lt;div id="app"&gt;
  &lt;button @click="a++"&gt;a加1&lt;/button&gt;
  &lt;p&gt;{{ message }}&lt;/p&gt;
&lt;/div&gt;</pre>
</div>
<div>
<pre>&lt;script&gt;
var vm = new Vue({
  el: '#app',
  data: {
    a: 1,
    message:''
  },
  watch: {
    a: function (val, oldVal) {
      this.message = 'a的旧值为' + oldVal + '，新值为' + val;
    }
  }
})
&lt;/script&gt;</pre>
</div>

　　上面代码中，当a的值发生变化时， 通过watch的监控，使message输出相应的内容

<iframe src="https://demo.xiaohuochai.site/vue/base/b1.html" frameborder="0" width="320" height="100"></iframe>

【$watch】

　　除了使用数据选项中的watch方法以外，还可以使用实例对象的$watch方法，&nbsp;该方法的返回值是一个取消观察函数，用来停止触发回调

<div>
<pre>&lt;div id="app"&gt;
  &lt;button @click="a++"&gt;a加1&lt;/button&gt;
  &lt;p&gt;{{ message }}&lt;/p&gt;
&lt;/div&gt;</pre>
</div>
<div>
<pre>&lt;script&gt;
var vm = new Vue({
  el: '#app',
  data: {
    a: 1,
    message:''
  }
})
var unwatch = vm.$watch('a',function(val, oldVal){
  if(val === 10){
    unwatch();
  }
  this.message = 'a的旧值为' + oldVal + '，新值为' + val; 
})
&lt;/script&gt;</pre>
</div>

　　上面的代码中，当a的值更新到10时，触发unwatch()，来取消观察。点击按钮时，a的值仍然会变化，但是不再触发watch的回调函数


![vue_base_dataOption7](https://pic.xiaohuochai.site/blog/vue_base_dataOption7.gif)
