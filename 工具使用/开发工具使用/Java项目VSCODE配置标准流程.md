# Java 项目 IDE 配置标准流程

> **作者**: erik.zhou  
> **适用场景**: Java Maven 项目的 IDE 级别配置  
> **目标**: 统一项目的 JDK 和 Maven 环境，实现开箱即用

---

## 📋 配置概述

本文档提供了一套标准化的 Java 项目 IDE 配置流程，配置完成后：

- ✅ 项目自动使用指定的 JDK 版本
- ✅ 项目自动使用指定的 Maven 配置
- ✅ 支持一键编译、测试、打包
- ✅ 支持断点调试
- ✅ 配置仅对当前项目生效，不影响其他项目

---

## 🎯 配置前准备

### 1. 确认 JDK 路径

```bash
# macOS 查看已安装的 JDK
/usr/libexec/java_home -V

# 输出示例：
# 17.0.16 (arm64) "Homebrew" - "OpenJDK 17.0.16" /opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home
# 11.0.28 (arm64) "Microsoft" - "OpenJDK 11.0.28" /Users/erik.zhou/Library/Java/JavaVirtualMachines/ms-11.0.28/Contents/Home
```

记录你要使用的 JDK 路径，例如：
```
/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home
```

### 2. 确认 Maven 路径

```bash
# 查看 Maven 安装路径
which mvn

# 输出示例：
# /Users/erik.zhou/Desktop/www/project/config/maven/bin/mvn
```

记录以下信息：
- **Maven 主目录**: `/Users/erik.zhou/Desktop/www/project/config/maven`
- **Maven 可执行文件**: `/Users/erik.zhou/Desktop/www/project/config/maven/bin/mvn`
- **settings.xml 路径**: `/Users/erik.zhou/Desktop/www/project/config/maven/settings.xml`
- **本地仓库路径**: `/Users/erik.zhou/Desktop/www/project/config/maven/repository`

---

## 🎨 选择配置方式

本文档提供两种配置方式，你可以根据使用的编辑器选择：

### 方式 1：VSCode/通用 IDE 配置（推荐用于 VSCode）
- 配置目录：`.vscode/`
- 适用于：VSCode、Cursor 等基于 VSCode 的编辑器
- 特点：通用性强，配置直观

### 方式 2：Kiro 配置（推荐用于 Kiro）
- 配置目录：`.kiro/settings/`
- 适用于：Kiro 编辑器
- 特点：支持变量引用，一处修改全局生效

**💡 提示**：两种配置可以同时存在，互不影响。

---

## 📁 第一步：创建配置目录

### VSCode 配置

在项目根目录下创建 `.vscode` 目录：

```bash
cd /path/to/your/project
mkdir -p .vscode
```

### Kiro 配置

在项目根目录下创建 `.kiro/settings` 目录：

```bash
cd /path/to/your/project
mkdir -p .kiro/settings
```

---

## ⚙️ 第二步：创建配置文件

### 🔵 VSCode 配置方式

#### 2.1 创建 `.vscode/settings.json`

这是项目的核心配置文件，定义了 Java 和 Maven 环境。

```json
{
  "java.configuration.runtimes": [
    {
      "name": "JavaSE-17",
      "path": "/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home",
      "default": true
    }
  ],
  "java.home": "/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home",
  "java.jdt.ls.java.home": "/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home",
  "maven.executable.path": "/Users/erik.zhou/Desktop/www/project/config/maven/bin/mvn",
  "maven.settingsFile": "/Users/erik.zhou/Desktop/www/project/config/maven/settings.xml",
  "maven.localRepository": "/Users/erik.zhou/Desktop/www/project/config/maven/repository",
  "maven.terminal.useJavaHome": true,
  "maven.terminal.customEnv": [
    {
      "environmentVariable": "JAVA_HOME",
      "value": "/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home"
    }
  ],
  "java.import.maven.enabled": true,
  "java.configuration.maven.userSettings": "/Users/erik.zhou/Desktop/www/project/config/maven/settings.xml",
  "java.project.sourcePaths": [
    "src/main/java"
  ],
  "java.project.outputPath": "target/classes",
  "java.test.config": {
    "vmArgs": [
      "-Xmx2048m"
    ]
  },
  "files.exclude": {
    "**/.git": true,
    "**/.DS_Store": true,
    "**/target": false
  }
}
```

**配置说明**：
- `java.configuration.runtimes`: 配置项目使用的 JDK 版本
- `maven.executable.path`: Maven 可执行文件的完整路径
- `maven.settingsFile`: Maven settings.xml 配置文件路径
- `maven.localRepository`: Maven 本地仓库路径
- `java.test.config.vmArgs`: 测试运行时的 JVM 参数

**⚠️ 需要替换的路径**：
1. 所有 JDK 路径：替换为你的 JDK 安装路径
2. 所有 Maven 路径：替换为你的 Maven 配置路径
3. `JavaSE-17` 中的版本号：根据你的 JDK 版本修改（如 JavaSE-11、JavaSE-21）

---

### 2.2 创建 `.vscode/launch.json`

这是调试配置文件，用于运行和调试 Java 程序。

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "java",
      "name": "Debug (Launch) - Current File",
      "request": "launch",
      "mainClass": "${file}",
      "javaHome": "/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home",
      "vmArgs": "-Xmx2048m"
    },
    {
      "type": "java",
      "name": "Debug Test - Current Class",
      "request": "launch",
      "mainClass": "",
      "projectName": "your-project-name",
      "javaHome": "/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home",
      "vmArgs": "-Xmx2048m",
      "classPaths": [
        "${workspaceFolder}/target/test-classes",
        "${workspaceFolder}/target/classes"
      ]
    }
  ]
}
```

**配置说明**：
- `Debug (Launch) - Current File`: 调试当前打开的 Java 文件
- `Debug Test - Current Class`: 调试当前测试类

**⚠️ 需要替换的内容**：
1. 所有 `javaHome` 路径：替换为你的 JDK 路径
2. `projectName`: 替换为你的项目名称（pom.xml 中的 artifactId）

---

### 2.3 创建 `.vscode/tasks.json`

这是任务配置文件，用于快速执行 Maven 命令。

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Maven: Clean",
      "type": "shell",
      "command": "/Users/erik.zhou/Desktop/www/project/config/maven/bin/mvn",
      "args": [
        "clean",
        "-s",
        "/Users/erik.zhou/Desktop/www/project/config/maven/settings.xml"
      ],
      "options": {
        "cwd": "${workspaceFolder}",
        "env": {
          "JAVA_HOME": "/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home"
        }
      },
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared",
        "showReuseMessage": false,
        "clear": false
      },
      "problemMatcher": []
    },
    {
      "label": "Maven: Compile",
      "type": "shell",
      "command": "/Users/erik.zhou/Desktop/www/project/config/maven/bin/mvn",
      "args": [
        "compile",
        "-s",
        "/Users/erik.zhou/Desktop/www/project/config/maven/settings.xml"
      ],
      "options": {
        "cwd": "${workspaceFolder}",
        "env": {
          "JAVA_HOME": "/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home"
        }
      },
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared",
        "showReuseMessage": false,
        "clear": false
      },
      "problemMatcher": ["$maven"]
    },
    {
      "label": "Maven: Test",
      "type": "shell",
      "command": "/Users/erik.zhou/Desktop/www/project/config/maven/bin/mvn",
      "args": [
        "test",
        "-s",
        "/Users/erik.zhou/Desktop/www/project/config/maven/settings.xml"
      ],
      "options": {
        "cwd": "${workspaceFolder}",
        "env": {
          "JAVA_HOME": "/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home"
        }
      },
      "group": "test",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared",
        "showReuseMessage": false,
        "clear": false
      },
      "problemMatcher": ["$maven"]
    },
    {
      "label": "Maven: Clean Install",
      "type": "shell",
      "command": "/Users/erik.zhou/Desktop/www/project/config/maven/bin/mvn",
      "args": [
        "clean",
        "install",
        "-s",
        "/Users/erik.zhou/Desktop/www/project/config/maven/settings.xml"
      ],
      "options": {
        "cwd": "${workspaceFolder}",
        "env": {
          "JAVA_HOME": "/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home"
        }
      },
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared",
        "showReuseMessage": false,
        "clear": false
      },
      "problemMatcher": ["$maven"]
    },
    {
      "label": "Maven: Package (Skip Tests)",
      "type": "shell",
      "command": "/Users/erik.zhou/Desktop/www/project/config/maven/bin/mvn",
      "args": [
        "package",
        "-DskipTests",
        "-s",
        "/Users/erik.zhou/Desktop/www/project/config/maven/settings.xml"
      ],
      "options": {
        "cwd": "${workspaceFolder}",
        "env": {
          "JAVA_HOME": "/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home"
        }
      },
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared",
        "showReuseMessage": false,
        "clear": false
      },
      "problemMatcher": ["$maven"]
    },
    {
      "label": "验证 Java 和 Maven 环境",
      "type": "shell",
      "command": "echo '=== Java 版本 ===' && /opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home/bin/java -version && echo '\n=== Maven 版本 ===' && /Users/erik.zhou/Desktop/www/project/config/maven/bin/mvn -version",
      "options": {
        "cwd": "${workspaceFolder}",
        "env": {
          "JAVA_HOME": "/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home"
        }
      },
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": true,
        "panel": "shared",
        "showReuseMessage": false,
        "clear": true
      },
      "problemMatcher": []
    }
  ]
}
```

**配置说明**：
- `Maven: Clean`: 清理项目
- `Maven: Compile`: 编译项目（默认构建任务，可用 `Cmd+Shift+B` 快捷键）
- `Maven: Test`: 运行所有测试
- `Maven: Clean Install`: 清理并安装到本地仓库
- `Maven: Package (Skip Tests)`: 打包（跳过测试）
- `验证 Java 和 Maven 环境`: 检查配置是否正确

**⚠️ 需要替换的路径**：
1. 所有 `command` 中的 Maven 路径
2. 所有 `args` 中的 settings.xml 路径
3. 所有 `env.JAVA_HOME` 中的 JDK 路径
4. 验证任务中的 Java 和 Maven 完整路径

---

### 🟢 Kiro 配置方式

Kiro 配置采用**变量引用**机制，只需配置一个核心文件，其他配置自动引用。

#### 2.1 创建 `.kiro/settings/java.json`（核心配置）

这是 Kiro 的核心配置文件，所有其他配置都会引用这里的变量。

```json
{
  "java.jdkPath": "/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home",
  "java.version": "17",
  "maven.home": "/Users/erik.zhou/Desktop/www/project/config/maven",
  "maven.settings": "/Users/erik.zhou/Desktop/www/project/config/maven/settings.xml",
  "maven.localRepository": "/Users/erik.zhou/Desktop/www/project/config/maven/repository"
}
```

**配置说明**：
- `java.jdkPath`: JDK 安装路径
- `java.version`: JDK 版本号
- `maven.home`: Maven 主目录（不包含 bin）
- `maven.settings`: Maven settings.xml 配置文件路径
- `maven.localRepository`: Maven 本地仓库路径

**⚠️ 需要替换的路径**：
1. `java.jdkPath`: 替换为你的 JDK 路径
2. `maven.home`: 替换为你的 Maven 主目录
3. `maven.settings`: 替换为你的 settings.xml 路径
4. `maven.localRepository`: 替换为你的本地仓库路径

---

#### 2.2 创建 `.kiro/settings/launch.json`（运行和调试配置）

这个文件使用 `${config:xxx}` 语法引用 `java.json` 中的配置。

```json
{
  "version": "1.0.0",
  "configurations": [
    {
      "type": "java",
      "name": "Debug: Current Test Class",
      "request": "launch",
      "mainClass": "${file}",
      "projectName": "your-project-name",
      "jdkPath": "${config:java.jdkPath}",
      "vmArgs": "-Xmx2048m",
      "args": "",
      "env": {},
      "console": "integratedTerminal"
    },
    {
      "type": "maven",
      "name": "Maven: Clean",
      "request": "launch",
      "goals": ["clean"],
      "profiles": [],
      "skipTests": true,
      "mavenHome": "${config:maven.home}",
      "jdkPath": "${config:java.jdkPath}",
      "settings": "${config:maven.settings}"
    },
    {
      "type": "maven",
      "name": "Maven: Compile",
      "request": "launch",
      "goals": ["compile"],
      "profiles": [],
      "skipTests": true,
      "mavenHome": "${config:maven.home}",
      "jdkPath": "${config:java.jdkPath}",
      "settings": "${config:maven.settings}"
    },
    {
      "type": "maven",
      "name": "Maven: Test",
      "request": "launch",
      "goals": ["test"],
      "profiles": [],
      "skipTests": false,
      "mavenHome": "${config:maven.home}",
      "jdkPath": "${config:java.jdkPath}",
      "settings": "${config:maven.settings}",
      "vmArgs": "-Xmx2048m"
    },
    {
      "type": "maven",
      "name": "Maven: Clean Install",
      "request": "launch",
      "goals": ["clean", "install"],
      "profiles": [],
      "skipTests": false,
      "mavenHome": "${config:maven.home}",
      "jdkPath": "${config:java.jdkPath}",
      "settings": "${config:maven.settings}",
      "vmArgs": "-Xmx2048m"
    },
    {
      "type": "maven",
      "name": "Maven: Package (Skip Tests)",
      "request": "launch",
      "goals": ["package"],
      "profiles": [],
      "skipTests": true,
      "properties": {
        "skipTests": "true"
      },
      "mavenHome": "${config:maven.home}",
      "jdkPath": "${config:java.jdkPath}",
      "settings": "${config:maven.settings}"
    }
  ]
}
```

**配置说明**：
- 所有 `${config:xxx}` 会自动引用 `java.json` 中的配置
- 修改路径时只需修改 `java.json`，这里无需改动

**⚠️ 需要替换的内容**：
- `projectName`: 替换为你的项目名称（pom.xml 中的 artifactId）

---

#### 2.3 创建 `.kiro/settings/tasks.json`（任务配置）

```json
{
  "version": "1.0.0",
  "tasks": [
    {
      "label": "Maven: Clean",
      "type": "shell",
      "command": "${config:maven.home}/bin/mvn",
      "args": [
        "clean",
        "-s",
        "${config:maven.settings}"
      ],
      "options": {
        "env": {
          "JAVA_HOME": "${config:java.jdkPath}"
        }
      },
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      }
    },
    {
      "label": "Maven: Compile",
      "type": "shell",
      "command": "${config:maven.home}/bin/mvn",
      "args": [
        "compile",
        "-s",
        "${config:maven.settings}"
      ],
      "options": {
        "env": {
          "JAVA_HOME": "${config:java.jdkPath}"
        }
      },
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      }
    },
    {
      "label": "Maven: Test",
      "type": "shell",
      "command": "${config:maven.home}/bin/mvn",
      "args": [
        "test",
        "-s",
        "${config:maven.settings}"
      ],
      "options": {
        "env": {
          "JAVA_HOME": "${config:java.jdkPath}"
        }
      },
      "group": "test",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      }
    },
    {
      "label": "Maven: Clean Install",
      "type": "shell",
      "command": "${config:maven.home}/bin/mvn",
      "args": [
        "clean",
        "install",
        "-s",
        "${config:maven.settings}"
      ],
      "options": {
        "env": {
          "JAVA_HOME": "${config:java.jdkPath}"
        }
      },
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      }
    },
    {
      "label": "Maven: Package (Skip Tests)",
      "type": "shell",
      "command": "${config:maven.home}/bin/mvn",
      "args": [
        "package",
        "-DskipTests",
        "-s",
        "${config:maven.settings}"
      ],
      "options": {
        "env": {
          "JAVA_HOME": "${config:java.jdkPath}"
        }
      },
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      }
    },
    {
      "label": "验证 Java 和 Maven 配置",
      "type": "shell",
      "command": "echo",
      "args": [
        "'=== Java 版本 ===' && ${config:java.jdkPath}/bin/java -version && echo '\\n=== Maven 版本 ===' && ${config:maven.home}/bin/mvn -version"
      ],
      "options": {
        "env": {
          "JAVA_HOME": "${config:java.jdkPath}"
        }
      },
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": true,
        "panel": "shared"
      }
    }
  ]
}
```

**配置说明**：
- 所有路径都使用 `${config:xxx}` 变量引用
- 无需手动修改，自动从 `java.json` 读取配置

**💡 Kiro 配置的优势**：
1. **一处修改，全局生效** - 只需修改 `java.json`
2. **变量引用** - 使用 `${config:xxx}` 语法
3. **易于维护** - 配置集中管理

---

## 🔧 第三步：验证配置

### VSCode 验证

#### 3.1 重新加载窗口

配置完成后，重新加载 IDE 窗口：

1. 按 `Cmd+Shift+P`（macOS）或 `Ctrl+Shift+P`（Windows/Linux）
2. 输入 `Developer: Reload Window`
3. 回车执行

### 3.2 验证环境（VSCode）

1. 按 `Cmd+Shift+P`
2. 输入 `Tasks: Run Task`
3. 选择 `验证 Java 和 Maven 环境`

你应该看到类似以下输出：

```
=== Java 版本 ===
openjdk version "17.0.16" 2024-01-16
OpenJDK Runtime Environment Homebrew (build 17.0.16+0)
OpenJDK 64-Bit Server VM Homebrew (build 17.0.16+0, mixed mode, sharing)

=== Maven 版本 ===
Apache Maven 3.9.6
Maven home: /Users/erik.zhou/Desktop/www/project/config/maven
Java version: 17.0.16
```

### 3.3 编译项目（VSCode）

1. 按 `Cmd+Shift+B`（默认构建任务）
2. 或者：`Cmd+Shift+P` → `Tasks: Run Task` → `Maven: Compile`

如果编译成功，说明配置正确！

---

### Kiro 验证

#### 3.1 验证配置

在 Kiro 中运行验证任务：

1. 按 `Cmd+Shift+P` 打开命令面板
2. 输入 "Run Task" 或 "运行任务"
3. 选择 `验证 Java 和 Maven 配置`

你应该看到类似以下输出：

```
=== Java 版本 ===
openjdk version "17.0.16" 2024-01-16
OpenJDK Runtime Environment Homebrew (build 17.0.16+0)
OpenJDK 64-Bit Server VM Homebrew (build 17.0.16+0, mixed mode, sharing)

=== Maven 版本 ===
Apache Maven 3.9.6
Maven home: /Users/erik.zhou/Desktop/www/project/config/maven
Java version: 17.0.16
```

#### 3.2 编译项目

在 Kiro 中运行编译任务：

1. 按 `Cmd+Shift+P`
2. 输入 "Run Task"
3. 选择 `Maven: Compile` 或 `Maven: Clean Install`

如果编译成功，说明配置正确！

---

## 🚀 第四步：使用配置

### VSCode 使用方式

#### 4.1 运行 Maven 任务

**方式 1：通过快捷键**
- 编译项目：`Cmd+Shift+B`

**方式 2：通过任务菜单**
1. 按 `Cmd+Shift+P`
2. 输入 `Tasks: Run Task`
3. 选择要执行的任务

**方式 3：通过终端**
```bash
# 编辑器会自动设置 JAVA_HOME，可以直接运行
mvn clean install
mvn test
mvn package -DskipTests
```

### 4.2 调试测试

1. 打开测试类文件（例如：`src/test/java/com/example/MyTest.java`）
2. 在代码行号左侧点击设置断点
3. 点击类名或方法名旁边的 ▶️ 运行按钮
4. 或点击 🐛 调试按钮
5. 或按 `F5` 开始调试

### 4.3 运行指定测试（VSCode）

```bash
# 运行单个测试类
mvn test -Dtest=MyTest

# 运行指定包下的所有测试
mvn test -Dtest=com.example.*

# 运行指定测试方法
mvn test -Dtest=MyTest#testMethod
```

---

### Kiro 使用方式

#### 4.1 运行 Maven 任务

**方式 1：通过运行配置（推荐）**
1. 按 `Cmd+Shift+P`
2. 输入 "Run" 或 "运行"
3. 选择配置好的任务：
   - `Maven: Clean` - 清理项目
   - `Maven: Compile` - 编译项目
   - `Maven: Test` - 运行所有测试
   - `Maven: Clean Install` - 清理并安装
   - `Maven: Package (Skip Tests)` - 打包（跳过测试）

**方式 2：通过任务菜单**
1. 按 `Cmd+Shift+P`
2. 输入 "Tasks: Run Task"
3. 选择对应的 Maven 任务

**方式 3：通过终端**
```bash
# Kiro 会自动设置 JAVA_HOME，可以直接运行
mvn compile
mvn test
mvn clean install
```

#### 4.2 调试测试

1. 打开测试类文件
2. 在代码行号左侧点击设置断点
3. 按 `F5` 或点击 "Debug" 按钮
4. 选择 "Debug: Current Test Class" 配置

#### 4.3 修改配置

如果需要更换 JDK 或 Maven，**只需修改一个文件**：

编辑 `.kiro/settings/java.json`：

```json
{
  "java.jdkPath": "你的新 JDK 路径",
  "java.version": "你的 JDK 版本",
  "maven.home": "你的新 Maven 路径",
  "maven.settings": "你的新 settings.xml 路径",
  "maven.localRepository": "你的新本地仓库路径"
}
```

保存后，所有其他配置文件会自动使用新的路径！

---

## 📝 配置模板总结

### VSCode 目录结构

### VSCode 目录结构

```
your-project/
├── .vscode/
│   ├── settings.json      # Java 和 Maven 环境配置
│   ├── launch.json        # 调试配置
│   └── tasks.json         # 任务配置
├── src/
│   ├── main/java/
│   └── test/java/
├── pom.xml
└── README.md
```

### Kiro 目录结构

```
your-project/
├── .kiro/
│   ├── settings/
│   │   ├── java.json      # 核心配置（JDK 和 Maven 路径）✨
│   │   ├── launch.json    # 运行和调试配置
│   │   └── tasks.json     # 任务配置
│   └── README.md          # 使用说明（可选）
├── src/
│   ├── main/java/
│   └── test/java/
├── pom.xml
└── README.md
```

### 配置文件对比

| 配置项 | VSCode | Kiro |
|--------|--------|------|
| 配置目录 | `.vscode/` | `.kiro/settings/` |
| 核心配置 | `settings.json` | `java.json` |
| 调试配置 | `launch.json` | `launch.json` |
| 任务配置 | `tasks.json` | `tasks.json` |
| 变量引用 | ❌ 不支持 | ✅ 支持 `${config:xxx}` |
| 配置维护 | 需要修改多个文件 | 只需修改 `java.json` |

### 配置文件清单

#### VSCode 配置

| 文件 | 用途 | 必需 |
|------|------|------|
| `.vscode/settings.json` | Java 和 Maven 环境配置 | ✅ 必需 |
| `.vscode/launch.json` | 调试配置 | ⭐ 推荐 |
| `.vscode/tasks.json` | 任务配置（一键运行） | ⭐ 推荐 |

#### Kiro 配置

| 文件 | 用途 | 必需 |
|------|------|------|
| `.kiro/settings/java.json` | 核心配置（JDK 和 Maven 路径）| ✅ 必需 |
| `.kiro/settings/launch.json` | 运行和调试配置 | ⭐ 推荐 |
| `.kiro/settings/tasks.json` | 任务配置 | ⭐ 推荐 |

---

## 🔄 快速配置脚本

为了方便批量配置多个项目，可以使用以下脚本：

### macOS/Linux 脚本

创建 `setup-ide-config.sh`：

```bash
#!/bin/bash

# 配置变量（根据实际情况修改）
JDK_PATH="/opt/homebrew/Cellar/openjdk@17/17.0.16/libexec/openjdk.jdk/Contents/Home"
JDK_VERSION="17"
MAVEN_BIN="/Users/erik.zhou/Desktop/www/project/config/maven/bin/mvn"
MAVEN_SETTINGS="/Users/erik.zhou/Desktop/www/project/config/maven/settings.xml"
MAVEN_REPO="/Users/erik.zhou/Desktop/www/project/config/maven/repository"
PROJECT_NAME="your-project-name"

# 创建 .vscode 目录
mkdir -p .vscode

# 生成 settings.json
cat > .vscode/settings.json << EOF
{
  "java.configuration.runtimes": [
    {
      "name": "JavaSE-${JDK_VERSION}",
      "path": "${JDK_PATH}",
      "default": true
    }
  ],
  "java.home": "${JDK_PATH}",
  "java.jdt.ls.java.home": "${JDK_PATH}",
  "maven.executable.path": "${MAVEN_BIN}",
  "maven.settingsFile": "${MAVEN_SETTINGS}",
  "maven.localRepository": "${MAVEN_REPO}",
  "maven.terminal.useJavaHome": true,
  "maven.terminal.customEnv": [
    {
      "environmentVariable": "JAVA_HOME",
      "value": "${JDK_PATH}"
    }
  ],
  "java.import.maven.enabled": true,
  "java.configuration.maven.userSettings": "${MAVEN_SETTINGS}",
  "java.project.sourcePaths": ["src/main/java"],
  "java.project.outputPath": "target/classes",
  "java.test.config": {
    "vmArgs": ["-Xmx2048m"]
  }
}
EOF

echo "✅ 配置完成！"
echo "📁 配置文件位置: .vscode/settings.json"
echo "🔄 请重新加载 IDE 窗口"
```

使用方法：

```bash
# 1. 将脚本复制到项目根目录
# 2. 修改脚本中的配置变量
# 3. 添加执行权限
chmod +x setup-ide-config.sh

# 4. 运行脚本
./setup-ide-config.sh
```

---

## ❓ 常见问题

### Q1: 配置后 IDE 没有识别到 JDK？

**解决方案**：
1. 检查 JDK 路径是否正确
2. 重新加载窗口：`Cmd+Shift+P` → `Developer: Reload Window`
3. 检查 `.vscode/settings.json` 文件是否存在语法错误

### Q2: Maven 命令找不到？

**解决方案**：
1. 确认 Maven 可执行文件路径是否正确（应该是 `/path/to/maven/bin/mvn`）
2. 确认 settings.xml 路径是否正确
3. 在终端手动运行一次验证：`/path/to/maven/bin/mvn -version`

### Q3: 测试运行失败？

**解决方案**：
1. 先运行 `Maven: Clean Install` 编译项目
2. 检查 JUnit 版本是否兼容
3. 查看控制台输出的错误信息
4. 确认 `java.test.config.vmArgs` 中的内存配置是否足够

### Q4: 如何为不同项目使用不同的 JDK 版本？

**解决方案**：
每个项目的 `.vscode/settings.json` 是独立的，只需在不同项目中配置不同的 JDK 路径即可。

示例：
- 项目 A 使用 JDK 11：`"java.home": "/path/to/jdk11"`
- 项目 B 使用 JDK 17：`"java.home": "/path/to/jdk17"`

### Q5: 配置会影响其他项目吗？

**不会**。`.vscode` 目录下的配置只对当前项目生效，不会影响其他项目或全局设置。

---

## 📚 参考资料

- [VSCode Java 配置文档](https://code.visualstudio.com/docs/java/java-project)
- [Maven 官方文档](https://maven.apache.org/guides/)
- [JDK 安装指南](https://openjdk.org/install/)

---

## 📋 配置检查清单

在完成配置后，请检查以下项目：

- [ ] `.vscode/settings.json` 文件已创建
- [ ] JDK 路径配置正确
- [ ] Maven 路径配置正确
- [ ] settings.xml 路径配置正确
- [ ] 本地仓库路径配置正确
- [ ] 已重新加载 IDE 窗口
- [ ] 运行"验证 Java 和 Maven 环境"任务成功
- [ ] 能够成功编译项目
- [ ] 能够成功运行测试

---

## 🎯 总结

通过本文档的配置流程，你可以：

1. ✅ 为每个 Java 项目独立配置 JDK 和 Maven 环境
2. ✅ 实现一键编译、测试、打包
3. ✅ 支持断点调试
4. ✅ 团队成员使用统一的开发环境
5. ✅ 快速复制配置到其他项目

**建议**：
- 将配置文件模板保存到代码仓库中
- 在团队内部共享配置标准
- 定期更新 JDK 和 Maven 版本

---

**文档版本**: v1.0  
**最后更新**: 2026-02-03  
**作者**: erik.zhou
