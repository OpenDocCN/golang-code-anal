# `kubesploit\docs\_build\html\_static\searchtools.js`

```go
/*
 * searchtools.js
 * ~~~~~~~~~~~~~~~~
 *
 * Sphinx JavaScript utilities for the full-text search.
 *
 * :copyright: Copyright 2007-2020 by the Sphinx team, see AUTHORS.
 * :license: BSD, see LICENSE for details.
 *
 */

// 如果 Scorer 对象不存在，则创建一个简单的结果评分代码对象
if (!Scorer) {
  /**
   * Simple result scoring code.
   */
  var Scorer = {
    // 实现以下函数以进一步调整每个结果的评分
    // 该函数接受一个结果数组 [filename, title, anchor, descr, score]
    // 并返回新的评分
    /*
    score: function(result) {
      return result[4];
    },
    */

    // 查询匹配对象的全名
    objNameMatch: 11,
    // 或者匹配对象名称的最后一个点之后的部分
    objPartialMatch: 6,
    // 根据对象的优先级添加得分
    objPrio: {0:  15,   // used to be importantResults
              1:  5,   // used to be objectResults
              2: -5},  // used to be unimportantResults
    // 当优先级不在映射中时使用
    objPrioDefault: 0,

    // 标题中找到查询
    title: 15,
    partialTitle: 7,
    // 术语中找到查询
    term: 5,
    partialTerm: 2
  };
}

// 如果 splitQuery 函数不存在，则创建一个函数来将查询字符串拆分为单词数组
if (!splitQuery) {
  function splitQuery(query) {
    return query.split(/\s+/);
  }
}

/**
 * Search Module
 */
var Search = {

  _index : null, // 初始化搜索索引为null
  _queued_query : null, // 初始化排队查询为null
  _pulse_status : -1, // 初始化脉冲状态为-1

  htmlToText : function(htmlString) { // 定义将HTML转换为文本的函数
      var htmlElement = document.createElement('span'); // 创建一个span元素
      htmlElement.innerHTML = htmlString; // 将HTML字符串赋值给span元素的innerHTML
      $(htmlElement).find('.headerlink').remove(); // 在span元素中查找并移除类名为'headerlink'的元素
      docContent = $(htmlElement).find('[role=main]')[0]; // 在span元素中查找具有role属性为'main'的元素
      if(docContent === undefined) { // 如果找不到内容块
          console.warn("Content block not found. Sphinx search tries to obtain it " +
                       "via '[role=main]'. Could you check your theme or template."); // 输出警告信息
          return ""; // 返回空字符串
      }
      return docContent.textContent || docContent.innerText; // 返回内容块的文本内容
  },

  init : function() { // 定义初始化函数
      var params = $.getQueryParameters(); // 获取查询参数
      if (params.q) { // 如果存在查询参数
          var query = params.q[0]; // 获取查询参数的值
          $('input[name="q"]')[0].value = query; // 将查询参数的值赋值给input元素的value属性
          this.performSearch(query); // 执行搜索
      }
  },

  loadIndex : function(url) { // 定义加载索引的函数
    $.ajax({type: "GET", url: url, data: null, // 发送GET请求
            dataType: "script", cache: true, // 设置数据类型为script，启用缓存
            complete: function(jqxhr, textstatus) { // 请求完成后执行回调函数
              if (textstatus != "success") { // 如果请求不成功
                document.getElementById("searchindexloader").src = url; // 设置searchindexloader元素的src属性为url
              }
            }});
  },

  setIndex : function(index) { // 定义设置索引的函数
    var q;
    this._index = index; // 设置索引
    if ((q = this._queued_query) !== null) { // 如果存在排队查询
      this._queued_query = null; // 清空排队查询
      Search.query(q); // 执行查询
    }
  },

  hasIndex : function() { // 定义判断是否存在索引的函数
      return this._index !== null; // 返回索引是否不为null
  },

  deferQuery : function(query) { // 定义延迟查询的函数
      this._queued_query = query; // 将查询加入排队查询
  },

  stopPulse : function() { // 定义停止脉冲的函数
      this._pulse_status = 0; // 将脉冲状态设置为0
  },

  startPulse : function() { // 定义开始脉冲的函数
    if (this._pulse_status >= 0) // 如果脉冲状态大于等于0
        return; // 返回
    function pulse() { // 定义脉冲函数
      var i;
      Search._pulse_status = (Search._pulse_status + 1) % 4; // 更新脉冲状态
      var dotString = ''; // 初始化点字符串
      for (i = 0; i < Search._pulse_status; i++) // 循环脉冲状态次数
        dotString += '.'; // 添加点
      Search.dots.text(dotString); // 设置脉冲点的文本内容
      if (Search._pulse_status > -1) // 如果脉冲状态大于-1
        window.setTimeout(pulse, 500); // 延迟500毫秒后执行脉冲函数
    }
    // 调用pulse函数
    pulse();
  },

  /**
   * 执行搜索操作（或等待索引加载）
   */
  performSearch : function(query) {
    // 创建必需的界面元素
    this.out = $('#search-results');
    this.title = $('<h2>' + _('Searching') + '</h2>').appendTo(this.out);
    this.dots = $('<span></span>').appendTo(this.title);
    this.status = $('<p class="search-summary">&nbsp;</p>').appendTo(this.out);
    this.output = $('<ul class="search"/>').appendTo(this.out);

    // 设置搜索进度文本
    $('#search-progress').text(_('Preparing search...'));
    // 开始脉冲动画
    this.startPulse();

    // 如果索引已经加载，浏览器反应迅速
    if (this.hasIndex())
      this.query(query);
    else
      this.deferQuery(query);
  },

  /**
   * 执行搜索操作（需要搜索索引已加载）
   */
  query : function(query) {
    var i;

    // 对搜索词进行词干处理并添加到正确的列表中
    var stemmer = new Stemmer();
    var searchterms = [];
    var excluded = [];
    var hlterms = [];
    var tmp = splitQuery(query);
    var objectterms = [];
    for (i = 0; i < tmp.length; i++) {
      if (tmp[i] !== "") {
          objectterms.push(tmp[i].toLowerCase());
      }

      if ($u.indexOf(stopwords, tmp[i].toLowerCase()) != -1 || tmp[i] === "") {
        // 跳过这个“词”
        continue;
      }
      // 对单词进行词干处理
      var word = stemmer.stemWord(tmp[i].toLowerCase());
      // 防止词干处理将单词缩短到两个字符以下
      if(word.length < 3 && tmp[i].length >= 3) {
        word = tmp[i];
      }
      var toAppend;
      // 选择正确的列表
      if (word[0] == '-') {
        toAppend = excluded;
        word = word.substr(1);
      }
      else {
        toAppend = searchterms;
        hlterms.push(tmp[i].toLowerCase());
      }
      // 只有在列表中不存在时才添加
      if (!$u.contains(toAppend, word))
        toAppend.push(word);
    }
    // 构建高亮字符串
    var highlightstring = '?highlight=' + $.urlencode(hlterms.join(" "));
    // 输出搜索信息到控制台，用于调试
    // 输出需要搜索的必要条件
    // 输出需要排除的条件

    // 准备搜索
    var terms = this._index.terms;
    var titleterms = this._index.titleterms;

    // 存储搜索结果的数组，每个元素包含 [文件名, 标题, 锚点, 描述, 分数]
    var results = [];
    $('#search-progress').empty();

    // 作为对象进行查找
    for (i = 0; i < objectterms.length; i++) {
      var others = [].concat(objectterms.slice(0, i),
                             objectterms.slice(i+1, objectterms.length));
      results = results.concat(this.performObjectSearch(objectterms[i], others));
    }

    // 作为搜索条件在全文中进行查找
    results = results.concat(this.performTermsSearch(searchterms, excluded, terms, titleterms));

    // 让评分器使用自定义评分函数覆盖分数
    if (Scorer.score) {
      for (i = 0; i < results.length; i++)
        results[i][4] = Scorer.score(results[i]);
    }

    // 现在按分数排序（以相反的顺序出现，因为下面的显示函数使用pop()来检索项目），然后按字母顺序排序
    results.sort(function(a, b) {
      var left = a[4];
      var right = b[4];
      if (left > right) {
        return 1;
      } else if (left < right) {
        return -1;
      } else {
        // 相同的分数：按字母顺序排序
        left = a[1].toLowerCase();
        right = b[1].toLowerCase();
        return (left > right) ? -1 : ((left < right) ? 1 : 0);
      }
    });

    // 用于调试
    // Search.lastresults = results.slice();  // 复制一份
    // console.info('search results:', Search.lastresults);

    // 打印结果
    var resultCount = results.length;
    }
    displayNextItem();
  },

  /**
   * 搜索对象名称
   */
  performObjectSearch : function(object, otherterms) {
    var filenames = this._index.filenames;
    var docnames = this._index.docnames;
    var objects = this._index.objects;
    // 获取索引对象中的对象名称
    var objnames = this._index.objnames;
    // 获取索引对象中的标题
    var titles = this._index.titles;

    // 初始化变量 i
    var i;
    // 初始化结果数组
    var results = [];

    // 返回结果数组
    return results;
  },

  /**
   * 在索引中搜索全文术语
   */
  performTermsSearch : function(searchterms, excluded, terms, titleterms) {
    // 获取索引对象中的文档名称
    var docnames = this._index.docnames;
    // 获取索引对象中的文件名
    var filenames = this._index.filenames;
    // 获取索引对象中的标题
    var titles = this._index.titles;

    // 初始化变量 i, j, file
    var i, j, file;
    // 创建文件映射对象
    var fileMap = {};
    // 创建分数映射对象
    var scoreMap = {};
    // 初始化结果数组
    var results = [];

    // 在所需术语上执行搜索
    # 遍历搜索词列表
    for (i = 0; i < searchterms.length; i++) {
      # 获取当前搜索词
      var word = searchterms[i];
      # 初始化文件列表
      var files = [];
      # 初始化搜索结果对象
      var _o = [
        {files: terms[word], score: Scorer.term},
        {files: titleterms[word], score: Scorer.title}
      ];
      # 添加对部分匹配的支持
      if (word.length > 2) {
        # 遍历 terms 对象
        for (var w in terms) {
          # 如果 w 包含 word 并且 terms[word] 不存在
          if (w.match(word) && !terms[word]) {
            # 将部分匹配的结果添加到搜索结果对象中
            _o.push({files: terms[w], score: Scorer.partialTerm})
          }
        }
        # 遍历 titleterms 对象
        for (var w in titleterms) {
          # 如果 w 包含 word 并且 titleterms[word] 不存在
          if (w.match(word) && !titleterms[word]) {
              # 将部分匹配的结果添加到搜索结果对象中
              _o.push({files: titleterms[w], score: Scorer.partialTitle})
          }
        }
      }

      # 如果所有搜索结果都为 undefined，则跳出循环
      if ($u.every(_o, function(o){return o.files === undefined;})) {
        break;
      }
      # 遍历搜索结果对象
      $u.each(_o, function(o) {
        var _files = o.files;
        # 如果文件列表为 undefined，则跳过
        if (_files === undefined)
          return

        # 如果文件列表长度为 undefined，则转换为数组
        if (_files.length === undefined)
          _files = [_files];
        # 将文件列表合并到总文件列表中
        files = files.concat(_files);

        # 为每个文件中的搜索词设置得分
        for (j = 0; j < _files.length; j++) {
          file = _files[j];
          # 如果文件不在得分映射中，则添加
          if (!(file in scoreMap))
            scoreMap[file] = {};
          # 设置搜索词的得分
          scoreMap[file][word] = o.score;
        }
      });

      # 创建文件到搜索词的映射
      for (j = 0; j < files.length; j++) {
        file = files[j];
        # 如果文件在映射中并且不包含当前搜索词，则添加到映射中
        if (file in fileMap && fileMap[file].indexOf(word) === -1)
          fileMap[file].push(word);
        else
          fileMap[file] = [word];
      }
    }

    # 检查文件是否包含排除的搜索词
    for (file in fileMap) {
      var valid = true;

      // 检查是否所有要求都匹配
      var filteredTermCount = // as search terms with length < 3 are discarded: ignore
        searchterms.filter(function(term){return term.length > 2}).length
      if (
        fileMap[file].length != searchterms.length &&  // 如果文件对应的搜索项数量不等于搜索项的数量
        fileMap[file].length != filteredTermCount  // 或者文件对应的搜索项数量不等于过滤后的搜索项数量
      ) continue;  // 则跳过当前循环，继续下一个文件的检查

      // 确保搜索结果中没有任何被排除的项
      for (i = 0; i < excluded.length; i++) {
        if (terms[excluded[i]] == file ||  // 如果文件名等于被排除的项
            titleterms[excluded[i]] == file ||  // 或者标题中的项等于被排除的项
            $u.contains(terms[excluded[i]] || [], file) ||  // 或者文件名包含被排除的项
            $u.contains(titleterms[excluded[i]] || [], file)) {  // 或者标题中包含被排除的项
          valid = false;  // 则将 valid 置为 false
          break;  // 并跳出循环
        }
      }

      // 如果仍然是有效的结果，将其添加到结果列表中
      if (valid) {
        // 为文件选择一个（最大）分数。
        // 为了更好的排名，我们应该使用词频-逆文档频率等单词统计数据来计算排名...
        var score = $u.max($u.map(fileMap[file], function(w){return scoreMap[file][w]}));
        results.push([docnames[file], titles[file], '', null, score, filenames[file]]);
      }
    }
    return results;
  },

  /**
   * 用于返回包含给定文本的搜索摘要的辅助函数。keywords 是一个词干化后的单词列表，hlwords 是未经词干化的单词列表。
   * 第一个用于查找出现次数，后者用于对其进行高亮显示。
   */
  makeSearchSummary : function(htmlText, keywords, hlwords) {
    var text = Search.htmlToText(htmlText);  // 将 HTML 文本转换为纯文本
    var textLower = text.toLowerCase();  // 将文本转换为小写
    var start = 0;  // 初始化起始位置为 0
    $.each(keywords, function() {
      var i = textLower.indexOf(this.toLowerCase());  // 查找关键词在文本中的位置
      if (i > -1)
        start = i;  // 如果找到，则更新起始位置
    });
    start = Math.max(start - 120, 0);  // 将起始位置向前偏移 120，最小为 0
    // 根据条件判断是否在摘录文本前添加省略号
    var excerpt = ((start > 0) ? '...' : '') +
      // 从指定位置开始截取文本，并去除两侧空白字符
      $.trim(text.substr(start, 240)) +
      // 根据条件判断是否在摘录文本后添加省略号
      ((start + 240 - text.length) ? '...' : '');
    // 创建一个包含摘录文本的 div 元素
    var rv = $('<div class="context"></div>').text(excerpt);
    // 遍历高亮词列表，对摘录文本进行高亮处理
    $.each(hlwords, function() {
      rv = rv.highlightText(this, 'highlighted');
    });
    // 返回包含摘录文本的 div 元素
    return rv;
  }
# 结束一个 JavaScript 对象的定义
};

# 当文档加载完成后执行的函数
$(document).ready(function() {
  # 初始化搜索功能
  Search.init();
});
```