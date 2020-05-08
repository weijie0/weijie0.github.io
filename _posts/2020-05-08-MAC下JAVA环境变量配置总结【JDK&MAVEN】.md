### MAC下JAVA环境变量配置总结【JDK&MAVEN】

环境变量配置：

1、打开终端Terminal；

2、进入当前用户主目录，cd ~；

3、临时授权，sudo su；

4、输入密码；

5、创建.bash_profile文件，touch .bash_profile（如果存在则不必新建）；

6、打开.bash_profile文件，open .bash_profile（能打开则新建成功）；

7、输入jdk文件路径

*JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home*（jdk安装路径）

*MAVEN_HOME=/Users/\*/Downloads/maven-3.3.9*(maven安装路径)

 

export PATH=${PATH}:${MAVEN_HOME}/bin:${JAVA_HOME}/bin

保存并退出；

8、读取并执行文件中的命令，source .bash_profile；

 

9、在Terminal中输入java -version，显示jdk信息，则jdk配置成功

9、在Terminal中输入mvn -v，显示maven信息，则maven配置成功

 

PS：如果open命令打开文件不能编辑，可使用vi命令进行编辑
