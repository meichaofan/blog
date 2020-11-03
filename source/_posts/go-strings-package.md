---
title: go strings包介绍
date: 2019-06-30 15:22:04
tags: go
---

**strings**包提供了操作字符串的简单函数。

1. func Compare(a,b string) int
2. func Contains(s, substr string) bool
3. func Count(s, sep string) int
4. func Fields(s string) []string
5. func FieldsFunc(s string, f func(rune) bool) []string
6. func HasPrefix(s, prefix string) bool
7. func HasSuffix(s, suffix string) bool
8. func Index(s, sep string) int
9. func LastIndex(s,sep string) int
10. func Join(a []string,sep string) string
11. func Repeat(s string,count int) string
12. func Replace(s,old,new string,n int) string
13. func Split(s,sep string) []string
14. func SplitN(s,sep string, n int) []string
15. func ToLower(s string) string
16. funcc ToUpper(s string) string
17. func Trim(s string,cutset string) string

---

### 1. func Compare

```
func Compare(a, b string) int
```

根据字典顺序，比较两个字符串的大小。字典顺序，就是说，将多个字符串的同一位置的字符按照26个字母的顺序进行比对。a最小，z最大。

```
The result will be 0 if a==b, -1 if a < b, and +1 if a > b.
```

### 2. func Contains

判断，原字符串是否包含子串

```
fmt.Println(strings.Contains("seafood", "foo"))
fmt.Println(strings.Contains("seafood", "bar"))
fmt.Println(strings.Contains("seafood", ""))
fmt.Println(strings.Contains("", ""))
```

输出

```
true
false
true
true
```

### 3. func Count

```
func Count(s, sep string) int
```

Count计算s中非重叠sep实例的数量。如果sep是空字符串，则Count返回1 +len(s)。

```
fmt.Println（strings.Count（“cheese”，“e”））
fmt.Println（strings.Count（“five”，“”））//在每个符文之前和之后
```

输出

```
3 
5
```

### 4. func Fields

```
func Fields(s string) []string
```

以空格为分隔符，分割字符串s，并返回字符切片

### 5. func FieldsFunc

按照空格，分割字符串，同时对分割后的单个字符串，做相应的逻辑判断，如果匿名函数为真，则不返回该子串。最后返回一个字符串切片。
```
//只返回字母
func testFieldsFunc() {
        str := "- MY NAME is meichaofan JACK 1534"
            r := strings.FieldsFunc(str, func(r rune) bool {
                        return !unicode.IsLetter(r)
                            })
                fmt.Printf("%v", r)
}
//output 
// [MY NAME is meichaofan JACK]
```

### 6. func HasPrefix

判断字符串是否以某个前缀开头

```
func testHasPrefix()  {
        str := "meichaofan"
            r := strings.HasPrefix(str,"mei")
                fmt.Println(r)
}
//output
//true
```

### 7. func HasSuffix

判断字符串是否以某个后缀结束

```
func testHasSuffix()  {
        str := "/home/meichaofan"
            r := strings.HasPrefix(str,"/")
                fmt.Println(r)
}
//output
//false
```

### 8.func Index

```
func Index（s，sep string）int
```

Index 返回s中第一个sep子串的位置，如果不存在sep，返回-1

代码：

```
fmt.Println（strings.Index（“chicken”，“ken”））
fmt.Println（strings.Index（“chicken”，“dmr”））
```

输出：

```
4
-1
```

### 9.func LastIndex

返回子串最后一次出现的位置

### 10.func Join

字符串切片以**sep**合并

```
str := []string{"my", "name", "is", "jack"}
r:= strings.Join(str,"-")
fmt.Println(r)
//output
//my-name-is-jack
```

### 11. func Repeat

重复字符串

```
fmt.Println（“ba”+ strings.Repeat（“na”，2））
//output
//banana
```

### 12. func Replace

```
func Replace(s, old, new string, n int) string
```

字符串替换，将原串中**old**串，替换成**new**串。替换次数为**n**次。当n<0，替换次数没有限制。

```
fmt.Println(strings.Replace("oink oink oink", "k", "ky", 2))
fmt.Println(strings.Replace("oink oink oink", "oink", "moo", -1))
//output 
//oinky oinky oink
//moo moo moo
```

### 13.fun Split

```
func Split(s, sep string) []string
```

将字符串s按照分隔符**sep**分割。

```
fmt.Printf("%q\n", strings.Split("a,b,c", ","))
fmt.Printf("%q\n", strings.Split("a man a plan a canal panama", "a "))
fmt.Printf("%q\n", strings.Split(" xyz ", ""))
fmt.Printf("%q\n", strings.Split("", "Bernardo O'Higgins"))
```

输出

```
["a" "b" "c"]
["" "man " "plan " "canal panama"]
[" " "x" "y" "z" " "]
[""]
```

### 14. func SplitN

```
func SplitN（s，sep string，n int）[] string
```

n表示要返回的子字符串数：

```
n>0:最多n个子串，最后一个子字符串是未分割的余数
n==0:结果是nil
n<0;所有子串 等效于 split
```

代码

```
fmt.Printf（“％q \ n”，strings.SplitN（“a，b，c”，“，”，2））
z：= strings.SplitN（“a，b，c”，“，”，0 ）
fmt.Printf（“％q（nil =％v）\ n”，z，z == nil）
```

输出

```
[“a”“b，c”] 
[]（nil = true）
```

