这是一个常见的问题，

生产环境中应该直接使用lodash库，其中有一组函数处理这种情况；面试中，可以分成2种情况回答

只嵌套了1层
----

方法1：

```js
var flatten = [].concat.apply([], arr)
```

方法2：

```js
var flatten = arr.reduce(function(a,b) {
  return a.concat(b);
}
```

方法3，使用了es6的语法，这种方式不能用于很大的数组：
```js
var flatten = [].concat(...arr)
```

嵌套了n层
----

这种情况自然就是要用递归了

方法1，reduce版本很容易改写成递归调用的形式：

```js
function flatten(arr) {
  return arr.reduce(function (flat, toFlatten) {
    return flat.concat(Array.isArray(toFlatten) ? flatten(toFlatten) : toFlatten);
  }, []);
}
```

方法2，这种写法更加的朴素，但是很上去很好理解，也有不错的性能

```js
const flatten = function(arr, result = []) {
  for (let i = 0, length = arr.length; i < length; i++) {
    const value = arr[i];
    if (Array.isArray(value)) {
      flatten(value, result);
    } else {
      result.push(value);
    }
  }
  return result;
};
```


