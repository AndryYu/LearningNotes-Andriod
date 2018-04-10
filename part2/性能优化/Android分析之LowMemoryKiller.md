## Android分析之LowMemoryKiller
Android Kernel会定时执行一次检查，杀死一些进程，释放掉内存。依据的是Low memory killer机制。Low memory killer则是定时进行检查，主要通过进程的oom_adj来判断进程的重要程度。这个值越小，程序约重要，被杀的可能性越低。oom_adj的大小与进程的类型以及被调度的次序有关。

### oom_adj的值
adj文件存放着oom_adj内存警戒值(以4k为单位)
* 0  ==    1536    ==   FOREGROUND_APP_ADJ(前台进程)
* 1  ==    2048    ==   VISIBLE_APP_ADJ(可见进程)
* 2  ==    4096    ==   SECONDARY_SERVER_ADJ(可感知进程，比如后台音乐播放)
* 4  ==    xxxx    ==   HEAVY_WEIGHT_APP_ADJ(后台的重量级进程)
* 7  ==    5120    ==   HIDDEN_APP_MIN_ADJ
* 14  ==    5632   ==   CONTENT_PROVIDER_ADJ
* 15  ==    6144   ==   EMPTY_APP_ADJ(空进程)

也就是说，
* 当系统的剩余内存为小于6M时，警戒级数为0;
* 当系统内存剩余小于8M而大于6M时，警戒级数为1；
* 当内存小于64M大于16M的时候，警戒级数为12;

### LowMemoryKiller的工作机制
LMK开始工作时，首先根据阈值表确定当前的警戒级数，高于警戒级数的进程是待杀的范围。然后遍历所有进程的oom_adj值，找到大于min_adj的进程。若找到多个，则把占用进程最大的进程存放在selected中。最关键的一步就是，发送SIGKILL信息，杀掉该进程。
