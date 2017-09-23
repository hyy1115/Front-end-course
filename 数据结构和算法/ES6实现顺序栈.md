# ES6实现顺序栈
```javascript
/*
*栈和叠盘子类似，有栈底，栈顶，出栈pop和入栈push只能从栈顶操作。
*
*
*/
class Stack {
  constructor() {
    this.store = [] // 栈元素
    this.length = function() {
      // 栈的长度
      return this.store.length
    }
    this.push = function(element) {
      // 向栈顶添加元素
      return this.store.push(element)
    }
    this.pop = function() {
      // 删除栈顶元素
      if (this.length() === 0)
        return false
      return this.store.pop()
    }
    this.peek = function() {
      // 查看栈顶元素
      return this.store[this.length() - 1]
    }
    this.print = function() {
      // 查看栈的所有元素
      return this.store
    }
    this.clear = function() {
      // 清空栈
      this.store = []
    }
  }
}

// 使用栈类
let s = new Stack()
s.push(2) // [2]
s.push(3) // [2, 3]
s.pop() // [2]
s.peek() // 2
s.print() // [2]
s.length() // 1
s.clear() // []
```