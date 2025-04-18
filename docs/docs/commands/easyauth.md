---
outline: deep
lastUpdated: 2025-01-18T17:20:00+08:00
prev: false
next: false
---

# 登录

关于进服后登录账号的详细说明
（如果你对登录无问题可不阅读本文）

### 背景
由于本服昰离线服，无需微软的账户验证，其他玩家通过使用相同用户名本可以轻易冒充一个玩家进入服务器。为避免此类情况的发生，服务器装有 [EasyAuth](https://modrinth.com/mod/easyauth) 模组。我们要求玩家进服后注册当前的账号，并且之后进服需要登录。

玩家的密码存储在数据库中，被加盐加密，即使数据库泄漏也难以被破解，甚至连管理员和服主也无法看到，可以放心输入密码。

### 警告
- 如果不登录则什么都干不了，既不能移动也不能发消息！
- 进服后尽快在3分钟内完成登录或注册，否则会因为超时被踢出服务器。
- 密码长度不小于4个字符，不大于32个字符，不要用汉字或一些奇怪的字符。

### 指令
1. `/register <密码> <再次输入密码>` 注册账号，比如 `/register abcd abcd`
2. `/l <密码>` 登录账号，或者`/login <密码>`，先注册才能再登录
3. `/logout` 登出账号
4. `/account unregister <密码>` 注销账号
5. `/account changePassword <旧密码> <新密码>` 换密码

### 说明
- 每个玩家有5次尝试输入密码的机会，如果5次输入错误则会被踢出服务器，三分钟后才能重新登录，此时三分钟以内登录会显示密码错误。
- 如果你实在忘记了密码，请联系管理员或服主为你重置，注意他们看不到你的密码，只能重置。
- 退服后同一IP在三个小时内进服免登录。
