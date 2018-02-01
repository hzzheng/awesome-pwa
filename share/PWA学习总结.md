title: PWA学习总结
speaker: 郑超
transition: slide

[slide]

# PWA学习总结

[slide]

# 声明
---
1. 这次分享的目的是勾起大家对PWA的兴趣 {:&.bounceIn}
2. 不会讲太多细节，PWA里面很多技术点要讲很清楚可能都需要两三个小时
3. 重阳后面会再分享细节😂

[slide]

# 先看[DEMO](https://arcane-garden-92807.herokuapp.com/)
---
1. ServiceWorker
2. CacheStorage

[slide]
<video src="/video/2.mp4" controls >

[slide]
<img src="/imgs/16.jpeg" />

[slide]
<img src="/imgs/17.jpeg" />

[slide]

# PWA是什么

[slide]
[magic data-transition="zoomin"]
# Progressive Web Apps
====
# Progressive
====
# Progressive Enhancement
====
# Progressive Enhancement
---
* 举个例子，有一个传统的Mobile Web，你想优化它的用户体验，比方说在2G/3G弱网络环境下能够快速访问，就可以考虑使用[Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache)来做缓存提高重复访问的速度。但在不支持Cache API的终端下，网站可以走网络请求依然能够正常运行。
====
# Progressive Web Apps
1. 抽象地说：全新的Web应用概念，让Web更可靠、更快速、有类似Native一样的用户体验 {:&.bounceIn}
2. 具体地说：以ServiceWorker为技术核心的一系列新的Web API和标准协议([web push protocol](https://tools.ietf.org/html/draft-ietf-webpush-protocol-12))，通过这些新功能，可以让Web应用做到离线访问、消息推送、后台同步、添加到手机主屏等功能

[/magic]

[slide]

# PWA为什么重要
---
有人说PWA将会和AJAX一样是Web开发领域的里程碑技术 {:&.bounceIn}

[slide]
<img src="/imgs/3.jpeg">
---
<img src="/imgs/4.jpeg">

[slide]
![](https://jakearchibald.com/static/imgs/me.439a7c69520f.jpg)
---
[Jake Archibald](https://jakearchibald.com/)
---
* [Instant Loading: Building offline-first Progressive Web Apps - Google I/O 2016](https://www.youtube.com/watch?v=cmGr0RszHc8&list=WL&index=7&t=3s)

[slide]
<img src="/imgs/1.jpg">
[天猫超市Mobile Web的极致体验优化](https://tianchi.aliyun.com/forum/videoStream.html#postsId=3617)

[slide]
<img src="/imgs/5.jpg">

[slide]
<img src="/imgs/2.jpg">

[slide]

# 怎么写一个PWA

[slide]
[Learn how to build a PWA in 5 minutes](https://medium.com/dev-channel/learn-how-to-build-a-pwa-in-under-5-minutes-c860ad406ed)

[slide]
<video src="/video/1.mp4" controls>

[slide]
# P0事故来了
---
[PWA 在饿了么的实践经验](https://zhuanlan.zhihu.com/p/25800461)

[slide]
<img src="/imgs/6.jpg" />

[slide]
---
## Application Manifest
---
## ServiceWorker & Cache
---
## Web Push & Notification
---
## Background Sync
---

[slide]
---
## Application Manifest
---
1. 它是一个JSON文件，一般命名为Manifest.json，包含了一些配置信息 {:&.bounceIn}
2. 触发浏览器弹一个对话框问你是否添加应用到手机主屏
3. 对添加到手机主屏的Web App做一些外观设置

[slide]

```json
{
  "short_name": "AirHorner",
  "name": "Kinlan's AirHorner of Infamy",
  "icons": [
    {
      "src": "launcher-icon-1x.png",
      "type": "image/png",
      "sizes": "48x48"
    },
    {
      "src": "launcher-icon-2x.png",
      "type": "image/png",
      "sizes": "96x96"
    },
    {
      "src": "launcher-icon-4x.png",
      "type": "image/png",
      "sizes": "192x192"
    }
  ],
  "start_url": "index.html?launcher=true"
}
```
[The Web App Manifest](https://developers.google.com/web/fundamentals/web-app-manifest/)

[slide]
<img src="/imgs/7.jpeg" />

[slide]
<img src="/imgs/8.jpeg" />

[slide]
<img src="https://developers.google.com/web/fundamentals/app-install-banners/images/add-to-home-screen.gif" />

[slide]
# 关于添加到主屏
<img src="/imgs/9.jpeg" />

[slide]
---
## ServiceWorker & Cache
---
1. 它是一种特殊的Web Worker，在这个Worker里可以监听一些特殊的事件，比如"fetch", "push", "sync"等 {:&.bounceIn}
2. 它在后台运行，不会影响页面主线程，并且即便页面关闭也会存在
3. 技术上大量使用Promise

[slide]
# 可以监听哪些事件
<img src="/imgs/10.jpg" />

[slide]
# ServiceWorker生命周期
<img src="/imgs/11.jpeg" />

[slide]
```javascript
if ('serviceWorker' in navigator) {
  window.addEventListener('load', function() {
    navigator.serviceWorker.register('/sw.js').then(function(registration) {
      // Registration was successful
      console.log('ServiceWorker registration successful with scope: ', registration.scope);
    }, function(err) {
      // registration failed :(
      console.log('ServiceWorker registration failed: ', err);
    });
  });
}
```
[slide]
```javascript
var CACHE_NAME = 'my-site-cache-v1';
var urlsToCache = [
  '/',
  '/styles/main.css',
  '/script/main.js'
];

self.addEventListener('install', function(event) {
  // Perform install steps
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(function(cache) {
        console.log('Opened cache');
        return cache.addAll(urlsToCache);
      })
  );
});
```

[slide]
```javascript
self.addEventListener('activate', function(event) {

  var cacheWhitelist = ['pages-cache-v1', 'blog-posts-cache-v1'];

  event.waitUntil(
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        cacheNames.map(function(cacheName) {
          if (cacheWhitelist.indexOf(cacheName) === -1) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```

[slide]
```javascript
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
      .then(function(response) {
        // Cache hit - return response
        if (response) {
          return response;
        }
        return fetch(event.request);
      }
    )
  );
});
```

[slide]
```javascript
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
      .then(function(response) {
        // Cache hit - return response
        if (response) {
          return response;
        }
        // IMPORTANT: Clone the request. A request is a stream and
        // can only be consumed once. Since we are consuming this
        // once by cache and once by the browser for fetch, we need
        // to clone the response.
        var fetchRequest = event.request.clone();

        return fetch(fetchRequest).then(
          function(response) {
            var responseToCache = response.clone();

            caches.open(CACHE_NAME)
              .then(function(cache) {
                cache.put(event.request, responseToCache);
              });

            return response;
          }
        );
      })
    );
});
```
[slide]
# 缓存策略
---
1. Cache with network fallback {:&.bounceIn}
2. Cache only
3. Network only
4. Network with cache fallback
5. Cache, then network

[slide]
---
## Web Push & Notification
---
[slide]
<img src="/imgs/12.png">
[slide]
1. Push API {:&.bounceIn}
2. Notification API

[slide]
# Step 1
<img src="/imgs/13.jpeg" />

[slide]
# Step 2
<img src="/imgs/14.jpeg" />

[slide]
# Step 3
<img src="/imgs/15.jpeg" />


[slide]
---
## Background Sync
---
* 暂时只有Chrome 49+支持 {:&.bounceIn}

[slide]
[DEMO](https://www.youtube.com/watch?v=l4e_LFozK2k&feature=youtu.be)

[slide]
```javascript
if ('serviceWorker' in navigator && 'SyncManager' in window) {
  navigator.serviceWorker
    .register('./service-worker.js')
    .then(registration => navigator.serviceWorker.ready)
    .then(registration => { // register sync
      document.getElementById('requestButton').addEventListener('click', () => {
        registration.sync.register('image-fetch').then(() => {
            console.log('Sync registered');
        });
      });
    });
} else {
  document.getElementById('requestButton').addEventListener('click', () => {
    console.log('Fallback to fetch the image as usual');
  });
}
```

[slide]
```javascript
self.addEventListener('sync', function(event) {
  if (event.tag == 'image-fetch') {
    event.waitUntil(fetchDogImage());
  }
});
```
```javascript
function fetchDogImage () {
    fetch('./doge.png')
      .then(function (response) {
        return response;
      })
      .then(function (text) {
        console.log('Request successful', text);
      })
      .catch(function (error) {
        console.log('Request failed', error);
      });
  }
```

[slide]
# 谢谢


