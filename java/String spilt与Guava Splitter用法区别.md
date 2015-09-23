String split方法与Guava Splitter用法区别
========================================

今天同事写了一段使用String split方法的代码，如下所示，同事期望得到的是字符"1"，但是没想到却得到空字符。
```java
String targetStr = "1";
String[] splitStrs = targetStr.split("//|");
for (String spiltStr : splitStrs) {
    System.out.println(spiltStr);
}
```
同事修改成如下代码，可得到的结果也不正确
```java
String[] splitStrs = targetStr.split("|");
```
对这个问题我也不算特别清楚，因此只好先建议同事修改为如下代码，修改后同事获得了正确的结果
```java
final Iterable<String> stringIterable = Splitter.on("|").split(targetStr);
```
其实问题出在split方法的入参上，查阅String split方法可知道入参其实是一个**正则表达式**
```java
/* @param  regex
*         the delimiting regular expression
*
* @return  the array of strings computed by splitting this string
*          around matches of the given regular expression
*
* @throws  PatternSyntaxException
*          if the regular expression's syntax is invalid
*
* @see java.util.regex.Pattern
*
* @since 1.4
* @spec JSR-51
*/
public String[] split(String regex) {
  return split(regex, 0);
}
```
既然入参是正则表达式，那就应该用正则表达式去进行解答，在正则表达式中"|"代表的含义是或，因此之前的"//|"应当理解成为匹配字符中存在"//"或空字符，并按匹配进行分割。当按照空字符进行分割是，则得到的就是空字符与"1"了，并且将"//|"修改为"|"也无法纠正错误，仍然是按空字符进行分割。这里同事犯了两个错误：

- 错用了转义字符，应当使用"\"而不是"/"
- 误解了split方法的入参，没有意识到这是一个正则表达式

所以想要获得正确的结果，可以使用以下几种写法
```java
String[] splitStrs1 = targetStr.split("\\|");
String[] splitStrs2 = targetStr.split("[|]");
String[] splitStrs3 = targetStr.split(Pattern.quote("|"));
```
那么为何Guava Splitter能够获得正确的结果呢？还是要从参数入手，从on方法定义可以看出，作为分隔符的入参可以是多种类型的，如下。
```java
public static Splitter on(char separator);
public static Splitter on(final CharMatcher separatorMatcher);
public static Splitter on(final String separator);
@GwtIncompatible("java.util.regex")
public static Splitter on(final Pattern separatorPattern);
```
on方法的入参可以是单字符、字符串以及正则表达式，最大程度地保证了使用者不会因为没有检查入参含义而导致误用的问题，从易用性和友好程度角度出发，Guava的大多数方法确实更有优势，用Splitter实现上述代码，如下。
```java
final Iterable<String> splits1 = Splitter.on('|').split(targetStr);
final Iterable<String> splits2 = Splitter.on("|").split(targetStr);
final Iterable<String> splits3 = Splitter.on(new CharMatcher() {
    @Override
    public boolean matches(char c) {
        return c == '|';
    }
}).split(targetStr);
final Iterable<String> splits4 = Splitter.on(Pattern.compile("\\|")).split(targetStr);
```
