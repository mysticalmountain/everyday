[TOC]
单点登录（Single Sign-On），简称SSO。实现在多个应用系统中，用户只需登录一次就可以访问多个授信系统。

# 原理
## SSO体系中的角色
- User（多个）
- Web应用（多个） 
- SSO认证中心（一个）

# SSO实现模式
- 所有的认证都在SSO完成
- SSO通过一些方法告诉WEB应用，当前访问的用户是否已通过认证
- SSO认证中心和所有的WEB应用建立一种信任关系，也就是说WEB应用必须信任认证中心

# SSO主要实现方式
- 共享cookies

# 结构






# 问题
cas server session超时
cas server和cas client用户信息同步（可以考虑反向认证方式，cas server调用cas client提供接口做用户认证）
cas server和cas client都落地哪些数据
cas server和cas client都使用https访问
cas client完成在cas server的认证后建立自己的session，所以登录成功后访问本系统资源不会与cas server交互
cas server支持多节点，将session数据保存在DB或缓存服务器
针对boss场景cas client a 如何跳转至cas client b
cas client需要用用一套用户体系
虫子密码怎么搞

1. 登录单点系统
2. 单点系统显示用户可访问的子系统
3. 用户选择需要访问的子系统，进入

用户、角色、权限等数据落地在单点系统

