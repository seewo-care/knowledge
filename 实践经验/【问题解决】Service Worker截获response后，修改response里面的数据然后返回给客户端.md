### 需求

来自服务器的response的body为{ count: 1 }，通过service worker拦截处理后将response的body改为{ count: 2 }，然后返回给客户端

### 背景

Service Worker拦截到response后并不支持直接修改响应内的任何内容

### 解决方法

```javascript
function handleDataFetch(req, event){
      event.respondWith(
        fetch(req)
          .then(
            resp => {
            return resp.clone().json();
          }
        )
          .then(
            json => {            
            var res = new Response(JSON.stringify(json));
            return res;
          }
        )
    )
}
```

可以先将获取到的resp进行json化，然后访问json对象，修改里面的body（body可以不是json对象），最后以json对象为基础创建一个新的response，返回新的response

注意点：new Response，参数必须为json字符串
