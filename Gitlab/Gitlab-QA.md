---
title: "Gitlab QA"
date: "2020-07-23"
categories:
    - "技术"
tags:
    - "Gitlab"
    - "QA"
toc: false
indent: false
original: false
draft: true
---

## 更新记录

| 时间       | 内容                           |
| ---------- | ------------------------------ |
| 2020-06-03 | 初稿                           |
| 2020-07-23 | 从容器解决方案移出，独立为本篇 |

## Gitlab QA

### Q1

``` txt
    Q：gitlab接收一个push event触发构建，这个是监控所有的分支吗，分支模型是怎么样的

    A：不是的，按需。我们内部分支模型大概有四种，dev——>test——>release——>master。master以外的为了效率都会做自动触发
```

### Q2

``` txt
    Q：为什么不直接用gitlab-runner而接jenkins

    A：gitlab-runner 需要每个仓库都配置构建信息，当需要统一修改构建的时候很麻烦
```

### Q3

``` txt
    Q：持续集成系统具体的细节可以透露下吗？基于gitlab CI，jekins？或者小公司可以直接用Spinnaker这些吗？

    A：CI/CD 的话因为我们有自己现有的发布平台，背后的原理实际上还是调用jenkins去处理
```

### Q4

``` txt
    Q：和gitlab CI相比有什么优势

    A： 和 gitlab ci 相比的优势可以参考下 jenkins 与 jenkins X的对比。在用户角度来说，以应用为视角使用起来会更加方便，也方便利用社区资源。从架构和可维护性来说，Jenkins X 的架构会相对更加先进（与诞生年代有直接关系)。
```

### Q5

``` txt
    Q： 目前我们使用的gitlab-ci-runner 部署于k8s之外实现ci/cd。发现gitlab-ci在实际使用中，经常会遇到卡死报错。请问下，相比jenkins 做ci/cd 是会有什么优势，之前并没有使用过jenkins.

    A：gitlab-ci生产环境中，我们也没有使用，我们调研的结果是 1、有侵入性 2、pipeline功能较弱，但是有一个好处是遇到错误好像还可以继续执行。jenkins遇到错误会中断流程。
```

### Q6

``` txt
    Q：请问Jenkins webhook那些构建参数如何传入GitLab触发？

    A：webhook的触发和界面参数会有一些区别，我们在脚本里面做了处理。
```

### Q7

``` txt
    Q：离线部署，是不是通过打出镜像压缩包，然后带着镜像包到现场部署的容器云平台上，上传部署的方式？

    A：是在家里打出镜像压缩包，然后到现场解压出来，根据镜像类型进行处理，比如一些基础镜像，会直接上传到节点，业务的镜像会在部署完成后上传到Harbor，然后节点从Harbor去拉取。
```

### Q8

``` txt
    Q：GitLab CI 与 Jenkins和GitLab结合的CI，该如何选择？想知道更深层次的理解。

    A：还是要结合自己团队的实际情况做选择。从成熟度来说，肯定是 Jenkins用户最多，成熟度最高，缺点是侧重 Java，配置相对繁琐。GitLab CI相对简单，可以用 yaml，和 GitLab 结合的最好，但功能肯定没有 Jenkins全面。如果是小团队新项目，GitLab CI 又已经可以满足需求的话，并不需要上Jenkins，如果是较大的团队，又是偏 Java 的，个人更偏向 Jenkins。
```

### Q9

``` txt
    Q：有了Gerrit，为什么还要GitLab，Gerrit也可以托管代码啊？

    A：这个是有历史背景的，我们是先选择使用GitLab做代码托管，后期才加入Gerrit做code review。Gerrit在代码review方面比GitLab的merge request要方便许多，更适合企业内部使用。关于这个，我的想法是，要么将GitLab迁移到Gerrit，要么不用Gerrit，可以使用GitLab的merge request来进行review，那GitLab其实是可以不要的。
```

### Q10

``` txt
Q：公司环境较复杂：包含Java项目、PHP项目，Java项目目前大多是SpringBoot框架，PHP是ThinkPHP框架，项目架构并不复杂，有少许Java项目需要用Redis到Memcached、缓存机制。最大问题的是多，项目应该如何较好的依托Kubernetes顺利架构，将项目可持续集成？

A：我们的Redis这一类中间件还放在VM上，目前尚未打算搬移到Kubernetes上，Kubernetes+Docker天然是跨平台的，PHP也可以支持，并且对容器集群（既应用集群）管理非常出色，包含部分自动化运维，并不会因多种开发语言而增加负担，持续集成是另外一块，目前各大CI工具厂商也都支持Kubernetes，比较友好，我们采用的是GitLab-CI。
```

### Q11

``` txt
Q：SonarQube的权限控制及性能当面？

A：权限控制使用SonarQube提供的API，将项目跟GitLab中相应项目权限匹配起来，GitLab中可以查看这个项目代码，那么SonarQube中就能看到这个项目结果和Code。
```

### Q12

``` txt
Q: 你们是直接将SonarQube、GitLab/Jenkins的权限控制到一起了？怎样做的统一？

A：使用LDAP认证。
```

### Q13

``` txt
Q：Git Checkout的时候，你们的Git SCM没有考虑隐私安全的事情吗，比如代码权限受限？

A：Jenkins使用了一个最小权限用户去GitLab上拉代码。安全方面，Jenkins所有节点都是可控的。
```

### Q14

``` txt
Q：Jenkins的持续集成是怎么实现的？比如不同的源码仓库的提交触发，如GitHub、GitLab版本号怎么控制的？

A：Jenkins的CI流程触发可以有很多种，代码提交触发，定时触发，手动触发。版本号的控制也可以有很多方案，比如使用job的编号，使用Git的commit号，使用时间戳等等。
```

### Q15

``` txt
Q：请问，我们是java项目，在业务代码打成war包后，war包很大的情况下，在发布流程中，如何完成pod中的容器的代码更新，是采用挂载代码后重启容器方式，还是采用每次重新构建代码镜像，直接更新容器，或者有什么更好的建议吗

A：配置分离（上配置中心)，参数通过启动鉴权下载配置文件启动，这样子环境的更新只需要基于通过一个包即可。
```

### Q16

``` txt
Q：一个Job生成所有的Docker镜像，如果构建遇到问题，怎么去追踪这些记录？

A：在项目前期接入时，生成镜像的流程都作了宣传和推广。标准化的流程，会减少产生问题的机率。如果在构建中遇到问题，Prism4k的界面中，会直接有链接到本次建的次序号。点击链接，可直接定位到Console输出。
```

### Q17

``` txt
Q：Job和dind如何配合去实现打包镜像的呢？

A：首先是dind技术，通过挂载宿主机的docker client和dockersock，可以实现在容器内调用宿主机的Docker来做一些事情，这里我们主要就用于build。Kubernetes的Job则是用于执行这个构建worker的方式，利用Kubernetes的Job来调度构建任务，充分利用测试集群的空闲资源。
```

### Q18

``` txt
Q：请问下Maven的settings.xml怎么处理？本地Maven仓库呢？

A：我们构建了私有的 Maven 镜像， 私有镜像中是默认使用了我们的私有源。 对于项目中用户无需关注 settings.xml 中是否配置repo。
```

### Q19

``` txt
Q：生成新的镜像怎么自动打新的tag？

A：我们镜像Tag使用本次构建选定的Git版本，如分支名称或者Tag。
```

### Q20

``` txt
Q： 如何动态生成Dockerfile，如何在Docker镜像里配置JVM参数？

A：Dockerfile文件：我们是使用sh脚本生成的，将内容 >> Dockerfile中；JVM参数是在应用中配置的，发送构建消息时，作为消息内容送过去。
```

### Q21

``` txt
Q：Docker 的正确的使用姿势，在本地环境已经构建了企业私有 Registry Harbor，那么我要构建基于业务的应用时，是先从 Linux 系列的像 Ubuntu 或 CentOS 的 Base 的 Docker 镜像开始，然后通过 Dockerfile 定制业务需求，来使用吗？

A：我们基础镜像统一采用 CentOS 6.8，不同的业务有不同的 Dockerfile 模板，生成镜像的过程业务对 Dockerfile 是透明的。
```

### Q22

``` txt
Q：使用Pipeline先构建编译环境镜像，再编译，是否会导致整个流程需要很长时间？是否有优化方案？

A：编译镜像由于不会经常变动，因此这个镜像的构建通常使用cache就能直接完成，另外我们也把编译环境镜像打包这个步骤抽出来单独作为job执行了，这样在实际编译流程中就无需再进行编译环境构建。
```

### Q23

``` txt
Q：Docker存储考虑过Overlay当时吗？据说这种构建镜像比较快。

A：考虑过，当时也做过各个方面的测试，这种增量式的构建，肯定最快，但是我们需要有专人从源码级别对其进行维护，成本对于我们还是有点高，我们后期打算采用环境和代码分离的方式，即环境部署一次，代码多次部署来提升效率。
```

### Q24

``` txt
Q：您提到不过分强调测试自动化，尽量不改变测试流程，那么对于自动构建和单元测试的自动化有没有考虑呢？毕竟这些是比较消耗人力的部分。

A：自动构建我认为比较现实，单元测试有考虑。不过我们测试案例过于复杂，目前看短期实现不太现实。而且性能也是个问题，如果下一步要做我们会更多考虑一些特定场景。比如产品发布后的回归测试，这个有可能，但不会是普遍应用。
```

### Q25

``` txt
Q：自动化构建过程中，对应用的测试是怎么实现的？

A：单元测试可以在编译的时候完成，功能测试需要启动部署。
```

### Q26

``` txt
Q：通过镜像的构建脚本是怎么生成镜像的？在基础镜像上执行相关脚本么？一些端口存储卷环境变量这些镜像中的信息是怎么解决的？

A：我们对Dockerfile进行了封装，业务和开发人员不需要关心Dockerfile语法，直接写一个镜像构建脚本，最后根据一定的规则由Harbor生成Dockerfile，之后调用docker build去生成镜像。在这个过程中， 镜像的名称，版本都已经根据规则生成
```

### Q27

``` txt
Q：在构建时候，这些环境可以提前安装好？

A：应用里都有自己的版本概念，每个应用版本里有：镜像版本，环境变量、 export、Volmue等信息，所以在回退或者升级时候，最终的表现形式就是杀掉旧容器，根据版本的参数创建新容器。
```

### Q28

``` txt
Q：请问构建一次平均要多长时间？

A：现在Java、Dubbo、Python、go的多， 一般2分钟，而且有的镜像用户开启了自动构建后，在他们没意识的过程中，都已经构建完成。 到时候升级时候，选择对应的镜像版本即可。
```

### Q29

``` txt
Q：App的每一次提交都是一个version吗，是不是每次构建完测试完成，就可以发布了？

A：App 没有提交的概念，您说的应该是镜像，我们设计的是一个镜像对应一个Git仓库以及分支。当有push或者tag操作后，会自动触发构建，构建的行为是根据用户写的镜像构建shell脚本来决定的。 一般我们建议业务部门做出的镜像跟测试环境和生成环境没关系。 镜像就是镜像，只有应用有测试环境和生产环境。
```

> 参考链接:  
> 1、出自 DockerOne 社区问答  
>