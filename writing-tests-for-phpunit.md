编写 PHPUnit 测试
=================

writing-tests-for-phpunit.examples.StackTest.php 展示了如何用 PHPUnit
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
> `assertSame()`（参见appendixes.assertions）这样的断言方法用来对实际值与预期值的匹配做出断言。

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

|  
*Martin Fowler*：

当你想把一些东西写到 `print`
语句或者调试表达式中时，别这么做，改为将其写成测试。

测试的依赖关系
--------------

> *Adrian Kuhn et. al.*：
>
> 单元测试主要是作为一种良好实践来编写的，它能帮助开发人员识别并修复
> bug、重构代码，还可以看作被测软件单元的文档。要实现这些好处，理想的单元测试应当覆盖程序中所有可能的路径。一个单元测试通常覆盖一个函数或方法中的一个特定路径。但是，测试方法不一定是封装良好的独立实体。测试方法之间经常有隐含的依赖关系暗藏在测试的实现方案中。

PHPUnit支持对测试方法之间的显式依赖关系进行声明。这种依赖关系并不是定义在测试方法的执行顺序中，而是允许生产者（producer）返回一个测试基境（fixture）的实例，并将此实例传递给依赖于它的消费者（consumer）们。

<table>
<tbody>
<tr class="odd">
<td>生产者（producer），是能生成被测单元并将其作为返回值的测试方法。</td>
</tr>
<tr class="even">
<td>-</td>
</tr>
<tr class="odd">
<td>消费者（consumer），是依赖于一个或多个生产者及其返回值的测试方法。</td>
</tr>
<tr class="even">
<td>writing-tests-for-phpunit.examples.StackTest2.php 展示了如何用 <code>@depends</code> 标注来表示测试方法之间的依赖关系。</td>
</tr>
<tr class="odd">
<td>.. code-block:: php</td>
</tr>
<tr class="even">
<td>:caption: 用 <code>@depends</code> 标注来表示依赖关系</td>
</tr>
<tr class="odd">
<td>:name: writing-tests-for-phpunit.examples.StackTest2.php</td>
</tr>
<tr class="even">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="odd">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="even">
<td>final class StackTest extends TestCase</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>public function testEmpty(): array</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$stack = [];</td>
</tr>
<tr class="odd">
<td>$this-&gt;assertEmpty($stack);</td>
</tr>
<tr class="even">
<td>return $stack;</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>/**</td>
</tr>
<tr class="odd">
<td>* @depends testEmpty</td>
</tr>
<tr class="even">
<td>*/</td>
</tr>
<tr class="odd">
<td>public function testPush(array $stack): array</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>array_push($stack, 'foo');</td>
</tr>
<tr class="even">
<td>$this-&gt;assertSame('foo', $stack[count($stack)-1]);</td>
</tr>
<tr class="odd">
<td>$this-&gt;assertNotEmpty($stack);</td>
</tr>
<tr class="even">
<td>return $stack;</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>/**</td>
</tr>
<tr class="odd">
<td>* @depends testPush</td>
</tr>
<tr class="even">
<td>*/</td>
</tr>
<tr class="odd">
<td>public function testPop(array $stack): void</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>$this-&gt;assertSame('foo', array_pop($stack));</td>
</tr>
<tr class="even">
<td>$this-&gt;assertEmpty($stack);</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>在上例中，第一个测试<code>testEmpty()</code> 创建了一个新数组，并断言其为空。随后，此测试将此基境作为结果返回。第二个测试 <code>testPush()</code> 依赖于 <code>testEmpty()</code>，并将所依赖的测试之结果作为参数传入。最后，<code>testPop()</code> 依赖于 <code>testPush()</code>。</td>
</tr>
<tr class="even">
<td>.. admonition:: 注</td>
</tr>
<tr class="odd">
<td>默认情况下，生产者所产生的返回值将“原样”传递给相应的消费者。这意味着，如果生产者返回的是一个对象，那么传递给消费者的将是指向此对象的引用。但同样也可以（a）通过 <code>@depends clone</code> 来传递指向（深）拷贝对象的引用，或（b）通过 <code>@depends shallowClone</code> 来传递指向（正常浅）克隆对象（基于 PHP 关键字 <code>clone</code>）的引用。</td>
</tr>
<tr class="even">
<td>为了定位缺陷，我们希望把注意力集中于相关的失败测试上。这就是为什么当某个测试所依赖的测试失败时，PHPUnit 会跳过这个测试。利用测试之间的依赖关系可以改进缺陷定位，如writing-tests-for-phpunit.examples.DependencyFailureTest.php 所示。</td>
</tr>
<tr class="odd">
<td>.. code-block:: php</td>
</tr>
<tr class="even">
<td>:caption: 利用测试之间的依赖关系</td>
</tr>
<tr class="odd">
<td>:name: writing-tests-for-phpunit.examples.DependencyFailureTest.php</td>
</tr>
<tr class="even">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="odd">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="even">
<td>final class DependencyFailureTest extends TestCase</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>public function testOne(): void</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$this-&gt;assertTrue(false);</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>/**</td>
</tr>
<tr class="odd">
<td>* @depends testOne</td>
</tr>
<tr class="even">
<td>*/</td>
</tr>
<tr class="odd">
<td>public function testTwo(): void</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>.. parsed-literal::</td>
</tr>
<tr class="even">
<td>$ phpunit --verbose DependencyFailureTest</td>
</tr>
<tr class="odd">
<td>PHPUnit .0 by Sebastian Bergmann and contributors.</td>
</tr>
<tr class="even">
<td>FS</td>
</tr>
<tr class="odd">
<td>Time: 0 seconds, Memory: 5.00Mb</td>
</tr>
<tr class="even">
<td>There was 1 failure:</td>
</tr>
<tr class="odd">
<td>1) DependencyFailureTest::testOne</td>
</tr>
<tr class="even">
<td>Failed asserting that false is true.</td>
</tr>
<tr class="odd">
<td>/home/sb/DependencyFailureTest.php:6</td>
</tr>
<tr class="even">
<td>There was 1 skipped test:</td>
</tr>
<tr class="odd">
<td>1) DependencyFailureTest::testTwo</td>
</tr>
<tr class="even">
<td>This test depends on &quot;DependencyFailureTest::testOne&quot; to pass.</td>
</tr>
<tr class="odd">
<td>FAILURES!</td>
</tr>
<tr class="even">
<td>Tests: 1, Assertions: 1, Failures: 1, Skipped: 1.</td>
</tr>
<tr class="odd">
<td>测试可以使用多个 <code>@depends</code> 标注。PHPUnit 不会更改测试的运行顺序，因此你需要自行保证某个测试所依赖的所有测试均出现于这个测试之前。</td>
</tr>
<tr class="even">
<td>拥有多个 <code>@depends</code> 标注的测试，其第一个参数是第一个生产者提供的基境，第二个参数是第二个生产者提供的基境，以此类推。参见writing-tests-for-phpunit.examples.MultipleDependencies.php</td>
</tr>
<tr class="odd">
<td>.. code-block:: php</td>
</tr>
<tr class="even">
<td>:caption: 有多个依赖项的测试</td>
</tr>
<tr class="odd">
<td>:name: writing-tests-for-phpunit.examples.MultipleDependencies.php</td>
</tr>
<tr class="even">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="odd">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="even">
<td>final class MultipleDependenciesTest extends TestCase</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>public function testProducerFirst(): string</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$this-&gt;assertTrue(true);</td>
</tr>
<tr class="odd">
<td>return 'first';</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>public function testProducerSecond(): string</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>$this-&gt;assertTrue(true);</td>
</tr>
<tr class="even">
<td>return 'second';</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>/**</td>
</tr>
<tr class="odd">
<td>* @depends testProducerFirst</td>
</tr>
<tr class="even">
<td>* @depends testProducerSecond</td>
</tr>
<tr class="odd">
<td>*/</td>
</tr>
<tr class="even">
<td>public function testConsumer(string $a, string $b): void</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$this-&gt;assertSame('first', $a);</td>
</tr>
<tr class="odd">
<td>$this-&gt;assertSame('second', $b);</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>.. parsed-literal::</td>
</tr>
<tr class="odd">
<td>$ phpunit --verbose MultipleDependenciesTest</td>
</tr>
<tr class="even">
<td>PHPUnit .0 by Sebastian Bergmann and contributors.</td>
</tr>
<tr class="odd">
<td>...</td>
</tr>
<tr class="even">
<td>Time: 0 seconds, Memory: 3.25Mb</td>
</tr>
<tr class="odd">
<td>OK (3 tests, 4 assertions)</td>
</tr>
<tr class="even">
<td>.. _writing-tests-for-phpunit.data-providers:</td>
</tr>
<tr class="odd">
<td>数据供给器</td>
</tr>
<tr class="even">
<td>##############</td>
</tr>
<tr class="odd">
<td>测试方法可以接受任意参数。这些参数由一个或多个数据供给器方法（在writing-tests-for-phpunit.data-providers.examples.DataTest.php 中，是 <code>additionProvider()</code> 方法）提供。用 <code>@dataProvider</code> 标注来指定要使用的数据供给器方法。</td>
</tr>
<tr class="even">
<td>数据供给器方法必须声明为 <code>public</code>，其返回值要么是一个数组，其每个元素也是数组；要么是一个实现了 <code>Iterator</code> 接口的对象，在对它进行迭代时每步产生一个数组。每个数组都是测试数据集的一部分，将以它的内容作为参数来调用测试方法。</td>
</tr>
<tr class="odd">
<td>.. code-block:: php</td>
</tr>
<tr class="even">
<td>:caption: 使用返回数组的数组的数据供给器</td>
</tr>
<tr class="odd">
<td>:name: writing-tests-for-phpunit.data-providers.examples.DataTest.php</td>
</tr>
<tr class="even">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="odd">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="even">
<td>final class DataTest extends TestCase</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>/**</td>
</tr>
<tr class="odd">
<td>* @dataProvider additionProvider</td>
</tr>
<tr class="even">
<td>*/</td>
</tr>
<tr class="odd">
<td>public function testAdd(int $a, int $b, int $expected): void</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>$this-&gt;assertSame($expected, $a + $b);</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>public function additionProvider(): array</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>return [</td>
</tr>
<tr class="even">
<td>[0, 0, 0],</td>
</tr>
<tr class="odd">
<td>[0, 1, 1],</td>
</tr>
<tr class="even">
<td>[1, 0, 1],</td>
</tr>
<tr class="odd">
<td>[1, 1, 3]</td>
</tr>
<tr class="even">
<td>];</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>.. parsed-literal::</td>
</tr>
<tr class="even">
<td>$ phpunit DataTest</td>
</tr>
<tr class="odd">
<td>PHPUnit .0 by Sebastian Bergmann and contributors.</td>
</tr>
<tr class="even">
<td>...F</td>
</tr>
<tr class="odd">
<td>Time: 0 seconds, Memory: 5.75Mb</td>
</tr>
<tr class="even">
<td>There was 1 failure:</td>
</tr>
<tr class="odd">
<td>1) DataTest::testAdd with data set #3 (1, 1, 3)</td>
</tr>
<tr class="even">
<td>Failed asserting that 2 is identical to 3.</td>
</tr>
<tr class="odd">
<td>/home/sb/DataTest.php:9</td>
</tr>
<tr class="even">
<td>FAILURES!</td>
</tr>
<tr class="odd">
<td>Tests: 4, Assertions: 4, Failures: 1.</td>
</tr>
<tr class="even">
<td>当使用到大量数据集时，最好逐个用字符串键名对其命名，避免用默认的数字键名。这样输出信息会更加详细些，其中将包含打断测试的数据集所对应的名称。</td>
</tr>
<tr class="odd">
<td>.. code-block:: php</td>
</tr>
<tr class="even">
<td>:caption: 将数据供给器与命名数据集一起使用</td>
</tr>
<tr class="odd">
<td>:name: writing-tests-for-phpunit.data-providers.examples.DataTest1.php</td>
</tr>
<tr class="even">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="odd">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="even">
<td>final class DataTest extends TestCase</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>/**</td>
</tr>
<tr class="odd">
<td>* @dataProvider additionProvider</td>
</tr>
<tr class="even">
<td>*/</td>
</tr>
<tr class="odd">
<td>public function testAdd(int $a, int $b, int $expected): void</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>$this-&gt;assertSame($expected, $a + $b);</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>public function additionProvider(): array</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>return [</td>
</tr>
<tr class="even">
<td>'adding zeros' =&gt; [0, 0, 0],</td>
</tr>
<tr class="odd">
<td>'zero plus one' =&gt; [0, 1, 1],</td>
</tr>
<tr class="even">
<td>'one plus zero' =&gt; [1, 0, 1],</td>
</tr>
<tr class="odd">
<td>'one plus one' =&gt; [1, 1, 3]</td>
</tr>
<tr class="even">
<td>];</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>.. parsed-literal::</td>
</tr>
<tr class="even">
<td>$ phpunit DataTest</td>
</tr>
<tr class="odd">
<td>PHPUnit .0 by Sebastian Bergmann and contributors.</td>
</tr>
<tr class="even">
<td>...F</td>
</tr>
<tr class="odd">
<td>Time: 0 seconds, Memory: 5.75Mb</td>
</tr>
<tr class="even">
<td>There was 1 failure:</td>
</tr>
<tr class="odd">
<td>1) DataTest::testAdd with data set &quot;one plus one&quot; (1, 1, 3)</td>
</tr>
<tr class="even">
<td>Failed asserting that 2 is identical to 3.</td>
</tr>
<tr class="odd">
<td>/home/sb/DataTest.php:9</td>
</tr>
<tr class="even">
<td>FAILURES!</td>
</tr>
<tr class="odd">
<td>Tests: 4, Assertions: 4, Failures: 1.</td>
</tr>
<tr class="even">
<td>.. code-block:: php</td>
</tr>
<tr class="odd">
<td>:caption: 使用返回 Iterator 对象的数据供给器</td>
</tr>
<tr class="even">
<td>:name: writing-tests-for-phpunit.data-providers.examples.DataTest2.php</td>
</tr>
<tr class="odd">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="even">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="odd">
<td>final class DataTest extends TestCase</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>/**</td>
</tr>
<tr class="even">
<td>* @dataProvider additionProvider</td>
</tr>
<tr class="odd">
<td>*/</td>
</tr>
<tr class="even">
<td>public function testAdd(int $a, int $b, int $expected): void</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$this-&gt;assertSame($expected, $a + $b);</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>public function additionProvider(): CsvFileIterator</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>return new CsvFileIterator('data.csv');</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>.. parsed-literal::</td>
</tr>
<tr class="even">
<td>$ phpunit DataTest</td>
</tr>
<tr class="odd">
<td>PHPUnit .0 by Sebastian Bergmann and contributors.</td>
</tr>
<tr class="even">
<td>...F</td>
</tr>
<tr class="odd">
<td>Time: 0 seconds, Memory: 5.75Mb</td>
</tr>
<tr class="even">
<td>There was 1 failure:</td>
</tr>
<tr class="odd">
<td>1) DataTest::testAdd with data set #3 ('1', '1', '3')</td>
</tr>
<tr class="even">
<td>Failed asserting that 2 is identical to 3.</td>
</tr>
<tr class="odd">
<td>/home/sb/DataTest.php:11</td>
</tr>
<tr class="even">
<td>FAILURES!</td>
</tr>
<tr class="odd">
<td>Tests: 4, Assertions: 4, Failures: 1.</td>
</tr>
<tr class="even">
<td>.. code-block:: php</td>
</tr>
<tr class="odd">
<td>:caption: CsvFileIterator 类</td>
</tr>
<tr class="even">
<td>:name: writing-tests-for-phpunit.data-providers.examples.CsvFileIterator.php</td>
</tr>
<tr class="odd">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="even">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="odd">
<td>final class CsvFileIterator implements Iterator</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>private $file;</td>
</tr>
<tr class="even">
<td>private $key = 0;</td>
</tr>
<tr class="odd">
<td>private $current;</td>
</tr>
<tr class="even">
<td>public function __construct(string $file)</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$this-&gt;file = fopen($file, 'r');</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>public function __destruct()</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>fclose($this-&gt;file);</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>public function rewind(): void</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>rewind($this-&gt;file);</td>
</tr>
<tr class="odd">
<td>$this-&gt;current = fgetcsv($this-&gt;file);</td>
</tr>
<tr class="even">
<td>$this-&gt;key = 0;</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>public function valid(): bool</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>return !feof($this-&gt;file);</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>public function key(): int</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>return $this-&gt;key;</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>public function current(): array</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>return $this-&gt;current;</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>public function next(): void</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$this-&gt;current = fgetcsv($this-&gt;file);</td>
</tr>
<tr class="odd">
<td>$this-&gt;key++;</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>如果测试同时从 <code>@dataProvider</code> 方法和一个或多个 <code>@depends</code> 测试接收数据，那么来自于数据供给器的参数将先于来自所依赖的测试的。来自于所依赖的测试的参数对于每个数据集都是一样的。参见writing-tests-for-phpunit.data-providers.examples.DependencyAndDataProviderCombo.php</td>
</tr>
<tr class="odd">
<td>.. code-block:: php</td>
</tr>
<tr class="even">
<td>:caption: 在同一个测试中组合 @depends 和 @dataProvider</td>
</tr>
<tr class="odd">
<td>:name: writing-tests-for-phpunit.data-providers.examples.DependencyAndDataProviderCombo.php</td>
</tr>
<tr class="even">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="odd">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="even">
<td>final class DependencyAndDataProviderComboTest extends TestCase</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>public function provider(): array</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>return [['provider1'], ['provider2']];</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>public function testProducerFirst(): void</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$this-&gt;assertTrue(true);</td>
</tr>
<tr class="odd">
<td>return 'first';</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>public function testProducerSecond(): void</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>$this-&gt;assertTrue(true);</td>
</tr>
<tr class="even">
<td>return 'second';</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>/**</td>
</tr>
<tr class="odd">
<td>* @depends testProducerFirst</td>
</tr>
<tr class="even">
<td>* @depends testProducerSecond</td>
</tr>
<tr class="odd">
<td>* @dataProvider provider</td>
</tr>
<tr class="even">
<td>*/</td>
</tr>
<tr class="odd">
<td>public function testConsumer(): void</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>$this-&gt;assertSame(</td>
</tr>
<tr class="even">
<td>['provider1', 'first', 'second'],</td>
</tr>
<tr class="odd">
<td>func_get_args()</td>
</tr>
<tr class="even">
<td>);</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>.. parsed-literal::</td>
</tr>
<tr class="even">
<td>$ phpunit --verbose DependencyAndDataProviderComboTest</td>
</tr>
<tr class="odd">
<td>PHPUnit .0 by Sebastian Bergmann and contributors.</td>
</tr>
<tr class="even">
<td>...F</td>
</tr>
<tr class="odd">
<td>Time: 0 seconds, Memory: 3.50Mb</td>
</tr>
<tr class="even">
<td>There was 1 failure:</td>
</tr>
<tr class="odd">
<td>1) DependencyAndDataProviderComboTest::testConsumer with data set #1 ('provider2')</td>
</tr>
<tr class="even">
<td>Failed asserting that two arrays are identical.</td>
</tr>
<tr class="odd">
<td>--- Expected</td>
</tr>
<tr class="even">
<td>+++ Actual</td>
</tr>
<tr class="odd">
<td>@@ @@</td>
</tr>
<tr class="even">
<td>Array &amp;0 (</td>
</tr>
<tr class="odd">
<td>- 0 =&gt; 'provider1'</td>
</tr>
<tr class="even">
<td>+ 0 =&gt; 'provider2'</td>
</tr>
<tr class="odd">
<td>1 =&gt; 'first'</td>
</tr>
<tr class="even">
<td>2 =&gt; 'second'</td>
</tr>
<tr class="odd">
<td>)</td>
</tr>
<tr class="even">
<td>/home/sb/DependencyAndDataProviderComboTest.php:32</td>
</tr>
<tr class="odd">
<td>FAILURES!</td>
</tr>
<tr class="even">
<td>Tests: 4, Assertions: 4, Failures: 1.</td>
</tr>
<tr class="odd">
<td>.. code-block:: php</td>
</tr>
<tr class="even">
<td>:caption: 对单个测试使用多个数据供给器</td>
</tr>
<tr class="odd">
<td>:name: writing-tests-for-phpunit.data-providers.examples2.DataTest.php</td>
</tr>
<tr class="even">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="odd">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="even">
<td>final class DataTest extends TestCase</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>/**</td>
</tr>
<tr class="odd">
<td>* @dataProvider additionWithNonNegativeNumbersProvider</td>
</tr>
<tr class="even">
<td>* @dataProvider additionWithNegativeNumbersProvider</td>
</tr>
<tr class="odd">
<td>*/</td>
</tr>
<tr class="even">
<td>public function testAdd(int $a, int $b, int $expected): void</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$this-&gt;assertSame($expected, $a + $b);</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>public function additionWithNonNegativeNumbersProvider(): void</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>return [</td>
</tr>
<tr class="odd">
<td>[0, 1, 1],</td>
</tr>
<tr class="even">
<td>[1, 0, 1],</td>
</tr>
<tr class="odd">
<td>[1, 1, 3]</td>
</tr>
<tr class="even">
<td>];</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>public function additionWithNegativeNumbersProvider(): array</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>return [</td>
</tr>
<tr class="odd">
<td>[-1, 1, 0],</td>
</tr>
<tr class="even">
<td>[-1, -1, -2],</td>
</tr>
<tr class="odd">
<td>[1, -1, 0]</td>
</tr>
<tr class="even">
<td>];</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>.. parsed-literal::</td>
</tr>
<tr class="even">
<td>$ phpunit DataTest</td>
</tr>
<tr class="odd">
<td>PHPUnit .0 by Sebastian Bergmann and contributors.</td>
</tr>
<tr class="even">
<td>..F... 6 / 6 (100%)</td>
</tr>
<tr class="odd">
<td>Time: 0 seconds, Memory: 5.75Mb</td>
</tr>
<tr class="even">
<td>There was 1 failure:</td>
</tr>
<tr class="odd">
<td>1) DataTest::testAdd with data set #3 (1, 1, 3)</td>
</tr>
<tr class="even">
<td>Failed asserting that 2 is identical to 3.</td>
</tr>
<tr class="odd">
<td>/home/sb/DataTest.php:12</td>
</tr>
<tr class="even">
<td>FAILURES!</td>
</tr>
<tr class="odd">
<td>Tests: 6, Assertions: 6, Failures: 1.</td>
</tr>
<tr class="even">
<td>.. admonition:: 注</td>
</tr>
<tr class="odd">
<td>如果一个测试依赖于另外一个使用了数据供给器的测试，仅当被依赖的测试至少能在一组数据上成功时，依赖于它的测试才会运行。使用了数据供给器的测试，其运行结果是无法注入到依赖于此测试的其他测试中的。</td>
</tr>
<tr class="even">
<td>.. admonition:: 注</td>
</tr>
<tr class="odd">
<td>所有数据供给器方法的执行都是在对 <code>setUpBeforeClass()</code> 静态方法的调用和第一次对 <code>setUp()</code> 方法的调用之前完成的。因此，无法在数据供给器中使用创建于这两个方法内的变量。这是必须的，这样 PHPUnit 才能计算测试的总数量。</td>
</tr>
<tr class="even">
<td>.. _writing-tests-for-phpunit.exceptions:</td>
</tr>
<tr class="odd">
<td>对异常进行测试</td>
</tr>
<tr class="even">
<td>##################</td>
</tr>
<tr class="odd">
<td>writing-tests-for-phpunit.exceptions.examples.ExceptionTest.php 展示了如何用 <code>@expectException</code> 标注来测试被测代码中是否抛出了异常。</td>
</tr>
<tr class="even">
<td>.. code-block:: php</td>
</tr>
<tr class="odd">
<td>:caption: 使用 expectException() 方法</td>
</tr>
<tr class="even">
<td>:name: writing-tests-for-phpunit.exceptions.examples.ExceptionTest.php</td>
</tr>
<tr class="odd">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="even">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="odd">
<td>final class ExceptionTest extends TestCase</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>public function testException(): void</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>$this-&gt;expectException(InvalidArgumentException::class);</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>.. parsed-literal::</td>
</tr>
<tr class="odd">
<td>$ phpunit ExceptionTest</td>
</tr>
<tr class="even">
<td>PHPUnit .0 by Sebastian Bergmann and contributors.</td>
</tr>
<tr class="odd">
<td>F</td>
</tr>
<tr class="even">
<td>Time: 0 seconds, Memory: 4.75Mb</td>
</tr>
<tr class="odd">
<td>There was 1 failure:</td>
</tr>
<tr class="even">
<td>1) ExceptionTest::testException</td>
</tr>
<tr class="odd">
<td>Failed asserting that exception of type &quot;InvalidArgumentException&quot; is thrown.</td>
</tr>
<tr class="even">
<td>FAILURES!</td>
</tr>
<tr class="odd">
<td>Tests: 1, Assertions: 1, Failures: 1.</td>
</tr>
<tr class="even">
<td>除了 <code>expectException()</code> 方法外，还有 <code>expectExceptionCode()</code>、<code>expectExceptionMessage()</code> 和 <code>expectExceptionMessageMatches()</code> 方法可以用于为被测代码所抛出的异常建立预期。</td>
</tr>
<tr class="odd">
<td>.. admonition:: 注</td>
</tr>
<tr class="even">
<td>注意 <code>expectExceptionMessage()</code> 断言的是 <code>$actual</code> 讯息包含有 <code>$expected</code> 讯息，并不执行精确的字符串比较。</td>
</tr>
<tr class="odd">
<td>.. _writing-tests-for-phpunit.errors:</td>
</tr>
<tr class="even">
<td>对 PHP 错误、警告和通知进行测试</td>
</tr>
<tr class="odd">
<td>#########################################</td>
</tr>
<tr class="even">
<td>默认情况下，PHPUnit 将测试在执行中触发的 PHP 错误、警告、通知都转换为异常。先不说其他好处，这样就可以预期在测试中会触发 PHP 错误、警告或通知，如writing-tests-for-phpunit.exceptions.examples.ErrorTest.php 所示。</td>
</tr>
<tr class="odd">
<td>.. admonition:: 注</td>
</tr>
<tr class="even">
<td>PHP 的 <code>error_reporting</code> 运行时配置会对 PHPUnit 将哪些错误转换为异常有所限制。如果在这个特性上碰到问题，请确认 PHP 的配置中没有抑制你所关注的错误类型。</td>
</tr>
<tr class="odd">
<td>.. code-block:: php</td>
</tr>
<tr class="even">
<td>:caption: 预期会出现 PHP 错误、警告和通知</td>
</tr>
<tr class="odd">
<td>:name: writing-tests-for-phpunit.exceptions.examples.ErrorTest.php</td>
</tr>
<tr class="even">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="odd">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="even">
<td>final class ErrorTest extends TestCase</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>public function testDeprecationCanBeExpected(): void</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$this-&gt;expectDeprecation();</td>
</tr>
<tr class="odd">
<td>// （可选）测试讯息和某个字符串相等</td>
</tr>
<tr class="even">
<td>$this-&gt;expectDeprecationMessage('foo');</td>
</tr>
<tr class="odd">
<td>// 或者（可选）测试讯息和某个正则表达式匹配</td>
</tr>
<tr class="even">
<td>$this-&gt;expectDeprecationMessageMatches('/foo/');</td>
</tr>
<tr class="odd">
<td>trigger_error('foo', E_USER_DEPRECATED);</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>public function testNoticeCanBeExpected(): void</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>$this-&gt;expectNotice();</td>
</tr>
<tr class="even">
<td>// （可选）测试讯息和某个字符串相等</td>
</tr>
<tr class="odd">
<td>$this-&gt;expectNoticeMessage('foo');</td>
</tr>
<tr class="even">
<td>// 或者（可选）测试讯息和某个正则表达式匹配</td>
</tr>
<tr class="odd">
<td>$this-&gt;expectNoticeMessageMatches('/foo/');</td>
</tr>
<tr class="even">
<td>trigger_error('foo', E_USER_NOTICE);</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>public function testWarningCanBeExpected(): void</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$this-&gt;expectWarning();</td>
</tr>
<tr class="odd">
<td>// （可选）测试讯息和某个字符串相等</td>
</tr>
<tr class="even">
<td>$this-&gt;expectWarningMessage('foo');</td>
</tr>
<tr class="odd">
<td>// 或者（可选）测试讯息和某个正则表达式匹配</td>
</tr>
<tr class="even">
<td>$this-&gt;expectWarningMessageMatches('/foo/');</td>
</tr>
<tr class="odd">
<td>trigger_error('foo', E_USER_WARNING);</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>public function testErrorCanBeExpected(): void</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>$this-&gt;expectError();</td>
</tr>
<tr class="even">
<td>// （可选）测试讯息和某个字符串相等</td>
</tr>
<tr class="odd">
<td>$this-&gt;expectErrorMessage('foo');</td>
</tr>
<tr class="even">
<td>// 或者（可选）测试讯息和某个正则表达式匹配</td>
</tr>
<tr class="odd">
<td>$this-&gt;expectErrorMessageMatches('/foo/');</td>
</tr>
<tr class="even">
<td>trigger_error('foo', E_USER_ERROR);</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>如果测试代码使用了会触发错误的 PHP 内建函数，比如 <code>fopen</code>，有时候在测试中使用错误抑制符会很有用。通过抑制住错误通知，就能对返回值进行检查，否则错误通知将会导致 PHPUnit 的错误处理程序抛出异常。</td>
</tr>
<tr class="even">
<td>.. code-block:: php</td>
</tr>
<tr class="odd">
<td>:caption: 对会引发PHP 错误的代码的返回值进行测试</td>
</tr>
<tr class="even">
<td>:name: writing-tests-for-phpunit.exceptions.examples.TriggerErrorReturnValue.php</td>
</tr>
<tr class="odd">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="even">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="odd">
<td>final class ErrorSuppressionTest extends TestCase</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>public function testFileWriting(): void</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>$writer = new FileWriter;</td>
</tr>
<tr class="even">
<td>$this-&gt;assertFalse(@$writer-&gt;write('/is-not-writeable/file', 'stuff'));</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>final class FileWriter</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>public function write($file, $content)</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>$file = fopen($file, 'w');</td>
</tr>
<tr class="even">
<td>if ($file === false) {</td>
</tr>
<tr class="odd">
<td>return false;</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>// ...</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>.. parsed-literal::</td>
</tr>
<tr class="odd">
<td>$ phpunit ErrorSuppressionTest</td>
</tr>
<tr class="even">
<td>PHPUnit .0 by Sebastian Bergmann and contributors.</td>
</tr>
<tr class="odd">
<td>.</td>
</tr>
<tr class="even">
<td>Time: 1 seconds, Memory: 5.25Mb</td>
</tr>
<tr class="odd">
<td>OK (1 test, 1 assertion)</td>
</tr>
<tr class="even">
<td>如果不使用错误抑制符，此测试将会失败，并报告 <code>fopen(/is-not-writeable/file): failed to open stream: No such file or directory</code>。</td>
</tr>
<tr class="odd">
<td>.. _writing-tests-for-phpunit.output:</td>
</tr>
<tr class="even">
<td>对输出进行测试</td>
</tr>
<tr class="odd">
<td>##############</td>
</tr>
<tr class="even">
<td>有时候，想要断言（比如说）某方法的运行过程中生成了预期的输出（例如，通过 <code>echo</code> 或 <code>print</code>）。<code>PHPUnit\Framework\TestCase</code> 类使用 PHP 的<a href="http://www.php.net/manual/en/ref.outcontrol.php">输出缓冲</a>特性来为此提供必要的功能支持。</td>
</tr>
<tr class="odd">
<td>writing-tests-for-phpunit.output.examples.OutputTest.php 展示了如何用 <code>expectOutputString()</code> 方法来设定所预期的输出。如果没有产生预期的输出，测试将计为失败。</td>
</tr>
<tr class="even">
<td>.. code-block:: php</td>
</tr>
<tr class="odd">
<td>:caption: 对函数或方法的输出进行测试</td>
</tr>
<tr class="even">
<td>:name: writing-tests-for-phpunit.output.examples.OutputTest.php</td>
</tr>
<tr class="odd">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="even">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="odd">
<td>final class OutputTest extends TestCase</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>public function testExpectFooActualFoo(): void</td>
</tr>
<tr class="even">
<td>{</td>
</tr>
<tr class="odd">
<td>$this-&gt;expectOutputString('foo');</td>
</tr>
<tr class="even">
<td>print 'foo';</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>public function testExpectBarActualBaz(): void</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$this-&gt;expectOutputString('bar');</td>
</tr>
<tr class="odd">
<td>print 'baz';</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>.. parsed-literal::</td>
</tr>
<tr class="odd">
<td>$ phpunit OutputTest</td>
</tr>
<tr class="even">
<td>PHPUnit .0 by Sebastian Bergmann and contributors.</td>
</tr>
<tr class="odd">
<td>.F</td>
</tr>
<tr class="even">
<td>Time: 0 seconds, Memory: 5.75Mb</td>
</tr>
<tr class="odd">
<td>There was 1 failure:</td>
</tr>
<tr class="even">
<td>1) OutputTest::testExpectBarActualBaz</td>
</tr>
<tr class="odd">
<td>Failed asserting that two strings are equal.</td>
</tr>
<tr class="even">
<td>--- Expected</td>
</tr>
<tr class="odd">
<td>+++ Actual</td>
</tr>
<tr class="even">
<td>@@ @@</td>
</tr>
<tr class="odd">
<td>-'bar'</td>
</tr>
<tr class="even">
<td>+'baz'</td>
</tr>
<tr class="odd">
<td>FAILURES!</td>
</tr>
<tr class="even">
<td>Tests: 2, Assertions: 2, Failures: 1.</td>
</tr>
<tr class="odd">
<td>writing-tests-for-phpunit.output.tables.api</td>
</tr>
<tr class="even">
<td>中列举了用于对输出进行测试的各种方法</td>
</tr>
<tr class="odd">
<td>.. rst-class:: table</td>
</tr>
<tr class="even">
<td>.. list-table:: 测试输出的方法</td>
</tr>
<tr class="odd">
<td>:name: writing-tests-for-phpunit.output.tables.api</td>
</tr>
<tr class="even">
<td>:header-rows: 1</td>
</tr>
<tr class="odd">
<td>* - 方法</td>
</tr>
<tr class="even">
<td>- 含义</td>
</tr>
<tr class="odd">
<td>* - <code>void expectOutputRegex(string $regularExpression)</code></td>
</tr>
<tr class="even">
<td>- 设置输出预期为输出应当匹配正则表达式 <code>$regularExpression</code>。</td>
</tr>
<tr class="odd">
<td>* - <code>void expectOutputString(string $expectedString)</code></td>
</tr>
<tr class="even">
<td>- 设置输出预期为输出应当与 <code>$expectedString</code> 相等。</td>
</tr>
<tr class="odd">
<td>* - <code>bool setOutputCallback(callable $callback)</code></td>
</tr>
<tr class="even">
<td>- 设置回调函数，用来做诸如将实际输出规范化之类的动作。</td>
</tr>
<tr class="odd">
<td>* - <code>string getActualOutput()</code></td>
</tr>
<tr class="even">
<td>- 获取实际输出。</td>
</tr>
<tr class="odd">
<td>.. admonition:: 注</td>
</tr>
<tr class="even">
<td>在严格模式下，本身产生输出的测试将会失败。</td>
</tr>
<tr class="odd">
<td>.. _writing-tests-for-phpunit.error-output:</td>
</tr>
<tr class="even">
<td>错误相关信息的输出</td>
</tr>
<tr class="odd">
<td>###################</td>
</tr>
<tr class="even">
<td>当有测试失败时，PHPUnit 全力提供尽可能多的有助于找出问题所在的上下文信息。</td>
</tr>
<tr class="odd">
<td>.. code-block:: php</td>
</tr>
<tr class="even">
<td>:caption: 数组比较失败时生成的错误输出</td>
</tr>
<tr class="odd">
<td>:name: writing-tests-for-phpunit.error-output.examples.ArrayDiffTest.php</td>
</tr>
<tr class="even">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="odd">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="even">
<td>final class ArrayDiffTest extends TestCase</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>public function testEquality(): void</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$this-&gt;assertSame(</td>
</tr>
<tr class="odd">
<td>[1, 2, 3, 4, 5, 6],</td>
</tr>
<tr class="even">
<td>[1, 2, 33, 4, 5, 6]</td>
</tr>
<tr class="odd">
<td>);</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>.. parsed-literal::</td>
</tr>
<tr class="odd">
<td>$ phpunit ArrayDiffTest</td>
</tr>
<tr class="even">
<td>PHPUnit .0 by Sebastian Bergmann and contributors.</td>
</tr>
<tr class="odd">
<td>F</td>
</tr>
<tr class="even">
<td>Time: 0 seconds, Memory: 5.25Mb</td>
</tr>
<tr class="odd">
<td>There was 1 failure:</td>
</tr>
<tr class="even">
<td>1) ArrayDiffTest::testEquality</td>
</tr>
<tr class="odd">
<td>Failed asserting that two arrays are identical.</td>
</tr>
<tr class="even">
<td>--- Expected</td>
</tr>
<tr class="odd">
<td>+++ Actual</td>
</tr>
<tr class="even">
<td>@@ @@</td>
</tr>
<tr class="odd">
<td>Array (</td>
</tr>
<tr class="even">
<td>0 =&gt; 1</td>
</tr>
<tr class="odd">
<td>1 =&gt; 2</td>
</tr>
<tr class="even">
<td>- 2 =&gt; 3</td>
</tr>
<tr class="odd">
<td>+ 2 =&gt; 33</td>
</tr>
<tr class="even">
<td>3 =&gt; 4</td>
</tr>
<tr class="odd">
<td>4 =&gt; 5</td>
</tr>
<tr class="even">
<td>5 =&gt; 6</td>
</tr>
<tr class="odd">
<td>)</td>
</tr>
<tr class="even">
<td>/home/sb/ArrayDiffTest.php:7</td>
</tr>
<tr class="odd">
<td>FAILURES!</td>
</tr>
<tr class="even">
<td>Tests: 1, Assertions: 1, Failures: 1.</td>
</tr>
<tr class="odd">
<td>在这个例子中，数组中只有一个值不同，但其他值也都同时显示出来，以提供关于错误发生的位置的上下文信息。</td>
</tr>
<tr class="even">
<td>当生成的输出很长而难以阅读时，PHPUnit 将对其进行分割，并在每个差异附近提供少数几行上下文信息。</td>
</tr>
<tr class="odd">
<td>.. code-block:: php</td>
</tr>
<tr class="even">
<td>:caption: 长数组的数组比较失败时的错误输出</td>
</tr>
<tr class="odd">
<td>:name: writing-tests-for-phpunit.error-output.examples.LongArrayDiffTest.php</td>
</tr>
<tr class="even">
<td>&lt;?php declare(strict_types=1);</td>
</tr>
<tr class="odd">
<td>use PHPUnitFrameworkTestCase;</td>
</tr>
<tr class="even">
<td>final class LongArrayDiffTest extends TestCase</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>public function testEquality(): void</td>
</tr>
<tr class="odd">
<td>{</td>
</tr>
<tr class="even">
<td>$this-&gt;assertSame(</td>
</tr>
<tr class="odd">
<td>[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 2, 3, 4, 5, 6],</td>
</tr>
<tr class="even">
<td>[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 2, 33, 4, 5, 6]</td>
</tr>
<tr class="odd">
<td>);</td>
</tr>
<tr class="even">
<td>}</td>
</tr>
<tr class="odd">
<td>}</td>
</tr>
<tr class="even">
<td>.. parsed-literal::</td>
</tr>
<tr class="odd">
<td>$ phpunit LongArrayDiffTest</td>
</tr>
<tr class="even">
<td>PHPUnit .0 by Sebastian Bergmann and contributors.</td>
</tr>
<tr class="odd">
<td>F</td>
</tr>
<tr class="even">
<td>Time: 0 seconds, Memory: 5.25Mb</td>
</tr>
<tr class="odd">
<td>There was 1 failure:</td>
</tr>
<tr class="even">
<td>1) LongArrayDiffTest::testEquality</td>
</tr>
<tr class="odd">
<td>Failed asserting that two arrays are identical.</td>
</tr>
<tr class="even">
<td>--- Expected</td>
</tr>
<tr class="odd">
<td>+++ Actual</td>
</tr>
<tr class="even">
<td>@@ @@</td>
</tr>
<tr class="odd">
<td>11 =&gt; 0</td>
</tr>
<tr class="even">
<td>12 =&gt; 1</td>
</tr>
<tr class="odd">
<td>13 =&gt; 2</td>
</tr>
<tr class="even">
<td>- 14 =&gt; 3</td>
</tr>
<tr class="odd">
<td>+ 14 =&gt; 33</td>
</tr>
<tr class="even">
<td>15 =&gt; 4</td>
</tr>
<tr class="odd">
<td>16 =&gt; 5</td>
</tr>
<tr class="even">
<td>17 =&gt; 6</td>
</tr>
<tr class="odd">
<td>)</td>
</tr>
<tr class="even">
<td>/home/sb/LongArrayDiffTest.php:7</td>
</tr>
<tr class="odd">
<td>FAILURES!</td>
</tr>
<tr class="even">
<td>Tests: 1, Assertions: 1, Failures: 1.</td>
</tr>
<tr class="odd">
<td>.. _writing-tests-for-phpunit.error-output.edge-cases:</td>
</tr>
<tr class="even">
<td>边缘情况</td>
</tr>
</tbody>
</table>

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

在这个例子中，第一个索引项中的 `1` 和 `'1'` 在报告中被视为不同，虽然
`assertEquals()` 认为这两个值是匹配的。
