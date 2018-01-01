**本章和react没有多大关系，纯粹是JavaScript语法探讨。**

apply和call想必都不陌生了，那么spread是什么呢？它的中文名称叫做“扩展运算符”，是ES2015定义的语法。
这3者有着密切的关联，如果你掌握了apply和call，那么等你看完spread的用法，会发现新大陆。

**apply**
前端面试中，很多面试官会问你，apply是做什么的？apply和call的区别？

你可以告诉面试官，apply和call只有一个区别，就是传入的参数格式不同，apply传入一个数组参数 [argsArray]，而call传入参数列表 arg1, arg2, ...

如果你还不懂，那么请详细看。

1、apply的语法结构

有2个参数
thisArg：在 fun 函数运行时指定的 this 值，非严格模式下，也就是没有写"use strict"，通常这个值设置为null或者undefined，则指向全局对象（大部分是时候是window）
argsArray：数组对象或者类数组对象，类数组对象只支持ES5，你可以不传任何参数，则为null 或 undefined。
fun.apply(thisArg, [argsArray])

2、apply的用法

有3种用法
a（使用apply链接构造器：和本文无关）
b（在monkey-patching中使用apply）

c（使用apply和内置函数）：内置函数有很多，比如Array实现的内置函数、Math实现的内置函数，...

案例1：假设我们定义2个数组，使用apply调用Array的内置函数push()，可以写成下面这样。

    var arr1 = [1, 3, 4]
    var arr2 = [2, 4, 5]
    //第一个参数表示正在调用调用push()的对象，这行代码的意思是说把arr2的参数传入arr1，你可以看成是arr1.push(arr2)这种行为，但不是等价的。
    Array.prototype.push.apply(arr1, arr2)  //[1, 3, 4, 2, 4, 5]
    //类似下面这种写法，他们的输出是一样的。
    arr2.forEach(function(value) {
    arr1.push(value)
    }) // [1, 3, 4, 2, 4, 5]

案例2：假设我们调用内置函数Math的max()方法，使用apply可以写成下面这样。

    var num = [6, 3, 10, 2, 8];
    Math.max.apply(null, num); // null说明当前函数执行的上下文是在全局作用域，输出10
    
    //如果换成for循环来判断最大值，恐怕你又得写多几行代码了。

**总结：apply作为内置函数调用传参是最经常使用的场景，唯一的难点在于指定第一个参数的运行环境，毕竟一个this可以难倒一大批前端工程师。**

**call**

call和apply类似，你掌握了apply，也就会用call了。

1、call的语法结构：

thisArg：标准叫法，指向当前调用该函数的上下文。不扯这么难听的叫法，和apply的thisArg是完全一样的。
arg1, arg2, arg3, ...：参数列表，对，这就是和apply唯一的区别，可以传入多个参数列表。
fun.call(thisArg, arg1, arg2, arg3, ...)

2、call的用法

也有3种用法

a（使用call方法调用父构造函数）：我就不重新构造一个父类函数了，还是拿刚才操作Array内置对象的方式来举例子。

案例1：假设我们定义2个数组，使用call调用Array的内置函数push()，可以写成下面这样。

    var arr1 = []
    var arr2 = [2, 4, 5]
    var arr3 = ["大神", "美女"]
    //第一个参数表示正在调用调用push()的对象，这行代码的意思是说把arr2、arr3的参数传入arr1，注意call和apply的区别，call是把这个arr2和arr3数组参数传入，不会拆分数组元素。
    Array.prototype.push.call(arr1, arr2, arr3)  
    //你以为可能是[2, 4, 5, "大神", "美女"]
    //其实是这样的 输出[[2, 4, 5], ["大神", "美女"]]

案例2：假设我们调用内置函数Math的max()方法，使用call写成下面这样，但是他是错的，你知道为什么吗？

    var num = [6, 3, 10, 2, 8];
    Math.max.call(null, num); //输出NaN
    //因为用call传max(num)的参数是一个数组，并不是数组元素列表，所以你还得靠for循环来实现。

b（使用call方法调用匿名函数）:

案例：在一个for循环中，有一个匿名函数，设置一个定时器，想要在定时器定时输出循环的key和value。

    var people = [
      {name: '阿里妈妈', age: 18},
      {name: '阿里爸爸', age: 22}
    ];
    //一开始你可能会这样写：
    for (var i = 0; i < people.length; i++) {
        (function (i) { 
            setTimeout(function () { 
              console.log(this.name + ': ' + this.age); 
            }, i * 1000)
        })(i);
    } //输出 
    //"JS Bin Output : undefined"
    //"JS Bin Output : undefined"

然后发现this指向不对，你可能会想到给函数设置一个变量指向当前的this，但是这样也不对，因为输出的this.name和this.age是指向数组animals的对象，那么这时候你需要给匿名函数用到call来绑定this指向的是animals里面循环的对象。

    for (var i = 0; i < people.length; i++) {
        (function (i) { 
            setTimeout(function () { 
              console.log(this.name + ': ' + this.age); 
            }, i * 1000)
        }).call(people[i], i);
    } //输出
    //"JS Bin Output : undefined"
    //"JS Bin Output : undefined"

然而你会发现输出还是undefined，为什么呢？

原因在于匿名函数里面的this是指向了people[i]的对象，但是setTimeout里面的this是指向window的，所以你需要用到bind()绑定this。

    for (var i = 0; i < people.length; i++) {
        (function (i) { 
    //这里也可以用var  _this = this来改变this的上下文（指向）
            setTimeout(function () { 
              console.log(this.name + ': ' + this.age); 
            }.bind(this), i * 1000)
        }).call(people[i], i);
    } //输出
    //"阿里妈妈: 18"
    //"阿里爸爸: 22"

好了，大功告成，你看懂没？

c（使用call方法调用函数并且指定上下文的'this'）：
案例：定义一个people函数，用来输出name和age参数，接着定义一个参数对象，我们要把这个对象传入函数，使函数的this可以指向i对象，这样才能保证读取到i对象里面的key。

    function people() {
      var arr = this.name + this.age;
      console.log(arr);
    }
    var i = {
      name: '阿里爸爸', 
      age: 22
    };
    //如果你直接people(i)，是读取不到的，因为people相当于构造函数，那么通过call的方式就简单了，直接下面这种形式。把people的this用call绑定到i对象上，你用apply也是一样的。
    people.call(i);  ===  people.apply(i)
    //输出 "阿里爸爸22"

**总结：call和apply用法是类似的，唯一的区别在于参数传入的方式，apply是传入一个数组或者类数组参数，call可以传入多个。**

**spread**
主角隆重登场！！！
我们叫他扩展运算符，他的写法是...（三个小点点）
请在 http://babeljs.cn/ 测试下面代码。
spread出现在ES2015标准，如果你是老前端，只是听过这个语法，但是没有用过，那么，非常欢迎你使用它。

还记得我讲apply的时候用到的操作内置函数的案例吗？

我们可以用spread来实现。

编译前：

    var arr1 = [1, 3, 4]
    var arr2 = [2, 4, 5]
    arr1.push(...arr2)
    console.log(arr1) // [1, 3, 4, 2, 4, 5]
    
    var num = [6, 3, 10, 2, 8];
    var max = Math.max(...num)
    console.log(max) // 10
    
编译后：

    var arr1 = [1, 3, 4];
    var arr2 = [2, 4, 5];
    arr1.push.apply(arr1, arr2);
    console.log(arr1); // [1, 3, 4, 2, 4, 5]
    
    var num = [6, 3, 10, 2, 8];
    var max = Math.max.apply(Math, num);
    console.log(max); // 10

还记得我讲call 的时候用到的操作内置函数的案例吗？

编译前：
    var arr1 = []
    var arr2 = [2, 4, 5]
    var arr3 = ["大神", "美女"]
    
    arr1.push(...arr2, ...arr3)
    console.log(arr1) // [2, 4, 5, "大神", "美女"]
编译后：

    var arr1 = [];
    var arr2 = [2, 4, 5];
    var arr3 = ["大神", "美女"];
    
    arr1.push.apply(arr1, arr2.concat(arr3));
    console.log(arr1); // [2, 4, 5, "大神", "美女"]

看到这里，你发现spread的用法了吗？

它可以把我们的数组元素解构出来，还可以解构类数组，这里就不讲解了。

**总结：
1、在一些内置函数调用传参的时候，我们可以不再用apply和call，用spread运算符即可。
2、apply和call用途还是非常广的，当然还有bind，最近几年，你可能都离不开它们。
3、在react开发中，spread运算符在各个组件中运用非常广泛，你可以更加详细的了解它。**

