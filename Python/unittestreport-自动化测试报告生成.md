#### 前言

对于自动化测试来说，如何能把测试结果更直观更有效的展示出来一直是测试人员的一块心病。目前主流的有很多生成测试报告的第三方库，例如，`HTMLTestRunner`、`BeautifulReport`、`Allure`，等等。不过这些库使用起来各有利弊，总之没有一个库能既满足实用又满足美观。

最近看到一个unittest框架配套的测试报告生成插件，可以做到与unittest框架无缝衔接，并且支持多种格式的自动化测试报告。



#### 什么是unittestreport？

`unittestreport`是基于unittest开发的一个功能扩展库，开发之初，开发人员只是计划开发一个unittest生成html测试报告的模块，因此命名为`unittestreport`(听起来很接地气，有没有...😌)。目前已经迭代了数个版本，其提供的功能也越来越丰富。目前已经支持的功能有：

- HTML测试报告生成
- unittest数据驱动
- 失败用例rerun
- 多线程并发
- 测试结果推送(邮箱、钉钉、企业微信)



#### 安装

```bash
pip install unittestreport

pip install pytest-testreport   # 支持pytest生成报告
```



#### 生成测试报告

`unittestreport`中封装了一个`TestRunner`类，可以用来代替unittest中的`TextTestRunner`来执行测试用例，执行完测试用例后会自动生成测试报告。并且支持各种测试报告的风格。

❕❕不过实际测试发现，PyCharm中执行的用例不会生成测试报告，只有在终端执行的测试用例会生成测试报告。

```python
import unittest
from unittestreport import TestRunner

class MyTestCase(unittest.TestCase):

    def setUp(self):
        """用例初始化等操作"""
        self.num1 = 24
        self.num2 = 2

    def tearDown(self):
        """恢复用例执行环境"""

    def test_div(self):
        self.assertEqual(self.num1 / self.num2, 12)

    def test_add(self):
        self.assertEqual(self.num1 + self.num2, 22)

    def test_any(self):
        self.assertEqual(self.num1 % self.num2, 5)


if __name__ == '__main__':
    case = unittest.defaultTestLoader.discover("./")
    runner = TestRunner(case)
    runner.run()
```

`TestRunner`类创建测试报告时，可以通过指定测试参数生成自定义的测试报告。

- `suites`：测试用例套，测试集

- `filename`：测试报告文件名

- `report_dir`：测试报告存放路径

- `title`：测试报告的标题名称

- `templates`：指定生成测试报告的模板类型(etc: 1、2、3)

  - 测试报告类型1

    ![style1](../../img/%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A%E6%A8%A1%E6%9D%BF%E7%B1%BB%E5%9E%8B1-6610536.png)

  -  测试报告类型2

    ![style2](../../img/%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A%E6%A8%A1%E6%9D%BF%E7%B1%BB%E5%9E%8B2-6610562.png)

  - 测试报告类型3

    ![style3](../../img/%E6%B5%8B%E8%AF%95%E6%8A%A5%E5%91%8A%E6%A8%A1%E6%9D%BF%E7%B1%BB%E5%9E%8B3-6610580.png)

- `tester`： 测试人员名称

```python
suite = unittest.defaultTestLoader.discover(r'C:\project\open_class\Py0507\testcase')

# 2、创建一个用例运行程序
runner = unittestreport.TestRunner(suite,
                                   tester='测试人员—小柠檬',
                                   filename="test",
                                   report_dir=".",
                                   title='这里设置报告标题',
                                   desc='项目测试生成的报告描述',
                                   templates=2
                                   )

# 3、运行测试用例
runner.run()
```



#### 失败用例rerun

新的`unittestreport`中对`rerun`方法进行了优化，测试时指定`count`和`interval`即可将执行失败的用例多次执行。

- `count`：指定用例失败后重新运行的次数
- `interval`：指定每次重新执行用例的时间间隔

```python
runner = TestRunner(suite=suite)
runner.run(count=3, interval=2)
```



#### 测试报告发送邮件

目前邮件接口只支持`465`和`25`两个端口



#### 数据驱动

> 数据驱动的目的是将测试数据和用例逻辑进行分离，提高代码的重用率，以及用例的维护.

使用方法

```python
from unittestreport import ddt, list_data,json_data,yaml_data
```

关于数据驱动本，unittestreport.dataDriver模块中实现了三个使用方法，支持使用列表(可迭代对象)、json文件、yaml文件来生成测试用例.

- List_data：用例数据保存在可迭代对象（列表）中时使用

  ```python
  from unittestreport import ddt, list_data
  @ddt
  class TestClass(unittest.TestCase):
      cases = [{'title': '用例1', 'data': '用例参数', 'expected': '预期结果'}, 
               {'title': '用例2', 'data': '用例参数', 'expected': '预期结果'},
               {'title': '用例3', 'data': '用例参数', 'expected': '预期结果'}]
      
      @list_data(cases)
      def test_case(self, data):
          pass
  ```

- Json_data：用例数据保存在json文件中时使用

  ```python
  from unittestreport import ddt,json_data
  
  @ddt
  class TestClass(unittest.TestCase):
      @json_data('C:/xxx/xxx.json')
      def test_case(self, data):
          pass
  ```

- Yaml_data：用例数据保存在json文件中时使用

  ```python
  from unittestreport import ddt,yaml_data
  
  @ddt
  class TestClass(unittest.TestCase):
      @yaml_data("C:/xxxx/xxx/cases.yaml")
      def test_case(self, data):
          pass
  ```

##### 注意点：

* 关于使用ddt的时候进行数据驱动，指定测试报告中的用例描述：
* 测试报告中的用例描述默认使用的是用例方法的文档字符串注释，
* 如果要给每一条用例添加用例描述，需要在用例数据中添加title或者desc字段，字段对应的数据会自动设置为测试报告中用例的描述



#### 并发执行用例

`unittestreport`中同样提供了对并发执行用例的支持，使用`TestRunner.run()`方法执行测试用例时，加上`thread_count`参数即可执行运行用例时开启的线程数量。

```python
runner = TestRunner(suite=suite)
runner.run(thread_count=3)
```

:warning: 使用并发方式执行用例时，需要考虑以下几个问题

- 用例执行之间是否有依赖关系
- 用例之间是否会有修改公共资源(全局变量)的情况

