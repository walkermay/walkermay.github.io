---
layout: post
title: MakeFile学习笔记
date: 2024-08-14
author: HaitaoMei
tags: [makefile, code]
comments: true
---

## 基本规则

目标：依赖条件

（TAB一下）命令



## 举例并细节学习

### 1. CC=/usr/bin/gcc

指定使用的编译器为 `/usr/bin/gcc`。

### 2. K3AE_PATH=../kyber_NIST_round3_with_attack_extensions/

如果我有多个文件夹，在主文件夹打开终端，则需要给另一个文件夹设立快捷名

### 3. CFLAGS += -march=native -fomit-frame-pointer -O3 -I$(K3AE_PATH)

**`CFLAGS`**：编译器标志，定义了编译过程中使用的优化选项和包含目录。

- `-march=native`：优化生成代码以匹配当前的处理器架构。
- `-fomit-frame-pointer`：不在生成的代码中保存帧指针，进一步优化内存使用。
- `-O3`：高级优化选项。
- `-I$(K3AE_PATH)`：指定包含目录，告诉编译器在 `K3AE_PATH` 目录中寻找头文件。

### 4. LDFLAGS=-lcrypto -O3

链接标志，告诉链接器链接 `libcrypto` 库，并使用 `-O3` 优化选项。

### 5. SOURCES= adaptive_parallel_singlethread.c \ 	 $(K3AE_PATH)cbd.c \

指定源文件包括哪些

### 6. HEADERS= adaptive_parallel_singlethread.h \$(K3AE_PATH)api.h \

指定头文件包含哪些

### 7. all：run test measure

该目标由三个子目标 `run`、`test` 和 `measure` 组成，表示执行 `make all` 时会依次构建 `run`、`test` 和 `measure`。

### 8. run: attack_kyber512 \ attack_kyber768 \ attack_kyber1024

这些目标分别依赖于不同的攻击程序，这些程序是针对不同参数集（Kyber 512、768 和 1024）的。

### 9. 生成可执行文件

#### 9.1 pair_parallel_attack_oracle: $(HEADERS) $(SOURCES) $(CC) $(CFLAGS) -o $@ $(SOURCES) $(LDFLAGS) -lm

**目标文件：** `pair_parallel_attack_oracle` 是最终生成的可执行文件的名称。

**依赖关系：** `$(HEADERS) $(SOURCES)` 表示 `pair_parallel_attack_oracle` 依赖于这些头文件和源文件。如果任何一个依赖文件发生变化，`Makefile` 将重新生成 `pair_parallel_attack_oracle`。

**命令：**`$(CC) $(CFLAGS) -o $@ $(SOURCES) $(LDFLAGS) -lm`

- `$(CC)`：使用之前定义的编译器。
- `$(CFLAGS)`：使用定义的编译标志。
- `-o $@`：`$@` 是自动变量，表示当前目标（即 `pair_parallel_attack_oracle`）。
- `$(SOURCES)`：使用所有源文件。
- `$(LDFLAGS)`：使用定义的链接标志。
- `-lm`：链接数学库（`libm`），用于处理数学函数

#### 9.2 attack_kyber512: $(HEADERS) $(SOURCES) BRT_kyber512.c run_attack.c | create_dir_run $(CC) $(CFLAGS) -DKYBER_K=2 -o run/$@ $(SOURCES) run_attack.c BRT_kyber512.c $(LDFLAGS)

**`BRT_kyber512.c` 和 `run_attack.c`**：这些是额外的源文件，它们特定于 `attack_kyber512` 目标。

**`| create_dir_run`**：这个竖线 `|` 表示 `create_dir_run` 是一个依赖目标，但它不会影响 `attack_kyber512` 目标的更新。`create_dir_run` 主要确保 `run` 目录已经创建。

**`-DKYBER_K=2`**：这是一个编译时定义的宏。在 C/C++ 代码中，`KYBER_K` 将被定义为 2，表示使用 Kyber 512 参数集。这通常用于控制代码中与参数集相关的条件编译。

**`-o run/$@`**：`-o` 表示输出文件的路径和名称。`$@` 是一个自动变量，代表当前目标名，即 `attack_kyber512`。由于 `run/` 是输出目录，因此生成的可执行文件路径为 `run/attack_kyber512`。

**`$(SOURCES) run_attack.c BRT_kyber512.c`**：这些是参与编译的源文件。`$(SOURCES)` 包含通用的源文件列表，而 `run_attack.c` 和 `BRT_kyber512.c` 是特定于这个目标的源文件。

### 10. 清理目标

.PHONY: clean 

clean: 

​	-rm run/attack_kyber512

**`.PHONY`**：声明 `clean` 为伪目标，表示 `clean` 不是一个真正的文件，而是一个命令目标。

**`clean` 规则：** `clean` 目标用于删除生成的可执行文件。`-rm` 命令会删除文件 `pair_parallel_attack_oracle`，`-` 表示忽略删除文件时可能出现的错误（例如文件不存在时）。

