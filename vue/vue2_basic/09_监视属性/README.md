### 监视属性 vs 计算属性 vs 方法
- 监视属性和computed都是执行回调函数:监视属性是要监视的属性发生改变时执行回调;计算属性是其依赖数据发生改变时执行回调。那么把要监视的属性换成依赖数据用计算属性实现回调，不就可以实现相同的功能吗?
    - 确实是这样,不过一来语义不够清晰 增加代码理解难度 不合实际需求,二来实现复杂度增加。
- 共同点: 三者都能作为回调，理论上可以互相替代。当然方法作为回调可能要用插值表达式等方法引入，此时回调条件和计算属性一样都是依赖的数据发生变化
- 差异性: 语义性不同。
### 监视属性有哪些应用场景?

1. 表单验证：当用户输入表单中的某个值时，需要验证其是否符合要求，例如必填项、输入格式、长度限制等。可以通过监视属性来监听表单值的变化，当值发生变化时触发验证函数，从而实时验证表单输入是否合法。
2. 性能优化：当某些计算量较大的操作只有在特定条件下才需要执行时，可以通过监视属性来监听条件变化，从而避免不必要的计算，从而提高性能。
> 假设我们有一个数据列表，其中包含大量数据，我们需要根据用户的输入来筛选出符合条件的数据。由于数据量很大，如果每次用户输入都重新计算筛选后的数据，会非常耗费计算资源，降低应用程序的性能。因此，我们可以通过监视属性来实现懒执行，只有在用户输入满足某个条件时才计算筛选后的数据。
> 

### 监视属性和计算属性原理比较(重要)
- 都是用Object.defineProperty()实现的
- 计算属性是用defineProperty()给对象创造了一个崭新的属性,默认是get方法,当访问这个属性时,会执行get方法,基于已有的属性计算出一个返回值
- 监视属性是用defineProperty()给要监视的已有属性添加了一个set方法，如果监视的是一个对象并且开启了深度监视，那么就递归地给要监视对象的所有属性添加set方法。
- vue给计算属性的实现额外做了一些数据依赖的相关工作，对所有依赖的属性也用defineProperty添加或者修改了get方法，以让他们在发生变化时，去通知调用计算属性的get方法，从而触发计算属性的回调函数。如果是双向绑定的依赖数据,依赖属性中的set方法似乎也有可能去调用计算属性的get方法。----具体了解还要看源码实现。(chatgpt: 如果依赖属性是双向绑定的，即使用了 v-model 或者 this.$watch 等方式实现双向数据绑定，那么当依赖属性的值发生变化时，会触发这个属性的 setter 方法，这个 setter 方法可以进一步触发计算属性的 getter 方法)
- 而vue实现监视属性特性相对就容易很多,只需要在监视属性的set方法中执行回调函数即可。

- 思考:Object.defineProperty中定义了get和set方法，如果get中修改了自身这个属性，会调用set方法吗? 参见下面的两段代码。
```javascript
let obj = {};
let value = 'initial value';

Object.defineProperty(obj, 'prop', {
  get: function() {
    value = 'new value';
    console.log("this",this)
    this.prop="test value"
    return value;
  },
  set: function(newValue) {
    console.log('set method called with value:', newValue);
    value = newValue;
  }
});  
```
```javascript
let obj = {};
let value = 'initial value';

Object.defineProperty(obj, 'prop', {
  get: function() {
    value = 'new value';
    console.log("this",this)
    this.prop="test value"
    return value;
  },
  // set: function(newValue) {
  //   console.log('set method called with value:', newValue);
  //   value = newValue;
  // }
});  
```