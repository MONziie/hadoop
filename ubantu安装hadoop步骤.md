### 一、创建Hadoop用户
按 ctrl+alt+t 打开终端窗口，输入如下命令创建新用户 :

'''
sudo useradd -m hadoop -s /bin/bash
'''
###### 这条命令创建了可以登陆的 hadoop 用户，并使用 /bin/bash 作为 shell

接着，设置密码：（以下命令设置为hadoop）

'''
sudo passwd hadoop
'''

    可为 hadoop 用户增加管理员权限，方便部署：

    '''
    sudo adduser hadoop sudo
    '''
    
最后，创建好用户要注销当前用户，右上角选择注销，返回登录hadoop用户


### 二、更新apt
1. 使用 apt 安装软件,新用户登录后先更新

按 ctrl+alt+t 打开终端窗口，执行：

'''
sudo apt-get update
'''

#######  ADDITION 更改软件源 ADDITION
若出现 “Hash校验和不符” 的提示，可通过更改软件源来解决。操作步骤：
【系统设置】--【软件和更新】--【下载自】--【其他节点】--mirrors.aliyun.com】--【选择服务器】
-- 提示列表信息过时，点击【重新载入】--。如果还是提示错误，请选择其他服务器节点如 mirrors.163.com 再次进行尝试。更新成功后，再次执行 sudo apt-get update 就正常了。

2. 更改配置文件 VIM
安装vim
'''
sudo apt-get install vim
'''


### 三、安装SSH、配置SSH无密码登陆
集群、单节点模式都需要用到 SSH 登陆
Ubuntu 默认已安装了 SSH client，此外还需要安装 SSH server：
'''
sudo apt-get install openssh-server
'''


安装后，可以使用如下命令登陆本机：
'''
ssh localhost
'''

###### 为方便每次登录不输入密码，配置成SSH无密码登陆：
###### 首先回到终端窗口，使用ssh-keygen 生成密钥，并将密钥加入到授权中：

'''
exit                           # 退出刚才的 ssh localhost
cd ~/.ssh/                     # 若没有该目录，请先执行一次ssh localhost
ssh-keygen -t rsa              # 会有提示，都按回车就可以
cat ./id_rsa.pub >> ./authorized_keys  # 加入授权
'''



### 四、安装Java环境

三种安装JDK的方式：
#### （1）第1种安装JDK方式（手动安装，推荐采用本方式）

A. 请把压缩格式的文件jdk-8u162-linux-x64.tar.gz下载到本地电脑，假设保存在“/home/linziyu/Downloads/”目录下。
在Linux命令行界面中，执行如下Shell命令（注意：当前登录用户名是hadoop）：

'''
cd /usr/lib
sudo mkdir jvm        #创建/usr/lib/jvm目录用来存放JDK文件
cd ~                  #进入hadoop用户的主目录
cd Downloads          #注意区分大小写字母，刚才已经通过FTP软件把JDK安装包jdk-8u162-linux-x64.tar.gz上传到该目录下
sudo tar -zxvf ./jdk-8u162-linux-x64.tar.gz -C /usr/lib/jvm   
                      #把JDK文件解压到/usr/lib/jvm目录下
'''

B. JDK文件解压缩以后，可以执行如下命令到/usr/lib/jvm目录查看一下：

'''
cd /usr/lib/jvm
ls
'''
可以看到，在/usr/lib/jvm目录下有个jdk1.8.0_162目录。
下面继续执行如下命令，设置环境变量：

'''
cd ~
vim ~/.bashrc
'''



C. 上面命令使用vim编辑器（查看vim编辑器使用方法）打开了hadoop这个用户的环境变量配置文件，请在这个文件的开头位置，添加如下几行内容：
'''
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_162
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
'''
保存.bashrc文件并退出vim编辑器。
然后，继续执行如下命令让.bashrc文件的配置立即生效：

'''
source ~/.bashrc
'''


这时，可以使用如下命令查看是否安装成功：
'''
java -version
'''

如果能够在屏幕上返回如下信息，则说明安装成功：

  hadoop@ubuntu:~$ java -version
  java version "1.8.0_162"
  Java(TM) SE Runtime Environment (build 1.8.0_162-b12)
  Java HotSpot(TM) 64-Bit Server VM (build 25.162-b12, mixed mode)`````````````````````````````````


#### （2）第2种安装JDK方式：
安装好 OpenJDK 

'''
sudo apt-get install openjdk-7-jre openjdk-7-jdk
'''

找到相应的安装路径，这个路径是用于配置 JAVA_HOME 环境变量的:

'''
dpkg -L openjdk-7-jdk | grep '/bin/javac'
'''

该命令会输出一个路径，除去路径末尾的 “/bin/javac”，剩下的就是正确的路径了。如输出路径为 /usr/lib/jvm/java-7-openjdk-amd64/bin/javac，则我们需要的路径为 /usr/lib/jvm/java-7-openjdk-amd64。

接着需要配置一下 JAVA_HOME 环境变量，为方便，我们在 ~/.bashrc 中进行设置:

'''
vim ~/.bashrc
'''

在文件最前面添加如下单独一行（注意 = 号前后不能有空格），将“JDK安装路径”改为上述命令得到的路径，并保存：

'''
export JAVA_HOME=JDK安装路径
'''

接着还需要让该环境变量生效，执行如下代码：

'''
source ~/.bashrc    # 使变量设置生效
'''

设置好后我们来检验一下是否设置正确：

'''
echo $JAVA_HOME     # 检验变量值
java -version
$JAVA_HOME/bin/java -version  # 与直接执行 java -version 一样
'''

如果设置正确的话，$JAVA_HOME/bin/java -version
这样，Hadoop 所需的 Java 运行环境就安装好了。


#### （3）第3种安装JDK方式

根据大量电脑安装Java环境的情况我们发现，部分电脑按照上述的第一种安装方式会出现安装失败的情况，这时，可以采用这里介绍的另外一种安装方式，命令如下：

'''
sudo apt-get install default-jre default-jdk
'''

上述安装过程需要访问网络下载相关文件，请保持联网状态。安装结束以后，需要配置JAVA_HOME环境变量，请在Linux终端中输入下面命令打开当前登录用户的环境变量配置文件.bashrc：

'''
vim ~/.bashrc
'''

在文件最前面添加如下单独一行（注意，等号“=”前后不能有空格），然后保存退出：

export JAVA_HOME=/usr/lib/jvm/default-java

接下来，要让环境变量立即生效，请执行如下代码：

'''
source ~/.bashrc    # 使变量设置生效
'''

执行上述命令后，可以检验一下是否设置正确：

'''
echo $JAVA_HOME     # 检验变量值
java -version
$JAVA_HOME/bin/java -version  # 与直接执行java -version一样
'''

至此，就成功安装了Java环境。下面就可以进入Hadoop的安装。












