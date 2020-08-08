## 学习细节

* 依赖文件，依次寻找以下文件

  GNUmakefile > makefile > Makefile(我们通常命名)

  可以使用-f指定

* 注释 #， 多行\， 反斜杠后不能有空格

* Makefile中把那些没有任何依赖只有执行动作的目标称为伪目标；通过.PHONY将目标声明为伪目标，防止磁盘上存在相同名称的文件，怎样会导致错误；

* 在命令之前使用"-"，意思是忽略命令的执行错误

* 回显， 把@放在命令之前，就不会显示命令了， make -n会打印出命令，而不会执行

* 命令包，就是代码块

  ```
  define xxx
  	echo "--->"
  endef
  ```

* 变量的引用 ```$(variable_name), ${variable_name}  ```

* 赋值操作

  ```
  # 递归展开式变量
  foo = $(bar)
  bar = "xxx"
  echo $(foo)
  
  # 直接展开式变量, 不能向后引用变量
  x := $(y)later
  y := "xxx"
  
  # ?= 操作符
  # 只有在变量没有赋值才可执行操作
  foo ?= bar
  
  ## 类似于
  ifeq ($(origin foo), undefined)
  	foo = bar
  endif
  
  # += 追加值
  ```

* 自动化变量

  ```
  $@ -- 目标文件
  $< -- 第一个依赖文件
  $^ -- 所有依赖文件
  $% -- 静态库文件
  $? -- 已经修改的文件
  ```

* ifeq/ifneq  else

  ```
  ifeq ($(DEBUG_SYMBOLS), TRUE) 
  	CFLAGS += -g -Wall -Werror -O0 
  else 
  	CFLAGS += -Wall -Werror -O2 
  endif 
  ```

* if

  ```
  $(if condition, then-part, else-part)
  ```

  

* for/foreach

  ```
  MODULES := $(ls ~)
  @for subdir in $(MODULES); \ 
  do $(MAKE) -C $(DIR)/$$subdir $@; \ 
  done 
  
  dirs := a b c d
  files := $(foreach dir, $(dirs), $(wildcard $(dir)/*))
  ```

* 通配符

  ```
  rm -f *.o
  
  objects = $(wildcard *.o)
  ```

* 函数的应用

  ```
  # wildcard
  # patsubst
  $(patsubst a, b, $(var))
  
  # subst
  comma :=,
  empty :=
  space := $(empty)$(empty)
  foo := a b c
  $(subst $(space), $(comma), $(foo))
  
  # findstring
  $(findstring a, a b c)
  
  # sort
  
  # filter/filter-out
  
  # dir/notdir
  
  # suffix/ addsuffix
  # addprefix
  
  # join
  
  # words
  
  # basename
  
  # origin  输出变量是如何定义的
  ```

* call

  ```
  reverse = $(2)$(1)
  foo = $(call reverse,a,b)
  ```

* error

  ```
  ERR = $(error found an error!)
  
  err: ;$(ERR)
  ```

  

* VPATH依赖搜索路径

  ```
  VPATH = src:./headers
  VPATH = %.c
  ```

* 替换引用

  ```
  ${var:a=b} # 将var字符以a结尾替换成以b结尾
  
  $(patsubst a, b, $(var))
  ```

* 定义多行变量

  ```
  define two-lines
  echo foo
  echo $(bar)
  endif
  
  # 使用override
  override define two-lines
  echo foo
  echo $(bar)
  endif
  ```

  



## 案例一

```
OS := $(shell uname | awk '{print tolower($$0)}')
MACHINE := $(shell uname -m)

DEV_ROCKS = "busted 2.0.0" "busted-htest 1.0.0" "luacheck 0.23.0" "lua-llthreads2 0.1.5" "http 0.3" "kong 2.0.0"
WIN_SCRIPTS = "bin/busted" "bin/kong"
BUSTED_ARGS ?= -v -o TAP
TEST_CMD ?= bin/busted $(BUSTED_ARGS)
SHELL_BASH ?= /bin/bash -c
test = "functional"
plugin = "test"

## -o TAP

ifeq ($(OS), darwin)
OPENSSL_DIR ?= /usr/local/opt/openssl
GRPCURL_OS ?= osx
else
OPENSSL_DIR ?= /usr
GRPCURL_OS ?= $(OS)
endif

.PHONY: install dependencies dev remove grpcurl \
	setup-ci setup-kong-build-tools \
	lint test test-integration test-plugins test-all \
	pdk-phase-check functional-tests \
	fix-windows \
	nightly-release release startplugin install-openresty clean

ROOT_DIR:=$(shell dirname $(realpath $(lastword $(MAKEFILE_LIST))))
KONG_SOURCE_LOCATION ?= $(ROOT_DIR)
KONG_BUILD_TOOLS_LOCATION ?= $(KONG_SOURCE_LOCATION)/../kong-build-tools
RESTY_VERSION ?= `grep RESTY_VERSION $(KONG_SOURCE_LOCATION)/.requirements | awk -F"=" '{print $$2}'`
RESTY_LUAROCKS_VERSION ?= `grep RESTY_LUAROCKS_VERSION $(KONG_SOURCE_LOCATION)/.requirements | awk -F"=" '{print $$2}'`
RESTY_OPENSSL_VERSION ?= `grep RESTY_OPENSSL_VERSION $(KONG_SOURCE_LOCATION)/.requirements | awk -F"=" '{print $$2}'`
RESTY_PCRE_VERSION ?= `grep RESTY_PCRE_VERSION $(KONG_SOURCE_LOCATION)/.requirements | awk -F"=" '{print $$2}'`
KONG_BUILD_TOOLS ?= '4.3.2'
KONG_VERSION ?= `cat $(KONG_SOURCE_LOCATION)/kong-*.rockspec | grep tag | awk '{print $$3}' | sed 's/"//g'`
OPENRESTY_PATCHES_BRANCH ?= master
KONG_NGINX_MODULE_BRANCH ?= master

#setup-ci:
#	OPENRESTY=$(RESTY_VERSION) \
#	LUAROCKS=$(RESTY_LUAROCKS_VERSION) \
#	OPENSSL=$(RESTY_OPENSSL_VERSION) \
#	OPENRESTY_PATCHES_BRANCH=$(OPENRESTY_PATCHES_BRANCH) \
#	KONG_NGINX_MODULE_BRANCH=$(KONG_NGINX_MODULE_BRANCH) \
#	.ci/setup_env.sh

#setup-kong-build-tools:
#	-rm -rf $(KONG_BUILD_TOOLS_LOCATION)
#	-git clone https://github.com/Kong/kong-build-tools.git $(KONG_BUILD_TOOLS_LOCATION)
#	cd $(KONG_BUILD_TOOLS_LOCATION); \
#	git reset --hard $(KONG_BUILD_TOOLS); \
#
#functional-tests: setup-kong-build-tools
#	cd $(KONG_BUILD_TOOLS_LOCATION); \
#	$(MAKE) setup-build && \
#	$(MAKE) build-kong && \
#	$(MAKE) test

#nightly-release:
#	sed -i -e '/return string\.format/,/\"\")/c\return "$(KONG_VERSION)\"' kong/meta.lua
#	$(MAKE) release
#
#release:
#	cd $(KONG_BUILD_TOOLS_LOCATION); \
#	$(MAKE) package-kong && \
#	$(MAKE) release-kong

install:
	@luarocks make OPENSSL_DIR=$(OPENSSL_DIR) CRYPTO_DIR=$(OPENSSL_DIR)

remove:
	-@luarocks remove kong

dependencies:
	@for rock in $(DEV_ROCKS) ; do \
	  if luarocks list --porcelain $$rock | grep -q "installed" ; then \
	    echo $$rock already installed, skipping ; \
	  else \
	    echo $$rock not found, installing via luarocks... ; \
	    luarocks install $$rock OPENSSL_DIR=$(OPENSSL_DIR) CRYPTO_DIR=$(OPENSSL_DIR); \
	  fi \
	done;

#grpcurl:
#	@curl -s -S -L \
#		https://github.com/fullstorydev/grpcurl/releases/download/v1.3.0/grpcurl_1.3.0_$(GRPCURL_OS)_$(MACHINE).tar.gz | tar xz -C bin;
#	@rm bin/LICENSE

dev: remove install dependencies

lint:
	@luacheck -q .
	@!(grep -R -E -n -w '#only|#o' spec && echo "#only or #o tag detected") >&2
	@!(grep -R -E -n -- '---\s+ONLY' t && echo "--- ONLY block detected") >&2

test:
	@$(TEST_CMD) spec/01-unit

test-integration:
	@$(TEST_CMD) spec/02-integration

test-plugins:
	@$(TEST_CMD) spec/03-plugins

test-all:
	@$(TEST_CMD) spec/

test-custom:
	@$(TEST_CMD) spec/04-custom

test-self:
	@$(TEST_CMD) $(file)

test-transform:
	@$(TEST_CMD) spec/02-plugins/28-klook_request_transformer/02-access_spec.lua

install-openresty:
	chmod +x script/openresty_deploy.sh
	sh script/openresty_deploy.sh

startplugin:
	@mkdir -p kong/plugins/$(plugin)/tests
	@mkdir kong/plugins/$(plugin)/tests/functional
	@mkdir kong/plugins/$(plugin)/tests/unit
	@touch kong/plugins/$(plugin)/handler.lua
	@touch kong/plugins/$(plugin)/access.lua
	@touch kong/plugins/$(plugin)/schema.lua
	@touch kong/plugins/$(plugin)/tests/functional/$(plugin)_spec.lua
	@touch kong/plugins/$(plugin)/tests/unit/$(plugin)_spec.lua
	@echo "create plugin success."

clean:
	@$(SHELL_BASH) 'echo -e "\033[32m -- clear logs complete. -- \033[0m"'
	@$(SHELL_BASH) 'echo "" > logs/error.log && echo "" > logs/access.log'

%:
	@export KONG_TEST_ENV=dev && $(TEST_CMD) kong/plugins/$@/tests/$(test)

```

