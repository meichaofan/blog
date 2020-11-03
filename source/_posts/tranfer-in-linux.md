---
title: linix之间几种文件传输方式
date: 2019-04-22 18:43:23
tags:
---

当我们需要在类UNIX主机间，互传文件，下面几个命令可以排上用场了

---

1. rsync
```
rsync -avz user@src_host:/path/to/file ./ --delete
```

2. scp

3. 
