# AICodeSpec — AI 辅助开发规范

一套面向 AI 辅助编码的 Java 后端开发规范文档，确保 AI 生成的代码风格统一、质量可控、可维护。

## 规范文档

| 文档 | 说明 |
|------|------|
| [Java 开发规范](./Java开发规范.md) | 命名、格式、OOP、并发、集合、异常日志等通用 Java 编码规范 |
| [MySQL 数据库设计与开发规范](./MySQL数据库设计与开发规范.md) | 建表规约、索引规范、SQL 编写、ORM 映射等数据库规范 |
| [Spring Boot + MyBatis Plus 后端开发规范](./Spring%20Boot%20%2B%20MyBatis%20Plus%20后端开发规范.md) | 分层架构、Entity/Mapper/Service/Controller 层编码模板与约定 |

## 技术栈

- **语言**: Java 17+
- **框架**: Spring Boot 3.x
- **ORM**: MyBatis-Plus
- **数据库**: MySQL 8.0+
- **工具库**: Lombok / Hutool / MapStruct
- **构建**: Maven

## 目录结构

```
.
├── Java开发规范.md                              # Java 通用规范
├── MySQL数据库设计与开发规范.md                    # 数据库规范
├── Spring Boot + MyBatis Plus 后端开发规范.md      # 框架层规范
└── README.md
```

## 快速使用

将本仓库作为 AI 编码助手的项目上下文（如 CLAUDE.md / GEMINI.md / AGENTS.md），或直接引用对应规范文件：

```
请遵循 Java开发规范.md 中的命名和代码格式要求，实现用户登录功能。
```

```
按照 Spring Boot + MyBatis Plus 后端开发规范.md 的分层模板，生成完整的 CRUD 模块。
```

```
根据 MySQL数据库设计与开发规范.md 的建表规约，设计订单模块的数据库表结构。
```

## 规范概要

### Java 开发规范
- 命名：大驼峰类名、小驼峰方法、全大写常量
- 格式：4 空格缩进、K&R 大括号、120 字符行宽
- OOP：POJO 用包装类型、构造不写业务逻辑、访问控制从严
- 并发：显式线程池、乐观/悲观锁、finally 释放资源
- 异常：不吞异常、finally 禁 return、优先自定义业务异常
- 日志：SLF4J 占位符、生产禁 debug、敏感信息脱敏

### MySQL 规范
- 命名：全小写下划线、is_xxx 布尔字段、pk_/uk_/idx_ 索引前缀
- 建表：必须有 id / create_time / update_time / is_deleted
- 类型：decimal 替代 float/double、varchar 不超 5000、utf8mb4 字符集
- 索引：最左前缀、覆盖索引、禁止左模糊、EXPLAIN 检查执行计划
- SQL：禁 SELECT *、COUNT(*)、禁超过三表 JOIN、#{} 防注入

### Spring Boot + MyBatis Plus 规范
- 分层：Controller → Service → Mapper，禁止跨层调用
- 注入：@RequiredArgsConstructor 构造器注入，禁 @Autowired 字段注入
- Entity：继承 Model、ASSIGN_ID 雪花主键、LocalDateTime 时间、@TableLogic 逻辑删除
- Service：继承 ServiceImpl、写操作加 @Transactional、LambdaQueryWrapper
- Controller：返回 R\<T\>、try-catch 统一异常、禁直接调用 Mapper
- 接口：RESTful 风格、名词复数、禁 URI 动词

## License

MIT
