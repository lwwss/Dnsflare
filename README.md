# 适用CloudFlare Pages

## Workers解决跨域问题

以下内容参考自[七夏浅笑](http://www.julydate.com/post/3278826680/ "七夏浅笑") ，在此表示感谢~

```javascript
async function handleRequest(request) {
    const { origin, pathname, searchParams } = new URL(request.url);
    const param = searchParams.toString() ? "?" + searchParams.toString() : "";
    const requestHeaders = request.headers.get("Access-Control-Request-Headers");
    const requestMethod = request.headers.get("Access-Control-Request-Method");
    const requestOrigin = request.headers.get("Origin") ? request.headers.get("Origin") : origin;
    
    // 响应预检请求
    if (request.method == "OPTIONS") {
        return new Response('{"Access": "OPTIONS"}', {
            status: 200,
            headers: {
                //"Access-Control-Allow-Origin": requestOrigin ? requestOrigin : "*", //不限制请求来源
                "Access-Control-Allow-Origin" : "https://dnsflare-lms.pages.dev",      //限制CROS域名
                "Access-Control-Allow-Credentials": "true",
                "Access-Control-Allow-Headers": requestHeaders ? requestHeaders : "*",
                "Access-Control-Allow-Methods": requestMethod ? requestMethod : "*",
                "Content-Type": "application/json"
            }
        });
    }
    
    // 处理正式请求
    const newRequest = new Request("https://api.cloudflare.com" + pathname + param, request);
    // 请求头删除来源
    newRequest.headers.set("referrer-policy", "no-referrer");
    newRequest.headers.delete("Referer");
    newRequest.headers.delete("Origin");
    // 发起请求
    const response = await fetch(newRequest);
    const newResponse = new Response(response.body, response);
    
    // 处理响应
    newResponse.headers.set("Access-Control-Allow-Origin", requestOrigin ? requestOrigin : "*");
    newResponse.headers.set("Access-Control-Allow-Credentials", "true");
    newResponse.headers.set("Access-Control-Allow-Headers", requestHeaders ? requestHeaders : "*");
    newResponse.headers.set("Access-Control-Allow-Methods", requestMethod ? requestMethod : "*");
    newResponse.headers.set('Access-Control-Expose-Headers', '*');
    newResponse.headers.set("Content-Type", "application/json");
    return newResponse;
}
addEventListener('fetch', event => {
    event.respondWith(handleRequest(event.request));
})
```

## 修改程序API路径

全局API路径位于`/src/utils/requests.ts`第4行`baseURL`，与反代路径相对应进行修改

## Pages部署

选择Vue框架，构建命令`npm run build`，输出目录`build`

------------

# Dnsflare作者介绍

可视化的修改 Cloudflare Zone 的解析地址

## 原因
Cloudflare 非常鸡贼的 ban 掉了 `externallyManaged` 用户访问 Cloudflare 控制台编辑 DNS Record 的功能, 然后有些 Partner 又已经跑了 ~~不是~~, 但是又眼馋 pro

### 优点
所有请求由本地浏览器产生, 服务端仅进行 CORS 处理

Partner 无法直接添加 A 记录 (据说), 而且 Partner API 在开启 2FA 的情况下无法使用

## 使用
到 [Cloudflare 的 API Token 设定](https://dash.cloudflare.com/profile/api-tokens) 中新建一个 Token, 给这个 Token 以下权限

- Zone.DNS 写权限 (用于写入 DNS 记录)
- Zone.Zone 读权限 (用于读取域名列表)

然后访问 [Example](https://dnsflare.indexyz.now.sh) 来登录到面板

## License
Open sourced under the MIT license.
