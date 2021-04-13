`nextTick` 作用是：将回调延迟到下次 DOM 更新循环之后执行。有两种用法 `Vue.nextTick` 和 `vm.$nextTick` ，使用如下：
```javascript
let vm = new Vue({
  el: '#app',
  data: {
    name: 'ayguo'
  },
  mounted(){
    // 方式一
    this.$nextTick( () => {
      this.name = '大兵'
      console.log(1)

      vm.$nextTick( () => {
        vm.name = '大熊3'
        console.log(2)
      })

    })
  },
  watch: {
    name(newVal, oldVal){
      console.log(newVal, oldVal)
    }
  }
})

// 方式二
vm.$nextTick( () => {
  vm.name = '大熊'
  console.log(3)
})

// 方式三
Vue.nextTick( () => {
  vm.name = '大个'
  console.log(4)
})

```

打印结果
> 1  
> 3  
> 4  
> 大个 ayguo  
> 2  
> 大熊3 大个


我们看一下为什么 `nextTick` 能在 `DOM` 更新之后执行？
先看源码：
```javascript
var isUsingMicroTask = false;

var callbacks = [];
var pending = false;

function flushCallbacks () {
  pending = false;
  var copies = callbacks.slice(0);
  callbacks.length = 0;
  for (var i = 0; i < copies.length; i++) {
    copies[i]();  // 执行回调函数
  }
}

var timerFunc;

if (typeof Promise !== 'undefined' && isNative(Promise)) {
  var p = Promise.resolve();
  timerFunc = function () {
    p.then(flushCallbacks); // 实现异步
    if (isIOS) { setTimeout(noop); }
  };
  isUsingMicroTask = true;
} else if (!isIE && typeof MutationObserver !== 'undefined' && (
  isNative(MutationObserver) ||
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  var counter = 1;
  var observer = new MutationObserver(flushCallbacks);
  var textNode = document.createTextNode(String(counter));
  observer.observe(textNode, {
    characterData: true
  });
  timerFunc = function () {
    counter = (counter + 1) % 2;
    textNode.data = String(counter);
  };
  isUsingMicroTask = true;
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  timerFunc = function () {
    setImmediate(flushCallbacks); // 实现异步
  };
} else {
  timerFunc = function () {
    setTimeout(flushCallbacks, 0);  // 实现异步
  };
}

function nextTick (cb, ctx) {
  var _resolve;
  callbacks.push(function () {
    if (cb) {
      try {
        cb.call(ctx);
      } catch (e) {
        handleError(e, ctx, 'nextTick');
      }
    } else if (_resolve) {
      _resolve(ctx);
    }
  });
  if (!pending) {
    pending = true;
    timerFunc();
  }
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(function (resolve) {
      _resolve = resolve;
    })
  }
}

// this.$nextTick 或 vm.$nextTick
Vue.prototype.$nextTick = function (fn) {
  return nextTick(fn, this)
};

// Vue.nextTick
Vue.nextTick = nextTick;

```


当调用 `nextTick` 时，会将回调函数放在 `callbacks` 中， 继续往下执行， 这时 `pending = false`, 然后执行 `timerFunc` 方法。
我们看看 `timerFunc` 方法，`timerFunc` 中实现使用到 `Promise.resolve().then()` 或 `setTimeout` ，然后调用 `flushCallbacks`, 然后执行 `callbacks` 里面的方法，以实现异步调用。








