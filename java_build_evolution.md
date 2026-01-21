# Java 构建演进史：从手工命令到 Maven，再到 Spring Boot

Java 项目的构建过程经历了从"完全手工"到"自动化工具化"的演进。从最早的 `javac/java/jar` 命令，到 Maven 的生命周期与依赖管理，再到 Spring Boot 的可执行 fat jar，整个流程变得越来越简单、标准化、可复用。

本文将带你回顾这一段发展历程，深入理解每个阶段的技术细节和演进动因。

---

## 1. 没有构建工具的时代：全靠手工命令

在 Maven 出现之前，Java 项目通常按下面的流程进行：

---

### 1.1 项目结构

```
project/
├── src/
│   └── com/example/app/Main.java
├── lib/
│   ├── log4j.jar
│   └── mysql.jar
├── bin/
└── dist/
```

---

### 1.2 编译：javac

```bash
javac \
  -encoding UTF-8 \
  -cp "lib/*" \
  -d bin \
  src/com/example/app/Main.java
```

**命令解释：**

| 参数 | 含义 |
|------|------|
| `javac` | Java 编译器 |
| `-encoding UTF-8` | 指定源文件编码为 UTF-8，避免中文乱码 |
| `-cp "lib/*"` | classpath，指定编译时依赖的 jar（所有 lib 目录下的 jar） |
| `-d bin` | 指定输出目录，把编译后的 class 文件输出到 bin |
| `src/.../Main.java` | 需要编译的源文件 |

**输出结果：**

```
bin/
└── com/example/app/Main.class
```

> 注意：第三方 jar 不会被复制到 bin，只是用于编译时引用。

---

### 1.3 运行：java

```bash
java \
  -cp "bin:lib/*" \
  com.example.app.Main
```

**命令解释：**

| 参数 | 含义 |
|------|------|
| `java` | Java 运行时 |
| `-cp "bin:lib/*"` | classpath：包含当前项目的 class 目录和第三方 jar |
| `com.example.app.Main` | 启动类（Main 方法所在类） |

---

### 1.4 打包成 jar 文件

先创建 manifest 文件 `manifest.txt`：

```
Main-Class: com.example.app.Main
```

然后打包：

```bash
jar cfm dist/app.jar manifest.txt -C bin .
```

**命令解释：**

| 参数 | 含义 |
|------|------|
| `jar` | 打包工具 |
| `c` | create：创建 jar |
| `f` | file：指定输出文件 |
| `m` | manifest：指定 manifest 文件 |
| `dist/app.jar` | 输出 jar 文件 |
| `manifest.txt` | 指定的 manifest |
| `-C bin .` | 进入 bin 目录，把 bin 目录下所有文件打包进去 |

---

### 1.5 部署方式一：外部依赖（非 fat jar）

部署目录结构：

```
app/
├── app.jar
└── lib/
    ├── log4j.jar
    └── mysql.jar
```

运行：

```bash
java -cp "app.jar:lib/*" com.example.app.Main
```

---

### 1.6 fat jar：把依赖合并到一个 jar

操作思路：

1. 解压 app.jar  
2. 解压所有 lib 下的 jar  
3. 再重新打包成一个 jar

运行：

```bash
java -jar app-fat.jar
```

---

### 这一阶段的痛点

- 依赖管理完全手工（下载、更新、冲突）
- 编译命令很长
- 打包过程繁琐
- 部署流程不统一
- 多人协作困难

---

## 2. Maven 出现：自动化、标准化、依赖管理

Maven 的出现解决了“手工构建”的大部分问题。

---

### 2.1 Maven 的核心思想

- **约定优于配置**：标准目录结构  
- **依赖管理**：自动下载、传递依赖、版本冲突解决  
- **生命周期**：compile/test/package/install/deploy  
- **插件机制**：通过插件实现编译、打包、测试等任务

---

### 2.2 Maven 的标准结构

```
my-app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   └── test/
│       └── java/
└── pom.xml
```

---

### 2.3 只需要一个命令

```bash
mvn package
```

**发生了什么：**

1. Maven 解析 `pom.xml`
2. 下载依赖到本地仓库（~/.m2/repository）
3. 编译（maven-compiler-plugin）
4. 测试（maven-surefire-plugin）
5. 打包（maven-jar-plugin）
6. 生成 jar 到 `target/`

---

### 2.4 普通 jar 的结果

普通 jar（不包含依赖）结构：

```
target/
└── my-app.jar
    ├── META-INF/
    │   └── MANIFEST.MF
    └── com/example/app/Main.class
```

---

### 2.5 方案 B：普通 jar + 外部依赖 + manifest Class-Path

在 `maven-jar-plugin` 中配置：

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jar-plugin</artifactId>
      <configuration>
        <archive>
          <manifest>
            <mainClass>com.example.app.Main</mainClass>
            <addClasspath>true</addClasspath>
            <classpathPrefix>lib/</classpathPrefix>
          </manifest>
        </archive>
      </configuration>
    </plugin>
  </plugins>
</build>
```

部署时：

```
app/
├── app.jar
└── lib/
    ├── log4j.jar
    └── mysql.jar
```

运行：

```bash
java -jar app.jar
```

---

### 2.6 fat jar（maven-shade-plugin）

使用 shade 插件把依赖合并进 jar：

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-shade-plugin</artifactId>
      <executions>
        <execution>
          <phase>package</phase>
          <goals><goal>shade</goal></goals>
          <configuration>
            <transformers>
              <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <mainClass>com.example.app.Main</mainClass>
              </transformer>
            </transformers>
            <finalName>app-fat</finalName>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

生成的 fat jar：

```
target/
└── app-fat.jar
    ├── META-INF/
    ├── com/example/app/Main.class
    └── org/apache/log4j/Logger.class
```

运行：

```bash
java -jar app-fat.jar
```

---

## 3. Spring Boot：可执行 jar 的标准化与自定义 classloader

Spring Boot 进一步简化部署：

- 默认生成可执行 jar
- 默认打成 fat jar
- 依赖 jar 以嵌套方式存放在 `BOOT-INF/lib/`
- 使用自定义 classloader 加载嵌套 jar

---

### 3.1 Spring Boot 的核心插件

Spring Boot 的可执行 fat jar 并不是 Maven 原生能力，而是通过 `spring-boot-maven-plugin` 实现的。

```xml
<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

当你执行：

```bash
mvn package
```

这个插件会在 **package 阶段**介入，对 Maven 生成的普通 jar 进行二次加工。

---

### 3.2 Spring Boot fat jar 的目录结构

```
app.jar
├── META-INF/
│   └── MANIFEST.MF
├── BOOT-INF/
│   ├── classes/
│   │   └── com/example/app/Main.class
│   └── lib/
│       ├── log4j.jar
│       └── mysql.jar
└── org/
    └── springframework/boot/loader/...
```

**说明：**

- `BOOT-INF/classes/`：项目自身的 class 文件
- `BOOT-INF/lib/`：所有第三方依赖 jar（完整的 jar 文件，不是解压合并）
- `org/springframework/boot/loader`：Spring Boot 内置启动器代码

---

### 3.3 MANIFEST.MF 中的关键配置

Spring Boot 会自动生成类似下面的 manifest：

```
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.example.app.Main
```

- `Main-Class`：真正被 JVM 执行的入口（Spring Boot 引导类）
- `Start-Class`：你的业务启动类

---

### 3.4 Spring Boot fat jar 的启动流程

当你执行：

```bash
java -jar app.jar
```

实际发生的流程是：

1. JVM 读取 `Main-Class`
2. 启动 `JarLauncher`
3. 创建自定义 `LaunchedURLClassLoader`
4. 扫描并加载：
   - `BOOT-INF/classes/`
   - `BOOT-INF/lib/*.jar`
5. 通过反射调用 `Start-Class` 的 `main` 方法

---

### 3.5 为什么 Spring Boot 选择"嵌套 jar"方案

Spring Boot 并没有像 `maven-shade-plugin` 那样把依赖 class 解压合并，而是选择嵌套 jar：

**优点：**
- 避免 class / 资源文件冲突
- 保留依赖 jar 的完整结构
- 更适合大型项目

**缺点：**
- 启动时 classloader 逻辑更复杂
- jar 体积仍然较大

---

### 3.6 Spring Boot fat jar vs Maven Shade

| 对比项 | Spring Boot | Shade |
|------|------------|-------|
| 依赖形式 | 嵌套 jar | class 合并 |
| 启动方式 | 自定义 classloader | JVM 默认 |
| 冲突风险 | 低 | 较高 |
| 可维护性 | 高 | 一般 |

---

> 可以理解为：
> **Spring Boot 在 jar 内部实现了一套"自己的类加载机制"**，
> 从而让一个 jar 文件具备完整应用的运行能力。

---

# 4. 总结：一张图看清楚演进

| 阶段 | 依赖管理 | 打包方式 | 运行方式 | 优点 | 缺点 |
|------|----------|----------|----------|------|------|
| 纯手工 | 手工下载 | 手工 jar | `java -cp` | 简单透明 | 太繁琐 |
| Maven | 自动下载、传递依赖 | 普通 jar / fat jar | `java -cp` 或 `java -jar` | 标准化、自动化 | 普通 jar 依赖仍需外部 |
| Spring Boot | 自动下载 | fat jar（嵌套 jar） | `java -jar` | 部署极简 | jar 体积大、依赖嵌套 |

---

# 5. 结语

从“手工编译、手工打包”到“自动化构建工具”，再到“Spring Boot 的可执行 fat jar”，Java 的构建体系逐步走向：

> **更标准、更自动、更适合生产部署**
