selpg
============
####golang for selpg(CLI)

[ 开发 Linux 命令行实用程序](https://www.ibm.com/developerworks/cn/linux/shell/clutil/index.html)
-----
_ _ _


1. usage
-----

```
1.  $ selpg -s=1 -e=1 input_file
	该命令将把“input_file”的第 1 页写至标准输出（也就是屏幕），因为这里没有重定向或管道。
2.	$ selpg -s=1 -e=1 < input_file
	该命令与示例 1 所做的工作相同，但在本例中，selpg 读取标准输入，而标准输入已被 shell／内核重定向为来自“input_file”而不是显式命名的文件名参数。输入的第 1 页被写至屏幕。
3.	$ other_command | selpg -s=10 -e=20
	“other_command”的标准输出被 shell／内核重定向至 selpg 的标准输入。将第 10 页到第 20 页写至 selpg 的标准输出（屏幕）。
4.	$ selpg -s=10 -e=20 input_file >output_file
	selpg 将第 10 页到第 20 页写至标准输出；标准输出被 shell／内核重定向至“output_file”。
5.	$ selpg -s=10 -e=20 input_file 2>error_file
	selpg 将第 10 页到第 20 页写至标准输出（屏幕）；所有的错误消息被 shell／内核重定向至“error_file”。请注意：在“2”和“>”之间不能有空格；这是 shell 语法的一部分（请参阅“man bash”或“man sh”）。
6.	$ selpg -s=10 -e=20 input_file >output_file 2>error_file
	selpg 将第 10 页到第 20 页写至标准输出，标准输出被重定向至“output_file”；selpg 写至标准错误的所有内容都被重定向至“error_file”。当“input_file”很大时可使用这种调用；您不会想坐在那里等着 selpg 完成工作，并且您希望对输出和错误都进行保存。
7.	$ selpg -s=10 -e=20 input_file >output_file 2>/dev/null
	selpg 将第 10 页到第 20 页写至标准输出，标准输出被重定向至“output_file”；selpg 写至标准错误的所有内容都被重定向至 /dev/null（空设备），这意味着错误消息被丢弃了。设备文件 /dev/null 废弃所有写至它的输出，当从该设备文件读取时，会立即返回 EOF。
8.	$ selpg -s=10 -e=20 input_file >/dev/null
	selpg 将第 10 页到第 20 页写至标准输出，标准输出被丢弃；错误消息在屏幕出现。这可作为测试 selpg 的用途，此时您也许只想（对一些测试情况）检查错误消息，而不想看到正常输出。
9.	$ selpg -s=10 -e=20 input_file | other_command
	selpg 的标准输出透明地被 shell／内核重定向，成为“other_command”的标准输入，第 10 页到第 20	  页被写至该标准输入。“other_command”的示例可以是 lp，它使输出在系统缺省打印机上打			 印。“other_command”的示例也可以 wc，它会显示选定范围的页中包含的行数、字数和字数。“other_command”可以是任何其它能从其标准输入读取的命令。错误消息仍在屏幕显示。
10.	$ selpg -s=10 -e=20 input_file 2>error_file | other_command
	与上面的示例 9 相似，只有一点不同：错误消息被写至“error_file”。
    $ selpg -s=10 -e=20 -l=10
    定义每一页为10行输出
    $ selpg -s=10 -e=20 -f
    定义文档为f格式，遇到'\f'为一页的终止符
```

2. test
------
> ./selpg -s=1 -e=1 -l=10 text.txt

![](https://github.com/moandy/go-lang/blob/master/selpg/imgtest/test1.png?raw=true)

> ./selpg -s=1 -e=1 -l=5 \~/Desktop/hello.cpp >\~/Desktop/text.txt

![](https://github.com/moandy/go-lang/blob/master/selpg/imgtest/test2.png?raw=true)

> ./selpg2 -s=1 -e=1 -f text.txt

![](https://github.com/moandy/go-lang/blob/master/selpg/imgtest/test3.png?raw=true)

> ./selpg2 -s=1 -e=1

![](https://github.com/moandy/go-lang/blob/master/selpg/imgtest/test4.png?raw=true)


3. design process
------

导入所需要的包
``` go
import (
	"bufio"
	"fmt"
	"os"
	"os/exec"
	"strconv"
	"strings"
)
```

定义结构提体

``` go
type selpgArgs struct {
	startPage  int    // start page of the article
	endPage    int    // end page of the article
	inFilename string // name of the file to be read
	pageLen    int    /* number of lines in one page, default value is 72,
	   can be overriden by "-lNumber" on command line */
	pageType rune /* type of the article, 'l' for lines-delimited, 'f' for form-feed-delimited
	   default is 'l'. */
	printDest string // destination of result pages
}
```

函数接口

``` go
func usage();
func processArgs(argNums int, args []string, saAddr *selpgArgs);
func processInput(saAddr *selpgArgs);
```