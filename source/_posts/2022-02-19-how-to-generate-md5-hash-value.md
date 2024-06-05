---
title: 如何產生 MD5 雜湊值
date: 2022-02-19 15:00:00
updated: 2022-02-19 15:00:00
categories: 程式
tags:
- Linux
- Ubuntu
- C
- programming language
---
最近需要在程式中計算字串的 MD5，因而搜尋了不少資料，像是如何在終端機產生，或是在 C 程式碼中添加它。

<!-- more -->

## 什麼是 MD5

MD5 是一種加密用的雜湊函數（hash function），可由一份檔案或者一段字串產生出 128 位元的雜湊值（hash value）。即使輸入值的差異極小，得到的雜湊值也會有顯著差異，因而被用來確認資料傳輸後的完整性。

比如說，在下載映像檔後，計算該檔案的 MD5，就可以跟「映像檔提供者計算出的 MD5」做比較，若兩者一致，則可視作檔案正常、無毀損。

現在已被證實，MD5 無法防止碰撞攻擊，故不應使用於安全性憑證。

## 在終端機的運用

以 ubuntu 16.04 LTS 來說，預載就包含 `md5sum` 這個指令，只要加上檔名，即可在終端機得到 MD5。

```sh
chihyu@chihyu-demo-ubuntu:~/project$ cat example_1.txt 
chihyu
chihyu@chihyu-demo-ubuntu:~/project$ md5sum example_1.txt 
249c8275bc209c09b2a8a8700d813590  example_1.txt
chihyu@chihyu-demo-ubuntu:~/project$ cat example_2.txt 
Chihyu
chihyu@chihyu-demo-ubuntu:~/project$ md5sum example_2.txt 
70f647210d6dabac410f268963c6e20f  example_2.txt
```

也可以直接加密字串，結尾有沒有**換行符號**會有顯著差異。

```sh
chihyu@chihyu-demo-ubuntu:~/project$ echo 'chihyu' | md5sum
249c8275bc209c09b2a8a8700d813590  -
chihyu@chihyu-demo-ubuntu:~/project$ echo -n 'chihyu' | md5sum
2c8b36be217ca4a5727c86a86f1f63ca  -
```

如果不想要行末顯示的檔名或者 - ，可以搭配 `cut` 指令使用。

```sh
chihyu@chihyu-demo-ubuntu:~/project$ echo -n 'chihyu' | md5sum | cut -d ' ' -f1
2c8b36be217ca4a5727c86a86f1f63ca
```

## 在 C 語言的運用

首先安裝必要的庫，才有函數可以呼叫，有些資料說要使用 `sudo apt-get install openssl` ，不過我使用的是這一個：

```
sudo apt-get install libssl-dev
```

接著，在 C 語言程式碼中引用 `#include <openssl/md5.h>` 就可以輕鬆產生 MD5。雖然我打算以 16 個 16 進位的字元來表示 MD5，但使用 openssl 函數返回的值會是 unsigned char array，尚須處理才會顯示成該格式。

以下是一段簡單的示範程式碼：

```sh
chihyu@chihyu-demo-ubuntu:~/project$ cat md5.c 
#include <stdio.h>
#include <string.h>
#include <openssl/md5.h>

int main(void)
{
	const char *str = "The quick brown fox jumps over the lazy dog";
	unsigned char digest[MD5_DIGEST_LENGTH];
	MD5_CTX context;

	MD5_Init(&context);
	MD5_Update(&context, str, strlen(str));
	MD5_Final(digest, &context);

	printf("str: '%s'\nmd5: '", str);
	for (int i = 0; i < MD5_DIGEST_LENGTH; i++) {
		printf("%02x", digest[i]);
	}
	printf("'\n");

	return 0;
}
chihyu@chihyu-demo-ubuntu:~/project$ gcc md5.c -lcrypto -o md5
chihyu@chihyu-demo-ubuntu:~/project$ ./md5 
str: 'The quick brown fox jumps over the lazy dog'
md5: '9e107d9d372bb6826bd81d3542a419d6'
```

需要特別注意的是，gcc 編譯時需要加上 `-lcrypto` 。

希望下次需要產生這個雜湊值時，這篇筆記可以讓自己迅速地解決。

## 額外技巧

使用 printf 搭配一些格式說明符（format specifier）可以改變顯示的格式，我個人較常使用的有：

- `%c` 顯示字元（character）。
- `%s` 顯示字串（string）。
- `%d` 顯示 10 進位制（decimal）的整數。
- `%x` 與 `%X` 顯示 16 進位制（hexadecimal）的整數。

16 進位制特別標注大小寫，是因為 10-15 會依據說明符，顯示為 a-f 或者 A-F。

也就是說，若將上方示範程式碼中 `printf("%02x", digest[i]);` 修改為 `printf("%02X", digest[i]);` ，結果也將變成 `md5: '9E107D9D372BB6826BD81D3542A419D6'` 。

至於 `%` 與 `x` 之間夾雜的 `02` 說明，無論該數值有幾位，都將以至少 2 個字元顯示，且以 0 填補空格。若未指定，則將顯示空白。另外，如果加上 `-` 將會靠左對齊。

```
format: "%05X"
md5: '0009E000100007D0009D000370002B000B6000820006B000D80001D0003500042000A400019000D6'

format: "%5X"
md5: '   9E   10   7D   9D   37   2B   B6   82   6B   D8   1D   35   42   A4   19   D6'

format: "%-5X"
md5: '9E   10   7D   9D   37   2B   B6   82   6B   D8   1D   35   42   A4   19   D6   '
```

還有一些比較進階，也比較少用的技巧，改天再整理出來。