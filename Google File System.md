# Google File System

------

### 命名空间

        命名空间是一种用于组织和管理命名的抽象概念，命名空间用于将命名对象（例如文件、目录、变量、函数等）进行逻辑上的划分和组织，以避免命名冲突、提高可管理性和可扩展性。

        所谓“命名对象”，也就是需要用名字来标识，或者就是需要一个名字的对象。

       GFS中有两个命名空间，一个是文件命名空间，一个是块命名空间。通过命名空间，GFS可以将文件系统中的文件和数据块进行逻辑上的划分和组织，使得用户可以方便地对文件进行访问和管理，同时避免了文件命名冲突和混乱。

**块的命名空间**

- 块的命名空间用于管理数据块（chunks）。每个数据文件都被分割成固定大小的数据块，并分配一个唯一的块标识符（chunk handle）。这些块标识符构成了块的命名空间，用于识别和访问数据块。
- 块的命名空间记录了每个数据块的标识符以及其属性信息，例如大小、版本号等。主节点负责维护块的命名空间，并根据客户端的请求来处理数据块的创建、删除等操作。

**文件的命名空间**

- 文件的命名空间用于管理文件和目录的层次结构。它包括文件和目录的逻辑组织关系，以及文件和目录的名称和路径等信息。文件的命名空间描述了文件系统中文件和目录之间的层次关系和逻辑结构。

- 文件的命名空间记录了每个文件和目录的名称、路径和属性信息，例如创建时间、修改时间等。主节点负责维护文件的命名空间，并处理文件和目录的创建、删除、重命名等操作。

In a nutshell：

        块的命名空间由块句柄组成，对块的操作和管理都在块的命名空间上进行；

        文件的命名空间描述的是文件和目录的层级结构、逻辑联系，对文件的操作和管理是在文件的命名空间上进行。

#### Prefix Compression - 前缀压缩

Prefix compression是一种压缩技术，通常应用于文件系统或数据库等存储系统中，用于减少存储元数据的空间消耗。GFS文件命名空间中就使用了该技术存储文件名。

在文件系统中，文件和目录的名称通常具有一定的层次结构和重复性。例如，许多文件和目录可能具有相同的前缀，例如路径中的文件夹名称。使用前缀压缩技术，系统可以检测并利用这些重复的前缀，将它们存储为一个共享的前缀，并将剩余部分存储为相对于共享前缀的偏移量。这样可以显著减少存储元数据所需的空间。

#### 文件到块的映射

- 文件到块的映射连通了两大命名空间，记录了每个文件与其包含的数据块之间的关系。在GFS中，每个文件由一个或多个数据块组成，而文件系统需要记录每个文件包含哪些数据块，以及这些数据块的顺序和位置。
- 这种映射关系由主节点维护，并且用于在客户端访问文件时将文件名解析为对应的数据块句柄，以便客户端能够根据文件名找到文件的实际数据块，从而进行读取或写入操作。





### chunk vs replica

> 一个文件会划分成chunk存储，而每个chunk通常会有三个replica副本

这句描述中，前半句的chunk实际上只是一个逻辑概念，是说文件在逻辑上是分块存储的；后半句的replica才是块的物理副本，是说一个逻辑块在物理上会分为三个副本存储。

**NOTE：**

        同一chunk的不同replica的chunk handle（块句柄）是一样的，因为块句柄的唯一标识性是针对逻辑块而言的，唯一标识的是文件划分成的chunk。而物理实现上一个chunk保有三个副本，master存储的元数据中就有chunk的replicas的位置信息。



## 系统交互

### lease机制

在计算机科学中，"lease"（租约）是一种控制资源访问的机制，通常用于分布式系统中。租约机制允许一个实体（通常是客户端）在一段时间内拥有对某个资源的独占访问权或控制权。这个实体在拥有租约期间负责管理和使用该资源，而其他实体则不能同时修改或访问该资源。

在分布式系统中，租约通常用于控制对共享资源的访问，以确保资源的一致性、可靠性和可用性。租约通常包括以下几个关键属性：

1. **拥有者（Owner）**：租约的拥有者是被授予对资源进行访问或控制的实体。这个实体负责管理和使用资源，并在租约期间保持对资源的独占控制。

2. **期限（Duration）**：租约的期限指的是租约的有效时间，即拥有者能够拥有对资源的访问或控制权的时间段。一旦租约过期，资源将被释放并重新分配给其他实体。

3. **续约（Renewal）**：在租约期限内，拥有者可以选择续约租约，以延长对资源的访问或控制权。续约通常需要定期执行，以确保资源不被其他实体篡夺。

在GFS中，当client像master请求mutation操作时，master需要授权lease给一个chunk，这个chunk被称作primary chunk。primary chunk会决定这一系列的mutation操作的执行顺序，并且将其转发给其他的replica，从而使得对某个chunk的mutation操作在全局的一致性。而master控制全局所有mutation操作的顺序，是通过授权lease的顺序和每个primary chunk选定的mutation order。





# Questions

1. 客户端读写文件时询问master的具体流程？master存储的三大元数据分别起了什么作用？