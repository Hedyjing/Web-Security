# Web-Security
## XSS(Cross Site Scripting)跨站脚本攻击
![](./mdImg/XSS原理.jpg)
### 演示

    url输入`http://localhost/?from=google<script>alert(1)<script/>`
### 原理

    content有数据变成了script脚本程序
### 危害

    1. 获取页面数据
    2. 获取Cookies: 通过document.cookie
    3. 劫持前端逻辑
    4. 发送请求
    5. 偷取网站任意数据
    6. 偷取用户资料
    7. 偷取用户密码登录态
    8. 欺骗用户
    9. .....
### 分类

    1. 反射型
      - url参数直接注入
      - url中会暴露脚本, 相对来说没有存储型隐蔽
    2. 存储型
      - 存储到DB后读取时注入
      - 评论中写入脚本, 当其他用户访问该评论页面时, 这条评论的内容会从数据库中取出所存储的脚本运行, 从而攻击用户
      - 危害更大, 因为更隐蔽
### XSS攻击注入点

  1. HTML节点内容

    如果节点的内容是动态生成, 包含用户输入的信息, 可能被插入脚本
  2. HTML属性
    
  ```html
  <!-- 属性为用户输入, 可能被输入脚本 -->
  <img src="#{image}"/>
  <img src="1" onerror="alert(1)"/>
  <!-- http://localhost/?image=1" onerror=alert(1) -->
  ```
  3. js代码
  ```js
  let data = "#{data}";
  // hello";alert(1);"    --> 黑客输入的内容
  let data = "hello";alert(1);"";
  ```
  4. 富文本

    - 一大段的html, 有个格式, 难点在于要区别用户插入的script脚
    - 比如qq邮箱的编辑器
    - 富文本得保留html
    - html有XSS攻击的风险
### 防御
  1. 浏览器的防御: 

    - X-XSS-Protection
    - 只防御反射型的XSS
    - 只防御出现在html内容和html属性中的
  ---
  2. html节点内容的防御: 

    - 对输入的内容进行转义: <&lt;和>&gt;
      1. 进入数据库之前进行转义
      2. 进行显示的时候转义
  ```js
  function escapeHtml(str) {
    str = str.replace(/</g, '&lt;')
    str = str.replace(/>/g, '&gt;')
  }
  ```
  3. html属性的防御

    1. 转义引号
    2. 转义空格
      
  ```js
  function escapeHtmlProperty(str) {
    if (!str) return '';
    str = str.replace(/"/g,'&quot;')
    str = str.replace(/'/g, '&#39')
    str = str.replace(/ /g, '&#32')
  }
  ```
  > html内容和属性的转义可以合并起来, 但是会有问题: 空格, 所以不要对空格转义
  ```js
  function escapeHtml(str) {
    if (!str) return '';
    str = str.replace(/&/g,'&amp')
    str = str.replace(/</g, '&lt;')
    str = str.replace(/>/g, '&gt;')
    str = str.replace(/"/g,'&quot;')
    str = str.replace(/'/g, '&#39')
  }
  ```
  4. js XSS攻击防御

    对数据中的引号进行转义, 但不能转义为html实体, 因为js识别不了
    1. 对引号转义
    2. 对斜杠\转义
    3. 最终方案: 最数据进行JSON.stringify
  5. 富文本的格式的防御

    - 黑名单的风险: 有太多的html标签和属性需要过滤
    - 所以按白名单保留部分标签和属性
    - 使用专门的xss过滤库: install xss
  6. CSP防御

    用于指定哪些内容可以执行, 没有指定的全部不信任`default-src self`

