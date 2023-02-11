## 基本的数据类型

###  1. Objects everywhere

#### 基本类型也是对象

* `Groovy` 中数字字面量
	* ![[Pasted image 20230113232933.png]]


### 2. Optional typing 

#### 1.  声明变量

1. 显式声明与隐式声明(使用关键字 `def` )
2. `Groovy` 会在运行时检查类型安全，而编译期不会。
3. 在 `Groovy` , 如果两个类型可直接转换，前提两者可互相转换， 具体转换实现参照  `DefaultTypeTransformation.castToType` 

###  3.  Overriding operators

#### Groovy 内置的重载

1.  `Groovy` 对常用操作符进行了重载；

#### 实现自定义的重载

 实现对应方法实现对操作符的重载
```groovy
import groovy.transform.Immutable;

@Immutable
class Money {
	int amount;
	String currency;

	Money plus (Money other) {
		if (null == other) return this;
		if (other.currency != currency) {
			throw new IllegalArgumentException("cannot add $other.currency to $currency")
		}
		return new Money(amount + other.amount, currency)
	}

	Money plus (Integer more) {
		return new Money(amount + more, currency);

	}
}
```

3. `强制转换` 时注意事项，当两个不同类型的对象；
	1. 仅支持可被允许的类型，不允许的类型抛出 `IllegalArgumentException`
	2. 如果参数类型不同于当前类型，提升参数类型为当前类型并且返回一个当前类型的对象。`BigDecimal.plus(Integer)`
	3. 用`双重分派(double dispatch)`处理普遍的参数； 
		*  a.method(b) 调用 b.method(a), 通过重载 `method(typeA) method(typeB)` 来实现；


###  4. Work with String

在 `groovy 有两种字符串类型
		1. `plain Strings ` are instances of `java.lang.String`
		2. `GString` are instances of `groovy.lang.GString`
		* `string interpolation` 字符串插值，在运行时解析和执行字符串的占位符；

#### 字符串字面量的类型

**指定字符串字面量**

![[Pasted image 20230114210450.png]]

**如何定义一个 char**
```groovy
char a = 'x'
Character b = 'x'
```

**如何将 string 转为 char**
```groovy
'x' as char
'x'.toCharacter()
```
#### Gstring
* 带有占位符的字符串，占位符形式可以为 `${expression}` 或者缩写 `$expresssion

### 5.  Working with regular expresssions

#### 操作符

`Groovy` 添加了三个便捷操作符
* `=~` : The regex find operator;
* `==~`: The regex match operator;
* `~string` : The regex pattern operator;

#### 示例

**示例程序**
```groovy
def twister = "she sells sea shells at the sea shore of seychelles"  
assert twister =~ /s.a/  
// find expression evaluates to a Matcher object;  
def finder = twister =~ /s.a/  
assert finder instanceof java.util.regex.Matcher;  
  
// twister must contain only words delimited by single spaces;  
assert twister ==~ /(\w+ \w+)*/  
def REGEX_WORD = /\w+/  
// Match expression evaluates to a Boolean  
def matchers = (twister ==~ /($REGEX_WORD $REGEX_WORD*)/)  
assert matchers instanceof java.lang.Boolean  
  
// Match is full unlike find;  
assert !(twister ==~ /s.e/)  
  
def wordsByX = twister.replaceAll(REGEX_WORD, 'x')  
assert wordsByX ==  "x x x x x x x x x x"  
  
def words = twister.split(/ /)  
assert words.size() == 10  
assert words[0] == 'she'
```

####  处理每个匹配

* `String.eachMath`,有两个参数，一个为正则表达式，另一个 针对每个匹配作处理 的闭包
* `Mather.each()`, 针对每个匹配作处理
* `replaceAll()`
```groovy
def myFairStringy = "The rain in Spain stays my mainly in the plain"
def wordEnding = /\w*ain/
def rhyme = /\b$wordEnding\b/
def found = ""
myFairStringy.eachMatch(rhyme) {match -> {
    found += match + ' '
}}
assert fonud == 'rain Spain plain'
found = ''
(myFairStringy =~ rhyme).each {match -> {
    found += match + ' '
}}
assert fonud == 'rain Spain plain'

def cloze = myFairStringy.replaceAll(rhyme){it-'ain'+'___'}
assert cloze == 'The r___ in Sp___ stays my mainly in the plain'
```

##### GDK 对于 Matcher 的扩展
* 在 `Groovy` 中, Matcher 可以当作是一个匹配结果列表来操作

```groovy
// The Gdk enhances the Matcher classs;  
def matcher = 'a b c' =~ /\S/  
assert matcher[0] == 'a'  
assert matcher[1..2] == ['b', 'c']  
assert matcher.size() == 3
```

* Groovy's parallel assignment feature
```groovy
def (a, b, c) = 'a b c' =~ /\S/  
assert a == 'a'  
assert b == 'b'  
assert c == 'c'
```

* Grouping in the match
```groovy
def match = 'a:1 b:2 c:3' =~ /(\S+):(\S+)/
assert match.hasGroup()
assert match[0] == ['a:1', 'a', '1']
assert match[1] [2] == '2'

// Matcher's each method
match.each {full, key, value ->
    assert full.size() == 3
    assert key.size() == 1 // a,b,c
    assert value.size() == 1 //1,2,3
}

// String.eachMath(regex){match -> }
'a:1 b:2 c:3'.eachMatch(/(\S+):(\S+)/){m ->
    print(m)
}
```

#### Pattern 的性能
	当创建`Patter`对象时会生成相应的有限状态机，正则表达式越复杂则状态机生成的时间也越长，所以需要将状态机的生成与模式匹配分开，以重用生成的状态机，提高性能，下面是书中给出的性能测试；
```groovy
def twister = 'she sells sea shells at the sea shore of seychelles'  
def regex = /\b(\w)\w*\1\b/  
def many = 100 * 1000  
start = System.nanoTime()  
many.times {  
    twister =~ regex  
}  
timeImplicit = System.nanoTime() - start;  
// Resuing the finite-machine  
start = System.nanoTime()  
pattern = ~regex  
many.times {  
    pattern.matcher(twister)  
}  
timePredef = System.nanoTime() - start  
println "implicit : $timeImplicit predefine : $timePredef "
```

#### 5. Patterns for the classification
* `~Strng` 创建的 `Pattern` 对象实现了 `isCase(String)`方法；它等同于字符串与该模式的完全匹配；
* `classification method` 是使用 `in` 操作符， `grep` 方法, `switch` case 的前置条件。

```groovy
def fourLetters = ~/\w{4}/
assert fourLetters.isCase('work')
assert 'love' in fourLetters
switch ('beer') {
    case fourLetters: assert true; break;
    default: assert false;
}

beasts = ['bear', 'wolf', 'tiger', 'regex']
assert beasts.grep(fourLetters) == ['bear', 'wolf']
```

### 6. Working with numbers

##### 1. 强制转换

对于 `+ - *`,  判断逻辑如下
* 如果两个操作数为 `Float` 和 `Double`, 那么结果数类型为 `Double`
* 如果操作数中有一个为`BigDecimal`, 那么结果为 `BigDecimal`
* 如果操作数中有一个为`BigInteger`, 那么结果为`BigInteger`
* 如果操作数中有一个为`Long` ，对么结果为`Long`
* 以上都不是； 则结果为 `Integer`

对于 `/` 除法，有以下规则:
* 如果有操作数为 `Float` 或 `Dobule` 那么结果为 `Double
* 否则 ，结果为`BigDecimal`, 其精度为两个操作数中最大的，并对结果进行四舍五入；并舍去多余的 0;

两个`Integer` 相除结果为 BigDecimal, 如果希望是`Integer`, 使用 `intDiv()`方法

#### 2. Gdk methods for number
```groovy
assert 1 == (-1).abs()  
assert 2 == 2.5.toInteger() // conversion  
assert 2 == 2.5 as Integer // enforced coercion  
assert 2 == (int) 2.5 // cast  
assert 3 == 2.5f.round()  
assert 3.142 == Math.PI.round(3)  
assert 4 == 4.5f.trunc()  
assert 2.718 == Math.E.trunc(3)  
assert '2.718'.isNumber() // String methods  
assert 5 == '5'.toInteger()  
assert 5 == '5' as Integer  
assert 53 == (int) '5' // gotcha!  
assert '6 times' == 6 + ' times' // Number + String
```

* 不要强制字符串为数值，如果字符串长度为1，那么将转为其`unicode`码值
```groovy
def a = (init)"1"
pritnln a // 49
```
* 使用类型转换来转换字符串为数值
```groovy
assert 5 == '5'.toInteger()  
assert 5 == '5' as Integer 
```

**times, upto, downto step**
* `times` is just for repetition;
* `upto` is for walking a sequence of increasing numbers;
* ` donwto` is for decreasing numbers
* `step` is the general version that walks until the end value by successively adding a step width;

```groovy
def store = ''
10.times {
    store += 'x'
}
assert store == 'xxxxxxxxxx'

store = ''
1.upto(5) { number ->
    store += number;
}
assert store == '12345'

store = ''
2.downto(-3) {number ->
    store += number + ' '
}
assert store == '2 1 0 -1 -2 -3 '

store = ''
0.step(0.5, 0.1) { number ->
    store += number + ' '
}
assert store == '0 0.1 0.2 0.3 0.4 '
```


## 复合的数据类型

### 1.  Working with ranges

##### 基本使用

* *`groovy.lang.Range`

三种形式
* `left..right`
* `(left..right)`
* `(left..<right)`  The value of the right isn't part of the range;

```groovy
assert (0..10).contains(0)  
assert (0..10).contains(5)  
assert (0..10).contains(10)  
  
assert !(0..10).contains(-1)  
assert !(0..10).contains(11)  
  
assert (0..<10).contains(9)  
assert !(0..<10).contains(10)  
  
def a = 0..10  
assert a instanceof Range  
assert a.contains(5)  
  
a = new IntRange(0, 10)  
assert a.contains(5)  
  
assert (0.0..1.0).contains(1.0)  
assert (0.0..1.0).containsWithinBounds(0.5)  
```
* `Date` 对象也可以用于 `range`, 因为 `GDK` 给 `Date`添加了 `previous` 与 `next` 方法；`GDK`也给`Date`添加了 `minus and plus operator`
```groovy
def today = new Date()  
def yesterday = today - 1  
assert (yesterday..today).size() == 2  
```
* `String` 同样添加了 `previous` 和 `next` 方法
```groovy
assert ('a'..'c').contains('b')
```
* 使用 `for...in..range`方法来遍历 `range`
```groovy
def log = ''  
for (element in 5..9) {  
    log += element  
}  
assert log == '56789'  
  
log = ''  
for (element in 9..5) {  
    log += element  
}  
assert log == '98765'  
  
log = ''  
(9..5).each {element ->  
    log += element  
}  
assert log == '98765'
```

##### 2. Ranges are objects
* `Range`是对象`groovy.lang.Range`, 可调用其方法，其中最常用的是`each`方法
```groovy
def result = ''
(5..9).each { element ->
    result += element
}
assert result == '56789'
```
* `Range`继承`List`, 本质是`Collection`;  所以`Range`可用于基于`Collection`的运算符重载；本质是调用`contains`方法
```groovy
// The syntactic sugar of isCase
assert 5 in 0..10 // the
assert (0..10).isCase(5)

// switch case
def age = 36
switch (age) {
    case 16..20: insuranceRate = 0.05; break;
    case 21..50: insuranceRate = 0.06; break;
    case 51..65: insuranceRate = 0.07; break;
    default: throw new IllegalArgumentException()
}
assert insuranceRate == 0.06

def ages = [20, 36, 42, 56]
def midage = 21..50
assert ages.grep(midage) == [36, 42]
```

##### 3.  实现可用于Range的类
* 实现 `previous` 与 `next`方法，这两个方法重载了`++`和`--`操作符
* 实现 `java.lang.Comparable`接口。该接口的`compareTo`方法重载了`<=>`操作符。

**示例**

```groovy
class WeekDay implements Comparable {

    static final DAYS = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']

    private int index = 0;

    WeekDay(String day) {
        index = DAYS.indexOf(day);
    }

    WeekDay next() {
        return new WeekDay(DAYS[index + 1 % DAYS.size()])
    }

    WeekDay previous() {
        return new WeekDay(DAYS[index - 1]);
    }

    @Override
    int compareTo(Object o) {
        return this.index <=> o.index
    }

    @Override
    String toString() {
        return DAYS[index];
    }
}

// test
def mon = new WeekDay('Mon')
def fri = new WeekDay('Fri')

def worklog = ''
for (day in mon..fri) {
    worklog += day.toString() + ' '
}

assert worklog == 'Mon Tue Wed Thu Fri '

```


### 2. Working with lists

#### 1. List的基本操作

##### 定义list

* Groovy 中 list 中的元素可以是任意类型或或者是不同的类型；

```groovy
[item1, item2, item3]
```

**使用示例**

* `Groovy` 中 `array`, `collection objects`, `string` 都扩展了 `toList`方法；
```groovy
myList = [1, 2, 3]

assert myList.size() == 3
// The resulting list can still be used with subscript operator;
assert myList[0] == 1
// Lists are by default of type java.util.ArrayList
assert myList instanceof ArrayList

// The sequence can be empty to declare an empty list;
List emptyList = []
assert emptyList.size() == 0

// Lists can be created and initialized at the same time by calling toList on range;
List longList = (0..1000).toList()
assert longList[555] == 555


// Lists are also be declared explicitly by calling the respective constructor;
List explicitList = new ArrayList()
explicitList.addAll(myList)
assert explicitList.size() == 3
explicitList[0] = 10
assert explicitList[0] == 10

explicitList = new LinkedList(myList)
assert explicitList.size() == 3
explicitList[0] = 10
assert explicitList[0] == 10
```

#### 2. 使用 List 操作符

##### 下标操作符

* Groovy 重载了 `getAt`方法，接收`range` 和集合参数；用于一段范围或指定下标的数据；
* 也重载了 `putAt`方法，接收`range`参数；将一个值列表赋值给整个子列表(原列表中)；
* 使用 `Range` 进行下标分配，不需要指定大小，如果值列表小于 `range`（或者是 `empty`）, 那么原列表将会缩小；当值列表大于`range`, 那么原列表将会扩容 ；

**使用示例**
```groovy
myList = ['a', 'b', 'c', 'd', 'e', 'f']
assert myList[0..2] == ['a', 'b', 'c']
assert myList[0, 2, 4] == ['a', 'c', 'e']

myList[0..2] = ['x', 'y', 'z']
assert myList == ['x', 'y', 'z', 'd', 'e', 'f']

myList[3..5] = []
assert myList = ['x', 'y', 'z']

myList[1..1] = [0, 1, 2]
assert myList = ['x', 0, 1, 2, 'z']
```
- 支持负数索引； `list[-1]`返回最后一个元素，`list[-2]` 返回倒数第二个元素；
- 支持负数索引组成的`Range`;  `list[-3...-1]` 返回最后三个元素
- 支持混合索引； `list[1..-2]` 去除第一个元素和最后一个元素；
* 支持`half-exclusive` , 如 `list[0..<-2]`

##### 添加和删除元素

* 支持的操作 
	* `plus(Object)`, `plus(Collection)`
	* `leftShift(Object)`
	* `minus(Collection)`,
	* `multiply`

**使用示例**

```groovy
myList = []  
  
myList += 'a'  
assert myList == ['a']  
  
myList += ['b', 'c']  
assert myList == ['a', 'b', 'c']  
  
myList = []  
myList << 'a' << 'b'  
assert myList == ['a', 'b']  
  
assert myList - ['b'] == ['a']  
  
assert myList * 2 == ['a', 'b', 'a', 'b']

```

##### list 应用于控制结构

```groovy
myList = ['a', 'b', 'c']  
assert myList.isCase('a')  
assert 'b' in myList  
  
def candidate = 'c'  
switch (candidate) {  
    case myList: assert true; break;  
    default: assert false  
}  
  
assert ['x', 'a', 'z'].grep(myList) == ['a']  
myList = []  
if (myList) assert false  
  
// Lists can be iterated with 'for' loop  
def expr = ''  
for (i in [1, '*', 5]) {  
    expr += i  
}  
assert expr == '1*5'

```


#### 3. 使用 list 的方法

##### 操作 List 的内容

1. List 中的元素类型可以是任意类型 
2. 如何去除重复元素：
	- 转换为 `Set`
	- 使用 `unique()`方法， 此方法不会元素的顺序；
3. 如何去除 `null`元素
	- 使用 `findAll` keeping all non-nulls
	- 使用`grep`
```groovy
def x = [1, 1, 1]
assert [1] == new HashSet(x).toList()
assert [1] == x.unique()


// Removing null
def x = [1, null , 1]
assert [1, 1] == x.findAll{it != null }
assert [1, 1] == x.grep{it}
```

**使用示例**
```groovy
assert [1, [2, 3]].flatten() == [1, 2, 3]  
assert [1, 2, 3].intersect([4, 3, 1]) == [1, 3]  
assert [1, 2, 3].disjoint([4, 5, 6])  
  
// treating a list like a stack  
list = [1, 2, 3]  
poped = list.pop()  
assert poped == 1  
assert list == [2, 3]  
  
assert [1, 2].reverse() == [2, 1]  
assert [3, 1, 2].sort() == [1, 2, 3]  
  
// comparing lists by first element  
def list = [[1, 0], [0, 1, 2]]  
// <=> compareTo();  
list = list.sort { a, b -> a[0] <=> b[0] }  
assert list == [[0, 1, 2], [1, 0]]  
  
// comparing lists by size  
list = list.sort { item -> item.size() }  
assert list == [[1, 0], [0, 1, 2]]  
  
// removing by index  
list = ['a', 'b', 'c']  
list.remove(2)  
assert list == ['a', 'b']  
// removing by value  
list.remove('b')  
assert list == ['a']  
  
  
// transforming one list into another  
def doubled = [1, 2, 3].collect { item -> item * 2 }  
assert doubled == [2, 4, 6]  
  
// finding every element matching the closure  
def odd = [1, 2, 3].findAll { item -> item % 2 == 1 }  
assert odd == [1, 3]

```

##### 访问 List 内容

**查询**
1. `count`, `min` , `max`, `first`, `head`, `tail`, `last`
2. `find`
3. `every`, `any`

**迭代**
1. `each`, `reverseEach`, `eachWithIndex`

**Accumulating**
1. `inject`, `sum`

```groovy
def list = [1, 2, 3]  
assert list.first() == 1  
assert list.head() == 1  
assert list.tail() == [2, 3]  
assert list.last() == 3  
assert list.count(2) == 1  
assert list.max() == 3  
assert list.min() == 1  
  
def even = list.find { item ->  
    item % 2 == 0  
}  
assert even == 2  
  
assert list.every { it < 5 }  
assert list.any { it -> it < 2 }  
  
def store = ''  
list.each { it ->  
    store += it  
}  
assert store == '123'  
  
store = ''  
list.reverseEach { it ->  
    store += it  
}  
assert store == '321'  
  
store = ''  
list.eachWithIndex { item, index ->  
    store += "$index:$item "  
}  
assert store == '0:1 1:2 2:3 '  
  
assert list.join('-') == '1-2-3'  
  
// accumulating  
result = list.inject(0) {clinks, guests ->  
    clinks + guests  
}  
assert result == 0 + 1 + 2 + 3  
assert list.sum() == 6  
  
factorial = list.inject(1) {fac, item ->  
    fac * item  
}  
assert factorial == 1 * 1 * 2 * 3
```

###  3.Working with maps

####  定义Map


###  4. Notes on Groovy collections 


### 留在后文

1. `Matcher` object, it can be used as a Boolean conditional;