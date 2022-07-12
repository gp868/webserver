# 项目技术
Linux C++、epoll、线程池、HTTP、多线程、MySQL

# 项目难点
1. 使用线程池 + 非阻塞socket + epoll(ET和LT均实现) + 事件处理(Reactor和模拟Proactor均实现) 的高并发模型；
2. 使用状态机解析HTTP请求报文，支持解析GET和POST请求，处理静态资源的请求；
3. 基于升序链表实现定时器，关闭超时的非活动连接；
4. 利用单例模式与阻塞队列实现同步/异步日志系统，记录服务器运行状态和错误信息；
5. 利用RAII机制实现了数据库连接池，减少数据库连接建立与关闭的开销；

# 测试
    ```C++
    // 建立yourdb库
    create database myDB;

    // 创建user表
    USE myDB;
    CREATE TABLE user(
        username char(50) NULL,
        passwd char(50) NULL
    );

    // 添加数据
    INSERT INTO user(username, passwd) VALUES('name', 'passwd');
    ```

* 修改main.cpp中的数据库初始化信息

    ```C++
    //数据库登录名,密码,库名
    string user = "root";
    string passwd = "****";
    string databasename = "myDB";
    ```

* build 编译项目

    ```C++
    sh ./build.sh
    ```

* 启动server服务器项目

    ```C++
    ./server
    ```

* 浏览器端 打开指定网页

    ```C++
    127.0.0.1:9006
    ```

# 个性化运行

```C++
./server [-p port] [-l LOGWrite] [-m TRIGMode] [-o OPT_LINGER] [-s sql_num] [-t thread_num] [-c close_log] [-a actor_model]
```

温馨提示:以上参数不是非必须，不用全部使用，根据个人情况搭配选用即可.

* -p，自定义端口号
	* 默认9006
* -l，选择日志写入方式，默认同步写入
	* 0，同步写入
	* 1，异步写入
* -m，listenfd和connfd的模式组合，默认使用LT + LT
	* 0，表示使用LT + LT
	* 1，表示使用LT + ET
  * 2，表示使用ET + LT
  * 3，表示使用ET + ET

-o，优雅关闭连接，默认不使用
* 0，不使用
* 1，使用

* -s，数据库连接数量
  * 默认为8
* -t，线程数量
  * 默认为8
* -c，关闭日志，默认打开
  * 0，打开日志
  * 1，关闭日志
* -a，选择反应堆模型，默认Proactor
  * 0，Proactor模型
  * 1，Reactor模型

测试示例命令与含义

```C++
./server -p 9007 -l 1 -m 0 -o 1 -s 10 -t 10 -c 1 -a 1
```

- [x] 端口9007
- [x] 异步写入日志
- [x] 使用LT + LT组合
- [x] 使用优雅关闭连接
- [x] 数据库连接池内有10条连接
- [x] 线程池内有10条线程
- [x] 关闭日志
- [x] Reactor反应堆模型


# 演示
- 浏览器输入`ip:port`显示welcome界面
![image1.png](https://s2.loli.net/2022/07/12/Pf3NzocekwqyprB.png)
- 点击⌈新用户⌋进入注册页面，点击⌈已有账号⌋进入登录界面
  - 注册
![image2.png](https://s2.loli.net/2022/07/12/wmEQipXxMANsa2R.png)
  - 登录
![image.png](https://s2.loli.net/2022/07/12/jvGLcgRiTaKBWrO.png)
- 登录后可访问图片和视频
![image3.png](https://s2.loli.net/2022/07/12/PYDShAdBECif3QK.png)
  - 访问图片
![image4.png](https://s2.loli.net/2022/07/12/hawuHseDOol7MIN.png)
![image6.png](https://s2.loli.net/2022/07/12/kEualyK4HUSeXrP.png)
  - 访问视频
![image5.png](https://s2.loli.net/2022/07/12/cAMDHQJkt5hKiWG.png)

# 压测
Webbench是Linux 上一款知名的、优秀的web性能压力测试工具。它是由Lionbridge公司开发。

- 测试处在相同硬件上，不同服务的性能以及不同硬件上同一个服务的运行状况。
- 展示服务器的两项内容：每秒钟响应请求数和每秒钟传输数据量。

基本原理：Webbench 首先fork出多个子进程，每个子进程都循环做web访问测试。子进程把访问的结果通过pipe告诉父进程，父进程做最终的统计结果。

测试示例
```
webbench -c 10000 -t 5 http://192.168.110.129:10000/index.html
参数：
  -c 表示客户端数
  -t 表示时间
```

本项目测试结果如下，可达到上万并发。
![VYOK@0NIO6G_E2KIK~0U_43.png](https://s2.loli.net/2022/07/12/uamFSxihzsgnCDR.png)

