### 经典问题【 if(a == 1 && a == 2 && a == 3) 】

```js
if(a == 1 && a == 2 && a == 3){
    //... 使之成立    
}
```

#### 思考方向 --- 【**利用隐式转换规则**】

`==` 操作符在左右数据类型不一致时，会先进行隐式转换。

`a == 1 && a == 2 && a == 3` 的值意味着其不可能是基本数据类型。因为如果 a 是 null 或者是 undefined bool类型，都不可能返回true。

因此可以推测 a 是复杂数据类型，JS 中复杂数据类型只有 `object`，回忆一下，Object 转换为原始类型会调用什么方法？

- 如果部署了 `[Symbol.toPrimitive]` 接口，那么调用此接口，若返回的不是基本数据类型，抛出错误。
- 如果没有部署[Symbol.toPrimitive]接口，那么根据要转换的类型，先调用valueOf/toString
  1. 非Date类型对象，`hint` 是 `default` 时，调用顺序为：`valueOf` >>> `toString`，即`valueOf` 返回的不是基本数据类型，才会继续调用 `toString`，如果`toString` 返回的还不是基本数据类型，那么抛出错误。
  2. 如果 `hint` 是 `string`(Date对象的hint默认是string) ，调用顺序为：`toString` >>> `valueOf`，即`toString` 返回的不是基本数据类型，才会继续调用 `valueOf`，如果`valueOf` 返回的还不是基本数据类型，那么抛出错误。
  3. 如果 `hint` 是 `number`，调用顺序为： `valueOf` >>> `toString`



### 7种解决方案

1. 利用 [Symbol.toPrimitive] 接口

   ```js
   let a = {
       [Symbol.toPrimitive]: (function(hint) {
               let i = 1;
               return function() {
                   return i++;
               }
       })()
   }
   console.log(a == 1 && a == 2 && a == 3); //true
   ```

2. 调用 valueOf 接口

   ```js
   let a = {
       valueOf: (function() {
           let i = 1;
           return function() {
               return i++;
           }
       })()
   }
   console.log(a == 1 && a == 2 && a == 3); //true
   ```

3. 利用 正则

   ```js
   let a = {
       reg: /\d/g,
       valueOf () {
           return this.reg.exec(123)[0]
       }
   }
   console.log(a == 1 && a == 2 && a == 3); //true
   ```

4. 利用数据劫持

   - 使用 Object.defineProperty 定义的属性，在获取属性时，会调用 get 方法。利用这个特性，我们在 window 对象上定义 a 属性

     ```js
     let i = 1;
     Object.defineProperty(window, 'a', {
         get: function() {
             return i++;
         }
     });
     console.log(a == 1 && a == 2 && a == 3); //true
     ```

5. 利用ES6 Proxy

   ```js
   let a = new Proxy({}, {
       i: 1,
       get: function () {
           return () => this.i++;
       }
   });
   console.log(a == 1 && a == 2 && a == 3); // true
   ```

6. 重写数组的 join

   ```js
   let a = [1, 2, 3];
   a.join = a.shift;
   console.log(a == 1 && a == 2 && a == 3); //true
   ```

7. 利用 with 关键字

> 注意：0 == '\n' //true