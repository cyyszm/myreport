# myreport
二、四周中，我完成的工作主要包括以下几个部分：
1.	学习、初步了解流程和需求
在实习前期，我学习了数据通信产品线上我们需要完成的工作，主要是webuild流程。首先我们装电脑并下载需要的工具，接触到了很多之前没有用过的软件，有wecode、MobaXterm、tortoisegit等；学习华为编程语言规范，除了学习熟悉的语言Python、C++，还学习了不熟悉的语言java；学习webuild流程；了解C++组需要实现的hook并初步了解hook和wrapper如何拼接；学习wrapper模块需要哪些功能，以及进入主程序后经过哪些流程。
程序主要在Linux系统上测试，通过MobaXterm远程连接linux系统。在此过程中熟悉了更多Linux命令，查看、编辑、移动、编译文件等。
2.	实现wrapper.py文件
wrapper文件由python编写，主要包含以下功能：执行gcc等指令，记录执行指令的开始时间和结束时间，由此计算指令执行的时长。解析gcc，make等指令，分析出指令中包含的信息，有源文件名称、目标文件名称、源文件路径、目标文件路径等，用于下一步信息的计算。在这个过程中我学习到了gcc命令是由什么组成的，-c把文件激活预处理，编译和汇编，生成obj文件，-o指定目标名称。通过向同学请教知道python库中有getopt函数可以解析指令。得到文件名称后，计算SHA1加密值。利用python中hashlib库，输入文件路径，得到SHA1值。
利用python的subprocss创建和管理子进程，将shell参数设置为True，则通过操作系统的shell执行指定的命令。设置env环境变量参数为my_env，表示是修改过的可以被hook到的环境变量。利用subprocess.Popen.pid获取进程的pid。
计算CPU使用率和内存使用率。生成随机数为每个任务设置id。将以上信息组装成字典写入json文件，后续通过生产者kafka传给消费者kafka.
3.	实现主程序
进入主程序后，将hook.c程序编译成so文件，生成动态链接库。设置LD_PRELOAD环境变量，使动态库加载的优先级最高。LD_PRELOAD环境变量可以影响程序的运行时的链接，允许定义在程序运行前优先加载的动态链接库。通过这个功能有选择性的载入操作系统不同动态链接库中的相同函数。通过这个环境变量，我们可以在主程序和其动态链接库的中间加载别的动态链接库，覆盖正常的函数库。可以以此功能来使用hook中的函数。
设置socket接口接收json文件。socket是“open—write/read—close”模式的一种实现，socket就提供了这些操作对应的函数接口。socket函数对应于普通文件的打开操作。普通文件的打开操作返回一个文件描述字，而socket()用于创建一个socket描述符（socket descriptor），它唯一标识一个socket。这个socket描述字跟文件描述字一样，后续的操作都有用到它，把它作为参数，通过它来进行一些读写操作。当我们调用socket创建一个socket时，返回的socket描述字它存在于协议族（address family，AF_XXX）空间中，但没有一个具体的地址。如果想要给它赋值一个地址，就必须调用bind()函数。通常服务器在启动的时候都会绑定一个众所周知的地址（如ip地址+端口号），用于提供服务，客户就可以通过它来接连服务器；而客户端就不用指定，有系统自动分配一个端口号和自身的ip地址组合。如果作为一个服务器，在调用socket()、bind()之后就会调用listen()来监听这个socket，如果客户端这时调用connect()发出连接请求，服务器端就会接收到这个请求。TCP服务器端依次调用socket()、bind()、listen()之后，就会监听指定的socket地址了。TCP客户端依次调用socket()、connect()之后就想TCP服务器发送了一个连接请求。TCP服务器监听到这个请求之后，就会调用accept()函数取接收请求，这样连接就建立好了。
建立生产者kafka，将json文件传给消费者kafka。Kafka是一个分布式的，支持多分区、多副本，基于 Zookeeper 的分布式消息流平台。Kafka中的数据单元被称为消息，消息的种类称为topic。向主题发布消息的客户端应用程序被称为生产者，持续不断向某个主题发送消息。Kafka.producer的参数有topic名称，key值，value值和partition分区。为了保证kafka的吞吐量，一个Topic可以设置多个分区。同一分区只能被一个消费者订阅。如果消费者端保持打开状态，就可以监听到生产端的消息。
三、	实习环节中遇到的问题及困扰和具体解决方案
一开始尝试安装linux系统时安装失败，可能是系统权限问题。后来连接了远程linux系统。
遇到较为复杂的gcc、g++指令时，不知道如何解析，也不确定每个部分的含义是什么。查阅资料后了解到-o指定可执行程序的名称，-c只要求编译器输出目标代码，而不用生成可执行程序，-g要求编译器在编译的时候提供我们以后对程序进行调试的信息。之后需要自学编译原理。
使用getopt库解析指令时，发现其对参数的识别较为局限。如果指令中有./+文件名的形式，则不能识别出来，且getopt函数识别出的opts是几段字符串组成的list形式，要提取出后面的文件名不易分割。因此将从头扫描整条指令，找到相应的./符号，将./符号替换成可以识别的参数符号，但不能影响其他本来的参数符号。
在获取时间和计算时长时，time包报错，换成datetime包后可以使用。但是datetime包输出的时间格式不是我们想要的格式，而且如果用默认的格式表示起止时间，计算时长时会溢出。搜索后了解到datetime包含有很多参数，可以改变参数来改变输出字符串的格式。选择合适的参数后计算时长就不会溢出。
执行hook.c时，会出现循环，原因是hook中调用wrapper，wrapper中设置了环境变量，将hook提高到最高的优先级，再次调用hook，以此循环。解决方案是加一个判断条件判断是否被hook，也就是代码中的TO_BE_HOOKED==0。加了判断条件后没有再出现循环。
四、	自己的体会
在四周的实习实践中，我学到了很多知识。在前期阶段通过听老师的讲解和分享了解了产品线上的主要流程和主要技术，认识到学校所学的知识和工作实践有很大区别。中期通过自己尝试写代码锻炼了代码能力，学习到很多代码库的使用。后期通过和C++组的交互联调，学习到两种语言的文件如何联系，可以通过argparse等库传递参数。
