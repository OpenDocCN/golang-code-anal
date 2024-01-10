# `kubesploit\docs\_build\html\_static\doctools.js`

```
/**
 * select a different prefix for underscore
 */
// 使用 Underscore.js 的 noConflict 方法，选择一个不同的前缀
$u = _.noConflict();

/**
 * make the code below compatible with browsers without
 * an installed firebug like debugger
if (!window.console || !console.firebug) {
  var names = ["log", "debug", "info", "warn", "error", "assert", "dir",
    "dirxml", "group", "groupEnd", "time", "timeEnd", "count", "trace",
    "profile", "profileEnd"];
  window.console = {};
  for (var i = 0; i < names.length; ++i)
    window.console[names[i]] = function() {};
}
 */

/**
 * small helper function to urldecode strings
 */
// 使用 jQuery 扩展方法 urldecode，对字符串进行 URL 解码
jQuery.urldecode = function(x) {
  return decodeURIComponent(x).replace(/\+/g, ' ');
};

/**
 * small helper function to urlencode strings
 */
// 使用 jQuery 扩展方法 urlencode，对字符串进行 URL 编码
jQuery.urlencode = encodeURIComponent;

/**
 * This function returns the parsed url parameters of the
 * current request. Multiple values per key are supported,
 * it will always return arrays of strings for the value parts.
 */
// 使用 jQuery 扩展方法 getQueryParameters，解析当前请求的 URL 参数
jQuery.getQueryParameters = function(s) {
  if (typeof s === 'undefined')
    s = document.location.search;
  var parts = s.substr(s.indexOf('?') + 1).split('&');
  var result = {};
  for (var i = 0; i < parts.length; i++) {
    var tmp = parts[i].split('=', 2);
    var key = jQuery.urldecode(tmp[0]);
    var value = jQuery.urldecode(tmp[1]);
    if (key in result)
      result[key].push(value);
    else
      result[key] = [value];
  }
  return result;
};

/**
 * highlight a given string on a jquery object by wrapping it in
 * span elements with the given class name.
 */
// 使用 jQuery 扩展方法 highlightText，对 jQuery 对象中的指定字符串进行高亮处理
jQuery.fn.highlightText = function(text, className) {
  function highlight(node, addItems) {
    # 检查节点类型是否为文本节点
    if (node.nodeType === 3) {
      # 获取节点的文本内容
      var val = node.nodeValue;
      # 在文本内容中查找指定文本的位置
      var pos = val.toLowerCase().indexOf(text);
      # 如果找到指定文本，并且父节点不包含指定的类名和"nohighlight"类名
      if (pos >= 0 &&
          !jQuery(node.parentNode).hasClass(className) &&
          !jQuery(node.parentNode).hasClass("nohighlight")) {
        # 创建一个新的 span 元素
        var span;
        # 检查节点是否在 SVG 中
        var isInSVG = jQuery(node).closest("body, svg, foreignObject").is("svg");
        # 如果在 SVG 中，则创建一个 tspan 元素
        if (isInSVG) {
          span = document.createElementNS("http://www.w3.org/2000/svg", "tspan");
        } else {
          # 否则创建一个普通的 span 元素，并设置类名
          span = document.createElement("span");
          span.className = className;
        }
        # 在 span 元素中添加指定文本内容
        span.appendChild(document.createTextNode(val.substr(pos, text.length)));
        # 在节点的父节点中插入新创建的 span 元素和剩余的文本内容
        node.parentNode.insertBefore(span, node.parentNode.insertBefore(
          document.createTextNode(val.substr(pos + text.length)),
          node.nextSibling));
        # 更新节点的文本内容
        node.nodeValue = val.substr(0, pos);
        # 如果在 SVG 中，则创建一个 rect 元素，并设置属性
        if (isInSVG) {
          var rect = document.createElementNS("http://www.w3.org/2000/svg", "rect");
          var bbox = node.parentElement.getBBox();
          rect.x.baseVal.value = bbox.x;
          rect.y.baseVal.value = bbox.y;
          rect.width.baseVal.value = bbox.width;
          rect.height.baseVal.value = bbox.height;
          rect.setAttribute('class', className);
          # 将创建的 rect 元素添加到 addItems 数组中
          addItems.push({
              "parent": node.parentNode,
              "target": rect});
        }
      }
    }
    # 如果节点不是按钮、选择框或文本域，则遍历其子节点并调用 highlight 函数
    else if (!jQuery(node).is("button, select, textarea")) {
      jQuery.each(node.childNodes, function() {
        highlight(this, addItems);
      });
    }
  }
  # 创建一个空数组 addItems
  var addItems = [];
  # 遍历每个元素并调用 highlight 函数
  var result = this.each(function() {
    highlight(this, addItems);
  });
  # 将 addItems 数组中的元素插入到其父节点之前
  for (var i = 0; i < addItems.length; ++i) {
    jQuery(addItems[i].parent).before(addItems[i].target);
  }
  # 返回结果
  return result;
};

/*
 * backward compatibility for jQuery.browser
 * This will be supported until firefox bug is fixed.
 */
// 如果 jQuery.browser 不存在，则定义 jQuery.uaMatch 函数
if (!jQuery.browser) {
  // 定义 jQuery.uaMatch 函数，用于匹配浏览器类型和版本
  jQuery.uaMatch = function(ua) {
    // 将用户代理字符串转换为小写
    ua = ua.toLowerCase();
    // 使用正则表达式匹配浏览器类型和版本
    var match = /(chrome)[ \/]([\w.]+)/.exec(ua) ||
      /(webkit)[ \/]([\w.]+)/.exec(ua) ||
      /(opera)(?:.*version|)[ \/]([\w.]+)/.exec(ua) ||
      /(msie) ([\w.]+)/.exec(ua) ||
      ua.indexOf("compatible") < 0 && /(mozilla)(?:.*? rv:([\w.]+)|)/.exec(ua) ||
      [];
    // 返回匹配结果
    return {
      browser: match[ 1 ] || "",
      version: match[ 2 ] || "0"
    };
  };
  // 定义 jQuery.browser 为空对象
  jQuery.browser = {};
  // 将浏览器类型和版本添加到 jQuery.browser 对象中
  jQuery.browser[jQuery.uaMatch(navigator.userAgent).browser] = true;
}

/**
 * Small JavaScript module for the documentation.
 */
// 定义 Documentation 对象
var Documentation = {
  // 初始化函数
  init : function() {
    // 修复火狐浏览器锚点 bug
    this.fixFirefoxAnchorBug();
    // 高亮搜索关键词
    this.highlightSearchWords();
    // 初始化索引表
    this.initIndexTable();
    // 如果开启了键盘导航，则初始化键盘监听器
    if (DOCUMENTATION_OPTIONS.NAVIGATION_WITH_KEYS) {
      this.initOnKeyListeners();
    }
  },

  /**
   * i18n support
   */
  // 翻译支持
  TRANSLATIONS : {},
  // 复数表达式函数
  PLURAL_EXPR : function(n) { return n === 1 ? 0 : 1; },
  // 本地化
  LOCALE : 'unknown',

  // gettext 和 ngettext 函数不访问 this，因此可以安全地绑定到不同的名称（_ = Documentation.gettext）
  // 获取翻译文本
  gettext : function(string) {
    var translated = Documentation.TRANSLATIONS[string];
    if (typeof translated === 'undefined')
      return string;
    return (typeof translated === 'string') ? translated : translated[0];
  },
  // 获取复数形式的翻译文本
  ngettext : function(singular, plural, n) {
    var translated = Documentation.TRANSLATIONS[singular];
    if (typeof translated === 'undefined')
      return (n == 1) ? singular : plural;
    return translated[Documentation.PLURALEXPR(n)];
  },
  // 添加翻译
  addTranslations : function(catalog) {
    for (var key in catalog.messages)
      this.TRANSLATIONS[key] = catalog.messages[key];
    this.PLURAL_EXPR = new Function('n', 'return +(' + catalog.plural_expr + ')');
  // 将全局变量 this.LOCALE 设置为 catalog.locale
  this.LOCALE = catalog.locale;
  },

  /**
   * 添加上下文元素，比如标题锚点链接
   */
  addContextElements : function() {
    // 遍历每个 div 元素下的第一个标题元素，为其添加一个锚点链接
    $('div[id] > :header:first').each(function() {
      $('<a class="headerlink">\u00B6</a>').
      attr('href', '#' + this.id).
      attr('title', _('Permalink to this headline')).
      appendTo(this);
    });
    // 遍历每个带有 id 属性的 dt 元素，为其添加一个锚点链接
    $('dt[id]').each(function() {
      $('<a class="headerlink">\u00B6</a>').
      attr('href', '#' + this.id).
      attr('title', _('Permalink to this definition')).
      appendTo(this);
    });
  },

  /**
   * 解决火狐浏览器的 bug
   * 参考：https://bugzilla.mozilla.org/show_bug.cgi?id=645075
   */
  fixFirefoxAnchorBug : function() {
    // 如果当前页面有锚点并且是火狐浏览器，则通过 setTimeout 来解决火狐浏览器的 bug
    if (document.location.hash && $.browser.mozilla)
      window.setTimeout(function() {
        document.location.href += '';
      }, 10);
  },

  /**
   * 在文本中突出显示 URL 中提供的搜索词
   */
  highlightSearchWords : function() {
    // 获取 URL 中的参数
    var params = $.getQueryParameters();
    // 如果存在 highlight 参数，则将其拆分成搜索词数组
    var terms = (params.highlight) ? params.highlight[0].split(/\s+/) : [];
    // 如果搜索词数组不为空
    if (terms.length) {
      // 获取文本内容的容器
      var body = $('div.body');
      // 如果找不到文本内容的容器，则使用 body 元素
      if (!body.length) {
        body = $('body');
      }
      // 通过 setTimeout 遍历搜索词数组，为每个搜索词添加突出显示样式
      window.setTimeout(function() {
        $.each(terms, function() {
          body.highlightText(this.toLowerCase(), 'highlighted');
        });
      }, 10);
      // 在搜索框下添加一个链接，用于隐藏搜索匹配结果
      $('<p class="highlight-link"><a href="javascript:Documentation.' +
        'hideSearchWords()">' + _('Hide Search Matches') + '</a></p>')
          .appendTo($('#searchbox'));
    }
  },

  /**
   * 初始化域索引切换按钮
   */
  initIndexTable : function() {
    // 选择所有 class 为 toggler 的 img 元素，并绑定点击事件处理函数
    var togglers = $('img.toggler').click(function() {
      // 获取当前 img 元素的 src 属性值
      var src = $(this).attr('src');
      // 获取当前 img 元素的 id 属性值，并截取出数字部分
      var idnum = $(this).attr('id').substr(7);
      // 选择所有 class 包含 cg- 和 idnum 的 tr 元素，并切换显示状态
      $('tr.cg-' + idnum).toggle();
      // 根据当前图片的 src 属性值判断，切换图片的显示状态
      if (src.substr(-9) === 'minus.png')
        $(this).attr('src', src.substr(0, src.length-9) + 'plus.png');
      else
        $(this).attr('src', src.substr(0, src.length-8) + 'minus.png');
    }).css('display', '');
    // 如果文档配置中设置了折叠索引，执行 togglers 的点击事件处理函数
    if (DOCUMENTATION_OPTIONS.COLLAPSE_INDEX) {
        togglers.click();
    }
  },

  /**
   * 辅助函数，用于隐藏搜索标记
   */
  hideSearchWords : function() {
    // 选择所有 class 为 highlight-link 的元素，并淡出隐藏
    $('#searchbox .highlight-link').fadeOut(300);
    // 移除所有 class 为 highlighted 的元素的 highlighted 类
    $('span.highlighted').removeClass('highlighted');
  },

  /**
   * 将相对 URL 转换为绝对 URL
   */
  makeURL : function(relativeURL) {
    // 返回相对 URL 拼接上文档配置中的 URL_ROOT
    return DOCUMENTATION_OPTIONS.URL_ROOT + '/' + relativeURL;
  },

  /**
   * 获取当前页面的相对 URL
   */
  getCurrentURL : function() {
    // 获取当前页面的路径
    var path = document.location.pathname;
    // 根据 / 分割路径
    var parts = path.split(/\//);
    // 遍历文档配置中的 URL_ROOT，根据 .. 移除 parts 中的路径部分
    $.each(DOCUMENTATION_OPTIONS.URL_ROOT.split(/\//), function() {
      if (this === '..')
        parts.pop();
    });
    // 拼接 parts 中的路径为 URL，并返回当前页面的相对 URL
    var url = parts.join('/');
    return path.substring(url.lastIndexOf('/') + 1, path.length - 1);
  },

  // 初始化键盘事件监听器
  initOnKeyListeners: function() {
    # 当按下键盘时触发事件
    $(document).keydown(function(event) {
      # 获取当前活动元素的标签类型
      var activeElementType = document.activeElement.tagName;
      # 如果当前活动元素不是文本框、输入框或下拉框，并且没有按下特殊键，则执行以下操作
      if (activeElementType !== 'TEXTAREA' && activeElementType !== 'INPUT' && activeElementType !== 'SELECT'
          && !event.altKey && !event.ctrlKey && !event.metaKey && !event.shiftKey) {
        # 根据按下的键码执行相应操作
        switch (event.keyCode) {
          # 当按下左箭头键时
          case 37: // left
            # 获取上一页的链接地址
            var prevHref = $('link[rel="prev"]').prop('href');
            # 如果存在上一页链接，则跳转到上一页并阻止默认行为
            if (prevHref) {
              window.location.href = prevHref;
              return false;
            }
          # 当按下右箭头键时
          case 39: // right
            # 获取下一页的链接地址
            var nextHref = $('link[rel="next"]').prop('href');
            # 如果存在下一页链接，则跳转到下一页并阻止默认行为
            if (nextHref) {
              window.location.href = nextHref;
              return false;
            }
        }
      }
    });
  }
// 快速为翻译创建别名
_ = Documentation.gettext;

// 当文档加载完成后执行初始化函数
$(document).ready(function() {
  Documentation.init();
});
```