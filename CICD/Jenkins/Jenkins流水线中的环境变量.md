

## 1. 介绍

在 Jenkins Pipeline 中，所有可用的环境变量通过全局变量 `env` 暴露，且在 Jenkinsfile 的任何位置都可访问。  
环境变量提供了构建期间的上下文信息（如构建号、工作空间、节点名称等），并且可用于脚本、构建参数、部署脚本等。

---

## 2. 常见内置环境变量（示例与说明）

| 变量名 | 示例值 | 说明 |
|---------|---------|------|
| `BUILD_ID` | 2025-10-30_09-40-18 | 当前构建 ID。对于 Jenkins 1.597+ 与 `BUILD_NUMBER` 相同。 |
| `BUILD_NUMBER` | 153 | 当前构建号。 |
| `BUILD_TAG` | jenkins-MyJob-17 | 可用于打包、标识构建产物。 |
| `BUILD_URL` | http://jenkins/job/MyJob/17/ | 当前构建的 URL。 |
| `EXECUTOR_NUMBER` | 0 | 执行该构建的执行器编号（从 0 开始）。 |
| `JAVA_HOME` | /usr/lib/jvm/java-17 | 当前任务使用的 JDK 路径。 |
| `JENKINS_URL` | https://jenkins.example.com/ | Jenkins 主 URL。 |
| `JOB_NAME` | my-pipeline | 任务名。 |
| `NODE_NAME` | build-node-1 | 运行任务的节点名。 |
| `WORKSPACE` | /var/lib/jenkins/workspace/my-pipeline | 构建工作目录。 |

> 更多变量请参考 `${YOUR_JENKINS_URL}/pipeline-syntax/globals#env`。

---

## 3. 在 Pipeline 中读取环境变量

在 Jenkinsfile 中可通过 `${env.VAR_NAME}` 或 `env.VAR_NAME` 引用变量：

```groovy
pipeline {
  agent any
  stages {
    stage('Example') {
      steps {
        echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
      }
    }
  }
}

```

------

## 4. 在 Pipeline 中设置环境变量

### 4.1 Declarative Pipeline（`environment` 指令）

Declarative Pipeline 支持在顶层和 stage 内定义 `environment` 块：

```groovy
pipeline {
  agent any
  environment {
    CC = 'clang'
    APP_ENV = 'dev'
  }
  stages {
    stage('Compile') {
      environment { DEBUG_FLAGS = '-g' }
      steps {
        sh 'echo CC=$CC, APP_ENV=$APP_ENV, DEBUG_FLAGS=$DEBUG_FLAGS'
      }
    }
  }
}

```

------

- 顶层定义：对整个 Pipeline 生效
- stage 内定义：仅对该阶段生效

### 4.2 Scripted Pipeline（`withEnv`）

```groovy
node {
  withEnv(['CC=clang', 'APP_ENV=staging']) {
    sh 'echo $CC $APP_ENV'
  }
}
```

## 5. 动态设置环境变量（运行时）

### Declarative Pipeline

```groovy
pipeline {
  agent any
  environment {
    GIT_SHORT = "${sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()}"
    EXIT_STATUS = "${sh(returnStatus: true, script: 'exit 1')}"
  }
  stages {
    stage('Show') {
      steps {
        sh 'echo GIT_SHORT=$GIT_SHORT; echo EXIT_STATUS=$EXIT_STATUS'
      }
    }
  }
}
```

> - `returnStdout`: 返回命令输出，建议 `.trim()` 去除换行。
> - `returnStatus`: 返回命令退出码。
> - 顶层必须设置 `agent`，否则无法在 `environment` 中执行 `sh`。

### Scripted Pipeline

```groovy
node {
  def ver = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
  env.VERSION = ver
  sh 'echo version is $VERSION'
}
```

------

## 6. 使用 Credentials 注入敏感信息

推荐使用 Jenkins Credentials 管理密码或 Token。

### Declarative

```groovy
pipeline {
  agent any
  environment {
    MY_SECRET = credentials('my-secret-id')
  }
  stages {
    stage('Use') {
      steps {
        sh 'echo Using secret: $MY_SECRET'
      }
    }
  }
}
```

### Scripted

```groovy
node {
  withCredentials([string(credentialsId: 'my-secret-id', variable: 'SECRET')]) {
    sh 'echo $SECRET'
  }
}
```

------

## 7. 在不同脚本中引用变量

| 脚本类型      | 写法       |
| ------------- | ---------- |
| Shell (`sh`)  | `$VAR`     |
| Batch (`bat`) | `%VAR%`    |
| PowerShell    | `$env:VAR` |

示例：

```groovy
steps {
  sh 'echo $BUILD_NUMBER'
  bat 'echo %BUILD_NUMBER%'
  powershell 'Write-Host $env:BUILD_NUMBER'
}
```

------

## 8. 环境变量作用域与优先级

优先级从高到低：

1. 动态设置 `env.VAR = ...`
2. stage 内定义
3. 顶层定义
4. Jenkins 全局属性
5. 插件或系统内置变量

------

## 9. 实战示例

### 示例 A：Git 短哈希作为版本号

```groovy
pipeline {
  agent any
  environment {
    APP_NAME = 'demo-app'
  }
  stages {
    stage('Prepare') {
      steps {
        script {
          env.VERSION = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        }
        echo "Building ${env.APP_NAME}:${env.VERSION}"
      }
    }
    stage('Build') {
      steps {
        sh 'docker build -t myregistry/${APP_NAME}:${VERSION} .'
      }
    }
  }
}
```

### 示例 B：使用 Credentials 登录 Docker

```groovy
pipeline {
  agent any
  environment { DOCKER_REG = 'registry.example.com' }
  stages {
    stage('Login') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'REG_USER', passwordVariable: 'REG_PSW')]) {
          sh 'docker login -u $REG_USER -p $REG_PSW $DOCKER_REG'
        }
      }
    }
  }
}
```

------

## 10. 最佳实践

- **敏感信息使用 Credentials**
- **使用 `.trim()` 清除换行**
- **集中声明变量**，便于维护
- **避免与系统变量重名**
- **日志中不输出凭据**
- **多节点构建时注意路径差异（WORKSPACE）**

------

## 11. 常见问题与排查

| 问题                          | 原因与解决                                               |
| ----------------------------- | -------------------------------------------------------- |
| `environment` 中无法执行 `sh` | 顶层未定义 `agent`，改为 `agent any` 或移到 `script{}`。 |
| `returnStdout` 输出含换行     | 使用 `.trim()`。                                         |
| 凭据为空或未遮蔽              | 检查凭据 ID、任务权限及类型。                            |
| shell 中变量无效              | 检查作用域及语法（sh 用 `$VAR`，bat 用 `%VAR%`）。       |

------

## 12. 参考

- 官方文档: `${YOUR_JENKINS_URL}/pipeline-syntax/globals#env`
- Jenkins Credentials Plugin 文档
- Declarative Pipeline Reference

------

```
---

是否需要我帮你再生成一份对照表（所有常见内置变量 + 示例 + 说明）单独做成 Markdown 表格？那样可以直接放进项目 Wiki。
```