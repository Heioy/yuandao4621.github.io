### 通过Gitlab配置CI

进入项目，选择`settings`-->`CI/CD`进入`CI/CD`配置界面

![Github-CI](../../../WorkSpace/notes/img/Gitlab-CI-1-6610345.png)

* #### General pipelines：

* #### Auto DevOps：

* #### Runners：

* #### Artifacts：

* #### Variables：

* #### Pipeline triggers：

* #### Deploy freezes：

#### Step 1: 配置 pipelines

点击`General pipelines`右侧的`expand`按钮，选择`Custom CI configuration path`、`Public pipelines`、`Test coverage parsing`进行配置。

* `Custom CI configuration path`：配置Gitlab CI执行的配置文件路径。默认会在项目根路径下创建一个`.gitlab-ci.yml`的配置文件，这个文件即CI的配置文件，因此，此处填写`.gitlab-ci.yml`文件名即可。若该文件不在项目根路径下，则需要加上路径。

  ![Gitlab-CI2](../../../WorkSpace/notes/img/Gitlab-CI-2-6610386.png)

* `Public pipelines`：pipeline 是Gitlab CI任务执行时的管道，通过pipeline可以看到任务执行的状态和输出。对于不同的项目，`Public pipelines`对于用户的可见度是不同的。这里配置保持默认。

  > 公开项目：任何人都可以看到`pipeline`的状态并且查看任务的输出结果
  >
  > 内部项目：仅登录的用户和指定用户可以看到`pipeline`的状态并且查看任务的输出结果
  >
  > 私有项目：仅项目成员(guest or higher)可以看到`pipeline`的状态并且查看任务的输出结果

* `Test coverage parsin`：测试代码的覆盖率。通过正则表达式进行匹配。官方给出了几种语言的匹配，如下

  > - Simplecov (Ruby) - `\(\d+.\d+\%\) covered`
  > - pytest-cov (Python) - `^TOTAL.+?(\d+\%)$`
  > - Scoverage (Scala) - `Statement coverage[A-Za-z\.*]\s*:\s*([^%]+)`
  > - phpunit --coverage-text --colors=never (PHP) - `^\s*Lines:\s*\d+.\d+\%`
  > - gcovr (C/C++) - `^TOTAL.*\s+(\d+\%)$`
  > - tap --coverage-report=text-summary (NodeJS) - `^Statements\s*:\s*([^%]+)`
  > - nyc npm test (NodeJS) - `All files[^|]*\|[^|]*\s+([\d\.]+)`
  > - excoveralls (Elixir) - `\[TOTAL\]\s+(\d+\.\d+)%`
  > - mix test --cover (Elixir) - `\d+.\d+\%\s+\|\s+Total`
  > - JaCoCo (Java/Kotlin) `Total.*?([0-9]{1,3})%`
  > - go test -cover (Go) `coverage: \d+.\d+% of statements`

配置完成后点击`Save changes`保存

#### Step 2: 配置`Runners`

  `Runners`是用于 GitLab CI/CD 执行任务的执行器。配置Runners前需要进行安装

* 安装1: 下载`Runner`安装包进行安装

  安装包下载地址：[Install GitLab Runner manually on GNU/Linux | GitLab](https://docs.gitlab.com/runner/install/linux-manually.html)

  ```bash
  # 以RedHat为例
  # Replace ${arch} with any of the supported architectures, e.g. amd64, arm, arm64
  # A full list of architectures can be found here https://gitlab-runner-downloads.s3.amazonaws.com/latest/index.html
  curl -LJO "https://gitlab-runner-downloads.s3.amazonaws.com/latest/rpm/gitlab-runner_${arch}.rpm"

* 安装2: 安装 || 更新

  ```bash
  # 安装
  rpm -i gitlab-runner_<arch>.rpm
  # 更新
  rpm -Uvh gitlab-runner_<arch>.rpm
  ```

* 安装3: 安装完成后查看安装版本

  ```bash
  which gitlab-runner && $(which gitlab-runner) -v
  ```

* 注册`Runner`

  ```bash
  # Step 1: "执行 sudo gitlab-runner register"
  # sudo gitlab-runner register
  Runtime platform                                    arch=amd64 os=linux pid=1517 revision=58ba2b95 version=14.2.0
  Running in system-mode.
  
  Enter the GitLab instance URL (for example, https://gitlab.com/):
  # Step 2: "URL 填写 项目CI配置页面下Runners 下的 URL"
  https://gitlab.woqutech.com/
  Enter the registration token:
  # Step 3: "token 同样填写CI配置页面下Runners 下的 token"
  tfA2fT7T7hiPJfGMsa2d
  Enter a description for the runner:
  # Step 4: "runner的描述，可以不填"
  [10-10-100-250]: QDataCloud-CI
  Enter tags for the runner (comma-separated):
  # Step 5: "runner执行对应的tag，不填"
  
  Registering runner... succeeded                     runner=tfA2fT7T
  Enter an executor: custom, docker, parallels, shell, virtualbox, docker-ssh, ssh, docker+machine, docker-ssh+machine, kubernetes:
  # Step 6: "选择一个执行器"
  custom
  Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
  
  ```

* 启动`runner`

  ```bash
  # 启动
  gitlab-runner start
  ## 重启
  gitlab-runner restart
  ```

  