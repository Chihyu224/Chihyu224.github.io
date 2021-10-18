---
title: 簡介結構
date: 2021-09-29 20:23:20
updated: 2021-09-29 20:23:20
categories: 程式
tags:
  - C
  - programming language
  - struct
  - 結構
---

前陣子做了個新功能，第一次用 C 語言實作一個結構（structure），所幸時間還算充裕，讓我前後修改三、四次，才終於定案並完工。這篇文章記錄著我陸陸續續修改的心路歷程。

<!-- more -->

先舉個例子，現在有幾座固定只能裝 8 本書的書架，而我想記錄每座書架上已經放了幾本書、每本書的書名與國際標準書號（ISBN, ）。書名可以使用字符指標的形式（char *）儲存，但是國際標準書號必須用字符矩陣的形式（char []）儲存。因為之後要對國際標準書號的尾碼做些風騷的操作……呃，我是指，要去驗證最後一碼的確認碼計算正確。

當然還有更多值得記錄的資料，但這個例子只是想要重現**儲存指標的矩陣**與**雙重矩陣**被包含在同一個結構中，可以讓初學者多麼崩潰，也可能只有我很崩潰啦……

## 定義基本的 struct

先依照目標定義一個結構，由於書本的數量一定會是整數，所以定義為 integer，其餘就如前文所述。另外，突然冒出來的 13 是現行的 ISBN 長度，以前是固定只有 10 碼啦……

```
struct Bookshelf {
	int count;
	char isbn[8][13];
	char *title[8];
};

/* Declare by
	struct Bookshelf bs1; */
```

## 使用 typedef 簡化聲明

每次聲明都要在前面打 `struct` 搞得我有點煩，馬上就給 `Bookshelf` 一個別名吧！順便也把寫死的那兩個數字獨立定義，之後要修改就只需要在最前面的 `define` 區塊變更。

```
#define MAX_BOOKS_CNT 8
#define ISBN_LEN 13

typedef struct Bookshelf {
	int count;
	char isbn[MAX_BOOKS_CNT][ISBN_LEN];
	char *title[MAX_BOOKS_CNT];
} BOOKSHELF;

/* Declare by
	BOOKSHELF bs1; */
```

最後完工的結構大概就長這個樣子，但其實它曾經長得超級複雜。最初我得到的指示要求**盡量減少記憶體的使用量，最好使用 malloc 動態分配記憶體空間**，讓我爬資料爬了一週才實作出來。結果成品被評為太過複雜，操作上容易出錯，未來維護不易。重新討論後才發現需要的空間其實沒多大，就修改方向為固定大小的結構了。

## 取用結構中的成員

在 C 語言之中，有兩個運算子（[operator][]）可以取得結構中的資料。有趣的是，我找不到其中一個運算子的正式稱呼，連英文都沒有。這邊就採取英文俗名的直譯來介紹：

- `.`  點運算子（dot operator）
- `->` 箭頭運算子（arrow operator）

### 點運算子作用於結構

先從簡單的點運算子開始舉例，當要在上述結構中儲存資料時，可以這麼做：

```
bs1.title[0] = "Example 1";
strncpy(bs1.isbn[0], "9789861234567", ISBN_LEN);
bs1.count = 1;
```

接著，再使用點運算子取出結構中的資料，並打印於畫面上：

```
printf("count=%d\n", bs1.count);
printf("ISBN=%c%c%c-%c%c%c-%c%c%c%c-%c%c-%c\n", bs1.isbn[0][0], bs1.isbn[0][1], bs1.isbn[0][2], bs1.isbn[0][3], bs1.isbn[0][4], bs1.isbn[0][5], bs1.isbn[0][6], bs1.isbn[0][7], bs1.isbn[0][8], bs1.isbn[0][9], bs1.isbn[0][10], bs1.isbn[0][11], bs1.isbn[0][12]);
printf("title=%s\n", bs1.title[0]);
```

### 如何處理結構指標

為了方便起見，可以把「儲存資料至結構中」獨立為一個函數：

```
void addBook(BOOKSHELF *bs, char *isbn, char *title)
{
	strncpy((*bs).isbn[(*bs).count], isbn, ISBN_LEN);
	(*bs).title[(*bs).count] = title;
	(*bs).count++;
}
```

以 C 語言來說，函數的引數在此處必須是指標，不然會有自以為自己寫進去的錯覺。剛從 C# 轉過來時，我常常被取值與取址搞得一個頭兩個大，雖有看到資料說 C# 之中分為 by value 與 by reference 兩種呼叫，但是當年的我根本還沒用到如此高深的 C#……總而言之，要讓資料確實寫入引數中，必須使用指標。

這時，由於點運算子（.）的優先度**高於**取值的一元運算子（*），導致需要添加很多的括號才能完成上述函數，否則 `*bs.count` 隱含的是 `*(bs.count)`，這不是我們想要的結果。

### 為簡化而生的箭頭運算子

雖然不清楚淵源，但我想，箭頭運算子（->）是為了簡化這個繁複的過程而誕生。

讓**「先對結構指標取值，再對該結構取用結構成員」**這兩個步驟的表達，由 `(*bs).count` 簡化為 `bs->count`，使程式碼變得更精簡、更易讀。以此運算子改寫上述函數，會得到：

```
void addBook(BOOKSHELF *bs, char *isbn, char *title)
{
	strncpy(bs->isbn[bs->count], isbn, ISBN_LEN);
	bs->title[bs->count] = title;
	bs->count++;
}
```

這樣是不是變得簡單明瞭了呢？

## 初始化結構成員

最後，在寫這篇文章的範例時，我發現剛被聲明的結構，內容基本上是亂碼。在儲存資料前，不小心取用到結構成員，往往會發生悲劇。如果不是太在意效能的話，可以在聲明時，一併將結構成員初始化為 0 或者依情境直接指定第一筆資料：

```
/* Initialize a struct to 0. */
BOOKSHELF bs1 = {};

/* Initialize a struct with partial data. */
BOOKSHELF bs1 = {
	.count = 0
};

/* Another example. */
BOOKSHELF bs1 = {
	.title[0] = "建構嵌入式LINUX系統",
	.isbn[0][0] = '9',
	.isbn[0][1] = '7',
	.isbn[0][2] = '8',
	.isbn[0][3] = '9',
	.isbn[0][4] = '8',
	.isbn[0][5] = '6',
	.isbn[0][6] = '7',
	.isbn[0][7] = '7',
	.isbn[0][8] = '9',
	.isbn[0][9] = '4',
	.isbn[0][10] = '3',
	.isbn[0][11] = '0',
	.isbn[0][12] = '7',
	.count = 1
};
```

這樣就不用擔心，聲明結構後立即取用，會不會出現各種亂碼啦！

## 一個可編譯的範例

以下附上書寫本文時的測試範例。通常我都會使用 `gcc -Wall <file name>`，以便確認所有的警告訊息。

```
#include <stdio.h>
#include <string.h>

#define MAX_BOOKS_CNT 8
#define ISBN_LEN 13

typedef struct Bookshelf {
	int count;
	char isbn[MAX_BOOKS_CNT][ISBN_LEN];
	char *title[MAX_BOOKS_CNT];
} BOOKSHELF;

int checkIsbn(char *in)
{
	return ('0' + 10 - ((in[0] + in[2] + in[4] + in[6] + in[8] + in[10] + (in[1] + in[3] + in[5] + in[7] + in[9] + in[11]) * 3 - '0' * 24) % 10)) == in[12] ? 0 : 1;
}

void addBook(BOOKSHELF *bs, char *isbn, char *title)
{
	printf("Try to add a book. ");
	printf("Its ISBN is '%s', and its title is '%s'.\n", isbn, title);

	if (bs->count >= MAX_BOOKS_CNT)
	{       
		printf("[Error] This bookshelf is full. ");
		printf("You cannot add this book '%s'!\n\n", title);
	}
	else if (checkIsbn(isbn) != 0)
	{       
		printf("[Error] Its ISBN is illegal. ");
		printf("You cannot add this book '%s'!\n\n", title);
	}
	else /* (bs->count < MAX_BOOKS_CNT) && (checkIsbn(isbn) == 0) */
	{       
		strncpy(bs->isbn[bs->count], isbn, ISBN_LEN);
		bs->title[bs->count] = title;
		bs->count++;
	}
}

void printBookshelf(BOOKSHELF bs)
{
	int idx;

	printf("Information:\n");
	printf("\tcount= %d\n", bs.count);
	printf("\tISBN | title=\n");
	for (idx = 0; idx < bs.count; idx++)
	{   
		printf("\t\t%d. %c%c%c-%c%c%c-%c%c%c%c-%c%c-%c | %s\n",
			(idx + 1), bs.isbn[idx][0], bs.isbn[idx][1],
			bs.isbn[idx][2], bs.isbn[idx][3], bs.isbn[idx][4],
			bs.isbn[idx][5], bs.isbn[idx][6], bs.isbn[idx][7],
			bs.isbn[idx][8], bs.isbn[idx][9], bs.isbn[idx][10],
			bs.isbn[idx][11], bs.isbn[idx][12], bs.title[idx]);
	}   
	printf("End\n\n");
}

int main()
{
	BOOKSHELF bs1 = {};

	printf("Before add anything...\n");
	printBookshelf(bs1);

	addBook(&bs1, "9789867794307", "建構嵌入式LINUX系統");
	addBook(&bs1, "9789862222225", "找不到第二本書了");
	addBook(&bs1, "9789863333333", "就只是舉個例子");
	addBook(&bs1, "9789864444441", "I am another example");
	addBook(&bs1, "9789865555559", "Do You Have a Nice Day");
	addBook(&bs1, "9789866666667", "Let's Try Again");
	addBook(&bs1, "9789867777775", "舉例七");
	addBook(&bs1, "9789868888883", "舉例八");
	addBook(&bs1, "9789869999990", "舉例九，妳大概看不到我");
	addBook(&bs1, "9789869999991", "舉例十，妳應該看不到我");

	printf("After add books...\n");
	printBookshelf(bs1);

	return 0;
}
```

以下則是執行結果：

```
chihyu@chihyu-demo-ubuntu:~/exercise/struct-exercise$ ./a.out
Before add anything...
Information:
	count= 0
	ISBN | title=
End

Try to add a book. Its ISBN is '9789867794307', and its title is '建構嵌入式LINUX系統'.
Try to add a book. Its ISBN is '9789862222225', and its title is '找不到第二本書了'.
Try to add a book. Its ISBN is '9789863333333', and its title is '就只是舉個例子'.
Try to add a book. Its ISBN is '9789864444441', and its title is 'I am another example'.
Try to add a book. Its ISBN is '9789865555559', and its title is 'Do You Have a Nice Day'.
Try to add a book. Its ISBN is '9789866666667', and its title is 'Let's Try Again'.
Try to add a book. Its ISBN is '9789867777775', and its title is '舉例七'.
Try to add a book. Its ISBN is '9789868888883', and its title is '舉例八'.
Try to add a book. Its ISBN is '9789869999990', and its title is '舉例九，妳大概看不到我'.
[Error] This bookshelf is full. You cannot add this book '舉例九，妳大概看不到我'!

Try to add a book. Its ISBN is '9789869999991', and its title is '舉例十，妳應該看不到我'.
[Error] This bookshelf is full. You cannot add this book '舉例十，妳應該看不到我'!

After add books...
Information:
	count= 8
	ISBN | title=
		1. 978-986-7794-30-7 | 建構嵌入式LINUX系統
		2. 978-986-2222-22-5 | 找不到第二本書了
		3. 978-986-3333-33-3 | 就只是舉個例子
		4. 978-986-4444-44-1 | I am another example
		5. 978-986-5555-55-9 | Do You Have a Nice Day
		6. 978-986-6666-66-7 | Let's Try Again
		7. 978-986-7777-77-5 | 舉例七
		8. 978-986-8888-88-3 | 舉例八
End

chihyu@chihyu-demo-ubuntu:~/exercise/struct-exercise$ 
```

[operator]: https://terms.naer.edu.tw/detail/15050969/ "取自國家教育研究院學術名詞翻譯，亦可譯作運算符"