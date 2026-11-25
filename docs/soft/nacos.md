---
title: Nacos
---

[配置鉴权](https://nacos.io/docs/next/manual/admin/auth/?spm=5238cd80.23f13fa2.0.0.30a87e14w7JX3T){target='_blank'}

* 生成密钥
openssl rand -base64 32


nacos.core.auth.enabled=true


* 认证测试

```
curl 'http://localhost:8848/nacos/v1/cs/ops/derby?sql=select+st.tablename+from+sys.systables+st'
```

curl -X GET "http://wjn:123@192.168.3.12:8848/nacos/v1/cs/configs?dataId=common-config.yml&group=DEFAULT_GROUP&tenant=park"

---
