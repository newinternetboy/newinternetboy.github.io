---
title:  "casbin(acl)"
date:   2024-01-25 11:27:26 +0800
categories: ACL
---

# 定义
Casbin是一个功能强大、高效、开源的权限控制实现方案

# 关键组件
## 权限策略持久化(Adapters)
在casbin,权限策略的存储是以中间件的方式提供。casbin可以使用连接器中间件通过LoadPolicy()、SavePolicy()来加载、保存权限策略。


# 参考资料
1. [casbin golang](https://github.com/casbin/casbin?tab=readme-ov-file#tutorials)
2. [RBAC API](https://casbin.org/docs/rbac-api/)
3. https://casbin.org/docs/adapters/