# Java 注解（Annotation）学习文档

## 1. 什么是注解

### 1.1 注解的定义

注解（Annotation）是Java 5引入的一种元数据机制，用于为代码提供额外的信息。注解本身不会改变程序的执行逻辑，但可以被编译器、开发工具或运行时环境读取和处理。

注解的本质是一个继承了 `java.lang.annotation.Annotation` 接口的特殊接口。

```java
// 注解的底层实现
public interface MyAnnotation extends java.lang.annotation.Annotation {
    String value();
}
```

### 1.2 注解的作用

1. **编译检查**：如 `@Override` 检查方法是否正确重写
2. **代码生成**：如 Lombok 使用注解生成样板代码
3. **运行时处理**：如 Spring 使用注解进行依赖注入
4. **文档生成**：如 `@Deprecated` 标记过时的API
5. **配置替代**：使用注解替代XML配置文件

### 1.3 注解与注释的区别

| 特性 | 注解（Annotation） | 注释（Comment） |
|------|-------------------|-----------------|
| 语法 | `@AnnotationName` | `//` 或 `/* */` |
| 作用对象 | 编译器、工具、运行时 | 开发人员 |
| 是否编译 | 可以保留到class文件或运行时 | 编译时完全丢弃 |
| 可读性 | 机器可读 | 仅人类可读 |
| 功能 | 影响程序行为 | 仅作说明 |

## 2. 注解的基本语法

### 2.1 使用注解

```java
@AnnotationName
public class MyClass {

    @AnnotationName(value = "example")
    private String field;

    @AnnotationName(param1 = "value1", param2 = 123)
    public void method() {
        // 方法体
    }
}
```

### 2.2 注解的位置

注解可以应用于：
- 类、接口、枚举
- 方法
- 字段
- 参数
- 构造函数
- 局部变量
- 包（package-info.java）
- 类型参数（Java 8+）

```java
// 类型使用注解示例（Java 8+）
public class TypeAnnotationExample {
    // 字段类型注解
    private @NonNull String name;

    // 泛型类型注解
    private List<@NonNull String> items;

    // 方法返回类型注解
    public @Nullable String getName() {
        return name;
    }

    // 异常类型注解
    public void process() throws @Critical Exception {
    }
}
```

### 2.3 注解的属性赋值

```java
// 1. 无属性注解
@Override
public void method() {}

// 2. 单值注解（value属性可省略名称）
@SuppressWarnings("unchecked")
public void method() {}

// 3. 多属性注解
@RequestMapping(value = "/api", method = RequestMethod.GET)
public void method() {}

// 4. 数组属性
@SuppressWarnings({"unchecked", "deprecation"})
public void method() {}

// 5. 单元素数组可省略花括号
@SuppressWarnings("unchecked")  // 等同于 {"unchecked"}
public void method() {}
```

## 3. 内置注解

Java提供了一些内置的标准注解：

### 3.1 @Override

标记方法重写父类方法，编译器会检查是否正确重写。

```java
public class Child extends Parent {
    @Override
    public void method() {
        // 重写父类方法
    }
}
```

### 3.2 @Deprecated

标记已过时的API，使用时编译器会发出警告。

```java
@Deprecated
public void oldMethod() {
    // 不推荐使用的方法
}

// Java 9+ 可以添加更多信息
@Deprecated(since = "1.5", forRemoval = true)
public void veryOldMethod() {
    // 将在未来版本中移除
}
```

### 3.3 @SuppressWarnings

抑制编译器警告。

```java
@SuppressWarnings("unchecked")
public void method() {
    List list = new ArrayList(); // 不会产生未检查警告
}

// 常见的警告类型
@SuppressWarnings({"unchecked", "deprecation", "rawtypes"})
```

**常见警告类型：**

| 警告类型 | 说明 |
|---------|------|
| `unchecked` | 未检查的类型转换 |
| `deprecation` | 使用了过时的API |
| `rawtypes` | 使用了原始类型 |
| `unused` | 未使用的变量或方法 |
| `serial` | 可序列化类缺少serialVersionUID |
| `all` | 抑制所有警告 |

### 3.4 @SafeVarargs

标记方法或构造函数的可变参数是类型安全的（Java 7+）。

```java
@SafeVarargs
public final <T> void method(T... args) {
    // 处理可变参数
}
```

### 3.5 @FunctionalInterface

标记接口为函数式接口，只能有一个抽象方法（Java 8+）。

```java
@FunctionalInterface
public interface MyFunction {
    void execute();

    // 可以有默认方法和静态方法
    default void defaultMethod() {}
    static void staticMethod() {}
}
```

## 4. 元注解

元注解（Meta-Annotation）是用于定义注解的注解，位于 `java.lang.annotation` 包中。

### 4.1 @Target

指定注解可以应用的位置。

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)  // 只能用于方法
public @interface MyMethodAnnotation {
}

@Target({ElementType.TYPE, ElementType.FIELD})  // 可用于类和字段
public @interface MyAnnotation {
}
```

**ElementType 枚举值：**

| 枚举值 | 说明 | 示例 |
|-------|------|------|
| `TYPE` | 类、接口、枚举 | `@Entity class User {}` |
| `FIELD` | 字段 | `@Column private String name;` |
| `METHOD` | 方法 | `@Override public void run() {}` |
| `PARAMETER` | 参数 | `void foo(@NotNull String s) {}` |
| `CONSTRUCTOR` | 构造函数 | `@Autowired public User() {}` |
| `LOCAL_VARIABLE` | 局部变量 | `@SuppressWarnings("unused") int x;` |
| `ANNOTATION_TYPE` | 注解类型 | `@Documented @interface MyAnn {}` |
| `PACKAGE` | 包 | package-info.java中使用 |
| `TYPE_PARAMETER` | 类型参数（Java 8+） | `class Box<@NonNull T> {}` |
| `TYPE_USE` | 任何类型使用（Java 8+） | `@NonNull String s;` |

### 4.2 @Retention

指定注解的保留策略。

```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)  // 运行时可见
public @interface RuntimeAnnotation {
}
```

**RetentionPolicy 枚举值：**

| 策略 | 说明 | 使用场景 |
|-----|------|---------|
| `SOURCE` | 只在源代码中保留，编译时丢弃 | `@Override`、`@SuppressWarnings` |
| `CLASS` | 编译到class文件中，但运行时不可见（默认值） | 字节码工具处理 |
| `RUNTIME` | 运行时可见，可通过反射读取 | Spring、JPA等框架注解 |

```
源代码(.java) --编译--> 字节码(.class) --JVM加载--> 运行时
    ↑                      ↑                        ↑
  SOURCE                 CLASS                   RUNTIME
```

### 4.3 @Documented

标记注解是否包含在JavaDoc中。

```java
import java.lang.annotation.Documented;

@Documented
public @interface MyDocumentedAnnotation {
}
```

### 4.4 @Inherited

标记注解是否可以被子类继承（仅对类有效，对接口、方法、字段无效）。

```java
import java.lang.annotation.Inherited;

@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface InheritedAnnotation {
}

@InheritedAnnotation
public class Parent {
}

// Child 类会继承 @InheritedAnnotation
public class Child extends Parent {
}

// 验证继承
Child.class.isAnnotationPresent(InheritedAnnotation.class);  // true
```

### 4.5 @Repeatable

允许在同一位置多次使用同一注解（Java 8+）。

```java
import java.lang.annotation.Repeatable;

// 1. 定义可重复注解
@Repeatable(Schedules.class)
@Retention(RetentionPolicy.RUNTIME)
public @interface Schedule {
    String day();
    String time() default "00:00";
}

// 2. 定义容器注解
@Retention(RetentionPolicy.RUNTIME)
public @interface Schedules {
    Schedule[] value();
}

// 3. 使用
@Schedule(day = "Monday", time = "09:00")
@Schedule(day = "Wednesday", time = "14:00")
@Schedule(day = "Friday", time = "16:00")
public void weeklyTask() {
}
```

## 5. 自定义注解

### 5.1 定义注解

使用 `@interface` 关键字定义注解。

```java
import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyAnnotation {
    // 注解元素（类似方法）
    String value() default "";  // 默认值
    int count() default 0;
    String[] tags() default {};
}
```

### 5.2 注解元素规则

1. **元素类型只能是：**
   - 基本类型（int, long, boolean等）
   - String
   - Class
   - 枚举
   - 注解
   - 以上类型的数组

2. **元素不能有参数**
3. **元素不能抛出异常**
4. **可以使用 `default` 指定默认值**

```java
public @interface ComplexAnnotation {
    // 基本类型
    int intValue() default 0;
    boolean boolValue() default true;

    // String
    String name();

    // Class
    Class<?> type() default Object.class;

    // 枚举
    Priority priority() default Priority.MEDIUM;

    // 注解
    Author author() default @Author(name = "Unknown");

    // 数组
    String[] tags() default {};
    int[] numbers() default {1, 2, 3};
}

enum Priority {
    LOW, MEDIUM, HIGH
}

@interface Author {
    String name();
}
```

### 5.3 使用自定义注解

```java
public class Example {

    // 使用默认值
    @MyAnnotation
    public void method1() {
    }

    // 指定value（特殊属性，可省略名称）
    @MyAnnotation("test")
    public void method2() {
    }

    // 指定多个属性
    @MyAnnotation(value = "test", count = 5, tags = {"tag1", "tag2"})
    public void method3() {
    }
}
```

### 5.4 注解的继承

注解本身不支持继承，但可以通过组合实现类似效果。

```java
// 基础注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface BaseService {
    String value() default "";
}

// 组合注解（包含多个注解）
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Component
@Transactional
public @interface TransactionalService {
    String value() default "";
}

// 使用组合注解
@TransactionalService("userService")
public class UserService {
    // 自动具有 @Component 和 @Transactional 的功能
}
```

## 6. 注解处理器

### 6.1 运行时注解处理（反射）

使用反射API在运行时读取注解信息。

```java
import java.lang.reflect.*;

public class AnnotationProcessor {

    public static void processAnnotations(Class<?> clazz) {
        // 处理类上的注解
        if (clazz.isAnnotationPresent(MyAnnotation.class)) {
            MyAnnotation annotation = clazz.getAnnotation(MyAnnotation.class);
            System.out.println("Class annotation: " + annotation.value());
        }

        // 处理方法上的注解
        for (Method method : clazz.getDeclaredMethods()) {
            if (method.isAnnotationPresent(MyAnnotation.class)) {
                MyAnnotation annotation = method.getAnnotation(MyAnnotation.class);
                System.out.println("Method: " + method.getName());
                System.out.println("Value: " + annotation.value());
                System.out.println("Count: " + annotation.count());
            }
        }

        // 处理字段上的注解
        for (Field field : clazz.getDeclaredFields()) {
            if (field.isAnnotationPresent(MyAnnotation.class)) {
                MyAnnotation annotation = field.getAnnotation(MyAnnotation.class);
                System.out.println("Field: " + field.getName());
            }
        }

        // 获取所有注解
        Annotation[] annotations = clazz.getAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println("Annotation: " + annotation.annotationType().getName());
        }
    }

    public static void main(String[] args) {
        processAnnotations(Example.class);
    }
}
```

### 6.2 编译时注解处理（APT）

使用注解处理器在编译时生成代码或进行检查（Annotation Processing Tool）。

```java
import javax.annotation.processing.*;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.*;
import javax.tools.Diagnostic;
import java.util.Set;

@SupportedAnnotationTypes("com.example.MyAnnotation")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class MyAnnotationProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations,
                          RoundEnvironment roundEnv) {
        for (TypeElement annotation : annotations) {
            Set<? extends Element> elements =
                roundEnv.getElementsAnnotatedWith(annotation);

            for (Element element : elements) {
                // 处理带注解的元素
                processingEnv.getMessager().printMessage(
                    Diagnostic.Kind.NOTE,
                    "Processing: " + element.getSimpleName()
                );

                // 可以生成新的源文件
                // JavaFileObject file = processingEnv.getFiler()
                //     .createSourceFile("GeneratedClass");
            }
        }
        return true;
    }
}
```

### 6.3 注解处理器的注册与配置

**方式1：使用 META-INF/services**

创建文件 `META-INF/services/javax.annotation.processing.Processor`：

```
com.example.MyAnnotationProcessor
```

**方式2：使用 Google Auto Service**

```java
import com.google.auto.service.AutoService;

@AutoService(Processor.class)
@SupportedAnnotationTypes("com.example.MyAnnotation")
public class MyAnnotationProcessor extends AbstractProcessor {
    // ...
}
```

**Maven 配置：**

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <annotationProcessorPaths>
                    <path>
                        <groupId>com.example</groupId>
                        <artifactId>my-annotation-processor</artifactId>
                        <version>1.0.0</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

## 7. 常见框架中的注解

### 7.1 Spring Framework

**依赖注入相关：**

```java
@Component          // 通用组件
@Service            // 业务层组件
@Repository         // 数据访问层组件
@Controller         // 控制层组件
@RestController     // RESTful控制器（@Controller + @ResponseBody）
@Autowired          // 自动装配
@Qualifier("name")  // 指定装配的bean名称
@Value("${key}")    // 注入配置值
@Primary            // 优先注入
@Lazy               // 延迟初始化
```

**配置相关：**

```java
@Configuration      // 配置类
@Bean               // 定义Bean
@ComponentScan      // 组件扫描
@PropertySource     // 加载属性文件
@Import             // 导入其他配置类
@Profile            // 环境配置
@Conditional        // 条件装配
```

**Web相关：**

```java
@RequestMapping("/api")         // 请求映射
@GetMapping("/users")           // GET请求
@PostMapping("/users")          // POST请求
@PutMapping("/users/{id}")      // PUT请求
@DeleteMapping("/users/{id}")   // DELETE请求
@PathVariable                   // 路径变量
@RequestParam                   // 请求参数
@RequestBody                    // 请求体
@ResponseBody                   // 响应体
@ResponseStatus                 // 响应状态码
@CrossOrigin                    // 跨域配置
```

**事务相关：**

```java
@Transactional                  // 事务管理
@Transactional(readOnly = true) // 只读事务
@Transactional(rollbackFor = Exception.class) // 回滚配置
```

**AOP相关：**

```java
@Aspect             // 切面
@Before             // 前置通知
@After              // 后置通知
@Around             // 环绕通知
@AfterReturning     // 返回后通知
@AfterThrowing      // 异常通知
@Pointcut           // 切点
```

### 7.2 JPA/Hibernate

**实体映射：**

```java
@Entity                         // 实体类
@Table(name = "users")          // 表名
@Id                             // 主键
@GeneratedValue(strategy = GenerationType.IDENTITY) // 主键生成策略
@Column(name = "user_name", length = 50, nullable = false) // 列映射
@Temporal(TemporalType.TIMESTAMP) // 时间类型
@Enumerated(EnumType.STRING)    // 枚举类型
@Transient                      // 不持久化
@Lob                            // 大对象
```

**关系映射：**

```java
@OneToOne           // 一对一
@OneToMany          // 一对多
@ManyToOne          // 多对一
@ManyToMany         // 多对多
@JoinColumn         // 外键列
@JoinTable          // 中间表
@Cascade            // 级联操作
@FetchType          // 加载策略
```

**查询相关：**

```java
@Query("SELECT u FROM User u WHERE u.name = :name")
@Param("name")
@NamedQuery
@NamedQueries
```

### 7.3 Lombok

**常用注解：**

```java
@Data               // 生成getter/setter/toString/equals/hashCode
@Getter             // 生成getter方法
@Setter             // 生成setter方法
@ToString           // 生成toString方法
@EqualsAndHashCode  // 生成equals和hashCode方法
@NoArgsConstructor  // 生成无参构造函数
@AllArgsConstructor // 生成全参构造函数
@RequiredArgsConstructor // 生成必需参数构造函数
@Builder            // 生成建造者模式
@Value              // 不可变类
@Slf4j              // 生成日志对象
@Log4j2             // 生成Log4j2日志对象
@Cleanup            // 自动资源管理
@SneakyThrows       // 隐藏受检异常
```

**示例：**

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Long id;
    private String name;
    private Integer age;
}
```

### 7.4 JUnit

**JUnit 4：**

```java
@Test               // 测试方法
@Before             // 每个测试前执行
@After              // 每个测试后执行
@BeforeClass        // 所有测试前执行一次
@AfterClass         // 所有测试后执行一次
@Ignore             // 忽略测试
@RunWith            // 指定运行器
@ParameterizedTest  // 参数化测试
```

**JUnit 5：**

```java
@Test               // 测试方法
@BeforeEach         // 每个测试前执行
@AfterEach          // 每个测试后执行
@BeforeAll          // 所有测试前执行一次
@AfterAll           // 所有测试后执行一次
@Disabled           // 禁用测试
@DisplayName        // 显示名称
@ParameterizedTest  // 参数化测试
@ValueSource        // 参数源
@CsvSource          // CSV参数源
@MethodSource       // 方法参数源
@RepeatedTest       // 重复测试
@Timeout            // 超时设置
```

### 7.5 Jackson

**JSON序列化/反序列化：**

```java
@JsonProperty("user_name")      // 属性映射
@JsonIgnore                     // 忽略属性
@JsonIgnoreProperties           // 忽略多个属性
@JsonFormat(pattern = "yyyy-MM-dd") // 格式化
@JsonInclude(Include.NON_NULL)  // 包含策略
@JsonSerialize                  // 自定义序列化
@JsonDeserialize                // 自定义反序列化
@JsonCreator                    // 指定构造函数
@JsonValue                      // 序列化为单个值
@JsonAlias                      // 别名
@JsonTypeInfo                   // 类型信息
```

### 7.6 Swagger/OpenAPI

**API文档注解：**

```java
@Api(tags = "用户管理")         // API分组
@ApiOperation("获取用户列表")   // 操作说明
@ApiParam("用户ID")             // 参数说明
@ApiModel("用户实体")           // 模型说明
@ApiModelProperty("用户名")     // 属性说明
@ApiResponse                    // 响应说明
@ApiResponses                   // 多个响应说明
```

**示例：**

```java
@RestController
@RequestMapping("/api/users")
@Api(tags = "用户管理")
public class UserController {

    @GetMapping("/{id}")
    @ApiOperation("根据ID获取用户")
    public User getUser(
        @PathVariable @ApiParam("用户ID") Long id) {
        return userService.findById(id);
    }
}
```

---

## 8. 最佳实践

### 8.1 合理选择保留策略

- **编译检查用 `SOURCE`**：如 `@Override`、`@SuppressWarnings`
- **运行时处理用 `RUNTIME`**：如 Spring、JPA 等框架注解
- **字节码工具处理用 `CLASS`**：默认策略，适合字节码增强工具

### 8.2 明确指定 @Target

限制注解的使用范围，避免误用。

```java
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface MyAnnotation {
}
```

### 8.3 提供有意义的默认值

```java
public @interface Cache {
    int timeout() default 3600;  // 默认1小时
    String key() default "";
    boolean enabled() default true;
}
```

### 8.4 使用组合注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Component
@Transactional
public @interface TransactionalService {
    String value() default "";
}
```

### 8.5 文档化注解

使用 `@Documented` 并添加详细的JavaDoc。

```java
/**
 * 标记方法需要缓存结果
 *
 * @author Your Name
 * @since 1.0
 */
@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Cacheable {
    /**
     * 缓存的键
     * @return 缓存键
     */
    String key();

    /**
     * 缓存超时时间（秒）
     * @return 超时时间
     */
    int timeout() default 3600;
}
```

### 8.6 避免过度使用注解

注解应该简化代码，而不是增加复杂性。如果逻辑复杂，考虑使用配置文件或代码实现。

**不好的做法：**

```java
@Cache(key = "user", timeout = 3600, enabled = true,
       evictOnUpdate = true, evictOnDelete = true,
       cacheManager = "redisCacheManager",
       keyGenerator = "customKeyGenerator",
       condition = "#id > 0",
       unless = "#result == null")
public User findUser(Long id) {
    // 注解过于复杂
}
```

**好的做法：**

```java
@Cacheable("user")
public User findUser(Long id) {
    // 简洁明了
}
```

### 8.7 注解命名规范

- 使用清晰、描述性的名称
- 遵循驼峰命名法
- 避免使用缩写
- 动词形式：`@Enable...`、`@Disable...`
- 名词形式：`@Service`、`@Component`

---

## 9. 实战示例

### 9.1 验证注解

```java
// 定义注解
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface NotNull {
    String message() default "Field cannot be null";
}

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Range {
    int min() default 0;
    int max() default Integer.MAX_VALUE;
    String message() default "Value out of range";
}

// 使用注解
public class User {
    @NotNull(message = "用户名不能为空")
    private String username;

    @Range(min = 18, max = 100, message = "年龄必须在18-100之间")
    private int age;

    // getters and setters
}

// 验证器
public class Validator {
    public static void validate(Object obj) throws IllegalAccessException {
        Class<?> clazz = obj.getClass();

        for (Field field : clazz.getDeclaredFields()) {
            field.setAccessible(true);

            // 检查 @NotNull
            if (field.isAnnotationPresent(NotNull.class)) {
                NotNull notNull = field.getAnnotation(NotNull.class);
                if (field.get(obj) == null) {
                    throw new IllegalArgumentException(notNull.message());
                }
            }

            // 检查 @Range
            if (field.isAnnotationPresent(Range.class)) {
                Range range = field.getAnnotation(Range.class);
                if (field.getType() == int.class) {
                    int value = field.getInt(obj);
                    if (value < range.min() || value > range.max()) {
                        throw new IllegalArgumentException(range.message());
                    }
                }
            }
        }
    }
}

// 测试
public class Main {
    public static void main(String[] args) {
        User user = new User();
        user.setUsername("John");
        user.setAge(25);

        try {
            Validator.validate(user);
            System.out.println("验证通过");
        } catch (Exception e) {
            System.out.println("验证失败: " + e.getMessage());
        }
    }
}
```

### 9.2 性能监控注解

```java
// 定义注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PerformanceMonitor {
    String value() default "";
}

// 使用AOP处理注解
@Aspect
@Component
public class PerformanceAspect {

    @Around("@annotation(monitor)")
    public Object monitorPerformance(ProceedingJoinPoint joinPoint,
                                    PerformanceMonitor monitor) throws Throwable {
        long startTime = System.currentTimeMillis();

        try {
            return joinPoint.proceed();
        } finally {
            long endTime = System.currentTimeMillis();
            String methodName = monitor.value().isEmpty()
                ? joinPoint.getSignature().getName()
                : monitor.value();

            System.out.println(methodName + " 执行时间: " +
                             (endTime - startTime) + "ms");
        }
    }
}

// 使用
@Service
public class UserService {

    @PerformanceMonitor("查询用户")
    public User findUser(Long id) {
        // 查询逻辑
        return new User();
    }
}
```

### 9.3 权限控制注解

```java
// 定义注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequiresPermission {
    String[] value();  // 需要的权限
    LogicalOperator operator() default LogicalOperator.AND;
}

enum LogicalOperator {
    AND, OR
}

// 使用AOP处理注解
@Aspect
@Component
public class PermissionAspect {

    @Autowired
    private PermissionService permissionService;

    @Before("@annotation(permission)")
    public void checkPermission(JoinPoint joinPoint, RequiresPermission permission) {
        String[] requiredPermissions = permission.value();
        LogicalOperator operator = permission.operator();

        boolean hasPermission = false;

        if (operator == LogicalOperator.AND) {
            // 需要所有权限
            hasPermission = permissionService.hasAllPermissions(requiredPermissions);
        } else {
            // 需要任一权限
            hasPermission = permissionService.hasAnyPermission(requiredPermissions);
        }

        if (!hasPermission) {
            throw new AccessDeniedException("权限不足");
        }
    }
}

// 使用
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping
    @RequiresPermission("user:read")
    public List<User> listUsers() {
        return userService.findAll();
    }

    @DeleteMapping("/{id}")
    @RequiresPermission(value = {"user:delete", "admin"}, operator = LogicalOperator.OR)
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

### 9.4 日志注解

```java
// 定义注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {
    String value() default "";  // 操作描述
    LogType type() default LogType.INFO;  // 日志类型
    boolean recordParams() default true;  // 是否记录参数
    boolean recordResult() default false;  // 是否记录返回值
}

enum LogType {
    INFO, WARN, ERROR, DEBUG
}

// 使用AOP处理注解
@Aspect
@Component
@Slf4j
public class LogAspect {

    @Around("@annotation(logAnnotation)")
    public Object logAround(ProceedingJoinPoint joinPoint, Log logAnnotation) throws Throwable {
        String methodName = joinPoint.getSignature().getName();
        String description = logAnnotation.value().isEmpty()
            ? methodName
            : logAnnotation.value();

        // 记录参数
        if (logAnnotation.recordParams()) {
            Object[] args = joinPoint.getArgs();
            log.info("方法 [{}] 开始执行，参数: {}", description, Arrays.toString(args));
        } else {
            log.info("方法 [{}] 开始执行", description);
        }

        long startTime = System.currentTimeMillis();
        Object result = null;

        try {
            result = joinPoint.proceed();

            // 记录返回值
            if (logAnnotation.recordResult()) {
                log.info("方法 [{}] 执行成功，返回值: {}, 耗时: {}ms",
                    description, result, System.currentTimeMillis() - startTime);
            } else {
                log.info("方法 [{}] 执行成功，耗时: {}ms",
                    description, System.currentTimeMillis() - startTime);
            }

            return result;
        } catch (Exception e) {
            log.error("方法 [{}] 执行失败，耗时: {}ms，异常: {}",
                description, System.currentTimeMillis() - startTime, e.getMessage());
            throw e;
        }
    }
}

// 使用
@Service
public class OrderService {

    @Log(value = "创建订单", type = LogType.INFO, recordParams = true, recordResult = true)
    public Order createOrder(OrderRequest request) {
        // 创建订单逻辑
        return new Order();
    }

    @Log("查询订单列表")
    public List<Order> listOrders(Long userId) {
        // 查询逻辑
        return orderRepository.findByUserId(userId);
    }
}
```
