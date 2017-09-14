<h1>总结跨域</h1>
<h2>（1）要解决跨域问题首先：要知道什么是跨域,产生跨域的原因,才能更好的解决跨域</h2>
<pre>
   跨域是指js在不同的域之间进行数据传输或通信，比如用ajax向一个不同的域请求数
 据，或者通过js获取页面中不同域的框架中(iframe)的数据。只要协议、域名、端口有
 任何一个不同，都被当作是不同的域,它是由游览器的同源策略造成的,是游览器对
 JavaScript施加的安全限制。
   没有跨域限制标签包括:img link script
</pre>

![icon_01.png](https://github.com/gaokaomim/kuayu/blob/master/image/icon_01.png)
<a href="https://segmentfault.com/a/1190000000718840#articleHeader1">参考地址</a>
<h2>（2）如何解决</h2>
<h3>1.JSONP</h3>

***
>  * JSONP(JSON with Padding)是JSON的一种“使用模式”，可用于解决主流浏览器的跨域数据访问的问题
>  * 原理:浏览器对XHR做了同源策略，可以利用 <script> 元素的这个开放策略来做到跨域请求的作用，网页可以得到从其他来源动态产生的 JSON 资料，而这种使用模式就是所谓的 JSONP(有可能是历史遗迹（漏洞）)
>  * 兼容性：所有浏览器都兼容这种方式;
>  * 优点：很明显前端可以很轻松的做到跨域请求;
>  * 缺点:JSONP只支持GET请求，不支持POST请求,它只支持跨域HTTP请求这种情况，不能解决不同域的两个页面之间如何进行JavaScript调用的问题
>  * 前端代码如下

<pre>
     /** 前端生成script标签，并将src中传入需要执行的callback **/
     <script>
        var script=document.createElement("script");
        script.type="text/javascript"
        script.src = "http://example.com?jsonp=cb";
        document.body.appendChild(script);
     </script>
</pre>
 
>  * 如果是使用jquery有封装好的$.getJSON方法可以使用,jquery会自动生成一个全局函数来替换callback=?中的问号，之后获取到数据后又会自动销毁，实际上就是起一个临时代理函数的作用。$.getJSON方法会自动判断是否跨域，不跨域的话，就调用普通的ajax方法；跨域的话，则会以异步加载js文件的形式来调用jsonp的回调函数。

<pre>
     /**  jquery源码  **/
   getJSON: function( url, data, callback ) {
		return jQuery.get( url, data, callback, "json" );
	},
</pre>

<pre>
     /** 使用方法如下 **/
   <script type="text/javascript">
    $.getJSON('http://example.com/data.php?callback=?,function(jsondata)'){
        //处理获得的json数据
    });
   </script>
</pre>
<pre>
 <script type="text/javascript">
     jQuery(document).ready(function(){
        $.ajax({
             type: "get",
             async: false,
             url: "http://flightQuery.com/jsonp/flightResult.aspx?code=CA1998",
             dataType: "jsonp",
             jsonp: "callback",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(一般默认为:callback)
             jsonpCallback:"flightHandler",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名，也可以写"?"，jQuery会自动为你处理数据
             success: function(json){
                 alert('您查询到航班信息：票价： ' + json.price + ' 元，余票： ' + json.tickets + ' 张。');
             },
             error: function(){
                 alert('fail');
             }
         });
     });
     </script>
</pre>
<h3>2.CORS</h3>

***
>  * CORS是指跨域资源共享是使用自定义的HTTP头部让游览器与服务器之间进行相互了解对方,从而决定请求或响应是应该成功还是失败,以降低跨域 HTTP 请求所带来的风险

  <a href="https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS#浏览器兼容性">使用场景</a>
***
*  XMLHttpRequest 或 Fetch 发起的跨域 HTTP 请求
*  Web 字体 (CSS 中通过 @font-face 使用跨域字体资源)
*  WebGL 贴图
*  使用 drawImage 将 Images/video 画面绘制到 canvas
*  vue-infinite-scroll
*  样式表（使用 CSSOM）
*  Scripts (未处理的异常)

  授权请求的方法,
> * GET
> * HEAD
> * POST
> * DELETE 
> * OPTIONS
  使用方法如下
> * 比如说，假如站点 http://foo.example 的网页应用想要访问 http://bar.other 的资源。http://foo.example 的网页中可能包含类似于下面的 JavaScript 代码   
<pre>
        /** 使用方法如下 **/
    var invocation =new XMLHttpRequest();
    var url = 'http://bar.other/resources/public-data/';
    function callOtherDomain(){
        if(invocation){
            invocation.open('GET',url,true);
            invocation.onreadystatechange = handler;
            invocation.send(); 
        }
    }
</pre>
<pre>
  服务器端对于CORS的支持，主要就是通过设置Access-Control-Allow-Origin来进行的。如果浏览器检测到相应的设置，就可以允许Ajax进行跨域的访问。（<a href="http://www.cnblogs.com/sloong/p/cors.html">参考</a>）
  <pre>
    /**      新增CORSFilter 类      **/
    @Component
    public class CORSFilter extends OncePerRequestFilter {
        @Override
        protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
            response.addHeader("Access-Control-Allow-Origin", "*");
            response.addHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE");
            response.addHeader("Access-Control-Allow-Headers", "Content-Type");
            response.addHeader("Access-Control-Max-Age", "1800");//30 min
            filterChain.doFilter(request, response);
        }
    }
</pre>
</pre>

<h3>3.通过修改document.domain来跨子域(这个方法有个问题)</h3>

***
>  * 跨域分成两种，主要是基于游览器的同源策略限制,这个限制有两种情况,第一种不能通过ajax的方法去请求不同源中的文档,第二种游览器不同域的框架之间不能进行js交互操作,而document.domain是解决第二种情况,
>  * 注意:在不同框架之间,是能够获取到彼此的window对象，但是不能获取到window对象里的属性和方法

<h3>4.使用window.name来进行跨域</h3>

***
>  * window对象有个name属性，该属性有个特征：即在一个窗口(window)的生命周期内,窗口载入的所有的页面都是共享一个window.name的，每个页面对window.name都有读写的权限，window.name是持久存在一个窗口载入过的所有页面中的，并不会因新页面的载入而进行重置。
> * 兼容性：所有浏览器都支持;
> * 优点:1.最简单的利用了浏览器的特性来做到不同域之间的数据传递;2.不需要前端和后端的特殊配制;
> * 缺点:1.window.name的size有大小限制(大概在2M左右)2.安全性：当前页面所有window都可以修改，很不安全
<pre>
    <!--代码实现  -->
    /**   a.html   **/
    <script type="text/javascript">
    window.name="我是来自a页面";
        setTimeout(function(){
            window.location='b.html';
        },1000)
    </script>
</pre>
<pre>
    /**   b.html   **/
    <script type="text/javascript">
       alert(window.name)
    </script>
</pre>

 > * 3.数据类型：传递数据只能限于字符串，如果是对象或者其他会自动被转化为字符串

![icon_02.png](https://github.com/gaokaomim/kuayu/blob/master/image/icon_02.png)

<h3>5.window.postMessage</h3>
<pre>
    window.postMessage方法是h5新引进的特性,可以使用它来向其它的window对象发送消息,不兼容低版本游览器,调用postMessage方法的window对象是指要接收消息的那一个window对象，该方法的第一个参数message为要
发送的消息，类型只能为字符串；第二个参数targetOrigin用来限定接收消息的那个window对象所在的域，如果不想限定域，可以使用通配符 *  。需要接收消息的window对象，可是通过监听自身的message事件来获取传
过来的消息，消息内容储存在该事件对象的data属性中。上面所说的向其他window对象发送消息，其实就是指一个页面有几个框架的那种情况，因为每一个框架都有一个window对象。在讨论第二种方法的时候，我们说过，不同域
的框架间是可以获取到对方的window对象的，而且也可以使用window.postMessage这个方法。下面看一个简单的示例，有两个页面
</pre>
<pre>
/**      代码如下  这个a页面上的代码    **/
<script>
       function onLoad() {
            var iframe = document.getElementById('iframe');
            var win = iframe.contentWindow;
            win.postMessage("哈哈,我是来自页面a的消息", '*');
        }
</script>
 <iframe id="iframe" src="https://gaokaomim.github.io/kuayu/b.html" onload="onLoad()">
</pre>
<pre>
/**      代码如下  这个b页面上的代码    **/
<script>
        window.onmessage = function (e) {
            e = e || event;
            alert(e.data);
        }
</script> 
</pre>
<h3>6.服务端</h3>
<pre>
  服务器端设置http header
   @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
        ModelAndView modelAndView) throws Exception {
        response.setHeader("Access-Control-Allow-Origin", "*");  //跨域请求处理
        response.setHeader("Access-Control-Allow-Methods", "POST");
        response.setHeader("Access-Control-Max-Age", "1000");
        response.setContentType("text/html; charset=GBK");
        //System.out.println(">>>MyInterceptor1>>>>>>>请求处理之后进行调用，但是在视图被渲染之前（Controller方法调用之后）");
    }]
</pre>
<h3>7.随便说下json和xml的区别</h3>
<pre>
   (1)XML定义扩展标记语言 (Extensible Markup Language, XML) ，用于标记电子文件使其具有结构性的标记语言，可以用来标记数据、定义数据类型，是一种允许用户对自己的标记语言进行定义的源语言
   (2)JSON定义JSON(JavaScript Object Notation)一种轻量级的数据交换格式，具有良好的可读和便于快速编写的特性
  1.XML的优缺点
   1.1XML的优点　　
    1.1.1格式统一，符合标准；　　
    1.1.2容易与其他系统进行远程交互，数据共享比较方便。
   1.2XML的缺点　　
    1.2.1XML文件庞大，文件格式复杂，传输占带宽；　　
    1.2.2服务器端和客户端都需要花费大量代码来解析XML，导致服务器端和客户端代码变得异常复杂且不易维护；　　
    1.2.3客户端不同浏览器之间解析XML的方式不一致，需要重复编写很多代码；　　
    1.2.4服务器端和客户端解析XML花费较多的资源和时间。
    2.JSON的优缺点
   2.1JSON的优点：　　
    2.1.1数据格式比较简单，易于读写，格式都是压缩的，占用带宽小；　　
    2.1.2易于解析，客户端JavaScript可以简单的通过eval()进行JSON数据的读取；　　
    2.1.3支持多种语言，包括ActionScript, C, C#, ColdFusion, Java, JavaScript, Perl, PHP, Python, Ruby等服务器端语言，便于服务器端的解析；　　
    2.1.4在PHP世界，已经有PHP-JSON和JSON-PHP出现了，偏于PHP序列化后的程序直接调用，PHP服务器端的对象、数组等能直接生成JSON格式，便于客户端的访问提取；　　
    2.1.5因为JSON格式能直接为服务器端代码使用，大大简化了服务器端和客户端的代码开发量，且完成任务不变，并且易于维护。
   2.2JSON的缺点　　
    2.2.1没有XML格式这么推广的深入人心和喜用广泛，没有XML那么通用性；　　
    2.2.2JSON格式目前在Web Service中推广还属于初级阶段。
  3.XML和JSON的优缺点对比
   3.2可读性方面。JSON和XML的数据可读性基本相同，JSON和XML的可读性可谓不相上下，一边是建议的语法，一边是规范的标签形式，XML可读性较好些。
   3.3可扩展性方面。XML天生有很好的扩展性，JSON当然也有，没有什么是XML能扩展，JSON不能的。
   3.4编码难度方面。XML有丰富的编码工具，比如Dom4j、JDom等，JSON也有json.org提供的工具，但是JSON的编码明显比XML容易许多，即使不借助工具也能写出JSON的代码，可是要写好XML就不太容易了。
   3.5解码难度方面。XML的解析得考虑子节点父节点，让人头昏眼花，而JSON的解析难度几乎为0。这一点XML输的真是没话说。
   3.6流行度方面。XML已经被业界广泛的使用，而JSON才刚刚开始，但是在Ajax这个特定的领域，未来的发展一定是XML让位于JSON。到时Ajax应该变成Ajaj(Asynchronous Javascript and JSON)了。
   3.7解析手段方面。JSON和XML同样拥有丰富的解析手段。
   3.8数据体积方面。JSON相对于XML来讲，数据的体积小，传递的速度更快些。
   3.9数据交互方面。JSON与JavaScript的交互更加方便，更容易解析处理，更好的数据交互。
   3.10数据描述方面。JSON对数据的描述性比XML较差。
   3.11传输速度方面。JSON的速度要远远快于XML。
</pre>
