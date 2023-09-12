---
title: oauth2
date: 2023-09-12 07:53:30
tags:
---
## 一、oauth2.0流程 & github oauth2 App registration
授权码(code)模式是OAuth2目前最安全最复杂的授权流程, 详细流程可以看下图:

![基本流程](./flow.webp)

## 二、github oauth

1. 账号下注册github App

![注册](./github_new_app.png)

2. 注册时填写callback url, 并记录好生成的 clientId & secret

![registered](./github_registered.png)

![callback](./github_callback.png)

3. 在需要oauth2登录github时redirect至该App

![oauth2](./github_oauth.png)

4. 若用户授权,github会带code callback至配置的后端回调地址
   ```
   https://xietiandi.tech/oauth2/authorize/callback?code=b00925eb6e607dca3e77
   ```

5. 后端此时可以根据此code向github换取access_token, postman为例

![ac](./github_ac.png)

6. 后端可以再用access_token请求用户的github相关资源, postman为例

![api](./github_api.png)

