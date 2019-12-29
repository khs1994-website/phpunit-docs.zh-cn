未完成的测试与跳过的测试
========================

未完成的测试
------------

开始写新的测试用例类时，可能想从写下空测试方法开始，比如：

    public function testSomething()
    {
    }

以此来跟踪需要编写的测试。空测试的问题是 PHPUnit
框架会将它们解读为成功。这种错误解读导致错误报告变得毫无用处——无法分辨出测试是真的成功了还是根本就未编写实现。在未实现的测试中调用
`$this->fail()`
同样没啥帮助，因为测试将被解读为失败。这和将未实现的测试解读为成功是一样的错误。

假如把成功的测试视为绿灯、测试失败视为红灯，那么还额外需要黄灯来将测试标记为未完成或尚未实现。`PHPUnit_Framework_IncompleteTest`
是一个标记接口，用于将测试方法抛出的异常标记为测试未完成或目前尚未实现而导致的结果。`PHPUnit_Framework_IncompleteTestError`
是这个接口的标准实现。

incomplete-and-skipped-tests.incomplete-tests.examples.SampleTest.php展示了一个测试用例类
`SampleTest`，它有一个测试方法
`testSomething()`。通过在测试方法中调用便捷方法
`markTestIncomplete()`（会自动抛出一个
`PHPUnit_Framework_IncompleteTestError` 异常）将这个测试标记为未完成。

    <?php
    use PHPUnit\Framework\TestCase;

    class SampleTest extends TestCase
    {
        public function testSomething()
        {
            // 可选：如果愿意，在这里随便测试点什么。
            $this->assertTrue(true, '这应该已经是能正常工作的。');

            // 在这里停止，并将此测试标记为未完成。
            $this->markTestIncomplete(
              '此测试目前尚未实现。'
            );
        }
    }
    ?>

在 PHPUnit 命令行测试执行器的输出中，未完成的测试记为 `I`，如下例所示：

    $ phpunit --verbose SampleTest
    PHPUnit 7.0.0 by Sebastian Bergmann and contributors.

    I

    Time: 0 seconds, Memory: 3.95Mb

    There was 1 incomplete test:

    1) SampleTest::testSomething
    This test has not been implemented yet.

    /home/sb/SampleTest.php:12
    OK, but incomplete or skipped tests!
    Tests: 1, Assertions: 1, Incomplete: 1.

incomplete-and-skipped-tests.incomplete-tests.tables.api列举了用于将测试标记为未完成的
API。

跳过测试
--------

并非所有测试都能在任何环境中运行。比如说，考虑这样一种情况：一个数据库抽象层，针对其所支持的各种数据库系统有多个不同的驱动程序。针对
MySQL 驱动程序的测试当然只在 MySQL 服务器可用才能运行。

incomplete-and-skipped-tests.skipping-tests.examples.DatabaseTest.php
展示了一个测试用例类 `DatabaseTest`，它有一个测试方法
`testConnection()`。在测试用例类的 `setUp()`模板方法中，检查了 MySQLi
扩展是否可用，并且在扩展不可用时用 `markTestSkipped()`
方法来跳过此测试。

    <?php
    use PHPUnit\Framework\TestCase;

    class DatabaseTest extends TestCase
    {
        protected function setUp()
        {
            if (!extension_loaded('mysqli')) {
                $this->markTestSkipped(
                  'MySQLi 扩展不可用。'
                );
            }
        }

        public function testConnection()
        {
            // ...
        }
    }
    ?>

在 PHPUnit 命令行测试执行器的输出中，被跳过的测试记为 `S`，如下例所示：

    $ phpunit --verbose DatabaseTest
    PHPUnit 7.0.0 by Sebastian Bergmann and contributors.

    S

    Time: 0 seconds, Memory: 3.95Mb

    There was 1 skipped test:

    1) DatabaseTest::testConnection
    The MySQLi extension is not available.

    /home/sb/DatabaseTest.php:9
    OK, but incomplete or skipped tests!
    Tests: 1, Assertions: 0, Skipped: 1.

incomplete-and-skipped-tests.skipped-tests.tables.api列举了用于跳过测试的
API。

用 @requires 来跳过测试
-----------------------

除了上述方法，还可以用 `@requires`
标注来表达测试用例的一些常见前提条件。

    <?php
    use PHPUnit\Framework\TestCase;

    /**
     * @requires extension mysqli
     */
    class DatabaseTest extends TestCase
    {
        /**
         * @requires PHP 5.3
         */
        public function testConnection()
        {
            // 测试要求有 mysqli 扩展，并且 PHP >= 5.3
        }

        // ... 所有其他要求有 mysqli 扩展的测试
    }
    ?>

如果使用了某种在特定版本的 PHP 下无法编译的语法，请在此章节内查找 XML
配置信息中关于版本依赖的信息：appendixes.configuration.testsuites
