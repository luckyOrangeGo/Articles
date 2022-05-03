# Shell Test

```bash
cd ~/

vim test.sh
```



```shell
#!/bin/bash

pwd

# 如果exit上面的命令出错，则返回值不为0
exit

# 也可使用自定义返回值
exit 127
```

test命令用于检查文件或者比较值
test可以做以下测试:
·文件测试
整数比较测试
·字符串测试
test测试语句可以简化为[]符号
[]符号还有扩展写法[]支持&&、‖、<、>

