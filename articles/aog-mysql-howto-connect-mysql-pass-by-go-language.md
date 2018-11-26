---
title: "Go 语言连接 MySQL PaaS"
description: "Go 语言连接 MySQL PaaS"
author: xingbing0909
resourceTags: 'Mysql, Go Language'
ms.service: Mysql
wacn.topic: aog
ms.topic: article
ms.author: v-tawe
ms.date: 11/20/2018
wacn.date: 11/20/2018
---

# Go 语言连接 MySQL PaaS

Go 语言工具下载链接点击[这里](https://golang.org/dl/)。

根据自己的需要选择合适的操作系统下载，例如我下载的是 go1.8.3.windows-amd64.msi。

![01](media/aog-mysql-howto-connect-mysql-pass-by-go-language/01.png "01")

下载后安装即可。

配置环境变量：

![02](media/aog-mysql-howto-connect-mysql-pass-by-go-language/02.png "02")

![03](media/aog-mysql-howto-connect-mysql-pass-by-go-language/03.png "03")

![04](media/aog-mysql-howto-connect-mysql-pass-by-go-language/04.png "04")

下面是 go 语言连接 MySQL PaaS 的脚本和方法：

```go
package main

import (
"fmt"
"database/sql"
_"github.com/go-sql-driver/mysql"
)

func main() {
   fmt.Println("Hello, World!")
   db,err := sql.Open("mysql","poddb%yongji:1qaz!QAZQAZ! @tcp(poddb.mysqldb.chinacloudapi.cn:3306)/zhdb")
   db.Exec("insert into user values(70,'aaaa','bbbb')")

   if err != nil{
          fmt.Println("0000000")
    }else{
           fmt.Println("success")
    }
}
```

运行 go 文件：

![05](media/aog-mysql-howto-connect-mysql-pass-by-go-language/05.jpg "05")

通过 workbench 连接到 MySQL PaaS，验证数据已经插入成功：

![06](media/aog-mysql-howto-connect-mysql-pass-by-go-language/06.jpg "06")