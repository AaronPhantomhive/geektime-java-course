# MySQL架构

## MySQL架构图

![MySQL架构图](.\note\MySQL架构图.png)

- Connectors连接器：负责跟客户端建立连接
- Management Serveices & Utilities系统管理和控制工具
- Connection Pool连接池：管理用户连接，监听并接收连接的请求，转发所有连接的请求到线程管理模块
- SQL Interface SQL接口：接受用户的SQL命令，并且返回SQL执行结果
- Parser解析器：SQL传递到解析器的时候会被解析器验证和解析
- Optimizer查询优化器：SQL语句在查询之前会使用查询优化器对查询进行优化，explain语句查看的SQL语句执行计划，就是由此优化器生成
- Cache和Buffer查询缓存：在MySQL5.7中包含缓存组件。在MySQL8中移除了
- Pluggable Storage Engines存储引擎：存储引擎就是存取数据、建立与更新索引、查询数据等技术的实现方法

