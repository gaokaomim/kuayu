<h1>总结跨域</h1>
<h2>（1）要解决跨域问题首先：要知道什么是跨域,产生跨域的原因,才能更好的解决跨域</h2>
<pre>
   跨域是指js在不同的域之间进行数据传输或通信，比如用ajax向一个不同的域请求数
 据，或者通过js获取页面中不同域的框架中(iframe)的数据。只要协议、域名、端口有
 任何一个不同，都被当作是不同的域,它是由游览器的同源策略造成的,是游览器对
 JavaScript施加的安全限制。
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
<h3>2.CORS</h3>

***
>  * CORS是指跨域资源共享是使用自定义的HTTP头部让游览器与服务器之间进行相互了解对方,从而决定请求或响应是应该成功还是失败,以降低跨域 HTTP 请求所带来的风险

### <a href="https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS#浏览器兼容性">使用场景</a>
***
*  XMLHttpRequest 或 Fetch 发起的跨域 HTTP 请求
*  Web 字体 (CSS 中通过 @font-face 使用跨域字体资源)
*  WebGL 贴图
*  使用 drawImage 将 Images/video 画面绘制到 canvas
*  vue-infinite-scroll
*  样式表（使用 CSSOM）
*  Scripts (未处理的异常)

### 授权请求的方法,
> * GET
> * HEAD
> * POST
> * DELETE 
> * OPTIONS
### 使用方法如下
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
    /**   新增CORSFilter 类      **/
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

     


