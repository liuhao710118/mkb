在 **Jenkins Pipeline** 中，`agent` 是一个非常核心的概念，用来定义 **Pipeline 或特定阶段（stage）在哪台节点（Node）上执行**。它决定了流水线的运行环境。

------

## 🧩 一、`agent` 的作用

简单来说：

> `agent` 用来指定 **Pipeline 任务要在哪执行**，可以是 Jenkins 主节点（Master）或某个从节点（Agent/Slave）。

每个 `agent` 都会启动一个执行环境（workspace），在其中运行 Pipeline 的步骤（steps）。

------

## 🧱 二、`agent` 的使用位置

| 位置                     | 作用范围                     |
| ------------------------ | ---------------------------- |
| 顶层（在 pipeline 块中） | 整个流水线都使用相同的 agent |
| stage 中                 | 仅该 stage 使用独立的 agent  |

示例：

```groovy
pipeline {
    agent any  // 整个流水线都在一个 agent 上运行

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
    }
}
```

或者每个 stage 自定义：

```groovy
pipeline {
    agent none  // 顶层不分配 agent

    stages {
        stage('Build') {
            agent { label 'build-node' }
            steps {
                echo 'Building on build-node...'
            }
        }
        stage('Test') {
            agent { docker 'python:3.10' }
            steps {
                sh 'pytest'
            }
        }
    }
}
```

------

## ⚙️ 三、`agent` 的常用类型

### 1. `agent any`

表示 Jenkins 可以在任何可用节点上运行。

```groovy
agent any
```

### 2. `agent none`

表示顶层 Pipeline 不分配默认 agent，必须在每个 stage 中单独指定。

```groovy
agent none
```

### 3. `agent { label 'xxx' }`

指定只在具有某标签的节点上执行。

```groovy
agent { label 'linux' }
```

### 4. `agent { node { label 'build' } }`

功能与上面类似，旧写法：

```groovy
agent {
    node {
        label 'build'
    }
}
```

### 5. `agent { docker 'image-name' }`

在 Docker 容器中运行 Pipeline。

```groovy
agent { docker 'maven:3.9.4-jdk-17' }
```

你也可以添加挂载、环境变量等：

```groovy
agent {
    docker {
        image 'python:3.10'
        args '-v /data:/data'
        reuseNode true
    }
}
```

### 6. `agent { dockerfile true }`

从项目根目录下的 `Dockerfile` 构建镜像并运行。

```groovy
agent {
    dockerfile {
        filename 'Dockerfile'
        dir 'build'
        additionalBuildArgs '--no-cache'
    }
}
```

### 7. `agent { kubernetes { ... } }`

在 Kubernetes 集群中动态创建 Pod 来运行（需要 Jenkins Kubernetes 插件）。

```groovy
agent {
    kubernetes {
        yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: build
            image: maven:3.9.4-jdk-17
            command:
            - cat
            tty: true
        '''
    }
}
```

------

## 🧠 四、组合使用示例

### 全局 + 局部 agent 混用

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            agent { docker 'python:3.10' }
            steps {
                sh 'pytest'
            }
        }
        stage('Deploy') {
            agent { label 'deploy-server' }
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

------

## 📘 五、执行逻辑总结

| 配置方式                         | 执行环境     | 典型用途     |
| -------------------------------- | ------------ | ------------ |
| `agent any`                      | 任意节点     | 简单流水线   |
| `agent none` + 每个 stage 自定义 | 各自独立节点 | 分布式构建   |
| `agent { docker ... }`           | 容器环境     | 隔离构建环境 |
| `agent { label ... }`            | 特定节点     | 资源控制     |
| `agent { kubernetes ... }`       | 动态 pod     | 云原生构建   |

