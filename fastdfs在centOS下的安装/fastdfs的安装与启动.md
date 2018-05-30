1.下载安装 libfastcommon ，这里是通过wget下载（我喜欢这种方式）。
wget https://github.com/happyfish100/libfastcommon/archive/V1.0.7.tar.gz
解压 libfastcommon，命令：
tar -zxvf V1.0.7.tar.gz
编译，进入libfastcommon-1.0.7目录，命令：
cd libfastcommon-1.0.7
./make.sh
安装，命令：
./make.sh install

2.下载安装 FastDFS，这里也是通过wget下载。
wget https://github.com/happyfish100/fastdfs/archive/V5.05.tar.gz
解压 FastDFS ，命令：
tar -zxvf V5.05.tar.gz
编译，进入fastfds-5.05目录，命令：
cd fastdfs-5.05
./make.sh
安装，命令：
./make.sh install



配置 Tracker 服务
上述安装成功后，在/etc/目录下会有一个fdfs的目录，进入它。会看到三个.sample后缀的文件，这是作者给我们的示例文件，我们需要把其中的tracker.conf.sample文件改为tracker.conf配置文件并修改它。看命令：

cp tracker.conf.sample tracker.conf
vim tracker.conf


打开tracker.conf文件，只需要找到你只需要该这两个参数就可以了。

# the base path to store data and log files
base_path=/data/fastdfs
# HTTP port on this tracker server
http.server_port=80

当然前提是你要有或先创建了/data/fastdfs目录。port=22122这个端口参数不建议修改，除非你已经占用它了。
修改完成保存并退出 vim ，这时候我们可以使用/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf start来启动 Tracker服务，但是这个命令不够优雅，怎么做呢？使用ln -s 建立软链接：

ln -s /usr/bin/fdfs_trackerd /usr/local/bin
ln -s /usr/bin/stop.sh /usr/local/bin
ln -s /usr/bin/restart.sh /usr/local/bin

这时候我们就可以使用service fdfs_trackerd start来优雅地启动 Tracker服务了，是不是比刚才带目录的命令好记太多了（懒是社会生产力）。你也可以启动过服务看一下端口是否在监听，命令：

启动服务：service fdfs_trackerd start
查看监听：netstat -unltp|grep fdfs


配置 Storage 服务
现在开始配置 Storage 服务，由于我这是单机器测试，你把 Storage 服务放在多台服务器也是可以的，它有 Group(组)的概念，同一组内服务器互备同步，这里不再演示。直接开始配置，依然是进入/etc/fdfs的目录操作，首先进入它。会看到三个.sample后缀的文件，我们需要把其中的storage.conf.sample文件改为storage.conf配置文件并修改它。还看命令：

cp storage.conf.sample storage.conf
vim storage.conf


打开storage.conf文件后，找到这两个参数进行修改：

# the base path to store data and log files
base_path=/data/fastdfs/storage
# store_path#, based 0, if store_path0 not exists, it's value is base_path
# the paths must be exist
store_path0=/data/fastdfs/storage
#store_path1=/home/yuqing/fastdfs2
# tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
tracker_server=192.168.198.129:22122



当然你的/data/fastdfs目录下要有storage文件夹，没有就创建一个，不然会报错的，日志以及文件都会在这个下面，启动时候会自动生成许多文件夹。stroage的port=23000这个端口参数也不建议修改，默认就好，除非你已经占用它了。
修改完成保存并退出 vim ，这时候我们依然想优雅地启动 Storage服务，带目录的命令不够优雅，这里还是使用ln -s 建立软链接：

ln -s /usr/bin/fdfs_storaged /usr/local/bin

执行命令启动服务：

service fdfs_storaged start

我们安装配置并启动了 Tracker 和 Storage 服务，也没有报错了。那他俩是不是在通信呢？我们可以监视一下：

/usr/bin/fdfs_monitor /etc/fdfs/storage.conf


stop是停止服务