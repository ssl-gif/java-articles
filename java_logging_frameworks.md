# Java 日志框架学习笔记

## 一、Java 日志框架发展历史

Java 日志框架经历了从简单到复杂、从内置到统一接口的演变：

### 1. JDK 内置日志（`java.util.logging`）
- **出现时间**：Java 1.4（2002）
- **特点**：
  - 内置于 JDK，无需额外依赖
  - 支持日志级别：`SEVERE`, `WARNING`, `INFO`, `CONFIG`, `FINE`, `FINER`, `FINEST`
  - 配置相对复杂，功能有限
- **问题**：
  - API 不够灵活
  - 扩展性差，格式化和输出渠道不够丰富

### 2. Apache Commons Logging（JCL）
- **出现时间**：2001 左右
- **特点**：
  - 提供日志 **抽象层**（Facade），可以在底层替换日志实现
  - 默认适配 Log4j 或 JDK Logging
- **问题**：
  - “类加载器问题”在复杂环境（如 Web 容器）下容易出错
  - 设计过于宽松，运行时容易切换失败

### 3. Log4j 1.x
- **出现时间**：2001
- **特点**：
  - 功能丰富，灵活配置（XML / properties）
  - 支持多种 Appender（控制台、文件、数据库等）
  - 日志级别：`DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL`
- **问题**：
  - 不支持异步日志（1.x 版本）
  - 配置复杂
  - 安全问题：Log4Shell 漏洞（Log4j2 修复）

### 4. SLF4J（Simple Logging Facade for Java）
- **出现时间**：2005
- **特点**：
  - **日志门面**（Facade），不提供实现，只提供统一 API
  - 支持绑定不同实现：Logback、Log4j、JDK Logging 等
  - 可以在编译时与具体日志实现解耦
- **常用绑定**：
  - `slf4j-log4j12` → Log4j 1.x
  - `log4j-slf4j-impl` → Log4j2
  - `logback-classic` → Logback

### 5. Logback
- **出现时间**：2006
- **特点**：
  - 由 Log4j 创始人设计，作为 SLF4J 的原生实现
  - 性能高，配置灵活，支持异步日志
  - 默认提供 XML 配置和压缩轮转日志

### 6. Log4j 2
- **出现时间**：2014
- **特点**：
  - 重写自 Log4j 1.x，解决性能和异步日志问题
  - 支持异步日志（AsyncAppender / LMAX Disruptor）
  - 支持插件化配置（XML, JSON, YAML, properties）
- **桥接包**：
  - `log4j-to-slf4j`：把 Log4j API 调用转到 SLF4J
  - `log4j-1.2-api`：兼容 Log4j 1.x 调用

---

## 二、各日志框架使用方法

### 1. JDK 内置日志（`java.util.logging`）
```java
import java.util.logging.Logger;
import java.util.logging.Level;

public class Main {
    private static final Logger logger = Logger.getLogger(Main.class.getName());

    public static void main(String[] args) {
        logger.severe("严重错误");
        logger.warning("警告");
        logger.info("信息");
        logger.fine("调试信息");
    }
}
```
- 配置文件：`logging.properties`
- 可配置 Handler（ConsoleHandler, FileHandler）和 Formatter

---

### 2. Log4j 1.x
```java
import org.apache.log4j.Logger;

public class Main {
    private static final Logger logger = Logger.getLogger(Main.class);

    public static void main(String[] args) {
        logger.debug("调试");
        logger.info("信息");
        logger.warn("警告");
        logger.error("错误");
        logger.fatal("致命错误");
    }
}
```
- 配置文件：`log4j.properties` 或 `log4j.xml`
- Appender 可写文件、控制台、邮件等

---

### 3. SLF4J + Logback
**依赖（Maven）**：
```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.4.11</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.7</version>
</dependency>
```
**使用方法**：
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Main {
    private static final Logger logger = LoggerFactory.getLogger(Main.class);

    public static void main(String[] args) {
        logger.debug("调试信息");
        logger.info("普通信息");
        logger.warn("警告");
        logger.error("错误");
    }
}
```
- 配置文件：`logback.xml`
- 支持异步日志、滚动日志等

---

### 4. Log4j 2 + SLF4J
**依赖（Maven）**：
```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.20.0</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.20.0</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.20.0</version>
</dependency>
```
**使用 SLF4J API**：
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Main {
    private static final Logger logger = LoggerFactory.getLogger(Main.class);

    public static void main(String[] args) {
        logger.info("使用 SLF4J API，底层 Log4j2 输出");
    }
}
```
- 配置文件：`log4j2.xml`
- 支持异步日志、插件化、JSON/YAML 配置

---

### 5. 框架选择指南
| 框架         | 类型           | 特点                              | 使用场景                                  |
|--------------|----------------|-----------------------------------|------------------------------------------|
| java.util.logging | 内置           | 简单，无依赖                        | 小型项目或快速原型                          |
| Log4j 1.x      | 实现           | 功能丰富，配置灵活                   | 旧项目                                    |
| JCL            | 门面           | 可切换底层实现，但类加载复杂         | 需要兼容老系统日志                         |
| SLF4J + Logback| 门面 + 实现     | 高性能，配置灵活，异步日志支持        | 新项目，推荐                              |
| SLF4J + Log4j2 | 门面 + 实现     | 高性能，异步日志，插件化，安全性高    | 新项目，尤其需要 Log4j2 功能和性能        |

---

## 三、SLF4J 桥接包、绑定包及原理

### 1. SLF4J 原理
- **API**：统一接口，如 `LoggerFactory.getLogger()`  
- **绑定（Binding）**：将 SLF4J API 调用绑定到具体日志实现  
- **桥接（Bridge）**：将第三方库不同日志调用重定向到 SLF4J  

### 2. 常用绑定包
| 绑定包                        | 作用                        |
|--------------------------------|----------------------------|
| `logback-classic`             | SLF4J → Logback（原生实现） |
| `log4j-slf4j-impl`            | SLF4J → Log4j2             |
| `slf4j-log4j12`                | SLF4J → Log4j1             |

### 3. 常用桥接包
| 桥接包                        | 作用                                                      |
|--------------------------------|----------------------------------------------------------|
| `jul-to-slf4j`                | 将 JDK Logging 调用转到 SLF4J                              |
| `log4j-over-slf4j`            | 将 Log4j 1.x 调用转到 SLF4J                                 |
| `log4j-to-slf4j`               | 将 Log4j2 API 调用转到 SLF4J                                |

**使用场景**：统一项目日志输出，第三方库使用不同日志框架时，通过桥接包重定向到统一日志实现。

---

### 4. 桥接包使用示例
```java
import java.util.logging.LogManager;
import org.slf4j.bridge.SLF4JBridgeHandler;

public class Main {
    public static void main(String[] args) {
        // 重定向 java.util.logging 到 SLF4J
        SLF4JBridgeHandler.removeHandlersForRootLogger();
        SLF4JBridgeHandler.install();

        java.util.logging.Logger julLogger = java.util.logging.Logger.getLogger("test");
        julLogger.info("JUL 日志，通过 SLF4J 输出");

        org.slf4j.Logger slf4jLogger = org.slf4j.LoggerFactory.getLogger(Main.class);
        slf4jLogger.info("SLF4J 日志");
    }
}
```

### 5. SLF4J 日志调用流程
```
第三方库日志 API
 ┌─────────────┐
 │   Log4j 1   │  ──┐
 └─────────────┘    │
                    ▼
                ┌─────────────┐
                │ SLF4J 桥接   │
                │ log4j-over- │
                │  slf4j      │
                └─────────────┘
                    │
 ┌─────────────┐    │
 │ java.util   │    │
 │ logging     │ ──┘
 └─────────────┘
                    │
                    ▼
                ┌─────────────┐
                │ SLF4J API    │
                └─────────────┘
                    │
                    ▼
                ┌─────────────┐
                │ 具体实现     │
                │ Logback/    │
                │ Log4j2      │
                └─────────────┘
```
- 所有第三方日志调用最终统一输出到项目选择的底层日志  
- 核心机制：**桥接包 + SLF4J API + 绑定实现**

---

### ✅ 小结
1. **历史演进**：JUL → Log4j1 → JCL → SLF4J → Logback / Log4j2  
2. **SLF4J 核心**：门面 + 绑定 + 桥接  
3. **项目实践**：
   - 新项目推荐：`SLF4J + Logback` 或 `SLF4J + Log4j2`
   - 第三方库使用不同日志时，加桥接包统一输出  
4. **性能 & 异步日志**：Logback 和 Log4j2 都支持异步日志，适合高性能需求  

