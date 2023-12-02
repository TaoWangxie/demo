<template>
  <div class="container"></div>
</template>
<script setup lang="ts">
import { error } from "console";
import { resolve } from "dns";
import { da } from "element-plus/es/locale";
import { forIn } from "lodash";
import { ref, onMounted, nextTick } from "vue";

/**
 * ===============1===============
 * promise
 */
class myPromise {
  constructor(executor) {
    this.status = "pending";
    this.result = "";
    this.resolvefns = [];
    this.rejectfns = [];
    let resolve = (value) => {
      this.result = value;
      this.status = "fulfilled";
      this.resolvefns.forEach((fn) => fn());
    };
    let reject = (value) => {
      this.result = value;
      this.status = "rejected";
      this.rejectfns.forEach((fn) => fn());
    };
    try {
      executor(resolve, reject);
    } catch (e) {
      reject(e);
    }
  }
  then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === "function" ? onFulfilled : (v) => v;
    onRejected =
      typeof onRejected === "function"
        ? onRejected
        : (v) => {
            throw v;
          };
    return new myPromise((resolve, reject) => {
      let resolvePromise = (cb) => {
        setTimeout(() => {
          try {
            let x = cb(this.result);
            if (x === myPromise) {
              throw new Error("myPromise can not be chained");
            } else if (x instanceof myPromise) {
              x.then(resolve, reject);
            } else {
              resolve(x);
            }
          } catch (error) {
            reject(error);
            throw error;
          }
        });
      };
      if (this.status === "pending") {
        this.resolvefns.push(() => resolvePromise(onFulfilled));
        this.rejectfns.push(() => resolvePromise(onRejected));
      } else if (this.status === "fulfilled") {
        resolvePromise(onFulfilled);
      } else if (this.status === "rejected") {
        resolvePromise(onRejected);
      }
    });
  }
}


// 测试
let p1 = new myPromise((resolve, reject) => {
  setTimeout(() => {
    console.log("11");
    resolve(1);
  }, 1000);
})
  .then((res) => {
    return res * 2;
  })
  .then((res) => {
    console.log(res * 2);
  });

/**
 * ===============2===============
 * promise.all.race.allsettled
 */

const promiseAll = (arr) => {
  return new Promise((resolve, reject) => {
    let result = new Array(arr.length).fill(null);
    let count = 0;
    arr.map((fn, index) => {
      Promise.resolve(fn)
        .then((res) => {
          result[index] = res;
          count++;
          if (count === arr.length) {
            resolve(result);
          }
        })
        .catch((err) => {
          reject(err);
        });
    });
  });
};
const promiseAllsettled = (arr) => {
  return new Promise((resolve, reject) => {
    let result = new Array(arr.length).fill(null);
    let count = 0;
    arr.map((fn, index) => {
      Promise.resolve(fn)
        .then((res) => {
          result[index] = {
            status: "fulfilled",
            value: res,
          };
          count++;
          if (count === arr.length) {
            resolve(result);
          }
        })
        .catch((err) => {
          result[index] = {
            status: "rejected",
            value: err,
          };
          count++;
          if (count === arr.length) {
            resolve(result);
          }
        });
    });
  });
};
const promiseRace = (arr) => {
  return new Promise((resolve, reject) => {
    arr.map((fn, index) => {
      Promise.resolve(fn)
        .then((res) => resolve(res))
        .catch((err) => reject(err));
    });
  });
};

/**
 * ===============3===============
 * async/await
 */

const fn = (num) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      num++;
      resolve(num);
    }, 1000);
  });
};
function* gen() {
  let data = yield fn(1);
  let data2 = yield fn(data);
  return data2;
}
const myAsync = (genF) => {
  return new Promise((resolve, reject) => {
    let gen = genF();
    const step = (fn) => {
      let next;
      try {
        next = fn();
      } catch (error) {
        return reject(error);
      }
      if (next.done) {
        return resolve(next.value);
      }
      Promise.resolve(next.value).then(
        (res) => {
          step(() => gen.next(res));
        },
        (e) => {
          step(() => gen.throw(e));
        }
      );
    };
    step(() => gen.next());
  });
};



/**
 * ===============4===============
 * EventBus
 */

class EventBus {
  constructor() {
    this.eventBus = {};
  }

  on(eventName, cb) {
    let task = this.eventsBus[eventName];
    if (task) {
      task.push(cb);
    } else {
      task = [cb];
    }
  }

  emit(eventName, data) {
    let task = this.eventsBus[eventName];
    if (task) {
      task.map((cb) => {
        cb && cb(data);
      });
    }
  }

  off(eventName, cb) {
    let task = this.eventsBus[eventName];
    if (task && task.indexOf(cb) !== -1) {
      task.splice(task.indexOf(cb), 1);
    }
  }
  once(eventName, data) {
    let task = this.eventsBus[eventName];
    if (task) {
      task.map((cb) => {
        cb && cb(data);
      });
      delete task;
    }
  }
}

/**
 * ===============5===============
 * call-apply-bind
 */

Function.prototype.mycall = function (context) {
  context =
    context === null || context === undefined ? window : Object(context);
  let args = [...arguments.slice(1)];
  let fn = Symbol("fn");
  context[fn] = this;
  let res = context[fn](...args);
  delete context[fn];
  return res;
};

Function.prototype.myapply = function (context, ...args) {
  context =
    context === null || context === undefined ? window : Object(context);
  let fn = Symbol("fn");
  context[fn] = this;
  let res = context[fn](...args);
  delete context[fn];
  return res;
};

Function.prototype.mybind = function (context) {
  let fn = this;
  let args = [...arguments].slice(1);
  let newFunction = () => {
    let newargs = args.concat(...arguments);
    if (this instanceof newFunction) {
      fn.apply(this, newargs);
    } else {
      fn.apply(context, newargs);
    }
    newFunction.prototype = Object.create(fn.prototype);
    return newFunction;
  };
};



/**
 * ===============6===============
 * 深拷贝
 */

function deepClone(target, weakMap = new WeakMap()) {
  if (weakMap.has(target)) {
    return weakMap.get(target);
  }
  if (typeof target === "object") {
    let cloneTarget;
    if (target instanceof Date) {
      cloneTarget = new Date(target);
    } else if (target instanceof Array) {
      cloneTarget = [];
    } else if (target instanceof RegExp) {
      cloneTarget = new RegExp(target.source, target.flags);
    } else {
      cloneTarget = {};
      weakMap.set(target, cloneTarget);
      for (let key in target) {
        if (target.hasOwnProperty(key)) {
          cloneTarget = deepClone(target[key], weakMap);
        }
      }
    }
    return cloneTarget;
  } else {
    return target;
  }
}

/**
 * ===============7===============
 * 防抖节流
 */
const debounce = (fn, daley) => {
  let timer = null;
  return function () {
    timer && clearTimeout(timer);
    timer = setTimeout(() => {
      fn.call(this, arguments);
    }, daley);
  };
};

const throttle = (fn, daley) => {
  let flag = true;
  return function () {
    if (!flag) return;
    flag = false;
    setTimeout(() => {
      fn.call(this, arguments);
      flag = true;
    }, daley);
  };
};


/**
 * ===============8===============
 * instanceof
 */

//  function instanceof(obj,fun){
//   let proto = obj.__proto__
//   let prototype = fun.prototype
//   while(true){
//     if(proto == null) return false
//     if(proto === prototype) return true
//     proto = proto.__proto__
//   }
//  }


/**
 * ===============9===============
 * new
 */
function _new(fn,...args){
  let obj = {}
  obj.__proto__ = fn.prototype
  let res = fn.apply(obj,args)
  if(res instanceof Object){
    return res
  }
  return obj
}



/**
 * ===============10===============
 * ajax
 */
const ajax = {
    get(url,fn){
        const xhr = XMLHttpRequest()
        xhr.open('GET',url,true)
        xhr.onreadystatechange = function(){
            if(xhr.readyState === 4){
                fn(xhr.responeText)
            }
        }
        xhr.send()
    },
    post(url,data,fn){
        const xhr = XMLHttpRequest()
        xhr.open('POST',url,true)
        xhr.setResquestHeader('Content-type','application/x-www-form-urlencoded')
        xhr.onreadystatechange = function(){
            if(chr.readyState === 4){
                fn(xhr.responeText)
            }
        }
        xhr.send(data)
    }
}

/**
 * promise并发限制
 */

 function multiRequest(urls, maxnum){
  let result = new Array(urls.length).fill(null)
  let count = 0
  return new Promise((resolve,reject)=>{
    while(count < maxnum){
      next()
    }
    function next(){
      count++
      if(count >= urls.length){
        !result.includes(false) && resolve(result)
        return
      }
      let url = urls[count]
      fetch(url).then((res)=>{
        result[count] = res
        if(count < urls.length){
          next()
        }
      }).catch((e)=>{
        result[count] = e
        if(count < urls.length){
          next()
        }
      })
    }
  })
 }

 /**
  * 数组交集
  */

 function intersection(arr1,arr2){
  let map = new Map()
  arr1.map((n)=>{
    map.set(n,true)
  })

  let res:any = []
  arr2.map((n)=>{
    if(map.get(n)){
      res.push(n)
      map.delete(n)
    }
  })
  return res
 }

 /**
  * 斐波那契数列
  */

  function fff(n){
    n = n && parseInt(n)
    if(n === 1 || n === 2){
      return 1
    }
    let n1 = 1
    let n2 = 1
    for (let i = 2; i < n; i++) {
      [n1,n2] = [n2,n1+n2]
    }
    return n2
  }

  /**
   * 数组中只出现一次的数字
   */

   function singlenumber(nums){
    let result = 0
    nums.map((item)=>{
      result ^= item
    })
    return result
   }

   /**
    * 两数之和
    */

    function twosum(nums,target){
      let map = new Map()
      for (let i = 0; i < nums.length; i++) {
        let neednum = target - nums[i]
        if(map.has(neednum)){
          return [i,map.get(neednum)]
        }
        map.set(nums[i],i)
      }
    }
    /**
     * 删除有序数组重复项 
     */
    function removeDuplicates(nums){
      let slow = 0
      let fast = 1
      if(nums.length === 0) return 0
      while(fast < nums.length){
        if(nums[slow] !== nums[fast]){
          nums[++slow] = nums(fast)
        }
        fast++
      }
      return slow+1
    }
</script>
<style lang="scss">
.container {
  width: 100%;
  height: 100vh;
  
}
</style>
