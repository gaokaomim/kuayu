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
