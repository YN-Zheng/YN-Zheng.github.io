# MyBatis

### 连接池

实际开发中都会使用连接池,减少获取连接所消耗的时间

在主配置文件SqlMapConfig.xml中的DataSource标签中, 可设置type属性来确定何种连接方式:

- UNPOOLED: 不使用连接池, 直接实现一个连接
- POOLED: 使用连接池
  - 如果有空闲池, 直接拿来用
  - 如果没有空闲的了,  
    - 连接数未到最大值, 新建连接
    - 否则使用最老的那个连接
- 服务器提供的JNDI技术实现(略)



### 事务

- 定义: 通过sqlsession对象的commit方法和rollback方法实现事务的提交和回滚
- 可以创建session时, 在 openSession(true) 中设置自动提交



