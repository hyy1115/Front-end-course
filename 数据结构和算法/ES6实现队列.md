# ES6实现队列
```javascript
/*
*队列的特点是在队尾插入元素，队头删除元素，也叫作先进先出
*
*
*/
class Queue {
  constructor() {
    this.store = [] //队列的元素
    this.enqueue = function(element) {
    // 队尾添加元素
      return this.store.push(element)
    }
    this.dequeue = function() {
      // 队首删除元素
      if (this.store.length === 0)
        return false
      return this.store.shift()
    }
    this.front = function() {
      // 读取队首元素
      return this.store[0]
    }
    this.back = function() {
      // 读取队尾元素
      return this.store[this.store.length - 1]
    }
    this.toString = function() {
      // 获取队列的所有元素
      return this.store
    }
    this.empty = function() {
      // 清空队列
      return this.store = []
    }
  }
}

//使用队列
let q = new Queue()
q.enqueue(2)
q.enqueue(4)
q.dequeue()
q.toString() // [4]
```