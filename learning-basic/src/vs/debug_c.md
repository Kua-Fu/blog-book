# MacOS avscode debug C/C++ with makefile

## 一、参考


> [Setup Visual Studio Code for Multi-File C++ Projects](https://dev.to/talhabalaj/setup-visual-studio-code-for-multi-file-c-projects-1jpi)

## 二、依赖环境

### 2.1 vscode 插件

![c](https://github.com/Kua-Fu/blog-book-images/blob/main/vscode/c_plgin.png?raw=true)

### 2.2 makefile

项目结构如下

```shell


➜  clox git:(main) ✗ tree .
.
├── LICENSE
├── Makefile
├── README.md
├── bin
│   └── clox
├── obj
│   ├── chunk.o
│   ├── main.o
│   └── memory.o
└── src
    ├── chunk.c
    ├── chunk.h
    ├── common.h
    ├── main.c
    ├── memory.c
    └── memory.h

3 directories, 13 files

```

Makefile如下

```makefile


CC = gcc
CFLAGS = -Wall -Wextra -g

SRC_DIR = src
OBJ_DIR = obj
BIN_DIR = bin

CLOX = $(BIN_DIR)/clox
SRC_FILES = $(wildcard $(SRC_DIR)/*.c)
OBJ_FILES = $(patsubst $(SRC_DIR)/%.c,$(OBJ_DIR)/%.o,$(SRC_FILES))

clox: $(OBJ_FILES)
	mkdir -p $(BIN_DIR) $(OBJ_DIR)
	$(CC) $(CFLAGS) -o $(CLOX) $(OBJ_FILES)

$(OBJ_DIR)/%.o: $(SRC_DIR)/%.c
	mkdir -p $(OBJ_DIR)
	$(CC) $(CFLAGS) -c $< -o $@

.PHONY: clean
clean:
	rm -rf $(BIN_DIR)
	rm -rf $(OBJ_DIR)
	rm -rf $(SRC_DIR)/*.o $(SRC_DIR)/clox


```


## 三、debug

### 3.1 创建task

![c_task](https://github.com/Kua-Fu/blog-book-images/blob/main/vscode/c_task.png?raw=true)

```task.json

// .vscode/task.json

{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build",
            "type": "shell",
            "command": "make",
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}

```

⚠️ 注意:

	* label 值为 build，表示该task的名称，唯一标识

	* type 值为 shell，表示终端命令
	
	* command 值为 make，表示执行make
	
	* 可以添加一个 "problemMatcher": "$gcc" 表示如何查看make执行过程
	


### 3.2 创建launch

![launch](https://github.com/Kua-Fu/blog-book-images/blob/main/vscode/c_launch.png?raw=true)

```

// .vscode/lanuch.json

{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "preLaunchTask": "build",
            "program": "${workspaceFolder}/bin/clox",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "lldb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}

```

⚠️ 注意:

	* preLaunchtask 表示预先执行的命令，即我们之前创建的task， build
	
	* program 是运行的命令，此处为 ./bin/clox
	
	* MIMode 在MacOS 为 lldb(linux 是 gdb)


### 3.3 运行

![debug](https://github.com/Kua-Fu/blog-book-images/blob/main/vscode/c_run_debug.png?raw=true)
