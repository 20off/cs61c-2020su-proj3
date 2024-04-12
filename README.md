# CS61CPU

Look ma, I made a CPU! Here's what I did:
整个构造中control logic花费时间最多，首先来讲讲这个
Control Logic:
    一、基本构造
    我选择了使用ROM来构建control logic，将control logic分为三部分，address decod
    -er、ROM、state control，这种构造基本思路如下，将输入的instrcutio
    -n通过and 以及 not gate解码（分析指令中opcode，fun3，fun7来解码），然后映射
    为ROM中相应控制信号的address，address decoder输出地址后，ROM接受地址并输出对
    应的控制信号，从而控制processor。state control用来处理需要判断的指令的信号输
    出（比如branch）
    
    二、address decoder
    解码部分，我选择分阶段解码，首先解码opcode确认指令类型，然后更具fun3和fun7进
    一步解码，每个指令的解码电路输出都由MUX控制，如果解码结果为1，就输出地址，否
    则输出0，解码成功后，所有的解码电路输出将只有一个是正确地址，其他都是0，最后
    使用or gate来得到正确的地址输出。

    三、ROM
    本次proj3，总共有38个指令左右，而必要输出的控制信号在16个bit，由此可以确定RO
    M的input address位数以及内部每个储存单位的bit数。输出信号的最高位，控制branc    -h，如果为1，表明这是一个Branch指令，PCSel信号需要根据后续结果而相应变化。这
    个bit就用来控制PCsel是由输出信号直接控制还是由COmparator的结果控制。

    四、state control
    除了上面提到的Branch的控制之外，由于需要实现swlt，我们也需要控制MEMRW信号，
    这也将收到Comparator结果的影响（首先得确认是这条指令），所以我们需要三重控制
    首先是这是S-type指令，输出信号将MEMRW变为1，然后我们需要直接通过instruction
    中的func3，区分是普通的sw还是swlt，如果是sw，那么MEMRW直接与输出信号相连接，
    而swlt则使之交由Conparater控制。

Cpu:
    Cpu中比较头疼的是L-type（I-type的一种），由于要lb，lw，lh三种load，然而contr
    -ol logic output对于这三个指令都是一样的，所以我们需要在cpu中通过func3直区分
    分这三者。swlt指令也类似，由于是完全不存在的指令，swlt的物理结构与之前的结构
    完全不相容，所以不得不为其专门搭建一条通路（MUX），然后直接有instrution中的f
    unc3部分控制。至于cpu不知道指令类型（没有经过解码）直接使用instruction，会不
    会产生这种控制干扰了其他类型指令的执行这种问题呢？实际上只要巧妙安排是不会的
    这与cpu的运行逻辑有关。就像插入nop指令一样，只要不进行写入（改变cpu和regfile
    状态）那么这个指令执行了和没执行没区别。
    另外注意到pipeline时，nop指令应该在instruction register的input处插入，不然
    可能出现错误指令已经传递给了excute部分时才插入nop的情况。另外一定要注意PC-4
    这个部分，jump指令需要储存PC-4，然而write-back stage不可能在这个cycle中就完
    成存储，需要在下个cycle中才行，而下个cycle中pc-4实际上又被加了2，变成了PC-8
    所以pc-4也需要register来分stage存储。

这个proj3花费了我一整周时间，现在终于完成了，希望接下来我的计算机课程学习能够更加顺利，希望这个cpu能帮助我能考上心仪的学校吧hhhh


-
