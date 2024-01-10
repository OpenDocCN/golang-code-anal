# `kubesploit\docs\_build\html\_static\underscore-1.3.1.js`

```
// Underscore.js 1.3.1
// 作者：Jeremy Ashkenas, DocumentCloud Inc.
// Underscore 在 MIT 许可下可以自由分发
// Underscore 的部分灵感来自于 Prototype、Oliver Steele's Functional 和 John Resig's Micro-Templating
// 详细信息和文档请参考：http://documentcloud.github.com/underscore

(function() {

  // 基准设置
  // --------------

  // 建立根对象，在浏览器中为 `window`，在服务器端为 `global`
  var root = this;

  // 保存 `_` 变量的先前值
  var previousUnderscore = root._;

  // 建立用于跳出循环迭代的对象
  var breaker = {};

  // 在被压缩但未被压缩的版本中节省字节
  var ArrayProto = Array.prototype, ObjProto = Object.prototype, FuncProto = Function.prototype;

  // 为了快速访问核心原型，创建快速引用变量
  var slice            = ArrayProto.slice,
      unshift          = ArrayProto.unshift,
      toString         = ObjProto.toString,
      hasOwnProperty   = ObjProto.hasOwnProperty;

  // 所有 **ECMAScript 5** 原生函数实现都在这里声明
  var
    nativeForEach      = ArrayProto.forEach,
    nativeMap          = ArrayProto.map,
    nativeReduce       = ArrayProto.reduce,
    nativeReduceRight  = ArrayProto.reduceRight,
    nativeFilter       = ArrayProto.filter,
    nativeEvery        = ArrayProto.every,
    nativeSome         = ArrayProto.some,
    nativeIndexOf      = ArrayProto.indexOf,
    nativeLastIndexOf  = ArrayProto.lastIndexOf,
    nativeIsArray      = Array.isArray,
    nativeKeys         = Object.keys,
    # 保存 FuncProto.bind 方法的引用
    nativeBind         = FuncProto.bind;

  # 创建对 Underscore 对象的安全引用，用于下面的使用
  var _ = function(obj) { return new wrapper(obj); };

  # 导出 Underscore 对象给 Node.js，同时向后兼容旧的 `require()` API。如果在浏览器中，通过字符串标识符将 `_` 添加为全局对象，以便在 Closure Compiler "advanced" 模式下使用。
  if (typeof exports !== 'undefined') {
    if (typeof module !== 'undefined' && module.exports) {
      exports = module.exports = _;
    }
    exports._ = _;
  } else {
    root['_'] = _;
  }

  # 当前版本
  _.VERSION = '1.3.1';

  # 集合函数
  # --------------------

  # 基石，一个 `each` 实现，又名 `forEach`。
  # 处理具有内置 `forEach`、数组和原始对象的对象。
  # 如果可用，委托给 **ECMAScript 5** 的原生 `forEach`。
  var each = _.each = _.forEach = function(obj, iterator, context) {
    if (obj == null) return;
    if (nativeForEach && obj.forEach === nativeForEach) {
      obj.forEach(iterator, context);
    } else if (obj.length === +obj.length) {
      for (var i = 0, l = obj.length; i < l; i++) {
        if (i in obj && iterator.call(context, obj[i], i, obj) === breaker) return;
      }
    } else {
      for (var key in obj) {
        if (_.has(obj, key)) {
          if (iterator.call(context, obj[key], key, obj) === breaker) return;
        }
      }
    }
  };

  # 返回将迭代器应用于每个元素的结果。
  # 如果可用，委托给 **ECMAScript 5** 的原生 `map`。
  _.map = _.collect = function(obj, iterator, context) {
    var results = [];
    if (obj == null) return results;
    if (nativeMap && obj.map === nativeMap) return obj.map(iterator, context);
    each(obj, function(value, index, list) {
      results[results.length] = iterator.call(context, value, index, list);
    });
    if (obj.length === +obj.length) results.length = obj.length;
  // 返回结果
  return results;
};

// **Reduce** 从数值列表中构建单个结果，也称为 `inject` 或 `foldl`。如果可用，委托给 **ECMAScript 5** 的原生 `reduce`。
_.reduce = _.foldl = _.inject = function(obj, iterator, memo, context) {
  // 检查是否有初始值
  var initial = arguments.length > 2;
  // 如果对象为空，则将其设置为数组
  if (obj == null) obj = [];
  // 如果原生的 reduce 存在并且与传入的 reduce 相同，则使用原生的 reduce
  if (nativeReduce && obj.reduce === nativeReduce) {
    // 如果有上下文，则绑定迭代器到上下文
    if (context) iterator = _.bind(iterator, context);
    // 如果有初始值，则使用初始值进行 reduce，否则直接 reduce
    return initial ? obj.reduce(iterator, memo) : obj.reduce(iterator);
  }
  // 遍历对象，对每个值进行迭代
  each(obj, function(value, index, list) {
    // 如果没有初始值，则将当前值作为初始值
    if (!initial) {
      memo = value;
      initial = true;
    } else {
      // 否则使用迭代器对当前值和初始值进行操作
      memo = iterator.call(context, memo, value, index, list);
    }
  });
  // 如果没有初始值，则抛出错误
  if (!initial) throw new TypeError('Reduce of empty array with no initial value');
  // 返回结果
  return memo;
};

// 右关联版本的 reduce，也称为 `foldr`。如果可用，委托给 **ECMAScript 5** 的原生 `reduceRight`。
_.reduceRight = _.foldr = function(obj, iterator, memo, context) {
  // 检查是否有初始值
  var initial = arguments.length > 2;
  // 如果对象为空，则将其设置为数组
  if (obj == null) obj = [];
  // 如果原生的 reduceRight 存在并且与传入的 reduceRight 相同，则使用原生的 reduceRight
  if (nativeReduceRight && obj.reduceRight === nativeReduceRight) {
    // 如果有上下文，则绑定迭代器到上下文
    if (context) iterator = _.bind(iterator, context);
    // 如果有初始值，则使用初始值进行 reduceRight，否则直接 reduceRight
    return initial ? obj.reduceRight(iterator, memo) : obj.reduceRight(iterator);
  }
  // 将对象转换为数组并进行反转
  var reversed = _.toArray(obj).reverse();
  // 如果有上下文并且没有初始值，则绑定迭代器到上下文
  if (context && !initial) iterator = _.bind(iterator, context);
  // 如果有初始值，则使用初始值进行 reduce，否则直接 reduce
  return initial ? _.reduce(reversed, iterator, memo, context) : _.reduce(reversed, iterator);
};

// 返回通过真值测试的第一个值。别名为 `detect`。
_.find = _.detect = function(obj, iterator, context) {
  var result;
  // 对每个值进行迭代，如果通过真值测试，则返回该值
  any(obj, function(value, index, list) {
    if (iterator.call(context, value, index, list)) {
      result = value;
      return true;
    }
  });
  // 返回结果
  return result;
};

// 返回所有通过真值测试的元素
// 如果可用，委托给 ECMAScript 5 的原生 `filter`
// 别名为 `select`
_.filter = _.select = function(obj, iterator, context) {
  var results = [];
  if (obj == null) return results;
  if (nativeFilter && obj.filter === nativeFilter) return obj.filter(iterator, context);
  each(obj, function(value, index, list) {
    if (iterator.call(context, value, index, list)) results[results.length] = value;
  });
  return results;
};

// 返回所有未通过真值测试的元素
_.reject = function(obj, iterator, context) {
  var results = [];
  if (obj == null) return results;
  each(obj, function(value, index, list) {
    if (!iterator.call(context, value, index, list)) results[results.length] = value;
  });
  return results;
};

// 确定所有元素是否都通过真值测试
// 如果可用，委托给 ECMAScript 5 的原生 `every`
// 别名为 `all`
_.every = _.all = function(obj, iterator, context) {
  var result = true;
  if (obj == null) return result;
  if (nativeEvery && obj.every === nativeEvery) return obj.every(iterator, context);
  each(obj, function(value, index, list) {
    if (!(result = result && iterator.call(context, value, index, list))) return breaker;
  });
  return result;
};

// 确定对象中是否至少有一个元素与真值测试匹配
// 如果可用，委托给 ECMAScript 5 的原生 `some`
// 别名为 `any`
var any = _.some = _.any = function(obj, iterator, context) {
  iterator || (iterator = _.identity);
  var result = false;
  if (obj == null) return result;
  if (nativeSome && obj.some === nativeSome) return obj.some(iterator, context);
  each(obj, function(value, index, list) {
    if (result || (result = iterator.call(context, value, index, list))) return breaker;
  });
  // 返回布尔值，表示结果是否为真
  return !!result;
};

// 使用 === 判断给定值是否包含在数组或对象中
// 别名为 `contains`
_.include = _.contains = function(obj, target) {
  var found = false;
  // 如果对象为空，则返回 false
  if (obj == null) return found;
  // 如果原生的 indexOf 存在且与 obj 的 indexOf 相同，则使用原生的 indexOf 方法判断是否包含目标值
  if (nativeIndexOf && obj.indexOf === nativeIndexOf) return obj.indexOf(target) != -1;
  // 使用 any 方法判断是否包含目标值
  found = any(obj, function(value) {
    return value === target;
  });
  return found;
};

// 对集合中的每个项调用一个方法（带参数）
_.invoke = function(obj, method) {
  // 获取除 obj 和 method 之外的参数
  var args = slice.call(arguments, 2);
  // 对集合中的每个项调用方法，并返回结果
  return _.map(obj, function(value) {
    return (_.isFunction(method) ? method || value : value[method]).apply(value, args);
  });
};

// 便捷版本的常见用例的 `map` 方法：获取属性
_.pluck = function(obj, key) {
  // 获取集合中每个项的指定属性值，并返回数组
  return _.map(obj, function(value){ return value[key]; });
};

// 返回最大元素或（基于元素的计算）
_.max = function(obj, iterator, context) {
  // 如果没有 iterator 并且 obj 是数组，则返回数组中的最大值
  if (!iterator && _.isArray(obj)) return Math.max.apply(Math, obj);
  // 如果没有 iterator 并且 obj 为空，则返回负无穷大
  if (!iterator && _.isEmpty(obj)) return -Infinity;
  // 初始化结果对象
  var result = {computed : -Infinity};
  // 遍历集合，计算最大值
  each(obj, function(value, index, list) {
    var computed = iterator ? iterator.call(context, value, index, list) : value;
    computed >= result.computed && (result = {value : value, computed : computed});
  });
  return result.value;
};

// 返回最小元素（或基于元素的计算）
_.min = function(obj, iterator, context) {
  // 如果没有 iterator 并且 obj 是数组，则返回数组中的最小值
  if (!iterator && _.isArray(obj)) return Math.min.apply(Math, obj);
  // 如果没有 iterator 并且 obj 为空，则返回正无穷大
  if (!iterator && _.isEmpty(obj)) return Infinity;
  // 初始化结果对象
  var result = {computed : Infinity};
  // 遍历集合，计算最小值
  each(obj, function(value, index, list) {
    var computed = iterator ? iterator.call(context, value, index, list) : value;
    computed < result.computed && (result = {value : value, computed : computed});
  });
  // 返回结果值
  return result.value;
};

// 对数组进行洗牌
_.shuffle = function(obj) {
  var shuffled = [], rand;
  // 遍历数组，对每个元素进行处理
  each(obj, function(value, index, list) {
    // 如果是第一个元素，直接赋值
    if (index == 0) {
      shuffled[0] = value;
    } else {
      // 生成随机数，交换元素位置
      rand = Math.floor(Math.random() * (index + 1));
      shuffled[index] = shuffled[rand];
      shuffled[rand] = value;
    }
  });
  // 返回洗牌后的数组
  return shuffled;
};

// 根据迭代器产生的标准对对象的值进行排序
_.sortBy = function(obj, iterator, context) {
  return _.pluck(_.map(obj, function(value, index, list) {
    return {
      value : value,
      criteria : iterator.call(context, value, index, list)
    };
  }).sort(function(left, right) {
    var a = left.criteria, b = right.criteria;
    return a < b ? -1 : a > b ? 1 : 0;
  }), 'value');
};

// 根据标准对对象的值进行分组
_.groupBy = function(obj, val) {
  var result = {};
  var iterator = _.isFunction(val) ? val : function(obj) { return obj[val]; };
  each(obj, function(value, index) {
    var key = iterator(value, index);
    (result[key] || (result[key] = [])).push(value);
  });
  return result;
};

// 使用比较函数确定对象应该插入的索引位置，以保持顺序。使用二分搜索。
_.sortedIndex = function(array, obj, iterator) {
  iterator || (iterator = _.identity);
  var low = 0, high = array.length;
  while (low < high) {
    var mid = (low + high) >> 1;
    iterator(array[mid]) < iterator(obj) ? low = mid + 1 : high = mid;
  }
  return low;
};

// 安全地将任何可迭代对象转换为真实的数组
_.toArray = function(iterable) {
  if (!iterable)                return [];
  if (iterable.toArray)         return iterable.toArray();
  if (_.isArray(iterable))      return slice.call(iterable);
  // 如果 iterable 是一个参数对象，则返回其切片
  if (_.isArguments(iterable))  return slice.call(iterable);
  // 否则返回 iterable 的值组成的数组
  return _.values(iterable);
};

// 返回对象中元素的数量
_.size = function(obj) {
  return _.toArray(obj).length;
};

// 数组函数
// ---------------

// 获取数组的第一个元素。传入 **n** 将返回数组中的前 N 个值。别名为 `head`。**guard** 检查允许它与 `_.map` 一起使用。
_.first = _.head = function(array, n, guard) {
  return (n != null) && !guard ? slice.call(array, 0, n) : array[0];
};

// 返回除数组最后一个条目之外的所有内容。在参数对象上特别有用。传入 **n** 将返回数组中除最后 N 个值之外的所有值。**guard** 检查允许它与 `_.map` 一起使用。
_.initial = function(array, n, guard) {
  return slice.call(array, 0, array.length - ((n == null) || guard ? 1 : n));
};

// 获取数组的最后一个元素。传入 **n** 将返回数组中的最后 N 个值。**guard** 检查允许它与 `_.map` 一起使用。
_.last = function(array, n, guard) {
  if ((n != null) && !guard) {
    return slice.call(array, Math.max(array.length - n, 0));
  } else {
    return array[array.length - 1];
  }
};

// 返回除数组第一个条目之外的所有内容。别名为 `tail`。在参数对象上特别有用。传入 **index** 将返回数组中从该索引开始的所有值。**guard** 检查允许它与 `_.map` 一起使用。
_.rest = _.tail = function(array, index, guard) {
  return slice.call(array, (index == null) || guard ? 1 : index);
};

// 从数组中删除所有假值。
_.compact = function(array) {
  return _.filter(array, function(value){ return !!value; });
};

// 返回数组的完全扁平化版本。
_.flatten = function(array, shallow) {
  // 使用 reduce 方法对数组进行迭代，将每个元素应用到指定的函数中，并将结果汇总为单个返回值
  return _.reduce(array, function(memo, value) {
    // 如果当前值是数组，则将其展开一层并连接到 memo 中
    if (_.isArray(value)) return memo.concat(shallow ? value : _.flatten(value));
    // 如果不是数组，则将其添加到 memo 中
    memo[memo.length] = value;
    return memo;
  }, []);
};

// 返回一个不包含指定值的数组版本
_.without = function(array) {
  // 使用 difference 方法找到 array 和指定值之间的差集
  return _.difference(array, slice.call(arguments, 1));
};

// 生成一个去重后的数组版本。如果数组已经排序，可以选择使用更快的算法。
// 别名为 `unique`。
_.uniq = _.unique = function(array, isSorted, iterator) {
  // 如果提供了迭代器函数，则对数组中的每个元素应用该函数
  var initial = iterator ? _.map(array, iterator) : array;
  var result = [];
  // 使用 reduce 方法对初始数组进行迭代
  _.reduce(initial, function(memo, el, i) {
    // 如果是第一个元素，或者（如果已排序）前一个元素不等于当前元素，或者（如果未排序）memo 中不包含当前元素
    if (0 == i || (isSorted === true ? _.last(memo) != el : !_.include(memo, el))) {
      // 将当前元素添加到 memo 和 result 中
      memo[memo.length] = el;
      result[result.length] = array[i];
    }
    return memo;
  }, []);
  return result;
};

// 生成一个包含所有传入数组中每个不同元素的数组
_.union = function() {
  // 使用 flatten 方法将传入的参数展开一层，并去重
  return _.uniq(_.flatten(arguments, true));
};

// 生成一个包含所有传入数组中共享的每个项的数组。别名为 "intersect" 以支持向后兼容。
_.intersection = _.intersect = function(array) {
  // 获取除第一个数组外的其余参数
  var rest = slice.call(arguments, 1);
  // 使用 filter 方法对去重后的 array 进行筛选，保留在其余数组中都存在的元素
  return _.filter(_.uniq(array), function(item) {
    return _.every(rest, function(other) {
      return _.indexOf(other, item) >= 0;
    });
  });
};

// 获取一个数组和多个其他数组之间的差集。只有存在于第一个数组中的元素将保留。
_.difference = function(array) {
  // 使用 flatten 方法将除第一个数组外的其余参数展开，并将其转换为一个扁平的数组
  var rest = _.flatten(slice.call(arguments, 1));
  // 使用 filter 方法对 array 进行筛选，保留不在 rest 中的元素
  return _.filter(array, function(value){ return !_.include(rest, value); });
};

// 将多个列表合并成一个数组，共享相同索引的元素会被放在一起
_.zip = function() {
    // 将参数转换为数组
    var args = slice.call(arguments);
    // 获取参数中最长的数组的长度
    var length = _.max(_.pluck(args, 'length'));
    // 创建一个长度为 length 的数组
    var results = new Array(length);
    // 遍历参数数组，将每个数组的第 i 个元素组成一个新的数组
    for (var i = 0; i < length; i++) results[i] = _.pluck(args, "" + i);
    // 返回结果数组
    return results;
  };

  // 如果浏览器不支持 indexOf 方法，则使用该函数
  // 返回数组中第一次出现指定元素的位置，如果数组中不包含该元素则返回 -1
  // 如果数组已经排序，并且 isSorted 为 true，则使用二分查找
  _.indexOf = function(array, item, isSorted) {
    if (array == null) return -1;
    var i, l;
    if (isSorted) {
      i = _.sortedIndex(array, item);
      return array[i] === item ? i : -1;
    }
    if (nativeIndexOf && array.indexOf === nativeIndexOf) return array.indexOf(item);
    for (i = 0, l = array.length; i < l; i++) if (i in array && array[i] === item) return i;
    return -1;
  };

  // 如果浏览器支持 lastIndexOf 方法，则使用该方法
  // 返回数组中最后一次出现指定元素的位置，如果数组中不包含该元素则返回 -1
  _.lastIndexOf = function(array, item) {
    if (array == null) return -1;
    if (nativeLastIndexOf && array.lastIndexOf === nativeLastIndexOf) return array.lastIndexOf(item);
    var i = array.length;
    while (i--) if (i in array && array[i] === item) return i;
    return -1;
  };

  // 生成一个包含等差数列的整数数组
  // 参数 start 为起始值，stop 为结束值，step 为步长
  _.range = function(start, stop, step) {
    if (arguments.length <= 1) {
      stop = start || 0;
      start = 0;
    }
    step = arguments[2] || 1;

    // 计算数组的长度
    var len = Math.max(Math.ceil((stop - start) / step), 0);
    var idx = 0;
    var range = new Array(len);

    // 循环生成数组
    while(idx < len) {
      range[idx++] = start;
      start += step;
    }
  // 返回一个范围对象
  return range;
};

// 函数（啊嗯）函数
// ------------------

// 可重用的构造函数，用于设置原型
var ctor = function(){};

// 创建一个绑定到给定对象的函数（分配`this`和参数，可选）。带参数的绑定也被称为`curry`。
// 如果可用，委托给**ECMAScript 5**的本机`Function.bind`。
// 我们首先检查`func.bind`，以便在`func`未定义时快速失败。
_.bind = function bind(func, context) {
  var bound, args;
  if (func.bind === nativeBind && nativeBind) return nativeBind.apply(func, slice.call(arguments, 1));
  if (!_.isFunction(func)) throw new TypeError;
  args = slice.call(arguments, 2);
  return bound = function() {
    if (!(this instanceof bound)) return func.apply(context, args.concat(slice.call(arguments)));
    ctor.prototype = func.prototype;
    var self = new ctor;
    var result = func.apply(self, args.concat(slice.call(arguments)));
    if (Object(result) === result) return result;
    return self;
  };
};

// 将对象的所有方法绑定到该对象。确保在对象上定义的所有回调都属于它。
_.bindAll = function(obj) {
  var funcs = slice.call(arguments, 1);
  if (funcs.length == 0) funcs = _.functions(obj);
  each(funcs, function(f) { obj[f] = _.bind(obj[f], obj); });
  return obj;
};

// 通过存储其结果来记忆一个昂贵的函数。
_.memoize = function(func, hasher) {
  var memo = {};
  hasher || (hasher = _.identity);
  return function() {
    var key = hasher.apply(this, arguments);
    return _.has(memo, key) ? memo[key] : (memo[key] = func.apply(this, arguments));
  };
};

// 延迟给定毫秒数的函数，然后使用提供的参数调用它。
_.delay = function(func, wait) {
  var args = slice.call(arguments, 2);
  // 返回一个延迟执行的函数，等待指定的时间后再执行
  return setTimeout(function(){ return func.apply(func, args); }, wait);
};

// 延迟执行一个函数，等待当前调用栈清空后再执行
_.defer = function(func) {
  return _.delay.apply(_, [func, 1].concat(slice.call(arguments, 1)));
};

// 返回一个函数，当调用时，在给定时间窗口内最多只会触发一次
_.throttle = function(func, wait) {
  var context, args, timeout, throttling, more;
  var whenDone = _.debounce(function(){ more = throttling = false; }, wait);
  return function() {
    context = this; args = arguments;
    var later = function() {
      timeout = null;
      if (more) func.apply(context, args);
      whenDone();
    };
    if (!timeout) timeout = setTimeout(later, wait);
    if (throttling) {
      more = true;
    } else {
      func.apply(context, args);
    }
    whenDone();
    throttling = true;
  };
};

// 返回一个函数，只有在停止调用N毫秒后才会被触发
_.debounce = function(func, wait) {
  var timeout;
  return function() {
    var context = this, args = arguments;
    var later = function() {
      timeout = null;
      func.apply(context, args);
    };
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
  };
};

// 返回一个函数，最多只会被执行一次，无论调用多少次。用于延迟初始化
_.once = function(func) {
  var ran = false, memo;
  return function() {
    if (ran) return memo;
    ran = true;
    return memo = func.apply(this, arguments);
  // 返回一个函数，该函数是作为第二个参数传递的第一个函数，允许您调整参数，在运行代码之前和之后执行代码，并有条件地执行原始函数。
  _.wrap = function(func, wrapper) {
    return function() {
      var args = [func].concat(slice.call(arguments, 0));
      return wrapper.apply(this, args);
    };
  };

  // 返回一个函数，该函数是一系列函数的组合，每个函数消耗后面函数的返回值。
  _.compose = function() {
    var funcs = arguments;
    return function() {
      var args = arguments;
      for (var i = funcs.length - 1; i >= 0; i--) {
        args = [funcs[i].apply(this, args)];
      }
      return args[0];
    };
  };

  // 返回一个函数，该函数在被调用N次后才会被执行。
  _.after = function(times, func) {
    if (times <= 0) return func();
    return function() {
      if (--times < 1) { return func.apply(this, arguments); }
    };
  };

  // 获取对象的属性名称。
  // 委托给**ECMAScript 5**的本机`Object.keys`
  _.keys = nativeKeys || function(obj) {
    if (obj !== Object(obj)) throw new TypeError('Invalid object');
    var keys = [];
    for (var key in obj) if (_.has(obj, key)) keys[keys.length] = key;
    return keys;
  };

  // 获取对象的属性值。
  _.values = function(obj) {
    return _.map(obj, _.identity);
  };

  // 返回对象上可用的函数名称的排序列表。
  // 别名为`methods`
  _.functions = _.methods = function(obj) {
    var names = [];
    for (var key in obj) {
      if (_.isFunction(obj[key])) names.push(key);
    }
    return names.sort();
  };

  // 使用传入对象中的所有属性扩展给定对象。
  _.extend = function(obj) {
  // 对传入的参数进行遍历，并对每个参数执行指定的函数
  each(slice.call(arguments, 1), function(source) {
    // 遍历每个参数对象的属性，并将其赋值给目标对象
    for (var prop in source) {
      obj[prop] = source[prop];
    }
  });
  // 返回合并后的对象
  return obj;
};

// 用默认属性填充给定对象
_.defaults = function(obj) {
  // 对传入的参数进行遍历，并对每个参数执行指定的函数
  each(slice.call(arguments, 1), function(source) {
    // 遍历每个参数对象的属性，如果目标对象的属性为空，则将其赋值为参数对象的属性值
    for (var prop in source) {
      if (obj[prop] == null) obj[prop] = source[prop];
    }
  });
  // 返回填充后的对象
  return obj;
};

// 创建对象的（浅层）克隆
_.clone = function(obj) {
  // 如果对象不是对象类型，则直接返回
  if (!_.isObject(obj)) return obj;
  // 如果对象是数组类型，则使用 slice 方法进行浅拷贝
  return _.isArray(obj) ? obj.slice() : _.extend({}, obj);
};

// 调用拦截器函数，并返回对象
// 这个方法的主要目的是在方法链中“插入”操作，以便在链中间结果上执行操作
_.tap = function(obj, interceptor) {
  // 调用拦截器函数
  interceptor(obj);
  // 返回对象
  return obj;
};

// 内部递归比较函数
function eq(a, b, stack) {
  // 相同的对象是相等的。`0 === -0`，但它们不是相同的。
  // 参见 Harmony `egal` 提案：http://wiki.ecmascript.org/doku.php?id=harmony:egal。
  if (a === b) return a !== 0 || 1 / a == 1 / b;
  // 严格比较是必要的，因为 `null == undefined`。
  if (a == null || b == null) return a === b;
  // 解包装任何包装对象。
  if (a._chain) a = a._wrapped;
  if (b._chain) b = b._wrapped;
  // 如果提供了自定义的 `isEqual` 方法，则调用它
  if (a.isEqual && _.isFunction(a.isEqual)) return a.isEqual(b);
  if (b.isEqual && _.isFunction(b.isEqual)) return b.isEqual(a);
  // 比较 `[[Class]]` 名称
  var className = toString.call(a);
  if (className != toString.call(b)) return false;
    switch (className) {
      // 根据对象类型进行不同的比较
      case '[object String]':
        // 字符串、数字、日期和布尔值通过值进行比较
        // 基本类型和对应的对象包装器是等价的；因此，`"5"` 等价于 `new String("5")`
        return a == String(b);
      case '[object Number]':
        // `NaN` 是等价的，但不是自反的。对其他数值进行 `egal` 比较
        return a != +a ? b != +b : (a == 0 ? 1 / a == 1 / b : a == +b);
      case '[object Date]':
      case '[object Boolean]':
        // 将日期和布尔值转换为数值基本值。日期通过它们的毫秒表示进行比较。注意，具有 `NaN` 毫秒表示的无效日期不是等价的。
        return +a == +b;
      // 正则表达式通过它们的源模式和标志进行比较
      case '[object RegExp]':
        return a.source == b.source &&
               a.global == b.global &&
               a.multiline == b.multiline &&
               a.ignoreCase == b.ignoreCase;
    }
    if (typeof a != 'object' || typeof b != 'object') return false;
    // 假设循环结构相等。检测循环结构的算法改编自 ES 5.1 第 15.12.3 节，抽象操作 `JO`。
    var length = stack.length;
    while (length--) {
      // 线性搜索。性能与唯一嵌套结构的数量成反比。
      if (stack[length] == a) return true;
    }
    // 将第一个对象添加到遍历对象的堆栈中。
    stack.push(a);
    var size = 0, result = true;
    // 递归比较对象和数组。
    if (className == '[object Array]') {
      // 如果传入的参数是数组类型，则进行深度比较
      size = a.length;
      // 比较数组长度，确定是否需要进行深度比较
      result = size == b.length;
      if (result) {
        // 如果数组长度相等，则深度比较数组内容，忽略非数字属性
        while (size--) {
          // 确保稀疏数组的交换相等性
          if (!(result = size in a == size in b && eq(a[size], b[size], stack))) break;
        }
      }
    } else {
      // 具有不同构造函数的对象不相等
      if ('constructor' in a != 'constructor' in b || a.constructor != b.constructor) return false;
      // 深度比较对象
      for (var key in a) {
        if (_.has(a, key)) {
          // 计算预期属性的数量
          size++;
          // 深度比较每个成员
          if (!(result = _.has(b, key) && eq(a[key], b[key], stack))) break;
        }
      }
      // 确保两个对象包含相同数量的属性
      if (result) {
        for (key in b) {
          if (_.has(b, key) && !(size--)) break;
        }
        result = !size;
      }
    }
    // 从遍历过的对象堆栈中移除第一个对象
    stack.pop();
    return result;
  }

  // 执行深度比较，检查两个对象是否相等
  _.isEqual = function(a, b) {
    return eq(a, b, []);
  };

  // 给定的数组、字符串或对象是否为空？
  // “空”对象没有可枚举的自有属性
  _.isEmpty = function(obj) {
    if (_.isArray(obj) || _.isString(obj)) return obj.length === 0;
    for (var key in obj) if (_.has(obj, key)) return false;
    return true;
  };

  // 给定的值是否是 DOM 元素？
  _.isElement = function(obj) {
    return !!(obj && obj.nodeType == 1);
  };

  // 给定的值是否是数组？
  // 委托给 ECMA5 的原生 Array.isArray
  _.isArray = nativeIsArray || function(obj) {
  // 判断给定的变量是否为数组
  return toString.call(obj) == '[object Array]';
};

// 判断给定的变量是否为对象
_.isObject = function(obj) {
  return obj === Object(obj);
};

// 判断给定的变量是否为参数对象
_.isArguments = function(obj) {
  return toString.call(obj) == '[object Arguments]';
};
// 如果不是参数对象，则重新定义 _.isArguments 方法
if (!_.isArguments(arguments)) {
  _.isArguments = function(obj) {
    return !!(obj && _.has(obj, 'callee'));
  };
}

// 判断给定的值是否为函数
_.isFunction = function(obj) {
  return toString.call(obj) == '[object Function]';
};

// 判断给定的值是否为字符串
_.isString = function(obj) {
  return toString.call(obj) == '[object String]';
};

// 判断给定的值是否为数字
_.isNumber = function(obj) {
  return toString.call(obj) == '[object Number]';
};

// 判断给定的值是否为 NaN
_.isNaN = function(obj) {
  // `NaN` 是唯一一个不满足 `===` 自反性的值
  return obj !== obj;
};

// 判断给定的值是否为布尔值
_.isBoolean = function(obj) {
  return obj === true || obj === false || toString.call(obj) == '[object Boolean]';
};

// 判断给定的值是否为日期
_.isDate = function(obj) {
  return toString.call(obj) == '[object Date]';
};

// 判断给定的值是否为正则表达式
_.isRegExp = function(obj) {
  return toString.call(obj) == '[object RegExp]';
};

// 判断给定的值是否为 null
_.isNull = function(obj) {
  return obj === null;
};

// 判断给定的变量是否为未定义
_.isUndefined = function(obj) {
  return obj === void 0;
};

// 是否拥有自有属性
_.has = function(obj, key) {
  return hasOwnProperty.call(obj, key);
};

// 实用函数
// -----------------

// 以 *noConflict* 模式运行 Underscore.js，将 `_` 变量返回给其之前的所有者。返回对 Underscore 对象的引用。
_.noConflict = function() {
  root._ = previousUnderscore;
  // 返回 this 对象
  return this;
};

// 保留默认迭代器的身份函数
_.identity = function(value) {
  return value;
};

// 运行一个函数 n 次
_.times = function (n, iterator, context) {
  for (var i = 0; i < n; i++) iterator.call(context, i);
};

// 为 HTML 插值转义字符串
_.escape = function(string) {
  return (''+string).replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;').replace(/'/g, '&#x27;').replace(/\//g,'&#x2F;');
};

// 将自定义函数添加到 Underscore 对象中，确保它们也正确添加到 OOP 包装器中
_.mixin = function(obj) {
  each(_.functions(obj), function(name){
    addToWrapper(name, _[name] = obj[name]);
  });
};

// 生成唯一的整数 id（在整个客户端会话中唯一）。用于临时 DOM id
var idCounter = 0;
_.uniqueId = function(prefix) {
  var id = idCounter++;
  return prefix ? prefix + id : id;
};

// 默认情况下，Underscore 使用 ERB 样式模板分隔符，更改以下模板设置以使用替代分隔符
_.templateSettings = {
  evaluate    : /<%([\s\S]+?)%>/g,
  interpolate : /<%=([\s\S]+?)%>/g,
  escape      : /<%-([\s\S]+?)%>/g
};

// 在自定义 `templateSettings` 时，如果不想定义插值、评估或转义正则表达式，我们需要一个保证不匹配的正则表达式
var noMatch = /.^/;

// 在插值、评估或转义中，移除先前添加的 HTML 转义
var unescape = function(code) {
  return code.replace(/\\\\/g, '\\').replace(/\\'/g, "'");
};

// JavaScript 微模板，类似于 John Resig 的实现。
// Underscore 模板处理任意分隔符，保留空格，并正确转义插入的代码中的引号
_.template = function(str, data) {
    # 保存当前的模板设置
    var c  = _.templateSettings;
    # 构建模板字符串
    var tmpl = 'var __p=[],print=function(){__p.push.apply(__p,arguments);};' +
      'with(obj||{}){__p.push(\'' +
      # 替换字符串中的特殊字符
      str.replace(/\\/g, '\\\\')
         .replace(/'/g, "\\'")
         .replace(c.escape || noMatch, function(match, code) {
           return "',_.escape(" + unescape(code) + "),'";
         })
         .replace(c.interpolate || noMatch, function(match, code) {
           return "'," + unescape(code) + ",'";
         })
         .replace(c.evaluate || noMatch, function(match, code) {
           return "');" + unescape(code).replace(/[\r\n\t]/g, ' ') + ";__p.push('";
         })
         .replace(/\r/g, '\\r')
         .replace(/\n/g, '\\n')
         .replace(/\t/g, '\\t')
         + "');}return __p.join('');";
    # 创建新的函数对象
    var func = new Function('obj', '_', tmpl);
    # 如果有数据，则调用函数并返回结果
    if (data) return func(data, _);
    # 否则返回一个函数，用于后续调用
    return function(data) {
      return func.call(this, data, _);
    };
  };

  # 添加一个“chain”函数，用于委托给包装器
  _.chain = function(obj) {
    return _(obj).chain();
  };

  # 面向对象编程的包装器
  # 如果 Underscore 被作为函数调用，它将返回一个包装对象，可以以面向对象的方式使用
  # 这个包装器包含所有 Underscore 函数的修改版本，包装对象可以被链式调用
  var wrapper = function(obj) { this._wrapped = obj; };

  # 将 `wrapper.prototype` 暴露为 `_.prototype`
  _.prototype = wrapper.prototype;

  # 辅助函数，用于继续链式调用中间结果
  var result = function(obj, chain) {
    return chain ? _(obj).chain() : obj;
  };

  # 一个方法，用于轻松地将函数添加到面向对象编程的包装器中
  var addToWrapper = function(name, func) {
    wrapper.prototype[name] = function() {
      var args = slice.call(arguments);
      unshift.call(args, this._wrapped);
      return result(func.apply(_, args), this._chain);
  };

  };
  // 将所有 Underscore 函数添加到包装对象中
  _.mixin(_);

  // 将所有修改器数组函数添加到包装对象中
  each(['pop', 'push', 'reverse', 'shift', 'sort', 'splice', 'unshift'], function(name) {
    var method = ArrayProto[name];
    wrapper.prototype[name] = function() {
      var wrapped = this._wrapped;
      method.apply(wrapped, arguments);
      var length = wrapped.length;
      if ((name == 'shift' || name == 'splice') && length === 0) delete wrapped[0];
      return result(wrapped, this._chain);
    };
  });

  // 将所有访问器数组函数添加到包装对象中
  each(['concat', 'join', 'slice'], function(name) {
    var method = ArrayProto[name];
    wrapper.prototype[name] = function() {
      return result(method.apply(this._wrapped, arguments), this._chain);
    };
  });

  // 开始链式调用一个包装的 Underscore 对象
  wrapper.prototype.chain = function() {
    this._chain = true;
    return this;
  };

  // 从包装和链式对象中提取结果
  wrapper.prototype.value = function() {
    return this._wrapped;
  };
# 调用一个匿名函数，并将当前上下文作为参数传入
}).call(this);
```