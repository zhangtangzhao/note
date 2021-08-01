---
sort: 3
---

# Dubbo

## dubbo服务暴露

1. 在spring加载完成之后的，dubbo重写了ApplicationListener.onApplicationEvent()方法
2. 在ServiceBean.export方法