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

###JSONP(JSON with Padding)是JSON的一种“使用模式”，可用于解决主流浏览器的跨域数据访问的问题
###解释
###兼容性：所有浏览器都兼容这种方式；
###优点：很明显前端可以很轻松的做到跨域请求；
###前端代码
<pre>
     /** 前端生成script标签，并将src中传入需要执行的callback **/
     <script>
        var script=document.createElement("script");
        script.type="text/javascript"
        script.src = "http://example.com?jsonp=cb";
        document.body.appendChild(script);
     </script>
</pre>
