---
title: linux下json解析神器 - jq
date: 2019-08-02 23:42:12
tags:
---

### jq

```
jq - commandline JSON processor [version 1.6]

Usage:  jq [options] <jq filter> [file...]
    jq [options] --args <jq filter> [strings...]
    jq [options] --jsonargs <jq filter> [JSON_TEXTS...]

jq is a tool for processing JSON inputs, applying the given filter to
its JSON text inputs and producing the filter's results as JSON on
standard output.

The simplest filter is ., which copies jq's input to its output
unmodified (except for formatting, but note that IEEE754 is used
for number representation internally, with all that that implies).

For more advanced filters see the jq(1) manpage ("man jq")
and/or https://stedolan.github.io/jq

Example:

    $ echo '{"foo": 0}' | jq .
    {
        "foo": 0
    }

For a listing of options, use jq --help.

```

### jf

改写成 shell 脚本执行模式
```
#!/bin/bash

argsCount=$#
if [ $argsCount -ne 1 ];then
    echo "Usage : jf json_str"
    exit 2
fi

jsonStr=$1

echo ${jsonStr} | /usr/local/bin/jq .
```

执行方法是：`jf '{"name":"meichaofan"}'`
