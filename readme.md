# java8 新的时间api

--
这是一次修改
本篇文章分为三个部分：
1. 基础的日期时间对象的使用
2. 操作和解析日期时间对象
3. 基于时区的调整，使用不同的历法 
---

由于众所周知的原因，java中的`java.util.Date`和`java.util.Calendar`无论从设计上还是使用上都存在问题，同时也不适应新的函数式编程的新浪潮。出于多方面原因的考虑，最后在java8中新增了`java.time`，这个专门处理时间相关问题的包。

## 1. LocalDate, LocalTime, Instant, Duration, Period

想了解time包中的时间api， `LocalDate, LocalTime, Instant, Duration, Period` 这几个类库应该是最基础的内容。

### 1.1 使用LocalDate, LocalTime

使用新的日期和时间api，`LocalDate`和`LocalTime`应该是基础中的基础，我们来一个一个了解。

**LocalDate**

`LocalDate` 第一次使用肯定会与`Date`产生联想，其实作为`time`包中基础的类，`LocalDate`和原`Date`对象有很大的不同。

首先，`LocalDate`对象是不可变对象（类似于`String`，对象的属性和值不可改变）；其次，只提供简单的日期信息，并不包含日期当天的时分秒等时间信息；当然也不包含任何时区相关的信息。所以简单来说，`LocalDate`对象是只记录了简单日期信息的**不可变**对象。

可以通过静态方法`of()`获取指定的日期或者使用工厂`now()`方法获取当前日期

```java
/**
    * 创建LocalDate对象的几种方式
    *
    * 1. of 方式
    * 2. now 方式
    *
    * 输出：
    * 2018-04-20
    * 2018-05-09
    */
public static void createLocalDateDemo() {

    // 通过of方法创建LocalDate对象
    int year = 2018;
    Month month = Month.of(4);
    // month是内置的枚举，直接通过具体的值指定月份也可以
    Month month2 = Month.APRIL;
    int day = 20;
    LocalDate ofDate = LocalDate.of(year, month, day);
    System.out.println(ofDate.toString());

    // 通过静态方法now 创建，当前日期
    LocalDate nowDate = LocalDate.now();
    System.out.println(nowDate);
}
```

读取`LocalDate`的属性也很简单：

```java
/**
 * 读取localDate的属性
 *
 * 输出：
 * year:2018  month:5  day-of-month:9
 */
public static void getLocalDateField() {

    LocalDate now = LocalDate.now();

    int year = now.getYear();
    Month month = now.getMonth();
    int day = now.getDayOfMonth();

    System.out.print("year:" + year + "  ");
    System.out.print("month:" + month.getValue() + "  ");
    System.out.println("day-of-month:" + day);
}
```

除了简单的年月日之外，`LocalDate`还记录了一些额外日期的信息

```java
// 除了简单的年月日之外，LocalDate还记录了一些十分有用的和日期相关的信息
boolean isLeapYear = now.isLeapYear(); // 是否是闰年
int lengthOfMonth = now.lengthOfMonth(); // 当前月份有多少天
DayOfWeek dow = now.getDayOfWeek(); // 当前是周几
```

如果你有看到`LocalDate`的源码，你会发现`LocalDate`实现了`Temporal`接口。所以，还可以通过传递`TemporalField`参数给`get`方法来获取指定的信息。`TemporalField`是一个接口，定义了如何访问`temporal`对象的某个字段。而`ChronoField`枚举则实现了这一接口，所以可以很方便的使用`get`方法获取到枚举元素的值。

```java
/**
* 通过Temporal接口获取LocalDate的属性
* 
* 输出：
* 通过Temporal访问LocalDate的属性：2018-5-9 
*/
public static void getFieldByTemporal() {

    LocalDate now = LocalDate.now();

    int year = now.get(ChronoField.YEAR);
    int month = now.get(ChronoField.MONTH_OF_YEAR);
    // 如果想把int类型的month转为枚举，可以使用of 方法
    Month month2 = Month.of(month);
    int day = now.get(ChronoField.DAY_OF_MONTH);

    System.out.println("通过Temporal访问LocalDate的属性：" + year + "-" +  month + "-" + day);
}
```

**LocalTime**

`LocalTime`存储的是单纯的时间信息，不包含日期。除此之外基本和`LocalDate`的属性相似，都是不可变对象，实现了`Temporal`接口等等。

首先来看`LocalTime`的创建

```java
/**
* 创建LocalTime
*
* 输出：
* 10:39:44.951
* 12:12:12.000100
*/
public static void createLocalTime() {

    // 通过工厂方法now创建
    LocalTime now = LocalTime.now();
    System.out.println(now);

    // 通过制定参数of 方法创建
    LocalTime ofTime = LocalTime.of(12, 12, 12,100000);
    System.out.println(ofTime);
}
```

然后是对应的属性的读取，同样也是和`LocalDate`类似

```java
/**
* 读取LocalTime的值
*
* 输出：
* 10:50:54
* 10:50:54
*/
public static void getLocalTimeField() {

    LocalTime nowTime = LocalTime.now();

    int hour = nowTime.getHour();
    int minute = nowTime.getMinute();
    int second = nowTime.getSecond();
    System.out.println(hour + ":" + minute + ":" + second);

    // 同样，LocalTime 也可以通过Temporal来获取指定属性的值
    int tempOfHour = nowTime.get(ChronoField.HOUR_OF_DAY);
    int tempOfMinute = nowTime.get(ChronoField.MINUTE_OF_HOUR);
    int tempOfSecond = nowTime.get(ChronoField.SECOND_OF_MINUTE);
    System.out.println(tempOfHour + ":" + tempOfMinute + ":" + tempOfSecond);

}
```

以上是`LocalDate`和`LocalTime`的基本使用，但是实际开发中其实我们用的最多的是格式化的String转为日期和时间对象。当然，新的时间api在这方面的支持也是相当完善的，而且比以前的效果更好更简洁：

```java
LocalDate date = LocalDate.parse("2018-12-12");
LocalTime time = LocalTime.parse("12:12:12");
System.out.println(date + " " + time); //输出：2018-12-12 12:12:12
```

查看源码可以看出，这里其实是使用默认的标准的ISO formatter，`DateTimeFormatter`是新版的时间格式化类，规定的如何将`String` 和`Local` 系列的日期时间对象对应起来，实际使用中可以使用该对象来完成字符串和日期时间对象之间的互转。

![LocalDate parse 源码](https://upload-images.jianshu.io/upload_images/1156415-7e33c7e1fd55f6ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 1.2 使用LocalDateTime

实际开发中，我们很少会将日期和时间拆开使用，大多数情况下两者都是存在的。新的time包中有`LocalDateTime` 这一组合对象，此时的`LocalDateTime`有一点点类似于`Date`了，同时存有日期和时间。不同之处在于`LocalDateTime`仍然是不可变对象，且不包含任何时区信息。

![LocalDateTime 内部使用的是LocalDate和LocalTime](https://upload-images.jianshu.io/upload_images/1156415-3c9c25330f5a20bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

创建`LocalDateTime`的方式和`LocalDate`与`LocalTime`类似

```java
/**
* 创建LocalDateTime 对象
*
* 指定时间为：2018-5-11T12:12:12
*/
public static void createLocalDateTime() {

    // 通过of 方法直接创建
    LocalDateTime dateTime = LocalDateTime.of(2018, 5, 11, 12, 12, 12);

    // 通过LocalDate和Time 合并实现
    LocalDate date = LocalDate.of(2018, 5, 11);
    LocalTime time = LocalTime.of(12, 12, 12);
    LocalDateTime dateTime2 = LocalDateTime.of(date, time);

    // 通过LocalDate的 atTime 创建
    LocalDateTime dateTime3 = date.atTime(time);
    LocalDateTime dateTime4 = date.atTime(12, 12, 12);

    // 通过LocalTime的 atDate 创建
    LocalDateTime dateTime5 = time.atDate(date);
}
```

因为是组合对象，所以可读取一部分来获取`LocalDate`或者`LocalTime`

```java
LocalDateTime now = LocalDateTime.now();
LocalDate date = now.toLocalDate();
LocalTime time = now.toLocalTime();
```

到目前为止我们了解了`LocalDate`，`LocalTime` 以及 `LocalDateTime` ，它们的关系如下：

![三者的关系](https://upload-images.jianshu.io/upload_images/1156415-e4daa6d585869743.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.3 Instant，关于机器的时间

作为人类，我们理解时间的概念都是几年几月几天几分几秒等等，毫无疑问机器肯定已经不会以这种方式处理时间，这一点从老的`Date`和`Calendar`就可以看出来。所以在`time`包中，类似于时间戳的这种底层的处理时间的类为`Instant`。

当然，我们最好的理解应该是：`Instant`是与机器交互的时间处理类。因此`Instant`不需要记录年，月，日等等，类似于时间戳，`Instant`记录的是从Unix元年（UTC时区1970年1月1日午夜零分）到现在的秒数，可以通过`ofEpochSecond`工厂方法创建，当然还存在一个增强版本，可以额外的接口一个以纳秒为单位的数值，来精确的记录时间。

```java
/**
* 创建Instant
*/
public static void createInstant() {

    // 通过工厂方法now创建
    Instant instant = Instant.now();

    // 通过工厂方法ofEpochSecond创建
    Long timestamp = instant.getEpochSecond();
    Instant instant1 = Instant.ofEpochSecond(timestamp);

    // 增强版本，可以传递一个纳秒，
    Instant instant2 = Instant.ofEpochSecond(1000);
    Instant instant3 = Instant.ofEpochSecond(1000, 0L);
    Instant instant4 = Instant.ofEpochSecond(999, 1_000_000_000L);
    Instant instant5 = Instant.ofEpochSecond(998, 2_000_000_000L);
    Instant instant6 = Instant.ofEpochSecond(1001, -1_000_000_000L);
}
```

> 关于`ofEpochSecond(int second, long nanoSecond)`的增强版本，会将纳秒调整在0~999,999,999 之间。所以当纳秒数超过这个范围的时候，程序会根据具体的值进行调整。所以，demo代码中的这几种方式创建的Instant都是相等的。
![增强重载版本的源码](https://upload-images.jianshu.io/upload_images/1156415-0420f7b389e29bb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Instant`是设计用于和机器交互的时间类，虽然实现了`Temporal`接口，但是内部是没有年月日，时分秒等属性的，因此一下代码的调用会扔出`Runtime异常`

```java
// 会扔出运行时异常
int day = Instant.now().get(ChronoField.DAY_OF_MONTH);
```
![image.png](https://upload-images.jianshu.io/upload_images/1156415-1bb6fe66908fc780.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 1.4 Duration和Period

处理常规的表达时间点的概念之外，time新增了表示时间段的类`Duration`和`Period`。在java8之前，我们只能通过数字加人为规定的单位来表达时间段这一概念。但是在实际开发中，老的时间api计算时间段是真的不方便，而且效率低下。使用的新表达时间段的类就可以很方便的解决这个问题

创建`Duration`的方式很简单，使用`between`方法即可，可以传入两个`Instant`，两个`LocalDateTime`或者两个`Localtime`对象来进行创建

```java
LocalDateTime from = LocalDateTime.of(2018, 4, 1, 0, 0, 0);
LocalDateTime to = LocalDateTime.now();
Duration duration = Duration.between(from, to);
System.out.println(duration); // PT953H28M16.279S 代表：953小时28分钟16.279秒
```

> 为什么不能将`Instant`和`LocalDateTime`混用呢，因为`Instant`是给机器设计的，`LocalDateTime`是给人设计的，两个目的不一样，因此不能混用。除此之外，`Duration`类是主要以秒和纳秒来表达时间段的，从单位上来说比较精确，因此也不能使用`LocalDate`来计算两个日期之间的时间段。

当然，如果要表达最小以天为单位的时间段，就可以使用`Period`类

```java
LocalDate from = LocalDate.of(2018, 4, 1);
LocalDate to = LocalDate.of(2018, 5, 2);
Period period = Period.between(from, to);
System.out.println(period); // P1M1D  表示：1个月零2天
```

到这里，我们就很明白了。`Duration`和`Period`都可以表示一段时间。两者最主要的却别在于度量的单位不同，`Duration`主要是以时分秒甚至于毫秒来较为精确的度量一段时间，而`Period`则是从年月日的角度来表示一段时间。实际开发中，可以视不同的业务需求来使用。

除了`between`之外，`Duration`和`Period`还有很多工厂方法来获取实例化的时间对象

```java
Duration threeMinutes = Duration.ofMinutes(3);  // 三分钟
Duration fiveMinutes = Duration.of(5, ChronoUnit.MINUTES); //五分钟

Period threeDays = Period.ofDays(3); //三天
Period twoWeeks = Period.ofWeeks(2); // 两周
Period oneYear = Period.ofYears(1); // 一年
Period fiveMonth = Period.ofMonths(5); //五个月
Period towYearsOneMonthTenDays = Period.of(2, 1, 10); // 两年一个月零十天
```

上述代码中只是简单地举了一个例子，其实`Duration`和`Period`中有很多相似的工厂方法来创建实例化的时间段。

| 方法名 | 是否是静态方法 | 方法描述|
|-----|----|-----|
|between| 是 | 创建两个时间点之间的interval|
|from| 是 | 由一个临时节点创建interval|
|of| 是 | 由它的组成部分创建interval的实例|
|parse| 是 | 由字符串创建nterval|
|addTo| 否 | 创建该interval的副本，并将其叠加到某个指定的Temporal对象|
|get| 否 | 读取该interval的状态|
|isNegative | 否 | 检查该interval是否为负值，不包含0|
|isZero| 否| 检查该interval是否为0|
|minus| 否 |减去一定的时间创建interval的副本|
|multipliedBy| 否 | 将interval乘以某个标量来创建其副本|
|negated| 否 | 以忽略某个时长的方式创建interval的副本|
|plus| 否 | 以增加某个时长的方式创建interval的副本|
|subtractFrom| 否 | 从指定的temporal对象中减去该interval来创建其副本|

## 2. 操作和解析日期与时间

除了创建和读取日期时间对象，实际开发中不可避免的存在修改，解析日期时间对象的需求，下面对这方面的内容进行讲解。

### 2.1 操作日期和时间对象

**with操作**

首先是修改日期时间对象。第一部分反复强调，以上我们提到的所有的日期时间对象都是固定的不可更改的对象。所以，下文除非特殊说明的情况下都是基于原对象修改后返回的新日期时间对象，而原对象的属性值都不变。

最常用的基本的修改日期和时间对象属性的方法是`withAttribute` 类型的方法。

```java
// 2018-04-01
LocalDate date = LocalDate.of(2018, 4 , 1);

// 使用 withAttribute 类型的方法可基于已有对象的属性修改创建得到新的日期对象，原对象不变
LocalDate date2 = date.withDayOfMonth(12); // 2018-04-12
System.out.println(date + " => " + date2);
LocalDate date3 = date2.withYear(2019); // 2019-04-12
System.out.println(date2 + " => " + date3);
```

当然，除了固定的修改某个字段的`with`方法之外还有通用的`with`方法，因为我们上面提到的所有的日期时间对象都实现了`Temporal`接口，这个就不在赘述，举例如下：

```java
// 也可以使用通用的with方法来对指定的属性进行修改, 比如之类指定修改月份这一属性
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 10); // 2019-10-12
System.out.println(date3 + " => " + date4);
```

**加减操作**

`with`类型的方法是直接基于原有属性修改为指定的属性，除此之外开发中也会存在基于已有时间的加减操作。比如两周之后，五个月之前等等。

```java
// 2018-04-01
LocalDate date = LocalDate.of(2018, 4, 1);

LocalDate date2 = date.plusDays(10); // 2018-04-11
System.out.println(date + " => " + date2);
LocalDate date3 = date2.minusMonths(2).plusYears(1); // 2018-02-11 => 2019-02-11
System.out.println(date2 + " => " + date3);

// 19年2月为28天，所以四周后为 2019-03-11
LocalDate date4 = date3.plus(4, ChronoUnit.WEEKS);
System.out.println(date3 + " => " + date4);
```

总结一下：到这里我们讲了两种操作方法**with类型的方法**和**加减类型的方法**。需要说明的是`LocalDate`， `LocalTime`， `LocalDateTime` 都是支持上述方法的。且with和加减方法都支持指定单位修改和传入指定单位两种修改模式。前者简单直接调用，后者则更为通用，实际开发中可视具体情况调用。

> 此外，提到了两个Chrono开头的枚举，一个是`ChronoField`，这个指定的日期时间对象的具体属性（比如：时间对象中的一小时的秒数，一秒钟的纳秒数等等，with方法修改的就是直接日期和时间对象的属性）。另一个`ChronoUnit`，这个指的是日期的长度单位（比如：年，月，周等等，加减类型的方法则是基于时间单位进行运算，从而修改日期时间对象的属性）。

`LocalDate`， `LocalTime`， `LocalDateTime`, `Instant` 这几个类中还存在着大量通用型的方法，实际开发中可以针对具体的需求来查看和使用，这里不再一一赘述。

**TemporalAdjuster**

本来讲到这里，关于日期和时间对象的修改已经满足了大部分的需求了。但是，实际开发中我们遇到的变态需求往往才是我们关注的重点，如何满足这一部分的需求才是重点需要描述的内容。

举例一下情况：

- 当前日期后的下一个周日
- 五月的第二个周四
- 当前月的最后一天
- 明年的第一天是周几

以上四个类似基于目前我们了解到的内容处理起来还是比较棘手的，因为这些逻辑都相对来说比较复杂，不是很直接。这个时候就需要`TemporalAdjuster`类来帮助我们更加灵活的处理和计算日期。

首先，`TemporalAdjuster`中预置了很多日常开发中比较常见的调整模式，我们可以借助通用的with方法，来对已有日期进行计算。下面我们对上面的四个例子来进行实现和说明。

```java
// 假设当前是：2018-04-02
LocalDate date = LocalDate.of(2018, 4, 2);

// 当前日期的下一个周日
LocalDate date2 = date.with(nextOrSame(DayOfWeek.SUNDAY));
System.out.println(date2); // 2018-04-08

// 五月的第二个周四
LocalDate date3 = date.plusMonths(1).withDayOfMonth(1) // 先修改日期至5月1日
        .with(nextOrSame(DayOfWeek.THURSDAY)) //如果5月1日为周四，则不往后，所以这里用nextOrSame
        .with(next(DayOfWeek.THURSDAY)); // 第二次，这个日期肯定是周四，所以强制往后，使用next
System.out.println(date3); // 2018-05-10

// 当月的最后一天的日期
LocalDate date4 = date.with(lastDayOfMonth());
System.out.println(date4); // 2018-04-30

// 明年的第一天是周几
DayOfWeek date5 = date.with(firstDayOfNextYear()).getDayOfWeek();
System.out.println(date5.getValue()); // 2 周二
```

**定制TemporalAdjuster**

当然了，这种复杂的日期调整规则除了常见的之外，还有很多奇奇怪怪的需求，这些需求都是预置的规则满足不了的。这个时候我们就需要根据自己的需求来实现对应的逻辑。

要实现自己的`TemporalAdjuster`也十分容易，首先来看一下其源码：

![TemporalAdjuster 函数式接口](https://upload-images.jianshu.io/upload_images/1156415-54aa018ab8b406fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

很明显，这是一个函数式申明的接口，对应的输入输出都是`Temporal`对象。所以，我们只需要针对这个接口实现对应的逻辑即可，如果项目中实现的逻辑较为复杂且多处调用，就可以抽象为静态的工具方法；否则直接使用lambda表达式即可。

这里我们举个例子，实现一个`TemporalAdjuster`，返回当前日期往后的第一个工作日。这里不考虑法定节假日（当然，如果实际项目中有这样的需求，则必须有法定节假日相关的接口或者配置数据，否则没有办法动态实现，因为目前来说国内的节假日都是国家根据当前的情况调整的）

规则抽象：
> 如果当前是周一到周四，则返回当前日期的下一天，否则返回下一个周一

实现：
```java
// 下一个工作日的实现
TemporalAdjuster nextWorkingDay = (Temporal temporal) -> {
    Objects.requireNonNull(temporal);
    int dayOfWeek = temporal.get(ChronoField.DAY_OF_WEEK);
    if (dayOfWeek >= 1 && dayOfWeek <= 4) {
        return temporal.plus(1L, ChronoUnit.DAYS);
    }
    return temporal.with(next(DayOfWeek.MONDAY));
};

// 测试
LocalDate date = LocalDate.of(2018, 4, 1);
System.out.println(date.with(nextWorkingDay)); // 2018-04-02
LocalDate date2 = date.plusMonths(3).plusDays(2);
System.out.println(date2.with(nextWorkingDay)); // 2018-07-04
```

### 2.2 解析和格式化日期和时间对象

处理日期和时间相关方面的业务，还有一个很重要的方面就是格式化输出日期和解析日期相关的字符串。在java8中，`java.time.format`包就是用来格式化和解析日期相关的内容。

上文我们提到过格式化输出日期的的类`DateTimeFormatter`就是`java.tiem.format`包下最常用的格式化日期时间的类。接下来的内容就围绕`DateTimeFormatter`来进行讲解。

**DateTimeFormatter基本使用**

`DateTimeFormatter`和原来的`java.util.DateFormat`最大的不同就是其是线程安全的。这是一个十分重要的点，线程安全意味着能够以单例的模式创建格式化的容器，并在多个线程之间共享。除此之外，其实新的`time`包中几乎所有的设计都在强调不可变性，这就意味着在多线程的情况下，新的`time`包中的内容我们可以大胆放心的使用，这在多线程流的配合下，处理大量的日期时间类数据时十分有效的。

关键字： **线程安全**

因为是线程安全的，所以`DateTimeFormatter`内置了很多常用的实例，如下：

```java
// 2018-04-01
LocalDate date = LocalDate.of(2018, 4, 1);

// 格式化输出
String basicIsoDateStr = date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20180401
String isoLocalDate = date.format(DateTimeFormatter.ISO_LOCAL_DATE); // 2018-04-01
System.out.println("格式化输出：\n" + basicIsoDateStr + "\n" + isoLocalDate);

// 解析
LocalDate date2 = LocalDate.parse(basicIsoDateStr, DateTimeFormatter.BASIC_ISO_DATE);
LocalDate date3 = LocalDate.parse(isoLocalDate, DateTimeFormatter.ISO_LOCAL_DATE);
System.out.println("解析输出：\n" + date2 + "\n" + date3);
```

这里需要说明的是，将日期时间格式化输出为字符串和将字符串解析为对应的日期时间对象往往同时出现的。换个角度理解，`DateTimeFormatter` 存在的意义就是将日期时间对象和特定格式的日期时间字符串联系起来，成为两者互转的一个纽带。

![纽带](https://upload-images.jianshu.io/upload_images/1156415-20b4c02cae2020bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



当然，实际开发中自定义格式化的格式也是不可避免的，如下：

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy年MM月dd日");
LocalDate date = LocalDate.of(2018, 4, 1);

String dateStr = date.format(formatter);
System.out.println(dateStr);

LocalDate date2 = LocalDate.parse(dateStr, formatter);
System.out.println(date2);
```

除了自定格式之外，本地化也是一个十分重要的点，如下 ：

```java
LocalDate date = LocalDate.of(2018, 4, 1);
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yy年 MMMM d", Locale.CHINA);

String dateStr = date.format(formatter);
System.out.println(dateStr); // 18年 四月 1
LocalDate date2 = LocalDate.parse(dateStr, formatter);
System.out.println(date2); // 2018-04-01
```
最后，需要说明的是`formatter`还支持builder模式，这样创建自定的格式时将会非常的高效和使用，如下：

```java
LocalDate date = LocalDate.of(2018, 04, 01);
DateTimeFormatter formatter = new DateTimeFormatterBuilder()
        .appendText(ChronoField.YEAR)
        .appendLiteral("年")
        .appendText(ChronoField.MONTH_OF_YEAR, TextStyle.FULL)
        .appendText(ChronoField.DAY_OF_MONTH, TextStyle.FULL_STANDALONE)
        .appendLiteral("日 ")
        .appendText(ChronoField.DAY_OF_WEEK, TextStyle.FULL)
        .parseCaseInsensitive()
        .toFormatter(Locale.CHINESE);

String dateStr = date.format(formatter);
System.out.println(dateStr); // 2018年四月1日 星期日
LocalDate date2 = LocalDate.parse(dateStr, formatter);
System.out.println(date2); // 2018-04-01
```

## 3. 处理不同的时区和历法

> 未完待续。。。

<br>

## 参考
----
- [《java8 实战》](https://book.douban.com/subject/26772632/)

## demo 代码
---
- [github: TimeDemo](https://github.com/xiaopihai7256/TimeDemo)

