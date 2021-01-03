命令行测试执行器
================

PHPUnit 命令行测试执行器可通过 phpunit 命令调用。下面的代码展示了如何用
PHPUnit 命令行测试执行器来运行测试：

上面这个调用例子中，PHPUnit 命令行测试执行器将在当前工作目录中寻找
ArrayTest.php 源文件并加载之。而在此源文件中应当能找到 `ArrayTest`
测试用例类，此类中的测试将被执行。

对于每个测试的运行，PHPUnit 命令行工具输出一个字符来指示进展：

`.`

> 当测试成功时输出。

`F`

> 当测试方法运行过程中一个断言失败时输出。

`E`

> 当测试方法运行过程中产生一个错误时输出。

`R`

> 当测试被标记为有风险时输出（参见risky-tests）。

`S`

> 当测试被跳过时输出（参见incomplete-and-skipped-tests）。

`I`

> 当测试被标记为不完整或未实现时输出（参见incomplete-and-skipped-tests）。

PHPUnit 区分*失败（failure）*与*错误（error）*。失败指的是被违背了的
PHPUnit 断言，例如一个失败的 `assertSame()`
调用。错误指的是意料之外的异常（exception）或 PHP
错误。这种差异已被证明在某些时候是非常有用的，因为错误往往比失败更容易修复。如果得到了一个非常长的问题列表，那么最好先对付错误，当错误全部修复了之后再试一次瞧瞧还有没有失败。

命令行选项
----------

让我们来瞧瞧以下代码中命令行测试执行器的各种选项：

`phpunit UnitTest`

> 运行由 `UnitTest` 类提供的测试。这个类应当在 UnitTest.php
> 源文件中声明。
>
> `UnitTest` 这个类必须满足以下二个条件之一：要么它继承自
> `PHPUnit\Framework\TestCase`；要么它提供 `public static suite()`
> 方法，这个方法返回一个 `PHPUnit\Framework\Test` 对象，比如，一个
> `PHPUnit\Framework\TestSuite` 类的实例。

`phpunit UnitTest UnitTest.php`

> 运行由 `UnitTest` 类提供的测试。这个类应当在指定的源文件中声明。

`--coverage-clover`

> 为运行的测试生成带有代码覆盖率信息的 XML
> 格式的日志文件。更多细节参见code-coverage-analysis。

`--coverage-crap4j`

> 生成 Crap4j
> 格式的代码覆盖率报告。更多细节请参见code-coverage-analysis。

`--coverage-html`

> 生成 HTML 格式的代码覆盖率报告。更多细节请参见code-coverage-analysis。

`--coverage-php`

> 生成一个序列化后的 PHP\_CodeCoverage 对象，此对象含有代码覆盖率信息。

`--coverage-text`

> 为运行的测试以人们可读的格式生成带有代码覆盖率信息的日志文件或命令行输出。

`--log-junit`

> 为运行的测试生成 JUnit XML 格式的日志文件。

`--testdox-html` 和 `--testdox-text`

> 为运行的测试以 HTML 或纯文本格式生成敏捷文档（参见 textui.testdox）。

`--filter`

> 只运行名称与给定模式匹配的测试。如果模式未用定界符包住，PHPUnit 将用
> `/` 定界符来将其包住。
>
> 测试名称将以以下格式之一进行匹配：
>
> `TestNamespace\TestCaseClass::testMethod`
>
> > 默认的测试名称格式等价于在测试方法内使用 `__METHOD__` 魔术常量。
>
> `TestNamespace\TestCaseClass::testMethod with data set #0`
>
> > 当测试拥有数据供给器时，数据的每轮迭代都会将其当前索引附加在默认测试名称结尾处。
>
> `TestNamespace\TestCaseClass::testMethod with data set "my named data"`
>
> > 当测试拥有使用命名数据集的数据供给器时，数据的每轮迭代都会将当前名称附加在默认测试名称结尾处。命名数据集的例子参见textui.examples.TestCaseClass.php。
>
> >     <?php
> >     use PHPUnit\Framework\TestCase;
> >
> >     namespace TestNamespace;
> >
> >     class TestCaseClass extends TestCase
> >     {
> >         /**
> >          * @dataProvider provider
> >          */
> >         public function testMethod($data)
> >         {
> >             $this->assertTrue($data);
> >         }
> >
> >         public function provider()
> >         {
> >             return [
> >                 'my named data' => [true],
> >                 'my data'       => [true]
> >             ];
> >         }
> >     }
>
> `/path/to/my/test.phpt`
>
> > 对于 PHPT 测试，其测试名称是文件系统路径。
>
> 有效的过滤器模式例子参见textui.examples.filter-patterns。
>
>     --filter 'TestNamespace\\TestCaseClass::testMethod'
>     --filter 'TestNamespace\\TestCaseClass'
>     --filter TestNamespace
>     --filter TestCaseClase
>     --filter testMethod
>     --filter '/::testMethod .*"my named data"/'
>     --filter '/::testMethod .*#5$/'
>     --filter '/::testMethod .*#(5|6|7)$/'
>
> 在匹配数据供给器时有一些额外的快捷方式，参见textui.examples.filter-shortcuts。
>
>     --filter 'testMethod#2'
>     --filter 'testMethod#2-4'
>     --filter '#2'
>     --filter '#2-4'
>     --filter 'testMethod@my named data'
>     --filter 'testMethod@my.*data'
>     --filter '@my named data'
>     --filter '@my.*data'

`--testsuite`

> 只运行名称与给定模式匹配的测试套件。

`--group`

> 只运行来自指定分组（可以多个）的测试。可以用 `@group`
> 标注为测试标记其所属的分组。
>
> `@author` 和 `@ticket` 标注都是 `@group`
> 的别名，分别允许基于作者和事务 ID 筛选测试。

`--exclude-group`

> 排除来自指定分组（可以多个）的测试。可以用 `@group`
> 标注为测试标记其所属的分组。

`--list-groups`

> 列出所有有效的测试分组。

`--test-suffix`

> 只查找文件名以指定后缀（可以多个）结尾的测试文件。

`--dont-report-useless-tests`

> 不报告事实上不测试任何内容的测试。详情参见risky-tests。

`--strict-coverage`

> 更严格对待意外的代码覆盖。详情参见risky-tests。

`--strict-global-state`

> 更严格对待全局状态篡改。详情参见risky-tests。

`--disallow-test-output`

> 更严格对待测试执行期间产生的输出。详情参见risky-tests。

`--disallow-todo-tests`

> 不执行文档注释块中含有 `@todo` 标注的测试。

`--enforce-time-limit`

> 根据测试规模对其加上执行时长限制。详情参见risky-tests。

`--process-isolation`

> 每个测试都在独立的 PHP 进程中运行。

`--no-globals-backup`

> 不要备份与还原 $GLOBALS。更多细节请参见fixtures.global-state。

`--static-backup`

> 备份与还原用户定义的类中的静态属性。更多细节请参见fixtures.global-state。

`--colors`

> 使用彩色输出。在 Windows 上，用
> [ANSICON](https://github.com/adoxa/ansicon) 或
> [ConEmu](https://github.com/Maximus5/ConEmu)。
>
> 本选项有三个可能的值：
>
> -
>
> > `never`：完全不使用彩色输出。当未使用 `--colors`
> > 选项时，这是默认值。
>
> -
>
> > `auto`：如果当前终端不支持彩色、或者输出被管道输出至其他命令、或输出被重定向至文件时，不使用彩色输出，其余情况使用彩色。
>
> -
>
> > `always`：总是使用彩色输出，即使当前终端不支持彩色、输出被管道输出至其他命令、或输出被重定向至文件。
>
> 当使用了 `--colors` 选项但未指定任何值时，将选择 `auto` 做为其值。

`--columns`

> 定义输出所使用的列数。如果将其值定义为
> `max`，则使用当前终端所支持的最大列数。

`--stderr`

> 选择输出到 `STDERR` 而非 `STDOUT`。

`--stop-on-error`

> 首次错误出现后停止执行。

`--stop-on-failure`

> 首次错误或失败出现后停止执行。

`--stop-on-risky`

> 首次碰到有风险的测试时停止执行。

`--stop-on-skipped`

> 首次碰到跳过的测试时停止执行。

`--stop-on-incomplete`

> 首次碰到不完整的测试时停止执行。

`--verbose`

> 输出更详尽的信息，例如不完整或者跳过的测试的名称。

`--debug`

> 输出调试信息，例如当一个测试开始执行时输出其名称。

`--loader`

> 指定要使用的 `PHPUnit\Runner\TestSuiteLoader` 实现。
>
> 标准的测试套件加载器将在当前工作目录和 PHP 的 `include_path`
> 配置指令中指定的每个目录内查找源文件。诸如 `Project_Package_Class`
> 这样的类名对应的源文件名为 Project/Package/Class.php。

`--repeat`

> 将测试重复运行指定次数。

`--testdox`

> 以 TestDox 格式报告测试进度。（参见 textui.testdox）。

`--printer`

> 指定要使用的结果输出器（printer）。输出器类必须扩展
> `PHPUnit\Util\Printer` 并且实现 `PHPUnit\Framework\TestListener`
> 接口。

`--bootstrap`

> 在测试前先运行一个 "bootstrap" PHP 文件。

`--configuration`、`-c`

> 从 XML 文件中读取配置信息。更多细节请参见 appendixes.configuration。
>
> 如果 phpunit.xml 或
> phpunit.xml.dist（按此顺序）存在于当前工作目录并且*未*使用
> `--configuration`，将自动从此文件中读取配置。
>
> 如果指定了目录且在此目录中存在 phpunit.xml 或
> phpunit.xml.dist（按此顺序）将自动从此文件中读取配置。

`--no-configuration`

> 忽略当前工作目录下的 phpunit.xml 与 phpunit.xml.dist。

`--include-path`

> 向 PHP 的 include\_path 开头添加指定路径（可以多个）。

`-d`

> 设置指定的 PHP 配置选项的值。

注

请注意，选项不能放在参数之后。

TestDox
-------

PHPUnit 的 TestDox
功能着眼于测试类及其所有测试方法的名称，将它们驼峰式大小写（camel
case）（或蛇式大小写（snake\_case））拼写的 PHP
名称转换为句子：`testBalanceIsInitiallyZero()`（或
`test_balance_is_initially_zero()`）转化为“Balance is initially
zero”。如果有多个测试方法的名字互相之间的差异只是一个或多个数字的后缀，例如
`testBalanceCannotBecomeNegative()` 和
`testBalanceCannotBecomeNegative2()`，假如所有这些测试都成功，句子“Balance
cannot become negative”只会出现一次。

来看一下从 `BankAccount` 类生成的敏捷文档：

另外，敏捷文档也能以 HTML 或纯文本格式生成并写入文件中，用
`--testdox-html` 和 `--testdox-text` 参数即可。

敏捷文档可以用于将对项目所使用的外部包所做出的假设文档化。使用外部包，你就暴露于这个包的行为与你所预期的不同的风险中，并且包的未来版本可能在你所不知道的情况下有微妙的改变并破坏你的代码。每次做出假设时就编写一个对应的测试可以处理这些风险。如果测试成功，那么假设就有效。如果所有的假设都通过测试来文档化，外部包在未来发布新版本就不会引起忧虑：如果测试成功，那么系统就应当能继续正常运作。
