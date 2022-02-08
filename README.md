# blaster



### 简介

  blaster 是一款弱密码隐患检测工具，用于网站登录弱密码检测。

  无论是在漏洞响应平台，还是日常工作中，大家或许在为图片验证码、登录加密等无法直接登录测试而烦恼，所以blaster应运而生。它支持导入用户名密码字典，多并发图片验证码识别，自动填充表单元素，无视任何登录加密。



### 内测

  在团队小伙伴内测期间，斩获多个专属厂商登录弱口令高危漏洞。

<img src="images/image-20220130144704974.png" alt="image-20220130144704974" style="zoom:50%;" />



### 功能

- **支持复选框勾选**：主要场景为阅读平台使用须知、 隐私政策单选表单等。
- **图片验证码识别**：采用python第三方库进行ocr识别。
- **无视任何加密方式**：它将用户名及密码字典填充至对应表单进行提交，无需逆向加密方式。
- **支持多并发**



### 配置

  服务端配置，该服务对应客户端配置中的 **captchabreak_serverurl**

  ```shell
  python3 cbhs.py -a user:pass	# 自定义服务端basic认证用户名及密码
  ```



  客户端配置 config.yaml

  ```yaml
  # 目标网址：http://www.example.com/login
  # 登录目标最终发送的请求数据包地址：http://www.example.com/api/user/login

  target_url: 'http://www.example.com/login' # 登录目标网址
  browser_path: 'C:\Program Files\Google\Chrome\Application\chrome.exe' # 浏览器的路径
  headless: false # 设置false显式运行浏览器 true则反之
  captchabreak_serverurl: 'http://user:pass@8.8.8.8:8080/cb' # 验证码识别服务器
  userinput_jspath: 'document.querySelector("#app > div > div.content > div.form > div.layout > div.input-cntr.input-cntr1 > input")'
  passinput_jspath: 'document.querySelector("#app > div > div.content > div.form > div.layout > div.input-cntr.input-cntr2 > input")'
  captchainput_jspath: 'document.querySelector("#app > div > div.content > div.form > div.layout > div.input-cntr.input-cntr3 > input")' # 验证码输入框js path
  captchaimg_jspath: 'document.querySelector("#app > div > div.content > div.form > div.layout > div.input-cntr.input-cntr3 > div.verification-img > img")' # 验证码图片js path
  checkbox_jspath: 'document.querySelector("#app > div > div.content > div.form > div.layout > div.agree-wrap > label > span > input")' # 复选框js path
  loginbutton_jspath: 'document.querySelector("#app > div > div.content > div.form > div.layout > button")'
  loginreq_pattern: '*user/login*' # 登录请求的url特征码，最终的登录数据包
  maxbody_bytes_display: 512 # 登录请求响应包最大限制，超过限制则不会显示
  concurrency: 1 # 并发数
  timeout_ms: 50000 # 浏览器中操作的超时时间(毫秒) 
  timeinterval_ms: 300 # 浏览器中操作登录过程中每个操作之间的时间间隔(毫秒)
  proxy: '' # 代理

  ```



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


### 下载

https://github.com/PoJun-Lab/blaster/releases

### 交流群加入

<img src="images/image-20220208103133707.png" alt="image-20220208103133707" style="zoom: 33%;" />
