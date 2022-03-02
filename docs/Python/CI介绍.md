#### 关于持续集成

`持续集成(Continuous Integration, CI)`是一种需要频繁提交代码到共享仓库的软件实践。频繁提交代码能较早检测到错误，减少在查找错误来源时开发者需要调试的代码量。频繁的代码更新也更便于软件开发团队的不同成语啊和并更改。这使开发者可以将更多时间用于编写代码，而减少在调试错误或解决合并冲突上所花费的时间。

提交代码到仓库时，可以持续创建并测试代码，以确保提交未引入错误。可以自定义`CI`的检查项，包含：`代码语法检查`、`安全性检查`、`代码覆盖率`、`功能测试`、`自定义检查项`等。

创建和测试代码需要服务器。可以在推送代码到仓库前在本地创建并测试更新，也可以使用CI服务器检查仓库中的新提交代码。

#### 使用GitHub Actions的持续集成

**GitHub** 中的`Actions` 同样提供了`CI`的功能，通过`Actions`可以在仓库构建代码并运行测试的工作流程。同时，**GitHub** 提供了虚拟机可以用来完成`CI`的工作流程。

可以配置`CI`工作流程在**GitHub**时间发生时执行(例如：当推送代码到仓库时)、按设定的时间表运行，或者在使用仓库分发web挂钩的外部事件发生时运行。

**GitHub**运行CI测试并在拉取请求中提供每次测试的结果，如果CI测试通过，推送的更改可供团队成员审查或合并；如果测试失败，则需要根据失败的原因进行修改。

##### 设置CI

`对仓库具有写入权限的任何人都可以使用 GitHub Actions 设置持续集成 (CI)。`

- 进入GitHub主页面，在仓库名称下，选择`Actions`
- 找到与对应编程语言对应的模版，点击`Set up this workflow(设置此工作流程)`
- 选择后单击`Start commit(开始提交)`
- 页面底部，输入`commit`信息，描述此事件的更改信息
- 在提交消息字段下面，选择将提交添加到`当前分支`还是`新建一个分支`
- 选择`Propose new file(提议新文件)`

##### 构建Python项目

使用持续集成(CI)工作流程来构建和测试Python项目。

如下为`Python 工作流程模版`：

```yaml
name: Python package

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [2.7, 3.5, 3.6, 3.7, 3.8]

    steps:
      - uses: actions/checkout@v2
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v2
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install flake8 pytest
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
      - name: Lint with flake8
        run: |
          # stop the build if there are Python syntax errors or undefined names
          flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
          # exit-zero treats all errors as warnings. The GitHub editor is 127 chars wide
          flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
      - name: Test with pytest
        run: |
          pytest
```





##### 跳过工作流程运行

如果想暂时阻止触发工作流程，可以对提交消息天假跳过指令。例如，在**推送代码**或**拉取代码**想暂时跳过CI的工作流程，可以在**推送代码**或**拉取代码**的命令指令中添加以下任何的字符：

- `[skip ci]`
- `[ci skip]`
- `[no ci]`
- `[skip actions]`
- `[actions skip]`

或者，使用`两个空行`后接`skip-checks: true` 或 `skip-checks:true`来跳过CI工作流程。

##### 工作流程运行通知

可以为`GitHub Actions`配置启用电子邮件或web通知，则在触发的工作流程运行完成时，会收到通知(包含工作流程运行的状态(成功、失败、取消和跳过))。

工作流程的通知默认会发送给最初创建该工作流程的用户。同时如果有其他用户更新了工作流程，后续通知则将会发送给该用户。

