# pchar: A Tool for Measuring Internet Path Characteristics   
***  

pchar是由Bruce A. Mah发布的一个检测链路状态的工具，其主页: http://www.kitchenlab.org/www/bmah/Software/pchar/
pchar利用与traceroute一致的原理，对链路的每一条的网络特征(如带宽、RTT、丢包率)进行探测。  

### 获取pchar  
***  

目前pchar的最新版是2005年发布的1.5版本，已经有十多年没有更新过。
可以通过以下命令获取该软件：  

    # wget http://www.kitchenlab.org/www/bmah/Software/pchar/pchar-1.5.tar.gz

pchar的编译和安装非常简单，在源码的README中也写的非常详细。  

### pchar能做什么？  
***  
直接执行一次./pchar就可以看到它的usage。简单来说，pchar支持使用多种协议，使用多种数据包大小来探测链路特征。  
具体的探测原理与traceroute一致，即通过依次设置IP数据包的TTL值为1,2,3...，  
然后分析各个节点在将TTL减为0时返回的Destination unreachable 的ICMP包来获取各个链路节点的网络特征。  
下面是使用pchar进行测试的两个样例输出, 各参数的含义请查看usage：  

    # ./pchar server -R 1 -I 256 -v
    pchar to server (221.168.20.22) using UDP/IPv4
    Using raw socket input
    Packet size increments from 32 to 1500 by 256
    5 test(s) per repetition
    1 repetition(s) per hop
     0: 221.168.10.11 (221.168.10.11)
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = 10.409070 ms, (b = 0.000014 ms/B), r2 = 0.097122
                           stddev rtt = 0.031562, stddev b = 0.000025
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 10.409070 ms, bw = 570400.219841 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
     1: 221.168.10.12 (221.168.10.12)
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = 20.678756 ms, (b = 0.000042 ms/B), r2 = 0.459694
                           stddev rtt = 0.033828, stddev b = 0.000026
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 10.269686 ms, bw = 283192.316089 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
     2: 221.168.20.22 (server)
        Path length:       2 hops
        Path char:         rtt = 20.678756 ms r2 = 0.459694
        Path bottleneck:   283192.316089 Kbps
        Path pipe:         732008 bytes
        Path queueing:     average = 0.000000 ms (0 bytes)
        Start time:        Wed Feb 17 23:05:41 2016
        End time:          Wed Feb 17 23:05:43 2016


    # ./pchar server -R 1 -I 256 -v -p ipv4icmp
    pchar to server (221.168.20.22) using ICMP/IPv4 (raw sockets)
    Using raw socket input
    Packet size increments from 32 to 1500 by 256
    5 test(s) per repetition
    1 repetition(s) per hop
     0: 221.168.10.11 (221.168.10.11)
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = 10.412832 ms, (b = 0.000013 ms/B), r2 = 0.759147
                           stddev rtt = 0.005453, stddev b = 0.000004
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 10.412832 ms, bw = 609921.955806 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
     1: 221.168.10.12 (221.168.10.12)
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = 20.712600 ms, (b = 0.000039 ms/B), r2 = 0.735101
                           stddev rtt = 0.021003, stddev b = 0.000014
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 10.299768 ms, bw = 303758.716799 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
     2: 221.168.20.22 (server)
        Path length:       2 hops
        Path char:         rtt = 20.712600 ms r2 = 0.735101
        Path bottleneck:   303758.716799 Kbps
        Path pipe:         786454 bytes
        Path queueing:     average = 0.000000 ms (0 bytes)
        Start time:        Wed Feb 17 21:54:54 2016
        End time:          Wed Feb 17 21:54:56 2016

在上面两例中，主要为了节省执行时间才使用了-R、-I选项，所以在测试准确性上面可能不太理想。  
如果不介意时间慢(默认的配置真的是非常的慢！)，可以使用默认的配置，从而获得更为准确的测试结果。  

从测试报告中，可以看到pchar会从以下几个方面来探测链路的状况：  
a. 丢包率  
b. RTT及波动  
c. 排队时延  
d. 瓶颈  

### 附录： 真实网络探测案例一则  
已隐去IP地址信息，并且使用ICMP协议探测。目前感觉用ICMP探测比用UDP探测会遇到更少的no response的情况。  
    # ./pchar X.X.X.X -I 256 -v -R 1 -p ipv4icmp
    pchar to X.X.X.X using ICMP/IPv4 (raw sockets)
    Using raw socket input
    Packet size increments from 32 to 1500 by 256
    5 test(s) per repetition
    1 repetition(s) per hop
     0: 
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = -1.319881 ms, (b = 0.005284 ms/B), r2 = 0.504432
                           stddev rtt = 2.491418, stddev b = 0.003024
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = --.--- ms, bw = 1514.120952 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
     1: 
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = -0.547262 ms, (b = 0.003880 ms/B), r2 = 0.544305
                           stddev rtt = 1.688805, stddev b = 0.002050
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 0.772619 ms, bw = --.--- Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
     2: 
        Partial loss:      4 / 5 (80%)                                              
        Partial char:      rtt = -nan ms, (b = -nan ms/B), r2 = -nan
                           stddev rtt = -nan, stddev b = -nan
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 0.000000 ms, bw = 0.000000 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
     3: 
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = 1.741813 ms, (b = -0.000207 ms/B), r2 = 0.345265
                           stddev rtt = 0.142484, stddev b = 0.000165
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 0.000000 ms, bw = 0.000000 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
     4: 
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = 5.060113 ms, (b = 0.000833 ms/B), r2 = 0.526933
                           stddev rtt = 0.393811, stddev b = 0.000456
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 3.318300 ms, bw = 7687.687688 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
     5: 
        Partial loss:      5 / 5 (100%)                                             
        Partial char:      rtt = 0.000000 ms, (b = 0.000000 ms/B), r2 = 0.000000
                           stddev rtt = 0.000000, stddev b = 0.000000
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 0.000000 ms, bw = 0.000000 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
     6: no probe responses
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = 23.298387 ms, (b = 0.003157 ms/B), r2 = 0.313270
                           stddev rtt = 2.331951, stddev b = 0.002699
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 23.298387 ms, bw = 2533.712730 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
     7: 
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = 4.769912 ms, (b = 0.001358 ms/B), r2 = 0.577582
                           stddev rtt = 0.579405, stddev b = 0.000671
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = --.--- ms, bw = --.--- Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
     8: 
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = 5.407413 ms, (b = 0.000411 ms/B), r2 = 0.524421
                           stddev rtt = 0.195395, stddev b = 0.000226
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 0.637500 ms, bw = --.--- Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
     9: 
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = 4.533725 ms, (b = -0.000068 ms/B), r2 = 0.252384
                           stddev rtt = 0.058354, stddev b = 0.000068
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 0.000000 ms, bw = 0.000000 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
    10: 
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = 4.163003 ms, (b = 0.000266 ms/B), r2 = 0.547205
                           stddev rtt = 0.115120, stddev b = 0.000140
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = --.--- ms, bw = 23953.216374 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
    11: 
        Partial loss:      5 / 5 (100%)                                             
        Partial char:      rtt = 0.000000 ms, (b = 0.000000 ms/B), r2 = 0.000000
                           stddev rtt = 0.000000, stddev b = 0.000000
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 0.000000 ms, bw = 0.000000 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
    12: no probe responses
        Partial loss:      5 / 5 (100%)                                             
        Partial char:      rtt = 0.000000 ms, (b = 0.000000 ms/B), r2 = 0.000000
                           stddev rtt = 0.000000, stddev b = 0.000000
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 0.000000 ms, bw = 0.000000 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
    13: no probe responses
        Partial loss:      0 / 5 (0%)                                               
        Partial char:      rtt = 5.117400 ms, (b = -0.000092 ms/B), r2 = 0.080097
                           stddev rtt = 0.275880, stddev b = 0.000180
        Partial queueing:  avg = 0.000000 ms (0 bytes)
        Hop char:          rtt = 0.000000 ms, bw = 0.000000 Kbps
        Hop queueing:      avg = 0.000000 ms (0 bytes)
    14: 
        Path length:       14 hops
        Path char:         rtt = 5.117400 ms r2 = 0.080097
        Path bottleneck:   1514.120952 Kbps
        Path pipe:         968 bytes
        Path queueing:     average = 0.000000 ms (0 bytes)
        Start time:        Wed Feb 17 13:27:21 2016
        End time:          Wed Feb 17 13:28:31 2016
