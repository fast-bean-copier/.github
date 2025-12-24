# Fast Bean Copier

Fast Bean Copier 是一个高性能的 Java Bean 拷贝工具，使用 APT（注解处理工具）在编译期自动生成拷贝代码，实现零运行时开销。

> v1.1 新特性：集合/数组字段深拷贝（含嵌套组合、多维数组）与反向拷贝；raw/无界通配符集合自动降级浅拷贝并输出编译期警告。

## 特性

- ✅ **编译期代码生成** - 使用 APT 在编译期生成拷贝代码，零运行时反射
- ✅ **类型安全** - 编译期类型检查，避免运行时错误
- ✅ **高性能** - 生成的代码直接调用 getter/setter，性能最优
- ✅ **易用** - 只需添加 `@CopyTarget` 注解即可
- ✅ **灵活** - 支持字段忽略、类型转换、集合处理
- ✅ **完整** - 支持双向拷贝、集合/Map/数组拷贝、嵌套对象

## 快速开始

### 1. 添加依赖

```xml
<!-- 注解模块 -->
<dependency>
    <groupId>com.github.jackieonway</groupId>
    <artifactId>fast-bean-copier-annotations</artifactId>
    <version>1.1.0</version>
</dependency>

<!-- 处理器模块（编译时依赖） -->
<dependency>
    <groupId>com.github.jackieonway</groupId>
    <artifactId>fast-bean-copier-processor</artifactId>
    <version>1.1.0</version>
    <scope>provided</scope>
</dependency>
```

### 2. 定义 DTO 类

```java
import com.github.jackieonway.copier.annotation.CopyTarget;

@CopyTarget(source = User.class, ignore = {"password"})
public class UserDto {
    private Long id;
    private String name;
    private String email;
    private Integer age;
    
    // getter/setter...
}
```

### 3. 编译

编译时会自动生成 `UserDtoCopier` 类：

```bash
mvn clean compile
```

### 4. 使用

```java
// 单个对象拷贝
User user = new User(1L, "张三", "secret", "zhangsan@example.com", 25);
UserDto userDto = UserDtoCopier.toDto(user);

// 反向拷贝
User converted = UserDtoCopier.fromDto(userDto);

// 集合拷贝
List<UserDto> userDtos = UserDtoCopier.toDtoList(users);
Set<UserDto> userDtoSet = UserDtoCopier.toDtoSet(users);

// 反向集合拷贝
List<User> convertedUsers = UserDtoCopier.fromDtoList(userDtos);
Set<User> convertedUserSet = UserDtoCopier.fromDtoSet(userDtoSet);
```

## 功能说明

### 同名字段拷贝

自动拷贝源类和目标类中同名的字段：

```java
@CopyTarget(source = User.class)
public class UserDto {
    private Long id;        // 自动从 User.id 拷贝
    private String name;    // 自动从 User.name 拷贝
    private String email;   // 自动从 User.email 拷贝
}
```

### 字段忽略

使用 `ignore` 属性忽略不需要拷贝的字段：

```java
@CopyTarget(source = User.class, ignore = {"password", "secretKey"})
public class UserDto {
    private Long id;
    private String name;
    private String email;
    // password 和 secretKey 不会被拷贝
}
```

### 类型转换

自动支持基本类型与包装类型的转换：

```java
// User 中使用基本类型
public class User {
    private long id;
    private int age;
    private double salary;
}

// UserDto 中使用包装类型
@CopyTarget(source = User.class)
public class UserDto {
    private Long id;        // long -> Long
    private Integer age;    // int -> Integer
    private Double salary;  // double -> Double
}

// 反向拷贝时自动处理 null 值
User user = UserDtoCopier.fromDto(userDto);
// 如果 userDto.age 为 null，则 user.age 为 0
```

### 集合/Map/数组拷贝（含反向）

支持 List、Set、Map、数组及嵌套组合：

```java
List<UserDto> dtos = UserDtoCopier.toDtoList(users);
Set<UserDto> dtoSet = UserDtoCopier.toDtoSet(users);
Map<String, UserDto> dtoMap = UserDtoCopier.toDtoMap(userMap);
UserDto[] dtoArr = UserDtoCopier.toDtoArray(userArr);

// 反向
List<User> users = UserDtoCopier.fromDtoList(dtos);
Set<User> userSet = UserDtoCopier.fromDtoSet(dtoSet);
Map<String, User> usersMap = UserDtoCopier.fromDtoMap(dtoMap);
User[] restoredArr = UserDtoCopier.fromDtoArray(dtoArr);
```

### Null 与通配符处理

所有方法都支持 null 值处理，集合/Map/数组中的 null 元素会被保留。对 raw/无界通配符集合，会降级为浅拷贝并给出编译期警告，建议为集合声明明确泛型。

```java
// 传入 null 返回 null
UserDto dto = UserDtoCopier.toDto(null);  // 返回 null

// 集合中的 null 元素被跳过
List<UserDto> dtos = UserDtoCopier.toDtoList(Arrays.asList(user1, null, user2));
// 结果中包含 3 个元素，第二个为 null
```

## 生成的代码示例

编译后自动生成的 `UserDtoCopier` 类：

```java
public final class UserDtoCopier {
    private UserDtoCopier() {
        throw new AssertionError("No instances of UserDtoCopier");
    }
    
    public static UserDto toDto(User source) {
        if (source == null) return null;
        UserDto target = new UserDto();
        target.setId(source.getId());
        target.setName(source.getName());
        target.setEmail(source.getEmail());
        target.setAge(source.getAge());
        return target;
    }
    
    public static User fromDto(UserDto source) {
        if (source == null) return null;
        User target = new User();
        target.setId(source.getId());
        target.setName(source.getName());
        target.setEmail(source.getEmail());
        target.setAge(source.getAge());
        return target;
    }
    
    public static List toDtoList(List sources) {
        if (sources == null) return null;
        List result = new ArrayList(sources.size());
        for (Object source : sources) {
            result.add(toDto((User) source));
        }
        return result;
    }
    
    // ... 其他方法
}
```

## 项目结构

```
fast-bean-copier/
├── fast-bean-copier-annotations/      # 注解定义模块
│   └── CopyTarget.java
├── fast-bean-copier-processor/        # APT 处理器模块
│   ├── BeanCopierProcessor.java
│   ├── CodeGenerator.java
│   ├── TypeUtils.java
│   └── FieldMapping.java
└── fast-bean-copier-examples/         # 示例和测试
    ├── User.java / UserDto.java
    ├── Product.java / ProductDto.java
    └── 测试类...
```

## 测试

项目包含完整的单元测试：

```bash
mvn clean test
```

测试覆盖：
- 同名字段拷贝
- 基本类型与包装类型转换
- 字段忽略功能
- 集合拷贝
- 嵌套对象处理
- Null 值处理

## 性能

Fast Bean Copier 在编译期生成代码，运行时性能与手写代码相同：

- 无反射开销
- 无动态代理开销
- 直接调用 getter/setter
- 性能最优

## 常见问题

### Q: 支持哪些 Java 版本？
A: Java 8 及以上版本。

### Q: 支持嵌套对象拷贝吗？
A: 支持。同名字段会直接拷贝，不同类型的嵌套对象需要在应用层手动处理。

### Q: 支持自定义转换器吗？
A: 当前版本不支持，可以在应用层手动处理特殊字段。

### Q: 生成的代码在哪里？
A: 在 `target/generated-sources/annotations/` 目录下。

## 文档

- [官方文档](https://fast-bean-copier.github.io/)

## 许可证

Apache License 2.0

## 作者

jackieonway

## 获取帮助

- 在 [GitHub Issues](https://github.com/fast-bean-copier/fast-bean-copier/issues) 中提出问题
