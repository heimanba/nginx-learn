# 概览
命令行主要是两个场景: alias和自定义脚本


## Makefile

### .DEFAULT_GOAL
用于告知用户在命令行中未指定目标时，应构建哪个目标。
```
bar:
	@echo "build"

foo:
	@echo "foo"
```
默认执行`make`指令的时候，默认执行的是第一个指令也就是`make bar`

如果在开头加上`.DEFAULT_GOAL := foo`，则会先执行`make foo`

### .PHONY
`.PHONY`后面的target表示一个伪造的`target`,而不是真实存在的文件`target`。(`Makefile`的target默认是文件)
当我们有下面的`Makefile`命令
```
clean:
	rm -rf clean
```
执行命令`make clean`时会报错 `make: 'clean' is up to date`这是`makefile target`和`dir`名字冲突造成的。
这时候可以借用`.PHONY`
```
.PHONY: clean

clean:
	rm -rf clean
```

### Self-Documented Makefile
当我们执行`make`指令的时候，希望能够索引出来所有指令的描述信息
```
SHELL := bash
.DEFAULT_GOAL := help
.PHONY: build clean help

build: ## build the project
	@echo "build"

clean: ## clean the project
	@echo ${NICK}

help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'
```
