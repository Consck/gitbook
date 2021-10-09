### 字符类型转换
* string转成int：
int, err := strconv.Atoi(string)
* string转成int64：
int64, err := strconv.ParseInt(string, 10, 64)
* int转成string：
string := strconv.Itoa(int)
* int64转成string：
string := strconv.FormatInt(int64,10)


### flag包

接收命令行传入的参数，提供参数处理的功能

`flag.StringVar(&name, "name", "匿名","姓名")`
第一个参数：存放值的参数地址
第二个参数：命令行参数的名称
第三个参数：命令行不输入时的默认值
第四个参数：该参数的描述信息，help命令时会显示

将命令行输入的参数传递到代码中的变量主要有两种方式：
第一种：StringVar和IntVar等方法，第一个参数是变量的地址；
第二种：String和Int等方法，将入参的值存入一个变量中，再将此变量的地址作为返回值返回；

### sync.RWMutex和sync.Mutex

Mutex为互斥锁，Lock()加锁，Unlock()解锁，使用Lock()加锁后，便不能再次对其进行加锁，直到利用Unlock()解锁对其解锁后，才能再次加锁．适用于读写不确定场景，即读写次数没有明显的区别，并且只允许只有一个读或者写的场景，所以该锁叶叫做全局锁

RWMutex是一个读写锁，该锁可以加多个读锁或者一个写锁，其经常用于读次数远远多于写次数的场景

### 命令
> 打包go脚本
go build -ldflags "-w -s"

