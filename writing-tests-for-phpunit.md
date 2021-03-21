编写 PHPUnit 测试
=================

`writing-tests-for-phpunit.examples.StackTest.php` 展示了如何用 PHPUnit
编写测试来对 PHP 数组操作进行测试。此示例介绍了用 PHPUnit
编写测试的基本惯例与步骤：

\#.

> 针对类 `Class` 的测试写在类 `ClassTest` 中。

\#.

> `ClassTest`（通常）继承自 `PHPUnit\Framework\TestCase`。

\#.

> 测试都是命名为 `test*` 的公用方法。
>
> 也可以在方法的文档注释块（docblock）中使用 `@test`
> 标注将其标记为测试方法。

\#.

> 在测试方法内，类似于
> `assertSame()`（参见`appendixes.assertions`）这样的断言方法用来对实际值与预期值的匹配做出断言。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class StackTest extends TestCase
    {
        public function testPushAndPop(): void
        {
            $stack = [];
            $this->assertSame(0, count($stack));

            array_push($stack, 'foo');
            $this->assertSame('foo', $stack[count($stack)-1]);
            $this->assertSame(1, count($stack));

            $this->assertSame('foo', array_pop($stack));
            $this->assertSame(0, count($stack));
        }
    }

> *Martin Fowler*：
>
> 当你想把一些东西写到 `print`
> 语句或者调试表达式中时，别这么做，改为将其写成测试。

测试的依赖关系
--------------

> *Adrian Kuhn et. al.*：
>
> 单元测试主要是作为一种良好实践来编写的，它能帮助开发人员识别并修复
> bug、重构代码，还可以看作被测软件单元的文档。要实现这些好处，理想的单元测试应当覆盖程序中所有可能的路径。一个单元测试通常覆盖一个函数或方法中的一个特定路径。但是，测试方法不一定是封装良好的独立实体。测试方法之间经常有隐含的依赖关系暗藏在测试的实现方案中。

PHPUnit支持对测试方法之间的显式依赖关系进行声明。这种依赖关系并不是定义在测试方法的执行顺序中，而是允许生产者（producer）返回一个测试基境（fixture）的实例，并将此实例传递给依赖于它的消费者（consumer）们。

-   生产者（producer），是能生成被测单元并将其作为返回值的测试方法。

-   消费者（consumer），是依赖于一个或多个生产者及其返回值的测试方法。

`writing-tests-for-phpunit.examples.StackTest2.php` 展示了如何用
`@depends` 标注来表示测试方法之间的依赖关系。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class StackTest extends TestCase
    {
        public function testEmpty(): array
        {
            $stack = [];
            $this->assertEmpty($stack);

            return $stack;
        }

        /**
         * @depends testEmpty
         */
        public function testPush(array $stack): array
        {
            array_push($stack, 'foo');
            $this->assertSame('foo', $stack[count($stack)-1]);
            $this->assertNotEmpty($stack);

            return $stack;
        }

        /**
         * @depends testPush
         */
        public function testPop(array $stack): void
        {
            $this->assertSame('foo', array_pop($stack));
            $this->assertEmpty($stack);
        }
    }

在上例中，第一个测试`testEmpty()`
创建了一个新数组，并断言其为空。随后，此测试将此基境作为结果返回。第二个测试
`testPush()` 依赖于
`testEmpty()`，并将所依赖的测试之结果作为参数传入。最后，`testPop()`
依赖于 `testPush()`。

注

默认情况下，生产者所产生的返回值将“原样”传递给相应的消费者。这意味着，如果生产者返回的是一个对象，那么传递给消费者的将是指向此对象的引用。但同样也可以（a）通过
`@depends clone` 来传递指向（深）拷贝对象的引用，或（b）通过
`@depends shallowClone` 来传递指向（正常浅）克隆对象（基于 PHP 关键字
`clone`）的引用。

为了定位缺陷，我们希望把注意力集中于相关的失败测试上。这就是为什么当某个测试所依赖的测试失败时，PHPUnit
会跳过这个测试。利用测试之间的依赖关系可以改进缺陷定位，如`writing-tests-for-phpunit.examples.DependencyFailureTest.php`
所示。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class DependencyFailureTest extends TestCase
    {
        public function testOne(): void
        {
            $this->assertTrue(false);
        }

        /**
         * @depends testOne
         */
        public function testTwo(): void
        {
        }
    }

$ phpunit --verbose DependencyFailureTest PHPUnit .0 by Sebastian
Bergmann and contributors.

FS

Time: 0 seconds, Memory: 5.00Mb

There was 1 failure:

1) DependencyFailureTest::testOne Failed asserting that false is true.

/home/sb/DependencyFailureTest.php:6

There was 1 skipped test:

1) DependencyFailureTest::testTwo This test depends on
"DependencyFailureTest::testOne" to pass.

FAILURES! Tests: 1, Assertions: 1, Failures: 1, Skipped: 1.

测试可以使用多个 `@depends` 标注。PHPUnit
不会更改测试的运行顺序，因此你需要自行保证某个测试所依赖的所有测试均出现于这个测试之前。

拥有多个 `@depends`
标注的测试，其第一个参数是第一个生产者提供的基境，第二个参数是第二个生产者提供的基境，以此类推。参见`writing-tests-for-phpunit.examples.MultipleDependencies.php`

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class MultipleDependenciesTest extends TestCase
    {
        public function testProducerFirst(): string
        {
            $this->assertTrue(true);

            return 'first';
        }

        public function testProducerSecond(): string
        {
            $this->assertTrue(true);

            return 'second';
        }

        /**
         * @depends testProducerFirst
         * @depends testProducerSecond
         */
        public function testConsumer(string $a, string $b): void
        {
            $this->assertSame('first', $a);
            $this->assertSame('second', $b);
        }
    }

$ phpunit --verbose MultipleDependenciesTest PHPUnit .0 by Sebastian
Bergmann and contributors.

...

Time: 0 seconds, Memory: 3.25Mb

OK (3 tests, 4 assertions)

数据供给器
----------

测试方法可以接受任意参数。这些参数由一个或多个数据供给器方法（在`writing-tests-for-phpunit.data-providers.examples.DataTest.php`
中，是 `additionProvider()` 方法）提供。用 `@dataProvider`
标注来指定要使用的数据供给器方法。

数据供给器方法必须声明为
`public`，其返回值要么是一个数组，其每个元素也是数组；要么是一个实现了
`Iterator`
接口的对象，在对它进行迭代时每步产生一个数组。每个数组都是测试数据集的一部分，将以它的内容作为参数来调用测试方法。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class DataTest extends TestCase
    {
        /**
         * @dataProvider additionProvider
         */
        public function testAdd(int $a, int $b, int $expected): void
        {
            $this->assertSame($expected, $a + $b);
        }

        public function additionProvider(): array
        {
            return [
                [0, 0, 0],
                [0, 1, 1],
                [1, 0, 1],
                [1, 1, 3]
            ];
        }
    }

$ phpunit DataTest PHPUnit .0 by Sebastian Bergmann and contributors.

...F

Time: 0 seconds, Memory: 5.75Mb

There was 1 failure:

1) DataTest::testAdd with data set \#3 (1, 1, 3) Failed asserting that 2
is identical to 3.

/home/sb/DataTest.php:9

FAILURES! Tests: 4, Assertions: 4, Failures: 1.

当使用到大量数据集时，最好逐个用字符串键名对其命名，避免用默认的数字键名。这样输出信息会更加详细些，其中将包含打断测试的数据集所对应的名称。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class DataTest extends TestCase
    {
        /**
         * @dataProvider additionProvider
         */
        public function testAdd(int $a, int $b, int $expected): void
        {
            $this->assertSame($expected, $a + $b);
        }

        public function additionProvider(): array
        {
            return [
                'adding zeros'  => [0, 0, 0],
                'zero plus one' => [0, 1, 1],
                'one plus zero' => [1, 0, 1],
                'one plus one'  => [1, 1, 3]
            ];
        }
    }

$ phpunit DataTest PHPUnit .0 by Sebastian Bergmann and contributors.

...F

Time: 0 seconds, Memory: 5.75Mb

There was 1 failure:

1) DataTest::testAdd with data set "one plus one" (1, 1, 3) Failed
asserting that 2 is identical to 3.

/home/sb/DataTest.php:9

FAILURES! Tests: 4, Assertions: 4, Failures: 1.

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class DataTest extends TestCase
    {
        /**
         * @dataProvider additionProvider
         */
        public function testAdd(int $a, int $b, int $expected): void
        {
            $this->assertSame($expected, $a + $b);
        }

        public function additionProvider(): CsvFileIterator
        {
            return new CsvFileIterator('data.csv');
        }
    }

$ phpunit DataTest PHPUnit .0 by Sebastian Bergmann and contributors.

...F

Time: 0 seconds, Memory: 5.75Mb

There was 1 failure:

1) DataTest::testAdd with data set \#3 ('1', '1', '3') Failed asserting
that 2 is identical to 3.

/home/sb/DataTest.php:11

FAILURES! Tests: 4, Assertions: 4, Failures: 1.

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class CsvFileIterator implements Iterator
    {
        private $file;
        private $key = 0;
        private $current;

        public function __construct(string $file)
        {
            $this->file = fopen($file, 'r');
        }

        public function __destruct()
        {
            fclose($this->file);
        }

        public function rewind(): void
        {
            rewind($this->file);

            $this->current = fgetcsv($this->file);
            $this->key     = 0;
        }

        public function valid(): bool
        {
            return !feof($this->file);
        }

        public function key(): int
        {
            return $this->key;
        }

        public function current(): array
        {
            return $this->current;
        }

        public function next(): void
        {
            $this->current = fgetcsv($this->file);

            $this->key++;
        }
    }

如果测试同时从 `@dataProvider` 方法和一个或多个 `@depends`
测试接收数据，那么来自于数据供给器的参数将先于来自所依赖的测试的。来自于所依赖的测试的参数对于每个数据集都是一样的。参见`writing-tests-for-phpunit.data-providers.examples.DependencyAndDataProviderCombo.php`

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class DependencyAndDataProviderComboTest extends TestCase
    {
        public function provider(): array
        {
            return [['provider1'], ['provider2']];
        }

        public function testProducerFirst(): void
        {
            $this->assertTrue(true);

            return 'first';
        }

        public function testProducerSecond(): void
        {
            $this->assertTrue(true);

            return 'second';
        }

        /**
         * @depends testProducerFirst
         * @depends testProducerSecond
         * @dataProvider provider
         */
        public function testConsumer(): void
        {
            $this->assertSame(
                ['provider1', 'first', 'second'],
                func_get_args()
            );
        }
    }

$ phpunit --verbose DependencyAndDataProviderComboTest PHPUnit .0 by
Sebastian Bergmann and contributors.

...F

Time: 0 seconds, Memory: 3.50Mb

There was 1 failure:

1) DependencyAndDataProviderComboTest::testConsumer with data set \#1
('provider2') Failed asserting that two arrays are identical. ---
Expected +++ Actual @@ @@ Array &0 ( - 0 =&gt; 'provider1' + 0 =&gt;
'provider2' 1 =&gt; 'first' 2 =&gt; 'second' )
/home/sb/DependencyAndDataProviderComboTest.php:32

FAILURES! Tests: 4, Assertions: 4, Failures: 1.

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class DataTest extends TestCase
    {
        /**
         * @dataProvider additionWithNonNegativeNumbersProvider
         * @dataProvider additionWithNegativeNumbersProvider
         */
        public function testAdd(int $a, int $b, int $expected): void
        {
            $this->assertSame($expected, $a + $b);
        }

        public function additionWithNonNegativeNumbersProvider(): void
        {
            return [
                [0, 1, 1],
                [1, 0, 1],
                [1, 1, 3]
            ];
        }

        public function additionWithNegativeNumbersProvider(): array
        {
            return [
                [-1, 1, 0],
                [-1, -1, -2],
                [1, -1, 0]
            ];
        }
     }

$ phpunit DataTest PHPUnit .0 by Sebastian Bergmann and contributors.

..F... 6 / 6 (100%)

Time: 0 seconds, Memory: 5.75Mb

There was 1 failure:

1) DataTest::testAdd with data set \#3 (1, 1, 3) Failed asserting that 2
is identical to 3.

/home/sb/DataTest.php:12

FAILURES! Tests: 6, Assertions: 6, Failures: 1.

注

如果一个测试依赖于另外一个使用了数据供给器的测试，仅当被依赖的测试至少能在一组数据上成功时，依赖于它的测试才会运行。使用了数据供给器的测试，其运行结果是无法注入到依赖于此测试的其他测试中的。

注

所有数据供给器方法的执行都是在对 `setUpBeforeClass()`
静态方法的调用和第一次对 `setUp()`
方法的调用之前完成的。因此，无法在数据供给器中使用创建于这两个方法内的变量。这是必须的，这样
PHPUnit 才能计算测试的总数量。

对异常进行测试
--------------

`writing-tests-for-phpunit.exceptions.examples.ExceptionTest.php`
展示了如何用 `@expectException` 标注来测试被测代码中是否抛出了异常。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class ExceptionTest extends TestCase
    {
        public function testException(): void
        {
            $this->expectException(InvalidArgumentException::class);
        }
    }

$ phpunit ExceptionTest PHPUnit .0 by Sebastian Bergmann and
contributors.

F

Time: 0 seconds, Memory: 4.75Mb

There was 1 failure:

1) ExceptionTest::testException Failed asserting that exception of type
"InvalidArgumentException" is thrown.

FAILURES! Tests: 1, Assertions: 1, Failures: 1.

除了 `expectException()` 方法外，还有
`expectExceptionCode()`、`expectExceptionMessage()` 和
`expectExceptionMessageMatches()`
方法可以用于为被测代码所抛出的异常建立预期。

注

注意 `expectExceptionMessage()` 断言的是 `$actual` 讯息包含有
`$expected` 讯息，并不执行精确的字符串比较。

对 PHP 错误、警告和通知进行测试
-------------------------------

默认情况下，PHPUnit 将测试在执行中触发的 PHP
错误、警告、通知都转换为异常。先不说其他好处，这样就可以预期在测试中会触发
PHP
错误、警告或通知，如`writing-tests-for-phpunit.exceptions.examples.ErrorTest.php`
所示。

注

PHP 的 `error_reporting` 运行时配置会对 PHPUnit
将哪些错误转换为异常有所限制。如果在这个特性上碰到问题，请确认 PHP
的配置中没有抑制你所关注的错误类型。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class ErrorTest extends TestCase
    {
        public function testDeprecationCanBeExpected(): void
        {
            $this->expectDeprecation();

            // （可选）测试讯息和某个字符串相等
            $this->expectDeprecationMessage('foo');

            // 或者（可选）测试讯息和某个正则表达式匹配
            $this->expectDeprecationMessageMatches('/foo/');

            \trigger_error('foo', \E_USER_DEPRECATED);
        }

        public function testNoticeCanBeExpected(): void
        {
            $this->expectNotice();

            // （可选）测试讯息和某个字符串相等
            $this->expectNoticeMessage('foo');

            // 或者（可选）测试讯息和某个正则表达式匹配
            $this->expectNoticeMessageMatches('/foo/');

            \trigger_error('foo', \E_USER_NOTICE);
        }

        public function testWarningCanBeExpected(): void
        {
            $this->expectWarning();

            // （可选）测试讯息和某个字符串相等
            $this->expectWarningMessage('foo');

            // 或者（可选）测试讯息和某个正则表达式匹配
            $this->expectWarningMessageMatches('/foo/');

            \trigger_error('foo', \E_USER_WARNING);
        }

        public function testErrorCanBeExpected(): void
        {
            $this->expectError();

            // （可选）测试讯息和某个字符串相等
            $this->expectErrorMessage('foo');

            // 或者（可选）测试讯息和某个正则表达式匹配
            $this->expectErrorMessageMatches('/foo/');

            \trigger_error('foo', \E_USER_ERROR);
        }
    }

如果测试代码使用了会触发错误的 PHP 内建函数，比如
`fopen`，有时候在测试中使用错误抑制符会很有用。通过抑制住错误通知，就能对返回值进行检查，否则错误通知将会导致
PHPUnit 的错误处理程序抛出异常。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class ErrorSuppressionTest extends TestCase
    {
        public function testFileWriting(): void
        {
            $writer = new FileWriter;

            $this->assertFalse(@$writer->write('/is-not-writeable/file', 'stuff'));
        }
    }

    final class FileWriter
    {
        public function write($file, $content)
        {
            $file = fopen($file, 'w');

            if ($file === false) {
                return false;
            }

            // ...
        }
    }

$ phpunit ErrorSuppressionTest PHPUnit .0 by Sebastian Bergmann and
contributors.

.

Time: 1 seconds, Memory: 5.25Mb

OK (1 test, 1 assertion)

如果不使用错误抑制符，此测试将会失败，并报告
`fopen(/is-not-writeable/file): failed to open stream: No such file or directory`。

对输出进行测试
--------------

有时候，想要断言（比如说）某方法的运行过程中生成了预期的输出（例如，通过
`echo` 或 `print`）。`PHPUnit\Framework\TestCase` 类使用 PHP
的[输出缓冲](http://www.php.net/manual/en/ref.outcontrol.php)特性来为此提供必要的功能支持。

`writing-tests-for-phpunit.output.examples.OutputTest.php` 展示了如何用
`expectOutputString()`
方法来设定所预期的输出。如果没有产生预期的输出，测试将计为失败。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class OutputTest extends TestCase
    {
        public function testExpectFooActualFoo(): void
        {
            $this->expectOutputString('foo');

            print 'foo';
        }

        public function testExpectBarActualBaz(): void
        {
            $this->expectOutputString('bar');

            print 'baz';
        }
    }

$ phpunit OutputTest PHPUnit .0 by Sebastian Bergmann and contributors.

.F

Time: 0 seconds, Memory: 5.75Mb

There was 1 failure:

1) OutputTest::testExpectBarActualBaz Failed asserting that two strings
are equal. --- Expected +++ Actual @@ @@ -'bar' +'baz'

FAILURES! Tests: 2, Assertions: 2, Failures: 1.

`writing-tests-for-phpunit.output.tables.api`
中列举了用于对输出进行测试的各种方法

table

<table>
<caption>测试输出的方法</caption>
<thead>
<tr class="header">
<th>方法</th>
<th>含义</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>void expectOutputRegex(string $regularExpression)</code></td>
<td>设置输出预期为输出应当匹配正则表达式 <code>$regularExpression</code>。</td>
</tr>
<tr class="even">
<td><code>void expectOutputString(string $expectedString)</code></td>
<td>设置输出预期为输出应当与 <code>$expectedString</code> 相等。</td>
</tr>
<tr class="odd">
<td><code>bool setOutputCallback(callable $callback)</code></td>
<td>设置回调函数，用来做诸如将实际输出规范化之类的动作。</td>
</tr>
<tr class="even">
<td><code>string getActualOutput()</code></td>
<td>获取实际输出。</td>
</tr>
</tbody>
</table>

注

在严格模式下，本身产生输出的测试将会失败。

错误相关信息的输出
------------------

当有测试失败时，PHPUnit
全力提供尽可能多的有助于找出问题所在的上下文信息。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class ArrayDiffTest extends TestCase
    {
        public function testEquality(): void
        {
            $this->assertSame(
                [1, 2,  3, 4, 5, 6],
                [1, 2, 33, 4, 5, 6]
            );
        }
    }

$ phpunit ArrayDiffTest PHPUnit .0 by Sebastian Bergmann and
contributors.

F

Time: 0 seconds, Memory: 5.25Mb

There was 1 failure:

1) ArrayDiffTest::testEquality Failed asserting that two arrays are
identical. --- Expected +++ Actual @@ @@ Array ( 0 =&gt; 1 1 =&gt; 2 - 2
=&gt; 3 + 2 =&gt; 33 3 =&gt; 4 4 =&gt; 5 5 =&gt; 6 )

/home/sb/ArrayDiffTest.php:7

FAILURES! Tests: 1, Assertions: 1, Failures: 1.

在这个例子中，数组中只有一个值不同，但其他值也都同时显示出来，以提供关于错误发生的位置的上下文信息。

当生成的输出很长而难以阅读时，PHPUnit
将对其进行分割，并在每个差异附近提供少数几行上下文信息。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class LongArrayDiffTest extends TestCase
    {
        public function testEquality(): void
        {
            $this->assertSame(
                [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 2,  3, 4, 5, 6],
                [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 2, 33, 4, 5, 6]
            );
        }
    }

$ phpunit LongArrayDiffTest PHPUnit .0 by Sebastian Bergmann and
contributors.

F

Time: 0 seconds, Memory: 5.25Mb

There was 1 failure:

1) LongArrayDiffTest::testEquality Failed asserting that two arrays are
identical. --- Expected +++ Actual @@ @@ 11 =&gt; 0 12 =&gt; 1 13 =&gt;
2 - 14 =&gt; 3 + 14 =&gt; 33 15 =&gt; 4 16 =&gt; 5 17 =&gt; 6 )

/home/sb/LongArrayDiffTest.php:7

FAILURES! Tests: 1, Assertions: 1, Failures: 1.

### 边缘情况

当比较失败时，PHPUnit
为输入值建立文本表示，然后以此进行对比。这种实现导致在差异指示中显示出来的问题可能比实际上存在的多。

这种情况只出现在对数组或者对象使用 `assertEquals()`
或其他“弱”比较函数时。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class ArrayWeakComparisonTest extends TestCase
    {
        public function testEquality(): void
        {
            $this->assertEquals(
                [1, 2, 3, 4, 5, 6],
                ['1', 2, 33, 4, 5, 6]
            );
        }
    }

$ phpunit ArrayWeakComparisonTest PHPUnit .0 by Sebastian Bergmann and
contributors.

F

Time: 0 seconds, Memory: 5.25Mb

There was 1 failure:

1) ArrayWeakComparisonTest::testEquality Failed asserting that two
arrays are equal. --- Expected +++ Actual @@ @@ Array ( - 0 =&gt; 1 + 0
=&gt; '1' 1 =&gt; 2 - 2 =&gt; 3 + 2 =&gt; 33 3 =&gt; 4 4 =&gt; 5 5 =&gt;
6 )

/home/sb/ArrayWeakComparisonTest.php:7

FAILURES! Tests: 1, Assertions: 1, Failures: 1.

在这个例子中，第一个索引项中的 `1` 和 `'1'` 在报告中被视为不同，虽然
`assertEquals()` 认为这两个值是匹配的。
