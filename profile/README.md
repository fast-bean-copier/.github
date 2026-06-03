# Fast Bean Copier

高性能的 Java Bean 拷贝工具，使用 APT 在编译期自动生成拷贝代码，零运行时开销。

> **v1.5.0 新特性**：Bean ↔ Map 转换（@CopyToMap/@CopyFromMap/@MapKeyStrategy）；API 清理（移除 beforeMapping 和单参数 customizer）。

## 特性

- ✅ 编译期代码生成，零运行时反射
- ✅ 类型安全，编译期检查
- ✅ Bean ↔ Map 转换（v1.5.0）
- ✅ 函数式处理：preProcessor + postProcessor 双处理器
- ✅ 深拷贝控制：@CopyField.deepCopy 字段级控制
- ✅ 嵌套对象深拷贝，支持多层级
- ✅ 多字段映射：多对一、一对多
- ✅ 内置 TypeConverter：数字、日期、枚举、JSON
- ✅ 依赖注入：Spring / CDI / JSR-330
- ✅ 更新现有对象：updateDto / updateEntity
- ✅ 条件映射、默认值、常量值
- ✅ 全局配置：包级别 + Properties 文件

## 快速开始

### 1. 添加依赖

```xml
<dependency>
    <groupId>com.github.jackieonway</groupId>
    <artifactId>fast-bean-copier-annotations</artifactId>
    <version>1.5.0</version>
</dependency>

<dependency>
    <groupId>com.github.jackieonway</groupId>
    <artifactId>fast-bean-copier-processor</artifactId>
    <version>1.5.0</version>
    <scope>provided</scope>
</dependency>
```

### 2. Bean ↔ Bean 拷贝

```java
@CopyTarget(source = User.class, ignore = {"password"})
public class UserDto {
    private Long id;
    private String name;
    private String email;
    // getter/setter...
}
```

```bash
mvn clean compile
```

```java
UserDto dto = UserDtoCopier.toDto(user);
User entity = UserDtoCopier.fromDto(dto);
List<UserDto> dtos = UserDtoCopier.toDtoList(users);

// 函数式处理
UserDto dto2 = UserDtoCopier.toDto(user,
    src -> { src.setName(src.getName().trim()); return src; },
    tgt -> { tgt.setDisplayName(tgt.getName().toUpperCase()); return tgt; }
);
```

### 3. Bean ↔ Map 转换（v1.5.0）

```java
@CopyToMap(keyStrategy = MapKeyStrategy.SNAKE_CASE)
@CopyFromMap
public class UserDto {
    private Long id;
    private String firstName;          // key: "first_name"
    @CopyField(mapKey = "email_addr")
    private String email;              // key: "email_addr"
}

// Bean → Map
Map<String, Object> map = UserDtoMapCopier.toMap(userDto);
List<Map<String, Object>> mapList = UserDtoMapCopier.toMapList(userDtos);

// Map → Bean
UserDto dto = UserDtoMapCopier.fromMap(map);
List<UserDto> dtos = UserDtoMapCopier.fromMapList(mapList);
```

## 从 v1.4.x 升级

v1.5.0 移除了 `beforeMapping` 和单参数 `customizer` 重载，请改用 `preProcessor` / `postProcessor`。详见 [快速入门 - 迁移指南](https://fast-bean-copier.github.io/getting-started.html) 与 [FAQ](https://fast-bean-copier.github.io/faq.html)。

## 文档

- [官方文档](https://fast-bean-copier.github.io) - 完整的参考文档

## 许可证

Apache License 2.0

## 作者

jackieonway

## 获取帮助

- 查看 [常见问题解答](https://fast-bean-copier.github.io/faq.html)
- 在 [GitHub Issues](https://github.com/jackieonway/fast-bean-copier/issues) 中提出问题
