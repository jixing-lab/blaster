# blaster



### 简介

  blaster 是一款弱密码隐患检测工具，用于网站登录弱密码检测。

  无论是在漏洞响应平台，还是日常工作中，大家或许在为图片验证码、登录加密等无法直接登录测试而烦恼，所以blaster应运而生。它支持导入用户名密码字典，多并发图片验证码识别，自动填充表单元素，无视任何登录加密。



### 内测

  在团队小伙伴内测期间，斩获多个专属厂商登录弱口令高危漏洞。

<img src="images/image-20220130144704974.png" alt="image-20220130144704974" style="zoom:50%;" />



### 功能

- **支持复选框勾选**：主要场景为阅读平台使用须知、 隐私政策单选表单等。
- **支持复杂场景的事件操作**：可根据不同场景，在登录前后自定义事件操作。
- **支持正则匹配反向过滤**：应对不同响应类型、长度，提取关键信息进行反向过滤。
- **图片验证码识别**：采用python第三方库进行ocr识别。
- **无视任何加密方式**：它将用户名及密码字典填充至对应表单进行提交，无需逆向加密方式。
- **支持多次输入账号密码错误才出现验证码情况**
- **支持多并发**



### 配置



  服务端搭建，该服务对应客户端配置中的 **captchabreak_serverurl**，它用于图片验证码识别。

  ```shell
python3 cbhs.py -a user:pass -p port	# 自定义服务端basic认证用户名密码、开放端口
  ```



  客户端配置 config.yaml

  ```yaml
# 示例目标网址：http://www.example.com/login
# 示例登录目标最终发送的请求数据包地址：http://www.example.com/api/user/login
# 示例图片验证码识别服务：python3 cbhs.py -a admin:123 -p 8999

target_url: 'http://www.example.com/login' # 登录目标网址
browser_path: 'C:\Program Files\Google\Chrome\Application\chrome.exe' # 浏览器的路径
headless: false # 设置false显式运行浏览器 true则反之
captchabreak_serverurl: 'http://admin:123@127.0.0.1:8999/cb' # 图片验证码识别服务，使用captchabreak_serverurl文件夹中的脚本进行搭建
before_all_js_expr: '' # 在开始填写账号密码之前执行的js表达式
userinput_jspath: 'document.querySelector("#loginid")'	# 用户名输入框js path
passinput_jspath: 'document.querySelector("#security_in")'	# 密码输入框js path
captchainput_jspath: 'document.querySelector("#jcaptcha")'	# 验证码输入框js path
captchaimg_jspath: 'document.querySelector("#vcode")'	# 验证码图片js path
before_login_js_expr: '' # 在点击登录之前执行的js表达式
loginbutton_jspath: 'document.querySelector("#vcode")'
loginreq_pattern: '*user/login*' # 登录请求的url特征码，实际登录数据包url path
body_exclude_regex: # 排除请求的正则，即只要命中其中任意一个正则的请求响应将被抛弃
  - 'regex1'
  - 'regex2'
maxbody_bytes_display: 512 # 登录请求响应包最大限制，超过限制则不会显示
concurrency: 1 # 并发数
timeout_ms: 50000 # 浏览器中操作的超时时间(毫秒)
timeinterval_ms: 300 # 浏览器中操作登录过程中每个操作之间的时间间隔(毫秒)
proxy: '' # 代理
  ```



- captchabreak_serverurl

    - 用于指定图片验证码服务，格式为 http://user:pass@127.0.0.1:port/cb

- before_all_js_expr

    - 在开始填写账号密码之前执行的js表达式，如页面打开后有弹出层，可以将通过该参数关闭弹窗，不局限于弹出层一个场景。

    - 示例场景：打开页面出现弹出层，需要关闭弹窗后输入账号密码。

    - 示例解决弹出层：判断某按钮（弹出层关闭按钮X）是否存在，如果存在则点击；找出关闭按钮的js path按照格式编写代码即可。

    - ```js
        null != jspath && jspath.click()	// js代码格式
        ```

    - 表达式不限于一个，如有其他场景或多个弹出层，可通过 && 或 || 进行叠加

- before_login_js_expr

    - 在点击登录之前执行的js表达式，如登录需要勾选阅读平台使用须知、 隐私政策单选表单等

    - 示例场景：输入账号密码后需要选择阅读平台使用须知、 隐私政策等表单才可登录

    - 示例解决表单勾选：判断某勾选框是否已经勾选如果未勾选则点击；找出表单勾选的js path按照格式编写代码即可。

    - ```js
        !jspath.checked && jspath.click() // js代码格式
        ```

    - 表达式不限于一个，如有其他场景或多个勾选，可通过 && 或 || 进行叠加

- body_exclude_regex

    - 排除请求的正则，即只要命中其中任意一个正则的请求响应将被抛弃，即反向grep。

    - 示例场景：登录后的响应页面长度较大，返回格式为HTML，其中有（账号不存在、密码错误、验证码错误、登录成功）等不同情况，由于响应长度较大我们无法轻易区分测试数据的真实响应情况。

    - 示例解决响应长度较大问题：经过手工登录测试发现响应的HTML中出现这些关键信息（如账号不存在、密码错误、验证码错误），根据自己的需求，编写正则表达式，在发现响应中存在这些匹配内容时不输出到终端及结果文件中。

    - 这里需要具备一些正则表达式的技能，如果使用者技能有些欠缺，可以直接使用关键信息进行排除，或是在交流群内寻求帮助。

    - ```yaml
        body_exclude_regex:
          - '账号不存在'
          - '验证码错误'
        ```

    - 该参数不限于一个表达式，使用者根据自己的需求来增删表达式即可。

  客户端配置中的jspath可在浏览器页面对应表单右键选择检查(Inspect)，并在检查中右键选中的表单标签，选择Copy>Copy JS path即可复制jspath。

<img src="images/image-20220128184615741.png" alt="image-20220128184615741" style="zoom:50%;" />



<img src="images/image-20220128185148609.png" alt="image-20220128185148609" style="zoom: 33%;" />



### 使用



  ```shell
  C:\Users\balster>blaster_win.exe
  Usage of blaster_win.exe:
    -c string
          config file	# 指定config.yaml
    -o string
          output file path (optional)	# 暴力破解测试数据输出文件位置
    -p string
          pass dict file path	# 指定密码字典
    -u string
          user dict file path	# 指定用户名字典
  ```



  ```shell
  C:\Users\blaster>blaster_win.exe -c conf.yaml -u user.txt -p pwds.txt -o res.csv
  2022/01/28 19:09:08 200 OPTIONS admin   admin   0
  2022/01/28 19:09:08 200 POST    admin   admin   86      {"result":-1,"errorCode":"10005","title":"fail","description":"10005","retValue":null}
  2022/01/28 19:09:10 200 POST    admin   admin123        135     {"result":-1,"errorCode":"10011","title":"fail","description":"密码过于简单，请使用手机号验证码登录","retValue":null}
  2022/01/28 19:09:13 200 POST    admin   123456  135     {"result":-1,"errorCode":"10011","title":"fail","description":"密码过于简单，请使用手机号验证码登录","retValue":null}
  2022/01/28 19:09:16 200 POST    admin   1234567 135     {"result":-1,"errorCode":"10011","title":"fail","description":"密码过于简单，请使用手机号验证码登录","retValue":null}
  2022/01/28 19:09:18 200 POST    admin   12345678        135     {"result":-1,"errorCode":"10011","title":"fail","description":"密码过于简单，请使用手机号验证码登录","retValue":null}
  2022/01/28 19:09:21 200 POST    admin   password        86      {"result":-1,"errorCode":"10005","title":"fail","description":"10005","retValue":null}
  2022/01/28 19:09:24 200 POST    admin   Aa123456.       86      {"result":-1,"errorCode":"10001","title":"fail","description":"10001","retValue":null}
  2022/01/28 19:09:26 200 POST    admin   p@$$w0rd        86      {"result":-1,"errorCode":"10005","title":"fail","description":"10005","retValue":null}
  2022/01/28 19:09:29 200 POST    admin   1q2W#e4r        86      {"result":-1,"errorCode":"10001","title":"fail","description":"10001","retValue":null}
  2022/01/28 19:09:32 200 POST    admin   P@$$w0rd        126     {"result":-1,"errorCode":"10008","title":"fail","description":"密码错误次数已满，请明天再登录","retValue":null}
  2022/01/28 19:09:34 200 POST    admin   password123     126     
  ```

<img src="images/image-20220128190926538.png" alt="image-20220128190926538" style="zoom:50%;" />

<img src="images/image-20220128191129181.png" alt="image-20220128191129181" style="zoom: 33%;" />



### TODO

- 验证码识别率优化
- 其他类型验证码识别



### UPDATE

- 2022.02.16
    - [-]删除表单勾选功能checkbox_jspath参数
    - [+]增加before_all_js_expr参数，在输入账号密码之前可以执行的js表达式
    - [+]增加before_login_js_expr参数，登录之前可执行的js表达式，支持checkbox_jspath表单勾选操作，该参数更加灵活
    - [+]增加body_exclude_regex参数，设置多个正则用来排除需要抛弃的请求响应包
    - [+]增加表单value去空填充



### 下载

https://github.com/PoJun-Lab/blaster/releases

### 交流群加入

群满请扫描下方二维码邀请进入，备注blaster

<img src="images/image-20220211111225264.png" alt="image-20220211111225264" style="zoom: 33%;" />
