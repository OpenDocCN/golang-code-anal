# `kubesploit\docs\_build\html\_static\underscore.js`

```go
// 使用立即执行函数创建一个闭包，避免变量污染全局作用域
(function(){
    // 定义函数q，用于比较两个值是否相等
    function q(a,c,d){
        // 如果a和c相等，判断是否为0或者1/0等特殊情况
        if(a===c) return a!==0 || 1/a==1/c;
        // 如果a或c为null，判断它们是否相等
        if(a==null || c==null) return a===c;
        // 如果a或c有isEqual方法，则调用该方法进行比较
        if(a.isEqual && b.isFunction(a.isEqual)) return a.isEqual(c);
        if(c.isEqual && b.isFunction(c.isEqual)) return c.isEqual(a);
        // 获取a和c的类型
        var e = l.call(a);
        // 如果类型不同，直接返回false
        if(e != l.call(c)) return false;
        // 根据类型进行不同的比较
        switch(e){
            case "[object String]":
                return a == String(c);
            case "[object Number]":
                return a != +a ? c != +c : a == 0 ? 1/a == 1/c : a == +c;
            case "[object Date]":
            case "[object Boolean]":
                return +a == +c;
            case "[object RegExp]":
                return a.source == c.source && a.global == c.global && a.multiline == c.multiline && a.ignoreCase == c.ignoreCase;
        }
        // 如果a和c不是对象类型，直接返回false
        if(typeof a != "object" || typeof c != "object") return false;
        // 避免循环引用，使用数组d记录已经比较过的对象
        for(var f = d.length; f--;)
            if(d[f] == a) return true;
        d.push(a);
        var f = 0, g = true;
        // 如果a是数组类型
        if(e == "[object Array]"){
            // 比较数组长度和每个元素是否相等
            if(f = a.length, g = f == c.length)
                for(; f--;)
                    if(!(g = f in a == f in c && q(a[f], c[f], d))) break;
        } else {
            // 如果a是对象类型
            if("constructor" in a != "constructor" in c || a.constructor != c.constructor) return false;
            // 比较对象的属性和属性值是否相等
            for(var h in a)
                if(b.has(a, h) && (f++, !(g = b.has(c, h) && q(a[h], c[h], d)))) break;
            if(g){
                for(h in c)
                    if(b.has(c, h) && !f--) break;
                g = !f;
            }
        }
        // 比较完毕，从数组d中弹出当前对象
        d.pop();
        return g;
    }
    // 创建全局变量r，指向this
    var r = this,
        // 创建全局变量G，指向r._，如果不存在则指向空对象
        G = r._,
        // 创建空对象n
        n = {},
        // 创建全局变量k，指向Array.prototype
        k = Array.prototype,
        // 创建全局变量o，指向Object.prototype
        o = Object.prototype,
        // 创建全局变量i，指向k.slice
        i = k.slice,
        // 创建全局变量H，指向k.unshift
        H = k.unshift,
        // 创建全局变量l，指向o.toString
        l = o.toString,
        // 创建全局变量I，指向o.hasOwnProperty
        I = o.hasOwnProperty,
        // 创建全局变量w，指向k.forEach
        w = k.forEach,
        // 创建全局变量x，指向k.map
        x = k.map,
        // 创建全局变量y，指向k.reduce
        y = k.reduce,
        // 创建全局变量z，指向k.reduceRight
        z = k.reduceRight,
        // 创建全局变量A，指向k.filter
        A = k.filter,
        // 创建全局变量B，指向k.every
        B = k.every,
        // 创建全局变量C，指向k.some
        C = k.some,
        // 创建全局变量p，指向k.indexOf
        p = k.indexOf,
        // 创建全局变量D，指向k.lastIndexOf
        D = k.lastIndexOf,
        // 创建全局变量o，指向Array.isArray
        o = Array.isArray,
        // 创建全局变量J，指向Object.keys
        J = Object.keys,
        // 创建全局变量s，指向Function.prototype.bind
        s = Function.prototype.bind,
        // 创建全局变量b，指向一个函数，用于创建m的实例
        b = function(a){
            return new m(a);
        };
    // 如果exports存在并且module存在并且module.exports存在，则将b赋值给module.exports
    if(typeof exports !== "undefined"){
        if(typeof module !== "undefined" && module.exports)
            exports = module.exports = b;
        exports._ = b;
    } else {
        r._ = b;
    }
    // 设置b的版本号
    b.VERSION = "1.3.1";
    // 设置b的each方法，指向b的forEach方法
    b.each = 
# 定义一个名为forEach的方法，用于遍历数组或对象，并对每个元素执行指定的函数
b.forEach=function(a,c,d){
    # 如果传入的数组不为空且具有forEach方法，则调用该方法
    if(a!=null)
        if(w && a.forEach===w)
            a.forEach(c,d);
        else
            # 如果传入的是类数组对象，则使用for循环遍历
            if(a.length===+a.length)
                for(var e=0,f=a.length;e<f;e++){
                    if(e in a && c.call(d,a[e],e,a)===n)
                        break;
                }
            else
                # 如果传入的是普通对象，则使用for...in循环遍历
                for(e in a)
                    if(b.has(a,e) && c.call(d,a[e],e,a)===n)
                        break;
};

# 定义一个名为map的方法，用于对数组中的每个元素执行指定的函数，并返回结果数组
b.map=b.collect=function(a,c,b){
    var e=[];
    # 如果传入的数组为空，则直接返回空数组
    if(a==null)
        return e;
    # 如果传入的数组具有map方法，则调用该方法
    if(x && a.map===x)
        return a.map(c,b);
    # 否则使用自定义的遍历方法对数组进行操作
    j(a,function(a,g,h){
        e[e.length]=c.call(b,a,g,h)
    });
    # 返回结果数组
    if(a.length===+a.length)
        e.length=a.length;
    return e;
};

# 定义一个名为reduce的方法，用于将数组中的元素累加或合并成单个值
b.reduce=b.foldl=b.inject=function(a,c,d,e){
    var f=arguments.length>2;
    a==null && (a=[]);
    # 如果传入的数组具有reduce方法，则调用该方法
    if(y && a.reduce===y)
        return e && (c=b.bind(c,e)), f ? a.reduce(c,d) : a.reduce(c);
    # 否则使用自定义的遍历方法对数组进行操作
    j(a,function(a,b,i){
        f ? d=c.call(e,d,a,b,i) : (d=a, f=true)
    });
    if(!f)
        throw new TypeError("Reduce of empty array with no initial value");
    return d;
};

# 定义一个名为reduceRight的方法，用于将数组中的元素从右向左累加或合并成单个值
b.reduceRight=b.foldr=function(a,c,d,e){
    var f=arguments.length>2;
    a==null && (a=[]);
    # 如果传入的数组具有reduceRight方法，则调用该方法
    if(z && a.reduceRight===z)
        return e && (c=b.bind(c,e)), f ? a.reduceRight(c,d) : a.reduceRight(c);
    var g=b.toArray(a).reverse();
    e && !f && (c=b.bind(c,e));
    return f ? b.reduce(g,c,d,e) : b.reduce(g,c);
};

# 定义一个名为find的方法，用于在数组中查找满足条件的第一个元素
b.find=b.detect=function(a,c,b){
    var e;
    # 使用自定义的遍历方法查找满足条件的元素
    E(a,function(a,g,h){
        if(c.call(b,a,g,h))
            return e=a, true;
    });
    return e;
};

# 定义一个名为filter的方法，用于在数组中筛选满足条件的元素
b.filter=b.select=function(a,c,b){
    var e=[];
    # 如果传入的数组为空，则直接返回空数组
    if(a==null)
        return e;
    # 如果传入的数组具有filter方法，则调用该方法
    if(A && a.filter===A)
        return a.filter(c,b);
    # 否则使用自定义的遍历方法对数组进行操作
    j(a,function(a,g,h){
        c.call(b,a,g,h) && (e[e.length]=a);
    });
    return e;
};

# 定义一个名为reject的方法，用于在数组中排除满足条件的元素
b.reject=function(a,c,b){
    var e=[];
    # 如果传入的数组为空，则直接返回空数组
    if(a==null)
        return e;
    # 使用自定义的遍历方法对数组进行操作
    j(a,function(a,g,h){
        c.call(b,a,g,h) || (e[e.length]=a);
    });
    return e;
};

# 定义一个名为every的方法，用于判断数组中的所有元素是否都满足指定条件
b.every=b.all=function(a,c,b){
    var e=true;
    # 如果传入的数组为空，则直接返回true
    if(a==null)
        return e;
    # 如果传入的数组具有every方法，则调用该方法
    if(B && a.every===B)
        return a.every(c,b);
    # 使用自定义的遍历方法对数组进行操作
    j(a,function(a,g,h){
        if(!(e=
# 定义了一个名为 E 的函数，参数为 a, c, d
e&&c.call(b,a,g,h)))return n});return e};var E=b.some=b.any=function(a,c,d){
    # 如果 c 不存在，则将 c 设置为 b.identity 函数
    c||(c=b.identity);
    # 初始化变量 e 为 false
    var e=false;
    # 如果 a 为 null，则返回 e
    if(a==null)return e;
    # 如果存在原生的 some 函数，则调用原生的 some 函数
    if(C&&a.some===C)return a.some(c,d);
    # 遍历数组 a，对每个元素执行回调函数
    j(a,function(a,b,h){
        # 如果 e 为真或者调用回调函数返回真，则将 e 设置为回调函数的返回值
        if(e=c.call(d,a,b,h))return n
    });
    # 返回 e 的布尔值
    return!!e
};
# 将 some 函数的别名设置为 E
b.some=b.any=function(a,c,d){
    # 调用 E 函数
    c||(c=b.identity);
    # 如果 a 为 null，则返回 false
    var e=false;
    if(a==null)return e;
    # 如果存在原生的 indexOf 函数，则调用原生的 indexOf 函数
    return p&&a.indexOf===p?a.indexOf(c)!=-1:b=E(a,function(a){return a===c})
};
# 定义了一个名为 include 的函数，参数为 a, c
b.include=b.contains=function(a,c){
    # 初始化变量 b 为 false
    var b=false;
    # 如果 a 为 null，则返回 b
    if(a==null)return b;
    # 如果存在原生的 indexOf 函数，则调用原生的 indexOf 函数
    return p&&a.indexOf===p?a.indexOf(c)!=-1:b=E(a,function(a){return a===c})
};
# 定义了一个名为 invoke 的函数，参数为 a, c
b.invoke=function(a,c){
    # 将参数转换为数组
    var d=i.call(arguments,2);
    # 对数组 a 中的每个元素执行回调函数
    return b.map(a,function(a){
        # 如果 c 是函数，则调用 c 函数，否则调用 a[c] 函数
        return(b.isFunction(c)?c||a:a[c]).apply(a,d)
    })
};
# 定义了一个名为 pluck 的函数，参数为 a, c
b.pluck=function(a,c){
    # 对数组 a 中的每个元素执行回调函数，返回结果组成的数组
    return b.map(a,function(a){
        # 返回每个元素的属性 c 的值
        return a[c]
    })
};
# 定义了一个名为 max 的函数，参数为 a, c, d
b.max=function(a,c,d){
    # 如果 c 不存在且 a 是数组，则调用原生的 Math.max 函数
    if(!c&&b.isArray(a))return Math.max.apply(Math,a);
    # 如果 c 不存在且 a 是空数组，则返回负无穷大
    if(!c&&b.isEmpty(a))return-Infinity;
    # 初始化变量 e，设置其 computed 属性为负无穷大
    var e={computed:-Infinity};
    # 遍历数组 a，对每个元素执行回调函数
    j(a,function(a,b,h){
        # 如果 c 存在，则调用 c 函数，否则将 b 设置为 a
        b=c?c.call(d,a,b,h):a;
        # 如果 b 大于等于 e.computed，则将 e 设置为当前元素和 b 的组合
        b>=e.computed&&(e={value:a,computed:b})
    });
    # 返回 e 的 value 属性
    return e.value
};
# 定义了一个名为 min 的函数，参数为 a, c, d
b.min=function(a,c,d){
    # 如果 c 不存在且 a 是数组，则调用原生的 Math.min 函数
    if(!c&&b.isArray(a))return Math.min.apply(Math,a);
    # 如果 c 不存在且 a 是空数组，则返回正无穷大
    if(!c&&b.isEmpty(a))return Infinity;
    # 初始化变量 e，设置其 computed 属性为正无穷大
    var e={computed:Infinity};
    # 遍历数组 a，对每个元素执行回调函数
    j(a,function(a,b,h){
        # 如果 c 存在，则调用 c 函数，否则将 b 设置为 a
        b=c?c.call(d,a,b,h):a;
        # 如果 b 小于 e.computed，则将 e 设置为当前元素和 b 的组合
        b<e.computed&&(e={value:a,computed:b})
    });
    # 返回 e 的 value 属性
    return e.value
};
# 定义了一个名为 shuffle 的函数，参数为 a
b.shuffle=function(a){
    # 初始化变量 b 为空数组
    var b=[],d;
    # 遍历数组 a，对每个元素执行回调函数
    j(a,function(a,f){
        # 如果 f 为 0，则将 b[0] 设置为当前元素
        f==0?b[0]=a:
        # 否则，生成一个随机数，将当前元素和随机位置的元素交换
        (d=Math.floor(Math.random()*(f+1)),b[f]=b[d],b[d]=a)
    });
    # 返回打乱后的数组
    return b
};
# 定义了一个名为 sortBy 的函数，参数为 a, c, d
b.sortBy=function(a,c,d){
    # 对数组 a 中的每个元素执行回调函数，返回结果组成的数组
    return b.pluck(b.map(a,function(a,b,g){
        # 返回一个对象，包含原始值和经过回调函数处理后的值
        return{value:a,criteria:c.call(d,a,b,g)}
    # 对结果数组进行排序
    }).sort(function(a,b){
        # 比较两个元素的 criteria 属性，进行排序
        var c=a.criteria,d=b.criteria;
        return c<d?-1:c>d?1:0
    # 返回排序后的结果数组中的 value 属性
    }),"value")
};
# 定义了一个名为 groupBy 的函数，参数为 a, c
b.groupBy=function(a,c){
    # 初始化变量 d 为空对象
    var d={},e=b.isFunction(c)?c:function(a){return a[c]};
    # 遍历数组 a，对每个元素执行回调函数
    j(a,function(a,b){
        # 根据回调函数的返回值，将元素分组
        var c=e(a,b);
        (d[c]||(d[c]=[])).push(a)
    });
    # 返回分组后的对象
    return d
};
# 定义了一个名为 sortedIndex 的函数，参数为 a,
// 定义一个函数，用于在数组中查找指定元素的位置
c,d){d||(d=b.identity);for(var e=0,f=a.length;e<f;){var g=e+f>>1;d(a[g])<d(c)?e=g+1:f=g}return e};
// 定义一个函数，将类数组对象转换为数组
b.toArray=function(a){return!a?[]:a.toArray?a.toArray():b.isArray(a)?i.call(a):b.isArguments(a)?i.call(a):b.values(a)};
// 定义一个函数，用于获取数组或类数组对象的长度
b.size=function(a){return b.toArray(a).length};
// 定义一个函数，用于获取数组的第一个元素
b.first=b.head=function(a,b,d){return b!=null&&!d?i.call(a,0,b):a[0]};
// 定义一个函数，用于获取数组的除最后一个元素外的所有元素
b.initial=function(a,b,d){return i.call(a,0,a.length-(b==null||d?1:b))};
// 定义一个函数，用于获取数组的最后一个元素
b.last=function(a,b,d){return b!=null&&!d?i.call(a,Math.max(a.length-b,0)):a[a.length-1]};
// 定义一个函数，用于获取数组的除第一个元素外的所有元素
b.rest=b.tail=function(a,b,d){return i.call(a,b==null||d?1:b)};
// 定义一个函数，用于过滤数组中的假值元素
b.compact=function(a){return b.filter(a,function(a){return!!a})};
// 定义一个函数，用于将嵌套数组扁平化
b.flatten=function(a,c){return b.reduce(a,function(a,e){if(b.isArray(e))return a.concat(c?e:b.flatten(e));a[a.length]=e;return a},[])};
// 定义一个函数，用于从数组中排除指定元素
b.without=function(a){return b.difference(a,i.call(arguments,1))};
// 定义一个函数，用于获取数组中的唯一元素
b.uniq=b.unique=function(a,c,d){var d=d?b.map(a,d):a,e=[];b.reduce(d,function(d,g,h){if(0==h||(c===true?b.last(d)!=g:!b.include(d,g)))d[d.length]=g,e[e.length]=a[h];return d},[]);return e};
// 定义一个函数，用于获取多个数组的并集
b.union=function(){return b.uniq(b.flatten(arguments,true))};
// 定义一个函数，用于获取多个数组的交集
b.intersection=b.intersect=function(a){var c=i.call(arguments,1);return b.filter(b.uniq(a),function(a){return b.every(c,function(c){return b.indexOf(c,a)>=0})})};
// 定义一个函数，用于获取两个数组的差集
b.difference=function(a){var c=b.flatten(i.call(arguments,1));return b.filter(a,function(a){return!b.include(c,a)})};
// 定义一个函数，用于将多个数组合并成一个二维数组
b.zip=function(){for(var a=i.call(arguments),c=b.max(b.pluck(a,"length")),d=Array(c),e=0;e<c;e++)d[e]=b.pluck(a,""+e);return d};
// 定义一个函数，用于在数组中查找指定元素的位置
b.indexOf=function(a,c,
// 定义一个函数，用于查找指定元素在数组中的索引位置
b.indexOf = function(a, c, d) {
    // 如果数组为空，则返回-1
    if (a == null) return -1;
    // 如果传入了排序函数，则使用排序函数进行查找
    var e;
    if (d) return d = b.sortedIndex(a, c), a[d] === c ? d : -1;
    // 如果支持原生的 indexOf 方法，则使用原生的 indexOf 方法进行查找
    if (p && a.indexOf === p) return a.indexOf(c);
    // 遍历数组，查找指定元素的索引位置
    for (d = 0, e = a.length; d < e; d++)
        if (d in a && a[d] === c) return d;
    return -1;
};
// 定义一个函数，用于从数组末尾开始查找指定元素在数组中的索引位置
b.lastIndexOf = function(a, b) {
    // 如果数组为空，则返回-1
    if (a == null) return -1;
    // 如果支持原生的 lastIndexOf 方法，则使用原生的 lastIndexOf 方法进行查找
    if (D && a.lastIndexOf === D) return a.lastIndexOf(b);
    // 从数组末尾开始遍历，查找指定元素的索引位置
    for (var d = a.length; d--;)
        if (d in a && a[d] === b) return d;
    return -1;
};
// 定义一个函数，用于生成指定范围内的数字数组
b.range = function(a, b, d) {
    // 如果只传入了一个参数，则将其作为结束值，起始值默认为0
    arguments.length <= 1 && (b = a || 0, a = 0);
    // 计算数组的步长
    for (var d = arguments[2] || 1, e = Math.max(Math.ceil((b - a) / d), 0), f = 0, g = Array(e); f < e;) g[f++] = a, a += d;
    return g;
};
// 定义一个空函数 F
var F = function() {};
// 定义一个函数，用于绑定函数的上下文和参数
b.bind = function(a, c) {
    var d, e;
    // 如果支持原生的 bind 方法，则使用原生的 bind 方法进行绑定
    if (a.bind === s && s) return s.apply(a, i.call(arguments, 1));
    // 如果传入的第一个参数不是函数，则抛出类型错误
    if (!b.isFunction(a)) throw new TypeError;
    // 获取除了第一个和第二个参数外的其他参数
    e = i.call(arguments, 2);
    // 返回一个新的函数
    return d = function() {
        // 如果不是通过 new 关键字调用，则将函数绑定到指定的上下文和参数
        if (!(this instanceof d)) return a.apply(c, e.concat(i.call(arguments)));
        F.prototype = a.prototype;
        var b = new F,
            g = a.apply(b, e.concat(i.call(arguments)));
        return Object(g) === g ? g : b;
    }
};
// 定义一个函数，用于绑定对象的所有方法到对象本身
b.bindAll = function(a) {
    var c = i.call(arguments, 1);
    // 如果没有传入方法名，则获取对象的所有方法名
    c.length == 0 && (c = b.functions(a));
    // 遍历所有方法，将方法绑定到对象本身
    j(c, function(c) {
        a[c] = b.bind(a[c], a)
    });
    return a;
};
// 定义一个函数，用于缓存函数的计算结果
b.memoize = function(a, c) {
    var d = {};
    // 如果没有传入计算缓存键的函数，则使用默认的缓存键计算函数
    c || (c = b.identity);
    return function() {
        var e = c.apply(this, arguments);
        return b.has(d, e) ? d[e] : d[e] = a.apply(this, arguments)
    }
};
// 定义一个函数，用于延迟执行指定函数
b.delay = function(a, b) {
    var d = i.call(arguments, 2);
    return setTimeout(function() {
        return a.apply(a, d)
    }, b)
};
// 定义一个函数，用于延迟执行指定函数（延迟时间为1毫秒）
b.defer = function(a) {
    return b.delay.apply(b, [a, 1].concat(i.call(arguments, 1)))
};
// 定义一个函数，用于限制函数的执行频率
b.throttle = function(a, c) {
    var d, e, f, g, h, i = b.debounce(function() {
        h = g = false
    }, c);
    return function() {
        d = this;
        e = arguments;
        var b;
        // 如果没有延迟执行的函数，则设置延迟执行的函数
        f || (f = setTimeout(function() {
            f = null;
            h && a.apply(d, e);
            i()
        }, c));
        g ? h = true;
# 定义函数 debounce，接受一个函数和一个延迟时间参数，返回一个新的函数
b.debounce=function(a,b){
    var d;  # 声明变量 d
    return function(){  # 返回一个匿名函数
        var e=this,  # 声明变量 e 并赋值为当前函数的 this 指向
        f=arguments;  # 声明变量 f 并赋值为函数的参数列表
        clearTimeout(d);  # 清除延迟执行的函数
        d=setTimeout(function(){  # 设置延迟执行的函数
            d=null;  # 变量 d 赋值为 null
            a.apply(e,f)  # 调用传入的函数 a，并传入参数 e 和 f
        },b)  # 设置延迟时间为参数 b
    }
};
# 定义函数 once，接受一个函数作为参数，返回一个新的函数
b.once=function(a){
    var b=false,  # 声明变量 b 并赋值为 false
    d;  # 声明变量 d
    return function(){  # 返回一个匿名函数
        if(b)  # 如果变量 b 为 true
            return d;  # 返回变量 d
        b=true;  # 变量 b 赋值为 true
        return d=a.apply(this,arguments)  # 调用传入的函数 a，并传入参数 this 和 arguments，并将结果赋值给变量 d
    }
};
# 定义函数 wrap，接受两个函数作为参数，返回一个新的函数
b.wrap=function(a,b){
    return function(){  # 返回一个匿名函数
        var d=[a].concat(i.call(arguments,0));  # 声明变量 d，并将函数 a 和参数列表合并成一个新的数组
        return b.apply(this,d)  # 调用函数 b，并传入参数 this 和 d
    }
};
# 定义函数 compose，接受多个函数作为参数，返回一个新的函数
b.compose=function(){
    var a=arguments;  # 声明变量 a，并赋值为参数列表
    return function(){  # 返回一个匿名函数
        for(var b=arguments,d=a.length-1;d>=0;d--)  # 循环遍历参数列表
            b=[a[d].apply(this,b)];  # 调用函数并传入参数，并将结果赋值给变量 b
        return b[0]  # 返回变量 b 的第一个元素
    }
};
# 定义函数 after，接受两个参数，返回一个新的函数
b.after=function(a,b){
    return a<=0?b():function(){  # 如果参数 a 小于等于 0，则直接调用函数 b，否则返回一个匿名函数
        if(--a<1)  # 如果参数 a 自减后小于 1
            return b.apply(this,arguments)  # 调用函数 b，并传入参数 this 和 arguments
    }
};
# 定义函数 keys，如果存在原生方法 J，则使用 J，否则自定义函数
b.keys=J||function(a){
    if(a!==Object(a))  # 如果参数 a 不是对象
        throw new TypeError("Invalid object");  # 抛出类型错误异常
    var c=[],  # 声明变量 c，并赋值为空数组
    d;  # 声明变量 d
    for(d in a)  # 遍历对象 a 的属性
        b.has(a,d)&&(c[c.length]=d);  # 如果对象 a 存在属性 d，则将属性名 d 添加到数组 c 中
    return c  # 返回数组 c
};
# 定义函数 values，接受一个对象作为参数，返回对象的值组成的数组
b.values=function(a){
    return b.map(a,b.identity)  # 调用 map 方法，传入参数 a 和函数 b.identity
};
# 定义函数 functions 和 methods，接受一个对象作为参数，返回对象的方法名组成的数组
b.functions=b.methods=function(a){
    var c=[],  # 声明变量 c，并赋值为空数组
    d;  # 声明变量 d
    for(d in a)  # 遍历对象 a 的属性
        b.isFunction(a[d])&&c.push(d);  # 如果对象 a 的属性是函数，则将属性名添加到数组 c 中
    return c.sort()  # 返回排序后的数组 c
};
# 定义函数 extend，接受一个对象和多个参数，将参数对象的属性复制到第一个对象中
b.extend=function(a){
    j(i.call(arguments,1),function(b){  # 调用 j 方法，传入参数列表和一个函数
        for(var d in b)  # 遍历参数对象的属性
            a[d]=b[d]  # 将属性复制到第一个对象中
    });
    return a  # 返回第一个对象
};
# 定义函数 defaults，接受一个对象和多个参数，将参数对象的属性复制到第一个对象中，但不覆盖已有属性
b.defaults=function(a){
    j(i.call(arguments,1),function(b){  # 调用 j 方法，传入参数列表和一个函数
        for(var d in b)  # 遍历参数对象的属性
            a[d]==null&&(a[d]=b[d]);  # 如果第一个对象的属性值为 null，则将参数对象的属性复制到第一个对象中
    });
    return a  # 返回第一个对象
};
# 定义函数 clone，接受一个对象作为参数，返回对象的浅拷贝
b.clone=function(a){
    return!b.isObject(a)?a:  # 如果参数不是对象，则直接返回参数
    b.isArray(a)?a.slice():  # 如果参数是数组，则返回数组的浅拷贝
    b.extend({},a)  # 否则返回对象的浅拷贝
};
# 定义函数 tap，接受一个对象和一个函数作为参数，对对象执行函数并返回对象本身
b.tap=function(a,b){
    b(a);  # 执行函数 b，传入参数 a
    return a  # 返回参数 a
};
# 定义函数 isEqual，接受两个参数，判断两个参数是否相等
b.isEqual=function(a,b){
    return q(a,b,[])  # 调用 q 方法，传入参数 a、b 和一个空数组
};
# 定义函数 isEmpty，接受一个对象作为参数，判断对象是否为空
b.isEmpty=function(a){
    if(b.isArray(a)||b.isString(a))  # 如果参数是数组或字符串
        return a.length===0;  # 返回数组或字符串的长度是否为 0
    for(var c in a)  # 遍历对象的属性
        if(b.has(a,c))  # 如果对象存在属性 c
            return false;  # 返回 false
    return true  # 否则返回 true
};
# 定义函数 isElement，接受一个参数，判断参数是否为 DOM 元素
b.isElement=function(a){
    return!!(a&&a.nodeType==1)  # 如果参数存在并且是 DOM 元素，则返回 true，否则返回 false
};
# 定义函数 isArray，如果存在原生方法 o，则使用 o，否则自定义函数
b.isArray=o||function(a){
    return l.call(a)=="[object Array]"  # 返回参数的类型是否为数组
};
# 定义函数 isObject，判断参数是否为对象
b.isObject=function(a){
    return a===Object(a)  # 返回参数是否为对象
};
# 检查参数是否为 Arguments 对象
b.isArguments=function(a){return l.call(a)=="[object Arguments]"};
# 如果参数不是 Arguments 对象，则重新定义 isArguments 函数
if(!b.isArguments(arguments))b.isArguments=function(a){return!(!a||!b.has(a,"callee")};
# 检查参数是否为函数
b.isFunction=function(a){return l.call(a)=="[object Function]"};
# 检查参数是否为字符串
b.isString=function(a){return l.call(a)=="[object String]"};
# 检查参数是否为数字
b.isNumber=function(a){return l.call(a)=="[object Number]"};
# 检查参数是否为 NaN
b.isNaN=function(a){return a!==a};
# 检查参数是否为布尔值
b.isBoolean=function(a){return a===true||a===false||l.call(a)=="[object Boolean]"};
# 检查参数是否为日期对象
b.isDate=function(a){return l.call(a)=="[object Date]"};
# 检查参数是否为正则表达式对象
b.isRegExp=function(a){return l.call(a)=="[object RegExp]"};
# 检查参数是否为 null
b.isNull=function(a){return a===null};
# 检查参数是否为 undefined
b.isUndefined=function(a){return a===void 0};
# 检查对象是否包含指定属性
b.has=function(a,b){return I.call(a,b)};
# 释放对 Underscore 的控制，返回全局对象
b.noConflict=function(){r._=G;return this};
# 返回参数本身
b.identity=function(a){return a};
# 多次执行指定函数
b.times=function(a,b,d){for(var e=0;e<a;e++)b.call(d,e)};
# 转义字符串中的特殊字符
b.escape=function(a){return(""+a).replace(/&/g,"&amp;").replace(/</g,"&lt;").replace(/>/g,"&gt;").replace(/"/g,"&quot;").replace(/'/g,"&#x27;").replace(/\//g,"&#x2F;")};
# 将指定对象的函数混入 Underscore
b.mixin=function(a){j(b.functions(a),function(c){K(c,b[c]=a[c])})};
# 生成唯一的 ID
var L=0;b.uniqueId=function(a){var b=L++;return a?a+b:b};
# 定义模板的特殊字符
b.templateSettings={evaluate:/<%([\s\S]+?)%>/g,interpolate:/<%=([\s\S]+?)%>/g,escape:/<%-([\s\S]+?)%>/g};
var t=/.^/,u=function(a){return a.replace(/\\\\/g,"\\").replace(/\\'/g,"'")};
# 编译模板字符串生成模板函数
b.template=function(a,c){var d=b.templateSettings,d="var __p=[],print=function(){__p.push.apply(__p,arguments);};with(obj||{}){__p.push('"+a.replace(/\\/g,"\\\\").replace(/'/g,"\\'").replace(d.escape||t,function(a,b){return"',_.escape("+
# 定义一个匿名函数，用于处理模板字符串的插值和逻辑表达式
u(b)+"),'"}).replace(d.interpolate||t,function(a,b){return"',"+u(b)+",'"}).replace(d.evaluate||t,function(a,b){return"');"+u(b).replace(/[\r\n\t]/g," ")+";__p.push('"}).replace(/\r/g,"\\r").replace(/\n/g,"\\n").replace(/\t/g,"\\t")+"');}return __p.join('');",e=new Function("obj","_",d);return c?e(c,b):function(a){return e.call(this,a,b)}};b.chain=function(a){return b(a).chain()};
# 定义一个函数，用于创建一个包装对象
var m=function(a){this._wrapped=a};
# 将包装对象的原型指向 b 的原型
b.prototype=m.prototype;
# 定义一个函数，用于判断是否需要链式调用
var v=function(a,c){return c?b(a).chain():a},K=function(a,c){m.prototype[a]=
# 将指定方法添加到包装对象的原型上，并根据是否需要链式调用返回结果
function(){var a=i.call(arguments);H.call(a,this._wrapped);return v(c.apply(b,a),this._chain)}};
# 将指定方法添加到包装对象的原型上，并根据是否需要链式调用返回结果
b.mixin(b);
# 遍历数组，将指定方法添加到包装对象的原型上，并根据是否需要链式调用返回结果
j("pop,push,reverse,shift,sort,splice,unshift".split(","),function(a){var b=k[a];m.prototype[a]=function(){var d=this._wrapped;b.apply(d,arguments);var e=d.length;(a=="shift"||a=="splice")&&e===0&&delete d[0];return v(d,this._chain)}});
# 遍历数组，将指定方法添加到包装对象的原型上，并根据是否需要链式调用返回结果
j(["concat","join","slice"],function(a){var b=k[a];m.prototype[a]=function(){return v(b.apply(this._wrapped,arguments),this._chain)}});
# 将链式调用标记设置为 true，并返回包装对象
m.prototype.chain=function(){this._chain=true;return this};
# 返回包装对象的值
m.prototype.value=function(){return this._wrapped}
# 调用函数
}).call(this);
```