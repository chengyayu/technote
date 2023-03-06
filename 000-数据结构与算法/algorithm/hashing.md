# 哈希算法（Hashing）

## 定义

将任意长度的二进制值串映射为固定长度的二进制值串，这个映射的规则就是**哈希算法**，而通过原始数据映射之后得到的二进制值串就是**哈希值**。

一个优秀的哈希算法满足以下要求：

- 从哈希值不能反向推导出原始数据（所以哈希算法也叫单向哈希算法）；
- 对输入数据非常敏感，哪怕原始数据只修改了一个 Bit，最后得到的哈希值也大不相同；
- 散列冲突的概率要很小，对于不同的原始数据，哈希值相同的概率非常小；
- 哈希算法的执行效率要尽量高效，针对较长的文本，也能快速地计算出哈希值。

## 应用

## 参考资料

- [一致哈希 - wikipedia](https://en.wikipedia.org/wiki/Consistent_hashing)
- [白话解析：一致性哈希算法(consistent hashing)](https://www.zsythink.net/archives/1182)
- [consistenthash - groupcache](https://github.com/golang/groupcache/blob/master/consistenthash/consistenthash.go)