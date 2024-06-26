# 实验原理

XSS 即（Cross Site Scripting）中文名称为：跨站脚本攻击。XSS的重点不在于跨站点，而在于脚本的执行。

XSS的原理是：恶意攻击者在web页面中会插入一些恶意的script代码。当用户浏览该页面的时候，那么嵌入到web页面中script代码会执行，因此会达到恶意攻击用户的目的。类似SQL注入攻击，都是将数据当作代码执行导致的问题。

XSS攻击最主要有如下分类：反射型、存储型、及 DOM-based型。 反射性和DOM-baseed型可以归类为非持久性XSS攻击。存储型可以归类为持久性XSS攻击。


## 1 反射型XSS攻击

反射型XSS攻击是将注入的恶意脚本添加到一个网址中，然后给用户发送这个网址。一旦用户打开这个网址，就会执行脚本并导致攻击。攻击负载和脚本跟随用户点击链接，并被嵌入到响应中，在浏览器上执行。

### 1.1 反射型XSS攻击的步骤

（1）攻击者构造一个带有恶意脚本的链接，其链接参数包含用户的输入。

（2）将链接发送给受害者。

（3）受害者点击链接时，恶意脚本会被浏览器解析并执行，从而执行攻击者的意图。

<center><img src="../assets/13.PNG" width = 600></center>

### 1.2 反射型XSS攻击的示例

（1）攻击者针对 http://www.example.com 的一个搜索页，在搜索页面的输入信息的对话框中输入如下脚本（query等号后面的内容） ，那么url上就会显示如下链接，且页面又反射出的攻击信息出现。

```
http://www.example.com/search?query=<script>alert('XSS')</script>
```

（2）攻击者将连接发送给被攻击者

（3）被攻击者点开链接后，会执行嵌入的XSS脚本，从而实现攻击者的意图。

### 1.3 反射型XSS攻击的防范

**入参的强校验&过滤**： 服务器端对参数进行强校验，检查是否存在不安全的字符或脚本（<,>,/等），并过滤掉它们。所有恶意代码将被替换为相应的字符，它们将被禁止对用户浏览器执行。

**输出编码/转义**：将用户的输入作为消息从服务器返回时，确保将HTML标签和JavaScript等脚本代码中的特殊字符转义或编码。例如，将<>等字符编码为 <和>以避免它们被浏览器解释为HTML标签。

**使用HTTPOnly cookie**：HTTPOnly cookie在请求不被攻击者利用基于脚本的执行语言时无法访问，也不能通过document.cookie来访问。

**使用安全控件**：对于特殊页面（例如登录页面），使用验证码和其他安全性控件。


## 2 存储型XSS攻击

存储型 XSS 攻击指的是攻击者将恶意脚本提交到受害网站的数据库中，当其他用户浏览包含该恶意脚本链接的页面时，就会执行该脚本，从而导致攻击者的目的得以实现。由于是将恶意脚本保存在数据库中，所有访问包含恶意代码的页面的用户都受到攻击。

### 2.1 存储型XSS攻击的步骤

（1）攻击者在受害网站上查找存在漏洞的输入表单，例如评论框或信息框等。

（2）攻击者将恶意代码或脚本插入到输入表单中，以便在提交表单时存储到数据库中。例如，攻击者可以在评论框中插入一段 JavaScript 代码，用于窃取存储在 Cookie 中的会话标识符。

（3）网站接收到含有恶意代码的表单数据，将其存储到数据库中。此时，攻击者的恶意代码已经写入到数据库中并保存下来。

（4）受害用户访问这个包含恶意代码的页面时，恶意代码从数据库中提取出来并在受害用户的浏览器上执行，触发攻击者设定的操作。

（5）攻击者利用受害用户的会话标记等获取受害者的身份和敏感信息。例如，可以利用恶意脚本窃取用户的个人信息、登录凭据或信用卡信息，并发送给攻击者。

<center><img src="../assets/14.png" width = 600></center>


### 2.2 存储型XSS攻击的示例

（1）攻击者针对 http://www.seed-server.com 中的个人主页修改中嵌入如下信息

```
<script>警告脚本;</script>  //警告脚本可以是 alert("XSS")
```

（2）被攻击者访问该攻击者的主页

（3）被攻击者点开链接后，页面会显示攻击者嵌入的脚本 XSS ，从而实现攻击者的意图。


### 2.3 存储型XSS攻击的防范

**输入过滤和验证**：对用户的输入进行强校验。过滤不安全的字符，校验数据类型、长度和格式等是否合法，防止不安全的数据被存储。

**输出编码/转义**：在输出用户数据之前，对数据进行编码转义，可以使用 HTML 或 URL 编码来处理特殊字符、脚本和标记，以防止恶意代码被执行。

**防御性编程**：使用安全的 API、利用验证机制、进行限制访问控制。

**内容安全策略（Content Security Policy）**： CSP 可以设置白名单和黑名单，限制页面加载的资源类型和来源，防止恶意脚本和样式被加载。

**使用 HTTPS**：使用 HTTPS 可以防止攻击者在传输过程中窃取会话标识符和敏感数据等信息。

**限制和控制用户输入**：限制用户可以输入的数据内容、长度和格式。


## 3 DOM型XSS攻击

DOM 型 XSS 攻击是一种利用 DOM 基于 HTML 解析过程中的安全漏洞进行的跨站攻击。DOM 型 XSS 攻击不涉及服务器的参与，完全基于客户端的机制，攻击者通过在客户端篡改网页中的 DOM 元素和属性，注入恶意代码进而达到攻击目的。需要特别注意以下的用户输入源 document.URL、location.hash、location.search、document.referrer 等。

### 3.1 DOM型XSS攻击的步骤

（1）攻击者诱导用户访问一个恶意网站或者跨站点的合法网站。

（2）网站中的 JavaScript 脚本将用户输入的数据组合成 DOM 片段。

（3）攻击者篡改了 DOM 片段或者修改了 DOM 的属性，注入恶意的脚本，从而执行了非法行为。

（4）当浏览器解析 DOM 片段时，执行了恶意脚本，使恶意代码被执行。

（5）攻击者成功地窃取了用户敏感信息或者完成了其他非法操作。

### 3.2 DOM型XSS攻击的示例

如果攻击者发给目标用户一个包含 DOM 型攻击的 url ，如下面的url所示，那么当目标用户点击这个 url 时，恶意脚本就会被注入到目标网页的 DOM 中，从而实现攻击。
 
```
http://www.seed-server.com/page?name=<script>恶意脚本</script>
```

### 3.3 DOM型XSS攻击的防范

**输入过滤和验证**：对用户的输入进行强校验。过滤不安全的字符，校验数据类型、长度和格式等是否合法，防止不安全的数据被存储。

**输出编码/转义**：在输出用户数据之前，对数据进行编码转义，可以使用 HTML 或 URL 编码来处理特殊字符、脚本和标记，以防止恶意代码被执行。

**使用 innerText 或 textContent 而不是 innerHTML**：避免将用户输入的数据直接插入到 innerHTML 中。可以使用白名单机制过滤不安全的标记，或使用innerText、textContent 等安全的API。

**内容安全策略（CSP）**：CSP 可以设置白名单和黑名单，限制页面加载的资源类型和来源，从而防止恶意脚本和样式被加载。

**安全沙箱**：应用沙盒技术限制 JavaScript 运行的环境，从而可以防止恶意 JS 脚本操作或者篡改文档 DOM 等。

**更新和升级浏览器**：定期升级浏览器，减少已知红旗漏洞的影响。

## 4 抵御 XSS 攻击

跨站脚本攻击（XSS）漏洞的根本问题是HTML允许JavaScript代码与数据混合。因此，为了解决这个根本问题，我们需要将代码与数据分离。在HTML页面中包含JavaScript代码有两种方式，一种是内联方式，另一种是链接方式。

内联方式直接将代码嵌入页面内，而链接方式则将代码放在外部文件中，然后从页面内部链接到它。

内联方式是导致跨站脚本攻击（XSS）漏洞的罪魁祸首，因为浏览器不知道代码最初来自哪里：是来自可信任的Web服务器还是来自不可信任的用户？没有这些知识，浏览器就不知道哪些代码是安全的，哪些是危险的。链接方式为浏览器提供了一个非常重要的信息，即代码的来源。然后，网站可以告诉浏览器哪些来源是可信的，这样浏览器就知道哪段代码是安全的。尽管攻击者也可以使用链接方式在输入中包含代码，但他们无法将其代码放置在那些可信任的位置。

网站如何告诉浏览器哪个代码源是可信的，这是通过使用一种称为内容安全策略（CSP）的安全机制来实现的。这种机制是专门为击败跨站脚本攻击（XSS）和点击劫持攻击而设计的。它已成为一种标准，现在大多数浏览器都支持它。CSP不仅限制JavaScript代码，还限制其他页面内容，例如限制图片、音频和视频的来源，以及限制页面是否可以放在iframe中（用于击败点击劫持攻击）。

在本次实验中，我们将只关注如何使用CSP来击败跨站脚本攻击（XSS）。