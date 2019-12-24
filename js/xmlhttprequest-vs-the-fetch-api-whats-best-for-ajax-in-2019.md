# [译]XMLHttpRequest和Fetch, 谁最适合AJAX？

> 原文地址：<https://www.sitepoint.com/xmlhttprequest-vs-the-fetch-api-whats-best-for-ajax-in-2019/>

## 目录
* [从AJAX到Ajax](#h2-1)
* [XMLHttpRequest](#h2-2)
* [Fetch](#h2-3)
  * [浏览器支持](#h3-1)
  * [默认无Cookie](#h3-2)
  * [错误不会被拒绝](#h3-3)
  * [不支持超时](#h3-4)
  * [中止Fetch](#h3-5)
  * [没有Progress](#h3-6)
* [XMLHttpRequest vs Fetch?](#h2-4)

2019年是ajax诞生的20周年。可以说，`XMLHttpRequest`的第一次实现是在1999年作为IE5.0 ActiveX组件发布。

在此之前，曾经有一些方法可以在不刷新页面的情况下从服务器获取数据，但是他们通常依赖笨拙的技术，例如`<script>`注入或第三方插件。微软开发了`XMLHttpRequest`初始版本, 用于替代Outlook基于浏览器的电子邮件客户端。

`XMLHttpRequest`直到2006年才成为Web标准，但在此之前已在大多数浏览器中被实现。由于它在Gmail和Google Maps中的采用，Jesse James Garrett在2005年发表了一篇文章：AJAX: A New Approach to Web Applications.这个新术语吸引了开发人员的关注。

## <span id="h2-1">从AJAX到Ajax</span>
AJAX是Asynchronous JavaScript and XML的简称。"Asynchronous"一词很明显，但是：
1. 虽然VBScript和Flash也可以实现，但是JavaScript更合适
2. 有效负载不必是XML，尽管在当时很流行。在今天，可以使用任何的数据格式，通常JSON是首选。

现在，我们将“Ajax”用作客户端从服务器获取数据并动态更新DOM,而无需刷新整个页面的通用术语。Ajax是大多数Web应用程序和单页应用程序（SPA）的核心技术。

## <span id="h2-2">XMLHttpRequest</span>
以下JavaScript代码展示了如何使用`XMLHttpRequest`(通常简称为XHR)向`http://domain/service`发出的HTTP GET请求。

```javascript
let xhr = new XMLHttpRequest();
xhr.open('GET', 'http://domain/service');

// request state change event
xhr.onreadystatechange = function() {
  // request completed?
  if (xhr.readyState !== 4) return;

  if (xhr.status === 200) {
    // request successful - show response
    console.log(xhr.responseText);
  } else {
    // request error
    console.log('HTTP error', xhr.status, xhr.statusText);
  }
};

// start request
xhr.send();
```

[`XMLHttpRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)对象有许多属性、方法和事件。例如，可以设置和监测以毫秒为单位的超时：
```javascript
// set timeout
xhr.timeout = 3000; // 3 seconds
xhr.ontimeout = () => console.log('timeout', xhr.responseURL);
```

并且progress事件可以报告长时间运行的文件上传：
```javascript
// upload progress
xhr.upload.onprogress = p => {
  console.log( Math.round((p.loaded / p.total) * 100) + '%') ;
}
```

属性的数量可能令人困惑，而且`XMLHttpRequest`的早期实现在跨浏览器之间也不一致。因此，很多库和框架都提供了Ajax的封装函数，例如`jQuery.ajax()`方法：
```javascript
// jQuery Ajax
$.ajax('http://domain/service')
  .done(data => console.log(data))
  .fail((xhr, status) => console.log('error:', status));
```

## <span id="h2-3">Fetch</span>
[Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)是`XMLHttpRequest`的现代替代方案。通用的[Header](https://developer.mozilla.org/en-US/docs/Web/API/Headers)，[Request](https://developer.mozilla.org/en-US/docs/Web/API/Request)和[Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)接口提供了一致性，同时Promises允许更简单的的链式调用和不需要回调的async/await。
```javascript
fetch(
    'http://domain/service',
    { method: 'GET' }
  )
  .then( response => response.json() )
  .then( json => console.log(json) )
  .catch( error => console.error('error:', error) );
```

Fetch简洁，优雅，易于理解，并且在PWA Service Worker中大量使用。为什么不用它替代古老的XMLHttpRequest呢？

不幸的是，Web开发从未如此明确。Fetch还不是用于Ajax的完美替代品...

### <span id="h3-1">浏览器支持</span>
Fetch API在大部分浏览器中得到了很好的[支持](https://caniuse.com/#search=fetch)，但是不支持所有版本的IE。使用2017年之前版本的Chrome，Firefox和Safari的人可能也会遇到问题。这些用户或许只占你用户群的一小部分……也有可能是主要客户。所以编码之前，请务必确认兼容性！

此外，与成熟的XHR对象相比，Fetch API较新，并且会接收更多正在进行的更新。这些更新不太可能破坏原始代码，但预计未来几年会进行一定的维护工作。

### <span id="h3-2">默认无Cookie</span>
与`XMLHttpRequest`不同，Fetch并不会默认发送cookie，因此应用的身份验证可能会失败。可以通过更改第二个参数中传递的[初始值](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch#Parameters)来解决此问题，例如：
```javascript
fetch(
  'http://domain/service',
  {
    method: 'GET',
    credentials: 'same-origin'
  }
)
```

### <span id="h3-3">错误不会被拒绝</span>
令人惊讶的是，HTTP错误（例如`404 Page Not Found` 或 `500 Internal Server Error`）不会导致Fetch返回的Promise标记为reject；`.catch()`也不会被执行。想要精确的判断 fetch是否成功，需要包含 promise resolved 的情况，此时再判断 `response.ok`是不是为 true。如下：
```javascript
fetch(
    'http://domain/service',
    { method: 'GET' }
  )
  .then( response => {
    if(response.ok) {
      return response.json();
    }
    throw new Error('Network response was not ok.');
  })
  .then( json => console.log(json) )
  .catch( error => console.error('error:', error) );
```

仅当请求无法完成时才触发`reject`，例如网络故障或请求被阻止。这会使错误捕获更加复杂。

### <span id="h3-14">不支持超时</span>
Fetch不支持超时，只要浏览器允许，请求将继续。解决方法是可以将Fetch包装在一个Promise中，例如：
```javascript
// fetch with a timeout
function fetchTimeout(url, init, timeout = 3000) {
  return new Promise((resolve, reject) => {
    fetch(url, init)
      .then(resolve)
      .catch(reject);
    setTimeout(reject, timeout);
  }
}
```

或使用`Promise.race()`决定fetch或timeout何时首先完成，例如：
```javascript
Promise.race([
  fetch('http://url', { method: 'GET' }),
  new Promise(resolve => setTimeout(resolve, 3000))
])
  .then(response => console.log(response))
```

### <span id="h3-5">中止Fetch</span>
通过`xhr.abort()`很容易结束一个XHR请求，另外也可以通过`xhr.onabort`函数监测事件解决。

之前一直无法中止一个Fetch请求，但是现在实现了[AbortController API](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)的浏览器可以支持它。这将触发一个信号，该信号可以传递给Fetch启动对象：
```javascript
const controller = new AbortController();

fetch(
  'http://domain/service',
  {
    method: 'GET'
    signal: controller.signal
  })
  .then( response => response.json() )
  .then( json => console.log(json) )
  .catch( error => console.error('Error:', error) );
```

Fetch可以通过调用`controller.abort()`来中止。Promise被标记reject后，会调用`.catch()`函数。

### <span id="h3-6">没有Progress</span>
在撰写本文时，Fetch仍不支持进度事件。因此，不可能显示文件上传或大型表单提交的进度状态。

## <span id="h2-4">XMLHttpRequest vs Fetch?</span>
最后，选择还是看你自己……除非你的应用是要求有上传进度条的IE客户端。你也可以选择将[Fetch polyfill](https://github.github.io/fetch/)与[Promise polyfill](https://www.npmjs.com/package/promise-polyfill)结合使用，以便在IE中执行Fetch代码。

对于更简单的Ajax调用，`XMLHttpRequest`是低级别的，更复杂的，并且你需要封装函数。不幸的是，一旦你开始考虑超时，中止调用和错误捕获的复杂性，Fetch也会如此。

Fetch的未来可期。但是，该API是相对较新，它不提供所有XHR的功能，并且某些参数设置比较繁琐。因此在以后的使用过程中，请注意上述问题。