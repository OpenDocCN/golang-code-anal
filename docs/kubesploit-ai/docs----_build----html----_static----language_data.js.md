# `kubesploit\docs\_build\html\_static\language_data.js`

```go
/*
 * language_data.js
 * ~~~~~~~~~~~~~~~~
 *
 * This script contains the language-specific data used by searchtools.js,
 * namely the list of stopwords, stemmer, scorer and splitter.
 *
 * :copyright: Copyright 2007-2020 by the Sphinx team, see AUTHORS.
 * :license: BSD, see LICENSE for details.
 *
 */

// 停用词列表，用于搜索工具
var stopwords = ["a","and","are","as","at","be","but","by","for","if","in","into","is","it","near","no","not","of","on","or","such","that","the","their","then","there","these","they","this","to","was","will","with"];


/* Non-minified version JS is _stemmer.js if file is provided */ 
/**
 * Porter Stemmer
 */
// 词干提取器
var Stemmer = function() {

  // 步骤2的替换规则
  var step2list = {
    ational: 'ate',
    tional: 'tion',
    enci: 'ence',
    anci: 'ance',
    izer: 'ize',
    bli: 'ble',
    alli: 'al',
    entli: 'ent',
    eli: 'e',
    ousli: 'ous',
    ization: 'ize',
    ation: 'ate',
    ator: 'ate',
    alism: 'al',
    iveness: 'ive',
    fulness: 'ful',
    ousness: 'ous',
    aliti: 'al',
    iviti: 'ive',
    biliti: 'ble',
    logi: 'log'
  };

  // 步骤3的替换规则
  var step3list = {
    icate: 'ic',
    ative: '',
    alize: 'al',
    iciti: 'ic',
    ical: 'ic',
    ful: '',
    ness: ''
  };

  var c = "[^aeiou]";          // 辅音
  var v = "[aeiouy]";          // 元音
  var C = c + "[^aeiouy]*";    // 辅音序列
  var V = v + "[aeiou]*";      // 元音序列

  var mgr0 = "^(" + C + ")?" + V + C;                      // [C]VC... 是 m>0
  var meq1 = "^(" + C + ")?" + V + C + "(" + V + ")?$";    // [C]VC[V] 是 m=1
  var mgr1 = "^(" + C + ")?" + V + C + V + C;              // [C]VCVC... 是 m>1
  var s_v   = "^(" + C + ")?" + v;                         // 词干中的元音

  // 提取词干
  this.stemWord = function (w) {
    var stem;
    var suffix;
    var firstch;
    var origword = w;

    if (w.length < 3)
      return w;

    var re;
    var re2;
    var re3;
    var re4;

    firstch = w.substr(0,1);
    if (firstch == "y")
      w = firstch.toUpperCase() + w.substr(1);

    // 步骤1a
    // 定义正则表达式，匹配以ss或者es结尾的单词
    re = /^(.+?)(ss|i)es$/;
    // 定义正则表达式，匹配以s结尾但不是ss结尾的单词
    re2 = /^(.+?)([^s])s$/;

    // 如果单词符合re的正则表达式
    if (re.test(w))
      // 替换符合re的正则表达式的部分
      w = w.replace(re,"$1$2");
    // 如果单词符合re2的正则表达式
    else if (re2.test(w))
      // 替换符合re2的正则表达式的部分
      w = w.replace(re2,"$1$2");

    // Step 1b
    // 定义正则表达式，匹配以eed结尾的单词
    re = /^(.+?)eed$/;
    // 定义正则表达式，匹配以ed或ing结尾的单词
    re2 = /^(.+?)(ed|ing)$/;
    // 如果单词符合re的正则表达式
    if (re.test(w)) {
      // 获取匹配到的部分
      var fp = re.exec(w);
      // 重新定义re为mgr0的正则表达式
      re = new RegExp(mgr0);
      // 如果单词符合re的正则表达式
      if (re.test(fp[1])) {
        // 定义正则表达式，匹配任意字符结尾
        re = /.$/;
        // 替换符合re的正则表达式的部分
        w = w.replace(re,"");
      }
    }
    // 如果单词符合re2的正则表达式
    else if (re2.test(w)) {
      // 获取匹配到的部分
      var fp = re2.exec(w);
      // 将stem赋值为匹配到的部分
      stem = fp[1];
      // 重新定义re2为s_v的正则表达式
      re2 = new RegExp(s_v);
      // 如果单词符合re2的正则表达式
      if (re2.test(stem)) {
        // 将w赋值为stem
        w = stem;
        // 定义正则表达式，匹配以at、bl、iz结尾的单词
        re2 = /(at|bl|iz)$/;
        // 定义正则表达式，匹配不是aeiouylsz中的任意字符重复一次的单词
        re3 = new RegExp("([^aeiouylsz])\\1$");
        // 定义正则表达式，匹配以C+v+非aeiouwxy结尾的单词
        re4 = new RegExp("^" + C + v + "[^aeiouwxy]$");
        // 如果单词符合re2的正则表达式
        if (re2.test(w))
          // 在单词末尾添加e
          w = w + "e";
        // 如果单词符合re3的正则表达式
        else if (re3.test(w)) {
          // 定义正则表达式，匹配任意字符结尾
          re = /.$/;
          // 替换符合re的正则表达式的部分
          w = w.replace(re,"");
        }
        // 如果单词符合re4的正则表达式
        else if (re4.test(w))
          // 在单词末尾添加e
          w = w + "e";
      }
    }

    // Step 1c
    // 定义正则表达式，匹配以y结尾的单词
    re = /^(.+?)y$/;
    // 如果单词符合re的正则表达式
    if (re.test(w)) {
      // 获取匹配到的部分
      var fp = re.exec(w);
      // 将stem赋值为匹配到的部分
      stem = fp[1];
      // 重新定义re为s_v的正则表达式
      re = new RegExp(s_v);
      // 如果单词符合re的正则表达式
      if (re.test(stem))
        // 在单词末尾添加i
        w = stem + "i";
    }

    // Step 2
    // 定义正则表达式，匹配以ational、tional、enci等结尾的单词
    re = /^(.+?)(ational|tional|enci|anci|izer|bli|alli|entli|eli|ousli|ization|ation|ator|alism|iveness|fulness|ousness|aliti|iviti|biliti|logi)$/;
    // 如果单词符合re的正则表达式
    if (re.test(w)) {
      // 获取匹配到的部分
      var fp = re.exec(w);
      // 将stem赋值为匹配到的部分
      stem = fp[1];
      // 将suffix赋值为匹配到的部分
      suffix = fp[2];
      // 重新定义re为mgr0的正则表达式
      re = new RegExp(mgr0);
      // 如果单词符合re的正则表达式
      if (re.test(stem))
        // 将w赋值为stem加上step2list[suffix]的值
        w = stem + step2list[suffix];
    }

    // Step 3
    // 定义正则表达式，匹配以icate、ative、alize等结尾的单词
    re = /^(.+?)(icate|ative|alize|iciti|ical|ful|ness)$/;
    // 如果单词符合re的正则表达式
    if (re.test(w)) {
      // 获取匹配到的部分
      var fp = re.exec(w);
      // 将stem赋值为匹配到的部分
      stem = fp[1];
      // 将suffix赋值为匹配到的部分
      suffix = fp[2];
      // 重新定义re为mgr0的正则表达式
      re = new RegExp(mgr0);
      // 如果单词符合re的正则表达式
      if (re.test(stem))
        // 将w赋值为stem加上step3list[suffix]的值
        w = stem + step3list[suffix];
    }

    // Step 4
    // 定义正则表达式，匹配以al、ance、ence等结尾的单词
    re = /^(.+?)(al|ance|ence|er|ic|able|ible|ant|ement|ment|ent|ou|ism|ate|iti|ous|ive|ize)$/;
    // 定义正则表达式，匹配以s或t结尾的单词
    re2 = /^(.+?)(s|t)(ion)$/;
    // 如果单词符合re的正则表达式
    if (re.test(w)) {
      // 获取匹配到的部分
      var fp = re.exec(w);
      // 将stem赋值为匹配到的部分
      stem = fp[1];
      // 重新定义re为mgr1的正则表达式
      re = new RegExp(mgr1);
      // 如果单词符合re的正则表达式
      if (re.test(stem))
        // 将w赋值为stem
        w = stem;
    }
    // 如果单词符合 re2 的正则表达式，则执行以下操作
    else if (re2.test(w)) {
      // 用 re2 匹配 w，返回匹配结果
      var fp = re2.exec(w);
      // 将匹配结果的第一部分和第二部分拼接成 stem
      stem = fp[1] + fp[2];
      // 重新设置 re2 为 mgr1 的正则表达式
      re2 = new RegExp(mgr1);
      // 如果 re2 仍然匹配 stem，则将 w 更新为 stem
      if (re2.test(stem))
        w = stem;
    }

    // Step 5
    // 设置 re 为匹配以 e 结尾的正则表达式
    re = /^(.+?)e$/;
    // 如果 re 匹配 w，则执行以下操作
    if (re.test(w)) {
      // 用 re 匹配 w，返回匹配结果
      var fp = re.exec(w);
      // 将匹配结果的第一部分赋值给 stem
      stem = fp[1];
      // 重新设置 re 为 mgr1 的正则表达式
      re = new RegExp(mgr1);
      // 重新设置 re2 为 meq1 的正则表达式
      re2 = new RegExp(meq1);
      // 设置 re3 为匹配以 C 开头，接着是一个元音字母，然后不是元音字母 w x y 结尾的正则表达式
      re3 = new RegExp("^" + C + v + "[^aeiouwxy]$");
      // 如果 stem 匹配 re 或者（stem 匹配 re2 且不匹配 re3），则将 w 更新为 stem
      if (re.test(stem) || (re2.test(stem) && !(re3.test(stem))))
        w = stem;
    }
    // 设置 re 为匹配以 ll 结尾的正则表达式
    re = /ll$/;
    // 重新设置 re2 为 mgr1 的正则表达式
    re2 = new RegExp(mgr1);
    // 如果 re 匹配 w 且 re2 也匹配 w，则执行以下操作
    if (re.test(w) && re2.test(w)) {
      // 设置 re 为匹配任意字符的正则表达式
      re = /.$/;
      // 将 w 中匹配 re 的部分替换为空字符串
      w = w.replace(re,"");
    }

    // 将单词开头的 Y 转换为小写的 y
    if (firstch == "y")
      w = firstch.toLowerCase() + w.substr(1);
    // 返回处理后的单词
    return w;
  }
# 定义一个立即执行函数，用于初始化一个包含特殊字符的对象
var splitChars = (function() {
    # 初始化一个空对象
    var result = {};
    # 定义一个包含单个字符的数组
    var singles = [96, 180, 187, 191, 215, 247, 749, 885, 903, 907, 909, 930, 1014, 1648,
         1748, 1809, 2416, 2473, 2481, 2526, 2601, 2609, 2612, 2615, 2653, 2702,
         2706, 2729, 2737, 2740, 2857, 2865, 2868, 2910, 2928, 2948, 2961, 2971,
         2973, 3085, 3089, 3113, 3124, 3213, 3217, 3241, 3252, 3295, 3341, 3345,
         3369, 3506, 3516, 3633, 3715, 3721, 3736, 3744, 3748, 3750, 3756, 3761,
         3781, 3912, 4239, 4347, 4681, 4695, 4697, 4745, 4785, 4799, 4801, 4823,
         4881, 5760, 5901, 5997, 6313, 7405, 8024, 8026, 8028, 8030, 8117, 8125,
         8133, 8181, 8468, 8485, 8487, 8489, 8494, 8527, 11311, 11359, 11687, 11695,
         11703, 11711, 11719, 11727, 11735, 12448, 12539, 43010, 43014, 43019, 43587,
         43696, 43713, 64286, 64297, 64311, 64317, 64319, 64322, 64325, 65141];
    var i, j, start, end;
    # 将单个字符数组中的每个字符作为对象的属性，并赋值为 true
    for (i = 0; i < singles.length; i++) {
        result[singles[i]] = true;
    }
    # 返回初始化后的对象
    return result;
})();
# 定义一个函数，用于将查询字符串按特殊字符进行分割
function splitQuery(query) {
    # 初始化一个空数组，用于存储分割后的结果
    var result = [];
    # 初始化起始位置为 -1
    var start = -1;
    # 遍历查询字符串
    for (var i = 0; i < query.length; i++) {
        # 如果当前字符是特殊字符
        if (splitChars[query.charCodeAt(i)]) {
            # 如果起始位置不为 -1
            if (start !== -1) {
                # 将起始位置到当前位置的子字符串添加到结果数组中
                result.push(query.slice(start, i));
                # 重置起始位置为 -1
                start = -1;
            }
        } else if (start === -1) {
            # 如果起始位置为 -1，将当前位置作为起始位置
            start = i;
        }
    }
    # 如果起始位置不为 -1，将起始位置到字符串末尾的子字符串添加到结果数组中
    if (start !== -1) {
        result.push(query.slice(start));
    }
    # 返回分割后的结果数组
    return result;
}
```