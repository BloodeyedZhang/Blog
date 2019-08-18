#  
# 快速导航
  * [设置下载镜像](#设置下载镜像)
 
# 设置下载镜像

$ cd .cargo 

$ vim config

 在文件中填入以下内容
 ```
[registry]
index = "https://mirrors.ustc.edu.cn/crates.io-index/"
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index/"
```