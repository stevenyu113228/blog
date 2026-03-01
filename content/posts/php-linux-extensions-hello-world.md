---
title: PHP Linux Extensions Hello World
date: 2022-02-14T22:34:56+08:00
draft: false
url: "/2022/02/14/php-linux-extensions-hello-world/"
categories:
  - 資訊安全
  - 網頁安全
showToc: true
TocOpen: false
---

要做這個主要是碩論的研究跟 RASP 有一點點關係，而 PHP RASP 需要透過 PHP Extenstions 來進行編譯以及使用，所以本文主要會記錄一些基本的 PHP Extenstions 的撰寫以及開發方式。

本文會需要使用到的環境

- 任意 Linux 或 WSL 搭配 Docker
- Docker Ubuntu 20.04
- PHP 7.3.33 Source Code

#### 環境建置

在安裝好 Ddocer 後，理論上一鍵就可以建置好乾淨的 Ubuntu 20.04 docker，理論上-v 的部分可以自己修改掛載點

```bash
sudo docker run -it -v /home/steven/compile-rasp/mount_point:/mount_point ubuntu:20.04
```

接下來可以安裝一些之後可能會用到的小工具

```bash
apt update
apt install wget unzip software-properties-common -y
apt-add-repository ppa:ondrej/php
apt install php7.3-dev -y
```

途中 software-properties-common 的安裝過程可能會跳出一些東西，就照著回答就好了。

接下來下載 php 相關的程式碼

```bash
cd mount_point
wget https://www.php.net/distributions/php-7.3.33.tar.gz
tar zxvf php-7.3.33.tar.gz
chmod -R 777 * # 因為我在 Docker 外使用 Mount，這樣處理權限比較簡單
```

#### 建立第一個 Extension

要寫 Extenstion，第一件事情是需要先建立一個 Extension，在原始碼路徑中 `php-7.3.33/ext` 中，有一個名為 `ext_skel.php` 的檔案，skel 是 skeleton (骨架) 的縮寫，也就是讓我們建立一個軀殼。

```bash
cd php-7.3.33/ext
php ext_skel.php --ext meow
```

這邊的 `--ext meow` 就是指定要建立的 Extension 的名稱，以這邊為例就是 `meow`

接下來在 `php-7.3.33/ext` 底下就會出現一個 `meow` 資料夾，我們接下來的工作也就都在裡面進行。

再來，我們需要更動一點 config 的設定，這邊需要動到的檔案是 `config.m4`，這個檔案看起來很多行，但事實上 `dnl` 開頭的東西都是註解而已，我們需要把 第 8 以及第 10 行，也就是 下面這兩行給解除註解 (把 `dnl` 刪除)，這兩行主要的意思是採用動態編譯的方法，未來會採用 `.so` 的檔案進行連結。

```c
PHP_ARG_WITH(meow, for meow support,
[  --with-meow             Include meow support])
```

#### 撰寫 Hello Meow程式

在程式的 `php_meow.h` 中，我們需要定義一個我們的 Hello World Funcion，在該檔案的 `#endif  /* PHP_MEOW_H */` 前輸入以下指令即可，

```c
PHP_FUNCTION(meow_hellomeow);
```

接下來就需要修改 `meow.c` 檔案了，在程式的 81 行中間新增一行 `PHP_FE(meow_hello,meow, NULL)`，整個 function 會變成這樣

```c
static const zend_function_entry meow_functions[] = {
	PHP_FE(meow_test1,		arginfo_meow_test1)
	PHP_FE(meow_test2,		arginfo_meow_test2)
	PHP_FE(meow_hellomeow, NULL)
	PHP_FE_END
};
```

再在程式的最下面寫我們的程式碼

```c
PHP_FUNCTION(meow_hellomeow)
{
	char *arg = NULL;
	int arg_len, len;
	char *strg;
	if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "s", &arg, &arg_len) == FAILURE) {
		return;
	}
	php_printf("This is my first meow meow!\n");
	RETURN_TRUE;
}
```

輸入 `phpize` 準備編譯的環境，再來輸入 `./configure` 來準備 Makefile，最後輸入 `make && make install`

這樣我們就編譯完成！ `make install` 會幫我們把 `php-7.3.33/ext/meow/modules` 中的 `meow.so` 搬到 php extensions 的位置，以這邊為例的話會在 `/usr/lib/php/20180731/`

#### 測試

接下來我們可以透過 php 指令來引入 `meow.`so 並且執行看看

```bash
php -d "extension=meow.so" -r "meow_hellomeow(8787);"
```

這樣我們就順利的運行了我們自己的第一個 Plugin！

![](/uploads/2022/02/2022-02-14-22-33-24.png)

Reference : [从0开始的PHP RASP的学习](https://xz.aliyun.com/t/7316#toc-9)
