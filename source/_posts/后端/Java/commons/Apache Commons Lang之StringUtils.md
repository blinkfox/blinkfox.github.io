---
title: Apache Commons Lang之StringUtils
date: 2018-09-27 00:19:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/09/27/commons-stringutils.jpg
categories: 后端
tags:
  - Java
  - Apache
---

> 码农不识Apache，码尽一生也枉然。

## 判断空字符串

### isEmpty(CharSequence cs)

判断是否是空字符串，代码示例：

```java
StringUtils.isEmpty(null)      = true
StringUtils.isEmpty("")        = true
StringUtils.isEmpty(" ")       = false
StringUtils.isEmpty("bob")     = false
StringUtils.isEmpty("  bob  ") = false
```

### isNotEmpty(CharSequence cs)

判断是否不是空字符串，与`isEmpty(CharSequence cs)`相反。

### isAnyEmpty(CharSequence... css)

判断是否含有空字符串，代码示例：

```java
StringUtils.isAnyEmpty(null)             = true
StringUtils.isAnyEmpty(null, "foo")      = true
StringUtils.isAnyEmpty("", "bar")        = true
StringUtils.isAnyEmpty("bob", "")        = true
StringUtils.isAnyEmpty("  bob  ", null)  = true
StringUtils.isAnyEmpty(" ", "bar")       = false
StringUtils.isAnyEmpty("foo", "bar")     = false
```

### isNoneEmpty(CharSequence... css)

判断是否都不是空字符串，与`isAnyEmpty(CharSequence... css)`相反。

### isAllEmpty(CharSequence... css)

判断是否都是空字符串，代码示例：

```java
StringUtils.isAllEmpty(null)             = true
StringUtils.isAllEmpty(null, "")         = true
StringUtils.isAllEmpty(new String[] {})  = true
StringUtils.isAllEmpty(null, "foo")      = false
StringUtils.isAllEmpty("", "bar")        = false
StringUtils.isAllEmpty("bob", "")        = false
StringUtils.isAllEmpty("  bob  ", null)  = false
StringUtils.isAllEmpty(" ", "bar")       = false
StringUtils.isAllEmpty("foo", "bar")     = false
```

### isBlank(CharSequence cs)

判断是否是“大空字符串”，代码示例：

```java
StringUtils.isBlank(null)      = true
StringUtils.isBlank("")        = true
StringUtils.isBlank(" ")       = true
StringUtils.isBlank("bob")     = false
StringUtils.isBlank("  bob  ") = false
```

### isNotBlank(CharSequence cs)

判断是否不是“大空字符串”，与`isBlank(CharSequence cs)`相反，与`isNotEmpty(CharSequence cs)`相似。

### isAnyBlank(CharSequence... css)

判断是否有“大空字符串”，与`isAnyEmpty(CharSequence... css)`相似。

### isNoneBlank(CharSequence... css)

判断是否都不是“大空字符串”，与`isAnyBlank(CharSequence... css)`相反，与`isNoneEmpty(CharSequence... css)`相似。

### isAllBlank(CharSequence... css)

判断是否都是“大空字符串”，与`isAllEmpty(CharSequence... css)`相似。

## 截取字符串

### trim(String str)

去除字符串前后的控制符，代码示例：

```java
StringUtils.trim(null)          = null
StringUtils.trim("")            = ""
StringUtils.trim("     ")       = ""
StringUtils.trim("abc")         = "abc"
StringUtils.trim("    abc    ") = "abc"
```

### trimToNull(String str)

去除字符串前后的控制符，如何是空字符串则转为`null`，代码示例：

```java
StringUtils.trimToNull(null)          = null
StringUtils.trimToNull("")            = null
StringUtils.trimToNull("     ")       = null
StringUtils.trimToNull("abc")         = "abc"
StringUtils.trimToNull("    abc    ") = "abc"
```

### trimToEmpty(String str)

去除字符串前后的控制符，如何是`null`则转为空字符串，代码示例：

```java
StringUtils.trimToEmpty(null)          = ""
StringUtils.trimToEmpty("")            = ""
StringUtils.trimToEmpty("     ")       = ""
StringUtils.trimToEmpty("abc")         = "abc"
StringUtils.trimToEmpty("    abc    ") = "abc"
```

### truncate(String str, int maxWidth)

截断字符串，具有以下特点：

- 如果str字符串的长度小于maxWidth，则直接返回str。
- 不满足第一条时，则为`substring(str, 0, maxWidth)`。
- 如果maxWidth小于0，则抛出IllegalArgumentException。
- 在任何情况下都不会返回长度大于maxWidth的字符串。

代码示例：

```java
StringUtils.truncate(null, 0)       = null
StringUtils.truncate(null, 2)       = null
StringUtils.truncate("", 4)         = ""
StringUtils.truncate("abcdefg", 4)  = "abcd"
StringUtils.truncate("abcdefg", 6)  = "abcdef"
StringUtils.truncate("abcdefg", 7)  = "abcdefg"
StringUtils.truncate("abcdefg", 8)  = "abcdefg"
StringUtils.truncate("abcdefg", -1) = throws an IllegalArgumentException
```

### truncate(String str, int offset, int maxWidth)

截断字符串，具有以下特点：

- 如果str字符串的长度小于maxWidth，则直接返回str。
- 不满足第一条时，则为`substring(str, offset, maxWidth)`。
- 如果maxWidth或者offset小于0，则抛出IllegalArgumentException。
- 在任何情况下都不会返回长度大于maxWidth的字符串。

代码示例：

```java
StringUtils.truncate(null, 0, 0) = null
StringUtils.truncate(null, 2, 4) = null
StringUtils.truncate("", 0, 10) = ""
StringUtils.truncate("", 2, 10) = ""
StringUtils.truncate("abcdefghij", 0, 3) = "abc"
StringUtils.truncate("abcdefghij", 5, 6) = "fghij"
StringUtils.truncate("raspberry peach", 10, 15) = "peach"
StringUtils.truncate("abcdefghijklmno", 0, 10) = "abcdefghij"
StringUtils.truncate("abcdefghijklmno", -1, 10) = throws an IllegalArgumentException
StringUtils.truncate("abcdefghijklmno", Integer.MIN_VALUE, 10) = "abcdefghij"
StringUtils.truncate("abcdefghijklmno", Integer.MIN_VALUE, Integer.MAX_VALUE) = "abcdefghijklmno"
StringUtils.truncate("abcdefghijklmno", 0, Integer.MAX_VALUE) = "abcdefghijklmno"
StringUtils.truncate("abcdefghijklmno", 1, 10) = "bcdefghijk"
StringUtils.truncate("abcdefghijklmno", 2, 10) = "cdefghijkl"
StringUtils.truncate("abcdefghijklmno", 3, 10) = "defghijklm"
StringUtils.truncate("abcdefghijklmno", 4, 10) = "efghijklmn"
StringUtils.truncate("abcdefghijklmno", 5, 10) = "fghijklmno"
StringUtils.truncate("abcdefghijklmno", 5, 5) = "fghij"
StringUtils.truncate("abcdefghijklmno", 5, 3) = "fgh"
StringUtils.truncate("abcdefghijklmno", 10, 3) = "klm"
StringUtils.truncate("abcdefghijklmno", 10, Integer.MAX_VALUE) = "klmno"
StringUtils.truncate("abcdefghijklmno", 13, 1) = "n"
StringUtils.truncate("abcdefghijklmno", 13, Integer.MAX_VALUE) = "no"
StringUtils.truncate("abcdefghijklmno", 14, 1) = "o"
StringUtils.truncate("abcdefghijklmno", 14, Integer.MAX_VALUE) = "o"
StringUtils.truncate("abcdefghijklmno", 15, 1) = ""
StringUtils.truncate("abcdefghijklmno", 15, Integer.MAX_VALUE) = ""
StringUtils.truncate("abcdefghijklmno", Integer.MAX_VALUE, Integer.MAX_VALUE) = ""
StringUtils.truncate("abcdefghij", 3, -1) = throws an IllegalArgumentException
StringUtils.truncate("abcdefghij", -2, 4) = throws an IllegalArgumentException
```

### left(String str, int len)

得到一个字符串最左边的len个字符

```java
StringUtils.left("abc", 0)   = ""
StringUtils.left("abc", 2)   = "ab"
StringUtils.left("abc", 4)   = "abc"
```

### right(String str, int len)

同`left(String str, int len)`相反，从右边截取len个字符。

### mid(String str, int pos, int len)

得到一个字符串中间的len个字符。

```java
StringUtils.mid("abc", 0, 2)   = "ab"
StringUtils.mid("abc", 0, 4)   = "abc"
StringUtils.mid("abc", 2, 4)   = "c"
StringUtils.mid("abc", 4, 2)   = ""
StringUtils.mid("abc", -2, 2)  = "ab"
```

### substringBefore(String str, String separator)

得到一个字符串第一个分隔符字符串之前的字符串。

```java
StringUtils.substringBefore("abc", "a")   = ""
StringUtils.substringBefore("abcba", "b") = "a"
StringUtils.substringBefore("abc", "c")   = "ab"
StringUtils.substringBefore("abc", "d")   = "abc"
StringUtils.substringBefore("abc", "")    = ""
StringUtils.substringBefore("abc", null)  = "abc"
```

### substringAfter(String str, String separator)

同`substringBefore(String str, String separator)`相反。得到一个字符串第一个分隔符字符串之后的字符串。

### substringBetween(String str, String open, String close)

得到一个字符串两个字符串之间字符串。

```java
StringUtils.substringBetween("", "", "")          = ""
StringUtils.substringBetween("", "", "]")         = null
StringUtils.substringBetween("", "[", "]")        = null
StringUtils.substringBetween("yabcz", "", "")     = ""
StringUtils.substringBetween("yabcz", "y", "z")   = "abc"
StringUtils.substringBetween("yabczyabcz", "y", "z")   = "abc"
```

### substringBetween(String str, String tag)

是`substringBetween(String str, String open, String close)`的特殊情形。得到一个字符串中同一个字符串之间的字符串。

## 比较字符串

### equals(CharSequence cs1, CharSequence cs2)

判断两字符串相等，代码示例：

```java
StringUtils.equals(null, null)   = true
StringUtils.equals(null, "abc")  = false
StringUtils.equals("abc", null)  = false
StringUtils.equals("abc", "abc") = true
StringUtils.equals("abc", "ABC") = false
```

### equalsIgnoreCase(CharSequence str1, CharSequence str2)

判断两字符串相等，忽略大小写，代码示例：

```java
StringUtils.equalsIgnoreCase(null, null)   = true
StringUtils.equalsIgnoreCase(null, "abc")  = false
StringUtils.equalsIgnoreCase("abc", null)  = false
StringUtils.equalsIgnoreCase("abc", "abc") = true
StringUtils.equalsIgnoreCase("abc", "ABC") = true
```

### equalsAny(CharSequence string, CharSequence... searchStrings)

比较一个字符串是否与其后的某个字符串相等，代码示例：

```java
StringUtils.equalsAny(null, (CharSequence[]) null) = false
StringUtils.equalsAny(null, null, null)    = true
StringUtils.equalsAny(null, "abc", "def")  = false
StringUtils.equalsAny("abc", null, "def")  = false
StringUtils.equalsAny("abc", "abc", "def") = true
StringUtils.equalsAny("abc", "ABC", "DEF") = false
```

### equalsAnyIgnoreCase(CharSequence string, CharSequence...searchStrings)

比较一个字符串是否与其后的某个字符串相等，忽略大小写，代码示例：

```java
StringUtils.equalsAnyIgnoreCase(null, (CharSequence[]) null) = false
StringUtils.equalsAnyIgnoreCase(null, null, null)    = true
StringUtils.equalsAnyIgnoreCase(null, "abc", "def")  = false
StringUtils.equalsAnyIgnoreCase("abc", null, "def")  = false
StringUtils.equalsAnyIgnoreCase("abc", "abc", "def") = true
StringUtils.equalsAnyIgnoreCase("abc", "ABC", "DEF") = true
```

### compare(String str1, String str2)

比较两字符串的大小，代码示例：

```java
StringUtils.compare(null, null)   = 0
StringUtils.compare(null , "a")   < 0
StringUtils.compare("a", null)    > 0
StringUtils.compare("abc", "abc") = 0
StringUtils.compare("a", "b")     < 0
StringUtils.compare("b", "a")     > 0
StringUtils.compare("a", "B")     > 0
StringUtils.compare("ab", "abc")  < 0
```

### compareIgnoreCase(String str1, String str2)

比较两字符串的大小，忽略大小写，代码示例：

```java
StringUtils.compareIgnoreCase(null, null)   = 0
StringUtils.compareIgnoreCase(null , "a")   < 0
StringUtils.compareIgnoreCase("a", null)    > 0
StringUtils.compareIgnoreCase("abc", "abc") = 0
StringUtils.compareIgnoreCase("abc", "ABC") = 0
StringUtils.compareIgnoreCase("a", "b")     < 0
StringUtils.compareIgnoreCase("b", "a")     > 0
StringUtils.compareIgnoreCase("a", "B")     < 0
StringUtils.compareIgnoreCase("A", "b")     < 0
StringUtils.compareIgnoreCase("ab", "ABC")  < 0
```

## 查找元素

### indexOf(CharSequence seq, int searchChar)

查找某个字符在字符串中第一次出现时的索引位置，代码示例：

```java
StringUtils.indexOf("aabaabaa", 'a') = 0
StringUtils.indexOf("aabaabaa", 'b') = 2
```

### indexOf(CharSequence seq, CharSequence searchSeq)

同`indexOf(CharSequence seq, int searchChar)`相似。

```java
StringUtils.indexOf("aabaabaa", "c")  = -1
StringUtils.indexOf("aabaabaa", "a")  = 0
StringUtils.indexOf("aabaabaa", "b")  = 2
StringUtils.indexOf("aabaabaa", "ab") = 1
StringUtils.indexOf("aabaabaa", "")   = 0
```

### indexOf(final CharSequence seq, final CharSequence searchSeq, final int startPos)

同`indexOf(CharSequence seq, int searchChar)`相似。

```java
StringUtils.indexOf("aabaabaa", "a", 0)  = 0
StringUtils.indexOf("aabaabaa", "b", 0)  = 2
StringUtils.indexOf("aabaabaa", "ab", 0) = 1
StringUtils.indexOf("aabaabaa", "b", 3)  = 5
StringUtils.indexOf("aabaabaa", "b", 9)  = -1
StringUtils.indexOf("aabaabaa", "b", -1) = 2
StringUtils.indexOf("aabaabaa", "", 2)   = 2
StringUtils.indexOf("abc", "", 9)        = 3
```

### indexOfIgnoreCase(CharSequence str, CharSequence searchStr)

同`indexOf(CharSequence seq, int searchChar)`相似,忽略大小写。

### lastIndexOf(CharSequence seq, int searchChar)

同`indexOf(CharSequence seq, int searchChar)`相似，从后面开始查找。

### contains(CharSequence seq, CharSequence searchSeq)

判断某字符串是否包含某子字符串。

```java
StringUtils.contains("", "")      = true
StringUtils.contains("abc", "")   = true
StringUtils.contains("abc", "a")  = true
StringUtils.contains("abc", "z")  = false
```

### containsIgnoreCase(CharSequence str, CharSequence searchStr)

同`contains(CharSequence seq, CharSequence searchSeq)`相似，忽略大小写。

### containsWhitespace(final CharSequence seq)

是`contains(CharSequence seq, CharSequence searchSeq)`的特殊情形，判断是否包含空白字符。

### containsAny(CharSequence cs, CharSequence... searchCharSequences)

判断某字符串是否包含其后的任意一个字符串。

```java
StringUtils.containsAny("abcd", "ab", null) = true
StringUtils.containsAny("abcd", "ab", "cd") = true
StringUtils.containsAny("abc", "d", "abc")  = true
```

### containsNone(CharSequence cs, String invalidChars)

判断某字符串是否不含其后字符串的任意一个字符。

```java
StringUtils.containsNone("ab", "")      = true
StringUtils.containsNone("abab", "xyz") = true
StringUtils.containsNone("ab1", "xyz")  = true
StringUtils.containsNone("abz", "xyz")  = false
```

## 分割字符串

### split(String str, String separatorChars)

将某字符串按字符分割成数组，默认按空格分组。

```java
StringUtils.split("abc def", null) = ["abc", "def"]
StringUtils.split("abc def", " ")  = ["abc", "def"]
StringUtils.split("abc  def", " ") = ["abc", "def"]
StringUtils.split("ab:cd:ef", ":") = ["ab", "cd", "ef"]
```

### split(String str, String separatorChars, int max)

将某字符串按字符分割成最大max长度的数组，默认按空格分组。

```java
StringUtils.split("ab cd ef", null, 0)   = ["ab", "cd", "ef"]
StringUtils.split("ab   cd ef", null, 0) = ["ab", "cd", "ef"]
StringUtils.split("ab:cd:ef", ":", 0)    = ["ab", "cd", "ef"]
StringUtils.split("ab:cd:ef", ":", 2)    = ["ab", "cd:ef"]
```

### splitByCharacterType(final String str)

按字符串类型划分字符串为数组。

```java
StringUtils.splitByCharacterType(null)         = null
StringUtils.splitByCharacterType("")           = []
StringUtils.splitByCharacterType("ab de fg")   = ["ab", " ", "de", " ", "fg"]
StringUtils.splitByCharacterType("ab   de fg") = ["ab", "   ", "de", " ", "fg"]
StringUtils.splitByCharacterType("ab:cd:ef")   = ["ab", ":", "cd", ":", "ef"]
StringUtils.splitByCharacterType("number5")    = ["number", "5"]
StringUtils.splitByCharacterType("fooBar")     = ["foo", "B", "ar"]
StringUtils.splitByCharacterType("foo200Bar")  = ["foo", "200", "B", "ar"]
StringUtils.splitByCharacterType("ASFRules")   = ["ASFR", "ules"]
```

## 连接字符串

### join(T... elements)

无连接符连接字符串。

```java
StringUtils.join(null)            = null
StringUtils.join([])              = ""
StringUtils.join([null])          = ""
StringUtils.join(["a", "b", "c"]) = "abc"
StringUtils.join([null, "", "a"]) = "a"
```

### join(Object[] array, String separator)

将提供的数组按连接符连成字符串。

```java
StringUtils.join(null, *)               = null
StringUtils.join([], *)                 = ""
StringUtils.join([null], *)             = ""
StringUtils.join(["a", "b", "c"], ';')  = "a;b;c"
StringUtils.join(["a", "b", "c"], null) = "abc"
StringUtils.join([null, "", "a"], ';')  = ";;a"
```

## 删除字符串

### deleteWhitespace(String str)

删除空白字符。

```java
StringUtils.deleteWhitespace(null)         = null
StringUtils.deleteWhitespace("")           = ""
StringUtils.deleteWhitespace("abc")        = "abc"
StringUtils.deleteWhitespace("   ab  c  ") = "abc"
```

### removeStart(String str, String remove)

删除指定字符串前缀的字符串。

```java
StringUtils.removeStart("www.domain.com", "www.")   = "domain.com"
StringUtils.removeStart("domain.com", "www.")       = "domain.com"
StringUtils.removeStart("www.domain.com", "domain") = "www.domain.com"
StringUtils.removeStart("abc", "")    = "abc"
```

### removeStartIgnoreCase(String str, String remove)

同`removeStart(String str, String remove)`相似，忽略大小写。

### removeEnd(String str, String remove)

同`removeStart(String str, String remove)`相反。

### removeEndIgnoreCase(String str, String remove)

同`removeEnd(String str, String remove)`相似，忽略大小写。

### remove(String str, String remove)

移除字符串中指定的字符串。

```java
StringUtils.remove("queued", "ue") = "qd"
StringUtils.remove("queued", "zz") = "queued"
```

### removeIgnoreCase(String str, String remove)

同`remove(String str, String remove)`相似，忽略大小写。

## 替换字符串

### replace(String text, String searchString, String replacement)

替换某字符串为另一个字符串。

```java
StringUtils.replace("aba", "a", null)  = "aba"
StringUtils.replace("aba", "a", "")    = "b"
StringUtils.replace("aba", "a", "z")   = "zbz"
```

### replaceIgnoreCase(String text, String searchString, String replacement)

同`replace(String text, String searchString, String replacement)`相似，忽略大小写。

### replace(String text, String searchString, String replacement, int max)

替换某字符串为另一个字符串,从左到右替换最大max次。

```java
StringUtils.replace("abaa", "a", null, -1) = "abaa"
StringUtils.replace("abaa", "a", "", -1)   = "b"
StringUtils.replace("abaa", "a", "z", 0)   = "abaa"
StringUtils.replace("abaa", "a", "z", 1)   = "zbaa"
StringUtils.replace("abaa", "a", "z", 2)   = "zbza"
StringUtils.replace("abaa", "a", "z", -1)  = "zbzz"
```

### replaceEach(String text, String[] searchList, String[] replacementList)

替换某些字符串为另一些字符串。

```java
StringUtils.replaceEach("aba", null, null) = "aba"
StringUtils.replaceEach("aba", new String[0], null) = "aba"
StringUtils.replaceEach("aba", null, new String[0]) = "aba"
StringUtils.replaceEach("aba", new String[]{"a"}, null)  = "aba"
StringUtils.replaceEach("aba", new String[]{"a"}, new String[]{""})  = "b"
StringUtils.replaceEach("aba", new String[]{null}, new String[]{"a"})  = "aba"
StringUtils.replaceEach("abcde", new String[]{"ab", "d"}, new String[]{"w", "t"})  = "wcte"
StringUtils.replaceEach("abcde", new String[]{"ab", "d"}, new String[]{"d", "t"})  = "dcte"
```

## 填充字符串

### repeat(final String str, final int repeat)

生成重复的字符串，repeat代表生成次数。

```java
StringUtils.repeat(null, 2) = null
StringUtils.repeat("", 0)   = ""
StringUtils.repeat("", 2)   = ""
StringUtils.repeat("a", 3)  = "aaa"
StringUtils.repeat("ab", 2) = "abab"
StringUtils.repeat("a", -2) = ""
```

### repeat(String str, String separator, int repeat)

生成重复的字符串，repeat代表生成次数。

```java
StringUtils.repeat(null, null, 2) = null
StringUtils.repeat(null, "x", 2)  = null
StringUtils.repeat("", null, 0)   = ""
StringUtils.repeat("", "", 2)     = ""
StringUtils.repeat("", "x", 3)    = "xxx"
StringUtils.repeat("?", ", ", 3)  = "?, ?, ?"
```

## 字符串计数

### countMatches(CharSequence str, CharSequence sub)

计算某字符串在字符串中的出现次数。

```java
StringUtils.countMatches("abba", null)  = 0
StringUtils.countMatches("abba", "")    = 0
StringUtils.countMatches("abba", "a")   = 2
StringUtils.countMatches("abba", "ab")  = 1
StringUtils.countMatches("abba", "xxx") = 0
```

## 字符测试

### isAlpha(CharSequence cs)

判断字符串是否是Unicode字母。

```java
StringUtils.isAlpha(null)   = false
StringUtils.isAlpha("")     = false
StringUtils.isAlpha("  ")   = false
StringUtils.isAlpha("abc")  = true
StringUtils.isAlpha("ab2c") = false
StringUtils.isAlpha("ab-c") = false
```

### isAlphaSpace(CharSequence cs)

同`isAlpha(CharSequence cs)`相似。判断字符串是否是Unicode字母或空格。

```java
StringUtils.isAlphaSpace(null)   = false
StringUtils.isAlphaSpace("")     = true
StringUtils.isAlphaSpace("  ")   = true
StringUtils.isAlphaSpace("abc")  = true
StringUtils.isAlphaSpace("ab c") = true
StringUtils.isAlphaSpace("ab2c") = false
StringUtils.isAlphaSpace("ab-c") = false
```

### isAlphanumeric(CharSequence cs)

同`isAlpha(CharSequence cs)`相似。判断字符串是否是Unicode字母或数字。

### isAlphanumericSpace(CharSequence cs)

同`isAlpha(CharSequence cs)`相似。判断字符串是否是Unicode字母、空格或数字。

### isNumeric(CharSequence cs)

判断字符串是否是数字。

```java
StringUtils.isNumeric("123")  = true
StringUtils.isNumeric("12 3") = false
StringUtils.isNumeric("ab2c") = false
StringUtils.isNumeric("12-3") = false
StringUtils.isNumeric("12.3") = false
StringUtils.isNumeric("-123") = false
StringUtils.isNumeric("+123") = false
```

### isNumericSpace(CharSequence cs)

同`isNumeric(CharSequence cs)`相似。判断字符串是否是空格或数字。

### getDigits(String str)

从字符串中提取出数字为字符串。

```java
StringUtils.getDigits(null)  = null
StringUtils.getDigits("")    = ""
StringUtils.getDigits("abc") = ""
StringUtils.getDigits("1000$") = "1000"
StringUtils.getDigits("1123~45") = "12345"
StringUtils.getDigits("(541) 754-3010") = "5417543010"
```

### isWhitespace(CharSequence cs)

判断是否是空格。

```java
StringUtils.isWhitespace(null)   = false
StringUtils.isWhitespace("")     = true
StringUtils.isWhitespace("  ")   = true
StringUtils.isWhitespace("abc")  = false
StringUtils.isWhitespace("ab2c") = false
StringUtils.isWhitespace("ab-c") = false
```

### isAllLowerCase(CharSequence cs)

判断字符串是否都是小写。

```java
StringUtils.isAllLowerCase(null)   = false
StringUtils.isAllLowerCase("")     = false
StringUtils.isAllLowerCase("  ")   = false
StringUtils.isAllLowerCase("abc")  = true
StringUtils.isAllLowerCase("abC")  = false
StringUtils.isAllLowerCase("ab c") = false
StringUtils.isAllLowerCase("ab1c") = false
StringUtils.isAllLowerCase("ab/c") = false
```

### isAllUpperCase(CharSequence cs)

同`isAllLowerCase`相反。判断字符串是否都是大写。

### isMixedCase(CharSequence cs)

同`isAllLowerCase`相似。判断字符串是否大小写都有。

## 默认字符串

### defaultString(String str)

得到默认字符串，默认空字符串。

```java
StringUtils.defaultString(null)  = ""
StringUtils.defaultString("")    = ""
StringUtils.defaultString("bat") = "bat"
```

### defaultString(String str, String defaultStr)

如果是null，则得到默认字符串。

```java
StringUtils.defaultString(null, "NULL")  = "NULL"
StringUtils.defaultString("", "NULL")    = ""
StringUtils.defaultString("bat", "NULL") = "bat"
```

### defaultIfEmpty(T str, T defaultStr)

同`defaultString(String str, String defaultStr)`相似。如果是空字符串，则得到默认字符串。

## 反转字符串

### reverse(final String str)

反转字符串。

```java
StringUtils.reverse(null)  = null
StringUtils.reverse("")    = ""
StringUtils.reverse("bat") = "tab"
```

## 缩写字符串

### abbreviate(String str, int maxWidth)

缩写字符串为最大maxWidth长度的字符串，使用`...`作为缩写的后缀，maxWidth不能小于等于3。

```java
StringUtils.abbreviate("", 4)        = ""
StringUtils.abbreviate("abcdefg", 6) = "abc..."
StringUtils.abbreviate("abcdefg", 7) = "abcdefg"
StringUtils.abbreviate("abcdefg", 8) = "abcdefg"
StringUtils.abbreviate("abcdefg", 4) = "a..."
StringUtils.abbreviate("abcdefg", 3) = IllegalArgumentException
```

### abbreviate(String str, String abbrevMarker, int maxWidth)

缩写字符串为最大maxWidth长度的字符串，使用`abbrevMarker`作为缩写的后缀，maxWidth不能小于等于`abbrevMarker`的长度。

```java
StringUtils.abbreviate("", "...", 4)        = ""
StringUtils.abbreviate("abcdefg", ".", 5)   = "abcd."
StringUtils.abbreviate("abcdefg", ".", 7)   = "abcdefg"
StringUtils.abbreviate("abcdefg", ".", 8)   = "abcdefg"
StringUtils.abbreviate("abcdefg", "..", 4)  = "ab.."
StringUtils.abbreviate("abcdefg", "..", 3)  = "a.."
StringUtils.abbreviate("abcdefg", "..", 2)  = IllegalArgumentException
StringUtils.abbreviate("abcdefg", "...", 3) = IllegalArgumentException
```

## 字符串钱后缀

### startsWith(CharSequence str, CharSequence prefix)

判断某字符串是否包含有指定前缀的字符串。

```java
StringUtils.startsWith(null, null)      = true
StringUtils.startsWith(null, "abc")     = false
StringUtils.startsWith("abcdef", null)  = false
StringUtils.startsWith("abcdef", "abc") = true
StringUtils.startsWith("ABCDEF", "abc") = false
```

### startsWithIgnoreCase(CharSequence str, CharSequence prefix)

同`startsWith(CharSequence str, CharSequence prefix)`相似。忽略大小写。

### startsWithAny(CharSequence sequence, CharSequence... searchStrings)

判断某字符串是否包含有其后任意一个指定前缀的字符串。

```java
StringUtils.startsWithAny(null, null)      = false
StringUtils.startsWithAny(null, new String[] {"abc"})  = false
StringUtils.startsWithAny("abcxyz", null)     = false
StringUtils.startsWithAny("abcxyz", new String[] {""}) = true
StringUtils.startsWithAny("abcxyz", new String[] {"abc"}) = true
StringUtils.startsWithAny("abcxyz", new String[] {null, "xyz", "abc"}) = true
StringUtils.startsWithAny("abcxyz", null, "xyz", "ABCX") = false
StringUtils.startsWithAny("ABCXYZ", null, "xyz", "abc") = false
```

### endsWith(CharSequence str, CharSequence suffix)

同`startsWith(CharSequence str, CharSequence prefix)`相反。

### endsWithIgnoreCase(CharSequence str, CharSequence suffix)

同`startsWithIgnoreCase(CharSequence str, CharSequence prefix)`相反。

### endsWithAny(CharSequence sequence, CharSequence... searchStrings)

同`startsWithAny(CharSequence sequence, CharSequence... searchStrings)`相反。