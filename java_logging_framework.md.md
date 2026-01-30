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
