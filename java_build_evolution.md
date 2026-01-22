# Java 构建演进史：从手工命令到 Maven，再到 Spring Boot

​	Java 项目的构建过程经历了从"完全手工"到"自动化工具化"的演进。从最早的 `javac/java/jar` 命令，到 Maven 的生命周期与依赖管理，再到 SpringBoot 的可执行 fat jar，整个流程变得越来越简单、标准化、可复用。本文将带你回顾这一段发展历程，深入理解每个阶段的技术细节和演进动因。

## 1. 没有构建工具的时代：全靠手工命令

在 Maven、Gradle 等构建工具出现之前（大约 2000 年代初期），Java 开发者需要手动执行编译、打包、部署的每一个步骤。这个时代的特点是：

- **依赖管理**：手工下载 jar 包，放到项目目录
- **编译**：使用 `javac` 命令逐个编译源文件
- **打包**：使用 `jar` 命令手动打包
- **部署**：手动复制文件到服务器

虽然繁琐，但这种方式让我们能够深入理解 Java 项目构建的本质。

### 1.1 项目结构

```
project/
├── src/              # 源代码目录
│   └── com/example/app/Main.java
├── lib/              # 第三方依赖 jar
│   ├── log4j.jar
│   └── mysql.jar
├── bin/              # 编译输出目录（.class 文件）
└── dist/             # 打包输出目录（.jar 文件）
```

**目录说明：**

| 目录 | 作用 |
|------|------|
| `src/` | 存放 Java 源代码（.java 文件） |
| `lib/` | 存放项目依赖的第三方 jar 包 |
| `bin/` | 编译后的 class 文件输出目录 |
| `dist/` | 最终打包的 jar 文件输出目录 |

> 注意：这种目录结构并非强制标准，每个项目可能有不同的组织方式，这也是后来 Maven "约定优于配置" 思想的由来。

### 1.2 编译：javac

```bash
javac -encoding UTF-8 -cp "lib/*" -d bin src/com/example/app/Main.java
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
└── com/
    └── example/
        └── app/
            └── Main.class
```

> 注意：第三方 jar 不会被复制到 bin，只是用于编译时引用。

### 1.3 运行：java

编译完成后，需要通过 `java` 命令运行程序。

```bash
java -cp "bin:lib/*" com.example.app.Main
```

**命令解释：**

| 参数 | 含义 |
|------|------|
| `java` | Java 运行时 |
| `-cp "bin:lib/*"` | classpath：包含当前项目的 class 目录和第三方 jar |
| `com.example.app.Main` | 启动类（Main 方法所在类） |

**关键点：**

1. **classpath 必须同时包含**：
   - 项目自己的 class 文件目录（`bin`）
   - 所有依赖的 jar 包（`lib/*`）

2. **路径分隔符**：
   - Linux/Mac：使用冒号 `:` 分隔（如 `bin:lib/*`）
   - Windows：使用分号 `;` 分隔（如 `bin;lib/*`）

3. **启动类必须是全限定名**：
   - 不能写成 `Main`，必须是 `com.example.app.Main`
   - 必须包含 `public static void main(String[] args)` 方法

### 1.4 打包成 jar 文件

jar（Java Archive）是 Java 的标准打包格式，本质上是一个 zip 压缩文件。

**步骤一：创建 manifest 文件**

manifest 文件用于描述 jar 包的元信息，最重要的是指定启动类。

创建 `manifest.txt`：

```
Main-Class: com.example.app.Main
```

> 注意：
> 1. `Main-Class` 后面必须有冒号和空格
> 2. 文件末尾必须有一个空行（否则最后一行配置可能无效）

**步骤二：打包**

```bash
jar cfm dist/app.jar manifest.txt -C bin .
```

**命令解释：**

| 参数 | 含义 |
|------|------|
| `jar` | 打包工具 |
| `cfm` | create：创建 jar   file：指定输出文件   manifest：指定 manifest 文件 |
| `dist/app.jar` | 输出 jar 文件 |
| `manifest.txt` | 指定的 manifest |
| `-C bin .` | 进入 bin 目录，把 bin 目录下所有文件打包进去 |

**生成的 jar 内部结构：**

```
app.jar
├── META-INF/
│   └── MANIFEST.MF          # manifest 信息
└── com/
    └── example/
        └── app/
            └── Main.class   # 应用 class 文件
```

**关键点：**

- jar 文件只包含项目自己的 class 文件
- 不包含第三方依赖（log4j.jar、mysql.jar 等）
- 运行时仍需要通过 `-cp` 指定依赖

### 1.5 部署方式一：外部依赖（非 fat jar）

这种方式将应用 jar 和依赖 jar 分开部署，运行时通过 classpath 指定所有 jar 的位置。

**部署目录结构：**

```
app/
├── app.jar
└── lib/
    ├── log4j.jar
    └── mysql.jar
```

**运行方式一：手动指定 classpath**

```bash
java -cp "app.jar:lib/*" com.example.app.Main
```

**运行方式二：使用 manifest 的 Class-Path**

可以在打包时，将依赖路径写入 manifest 文件：

```
Main-Class: com.example.app.Main
Class-Path: lib/log4j.jar lib/mysql.jar
```

这样运行时只需：

```bash
java -jar app.jar
```

JVM 会自动根据 `Class-Path` 加载依赖。

**优缺点：**

| 优点 | 缺点 |
|------|------|
| jar 体积小 | 部署时需要同时复制 lib 目录 |
| 更新依赖时只需替换单个 jar | 目录结构必须保持一致 |
| 便于排查依赖问题 | 运行命令较长（不使用 manifest 时） |

### 1.6 部署方式二：把依赖合并到一个 jar（fat jar）

fat jar（也叫 uber jar）将所有依赖的 class 文件解压后，与应用 class 合并打包成一个独立的 jar。

**操作步骤：**

```bash
# 1. 创建临时目录
mkdir tmp

# 2. 解压应用 jar
cd tmp && jar xf ../dist/app.jar

# 3. 解压所有依赖 jar
jar xf ../lib/log4j.jar
jar xf ../lib/mysql.jar

# 4. 重新打包（确保 manifest 包含 Main-Class）
jar cfm ../dist/app-fat.jar ../manifest.txt .
```

**fat jar 内部结构：**

```
app-fat.jar
├── META-INF/
│   └── MANIFEST.MF
├── com/
│   └── example/
│       └── app/
│           └── Main.class
└── org/
    └── apache/
        └── log4j/
            └── Logger.class
            └── ...
```

所有依赖的 class 文件被"平铺"到 jar 根目录下，与应用 class 混在一起。

**运行：**

```bash
java -jar app-fat.jar
```

**优缺点：**

| 优点 | 缺点 |
|------|------|
| 单文件部署，简单方便 | jar 体积较大 |
| 不依赖外部目录结构 | 可能出现同名 class 冲突 |
| 运行命令简洁 | 资源文件（如配置）可能被覆盖 |

### 1.7 这一阶段的痛点

手工构建时代虽然让我们理解了构建的本质，但在实际项目中存在诸多问题：

**1. 依赖管理完全手工**
- 需要手动下载每个依赖 jar 包
- 版本更新时需要逐个替换
- 传递依赖需要手动解决（A 依赖 B，B 依赖 C，需要同时下载 B 和 C）
- 版本冲突难以发现和解决

**2. 编译命令冗长且易错**
- 每次编译都要写一长串参数
- 多个源文件需要逐个指定或使用通配符
- classpath 配置复杂，容易遗漏

**3. 打包过程繁琐**
- 需要手动创建 manifest 文件
- fat jar 的制作需要多个步骤
- 容易出现文件遗漏或覆盖

**4. 部署流程不统一**
- 不同项目的目录结构各不相同
- 部署脚本需要针对每个项目定制
- 环境迁移困难

**5. 多人协作困难**
- 依赖版本难以统一
- 构建脚本难以共享
- 新成员上手成本高

**6. 缺乏标准化**
- 没有统一的项目结构约定
- 没有统一的构建流程
- 每个团队都有自己的"最佳实践"

这些痛点催生了自动化构建工具的出现，Maven 就是在这样的背景下诞生的。

## 2. Maven 出现：自动化、标准化、依赖管理

Maven 于 2004 年发布，是 Apache 软件基金会的开源项目。它的出现彻底改变了 Java 项目的构建方式，解决了手工构建时代的大部分痛点。

**Maven 带来的核心变革：**

- **标准化项目结构**：所有 Maven 项目遵循相同的目录约定
- **自动依赖管理**：通过坐标（GAV）自动下载和管理依赖
- **统一构建流程**：通过生命周期和插件实现标准化构建
- **中央仓库**：Maven Central 提供海量开源库

### 2.1 Maven 的核心思想

**1. 约定优于配置（Convention over Configuration）**

Maven 定义了标准的项目结构，开发者只需遵循约定，无需大量配置：

- `src/main/java`：存放源代码
- `src/main/resources`：存放资源文件
- `src/test/java`：存放测试代码
- `target/`：构建输出目录

**2. 依赖管理**

通过 GAV 坐标（GroupId、ArtifactId、Version）唯一标识依赖：

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.20.0</version>
</dependency>
```

Maven 会自动：
- 从中央仓库下载依赖
- 解析传递依赖（A 依赖 B，B 依赖 C，自动下载 C）
- 处理版本冲突（就近原则、第一声明原则）

**3. 生命周期（Lifecycle）**

Maven 定义了三套独立的生命周期：

- **clean**：清理项目（删除 target 目录）
- **default**：构建项目（编译、测试、打包、部署）
- **site**：生成项目站点文档

default 生命周期的主要阶段：

| 阶段 | 说明 |
|------|------|
| `validate` | 验证项目是否正确 |
| `compile` | 编译源代码 |
| `test` | 运行单元测试 |
| `package` | 打包成 jar/war |
| `verify` | 运行集成测试 |
| `install` | 安装到本地仓库 |
| `deploy` | 部署到远程仓库 |

**4. 插件机制**

Maven 的所有功能都通过插件实现：

- `maven-compiler-plugin`：编译代码
- `maven-surefire-plugin`：运行测试
- `maven-jar-plugin`：打包 jar
- `maven-shade-plugin`：打包 fat jar

### 2.2 Maven 的标准结构

```
my-app/
├── src/
│   ├── main/
│   │   ├── java/              # 源代码
│   │   │   └── com/example/app/Main.java
│   │   └── resources/         # 资源文件（配置、静态文件等）
│   │       └── application.properties
│   └── test/
│       ├── java/              # 测试代码
│       │   └── com/example/app/MainTest.java
│       └── resources/         # 测试资源文件
└── pom.xml                    # Maven 项目配置文件
```

**目录说明：**

| 目录 | 作用 |
|------|------|
| `src/main/java` | 存放 Java 源代码 |
| `src/main/resources` | 存放配置文件、静态资源等 |
| `src/test/java` | 存放单元测试代码 |
| `src/test/resources` | 存放测试用的配置文件 |
| `target/` | 构建输出目录（编译后的 class、打包的 jar 等） |
| `pom.xml` | 项目对象模型，定义项目信息、依赖、插件等 |

### 2.3 一个简单的 pom.xml 示例

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <!-- 项目坐标 -->
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <!-- 项目信息 -->
    <name>My Application</name>

    <!-- 依赖管理 -->
    <dependencies>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.20.0</version>
        </dependency>
    </dependencies>

    <!-- 构建配置 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 2.4 只需要一个命令

有了 pom.xml 后，构建项目只需一个命令：

```bash
mvn package
```

**执行流程：**

1. **解析 pom.xml**：读取项目配置和依赖信息
2. **下载依赖**：从 Maven Central 下载依赖到本地仓库（`~/.m2/repository`）
3. **编译**：使用 `maven-compiler-plugin` 编译 `src/main/java` 到 `target/classes`
4. **测试**：使用 `maven-surefire-plugin` 运行 `src/test/java` 中的测试
5. **打包**：使用 `maven-jar-plugin` 将 `target/classes` 打包成 jar
6. **输出**：生成 jar 到 `target/` 目录

**常用 Maven 命令：**

| 命令 | 说明 |
|------|------|
| `mvn clean` | 清理 target 目录 |
| `mvn compile` | 编译源代码 |
| `mvn test` | 运行测试 |
| `mvn package` | 打包成 jar/war |
| `mvn install` | 安装到本地仓库 |
| `mvn clean package` | 清理后重新打包 |

### 2.5 普通 jar 的结果

Maven 默认生成的是普通 jar（不包含依赖）。

**输出结构：**

```
target/
├── classes/                   # 编译后的 class 文件
│   └── com/example/app/Main.class
├── test-classes/              # 测试 class 文件
└── my-app-1.0.0.jar          # 打包的 jar
```

**jar 内部结构：**

```
my-app-1.0.0.jar
├── META-INF/
│   └── MANIFEST.MF
└── com/
    └── example/
        └── app/
            └── Main.class
```

**运行方式：**

```bash
# 需要手动指定 classpath
java -cp "target/my-app-1.0.0.jar:lib/*" com.example.app.Main
```

> 注意：普通 jar 不包含依赖，运行时仍需要外部 lib 目录。

### 2.6 方案 A：普通 jar + 外部依赖 + manifest Class-Path

这种方式通过配置 `maven-jar-plugin`，在 manifest 中自动生成 Class-Path，简化运行命令。

**配置 pom.xml：**

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
    <!-- 使用 maven-dependency-plugin 复制依赖到 lib 目录 -->
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-dependency-plugin</artifactId>
      <executions>
        <execution>
          <phase>package</phase>
          <goals>
            <goal>copy-dependencies</goal>
          </goals>
          <configuration>
            <outputDirectory>${project.build.directory}/lib</outputDirectory>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

**生成的 MANIFEST.MF：**

```
Main-Class: com.example.app.Main
Class-Path: lib/log4j-core-2.20.0.jar lib/log4j-api-2.20.0.jar
```

**部署结构：**

```
target/
├── my-app-1.0.0.jar
└── lib/
    ├── log4j-core-2.20.0.jar
    └── log4j-api-2.20.0.jar
```

**运行：**

```bash
java -jar target/my-app-1.0.0.jar
```

**优缺点：**

| 优点 | 缺点 |
|------|------|
| 运行命令简洁（java -jar） | 部署时需要保持目录结构 |
| jar 体积小，便于更新 | 依赖文件较多 |
| 依赖清晰可见 | 需要额外配置插件 |

### 2.7 方案 B：fat jar（maven-shade-plugin）

使用 `maven-shade-plugin` 将所有依赖的 class 文件解压合并到一个 jar 中。

**配置 pom.xml：**

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-shade-plugin</artifactId>
      <version>3.5.0</version>
      <executions>
        <execution>
          <phase>package</phase>
          <goals>
            <goal>shade</goal>
          </goals>
          <configuration>
            <transformers>
              <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <mainClass>com.example.app.Main</mainClass>
              </transformer>
            </transformers>
            <finalName>my-app-fat</finalName>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

**生成结果：**

```
target/
├── my-app-1.0.0.jar          # 原始 jar（不含依赖）
└── my-app-fat.jar            # fat jar（包含所有依赖）
```

**fat jar 内部结构：**

```
my-app-fat.jar
├── META-INF/
│   └── MANIFEST.MF
├── com/
│   └── example/
│       └── app/
│           └── Main.class    # 应用 class
└── org/
    └── apache/
        └── log4j/
            └── Logger.class  # 依赖 class（已解压合并）
```

**运行：**

```bash
java -jar target/my-app-fat.jar
```

## 3. Spring Boot：可执行 jar 的标准化与自定义 classloader

Spring Boot 于 2014 年发布，是 Spring 生态系统的重要组成部分。它的核心理念是"约定优于配置"，进一步简化了 Spring 应用的开发和部署。

**Spring Boot 在构建和部署方面的创新：**

- **开箱即用**：内嵌 Web 容器（Tomcat、Jetty），无需外部部署
- **可执行 jar**：默认生成可直接运行的 fat jar
- **嵌套 jar 方案**：依赖以完整 jar 形式存放在 `BOOT-INF/lib/`
- **自定义类加载器**：实现了 `LaunchedURLClassLoader` 加载嵌套 jar
- **统一打包方式**：所有 Spring Boot 应用使用相同的打包结构

### 3.1 Spring Boot 的核心插件

Spring Boot 的可执行 fat jar 并不是 Maven 原生能力，而是通过 `spring-boot-maven-plugin` 实现的。

**在 pom.xml 中配置：**

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <version>3.2.0</version>
      <executions>
        <execution>
          <goals>
            <goal>repackage</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

**插件的工作流程：**

当你执行：

```bash
mvn package
```

插件会在 **package 阶段**执行以下操作：

1. **Maven 先生成普通 jar**：`my-app-1.0.0.jar`（只包含项目 class）
2. **插件重新打包（repackage）**：
   - 将原始 jar 重命名为 `my-app-1.0.0.jar.original`
   - 创建新的 Spring Boot fat jar：`my-app-1.0.0.jar`
   - 将项目 class 放入 `BOOT-INF/classes/`
   - 将所有依赖 jar 放入 `BOOT-INF/lib/`
   - 添加 Spring Boot Loader 代码到 jar 根目录

**生成结果：**

```
target/
├── my-app-1.0.0.jar           # Spring Boot fat jar（可执行）
└── my-app-1.0.0.jar.original  # 原始 jar（不可执行）
```

### 3.2 Spring Boot fat jar 的目录结构

Spring Boot fat jar 采用了独特的"嵌套 jar"结构，与 maven-shade-plugin 的"class 合并"方案完全不同。

```
my-app-1.0.0.jar
├── META-INF/
│   └── MANIFEST.MF                    # 启动配置
├── BOOT-INF/
│   ├── classes/                       # 项目自身的 class 和资源文件
│   │   ├── com/
│   │   │   └── example/
│   │   │       └── app/
│   │   │           └── Application.class
│   │   └── application.properties
│   ├── lib/                           # 所有依赖 jar（完整的 jar 文件）
│   │   ├── spring-boot-3.2.0.jar
│   │   ├── spring-core-6.1.0.jar
│   │   ├── log4j-core-2.20.0.jar
│   │   └── ...
│   └── classpath.idx                  # classpath 索引文件
└── org/
    └── springframework/
        └── boot/
            └── loader/                # Spring Boot Loader 启动器
                ├── JarLauncher.class
                ├── LaunchedURLClassLoader.class
                └── ...
```

**关键目录说明：**

| 目录/文件 | 作用 |
|----------|------|
| `BOOT-INF/classes/` | 项目自身编译后的 class 文件和资源文件 |
| `BOOT-INF/lib/` | 所有第三方依赖 jar（完整 jar，不解压） |
| `BOOT-INF/classpath.idx` | 记录 classpath 顺序，优化类加载性能 |
| `org/springframework/boot/loader/` | Spring Boot 自定义的类加载器代码 |
| `META-INF/MANIFEST.MF` | 指定启动类和 classpath |

**与 maven-shade 的关键区别：**

- **Shade**：将所有 jar 解压，class 文件平铺合并
- **Spring Boot**：依赖 jar 保持完整，以嵌套方式存放

### 3.3 MANIFEST.MF 中的关键配置

Spring Boot 会自动生成包含特殊配置的 manifest 文件：

```
Manifest-Version: 1.0
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.example.app.Application
Spring-Boot-Version: 3.2.0
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Classpath-Index: BOOT-INF/classpath.idx
```

**配置项说明：**

| 配置项 | 说明 |
|--------|------|
| `Main-Class` | JVM 真正执行的入口类（Spring Boot Loader） |
| `Start-Class` | 你的业务启动类（包含 @SpringBootApplication） |
| `Spring-Boot-Classes` | 项目 class 文件的位置 |
| `Spring-Boot-Lib` | 依赖 jar 的位置 |
| `Spring-Boot-Classpath-Index` | classpath 索引文件位置 |

**为什么需要两个 Main-Class？**

- `Main-Class`：JVM 启动时执行的类（`JarLauncher`）
- `Start-Class`：实际的应用启动类（你的 `Application.main()`）

JVM 无法直接加载嵌套在 jar 内部的 jar，所以需要 `JarLauncher` 作为中间层。

### 3.4 Spring Boot fat jar 的启动流程

当你执行：

```bash
java -jar my-app-1.0.0.jar
```

实际发生的流程是：

**1. JVM 启动 JarLauncher**

```
JVM 读取 MANIFEST.MF
→ 找到 Main-Class: org.springframework.boot.loader.JarLauncher
→ 执行 JarLauncher.main()
```

**2. JarLauncher 创建自定义 ClassLoader**

```java
// 伪代码示例
LaunchedURLClassLoader classLoader = new LaunchedURLClassLoader(
    new URL[] {
        "jar:file:/path/to/app.jar!/BOOT-INF/classes/",
        "jar:file:/path/to/app.jar!/BOOT-INF/lib/spring-boot.jar!/",
        "jar:file:/path/to/app.jar!/BOOT-INF/lib/spring-core.jar!/",
        // ... 所有依赖 jar
    }
);
```

**3. 扫描并加载 classpath**

- 读取 `BOOT-INF/classpath.idx` 获取依赖顺序
- 将 `BOOT-INF/classes/` 添加到 classpath
- 将 `BOOT-INF/lib/*.jar` 逐个添加到 classpath

**4. 反射调用业务启动类**

```java
// 伪代码示例
Class<?> startClass = classLoader.loadClass("com.example.app.Application");
Method mainMethod = startClass.getMethod("main", String[].class);
mainMethod.invoke(null, new Object[] { args });
```

**5. 应用正常启动**

- 执行 `Application.main()`
- 启动 Spring 容器
- 启动内嵌 Web 容器（Tomcat/Jetty）

**完整流程图：**

```
java -jar app.jar
    ↓
JVM 读取 Main-Class
    ↓
启动 JarLauncher
    ↓
创建 LaunchedURLClassLoader
    ↓
加载 BOOT-INF/classes/
加载 BOOT-INF/lib/*.jar
    ↓
反射调用 Start-Class.main()
    ↓
应用启动
```

### 3.5 为什么 Spring Boot 选择"嵌套 jar"方案

Spring Boot 并没有像 `maven-shade-plugin` 那样把依赖 class 解压合并，而是选择嵌套 jar 方案。

**技术原因：**

1. **避免文件冲突**
   - 多个 jar 可能包含同名的 class 文件
   - 多个 jar 可能包含同名的资源文件（如 `META-INF/services/`）
   - 嵌套 jar 保持了每个依赖的独立性

2. **保留 jar 元信息**
   - 每个 jar 的 MANIFEST.MF 得以保留
   - 便于追踪依赖版本和来源
   - 便于调试和问题排查

3. **支持 jar 签名**
   - 依赖 jar 的数字签名得以保留
   - 不会因为解压合并而破坏签名

4. **更好的类隔离**
   - 每个 jar 保持独立的命名空间
   - 减少类加载冲突的可能性

> **核心理解：**
>
> Spring Boot 在 jar 内部实现了一套"自己的类加载机制"，通过 `JarLauncher` 和 `LaunchedURLClassLoader`，让 JVM 能够加载嵌套在 jar 内部的 jar 文件，从而实现了真正的"单文件部署"。
