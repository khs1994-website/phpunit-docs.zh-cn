断言
====

本附录列举可用的各种断言方法。

断言方法的用法：静态 vs. 非静态
-------------------------------

PHPUnit 的各个断言是在 `PHPUnit\Framework\Assert`
中实现的。`PHPUnit\Framework\TestCase` 则继承于
`PHPUnit\Framework\Assert`。

各个断言方法均声明为 `static`，可以从任何上下文以类似于
`PHPUnit\Framework\Assert::assertTrue()` 的方式调用，或者也可以用类似于
`$this->assertTrue()` 或 `self::assertTrue()` 的方式在扩展自
`PHPUnit\Framework\TestCase` 的类内调用。甚至可以用全局函数封装，例如
`assertTrue()`。

有个常见的疑问——对于那些 PHPUnit 的新手尤甚——是究竟应该用诸如
`$this->assertTrue()` 还是诸如 `self::assertTrue()`
这样的形式来调用断言才是“正确的方式”？简而言之：没有正确方式。同时，也没有错误方式。这基本上是个人喜好问题。

对于大多数人而言，由于测试方法是在测试对象上调用，因此用
`$this->assertTrue()` 会“觉的更正确”。然而请记住断言方法是声明为
`static`
的，这使其可以（重）用于测试对象的作用域之外。最后，全局函数封装让开发者能再少打一些字（用
`assertTrue()` 代替 `$this->assertTrue()` 或者 `self::assertTrue()`）。

assertArrayHasKey()
-------------------

`assertArrayHasKey(mixed $key, array $array[, string $message = ''])`

当 `$array` 不包含 `$key` 时报告错误，错误讯息由 `$message` 指定。

`assertArrayNotHasKey()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class ArrayHasKeyTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertArrayHasKey('foo', ['bar' => 'baz']);
        }
    }

assertClassHasAttribute()
-------------------------

`assertClassHasAttribute(string $attributeName, string $className[, string $message = ''])`

当 `$className::attributeName` 不存在时报告错误，错误讯息由 `$message`
指定。

`assertClassNotHasAttribute()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class ClassHasAttributeTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertClassHasAttribute('foo', stdClass::class);
        }
    }

assertClassHasStaticAttribute()
-------------------------------

`assertClassHasStaticAttribute(string $attributeName, string $className[, string $message = ''])`

当 `$className::attributeName` 不存在时报告错误，错误讯息由 `$message`
指定。

`assertClassNotHasStaticAttribute()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class ClassHasStaticAttributeTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertClassHasStaticAttribute('foo', stdClass::class);
        }
    }

assertContains()
----------------

`assertContains(mixed $needle, iterable $haystack[, string $message = ''])`

当 `$needle` 不是 `$haystack` 的元素时报告错误，错误讯息由 `$message`
指定。

`assertNotContains()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class ContainsTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertContains(4, [1, 2, 3]);
        }
    }

assertStringContainsString()
----------------------------

`assertStringContainsString(string $needle, string $haystack[, string $message = ''])`

当 `$needle` 不是 `$haystack` 的子字符串时报告错误，错误讯息由
`$message` 指定。

`assertStringNotContainsString()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class StringContainsStringTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertStringContainsString('foo', 'bar');
        }
    }

assertStringContainsStringIgnoringCase()
----------------------------------------

`assertStringContainsStringIgnoringCase(string $needle, string $haystack[, string $message = ''])`

当 `$needle` 不是 `$haystack` 的子字符串时报告错误，错误讯息由
`$message` 指定。

在 `$haystack` 中搜索 `$needle` 时，忽略大小写差异。

`assertStringNotContainsStringIgnoringCase()`
是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class StringContainsStringIgnoringCaseTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertStringContainsStringIgnoringCase('foo', 'bar');
        }
    }

assertContainsOnly()
--------------------

`assertContainsOnly(string $type, iterable $haystack[, boolean $isNativeType = null, string $message = ''])`

当 `$haystack` 并非仅包含类型为 `$type` 的变量时报告错误，错误讯息由
`$message` 指定。

`$isNativeType` 是一个标志，用来表明 `$type` 是否是原生 PHP 类型。

`assertNotContainsOnly()` 是与之相反的断言，并接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class ContainsOnlyTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertContainsOnly('string', ['1', '2', 3]);
        }
    }

assertContainsOnlyInstancesOf()
-------------------------------

`assertContainsOnlyInstancesOf(string $classname, Traversable|array $haystack[, string $message = ''])`

当 `$haystack` 并非仅包含类 `$classname` 的实例时报告错误，错误讯息由
`$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class ContainsOnlyInstancesOfTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertContainsOnlyInstancesOf(
                Foo::class,
                [new Foo, new Bar, new Foo]
            );
        }
    }

assertCount()
-------------

`assertCount($expectedCount, $haystack[, string $message = ''])`

当 `$haystack` 中的元素数量不是 `$expectedCount` 时报告错误，错误讯息由
`$message` 指定。

`assertNotCount()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class CountTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertCount(0, ['foo']);
        }
    }

assertDirectoryExists()
-----------------------

`assertDirectoryExists(string $directory[, string $message = ''])`

当 `$directory` 所指定的目录不存在时报告错误，错误讯息由 `$message`
指定。

`assertDirectoryDoesNotExist()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class DirectoryExistsTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertDirectoryExists('/path/to/directory');
        }
    }

assertDirectoryIsReadable()
---------------------------

`assertDirectoryIsReadable(string $directory[, string $message = ''])`

当 `$directory` 所指定的目录不是个目录或不可读时报告错误，错误讯息由
`$message` 指定。

`assertDirectoryIsNotReadable()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class DirectoryIsReadableTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertDirectoryIsReadable('/path/to/directory');
        }
    }

assertDirectoryIsWritable()
---------------------------

`assertDirectoryIsWritable(string $directory[, string $message = ''])`

当 `$directory` 所指定的目录不是个目录或不可写时报告错误，错误讯息由
`$message` 指定。

`assertDirectoryIsNotWritable()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class DirectoryIsWritableTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertDirectoryIsWritable('/path/to/directory');
        }
    }

assertEmpty()
-------------

`assertEmpty(mixed $actual[, string $message = ''])`

当 `$actual` 非空时报告错误，错误讯息由 `$message` 指定。

`assertNotEmpty()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class EmptyTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertEmpty(['foo']);
        }
    }

assertEquals()
--------------

`assertEquals(mixed $expected, mixed $actual[, string $message = ''])`

当两个变量 `$expected` 和 `$actual` 不相等时报告错误，错误讯息由
`$message` 指定。

`assertNotEquals()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class EqualsTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertEquals(1, 0);
        }

        public function testFailure2(): void
        {
            $this->assertEquals('bar', 'baz');
        }

        public function testFailure3(): void
        {
            $this->assertEquals("foo\nbar\nbaz\n", "foo\nbah\nbaz\n");
        }
    }

如果 `$expected` 和 `$actual`
是某些特定的类型，将使用更加专门的比较方式，参阅下文。

`assertEquals(DOMDocument $expected, DOMDocument $actual[, string $message = ''])`

当 `$expected` 和 `$actual` 这两个 DOMDocument 对象所表示的 XML
文档对应的无注释规范形式不相同时报告错误，错误讯息由 `$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class EqualsTest extends TestCase
    {
        public function testFailure(): void
        {
            $expected = new DOMDocument;
            $expected->loadXML('<foo><bar/></foo>');

            $actual = new DOMDocument;
            $actual->loadXML('<bar><foo/></bar>');

            $this->assertEquals($expected, $actual);
        }
    }

`assertEquals(object $expected, object $actual[, string $message = ''])`

当 `$expected` 和 `$actual`
这两个对象的属性值不相等时报告错误，错误讯息由 `$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class EqualsTest extends TestCase
    {
        public function testFailure(): void
        {
            $expected = new stdClass;
            $expected->foo = 'foo';
            $expected->bar = 'bar';

            $actual = new stdClass;
            $actual->foo = 'bar';
            $actual->baz = 'bar';

            $this->assertEquals($expected, $actual);
        }
    }

`assertEquals(array $expected, array $actual[, string $message = ''])`

当 `$expected` 和 `$actual` 这两个数组不相等时报告错误，错误讯息由
`$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class EqualsTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertEquals(['a', 'b', 'c'], ['a', 'c', 'd']);
        }
    }

assertEqualsCanonicalizing()
----------------------------

`assertEqualsCanonicalizing(mixed $expected, mixed $actual[, string $message = ''])`

当两个变量 `$expected` 和 `$actual` 不相等时报告错误，错误讯息由
`$message` 指定。

先对 `$expected` 和 `$actual`
进行规范化，然后再进行比较。例如，如果两个变量 `$expected` 和 `$actual`
是数组，则会先对这些数组进行排序然后再进行比较。当 `$expected` 和
`$actual`
是对象时，则每个对象都会被转换为一个包含所有私有、保护、和公开属性的数组。

`assertNotEqualsCanonicalizing()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class EqualsCanonicalizingTest extends TestCase
    {
        public function testFailure()
        {
            $this->assertEqualsCanonicalizing([3, 2, 1], [2, 3, 0, 1]);
        }
    }

assertEqualsIgnoringCase()
--------------------------

`assertEqualsIgnoringCase(mixed $expected, mixed $actual[, string $message = ''])`

当两个变量 `$expected` 和 `$actual` 不相等时报告错误，错误讯息由
`$message` 指定。

比较 `$expected` 和 `$actual` 时，忽略大小写差异。

`assertNotEqualsIgnoringCase()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class EqualsIgnoringCaseTest extends TestCase
    {
        public function testFailure()
        {
            $this->assertEqualsIgnoringCase('foo', 'BAR');
        }
    }

assertEqualsWithDelta()
-----------------------

`assertEqualsWithDelta(mixed $expected, mixed $actual, float $delta[, string $message = ''])`

当 `$expected` 和 `$actual` 之间的差的绝对值大于 `$delta`
时报告错误，错误讯息由 `$message` 指定。

请阅读《[What Every Computer Scientist Should Know About Floating-Point
Arithmetic](http://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html)》来了解为什么需要有
`$delta`。

`assertNotEqualsWithDelta()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class EqualsWithDeltaTest extends TestCase
    {
        public function testFailure()
        {
            $this->assertEqualsWithDelta(1.0, 1.5, 0.1);
        }
    }

assertObjectEquals()
--------------------

`assertObjectEquals(object $expected, object $actual, string $method = 'equals', string $message = ''])`

当按照 `$actual->$method($expected)` 判断出 `$actual` 不等于 `$expected`
时报告错误，错误讯息由 `$message` 指定。

在对象上使用 `assertEquals()`（以及其否断言形式
`assertNotEquals()`）而不注册自定义比较器来定制对象的比较方式是一种不良做法。但是，很不幸地，为每个要在测试中进行断言的对象都实现自定义比较器无论如何至少也是不方便的。

自定义比较器最常见的用例是值对象。这类对象一般都有一个
`equals(self $other): bool`
方法（或者名称不同的类似方法）用于比较这个同为值对象类型的两个实例。对于这类用例，`assertObjectEquals()`
让自定义对象比较变得很方便：

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class SomethingThatUsesEmailTest extends TestCase
    {
        public function testSomething(): void
        {
            $a = new Email('user@example.org');
            $b = new Email('user@example.org');
            $c = new Email('user@example.com');

            // This passes
            $this->assertObjectEquals($a, $b);

            // This fails
            $this->assertObjectEquals($a, $c);
        }
    }

    <?php declare(strict_types=1);
    final class Email
    {
        private string $email;

        public function __construct(string $email)
        {
            $this->ensureIsValidEmail($email);

            $this->email = $email;
        }

        public function asString(): string
        {
            return $this->email;
        }

        public function equals(self $other): bool
        {
            return $this->asString() === $other->asString();
        }

        private function ensureIsValidEmail(string $email): void
        {
            // ...
        }
    }

请注意：

-   `$actual` 对象必须存在名叫 `$method` 的方法
-   此方法必须只接受一个参数
-   相应的参数必须有声明类型
-   `$expected` 对象必须与这个声明的类型兼容
-   此方法必须有声明为 `bool` 的返回类型

如果以上假设中的任何一条不满足，或 `$actual->$method($expected)` 返回
`false`，则断言失败。

assertFalse()
-------------

`assertFalse(bool $condition[, string $message = ''])`

当 `$condition` 为 `true` 时报告错误，错误讯息由 `$message` 指定。

`assertNotFalse()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class FalseTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertFalse(true);
        }
    }

assertFileEquals()
------------------

`assertFileEquals(string $expected, string $actual[, string $message = ''])`

当 `$expected` 所指定的文件与 `$actual`
所指定的文件内容不同时报告错误，错误讯息由 `$message` 指定。

`assertFileNotEquals()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class FileEqualsTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertFileEquals('/home/sb/expected', '/home/sb/actual');
        }
    }

assertFileExists()
------------------

`assertFileExists(string $filename[, string $message = ''])`

当 `$filename` 所指定的文件不存在时报告错误，错误讯息由 `$message`
指定。

`assertFileDoesNotExist()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class FileExistsTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertFileExists('/path/to/file');
        }
    }

assertFileIsReadable()
----------------------

`assertFileIsReadable(string $filename[, string $message = ''])`

当 `$filename` 所指定的文件不是个文件或不可读时报告错误，错误讯息由
`$message` 指定。

`assertFileIsNotReadable()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class FileIsReadableTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertFileIsReadable('/path/to/file');
        }
    }

assertFileIsWritable()
----------------------

`assertFileIsWritable(string $filename[, string $message = ''])`

当 `$filename` 所指定的文件不是个文件或不可写时报告错误，错误讯息由
`$message` 指定。

`assertFileIsNotWritable()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class FileIsWritableTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertFileIsWritable('/path/to/file');
        }
    }

assertGreaterThan()
-------------------

`assertGreaterThan(mixed $expected, mixed $actual[, string $message = ''])`

当 `$actual` 的值不大于 `$expected` 的值时报告错误，错误讯息由
`$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class GreaterThanTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertGreaterThan(2, 1);
        }
    }

assertGreaterThanOrEqual()
--------------------------

`assertGreaterThanOrEqual(mixed $expected, mixed $actual[, string $message = ''])`

当 `$actual` 的值不大于且不等于 `$expected` 的值时报告错误，错误讯息由
`$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class GreatThanOrEqualTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertGreaterThanOrEqual(2, 1);
        }
    }

assertInfinite()
----------------

`assertInfinite(mixed $variable[, string $message = ''])`

当 `$actual` 不是 `INF` 时报告错误，错误讯息由 `$message` 指定。

`assertFinite()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class InfiniteTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertInfinite(1);
        }
    }

assertInstanceOf()
------------------

`assertInstanceOf($expected, $actual[, $message = ''])`

当 `$actual` 不是 `$expected` 的实例时报告错误，错误讯息由 `$message`
指定。

`assertNotInstanceOf()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class InstanceOfTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertInstanceOf(RuntimeException::class, new Exception);
        }
    }

assertIsArray()
---------------

`assertIsArray($actual[, $message = ''])`

当 `$actual` 的类型不是 `array` 时报告错误，错误讯息由 `$message` 指定。

`assertIsNotArray()` 是与之相反的断言，接受相同的参数。

    <?php
    use PHPUnit\Framework\TestCase;

    class ArrayTest extends TestCase
    {
        public function testFailure()
        {
            $this->assertIsArray(null);
        }
    }

    $ phpunit ArrayTest
    PHPUnit |version|.0 by Sebastian Bergmann and contributors.

    F

    Time: 0 seconds, Memory: 5.00Mb

    There was 1 failure:

    1) ArrayTest::testFailure
    Failed asserting that null is of type "array".

    /home/sb/ArrayTest.php:8

    FAILURES!
    Tests: 1, Assertions: 1, Failures: 1.

assertIsBool()
--------------

`assertIsBool($actual[, $message = ''])`

当 `$actual` 的类型不是 `bool` 时报告错误，错误讯息由 `$message` 指定。

`assertIsNotBool()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class BoolTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertIsBool(null);
        }
    }

    $ phpunit BoolTest
    PHPUnit |version|.0 by Sebastian Bergmann and contributors.

    F

    Time: 0 seconds, Memory: 5.00Mb

    There was 1 failure:

    1) BoolTest::testFailure
    Failed asserting that null is of type "bool".

    /home/sb/BoolTest.php:8

    FAILURES!
    Tests: 1, Assertions: 1, Failures: 1.

assertIsCallable()
------------------

`assertIsCallable($actual[, $message = ''])`

当 `$actual` 的类型不是 `callable` 时报告错误，错误讯息由 `$message`
指定。

`assertIsNotCallable()` 是与之相反的断言，接受相同的参数。

    <?php
    use PHPUnit\Framework\TestCase;

    class CallableTest extends TestCase
    {
        public function testFailure()
        {
            $this->assertIsCallable(null);
        }
    }

    $ phpunit CallableTest
    PHPUnit |version|.0 by Sebastian Bergmann and contributors.

    F

    Time: 0 seconds, Memory: 5.00Mb

    There was 1 failure:

    1) CallableTest::testFailure
    Failed asserting that null is of type "callable".

    /home/sb/CallableTest.php:8

    FAILURES!
    Tests: 1, Assertions: 1, Failures: 1.

assertIsFloat()
---------------

`assertIsFloat($actual[, $message = ''])`

当 `$actual` 的类型不是 `float` 时报告错误，错误讯息由 `$message` 指定。

`assertIsNotFloat()` 是与之相反的断言，接受相同的参数。

    <?php
    use PHPUnit\Framework\TestCase;

    class FloatTest extends TestCase
    {
        public function testFailure()
        {
            $this->assertIsFloat(null);
        }
    }

    $ phpunit FloatTest
    PHPUnit |version|.0 by Sebastian Bergmann and contributors.

    F

    Time: 0 seconds, Memory: 5.00Mb

    There was 1 failure:

    1) FloatTest::testFailure
    Failed asserting that null is of type "float".

    /home/sb/FloatTest.php:8

    FAILURES!
    Tests: 1, Assertions: 1, Failures: 1.

assertIsInt()
-------------

`assertIsInt($actual[, $message = ''])`

当 `$actual` 的类型不是 `int` 时报告错误，错误讯息由 `$message` 指定。

`assertIsNotInt()` 是与之相反的断言，接受相同的参数。

    <?php
    use PHPUnit\Framework\TestCase;

    class IntTest extends TestCase
    {
        public function testFailure()
        {
            $this->assertIsInt(null);
        }
    }

    $ phpunit IntTest
    PHPUnit |version|.0 by Sebastian Bergmann and contributors.

    F

    Time: 0 seconds, Memory: 5.00Mb

    There was 1 failure:

    1) IntTest::testFailure
    Failed asserting that null is of type "int".

    /home/sb/IntTest.php:8

    FAILURES!
    Tests: 1, Assertions: 1, Failures: 1.

assertIsIterable()
------------------

`assertIsIterable($actual[, $message = ''])`

当 `$actual` 的类型不是 `iterable` 时报告错误，错误讯息由 `$message`
指定。

`assertIsNotIterable()` 是与之相反的断言，接受相同的参数。

    <?php
    use PHPUnit\Framework\TestCase;

    class IterableTest extends TestCase
    {
        public function testFailure()
        {
            $this->assertIsIterable(null);
        }
    }

    $ phpunit IterableTest
    PHPUnit |version|.0 by Sebastian Bergmann and contributors.

    F

    Time: 0 seconds, Memory: 5.00Mb

    There was 1 failure:

    1) IterableTest::testFailure
    Failed asserting that null is of type "iterable".

    /home/sb/IterableTest.php:8

    FAILURES!
    Tests: 1, Assertions: 1, Failures: 1.

assertIsNumeric()
-----------------

`assertIsNumeric($actual[, $message = ''])`

当 `$actual` 的类型不是 `numeric` 时报告错误，错误讯息由 `$message`
指定。

`assertIsNotNumeric()` 是与之相反的断言，接受相同的参数。

    <?php
    use PHPUnit\Framework\TestCase;

    class NumericTest extends TestCase
    {
        public function testFailure()
        {
            $this->assertIsNumeric(null);
        }
    }

    $ phpunit NumericTest
    PHPUnit |version|.0 by Sebastian Bergmann and contributors.

    F

    Time: 0 seconds, Memory: 5.00Mb

    There was 1 failure:

    1) NumericTest::testFailure
    Failed asserting that null is of type "numeric".

    /home/sb/NumericTest.php:8

    FAILURES!
    Tests: 1, Assertions: 1, Failures: 1.

assertIsObject()
----------------

`assertIsObject($actual[, $message = ''])`

当 `$actual` 的类型不是 `object` 时报告错误，错误讯息由 `$message`
指定。

`assertIsNotObject()` 是与之相反的断言，接受相同的参数。

    <?php
    use PHPUnit\Framework\TestCase;

    class ObjectTest extends TestCase
    {
        public function testFailure()
        {
            $this->assertIsObject(null);
        }
    }

    $ phpunit ObjectTest
    PHPUnit |version|.0 by Sebastian Bergmann and contributors.

    F

    Time: 0 seconds, Memory: 5.00Mb

    There was 1 failure:

    1) ObjectTest::testFailure
    Failed asserting that null is of type "object".

    /home/sb/ObjectTest.php:8

    FAILURES!
    Tests: 1, Assertions: 1, Failures: 1.

assertIsResource()
------------------

`assertIsResource($actual[, $message = ''])`

当 `$actual` 的类型不是 `resource` 时报告错误，错误讯息由 `$message`
指定。

`assertIsNotResource()` 是与之相反的断言，接受相同的参数。

    <?php
    use PHPUnit\Framework\TestCase;

    class ResourceTest extends TestCase
    {
        public function testFailure()
        {
            $this->assertIsResource(null);
        }
    }

    $ phpunit ResourceTest
    PHPUnit |version|.0 by Sebastian Bergmann and contributors.

    F

    Time: 0 seconds, Memory: 5.00Mb

    There was 1 failure:

    1) ResourceTest::testFailure
    Failed asserting that null is of type "resource".

    /home/sb/ResourceTest.php:8

    FAILURES!
    Tests: 1, Assertions: 1, Failures: 1.

assertIsScalar()
----------------

`assertIsScalar($actual[, $message = ''])`

当 `$actual` 的类型不是 `scalar` 时报告错误，错误讯息由 `$message`
指定。

`assertIsNotScalar()` 是与之相反的断言，接受相同的参数。

    <?php
    use PHPUnit\Framework\TestCase;

    class ScalarTest extends TestCase
    {
        public function testFailure()
        {
            $this->assertIsScalar(null);
        }
    }

    $ phpunit ScalarTest
    PHPUnit |version|.0 by Sebastian Bergmann and contributors.

    F

    Time: 0 seconds, Memory: 5.00Mb

    There was 1 failure:

    1) ScalarTest::testFailure
    Failed asserting that null is of type "scalar".

    /home/sb/ScalarTest.php:8

    FAILURES!
    Tests: 1, Assertions: 1, Failures: 1.

assertIsString()
----------------

`assertIsString($actual[, $message = ''])`

当 `$actual` 的类型不是 `string` 时报告错误，错误讯息由 `$message`
指定。

`assertIsNotString()` 是与之相反的断言，接受相同的参数。

    <?php
    use PHPUnit\Framework\TestCase;

    class StringTest extends TestCase
    {
        public function testFailure()
        {
            $this->assertIsString(null);
        }
    }

assertIsReadable()
------------------

`assertIsReadable(string $filename[, string $message = ''])`

当 `$filename` 所指定的文件或目录不可读时报告错误，错误讯息由 `$message`
指定。

`assertIsNotReadable()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class IsReadableTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertIsReadable('/path/to/unreadable');
        }
    }

assertIsWritable()
------------------

`assertIsWritable(string $filename[, string $message = ''])`

当 `$filename` 所指定的文件或目录不可写时报告错误，错误讯息由 `$message`
指定。

`assertIsNotWritable()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class IsWritableTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertIsWritable('/path/to/unwritable');
        }
    }

assertJsonFileEqualsJsonFile()
------------------------------

`assertJsonFileEqualsJsonFile(mixed $expectedFile, mixed $actualFile[, string $message = ''])`

当 `$actualFile` 对应的值与 `$expectedFile`
对应的值不匹配时报告错误，错误讯息由 `$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class JsonFileEqualsJsonFileTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertJsonFileEqualsJsonFile(
              'path/to/fixture/file', 'path/to/actual/file');
        }
    }

assertJsonStringEqualsJsonFile()
--------------------------------

`assertJsonStringEqualsJsonFile(mixed $expectedFile, mixed $actualJson[, string $message = ''])`

当 `$actualJson` 对应的值与 `$expectedFile`
对应的值不匹配时报告错误，错误讯息由 `$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class JsonStringEqualsJsonFileTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertJsonStringEqualsJsonFile(
                'path/to/fixture/file', json_encode(['Mascot' => 'ux'])
            );
        }
    }

assertJsonStringEqualsJsonString()
----------------------------------

`assertJsonStringEqualsJsonString(mixed $expectedJson, mixed $actualJson[, string $message = ''])`

当 `$actualJson` 对应的值与 `$expectedJson`
对应的值不匹配时报告错误，错误讯息由 `$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class JsonStringEqualsJsonStringTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertJsonStringEqualsJsonString(
                json_encode(['Mascot' => 'Tux']),
                json_encode(['Mascot' => 'ux'])
            );
        }
    }

assertLessThan()
----------------

`assertLessThan(mixed $expected, mixed $actual[, string $message = ''])`

当 `$actual` 的值不小于 `$expected` 的值时报告错误，错误讯息由
`$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class LessThanTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertLessThan(1, 2);
        }
    }

assertLessThanOrEqual()
-----------------------

`assertLessThanOrEqual(mixed $expected, mixed $actual[, string $message = ''])`

当 `$actual` 的值不小于且不等于 `$expected` 的值时报告错误，错误讯息由
`$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class LessThanOrEqualTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertLessThanOrEqual(1, 2);
        }
    }

assertNan()
-----------

`assertNan(mixed $variable[, string $message = ''])`

当 `$variable` 不是 `NAN` 时报告错误，错误讯息由 `$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class NanTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertNan(1);
        }
    }

assertNull()
------------

`assertNull(mixed $variable[, string $message = ''])`

当 `$actual` 不是 `null` 时报告错误，错误讯息由 `$message` 指定。

`assertNotNull()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class NullTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertNull('foo');
        }
    }

assertObjectHasAttribute()
--------------------------

`assertObjectHasAttribute(string $attributeName, object $object[, string $message = ''])`

当 `$object->attributeName` 不存在时报告错误，错误讯息由 `$message`
指定。

`assertObjectNotHasAttribute()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class ObjectHasAttributeTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertObjectHasAttribute('foo', new stdClass);
        }
    }

assertMatchesRegularExpression()
--------------------------------

`assertMatchesRegularExpression(string $pattern, string $string[, string $message = ''])`

当 `$string` 不匹配于正则表达式 `$pattern` 时报告错误，错误讯息由
`$message` 指定。

`assertDoesNotMatchRegularExpression()`
是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class RegExpTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertMatchesRegularExpression('/foo/', 'bar');
        }
    }

assertStringMatchesFormat()
---------------------------

`assertStringMatchesFormat(string $format, string $string[, string $message = ''])`

当 `$string` 不匹配于 `$format` 定义的格式时报告错误，错误讯息由
`$message` 指定。

`assertStringNotMatchesFormat()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class StringMatchesFormatTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertStringMatchesFormat('%i', 'foo');
        }
    }

格式定义字符串中可以使用如下占位符：

-

> `%e`：表示目录分隔符，例如在 Linux 系统中是 `/`。

-

> `%s`：一个或多个除了换行符以外的任意字符（非空白字符或者空白字符）。

-

> `%S`：零个或多个除了换行符以外的任意字符（非空白字符或者空白字符）。

-

> `%a`：一个或多个包括换行符在内的任意字符（非空白字符或者空白字符）。

-

> `%A`：零个或多个包括换行符在内的任意字符（非空白字符或者空白字符）。

-

> `%w`：零个或多个空白字符。

-

> `%i`：带符号整数值，例如 `+3142`、`-3142`。

-

> `%d`：无符号整数值，例如 `123456`。

-

> `%x`：一个或多个十六进制字符。所谓十六进制字符，指的是在以下范围内的字符：`0-9`、`a-f`、`A-F`。

-

> `%f`：浮点数，例如 `3.142`、`-3.142`、`3.142E-10`、`3.142e+10`。

-

> `%c`：单个任意字符。

-

> `%%`：原本的百分比字符：`%`。

assertStringMatchesFormatFile()
-------------------------------

`assertStringMatchesFormatFile(string $formatFile, string $string[, string $message = ''])`

当 `$string` 不匹配于 `$formatFile`
的内容所定义的格式时报告错误，错误讯息由 `$message` 指定。

`assertStringNotMatchesFormatFile()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class StringMatchesFormatFileTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertStringMatchesFormatFile('/path/to/expected.txt', 'foo');
        }
    }

assertSame()
------------

`assertSame(mixed $expected, mixed $actual[, string $message = ''])`

当两个变量 `$expected` 和 `$actual`
的值与类型不完全相同时报告错误，错误讯息由 `$message` 指定。

`assertNotSame()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class SameTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertSame('2204', 2204);
        }
    }

`assertSame(object $expected, object $actual[, string $message = ''])`

当两个变量 `$expected` 和 `$actual`
不是指向同一个对象的引用时报告错误，错误讯息由 `$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class SameTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertSame(new stdClass, new stdClass);
        }
    }

assertStringEndsWith()
----------------------

`assertStringEndsWith(string $suffix, string $string[, string $message = ''])`

当 `$string` 不以 `$suffix` 结尾时报告错误，错误讯息由 `$message` 指定。

`assertStringEndsNotWith()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class StringEndsWithTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertStringEndsWith('suffix', 'foo');
        }
    }

assertStringEqualsFile()
------------------------

`assertStringEqualsFile(string $expectedFile, string $actualString[, string $message = ''])`

当 `$expectedFile` 所指定的文件其内容不是 `$actualString`
时报告错误，错误讯息由 `$message` 指定。

`assertStringNotEqualsFile()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class StringEqualsFileTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertStringEqualsFile('/home/sb/expected', 'actual');
        }
    }

assertStringStartsWith()
------------------------

`assertStringStartsWith(string $prefix, string $string[, string $message = ''])`

当 `$string` 不以 `$prefix` 开头时报告错误，错误讯息由 `$message` 指定。

`assertStringStartsNotWith()` 是与之相反的断言，并接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class StringStartsWithTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertStringStartsWith('prefix', 'foo');
        }
    }

assertThat()
------------

可以用 `PHPUnit\Framework\Constraint` 类来订立更加复杂的断言。随后可以用
`assertThat()`
方法来评定这些断言。appendixes.assertions.assertThat.example
展示了如何用 `logicalNot()` 和 `equalTo()` 约束条件来表达与
`assertNotEquals()` 等价的断言。

`assertThat(mixed $value, PHPUnit\Framework\Constraint $constraint[, $message = ''])`

当 `$value` 不符合约束条件 `$constraint` 时报告错误，错误讯息由
`$message` 指定。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class BiscuitTest extends TestCase
    {
        public function testEquals(): void
        {
            $theBiscuit = new Biscuit('Ginger');
            $myBiscuit  = new Biscuit('Ginger');

            $this->assertThat(
              $theBiscuit,
              $this->logicalNot(
                $this->equalTo($myBiscuit)
              )
            );
        }
    }

appendixes.assertions.assertThat.tables.constraints 列举了所有可用的
`PHPUnit\Framework\Constraint` 类。

assertTrue()
------------

`assertTrue(bool $condition[, string $message = ''])`

当 `$condition` 为 `false` 时报告错误，错误讯息由 `$message` 指定。

`assertNotTrue()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class TrueTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertTrue(false);
        }
    }

assertXmlFileEqualsXmlFile()
----------------------------

`assertXmlFileEqualsXmlFile(string $expectedFile, string $actualFile[, string $message = ''])`

当 `$actualFile` 对应的 XML 文档与 `$expectedFile` 对应的 XML
文档不相同时报告错误，错误讯息由 `$message` 指定。

`assertXmlFileNotEqualsXmlFile()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class XmlFileEqualsXmlFileTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertXmlFileEqualsXmlFile(
              '/home/sb/expected.xml', '/home/sb/actual.xml');
        }
    }

assertXmlStringEqualsXmlFile()
------------------------------

`assertXmlStringEqualsXmlFile(string $expectedFile, string $actualXml[, string $message = ''])`

当 `$actualXml` 对应的 XML 文档与 `$expectedFile` 对应的 XML
文档不相同时报告错误，错误讯息由 `$message` 指定。

`assertXmlStringNotEqualsXmlFile()` 是与之相反的断言，并接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class XmlStringEqualsXmlFileTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertXmlStringEqualsXmlFile(
              '/home/sb/expected.xml', '<foo><baz/></foo>');
        }
    }

assertXmlStringEqualsXmlString()
--------------------------------

`assertXmlStringEqualsXmlString(string $expectedXml, string $actualXml[, string $message = ''])`

当 `$actualXml` 对应的 XML 文档与 `$expectedXml` 对应的 XML
文档不相同时报告错误，错误讯息由 `$message` 指定。

`assertXmlStringNotEqualsXmlString()` 是与之相反的断言，接受相同的参数。

    <?php declare(strict_types=1);
    use PHPUnit\Framework\TestCase;

    final class XmlStringEqualsXmlStringTest extends TestCase
    {
        public function testFailure(): void
        {
            $this->assertXmlStringEqualsXmlString(
              '<foo><bar/></foo>', '<foo><baz/></foo>');
        }
    }
