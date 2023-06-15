# 日期与时间
{docsify-updated}

### Instant-机器时间
Java 用 `Instant` 表示时间线上的某个点，**可以用来表示时间戳或者称为机器时间**。时间线的原点被设置为穿过伦敦格林威治天文台的本初子午线所处时区的1970年1月1日的午夜。 `Instant.MIN` 的值可向回追溯10亿年，`Instant.MAX` 是公1000000000年的12月31日。
`Instant.now()`：返回当前的时刻。

为了得到两个时刻之间的时间差，可以使用 `Duration Duration.between(Instant start,Instant end)` 方法。`Duration` 代表两个时刻之间的时间差。可以通过以下一些方法来获得 `Duration` 表示的按照常规单位度量的时间长度。
`toNonos`,`toMills`,`toSeconds`,`toMinutes`,`toHours`,`toDays`。

### 本地时间
机器时间对机器来说是很好识别的，但是对人类来说却不是很方便。Java中有两种人类时间：本地日期/时间和时区时间。

本地日期/时间包含日期和当天的时间，但是不包含任何时区信息。这种时间不能对应精确的机器时间。因为，同一个日期在不同时区对应的机器时间是不同的。时区时间能表示一个精确的机器时间。通俗的解释就是，本地时间可以用来代表一个时间字符串，如 "2023-06-15 00:00:00" ，但是这个字符串所代表的精确时间对不同时区的人来说是不一样的。

`LocalDate`代表带有年月日的日期。为了构建 `LocalDate` 对象可以是该类的静态方法 `now` 或 `of`：
```
LocalDate localDate = LocalDate.now();
LocalDate localDate2 = LocalDate.of(1990,1,19);
localDate = LocalDate.of(1990, Month.JANUARY,19);
Period period = localDate.until(localDate2);
long days = localDate.until(localDate2, ChronoUnit.DAYS);
```
类似与两个 `Instant` 之间的时长用 `Duration` 表示，两个本地时间之间的时长用 `Period` 表示，代表了流逝的年、月或日的数量。

与`LocalDate`类似的还有`LocalTime`和 `LocalDateTime`，表示的本地时间。
```
LocalTime time1 = LocalTime.now();
LocalTime time2 = LocalTime.of(8,20,0);
LocalDateTime localDateTime = LocalDateTime.now();
localDateTime = LocalDateTime.of(localDate,time1);
```

#### 日期调整器
对于日程安排应用来说，经常需要计算诸如"每个月的第一个星期二"这种计算。 `TemporalAdjusters` 提供了大量的常见调整的静态方法。
<center><img src="pics/TemporalAdjusters.jpg" width="70%"></center>

也可以通过实现 `TemporalAdjuster` 接口自定义自己的调整器。

### 时区时间
Java 使用 `ZonedDateTime` 来代表时区时间。`ZoneId` 代表不同的时区。

### 日期/时间与字符串的互转

+ 日期时间转字符串：

	`DateTimeFormatter` 提供了三种不同的日期/时间格式器：
	1. 预定义的标准格式器
	<center><img src="pics/predefined-formatter.jpg" width="70%"></center>

	这些预定义的格式器作为 `DateTimeFormatter` 的静态成员，可以直接通过调用`format()`方法来格式化日期。
	```
		String str = DateTimeFormatter.ISO_LOCAL_DATE_TIME.format(localDateTime);
	```

	2. Locale 相关的格式器
	标准格式器主要是为了机器刻度的时间而设置的，表示人类时间可以用Locale相关的格式化器。
		<center><img src="pics/locale-dateformatter.jpg" width="70%"></center>

		静态方法`ofLocalizedDate`,`ofLocalizedTime`,`ofLocalizedDateTime`可以创建这种格式器。
		```
		DateTimeFormatter formatter = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG);
		String str = formatter.withLocale(Locale.CHINA).format(localDateTime);
		```
	
	3. 定制模式的格式器
	```
		DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
		formatter.format(localDatetime);
	```
		<center><img src="pics/formater.jpg" width="70%"></center>

+ 字符串转时间
  ```
	LocalDateTime.parse("1990-01-01 1:00:00");
	LocalDate.parse("1990-01-01");
	ZonedDateTime zonedDateTime = ZonedDateTime.parse("1990-01-01 1:00:00",DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
  ```

   