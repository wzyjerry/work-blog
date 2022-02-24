# Gitlab CI/CD

默认配置下每个存储库都开启了`CI/CD`服务，当检测到存储库根目录包含`.gitlab-ci.yml`文件时，如果本次提交的`commit`信息不包含`[ci skip]``或``[skip ci]`，`Gitlab`就会启动`pipeline`。

每个`pipeline`根据配置都会由若干个`job`组成，每个`job`都由特定的执行器执行`gitlab-runner`，每个执行器运行在特定的机器上，可以使用`tags`指定一个具体的执行器。`runner`可以配置以多种方式执行`job`定义的脚本，目前所有使用的`runner`的执行方式都是`docker`。在这种模式下，每个`job`都会在一个干净的独立镜像中执行，镜像名由`image`指定。`job`开始时会拉取当前存储库，切换到对应分支，设置工作目录为存储库根目录，并注入环境变量。接下来按`script`节定义的命令由`sh`解释执行，如果过程中有任意非`0`返回值则`job`失败；否则阶段成功。

一个典型的`.gitlab-ci.yml`如下所示

```yml
stages: # (1)
- build
- deploy

build: # (2)
  image: docker.aminer.cn/docker/golang:1.16.10 # (3)
  stage: build # (4)
  variables:
    GIT_SUBMODULE_STRATEGY: recursive # (5)
  script:
    - mv main.go main.template
    - export VERSION=`cat version`.${CI_PIPELINE_IID}
    - envsubst '$VERSION' < main.template > main.go
    - go mod tidy
    - go get github.com/swaggo/swag/cmd/swag
    - swag init
    - go build -ldflags "-s -X 'main.BuildDate=`TZ="Asia/Shanghai" date`' -X 'main.Version=$VERSION'" -o ./bin/$CI_PROJECT_NAME # (6)
    - cp -r data bin/data
    - docker build -t $CI_REGISTRY/$CI_PROJECT_PATH:$VERSION .
    - docker login -u $CI_DEPLOY_USER -p $CI_DEPLOY_PASSWORD $CI_REGISTRY
    - docker push $CI_REGISTRY/$CI_PROJECT_PATH:$VERSION # (7)
  tags: # (8)
    - dev

deploy_local:
  image: docker.aminer.cn/docker/kubectl:1.22.3
  stage: deploy
  script:
    - export VERSION=`cat version`.${CI_PIPELINE_IID}
    - export CONFIG=local
    - export HOST=aminer-org.private.aminer.cn
    - export KUBECONFIG=/etc/kubernetes/admin.conf
    - envsubst '$VERSION$CI_PROJECT_NAME$CONFIG$HOST$CI_REGISTRY$CI_PROJECT_PATH' < config/local/k8s.yml | kubectl apply -n data-middle-platform -f - # (9)
  only: # (10)
    - master
  tags:
    - dev
```

- (1) `stages`定义了两个`pipeline`阶段：`build`和`deploy`，这些阶段会按顺序执行

- (2) 一个`job`，名称可以自定义，会作为`pipeline`和`job`的显示名称

- (3) `job`的执行镜像，这里使用`golang:1.16.10`构建`go`应用程序

- (4) `job`对应的阶段，与`stages`定义元组的某一项对应

- (5) 使用`submodule`时需要设定的环境变量，指示`runner`递归拉取所有的子模块

- (6) 设置参数构建可执行文件

- (7) 推送到`gitlab`同名的`docker`容器存储库

- (8) 指明该`job`可以被哪个`runner`执行，这里`dev`是`runner`的`tag`

- (9) 将配置应用到`k8s`集群，使用`kubectl apply -n <ns> -f -`从`stdin`读取配置

- (10) 指明该`job`在哪个分支有效，因此可以通过相同`stage`指定不同`only`分环境部署

`runner`会按顺序执行`stages`定义每个阶段的所有满足条件的`job`，如果`job`可以被多个`runner`执行，选择一个；否则`job`被阻塞。

## 附录

### 手动阶段

```yml
when:
  manual
```
