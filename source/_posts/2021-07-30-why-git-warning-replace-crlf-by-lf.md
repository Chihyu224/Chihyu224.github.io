---
title: 為何 git 要警告 CRLF will be replaced by LF
categories: 程式
tags:
  - git
  - CRLF
  - LF
  - 換行字元
  - Linux
  - Ubuntu
  - Windows
date: 2021-07-30 00:32:48
updated: 2021-07-30 00:32:48
---


最近使用 git 的時候，跳出一個我第一次見到的警告訊息 `CRLF will be replaced by LF` 。雖然警告訊息應該不會有太嚴重的影響，但我還是很好奇發生了什麼事，所以拿去問了一下 google。這才知道，原來不同作業系統的換行字元有可能會不同啊！
<!-- more -->

## 事發經過：當 git add 時出現 warning
首先，收到的警告訊息大概長這樣：

```sh
chihyu@chihyu-demo-ubuntu:~/project/test$ git add .
warning: CRLF will be replaced by LF in foo.c.
The file will have its original line endings in your working directory.
```

直譯成中文大概就是，在 foo.c 這個檔案中的 CRLF 將要被 LF 取代。

然而，什麼是 CRLF 啊？LF 又是什麼東西？為什麼會突然出現這個警告訊息？取代掉會有什麼影響嗎？如果有影響的話，有辦法不讓 CRLF 被 LF 取代嗎？

## 換行字元大不同
原來，在電腦中，每一行的最後都有一個人眼看不見的**換行字元**告訴電腦「這一行結束囉！」這個換行字元在英文中，有很多種叫法，包含：newline、line ending、end of line (EOL)、line feed、line break，其中的 line feed 縮寫就是 LF。

然而，**不同的作業系統可能會用不同的換行字元**，比如說，Windows 使用的是 CRLF；Linux 使用的則是 LF，還有一些我沒接觸過的作業系統會使用 CR，甚至使用 LFCR 等等。

如果以 C 語言來說，常常會出現的 `\n` 就是 LF！而 CR 則以 `\r` 來表示。但我用 C 語言工作大約九個月，還真沒有看過 `\r` 這個東西……

之前寫 C# 時，則會使用 `Environment.NewLine` 或者是 `"\r\n"` 來換行。這樣想想，就是因為 Windows 使用 CR + LF，所以才會轉換為 `"\r" + "\n"` 嘛！印象中，使用 `Environment.NewLine` 還會自動偵測環境，去決定要使用哪一種換行字元。不過，會使用 C# 的環境應該都是 Windows 吧……

## 出現原因：作業系統不一樣
至於這個警告訊息的緣由，是這次新增到 git 的修改中，有些是同事傳給我的檔案，而他的電腦作業系統就是 Windows！所以，這就是個，部門成員的作業系統不同，而發生的意外小插曲。

有些人見到的警告訊息會跟我相反，呈現為 `LF will be replaced by CRLF` ，那就是源自於 Linux 傳至 Windows 導致的囉！

雖然[查到的資料][]指出，可以藉由 `git config core.autocrlf` 確認目前在 git 中的設定，但是我使用該指令毫無反應……後來是直接透過 `cat $HOME/.gitconfig` 查看設定檔，發現根本不包含這項設定。

```sh
chihyu@chihyu-demo-ubuntu:~/project/test$ cat $HOME/.gitconfig
[user]
	email = chihyu@x.com
	name = chihyu
[core]
[core]
```

## 變更換行字元可能帶來的影響
總之，git 會預設作業系統綁定某種換行字元，當使用者打算交給 git 控管的文件中，有著與該作業系統不相符的換行字元時，就會自動執行取代，並且顯示對應的警告訊息。

使用者可以特別使用 `git config core.autocrlf false` 去改變換行字元的設定，另外還有著兩種選擇：true 和 input。在 ubuntu 16.04 之中，我嘗試在這三種設定下，新增一個內含 CRLF 的檔案到 git，結果如下：

```sh
chihyu@chihyu-demo-ubuntu:~/project/test$ git config core.autocrlf
input
chihyu@chihyu-demo-ubuntu:~/project/test$ git add test-1.c
warning: CRLF will be replaced by LF in test-1.c.
The file will have its original line endings in your working directory.
chihyu@chihyu-demo-ubuntu:~/project/test$ git config core.autocrlf false
chihyu@chihyu-demo-ubuntu:~/project/test$ git config core.autocrlf
false
chihyu@chihyu-demo-ubuntu:~/project/test$ git add test-2.c
warning: CRLF will be replaced by LF in test-2.c.
The file will have its original line endings in your working directory.
chihyu@chihyu-demo-ubuntu:~/project/test$ git config core.autocrlf true
chihyu@chihyu-demo-ubuntu:~/project/test$ git config core.autocrlf
true
chihyu@chihyu-demo-ubuntu:~/project/test$ git add test-3.c
chihyu@chihyu-demo-ubuntu:~/project/test$ git config core.autocrlf xxx
chihyu@chihyu-demo-ubuntu:~/project/test$ git config core.autocrlf
xxx
chihyu@chihyu-demo-ubuntu:~/project/test$ git add test-4.c
fatal: bad numeric config value 'xxx' for 'core.autocrlf': invalid unit
```

儘管在 `git help config` 之中沒有看到 false 的選項，但確實可以將其設為 false，至少不會像隨手設定的 xxx 一樣顯示「無效單元」。

最後，我還是把該設定移除，恢復原本的樣子。畢竟，往後在我的工作電腦上，這些檔案終究會以 LF 作為換行字元。

真正重要的可能是，**shell script 必須要以 LF 結尾，不然會以非預期的方式執行**。

## 如果只是想關閉警告
此外，我發現有個[讓 git 繼續運作，但是不顯示警告的方法][]： `git config --global core.safecrlf false` 。

使用這個設定後，git 還是會做它預設要做的事情，只是它不會重複在畫面上警告使用者而已，就只是個讓人眼不見為淨的指令！


[查到的資料]: https://stackoverflow.com/questions/1967370/git-replacing-lf-with-crlf "Stack Overflow"
[讓 git 繼續運作，但是不顯示警告的方法]: https://stackoverflow.com/questions/5834014/lf-will-be-replaced-by-crlf-in-git-what-is-that-and-is-it-important "Stack Overflow"