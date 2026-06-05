# Spring Boot + MyBatis Plus 后端开发规范

> 整合 MyBatis Plus 与 Spring Boot 项目规范，技术栈：JDK 17+ / Spring Boot 3.x / MyBatis-Plus / Lombok / Hutool / Maven / MySQL。

## 1. 设计原则与分层职责

- 遵循 SRP、OCP、DIP、ISP、KISS、DRY 原则。
- 分层调用链路：`Controller → Service → Mapper → Database`，**禁止跨层调用**（Controller 直接调用 Mapper）。
- 各层职责清晰：
  - Controller：接收参数、封装返回结果，不写业务逻辑。
  - Service：业务逻辑处理，事务控制，组合 Mapper 操作。
  - Mapper：仅负责数据库 CRUD、SQL 编写与映射，不参杂业务。

## 2. 项目包结构

```
com.company.project
├── common
│   ├── core
│   │   ├── constant        # 常量接口
│   │   └── util            # 通用工具（如统一返回 R<T>）
│   └── config              # 全局配置类
├── exception               # 全局异常处理
├── <business-module>       # 业务模块（如 dining）
│   ├── controller          # 控制器
│   ├── service             # 服务接口
│   │   └── impl            # 服务实现
│   ├── mapper              # Mapper 接口
│   ├── entity              # 数据库实体
│   ├── dto                 # 数据传输对象
│   ├── vo                  # 视图对象
│   └── convert             # 对象转换器（MapStruct）
└── job                     # 定时任务
```

## 3. 通用约定

### 3.1 依赖注入
- **强制** 使用 `@RequiredArgsConstructor` + `private final` 构造器注入。
- **禁止** `@Autowired` 字段注入。

### 3.2 统一返回结果
- Controller 返回值统一为 `R<T>`，常用：`R.ok()`、`R.ok(data)`、`R.failed("msg")`。
- JSON 结构：`{ "code": 200, "message": "success", "data": {} }`。

### 3.3 注释规范
- 类、接口、Service 方法使用 `/** */` Javadoc（中文描述功能、参数、返回值）。
- Controller 方法上方用 `// 行注释` 说明用途。
- Entity/DTO/VO 每个字段须 `/** */` 中文说明。

## 4. Entity 层规范

- 注解：`@Data`、`@TableName("表名")`、`@EqualsAndHashCode(callSuper = true)`（继承 Model 时）。
- **必须继承** `Model<Entity>`。
- 主键：`@TableId(type = IdType.ASSIGN_ID)`，类型为 `String`（雪花算法）。
- 审计字段自动填充：
  - `createTime`：`@TableField(fill = FieldFill.INSERT)`，类型 `LocalDateTime`。
  - `updateTime`：`@TableField(fill = FieldFill.INSERT_UPDATE)`。
- 逻辑删除：`@TableLogic` + `@TableField(fill = FieldFill.INSERT)`，字段名 `delFlag`，类型 `Integer`（0-正常，1-删除）。
- 时间字段统一使用 `LocalDateTime`，数据库字段全小写下划线。

```java
@Data
@TableName("dish")
@EqualsAndHashCode(callSuper = true)
public class DishEntity extends Model<DishEntity> {
    @TableId(type = IdType.ASSIGN_ID)
    private String id;
    private String name;
    private Integer status;
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
    @TableLogic
    @TableField(fill = FieldFill.INSERT)
    private Integer delFlag;
}
```

## 5. Mapper 层规范

- 接口加 `@Mapper`，继承 `BaseMapper<Entity>`。
- 简单 CRUD 直接使用 MyBatis-Plus 内置方法，**禁止重复声明** `selectById` 等基础方法。
- 复杂查询（多表、聚合、动态 SQL）写在对应的 XML 文件中，namespace 完整匹配接口全限定名。
- XML 文件命名：`XxxMapper.xml`，方法名语义清晰（如 `selectPage`、`selectListByCondition`）。
- 禁止在注解 `@Select` 中编写复杂 SQL（多表 JOIN、GROUP BY 等）。

```java
@Mapper
public interface DishMapper extends BaseMapper<DishEntity> {
    // 复杂查询扩展
    List<DishVO> selectDishWithIngredients(@Param("id") Long id);
}
```

## 6. Service 层规范

### 接口定义
- **必须** 继承 `IService<Entity>`。
- 方法使用 `/** */` Javadoc 描述功能、参数、返回值。

### 实现类
- 注解 `@Service`、`@Slf4j`、`@RequiredArgsConstructor`。
- **必须** 继承 `ServiceImpl<Mapper, Entity>` 并实现对应接口。
- 写操作（新增、修改、删除）**必须**加 `@Transactional(rollbackFor = Exception.class)`。
- 依赖注入优先注入 Mapper（避免 Service 间循环依赖）。

```java
@Service
@Slf4j
@RequiredArgsConstructor
public class DishServiceImpl extends ServiceImpl<DishMapper, DishEntity> implements DishService {
    private final DishIngredientMapper dishIngredientMapper;

    @Override
    @Transactional(rollbackFor = Exception.class)
    public boolean saveDish(DishSaveDTO dto) {
        DishEntity entity = new DishEntity();
        BeanUtil.copyProperties(dto, entity);
        return save(entity);
    }
}
```

## 7. Controller 层规范

- 类注解：`@RestController`、`@RequiredArgsConstructor`、`@RequestMapping("/模块/业务")`。
- 职责单一：接收参数并调用 Service，返回 `R<T>`，**禁止出现 Mapper**。
- 每个接口方法必须包裹 `try-catch`，异常抛出 `ResponseStatusException` 并附加明确消息。
- 标准模板：
  - 分页：`@GetMapping("/page")`，入参接收 `XxxQueryDTO`，返回 `R<Page<XxxVO>>`。
  - 详情：`@GetMapping("/{id}")`。
  - 列表：`@GetMapping("/list")`。
  - 新增：`@PostMapping`，入参 `@RequestBody XxxSaveDTO`。
  - 修改：`@PutMapping`，入参 `@RequestBody XxxSaveDTO`。
  - 删除：`@DeleteMapping`，入参 `@RequestBody List<String> ids`。

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/dining/dish")
public class DishController {
    private final DishService dishService;

    @GetMapping("/page")
    public R<Page<DishVO>> getPage(DishQueryDTO dto) {
        try {
            return R.ok(dishService.queryPage(dto));
        } catch (Exception e) {
            throw new ResponseStatusException(HttpStatus.INTERNAL_SERVER_ERROR, "分页查询菜品失败" + e.getMessage());
        }
    }
}
```

## 8. DTO / VO 与数据转换

- DTO 用于接收前端参数，VO 用于返回数据，Entity 仅与数据库交互。
- 数据流转：`Controller ← DTO → Service → Entity → VO → 前端`，**禁止直接返回 Entity**。
- VO 中的时间字段统一加 `@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")`。
- 对象拷贝推荐使用 Hutool `BeanUtil.copyProperties()` 或 MapStruct。

```java
@Data
public class DishSaveDTO {
    private String id;
    private String name;
}

@Data
public class DishVO {
    private String id;
    private String name;
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private LocalDateTime createTime;
}
```

## 9. REST 接口设计

遵循资源导向，URI 中禁止动词，全部使用名词复数。

| 操作       | 方法   | 示例                 |
| ---------- | ------ | -------------------- |
| 分页查询   | GET    | `/users?page=1&size=20` |
| 详情查询   | GET    | `/users/{id}`        |
| 列表查询   | GET    | `/users/list`        |
| 新增       | POST   | `/users`             |
| 修改       | PUT    | `/users`             |
| 局部修改   | PATCH  | `/users/{id}`        |
| 删除       | DELETE | `/users/{id}`        |

**错误示例**：`GET /getUser`、`POST /addUser`、`POST /deleteUser`。

## 10. 查询构造规范

### 10.1 条件构造器
- **必须优先使用** `LambdaQueryWrapper` 和 `LambdaUpdateWrapper`，禁止字符串字段名。
- 使用 `Wrappers.lambdaQuery()` 或 `Wrappers.<Entity>lambdaQuery()` 构建。
- 条件拼接支持带判断的简洁写法：`wrapper.like(StringUtils.hasText(keyword), Entity::getName, keyword)`。
- 逻辑删除条件统一：`wrapper.eq(Entity::getDelFlag, 0)`。
- 字符串非空判断用 `StrUtil.isNotBlank()`（Hutool），集合非空用 `CollUtil.isNotEmpty()`。

```java
LambdaQueryWrapper<User> wrapper = Wrappers.lambdaQuery();
wrapper.eq(User::getStatus, 1)
       .like(StrUtil.isNotBlank(name), User::getUsername, name);
```

### 10.2 分页查询
- **必须使用** MyBatis-Plus 分页插件：`Page<Entity> page = new Page<>(current, size);`
- 返回 `Page<VO>`，**禁止** 先查全部再内存分页（`list.stream().skip().limit()`）。

### 10.3 批量操作
- 优先使用 Service 内置的 `saveBatch()`、`updateBatchById()` 等方法。
- 大批量数据（超过1000条）分批处理，每批 500~1000 条，使用 `Lists.partition(list, 1000)`。

## 11. 逻辑删除与自动填充

- 逻辑删除字段统一命名 `delFlag`，类型 `Integer`（0-正常，1-删除），禁止 `is_delete`、`delete_flag` 等。
- 数据库字段：`del_flag tinyint default 0`。
- Entity 对应属性加 `@TableLogic` 和 `@TableField(fill = FieldFill.INSERT)`。
- 自动填充实现 `MetaObjectHandler`，处理 `createTime` 和 `updateTime`。

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }
    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }
}
```

## 12. 异常处理规范

- Controller 每个方法均需 `try-catch`，捕获 `Exception` 并抛出 `ResponseStatusException`，错误消息包含具体业务描述。
- **禁止** 空 catch 吞异常。
- 全局异常处理器 `@RestControllerAdvice` 兜底，处理业务异常 `BusinessException` 和未知异常，返回 `R.failed("...")`。
- 自定义业务异常推荐 `BusinessException`，扩展 `RuntimeException`。

## 13. 日志规范

- 统一使用 Lombok `@Slf4j`。
- 日志级别：info 记录关键业务操作，error 记录异常详情（含参数），debug 用于开发调试。
- 日志输出必须使用占位符 `log.info("用户登录成功, userId={}", userId);`
- **禁止** `System.out.println()` 和 `e.printStackTrace()`。

## 14. 枚举与常量

- 所有固定值范围的状态、类型使用枚举，**禁止魔法值**（如 `if(status == 1)`）。
- 枚举类名加 `Enum` 后缀，内部字段 `code`、`desc`，使用 `@Getter`、`@RequiredArgsConstructor`。

```java
@Getter
@RequiredArgsConstructor
public enum UserStatusEnum {
    ENABLE(1, "启用"),
    DISABLE(0, "禁用");
    private final Integer code;
    private final String desc;
}
```

## 15. 接口文档与测试

- 使用 OpenAPI 注解：Controller 类加 `@Tag(name = "模块名")`，方法加 `@Operation(summary = "说明")`。
- 单元测试类命名 `被测试类 + Test`，方法命名 `shouldXxx` 或 `methodName_condition_expectedBehavior`，覆盖 Service、Util、Converter 层。

## 16. 快速检查清单

| 层级         | 检查项 |
| ------------ | ------ |
| **Entity**   | 继承 Model，主键 ASSIGN_ID String，时间 LocalDateTime，逻辑删除 delFlag + @TableLogic，字段注释。 |
| **Mapper**   | 继承 BaseMapper，复杂查询用 XML，禁止字符串字段拼接。 |
| **Service**  | 接口继承 IService，实现继承 ServiceImpl，写操作加 @Transactional，使用 LambdaQueryWrapper。 |
| **Controller**| @RequiredArgsConstructor 注入，返回 R<T>，try-catch 抛 ResponseStatusException，禁止调用 Mapper。 |
| **DTO/VO**   | 字段注释，VO 时间加 @JsonFormat，禁止返回 Entity。 |
| **通用**     | 构造器注入，分页用 Page，批量操作分批，魔法值替换为枚举，日志用 @Slf4j。 |