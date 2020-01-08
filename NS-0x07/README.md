# Web 应用漏洞攻防

## 实验要求

- [x] 每个实验环境完成不少于 5 种不同漏洞类型的漏洞利用练习；
- [x] 定位缺陷代码
- [x] 对源码部分进行改进

## 完成项目

- [x] Authentication Flows  : Forgot Password
- [x] Authentication Flows : Multi Level Login 2
- [x] Cross-Site Scripting : Stored XSS Attacks
- [x] Injection Flows : Command injection
- [x] Parameter Tampering ： Bypass HTML Field Restrictions
- [x] Session Management Flows : Session Fixation
- [x] Outdated Whitelist
- [x] Error Handling
- [x] DOM XSS
- [x] Zero Stars
- [x] Repetitive Registration
- [x] Confidential Document
- [x] Missing Encoding

## Webgoat

### Authentication Flows  : Forgot Password

这个挑战中，找回密码的机制比较弱，输入用户名，然后输入用户名对应的问题的答案，如果答案对上，则显示你的密码。而例子中的问题又非常简单——「最喜欢的颜色」，本来想找一个颜色字典写个脚本试一下，结果发现rgb试到green就出来了......

![](img/FP.jpg)

改进方法，首先肯定不能是单一问题，其次问题要尽可能隐私和个人化，使得别人不好猜出来，同时增加一些诸如手机邮箱等二次验证。最后不应该返回原密码，而是将功能改成「修改密码」。

### Authentication Flows : Multi Level Login 2

该挑战中，你拥有了Joe的用户名和密码，现在想登陆Jane的用户。首先我们登陆Joe的账号，使用burpsuite抓包看一下

![](img/ML1.jpg)

发现在提交`TAN`的时候，存在一个post的数据`hidden_user`,我们尝试将此处数值修改为Jane，成功得到Jane的信息

![](img/ML2.jpg)

源码中，登陆用户时会进行`tan`值的验证

![](img/ML3.jpg)

但是在`hidden_user`时，却没有重新对当前`tan`值和当前`hidden_user`名进行对应的验证就直接返回`hidden_user`的信息。此处应该执行上图中的类似操作，使得每个用户只能唯一查看自己的信息。

![](img/ML4.jpg)


### Cross-Site Scripting : Stored XSS Attacks

存储型xss，首先我们登陆Tom的账户。先试试搜索的功能，搜索他人的账户，得到如下信息。

![](img/CSS1.jpg)

以上这些信息，我们自己是可以修改自己的个人信息的，同时别人是可以查到这些信息的，这也就给了我们XSS的利用空间。将个人信息中的`Last Name`修改成`<script language="javascript" type="text/javascript">alert("Ha Ha Ha");</script>`

![](img/CSS2.jpg)

这样当有人搜索Tom的信息的时候，就会在前端执行`alert("Ha Ha Ha");`的语句

![](img/CSS3.jpg)

同理，我们就可以使用一些更复杂的前端语句来偷取cookie或者对页面进行各种操作。改进方法的话，自然是不能直接允许用户将各种字符串都写进个人信息，要进行必要的字符串验证过滤。并且将信息全当成「字符串」在前端上显示。

### Injection Flows : Command injection

分析代码可以发现，不同文件内容的获取，是根据post参数中的文件名得到的，而命令的执行是使用简单拼接

![](img/Ci1.jpg)

同时存在一个命令数组，我们可以通过引号闭合和命令拼接的方式，执行其中的命令

![](img/Ci3.jpg)

![](img/Ci4.jpg)

至于解决方法，自然是严格过滤post数据，对引号等敏感符号进行特殊转义，将指定post以外的任何其他情况均视为不合法。同时不要进行执行命令的拼接而应该预设好。

### Parameter Tampering ： Bypass HTML Field Restrictions

检查一下源码，发现就是让我们提交的每个数据都不能符合要求，那么那burpsuite改一下即可

![](img/BHFR.jpg)

至于改进方法，前端手段是远远不够的，对于用户提交，后台必须要进行合法性的检查。

### Session Management Flows : Session Fixation

分析代码可知，如果我们在当前页面附加一个会话的SID，诱骗受害者点击，受害者会登录并且系统将这个SID与当前用户信息添加到会话中

![](img/SF1.jpg)

此时这个ID唯一对应一个用户，如果攻击者拿到这个ID，即可以受害者的账户进行页面访问。

按照实验步骤来，首先我们发一封邮件，邮件内容的连接中包含`&SID=tmp`字段

![](img/SF2.jpg)

受害者点击连接并登陆

![](img/SF3.jpg)

![](img/SF4.jpg)

此时我们带有`SID=tmp`的参数去访问页面，即可以受害者角色登录

![](img/SF5.jpg)

至于改进措施，我能想到的有以下几点

1. 禁止以get和post参数形式传SID，防止他人构造恶意链接。
2. 每次登陆都更换sid，且sid的生成应该与用户有相关性，比如登陆成功后，sid=hash(用户名+时间戳+随机数)，也不能简单地直接进行绑定。
3. 设置httpOnly为true，防止客户端xss脚本拿到cookie进行会话劫持。

## Juice Shop

### Finding the Score Board

首先找到积分板，其实也挺好猜的，/#/score-board就是其页面

![](img/fts1.jpg)

### Error Handling

次数要引发一个错误，逛网提示上说，Juice shop没怎么检查用户输入，让我们尝试向表单提交错误的输入。我们可以先去看一下登录界面是否存在sql注入，果然存在。

![](img/eh1.jpg)

### DOM XSS

要求我们通过xss执行`alert('xss')`，这里就需要一点探索了，经过短暂的摸索后发现，搜索页面会显示我们的输入

![](img/dx1.jpg)

按照要求填写xss的数据，即可实现alert

![](img/dx2.jpg)

### Zero Stars

这个题让我们给这个网站打0星。首先进入评价页面`/#/contact`，先提交一个评价，用burpsuite分析一下

![](img/zs1.jpg)

尝试将`rating`改为0，成功打0星~

![](img/zs2.jpg)

### Repetitive Registration

DRY原则就是「Don't Repeat Yourself」。题目让我们注册的时候不要重复输入注册密码，按照前几题的惯性，猜测那里应该是前端验证，burpsuite对应字段改为空。

![](img/rr1.jpg)

成功完成此题~

![](img/rr2.jpg)

### Confidential Document

查阅机密文件。提示中说让我们分析传递文件的地方。官方中文提示中，这个「传递」用的极其不好，一开始我还以为是让我们找可以用户上传文件的地方，后来去找能下载文件的地方，发现这样一个连接`http://localhost:3000/ftp/legal.md`。这里有个ftp，大概就是提示中说的delivery。

![](img/cd1.jpg)

既然是ftp，那我们尝试看看该目录底下有没有别的「机密文件」。访问`http://localhost:3000/ftp/`，果然有。

![](img/cd2.jpg)

能点能下载的文件，都访问了一边，等回来的时候发现就挑战成功了。

![](img/cd3.jpg)

### Missing Encoding

「检索Bjoern猫"乱斗模式"的照片」。一开始看这个描述我根本没看懂是在说啥。简单说就是，照片墙里现在有个看不了的照片，我们现在要把它找回来。观察网页，发现出错的图片这里有个图片地址。

![](img/mc1.jpg)

观察提交的数据，发现该照片的地址没有传完全（遇到`#`号之后就断开了），而别的可显示图片的地址在请求的时候，是经过URL编码的。

![](img/mc2.jpg)

![](img/mc3.jpg)

我们将请求地址经过url编码再访问，即可获得正确图片~

![](img/mc4.jpg)

![](img/mc5.jpg)


