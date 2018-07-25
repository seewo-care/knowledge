### 需求

Service Worker拦截到一个请求后，假设请求的body是json，内容为{ count:1 }，SW需要修改该请求的内容为{ count: 2 }，然后才发给服务器

### 背景

Service Worker拦截到一个请求后，不能直接修改该请求的内容

### 解决方法：

```javascript
const fetchRequest = req.clone();
fetchRequest.json()
	.then(
		json => {
      
      json.params.dataId = editedDataId;
      //准备一个新请求
      const newReq = new Request(fetchRequest.url, {
        bodyUsed: false,
        credentials: fetchRequest.credentials,
        headers: fetchRequest.headers,
        integrity: fetchRequest.integrity,
        method: fetchRequest.method,
        mode: fetchRequest.mode,
        redirect: fetchRequest.redirect,
        referrer: fetchRequest.referrer,
        referrerPolicy: fetchRequest.referrerPolicy,
        body: JSON.stringify(json)
      });
      
      fetch(newReq)
      	.then(
      		res => {
            //to do
          }
      	)
      	.catch(
      		errResp => {
            //to do
          }
      	)      
    }	
	)
```

先将请求克隆下来，如果请求的body为json，可以将其json化，修改参数，然后按上述的方法创建一个新的请求



### 效果

可以实现Service Worker拦截并发送任意请求，任意参数
