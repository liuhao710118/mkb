## 流水线模板参考

好的，我可以给你整理一个 **Jenkins Declarative Pipeline 模板**，覆盖大部分常用功能：参数化、环境变量、工具、并行/顺序阶段、条件执行、输入确认、失败处理等，可作为参考模板直接使用或改造。

```groovy
pipeline {
    agent any

    // --------------------------
    // 参数化
    // --------------------------
    parameters {
        string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '部署环境')
        booleanParam(name: 'DEBUG_BUILD', defaultValue: false, description: '是否开启调试模式')
        choice(name: 'CHOICE', choices: ['option1','option2','option3'], description: '选择执行选项')
        password(name: 'SECRET_PASSWORD', defaultValue: '', description: '敏感密码')
    }

    // --------------------------
    // 全局环境变量
    // --------------------------
    environment {
        JAVA_HOME = tool name: 'jdk-21', type: 'jdk'
        MAVEN_HOME = tool name: 'apache-maven-3.9.3', type: 'maven'
        GLOBAL_KEY = credentials('global-secret')
    }

    // --------------------------
    // 全局选项
    // --------------------------
    options {
        timeout(time: 1, unit: 'HOURS')           // 全局超时
        timestamps()                              // 控制台加时间戳
        buildDiscarder(logRotator(numToKeepStr: '5')) // 保留最近5次构建
        disableConcurrentBuilds()                 // 禁止并行构建
        parallelsAlwaysFailFast()                 // 并行阶段失败即终止其他并行
    }

    // --------------------------
    // 自动触发器
    // --------------------------
    triggers {
        cron('H 2 * * *') // 每天凌晨2点触发
    }

    // --------------------------
    // 流水线阶段
    // --------------------------
    stages {

        // -----------------------------------
        // 构建阶段
        // -----------------------------------
        stage('Build') {
            agent { label 'builder-node' }
            when { expression { return params.CHOICE == 'option1' } }
            environment {
                BUILD_ENV = 'local'
            }
            steps {
                echo "Building project in ${params.DEPLOY_ENV} environment"
                sh '${MAVEN_HOME}/bin/mvn clean package -DskipTests'
            }
            post {
                success { echo 'Build succeeded!' }
                failure { echo 'Build failed!' }
            }
        }

        // -----------------------------------
        // 测试阶段 - 并行执行
        // -----------------------------------
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        echo 'Running unit tests'
                        sh '${MAVEN_HOME}/bin/mvn test'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        echo 'Running integration tests'
                        sh '${MAVEN_HOME}/bin/mvn verify -Pintegration'
                    }
                }
            }
        }

        // -----------------------------------
        // 部署阶段 - 顺序执行
        // -----------------------------------
        stage('Deploy') {
            when { branch 'main' }
            input {
                message "确认是否部署到 ${params.DEPLOY_ENV}?"
                ok "Deploy"
                submitter "admin,devops"
            }
            stages {
                stage('Deploy App') {
                    steps {
                        echo "Deploying application to ${params.DEPLOY_ENV}"
                        sh "ansible-playbook deploy.yml -e env=${params.DEPLOY_ENV}"
                    }
                }
                stage('Smoke Test') {
                    steps {
                        echo 'Running smoke tests after deployment'
                        sh './smoke_test.sh'
                    }
                }
            }
        }

        // -----------------------------------
        // 矩阵示例 - 多环境/多版本
        // -----------------------------------
        stage('Matrix Build') {
            matrix {
                axes {
                    axis {
                        name 'PLATFORM'
                        values 'linux','windows'
                    }
                    axis {
                        name 'JDK_VERSION'
                        values '8','11'
                    }
                }
                stages {
                    stage('Compile') {
                        steps {
                            echo "Building on ${PLATFORM} with JDK ${JDK_VERSION}"
                        }
                    }
                }
            }
        }
    }

    // --------------------------
    // 构建后操作
    // --------------------------
    post {
        always {
            echo '流水线结束'
        }
        success {
            echo '流水线成功完成'
        }
        failure {
            echo '流水线执行失败'
        }
        cleanup {
            echo '执行清理操作'
        }
    }
}
```

### 🔹 特性说明

1. **参数化**：string、boolean、choice、password。
2. **全局环境变量**：支持工具路径和凭证。
3. **全局选项**：超时、时间戳、并行失败快终止、保留构建历史。
4. **触发器**：cron 定时触发。
5. **阶段控制**：
   - 顺序阶段 (`stages`)
   - 并行阶段 (`parallel`)
   - 矩阵阶段 (`matrix`)
6. **条件执行**：`when` 支持表达式或分支判断。
7. **人工确认**：`input` 指令。
8. **构建后处理**：`post` 支持 `always/failure/success/cleanup`。



## agent（执行节点）

### 1 agent（执行节点）

`agent` 指定 Pipeline 或阶段在 Jenkins 中 **运行的节点或容器**。

- **必须**在顶层 pipeline 内定义，阶段内可选。
- **顶层 agent 与阶段 agent 的差异**主要在 **timeout 计时**：

**顶层 Agent 示例**：

```groovy
pipeline {
    agent any
    options {
        // timeout 计时从 agent 分配完成后开始
        // timeout 选项是用于限制流水线的执行时间 当超过之前时间 流水线就会被自动终止防止假运行
        timeout(time: 1, unit: 'SECONDS')
    }
    stages {
        stage('示例') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

**阶段 Agent 示例**：

```groovy
pipeline {
    agent none
    stages {
        stage('示例') {
            agent any
            options {
                // timeout 计时从 agent 分配前开始，包括分配时间
                timeout(time: 1, unit: 'SECONDS')
            }
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

> 注意：阶段级 timeout 包含 agent 分配时间，如果节点分配延迟可能导致 Pipeline 失败。

------

### 2 Agent 的类型

| 类型           | 描述                                       | 示例                                  |
| -------------- | ------------------------------------------ | ------------------------------------- |
| **any**        | 在任何可用节点运行                         | `agent any`                           |
| **none**       | 不分配全局节点，每个阶段必须单独指定 agent | `agent none`                          |
| **label**      | 在指定 label 节点运行                      | `agent { label 'my-label' }`          |
| **node**       | 类似 label，但可配置 workspace             | `agent { node { label 'my-label' } }` |
| **docker**     | 在 Docker 容器中运行                       | `agent { docker 'maven:3.9.3' }`      |
| **dockerfile** | 使用仓库中的 Dockerfile 构建容器运行       | `agent { dockerfile true }`           |
| **kubernetes** | 在 Kubernetes Pod 内运行                   | 详见下文示例                          |

------

#### Docker Agent 示例

**示例 1：顶层 Docker Agent**

```groovy
pipeline {
    agent { docker 'maven:3.9.3-eclipse-temurin-17' } 
    stages {
        stage('构建') {
            steps {
                sh 'mvn -B clean verify'
            }
        }
    }
}
```

**示例 2：阶段级 Docker Agent**

```groovy
pipeline {
    agent none
    stages {
        stage('构建') {
            agent { docker 'maven:3.9.9-eclipse-temurin-21' } 
            steps {
                echo 'Hello, Maven'
                sh 'mvn --version'
            }
        }
        stage('测试') {
            agent { docker 'openjdk:21-jre' } 
            steps {
                echo 'Hello, JDK'
                sh 'java -version'
            }
        }
    }
}
```

> 使用 `agent none` 可以避免全局 Executor 占用，每个阶段可独立选择节点或容器。

------

#### Kubernetes Agent 示例

```groovy
pipeline {
    agent {
        kubernetes {
            defaultContainer 'kaniko'
            yaml '''
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - sleep
    args:
    - 99d
    volumeMounts:
      - name: aws-secret
        mountPath: /root/.aws/
      - name: docker-registry-config
        mountPath: /kaniko/.docker
  volumes:
    - name: aws-secret
      secret:
        secretName: aws-secret
    - name: docker-registry-config
      configMap:
        name: docker-registry-config
'''
        }
    }
    stages {
        stage('构建') {
            steps {
                echo '在 Kubernetes Pod 内运行'
            }
        }
    }
}
```

> 需在 Kubernetes 中提前创建 `aws-secret`（Kaniko 访问 ECR）及 `docker-registry-config`（Docker 配置）。

------

### 3 共用的 Agent 选项

| 选项            | 描述                | 适用类型                 |
| --------------- | ------------------- | ------------------------ |
| label           | 节点 label          | node、docker、dockerfile |
| customWorkspace | 自定义工作空间      | node、docker、dockerfile |
| reuseNode       | 重用顶层节点        | docker、dockerfile       |
| args            | docker run 额外参数 | docker、dockerfile       |

#### 3.1 label（节点标签）

- 类型：字符串
- 描述：指定 Pipeline 或阶段运行的节点标签或标签条件。
- 适用类型：`node`、`docker`、`dockerfile`
- 注意：`node` 类型必须指定 `label`

```groovy
agent {
    node {
        label 'my-defined-label'
    }
}
```

------

#### 3.2 customWorkspace（自定义工作空间）

- 类型：字符串
- 描述：在指定路径下运行 Pipeline 或阶段，而不是使用默认工作空间。
  - 相对路径：工作空间位于节点 workspace 根目录下
  - 绝对路径：直接指定完整路径

```groovy
agent {
    node {
        label 'my-defined-label'
        customWorkspace '/some/other/path'
    }
}
```

- 适用类型：`node`、`docker`、`dockerfile`

------

#### 3.3 reuseNode（重用节点）

**作用**：在阶段级 Docker agent 中，如果 `reuseNode: true`，容器会在 **Pipeline 顶层指定的节点** 上运行，并共享顶层的工作空间，而不是新分配节点。

**注意：**<font color="red">**仅在阶段级 agent 中生效**</font>

```groovy
pipeline {
    agent {
        node {
            label 'build-node'
        }
    }
    stages {
        stage('构建阶段') {
            agent {
                docker {
                    image 'maven:3.9.3-eclipse-temurin-17'
                    reuseNode true
                }
            }
            steps {
                echo "当前工作目录: ${pwd()}"
                sh 'mvn -B clean package'
            }
        }
        stage('测试阶段') {
            agent {
                docker {
                    image 'maven:3.9.3-eclipse-temurin-17'
                    reuseNode true
                }
            }
            steps {
                echo "共享顶层节点工作空间"
                sh 'mvn test'
            }
        }
    }
}

```





------

#### 3.4 args（运行参数）

**作用**：给 `docker run` 传递额外参数，如挂载目录、环境变量等。

```groovy
pipeline {
    agent none
    stages {
        stage('自定义参数 Docker') {
            agent {
                docker {
                    image 'maven:3.9.3-eclipse-temurin-17'
                    args '-v /tmp:/tmp -e MAVEN_OPTS="-Xmx512m"'
                }
            }
            steps {
                echo '在 Docker 容器中挂载 /tmp 并设置 MAVEN_OPTS'
                sh 'echo $MAVEN_OPTS'
                sh 'mvn -B clean package'
            }
        }
    }
}

```



好的，我将这一段 **`stages` 与 `steps`** 的内容翻译整理成中文易懂文档，并附示例。

------

## stages（阶段）

`stages` 用于定义 **Pipeline 中的一个或多个阶段（stage）**，是 Pipeline 中执行主要任务的部分。

> 通常每个阶段对应持续交付流程中的一个离散环节，例如：**构建（Build）、测试（Test）、部署（Deploy）**。

### 基本信息

- **必需**：是
- **参数**：无
- **允许位置**：顶层 pipeline 块内，或 stage 内嵌套（矩阵等场景）

------

### 示例：声明式 Pipeline 的 stages

```groovy
pipeline {
    agent any
    stages { 
        stage('示例阶段') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

**说明**：

1. `stages` 放在顶层 Pipeline 内，通常紧跟在 `agent`、`options` 等指令之后。
2. 每个 `stage` 定义一个具体任务或流程环节。

------

## steps（步骤）

`steps` 用于定义 **在阶段内要执行的一系列操作**。
 每个阶段至少要包含一个或多个步骤，否则阶段不会执行任何工作。

### 基本信息

- **必需**：是
- **参数**：无
- **允许位置**：每个 stage 块内

------

### 示例：单步骤 Stage

```groovy
pipeline {
    agent any
    stages {
        stage('示例阶段') {
            steps { 
                echo 'Hello World'
            }
        }
    }
}
```

**说明**：

1. `steps` 内可以放置任意 Pipeline 支持的步骤，例如 `sh`、`echo`、`checkout` 等。
2. 每个 `stage` 必须包含 `steps`，否则该阶段不会执行任何操作。

### `script`

- **作用**：允许在 Declarative Pipeline 中嵌入 **Scripted Pipeline** 的 Groovy 代码。
- **用途**：
  - 当 Declarative 本身无法实现复杂逻辑时，可以用 `script` 执行 Groovy 脚本。
  - 非复杂逻辑最好保持在 Declarative 中，否则建议抽到 Shared Library。
- **注意**：不要滥用 `script`，否则失去 Declarative 的可读性和结构优势。

------

### 示例

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'

                script { 
                    // 这里可以写任意 Scripted Pipeline / Groovy 代码
                    def browsers = ['chrome', 'firefox']
                    for (int i = 0; i < browsers.size(); ++i) {
                        echo "Testing the ${browsers[i]} browser"
                    }
                }
            }
        }
    }
}
```

✅ 输出：

```
Hello World
Testing the chrome browser
Testing the firefox browser
```

------

### 总结

- **Declarative Pipeline** 强调结构化、易读和可维护。
- **Scripted Pipeline** 是完整的 Groovy DSL，更灵活，但可读性差。
- `script` 步骤是 Declarative 的 **逃生出口**，用于处理复杂逻辑或循环。

## post（后置操作）

`post` 用于定义 **Pipeline 或阶段运行完成后的额外步骤**，具体执行哪一组步骤取决于 `post` 放置的位置（顶层 Pipeline 或阶段内）。

`post` 支持多种 **条件块（post-condition blocks）**，用于根据运行状态执行不同的步骤。条件块的执行顺序如下：

------

### 1 基本信息

- **必需**：否
- **参数**：无
- **允许位置**：顶层 pipeline 块 或 每个 stage 块

------

### 2 条件块说明

| 条件             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| **always**       | 无论 Pipeline 或阶段的运行状态如何，都会执行此步骤。         |
| **changed**      | 仅当本次 Pipeline 的运行状态与上一次不同，才执行此步骤。     |
| **fixed**        | 仅当本次运行成功且上一次运行失败或不稳定时执行。             |
| **regression**   | 仅当本次运行失败、不稳定或被中止，且上一次运行成功时执行。   |
| **aborted**      | 仅当本次 Pipeline 被手动中止（状态为 aborted，Web UI 通常显示灰色）时执行。 |
| **failure**      | 仅当本次运行失败（状态为 failed，Web UI 通常显示红色）时执行。 |
| **success**      | 仅当本次运行成功（状态为 success，Web UI 通常显示蓝色或绿色）时执行。 |
| **unstable**     | 仅当本次运行不稳定（通常是测试失败或代码违规，Web UI 显示黄色）时执行。 |
| **unsuccessful** | 仅当本次运行不是成功状态时执行。适用于阶段，当构建本身不稳定时也会触发。 |
| **cleanup**      | 在所有其他 post 条件块执行完后，无论状态如何，都会执行此步骤。 |

------

### 3 示例：顶层 Post Section

```groovy
pipeline {
    agent any
    stages {
        stage('示例阶段') {
            steps {
                echo 'Hello World'
            }
        }
    }
    post { 
        always { 
            echo '我总会再次打招呼！'
        }
        success {
            echo '恭喜，Pipeline 成功运行！'
        }
        failure {
            echo '抱歉，Pipeline 运行失败！'
        }
        cleanup {
            echo '清理工作总是会执行'
        }
    }
}
```

**说明**：

1. **always**：无论成功还是失败，都会执行 `"我总会再次打招呼！"`。
2. **success**：仅在 Pipeline 成功时执行。
3. **failure**：仅在 Pipeline 失败时执行。
4. **cleanup**：在所有 post 条件执行完后执行清理操作。

> 通常情况下，`post` 放在 Pipeline 的末尾。
>  `post` 条件块内部的步骤语法与 `steps` 部分相同。



好的，我将这一段 **`environment` 指令** 内容整理翻译成中文易懂文档，并附详细示例。

------

## environment（环境变量）

### 常见内置环境变量（示例与说明）

| 变量名            | 示例值                                 | 说明                                                      |
| ----------------- | -------------------------------------- | --------------------------------------------------------- |
| `BUILD_ID`        | 2025-10-30_09-40-18                    | 当前构建 ID。对于 Jenkins 1.597+ 与 `BUILD_NUMBER` 相同。 |
| `BUILD_NUMBER`    | 153                                    | 当前构建号。                                              |
| `BUILD_TAG`       | jenkins-MyJob-17                       | 可用于打包、标识构建产物。                                |
| `BUILD_URL`       | http://jenkins/job/MyJob/17/           | 当前构建的 URL。                                          |
| `EXECUTOR_NUMBER` | 0                                      | 执行该构建的执行器编号（从 0 开始）。                     |
| `JAVA_HOME`       | /usr/lib/jvm/java-17                   | 当前任务使用的 JDK 路径。                                 |
| `JENKINS_URL`     | https://jenkins.example.com/           | Jenkins 主 URL。                                          |
| `JOB_NAME`        | my-pipeline                            | 任务名。                                                  |
| `NODE_NAME`       | build-node-1                           | 运行任务的节点名。                                        |
| `WORKSPACE`       | /var/lib/jenkins/workspace/my-pipeline | 构建工作目录。                                            |

> 更多变量请参考 `${YOUR_JENKINS_URL}/pipeline-syntax/globals#env`。

---



`environment` 指令用于定义 **环境变量（key-value 键值对）**，可以应用到整个 Pipeline 或特定阶段。

- 顶层定义：对整个 Pipeline 所有步骤生效
- 阶段内定义：仅对该阶段的步骤生效

------

### 1 基本信息

- **必需**：否
- **参数**：无
- **允许位置**：顶层 pipeline 或 stage 内
- **支持的方法**：`credentials()`，可访问 Jenkins 预定义的凭据（Credentials）

------

### 2 支持的凭据类型

| 类型                                      | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| **Secret Text（文本密钥）**               | 指定的环境变量将被设置为密钥文本内容                         |
| **Secret File（文件密钥）**               | 指定的环境变量将被设置为临时创建的文件路径                   |
| **Username and password（用户名和密码）** | 环境变量设置为 `username:password`，同时自动创建两个附加环境变量：`MYVARNAME_USR` 和 `MYVARNAME_PSW` |
| **SSH with Private Key（SSH 私钥）**      | 环境变量设置为临时创建的 SSH 私钥路径，同时自动创建 `MYVARNAME_USR` 和 `MYVARNAME_PSW`（存放密码短语） |

> 如果使用不支持的凭据类型，Pipeline 会失败，报错类似：
>  `CredentialNotFoundException: No suitable binding handler could be found for type <unsupportedType>`

------

### 3 示例 1：顶层 Secret Text

```groovy
pipeline {
    agent any
    environment { 
        CC = 'clang'  // 设置环境变量 CC
    }
    stages {
        stage('示例阶段') {
            environment { 
                AN_ACCESS_KEY = credentials('my-predefined-secret-text')  // 使用 Jenkins 凭据
            }
            steps {
                sh 'printenv'  // 打印所有环境变量
            }
        }
    }
}
```

**说明**：

1. 顶层 `environment`：`CC=clang` 对整个 Pipeline 生效
2. 阶段内 `environment`：`AN_ACCESS_KEY` 仅对当前阶段有效
3. `credentials('凭据ID')` 可访问 Jenkins 预定义凭据

------

### 4 示例 2：用户名/密码与 SSH 凭据

```groovy
pipeline {
    agent any
    stages {
        stage('用户名/密码示例') {
            environment {
                SERVICE_CREDS = credentials('my-predefined-username-password')
            }
            steps {
                sh 'echo "Service 用户是 $SERVICE_CREDS_USR"'
                sh 'echo "Service 密码是 $SERVICE_CREDS_PSW"'
                sh 'curl -u $SERVICE_CREDS https://myservice.example.com'
            }
        }
        stage('SSH 私钥示例') {
            environment {
                SSH_CREDS = credentials('my-predefined-ssh-creds')
            }
            steps {
                sh 'echo "SSH 私钥路径: $SSH_CREDS"'
                sh 'echo "SSH 用户: $SSH_CREDS_USR"'
                sh 'echo "SSH 密码短语: $SSH_CREDS_PSW"'
            }
        }
    }
}
```

**说明**：

1. **Username/Password**：
   - `SERVICE_CREDS` = `username:password`
   - 自动创建 `$SERVICE_CREDS_USR` 和 `$SERVICE_CREDS_PSW`
2. **SSH Private Key**：
   - `SSH_CREDS` = SSH 私钥文件路径
   - 自动创建 `$SSH_CREDS_USR` 和 `$SSH_CREDS_PSW`（存放 passphrase）

------

### 总结

- `environment` 可以在 Pipeline 顶层或阶段内定义
- 可结合 Jenkins 凭据（Secret Text / Secret File / Username/Password / SSH Key）使用
- 阶段级环境变量只对当前阶段有效，顶层环境变量对整个 Pipeline 生效
- 使用凭据时推荐 `credentials()` 方法，安全且方便管理。

------

## options（选项）

`options` 指令用于 **在 Pipeline 或阶段内部配置特定选项**，可用于控制构建行为、超时、重试等。

- 顶层 Pipeline 内的 `options` 作用于整个 Pipeline
- 阶段内的 `options` 仅作用于当前阶段

------

### 1. 基本信息

- **必需**：否
- **参数**：无
- **允许位置**：Pipeline 顶层或阶段内（阶段内有一定限制）

------

### 2 可用 Pipeline 全局选项

| 选项                        | 描述                                                         |
| --------------------------- | ------------------------------------------------------------ |
| **buildDiscarder**          | 保存最近若干次构建的工件和控制台输出。例如：`buildDiscarder(logRotator(numToKeepStr: '1'))` |
| **checkoutToSubdirectory**  | 在子目录中进行源码检出。例如：`checkoutToSubdirectory('foo')` |
| **disableConcurrentBuilds** | 禁止 Pipeline 并发执行，可选择队列或中止正在执行的构建。例如：`disableConcurrentBuilds()` 或 `disableConcurrentBuilds(abortPrevious: true)` |
| **disableResume**           | 禁止在 Jenkins 控制器重启后恢复 Pipeline。例如：`disableResume()` |
| **newContainerPerStage**    | 与顶层 docker/dockerfile agent 配合使用，每个阶段使用新的容器，而不是共享同一个容器 |
| **overrideIndexTriggers**   | 重写分支索引触发器设置。例如：`overrideIndexTriggers(true)`  |
| **preserveStashes**         | 保留完成构建的 stash 文件，可用于阶段重启。例如：`preserveStashes()` 或 `preserveStashes(buildCount: 5)` |
| **quietPeriod**             | 设置 Pipeline 的静默期（秒），覆盖全局默认值。例如：`quietPeriod(30)` |
| **retry**                   | 构建失败时重试指定次数。例如：`retry(3)`                     |
| **skipDefaultCheckout**     | 跳过默认的源码检出。例如：`skipDefaultCheckout()`            |
| **skipStagesAfterUnstable** | 构建状态变为 UNSTABLE 后跳过后续阶段。例如：`skipStagesAfterUnstable()` |
| **timeout**                 | 设置 Pipeline 超时时间，超时后 Jenkins 会中止。例如：`timeout(time: 1, unit: 'HOURS')` |
| **timestamps**              | 在控制台输出前加上时间戳。例如：`timestamps()`               |
| **parallelsAlwaysFailFast** | 对所有并行阶段设置 failFast 为 true，例如：`parallelsAlwaysFailFast()` |
| **disableRestartFromStage** | 禁用 UI 中的“从阶段重启”选项，例如：`disableRestartFromStage()` |

------

### 3 全局 Pipeline 示例：超时设置

```groovy
pipeline {
    agent any
    options {
        timeout(time: 1, unit: 'HOURS')  // 全局 Pipeline 超时 1 小时
    }
    stages {
        stage('示例阶段') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

------

### 4 阶段级 options

阶段内 `options` 仅能包含部分选项，如：`retry`、`timeout`、`timestamps`、`skipDefaultCheckout` 等。

> 阶段内 `options` 在进入 agent 或检查 `when` 条件前生效。

#### 可用阶段选项

| 选项                    | 描述                                                  |
| ----------------------- | ----------------------------------------------------- |
| **skipDefaultCheckout** | 阶段跳过默认源码检出，例如：`skipDefaultCheckout()`   |
| **timeout**             | 阶段超时设置，例如：`timeout(time: 1, unit: 'HOURS')` |
| **retry**               | 阶段失败时重试，例如：`retry(3)`                      |
| **timestamps**          | 阶段控制台输出加时间戳，例如：`timestamps()`          |

------

### 5 阶段超时示例

```groovy
pipeline {
    agent any
    stages {
        stage('示例阶段') {
            options {
                timeout(time: 1, unit: 'HOURS')  // 阶段超时 1 小时
            }
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

------

### 总结

1. **顶层 Pipeline options**：控制整个 Pipeline 的行为，如超时、重试、静默期、并行策略等。
2. **阶段级 options**：只作用于当前阶段，通常用于阶段超时、重试、时间戳和跳过源码检出。
3. `options` 是提高 Pipeline 可控性和稳定性的关键指令。



好的，我将 **`parameters` 指令** 内容整理成中文易懂文档，并附示例说明。

------

## parameters（构建参数）

`parameters` 指令用于 **定义用户在触发 Pipeline 时需要提供的参数**。

- 用户在手动触发或通过 API 调用构建时，可以填写这些参数。
- 参数值在 Pipeline 执行过程中可通过 **`params` 对象** 或环境变量访问。

> 在 POSIX Shell（bash/ksh）中使用 `${PARAMETER_NAME}`
>  在 PowerShell 中使用 `${Env:PARAMETER_NAME}`
>  在 Windows cmd.exe 中使用 `%PARAMETER_NAME%`

------

### 1 基本信息

- **必需**：否
- **参数**：无
- **允许位置**：只能在顶层 `pipeline` 块中出现一次

------

### 2 可用参数类型

| 类型             | 说明       | 示例                                                         |
| ---------------- | ---------- | ------------------------------------------------------------ |
| **string**       | 单行字符串 | `string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '部署环境')` |
| **text**         | 多行文本   | `text(name: 'DEPLOY_TEXT', defaultValue: 'One\nTwo\nThree\n', description: '输入文本')` |
| **booleanParam** | 布尔值     | `booleanParam(name: 'DEBUG_BUILD', defaultValue: true, description: '是否开启调试')` |
| **choice**       | 枚举选择   | `choice(name: 'CHOICES', choices: ['one','two','three'], description: '选择一个值')` |
| **password**     | 密码       | `password(name: 'PASSWORD', defaultValue: 'SECRET', description: '输入密码')` |

------

### 3 示例：Declarative Pipeline 参数

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: '我应该向谁问好？')

        text(name: 'BIOGRAPHY', defaultValue: '', description: '输入此人的相关信息')

        booleanParam(name: 'TOGGLE', defaultValue: true, description: '切换此值')

        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: '选择一个选项')

        password(name: 'PASSWORD', defaultValue: 'SECRET', description: '输入密码')
    }
    stages {
        stage('示例阶段') {
            steps {
                echo "Hello ${params.PERSON}"

                echo "Biography: ${params.BIOGRAPHY}"

                echo "Toggle: ${params.TOGGLE}"

                echo "Choice: ${params.CHOICE}"

                echo "Password: ${params.PASSWORD}"
            }
        }
    }
}
```

**说明**：

1. **string/text**：文本输入，`text` 支持多行
2. **booleanParam**：布尔切换开关
3. **choice**：下拉选择，第一项为默认值
4. **password**：输入密码，Jenkins 会隐藏显示
5. 访问方式：通过 `${params.PARAM_NAME}`

好的，我将 **`triggers` 指令** 以及 Jenkins cron 语法整理成中文易懂文档，并附示例。

------

##  triggers（触发器）

`triggers` 指令用于 **定义自动触发 Pipeline 的方式**。

- 对于 GitHub、BitBucket 等集成的 Pipeline，通常不需要触发器，因为 webhook 已经自动触发。
- 可用触发器包括：`cron`、`pollSCM`、`upstream`。

------

### .1 基本信息

- **必需**：否
- **参数**：无
- **允许位置**：顶层 `pipeline` 块中，只能出现一次

------

### .2 可用触发器类型

| 触发器       | 描述                                                         | 示例                                                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **cron**     | 定义固定间隔自动触发 Pipeline，使用 Jenkins cron 语法        | `triggers { cron('H */4 * * 1-5') }`                         |
| **pollSCM**  | 定期检查源码变化，如果有新提交，则触发 Pipeline（仅 Jenkins 2.22+） | `triggers { pollSCM('H */4 * * 1-5') }`                      |
| **upstream** | 当上游作业完成并达到指定阈值时触发 Pipeline                  | `triggers { upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS) }` |

------

### .3 示例：cron 触发

```groovy
pipeline {
    agent any
    triggers {
        cron('H */4 * * 1-5')  // 每周一到周五，每 4 小时随机触发一次
    }
    stages {
        stage('示例阶段') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

------

### .4 Jenkins cron 语法

Jenkins cron 类似于标准 cron，包含 5 个字段（TAB 或空格分隔）：

```
MINUTE HOUR DOM MONTH DOW
```

| 字段   | 范围 | 描述                      |
| ------ | ---- | ------------------------- |
| MINUTE | 0-59 | 分钟                      |
| HOUR   | 0-23 | 小时                      |
| DOM    | 1-31 | 月份中的日                |
| MONTH  | 1-12 | 月份                      |
| DOW    | 0-7  | 星期几（0 和 7 表示周日） |

### 支持的符号

| 符号             | 描述                                        |
| ---------------- | ------------------------------------------- |
| `*`              | 表示所有可能值                              |
| `M-N`            | 表示范围                                    |
| `M-N/X` 或 `*/X` | 步进间隔，每 X 单位执行一次                 |
| `A,B,…,Z`        | 枚举多个值                                  |
| `H`              | hash 值，自动均衡调度，避免同时触发过多作业 |

> H 可以与范围或步进结合使用，例如：
>  `H H(0-7) * * *` → 每天凌晨 0-7 点之间随机触发一次
>  `H/15 * * * *` → 每 15 分钟触发一次，使用 hash 均衡负载

------

### .5 常用 cron 示例

| 说明                                         | cron 表达式                                |
| -------------------------------------------- | ------------------------------------------ |
| 每 15 分钟触发一次                           | `triggers { cron('H/15 * * * *') }`        |
| 每小时前半段，每 10 分钟触发一次             | `triggers { cron('H(0-29)/10 * * * *') }`  |
| 工作日 9:45 到 15:45，每两小时触发一次       | `triggers { cron('45 9-16/2 * * 1-5') }`   |
| 工作日 9-16 点，每两小时触发一次（随机分钟） | `triggers { cron('H H(9-16)/2 * * 1-5') }` |
| 每月 1 号和 15 号触发（除 12 月外）          | `triggers { cron('H H 1,15 1-11 *') }`     |

------

### 总结

- `triggers` 可以自动触发 Pipeline，减少手动操作
- **cron**：定时触发
- **pollSCM**：检查源码更新触发
- **upstream**：上游作业完成触发
- Jenkins cron 支持 `H` 符号均衡负载，推荐使



好的，我将 **`tools` 指令** 内容整理成中文易懂文档，并附示例。

------

## tools（工具）

`tools` 指令用于 **在 Pipeline 中指定需要的构建工具**，Jenkins 会自动安装并将其加入 PATH 环境变量。

> 如果顶层或阶段 agent 为 `none`，该指令将被忽略。

------

### 1 基本信息

- **必需**：否
- **参数**：无
- **允许位置**：顶层 `pipeline` 块或阶段块中

------

### 2 支持的工具

| 工具       | 描述                 |
| ---------- | -------------------- |
| **maven**  | Apache Maven         |
| **jdk**    | Java Development Kit |
| **gradle** | Gradle 构建工具      |

> 注意：工具名称必须在 Jenkins 管理界面 **Manage Jenkins → Tools** 中提前配置好。

------

### 3 示例：顶层工具配置

```groovy
pipeline {
    agent any
    tools {
        maven 'apache-maven-3.0.1'  // 使用 Jenkins 已配置的 Maven
    }
    stages {
        stage('示例阶段') {
            steps {
                sh 'mvn --version'
            }
        }
    }
}
```

**说明**：

1. `tools { maven 'apache-maven-3.0.1' }`：指定 Maven 版本
2. Jenkins 会在执行 Pipeline 前自动安装该工具，并将其路径加入 PATH
3. `sh 'mvn --version'` 可以直接使用工具命令

------

### 4 阶段级工具配置（可选）

工具也可以在单独阶段中指定：

```groovy
stage('构建阶段') {
    tools {
        jdk 'OpenJDK-17'
        maven 'apache-maven-3.8.7'
    }
    steps {
        sh 'java -version'
        sh 'mvn clean package'
    }
}
```

- **优先级**：阶段级工具会覆盖 Pipeline 顶层工具配置
- 可以同时指定多种工具



好的，我将 **`input` 指令** 整理成中文易懂文档，并附示例。

------

##  input（输入）

`input` 指令用于 **在阶段执行时提示用户输入或确认**。

- 阶段会在应用完选项、进入 agent 或评估 when 条件前暂停，等待用户批准。
- 用户批准后，阶段继续执行。
- 输入的参数会在阶段剩余步骤中作为环境变量可用。

------

### 1 配置选项

| 参数                   | 是否必需 | 描述                                                   |
| ---------------------- | -------- | ------------------------------------------------------ |
| **message**            | 必需     | 提示用户的消息                                         |
| **id**                 | 可选     | 输入标识符，默认根据阶段名生成                         |
| **ok**                 | 可选     | 输入表单的“确定”按钮文字                               |
| **submitter**          | 可选     | 允许提交输入的用户或外部组，逗号分隔，默认允许所有用户 |
| **submitterParameter** | 可选     | 如果提供，将设置一个环境变量保存提交者用户名           |
| **parameters**         | 可选     | 提示用户输入的参数列表（支持 string、booleanParam 等） |

------

### 2 示例：Input Step

```groovy
pipeline {
    agent any
    stages {
        stage('示例阶段') {
            input {
                message "是否继续执行？"
                ok "是的，继续"
                submitter "alice,bob"
                parameters {
                    string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: '我应该向谁问好？')
                }
            }
            steps {
                echo "Hello, ${PERSON}, 很高兴见到你。"
            }
        }
    }
}
```

**说明**：

1. 阶段会暂停并显示提示信息 `"是否继续执行？"`
2. 仅 `alice` 和 `bob` 可以批准输入
3. 用户可以输入参数 `PERSON`，阶段中通过 `${PERSON}` 使用
4. 点击 `"是的，继续"` 后，阶段继续执行

------

### 3 使用场景

- 部署阶段确认：生产环境部署前手动确认
- 动态参数输入：根据用户选择或输入调整后续执行逻辑
- 结合 `when` 条件，实现更智能的阶段控制

------

##  when（条件执行）

`when` 指令用于 **判断阶段是否执行**，根据指定条件决定是否进入该阶段。

- 阶段内可以包含一个或多个条件。
- 多个条件默认是 **allOf**（全部满足才执行）。
- 可使用 **not、allOf、anyOf** 构建复杂条件。
- 条件可以指定在 **agent、input、options** 前执行。

------

### 1 基本信息

- **必需**：否
- **参数**：无
- **允许位置**：阶段（`stage`）内

------

### 2 内置条件

| 条件              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| **branch**        | 当构建的分支匹配指定模式时执行（多分支 Pipeline）            |
| **buildingTag**   | 当构建的是 Tag 时执行                                        |
| **changelog**     | SCM 变更日志包含指定正则时执行                               |
| **changeset**     | SCM 变更集中包含指定文件时执行                               |
| **changeRequest** | 当前构建为 Pull Request/Merge Request 时执行，可过滤 author、branch、target 等 |
| **environment**   | 当指定环境变量等于给定值时执行                               |
| **equals**        | 当实际值等于期望值时执行                                     |
| **expression**    | 当 Groovy 表达式返回 true 时执行                             |
| **tag**           | TAG_NAME 变量匹配指定模式时执行                              |
| **not**           | 当嵌套条件为 false 时执行                                    |
| **allOf**         | 当所有嵌套条件为 true 时执行                                 |
| **anyOf**         | 当至少一个嵌套条件为 true 时执行                             |
| **triggeredBy**   | 当当前构建由指定触发器触发时执行                             |

------

### 3 高级执行顺序控制

| 参数              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| **beforeAgent**   | true 时，在进入 agent 前评估条件（默认在 agent 后评估）      |
| **beforeInput**   | true 时，在 input 前评估条件（优先级高于 beforeAgent）       |
| **beforeOptions** | true 时，在 options 前评估条件（优先级高于 beforeInput 和 beforeAgent） |

------

### 4 示例

#### 示例 15：单条件

```groovy
stage('Example Deploy') {
    when { branch 'production' }
    steps { echo 'Deploying' }
}
```

#### 示例 16：多条件（默认 allOf）

```groovy
stage('Example Deploy') {
    when {
        branch 'production'
        environment name: 'DEPLOY_TO', value: 'production'
    }
    steps { echo 'Deploying' }
}
```

#### 示例 17：嵌套 allOf

```groovy
stage('Example Deploy') {
    when {
        allOf {
            branch 'production'
            environment name: 'DEPLOY_TO', value: 'production'
        }
    }
    steps { echo 'Deploying' }
}
```

#### 示例 18：嵌套 anyOf

```groovy
stage('Example Deploy') {
    when {
        branch 'production'
        anyOf {
            environment name: 'DEPLOY_TO', value: 'production'
            environment name: 'DEPLOY_TO', value: 'staging'
        }
    }
    steps { echo 'Deploying' }
}
```

#### 示例 19：表达式条件 + 嵌套

```groovy
stage('Example Deploy') {
    when {
        expression { BRANCH_NAME ==~ /(production|staging)/ }
        anyOf {
            environment name: 'DEPLOY_TO', value: 'production'
            environment name: 'DEPLOY_TO', value: 'staging'
        }
    }
    steps { echo 'Deploying' }
}
```

#### 示例 20：beforeAgent

```groovy
stage('Example Deploy') {
    agent { label 'some-label' }
    when {
        beforeAgent true
        branch 'production'
    }
    steps { echo 'Deploying' }
}
```

#### 示例 21：beforeInput

```groovy
stage('Example Deploy') {
    when { beforeInput true; branch 'production' }
    input { message "Deploy to production?"; id "simple-input" }
    steps { echo 'Deploying' }
}
```

#### 示例 22：beforeOptions

```groovy
stage('Example Deploy') {
    when { beforeOptions true; branch 'testing' }
    options { lock label: 'testing-deploy-envs', quantity: 1, variable: 'deployEnv' }
    steps { echo "Deploying to ${deployEnv}" }
}
```

#### 示例 23：triggeredBy

```groovy
stage('Example Deploy') {
    when { triggeredBy "TimerTrigger" }
    steps { echo 'Deploying' }
}
```



## 三种复杂 stage 结构

 **Sequential / Parallel / Matrix** 的结构示意图

```markdown
流水线 Pipeline
├── agent（代理）
├── options（选项）
├── environment（环境变量）
├── stages（阶段）
│   ├── 阶段 1（普通阶段）
│   │   └── steps（步骤）
│   │       └── …
│   ├── 阶段 2（顺序阶段 Sequential）
│   │   └── stages（嵌套阶段）
│   │       ├── 子阶段 2.1
│   │       │   └── steps
│   │       ├── 子阶段 2.2
│   │       │   └── steps
│   │       └── 子阶段 2.3（顺序内的并行 Parallel）
│   │           └── parallel（并行阶段）
│   │               ├── 并行阶段 A
│   │               │   └── steps
│   │               └── 并行阶段 B
│   │                   └── steps
│   ├── 阶段 3（并行阶段 Parallel）
│   │   └── parallel（并行）
│   │       ├── 并行阶段 X
│   │       │   └── steps
│   │       ├── 并行阶段 Y
│   │       │   └── steps
│   │       └── 并行阶段 Z（并行内的顺序 Sequential）
│   │           └── stages
│   │               ├── 子阶段 Z1
│   │               └── 子阶段 Z2
│   └── 阶段 4（矩阵阶段 Matrix）
│       └── matrix（矩阵）
│           ├── axes（轴）
│           │   ├── 轴1: 值1, 值2
│           │   └── 轴2: 值A, 值B
│           ├── stages（每个单元按顺序执行的阶段）
│           │   ├── 单元阶段 1
│           │   └── 单元阶段 2
│           └── excludes（排除的单元，可选）
└── post（后置步骤）
```




| 类型       | 特点                   | 每个 stage 可选块                  |
| ---------- | ---------------------- | ---------------------------------- |
| Sequential | 子 stage 顺序执行      | steps / stages / parallel / matrix |
| Parallel   | 子 stage 并行执行      | steps / stages / parallel / matrix |
| Matrix     | 多维组合 cell 并行执行 | steps / stages / parallel / matrix |



1. **阶段类型限制**：每个阶段只能有 `steps`、`stages`、`parallel` 或 `matrix` 中的一个。
2. **嵌套规则**：
   - Parallel 或 Matrix 内不能再嵌套同类型的 parallel 或 matrix。
   - 顺序阶段（Sequential）内可以包含 parallel 或 matrix。
3. **failFast**：
   - Parallel / Matrix 可设置 failFast，某个子任务失败会中断整个并行或矩阵执行。



### 1 Sequential Stages

- **概念**：在一个 stage 内部使用 `stages` 来顺序执行多个子 stage。
- **规则**：
  - 每个 stage 只能有 **一个**：`steps`、`stages`、`parallel` 或 `matrix`。
  - 嵌套的 stage 可以使用 agent、tools、when 等功能。
- **示例**：

```groovy
stage('Sequential') {
    stages {
        stage('Step 1') { steps { echo 'Step 1' } }
        stage('Step 2') { steps { echo 'Step 2' } }
    }
}
```

------

### 2 Parallel Stages

- **概念**：在一个 stage 内部使用 `parallel` 来同时执行多个子 stage。
- **规则**：
  - 同样每个 stage 只能有一个 steps、stages、parallel 或 matrix。
  - 可以使用 `failFast true`，某个并行任务失败时，全部并行任务会被中断。
  - 可在 pipeline 级别使用 `parallelsAlwaysFailFast()`。
- **示例**：

```groovy
stage('Parallel Stage') {
    parallel {
        stage('A') { steps { echo 'A' } }
        stage('B') { steps { echo 'B' } }
    }
}
```

------

### 3 Matrix

- **概念**：定义多维矩阵组合（cell），每个 cell 可以运行自己的 stage 序列。
- **规则**：
  - 必须包含 `axes`（定义维度和取值）和 `stages`。
  - 可使用 `excludes` 移除某些组合。
  - 每个 stage 只能有 steps、stages、parallel 或 matrix。
  - 可控制 failFast 或使用 pipeline 级别 `parallelsAlwaysFailFast()`。
- **示例**（一个轴三 cell）：

```groovy
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
    }
    stages {
        stage('Build') { steps { echo "Building on ${PLATFORM}" } }
    }
}
```
