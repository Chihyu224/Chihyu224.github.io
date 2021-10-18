---
title: 建構 OpenWrt 映像檔可能出現的錯誤訊息
date: 2021-10-18 22:17:01
updated: 2021-10-18 22:17:01
categories: 程式
tags:
- OpenWrt
- Linux
- Ubuntu
---

這幾天開始接觸 OpenWrt。雖說網路上資源豐富，但我卻卡在不會下載檔案。繞了一圈路才發現，其實我需要的是原始碼，只要使用 `git clone` 就可以得到一份完整檔案了啊！完全不明白自己花了好幾個小時下載、解壓縮 OpenWrt 官方網站上的映像檔（image）是想要做什麼，人生總是無意間就被浪費了呢！

<!-- more -->

## 從下載開始

首先，移動到準備放置 OpenWrt 原始碼的資料夾，接著使用指令 `git clone https://git.openwrt.org/openwrt/openwrt.git`，等待克隆的時間長短取決於網速快慢。

當然，電腦首先要安裝好 Git，不然會跳出錯誤訊息。之前在 Windows 10 架設這個部落格時，我就犯過蠢，誤以為安裝 GitHub Desktop 等同安裝 Git，鬼打牆了老半天。

克隆完成會得到一個叫 openwrt 的資料夾。可以在該資料夾內使用 `git log` 查看修改紀錄，或者使用 `git tag` 確認有哪些標籤，也能使用 `git branch -a` 查看本地與遠端的所有分支。以現狀來說，最新版本的 OpenWrt 21.02.0 發行不到兩個月，因而我使用 `git checkout v19.07.8` 切換回前一個版本的標籤處再建構。

## 快速入門

依據跟著原始碼一起被下載的 README 即可迅速地建構 OpenWrt。另外，似乎在 21.02.0 開始將 README 改為 README.md，內容也寫得更詳細。但基本上是以下四個步驟，依序使用就能成功建構映像檔。

1. `./scripts/feeds update -a`
2. `./scripts/feeds install -a`
3. `make menuconfig`
4. `make`

其中，第三步可以調整設定值，讓建構出的映像檔符合自身需求。當然也可以直接修改資料夾中的 .config，只是 `make menuconfig` 可使其以圖形化界面顯示，更易於理解各項設定值。

## 缺乏相依套件

當進行到建構第三步驟，使用 `make menuconfig` 時，跳出了下列錯誤訊息：

```
Build dependency: Please install ncurses. (Missing libncurses.so or ncurses.h)
Build dependency: Please install GNU 'awk'
```

原因是使用中的主機缺乏某些依賴元件。如果作業系統跟我一樣使用 Ubuntu，那麼只需要使用 `sudo apt-get install libncurses5-dev gawk` 這行指令。libncurses5-dev 對應 ncurses；gawk 則是對應 awk，安裝完成就能順利執行。

如果使用其他作業系統，請參照 [OpenWrt 官方網站的對照表][1]，安裝應當預先準備的套件。

## 缺乏建構 VDI 所需套件

接著在進行第四步驟 `make` 又顯示 Error……這次在錯誤訊息中，可以看到建議：

```
Please re-run make with -j1 V=s or V=sc for a higher verbosity level to see what's going on
```

遵循建議，增加引數，使用 `make -j1 V=s` 再次編譯，得到非常詳盡的輸出。老實說，一時之間我根本看不出問題出在哪裡，後來又執行了一次 `make -j1 V=s > log.txt`，接著在 log.txt 之中搜尋 ERROR 或者 WARNING 等詞彙，才找到關鍵：

```
WARNING: Install qemu-img to create VDI/VMDK images
```

看來這次的問題來自，我想建構 VDI 映像檔（Virtual Desktop Infrastructure image），卻未安裝必需套件 QEMU。搜尋後得知，Ubuntu 系統使用 `sudo apt-get install qemu` 即可解決。

如果不是要建構給 VirtualBox 或者 VMware 的映像檔，應該就不需要這個套件。

## 建構成果

最後，如果一切順利，沒有再顯示任何錯誤訊息，將能在 ~/openwrt/bin/targets/ 資料夾中找到建構出的映像檔。

由於我的建構目標是：

- x86 64 位元的架構
- 檔案系統使用 ext4（Fourth extended filesystem，第四代擴充套件檔案系統）
- 預計安裝在 VirtualBox

所以完整的映像檔路徑為 ~/openwrt/bin/targets/x86/64/openwrt-x86-64-combined-ext4.vdi，後續則依使用情境決定如何運用建構好的映像檔。

## 額外說明

### 關於 make

可以加上引數，形成 `make -j24`，讓更多 CPU 核心被用來建構映像檔，多執行續執行以節省時間。基本上，`-j` 後面的數字越大越好，當超出實際可使用的 CPU 核心數量時，會自動以可使用的最大值運作。

### 關於 x86 與 x64

在選擇目標系統為 x86 之後，還需要選擇 32 位元或者是 64 位元。這一點讓我剛開始有點困惑，想說 Windows 之中顯示 x86 不就限定於 32 位元了嗎？與 x86 相對的不是 x64 嗎？

所幸看到[這篇文章][2]中的解釋：

> x86其實是指過去處理器的通稱，Intel以前的處理器大多以86為其產品編號的末兩碼——8086、80286、386、486……。即使到了Pentium時代，大家私底下還是用586、686這樣的說法。

總之，就我的理解，32 位元處理器曾經稱霸過一段時間，導致 x86 一詞已泛指 32 位元處理器。但實質意義上，x86 指稱的是**基於 Intel 8086 的處理器指令集架構**，所以還需要選擇位元。

真正與 x86 相對的，應該會是其他的目標系統架構，比如說：Broadcom BCM47xx/53xx (ARM)、Lantiq、MediaTek Ralink ARM、NVIDIA Tegra、Qualcomm Atheros IPQ806X 等等，但這些可能要接觸嵌入式系統（embedded system）才有機會使用到吧。

至於為何只有 32 位元與 64 位元兩個選項？因為 OpenWrt 這個專案目前就只有支援這兩種。我猜想是由於更大或更小的使用者都太少了。

### 關於檔案系統

此外，Linux 陣營較常使用的檔案系統，除了 ext4 還有 squashfs。根據[維基百科 SquashFS 條目][3]，是一套唯讀的壓縮檔案系統，適用於較缺乏系統資源的電腦。

另一方面，微軟開發的檔案系統有 NTFS（New Technology File System）與 FAT32（File Allocation Table）。後者尾綴的 32 是指 32 位元，目前於 USB 與較舊的微軟作業系統中可見。

[1]: https://openwrt.org/docs/guide-developer/build-system/install-buildsystem "[OpenWrt Wiki] Build System Setup"
[2]: https://www.ithome.com.tw/node/56880 "iT自救術─x86和x64到底有什麼差異？"
[3]: https://zh.wikipedia.org/wiki/SquashFS "SquashFS - 維基百科"