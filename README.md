# 简易FAT16文件系统

## 项目简介

这是一个基于内存模拟的简易FAT16文件系统实现，使用C语言编写。该系统模拟了真实文件系统的核心功能，包括文件和目录的创建、删除、读写等操作，并通过交互式命令行界面提供用户友好的操作体验。

### 主要特性

- **基于内存的文件系统**：使用字符数组模拟磁盘存储
- **FAT表管理**：采用FAT（文件分配表）机制管理磁盘块的分配
- **目录树结构**：支持层级目录结构，可创建子目录
- **文件描述符机制**：支持多文件同时打开和读写操作
- **交互式命令行**：提供类似Unix shell的操作界面

### 技术规格

| 参数 | 值 |
|------|-----|
| 磁盘大小 | 2560 字节 |
| 块大小 | 64 字节 |
| 总块数 | 40 块 |
| 最大目录项 | 32 个 |
| 最大打开文件数 | 16 个 |
| 文件描述符范围 | 0-15 |

## 项目结构

```
FAT-16/
├── fat16/                      # 主要实现版本
│   ├── include/
│   │   └── fat.h              # 头文件，定义数据结构和函数接口
│   ├── src/
│   │   ├── main.c             # 主程序，命令行交互界面
│   │   ├── init.c             # 文件系统初始化
│   │   ├── dir.c              # 目录操作实现
│   │   ├── file.c             # 文件操作实现
│   │   └── fd.c               # 文件描述符管理
│   ├── obj/                   # 编译生成的目标文件
│   ├── bin/                   # 可执行文件
│   └── makefile               # 编译配置文件
├── MultiProcess/              # 多进程版本
│   └── (结构同fat16/)
└── README.md
```

## 系统架构

### 核心数据结构

- **Disk[]**: 模拟磁盘的字符数组
- **FAT[]**: 文件分配表，管理磁盘块的分配状态
- **Directory[]**: 目录项数组，存储文件和目录信息
- **FDTable[]**: 文件描述符表，管理打开的文件

### 目录项结构

```c
typedef struct {
    char name[16];      // 文件/目录名
    int size;           // 文件大小
    size_t start_block; // 起始块号
    int ac;             // 使用状态：0=未使用, 1=已使用
    int type;           // 类型：0=文件, 1=目录
    int parent;         // 父目录索引
} DirEntry;
```

### 文件描述符结构

```c
typedef struct {
    int used;           // 使用状态：0=空闲, 1=已使用
    int dir_index;      // 对应的目录项索引
    int offset;         // 当前读写偏移
    int mode;           // 打开模式
} FileDescriptor;
```

## 编译与运行

### 环境要求

- GCC 编译器
- Linux/Unix 系统（或支持GCC的环境）
- Make 工具

### 编译步骤

1. 进入项目目录：
```bash
cd /home/FAT-16/fat16
```

2. 编译项目：
```bash
make
```

编译成功后，可执行文件将生成在 `bin/app`。

3. 清理编译文件（可选）：
```bash
make clean
```

### 运行程序

```bash
./bin/app
```

启动后将进入交互式命令行界面，提示符格式为：
```
SimpleFAT:/path/to/current/dir$
```

## 使用指南

### 命令列表

| 命令 | 格式 | 说明 |
|------|------|------|
| `help` | `help` | 显示所有可用命令 |
| `ls` | `ls` | 列出当前目录内容 |
| `mkdir` | `mkdir <dirname>` | 创建新目录 |
| `cd` | `cd <dirname>` | 切换目录（支持 `..` 返回上级，`/` 返回根目录） |
| `touch` | `touch <filename>` | 创建新文件 |
| `rm` | `rm <filename>` | 删除文件 |
| `cat` | `cat <filename>` | 查看文件内容 |
| `open` | `open <filename> <mode>` | 打开文件，mode可选：`r`（读）、`w`（写）、`rw`（读写） |
| `close` | `close <fd>` | 关闭文件描述符 |
| `write` | `write <fd> <content>` | 向文件写入内容 |
| `read` | `read <fd> [size]` | 从文件读取内容（size可选，不指定则读取全部） |
| `seek` | `seek <fd> <offset>` | 设置文件读写偏移量 |
| `cls` | `cls` | 清屏 |
| `exit` | `exit` | 退出程序 |

### 使用示例

#### 1. 基本目录操作
```bash
# 查看当前目录
SimpleFAT:/$ ls

# 创建目录
SimpleFAT:/$ mkdir documents
SimpleFAT:/$ mkdir images

# 进入目录
SimpleFAT:/$ cd documents
SimpleFAT:/documents$

# 返回上级目录
SimpleFAT:/documents$ cd ..
SimpleFAT:/$
```

#### 2. 文件操作
```bash
# 创建文件
SimpleFAT:/$ touch readme.txt
SimpleFAT:/$ touch data.txt

# 查看目录内容
SimpleFAT:/$ ls
FILE    readme.txt    0 bytes
FILE    data.txt      0 bytes
DIR     documents     0 bytes
DIR     images        0 bytes
```

#### 3. 文件读写
```bash
# 以写模式打开文件
SimpleFAT:/$ open readme.txt w
opened fd = 0

# 写入内容
SimpleFAT:/$ write 0 "Hello, FAT16 File System!"

# 关闭文件
SimpleFAT:/$ close 0

# 查看文件内容
SimpleFAT:/$ cat readme.txt
Hello, FAT16 File System!
```

#### 4. 高级文件操作
```bash
# 以读写模式打开文件
SimpleFAT:/$ open data.txt rw
opened fd = 1

# 写入多行数据
SimpleFAT:/$ write 1 "Line 1: First data"
SimpleFAT:/$ write 1 "Line 2: Second data"

# 移动文件指针
SimpleFAT:/$ seek 1 0

# 读取指定字节数
SimpleFAT:/$ read 1 10
Line 1: Fi

# 读取剩余内容
SimpleFAT:/$ read 1
rst dataLine 2: Second data

# 关闭文件
SimpleFAT:/$ close 1
```

#### 5. 删除文件
```bash
# 删除文件
SimpleFAT:/$ rm oldfile.txt
File deleted successfully : oldfile.txt
```

## 实现细节

### FAT表机制

系统使用FAT表来管理磁盘块的分配：
- `FAT[i] = 0`: 块i未分配
- `FAT[i] = -1`: 块i是文件的最后一个块
- `FAT[i] = k`: 块i的下一个块是k

### 文件分配策略

1. **创建文件时**：在Directory数组中找到空闲项，分配目录项
2. **写入数据时**：
   - 检查文件是否有起始块，没有则分配
   - 根据写入大小计算所需块数
   - 在FAT表中查找空闲块并分配
   - 将数据写入对应的磁盘块
3. **删除文件时**：释放文件占用的所有磁盘块，在FAT表中标记为空闲

### 目录遍历

- 使用 `parent` 字段维护目录树结构
- 根目录的索引为0，parent为-1
- 支持递归计算目录大小

## 注意事项

1. **文件名长度限制**：最大15个字符
2. **磁盘空间有限**：总容量2560字节，请注意存储空间
3. **同时打开文件数**：最多16个文件
4. **文件描述符**：使用后记得关闭，释放资源
5. **读写模式**：以读模式打开的文件不能写入，反之亦然
6. **数据持久化**：当前实现为内存模拟，程序退出后数据不保存

## 多进程版本

项目中还包含一个 `MultiProcess/` 目录，这是该文件系统的多进程版本实现。编译和运行方式与主版本相同：

```bash
cd MultiProcess
make
./bin/app
```

## 许可证

本项目为教学演示项目，可自由学习和使用。

## 作者

简易FAT16文件系统开发团队

---

你可以将上述内容复制到 `README.md` 文件中。这个README包含了项目的完整介绍、技术规格、编译运行教程、使用指南和实现细节，应该能够帮助用户快速理解和使用这个文件系统。
        
