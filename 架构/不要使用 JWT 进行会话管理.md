# [不要使用 JWT 进行会话管理](https://www.cnblogs.com/JacZhu/p/9779975.html)

*英文原文地址：[Stop using JWT for sessions](http://cryto.net/~joepie91/blog/2016/06/13/stop-using-jwt-for-sessions/)*

最近我发现越来越多的人推荐使用 JWT 来在 Web 应用中管理会话（Session），这是一个非常非常糟糕的主意，在这篇文章中我会详细地解释为什么“不要使用 JWT 进行会话管理”。

为了方便叙述的同时避免混淆，我需要先定义一些概念：

- 无状态 JWT（Stateless JWT）：一个包含了所有会话相关数据的 JWT token。
- 有状态 JWT（Stateful JWT）：一个仅包含了会话相关数据 ID 的 JWT token，真正的会话相关数据被存储在服务端。
- Session token/cookie：传统的 session ID，例如我们之前已经用过了很多年的 Session Cookie。会话相关的数据存储在服务端。

注意：这篇文章并不是用来说服你绝不要去使用 JWT，而是为了说明 JWT 并不适合用来实现 Session 机制，并且这样做会带来一定的风险。在文章的末尾，我也会简要的介绍一些适合 JWT 使用的场景。

## 提前注意

很多人总是错误的想去比较 cookies 跟 JWT。这种比较是毫无意义的——cookies 是一种存储机制，而 JWT 是一种加密签名的令牌机制。

它们之间并不是相互对立的，相反的，它们既可以独立使用还可以配合使用。真正应该比较的是 session 跟 JWT 以及 cookies 跟 Local Storage。

在这篇文章中，我将主要对 sessions 与 JWT tokens 进行比较，必要时也会穿插一些 cookies 与 Local Storage 之间的对比。

## JWT 经常被人宣传的优点

人们在安利 JWT 的时候经常列出下面这些好处：

- 容易水平扩展
- 简单易用
- 更加灵活
- 更加安全
- 内建的过期功能
- 不需要向用户请求“cookie 许可”
- 预防 CSRF
- 在移动端表现的更好
- 就算用户禁用了 cookies 也能照常工作

接下来我将逐个解释说明为什么这些所谓的优点都是错误的或者具有误导性的。我的一些解释可能会显得有些含糊，但这不能怪我，因为这些所谓的优点本身就是含糊不清的。

### 容易水平扩展

这是之前列出来的 JWT 优势列表中唯一在技术层面上说的比较正确的，不过前提是你在使用 无状态 JWT tokens。然而现实情况是大多数人实际并不要这种可扩展性——我们已经有了很多种简单的方式来进行水平扩张，除非你的用户量跟贴吧不相上下，否则你是不需要无状态会话（斯stateless sessions）的。

下面是一些扩张*有状态*会话的例子：

1. 在单台服务器上运行多个后端进程：可以在同台服务器上运行一个 Redis 来进行会话数据存储
2. 在多台服务器上运行后端进程：可以用一个独立的服务器来运行 Redis 来进行会话存储
3. 在多个集群中的多台服务器上运行后端进程：Sticky Sessions。

这些就是现有软件技术能够良好支持的场景。通常，你的大多数应用不会到达第三个场景中提到的规模。

或许你认为你应该为你的网站提前做好准备，来应对将来可能会发生的水平扩张。然而，在实际的开发中，替换 session 机制是一项微不足道的工作，而你所需要付出的代价往往只是让每个用户强制登出系统*一次*。考虑到后面我将要列出的一系列缺点，提前去实现 JWT 其实并不划算。

### 简单易用

实际上 JWT 并没有那么简单易用。你需要同时手动管理客户端与服务端的会话状态，然而传统的 Session cookies 不用我们做额外的工作就能够正常工作，开箱即用。JWT 从任何方面来看都不够简单。

### 更加灵活

我还没有见过有人解释 JWT 是怎么更加灵活的。基本上所有的主流的 seesion 实现都允许你在会话中存储任意的数据，而这跟 JWT 的工作原理没有任何不同。据我所知，这只是人云亦云的说法。如果你不同意的话，可以联系我，发一些例子给我瞧瞧。

### 更加安全

很多人觉得 JWT tokens 更加安全，因为它使用了密码学算法进行签名校验。不过有签名（signed）的 cookies 也比无签名（unsigned）的 cookies 要安全得多，所以这项特性并不是 JWT 独有的，优秀的 seesion 实现都是会使用有签名 cookies 的。

“使用了密码学算法进行签名校验”并不会神奇地让一个东西变得更加安全，除非是针对某一特殊目的，且是针对这一特殊目的的一个有效的解决方案。错误地使用加密算法有时甚至会让你的应用变得不够安全。

另一种对“更加安全”的解释是“JWT 不通过 cookie 发送”，这简直是滑天下之大稽——cookie 只是一个普普通通的 HTTP 头罢了，并不会导致什么安全风险。实际上，cookies 会被*特别地*对待，来对抗恶意的客户端代码，相关的内容稍后我会提到。

如果你真的担心有人会拦截你的 session cookie，你应该使用 HTTPS——如果不使用 HTTPS 的话，任何类型的 session 实现，包括 JWT， 都是可拦截的。

### 内建的过期功能

这同样不是一个 JWT 独有的功能。过期限制在大多数的框架的 session 实现中都可以做到，甚至还做的更好——你可以做到在不需要会话数据的时候将其完全清除，这是在使用有状态 JWT 是无法实现的。

### 不需要向用户请求“cookie 许可”

这同样也是滑天下之大稽的说法。从来就没有什么限制 cookie 使用的法律——有关 cookie 的各种法律实际上涵盖了所有的非必需的持久性用户标识符，你可以想到的任何会话管理机制都被包括其中。

简单来说：

- 如果你出于实现必须的功能性目的来使用 cookies 或者 tokens 的话（例如，保持用户的登录状态），那么你是不需要征得用户同意的。
- 如果你处于其他非必须的功能性目的来使用 cookies 或者 tokens 的话（例如，分析用户行为），那么你就需要获得用户的许可。

### 预防 CSRF

讲真，JWT 并没有防范 CSRF 攻击的能力。通常，我们有两种方式来存储 JWT：

- 存储在 Cookie 中：现在你仍然对 CSRF 攻击没有抵抗力，并仍然需要依赖其他的保护措施。
- 存储在其他地方，例如，Local Storage：现在你能够抵抗 CSRF 攻击了，不过你的网站也需要使用 JS 代码才能正常工作了，同时也让你的网站更容易遭到另一种完全不同的甚至更加糟糕的威胁了。更多相关的内容我会在后面提到。

应对 CSRF 攻击的正确姿势是使用 CSRF token，而会话管理机制跟其毫无关系。

### 在移动端表现的更好

这也是毫无根据的，任何移动端的浏览器、每一个的主流的移动端开发框架以及严谨的 HTTP 库都是支持 cookies 的，所以也能够支持 session。所以 session 对移动端的支持不存在任何问题。

### 就算用户禁用了 cookies 也能照常工作

未必如此。如果一个用户禁用了 cookies，那么他通常也会禁用掉其他的持久化手段，包括 Local Storage 以及其他各种能够直接用来在客户端存储会话数据的手段。不管用不用 JWT，这已经变成了另一个问题——而且，尝试在禁用客户端存储的情况下让身份认证照常工作有点偏离我们的开发目标。

最重要的是，那些禁用了所有 cookies 的用户通常能够理解这样做会让身份认证功能失效，如果他们在意网站中的这个功能的话，是会单独的取消对该网站 cookies 的禁用的。所以，这并不是一个网站开发人员需要去解决的一个问题，你更值得去做的是向用户解释为什么你的网站需要使用 cookies 才能正常工作。

## JWT 的缺点

至此，我已经逐个解释了为什么 JWT 的那些所谓的优点错误的，如果你认为“这没什么大不了的，就算 JWT 不能帮助我解决更多的问题，也不影响我使用它来进行会话管理”，那你就错了。使用 JWT 来进行会话管理存在不少缺点，其中一些甚至会造成严重安全问题。

### 占用更多存储空间

JWT token 通常不会很小，尤其是那些存储了所有会话数据的无状态 JWT token。你会很快就达到 Cookie 或者 URL 的容量上限，所以你决定使用 Local Storage 来存储 JWT token，不过······

### 更不安全

当你将 JWT 存储在 Cookie 中的时候，它跟其他的 Session ID 的存储方式没有什么区别。但是当你把它存储在其他地方的时候，你将面对另一种恶意攻击的威胁。具体的内容在[这篇文章](http://blog.prevoty.com/does-jwt-put-your-web-app-at-risk)中的“Storing sessions”一节中说明了。

> We pick up where we left off: back at local storage, an awesome HTML5 addition that adds a key/value store to browsers and cookies. So should we store JWTs in local storage? It might make sense given the size that these tokens can reach. Cookies typically top out somewhere around 4k of storage. For a large-sized token, a cookie might be out of the question and local storage would be the obvious solution. However, local storage doesn’t provide any of the same security mechanisms that cookies do.
>
> Local storage, unlike cookies, doesn’t send the contents of your data store with every single request. The only way to retrieve data out of local storage is by using JavaScript, which means any attacker supplied JavaScript that passes the Content Security Policy can access and exfiltrate it. Not only that, but JavaScript also doesn’t care or track whether or not the data is sent over HTTPS. As far as JavaScript is concerned, it’s just data and the browser will operate on it like it would any other data.
>
> After all the trouble those engineers went through to make sure nobody is going to make off with our cookie jar, here we are trying to ignore all the fancy tricks they’ve given us. That seems a little backwards to me.

> 我们回到了曾经离开的地方：Local Storage，一个 HTML5 中极好的新功能，为了浏览器提供了新的键值存储能力。所以我们应该将 JWT token 存储在这里面吗？考虑到 token 可能会达到的大小，这么做是有意义的。Cookies 中往往只能存储 4k 左右的内容，对于一些大体积的 token 来说，使用 Local Storage 来存储是一个显而易见的解决方案。然而 Local Storage 并没有提供类似 cookie 拥有的安全机制。
>
> 与 Cookies 不同，Local Storage 中的内容并不会随着请求发送。取出 Local Storage 中数据的方法只有使用 JS，这意味着任何绕过了 CSP（Content Security Policy）的恶意脚本都可以获取其中的内容。除此之外，JS 也不关心这些数据是否是通过 HTTPS 传输的，对于 JS 来说，这些仅仅只是普通的数据而已，而且浏览器也会像对待其他数据一样对待存储在 Local Storage 中的 JWT tokens。
>
> 工程师们早就已经通过 Cookies 来解决了这些麻烦，但是在这里我们却又要放弃这些已有的成果，在我看来像是在开倒车。

简单来说，除了使用 cookies 来存储敏感数据，我们别无他选，不管你是否使用 JWT。

### 你没法让单独的一个 JWT token 失效

使用 Session 的时候，我们可以随时在服务端让某个用户的会话失效，而对于无状态 JWT 来说，这是无法做到的。JWT 从一开始就被设计为只要不过期就始终保持有效性，这意味着你无法在一些必要的情况下及时的去让单个无状态 JWT 失效。例如，当你检测到用户账号被入侵时，或者用户修改了密码后，你都没办法让他们的会话状态失效。

除非你从头造一个复杂（且有状态）的模块来检测并拒绝需要强行失效的 token，否则你在应对这种情况的时候基本上是无能为力的。

### 数据会变得陈旧

就像缓存一样，存储在无状态 JWT 中的最终会慢慢的变得陈旧，而且无法及时同步数据库中的最新数据。

这意味着当一个用户更新了自己的个人信息之后，之前签发的 JWT token 中的信息将会变得过时，而另一种更加严重的情况是，即便你撤销了某个用户的管理员权限，只要在你进行撤销操作前的 token 没有过期，你就不能真正的完全禁止他使用管理员权限访问系统，除非你把整个网站关掉。

### JWT 的实现缺乏实战检验

上面我们提到的缺点都是无状态 JWT 才会有的，那么你可能就会想，用有状态 JWT 不就好了。然而，有状态的 JWT 跟普通的 session cookies 并没有什么太大的区别，而且这些有状态 JWT 的实现往往没有经历过足够的实战检验。

现有 Session 实现（如，[express-session](https://github.com/expressjs/session)）已经在生产环境中稳定的工作了许多年，在这期间它的安全性也提升了很多。而你自己实现的基于 JWT 的 Session 并不能给你带来这些安全方面的保证，而且第三方的实现也少有真正在生产环境中使用的。

## 总结

无状态 JWT 无法被强制失效或者被及时更新，并且会带来体积过大或者一些由于存储不当而造成的安全隐患。有状态的 JWT 跟 session cookies 并没有太大差别并且缺乏真实生产环境的检验与现有客户端程序的支持。

除非你正在做一个超大规模的应用程序，那么你就不应该使用 JWT 来管理用户会话——只要使用 Session 就好了。

## JWT 的用武之地

在这片文章的开头我就说了，JWT 能够发挥作用的地方不在于实现 session 机制。把 JWT 作为一次性授权 token 使用，才是正确的使用姿势。

根据 [Json Web Token specification](https://tools.ietf.org/html/rfc7519)：

> JSON Web Token (JWT) is a compact, URL-safe means of representing claims to be transferred between two parties. [...] enabling the claims to be digitally signed or integrity protected with a Message Authentication Code (MAC) and/or encrypted.
>
> JWT 是一种简洁的，URL 安全的，用来表示在双方之间传递 claims 的方式。它让 claims 能够通过消息验证码（MAC）进行数字签名或者加密解密。

这里的 “claim”可以理解为类似“命令”或者需要一次性授权的东西，例如：

> 你好，服务器 B，服务器 A 跟我说我可以 ，然后这里是（加密后的）凭证。

举个例子，假设你在运营一个文件存储服务，用户需要登陆后才能下载他们的文件，但是文件却需要通过另一个“无状态的下载服务”获取。在这种情况下，你就需要让你的应用服务器签发一个“下载 Token”，这样客户端就可以通过这个 token 去访问“下载服务”来下载文件。

以这种方式使用 JWT 的时候，需要注意以下几点：

- **Token 的有效期要足够短**。它们只要在几分钟内有效，能够让客户端发起下载文件的请求就够了。
- **Token 只能被使用一次**。应用服务器应该在每次下载文件的时候签发一个新的 token，每个 token 只能够用来请求文件一次，之后就会被抛弃，所以也就不需要去存储 token 的状态。
- **应用服务器仍然需要使用 sessions**。只有“下载服务”需要使用 token 来验证每一次的下载请求，因为它并不会存储任何状态。

如你所见，将 JWT 与 Session 结合使用是有意义的——它们之间分工明确，而且有的时候你还需要它们配合工作。最后，不要使用 JWT 来存储被持久化的或者长期存在的数据。