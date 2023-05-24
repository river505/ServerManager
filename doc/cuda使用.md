# 普通用户使用cuda

服务器已有cuda：

cuda-10.2     不建议使用 GCC兼容性不好

cuda-11.3     建议使用

cuda-12.0     建议使用



由于普通用户无sudo权限，因此无法自行安装cuda

如果必须使用其他版本的cuda，请联系管理员安装，并说明为什么使用，管理员负责安装

普通用户可以使用，使用方法为：

普通用户只需修改自己的环境变量，将已安装的cuda路径映射到自己的环境变量中即可，具体如下：



1. 进入目录下，查看有什么版本的cuda

```Bash
(base) wt@wt-Ubuntu:~$ cd /usr/local
(base) wt@wt-Ubuntu:/usr/local$ ls
bin  cuda  cuda-11.3  etc  games  include  lib  man  sbin  share  src
```

1. 修改个人用户环境变量

```Bash
vim ~/.bashrc
```

在文件尾部添加相应路径：(其中cuda-11.3请修改为自己对应的版本，例如cuda-10.2 cuda-11.7 等)

```Bash
export PATH="/usr/local/cuda-11.3/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda-11.3/lib64/:$LD_LIBRARY_PATH"
```

:wq 保存退出

1. 更新环境变量

```Bash
source ~/.bashrc
```

1. 查看nvcc版本，测试是否成功

```Bash
nvcc -V
```

# 