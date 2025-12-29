# running copyparty on windows （在windows上运行copyparty）

this is a complete example / quickstart for running copyparty on windows, optionally as a service (autostart on boot)

这是一个在Windows上将copyparty作为自动启动服务运行的完整案例（快速指南）

you will definitely need either [copyparty.exe](https://github.com/9001/copyparty/releases/latest/download/copyparty.exe) (comfy, portable, more features) or [copyparty-sfx.py](https://github.com/9001/copyparty/releases/latest/download/copyparty-sfx.py) (smaller, safer)

你需要获取[copyparty.exe](https://github.com/9001/copyparty/releases/latest/download/copyparty.exe) (合适，可移动，更多功能) 或者 [copyparty-sfx.py](https://github.com/9001/copyparty/releases/latest/download/copyparty-sfx.py) (py脚本更小更安全)

* if you decided to grab `copyparty-sfx.py` instead of the exe you will also need to install the ["Latest Python 3 Release"](https://www.python.org/downloads/windows/)
* 如果你想要使用`copyparty-sfx.py`phython脚本而不是copyparty.exe版本运行copyparty，那么你需要安装最新的phython3运行环境 ["Latest Python 3 Release"](https://www.python.org/downloads/windows/)

then you probably want to download [FFmpeg](https://www.gyan.dev/ffmpeg/builds/ffmpeg-git-full.7z) and put `ffmpeg.exe` and `ffprobe.exe` in your PATH (so for example `C:\Windows\System32\`) -- this enables thumbnails, audio transcoding, and making music metadata searchable

然后你可能需要下载 [FFmpeg](https://www.gyan.dev/ffmpeg/builds/ffmpeg-git-full.7z) 并且将 `ffmpeg.exe` and `ffprobe.exe` 放置在你Windows系统环境下的 PATH (比如你系统的system32文件夹下 `C:\Windows\System32\`) -- 这样能让copyparty开启缩略图功能, 音频解码转码功能和音乐元数据变得可以搜索

## the config file（配置文件）

open up notepad and save the following as `c:\users\you\documents\party.conf` (for example)
打开记事本或者其他文字编辑软件并将其存储如下`c:\users\you\documents\party.conf` party.conf其实是txt文件

```yaml
[global]
  lo: ~/logs/cpp-%Y-%m%d.xz  # log to c:\users\you\logs\ 将日志存储到此路径
  e2dsa, e2ts, z    # sets 3 flags; see explanation 设置三个运行参数
  p: 80, 443  # listen on ports 80 and 443, not 3923 在80 443这2个端口监听，而不是软件默认的3923
  theme: 2    # 默认主题: protonmail-monokai
  lang: chi   # 默认语言：中文

[accounts]                  # 用户名和密码
  kevin: shangalabangala    # 定义kevin为用户名，密码为shangalabangala

[/]               # create a volume available at / 在`/`位置创建一个磁盘卷
  c:\pub          # sharing this filesystem location `/`位置磁盘卷对应的Windows主机的文件共享位置
  accs:           # and set permissions: 设置文件夹访问权限
    r: *          # everyone can read/download files, 每个人都能读取`/`（对应Windows下的`c:\pub`
    rwmd: kevin   # kevin can read/write/move/delete kevin用户能读取写入移动删除文件

[/inc]            # create another volume at /inc
  c:\pub\inc      # sharing this filesystem location
  accs:           # permissions:
    w: *          # everyone can upload, but not browse 所有用户包括匿名用户能够写入
    rwmd: kevin   # kevin is admin here too

[/music]          # and a third volume at /music
  ~/music         # which shares c:\users\you\music 注意~/music是相对目录，根据运行copyparty的Windows用户环境会发生改变
  accs:
    r: *
    rwmd: kevin
```


### config explained: [global]（对于配置文件[global]区域参数的解释）

the `[global]` section accepts any config parameters [listed here](https://ocv.me/copyparty/helptext.html), also viewable by running copyparty (either the exe or the sfx.py) with `--help`, so this is the same as running copyparty with arguments `--lo c:\users\you\logs\copyparty-%Y-%m%d.xz -e2dsa -e2ts -z -p 80,443 --theme 2 --lang nor`
在`[global]`区域能够接受所有列举在此[listed here](https://ocv.me/copyparty/helptext.html)的传入参数，你也可以通过 运行copyparty 带`--help` 传入参数查看。上文party.conf这也相当于用带参数的命令行运行`--lo c:\users\you\logs\copyparty-%Y-%m%d.xz -e2dsa -e2ts -z -p 80,443 --theme 2 --lang chi`
* `lo: ~/logs/cpp-%Y-%m%d.xz` writes compressed logs (the compression will make them delayed) 写入压缩日志（压缩会导致延时）
* `e2dsa` enables the file indexer, which enables searching and upload-undo `e2dsa`开启文件索引，能够搜索和上传
* `e2ts` enables music metadata indexing, making albums / titles etc. searchable too `e2ts`开启音乐文件元数据索引，专辑标题也能搜索
  * but the improved upload speed from `e2dsa` is not affected 但是`e2dsa`开启的上传加速不起作用
* `z` enables zeroconf, making the server available at `http://HOSTNAME.local/` from any other machine in the LAN `z`开启默认，让系统服务在`http://HOSTNAME.local/`上被LAN口所有机器访问
* `p: 80,443` listens on the ports `80` and `443` instead of the default `3923` `p: 80,443`让程序在`80`和`443`进行监听
* `lang: chi` sets default language to Chinese 设置默认语言为中文


### config explained: [accounts]

the `[accounts]` section defines all the user accounts, which can then be referenced when granting people access to the different volumes
`[accounts]`区域规定了所有用户账号，用于确认不同用户获取不同硬盘卷权限


### config explained: volumes

then we create three volumes, one at `/`, one at `/inc`, and one at `/music` 我们在样例里面创建了3个卷，分别在  `/`,与 `/inc`,和 `/music`
* `/` and `/music` are readable without requiring people to login (`r: *`) but you need to login as kevin to write/move/delete files (`rwmd: kevin`) `/` and `/music` 两个卷允许用户不登录读取，但是你需要以Kevin身份登录进行写入/移动/删除 文件操作
* anyone can upload to `/inc` but you must be logged in as kevin to see the files inside 所有人都能上传文件至卷`/inc`，但是你必须以Kevin身份登录查看文件


## run copyparty

to test your config it's best to just run copyparty in a console to watch the output:
在终端运行以下参数查看party.conf文件配置是否正确（小贴士，可以在win文件浏览器地址栏打cmd加回车进入该地址而不用使用烦人的CD（change dictionary）命令）

```batch
copyparty.exe -c party.conf
```
作为个人爱好习惯用bat启动，那么bat命令如下：

```batch
cd /d "%~dp0"
copyparty.exe -c party.conf
```
其中`cd /d "%~dp0" `表示bat获取自身目录，当然 `copyparty.exe`与 `party.conf`和bat在同一目录下。



or if you wanna use `copyparty-sfx.py` instead of the exe (understandable),
如果你使用`copyparty-sfx.py` phython脚本则运行以下命令：

```batch
%localappdata%\programs\python\python311\python.exe copyparty-sfx.py -c party.conf
```

(please adjust `python311` to match the python version you installed, i'm not good enough at windows to make that bit generic)
（配置`python311`为你安装的phython版本，311只是一个参考）


## run it as a service 将copyparty作为服务自启动（这个我不想用，所以不翻译了）

to run this as a service you need [NSSM](https://nssm.cc/ci/nssm-2.24-101-g897c7ad.zip), so put the exe somewhere in your PATH

then either do this for `copyparty.exe`:
```batch
nssm install cpp %homedrive%%homepath%\downloads\copyparty.exe -c %homedrive%%homepath%\documents\party.conf
```

or do this for `copyparty-sfx.py`:
```batch
nssm install cpp %localappdata%\programs\python\python311\python.exe %homedrive%%homepath%\downloads\copyparty-sfx.py -c %homedrive%%homepath%\documents\party.conf
```

then after creating the service, modify it so it runs with your own windows account (so file permissions don't get wonky and paths expand as expected):
```batch
nssm set cpp ObjectName .\yourAccoutName yourWindowsPassword
nssm start cpp
```

and that's it, all good

if it doesn't start, enable stderr logging so you can see what went wrong:
```batch
nssm set cpp AppStderr %homedrive%%homepath%\logs\cppsvc.err
nssm set cpp AppStderrCreationDisposition 2
```
