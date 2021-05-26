---
title: 經 ssh 登入伺服器且不用輸入密碼
date: 2021-05-11 22:00:00
categories: 程式
tags: 
- Linux
- Ubuntu
- ssh
---

由於這幾天需要將一些例行動作轉為自動化流程，於是就面臨了「不想在程式碼中直接顯示密碼，可是自動化流程中需要以個人帳號登入伺服器」的問題。

雖然在開始作業前，就收到提示「可以使用 ssh key」，但身為一個菜鳥工程師，即使聽過「公鑰」跟「私鑰」這兩個專有名詞，卻搞不清楚具體上要如何進行作業啊！

<!-- more -->

##### 簡介
這是經由 ssh 免輸入密碼就登入 Linux 伺服器的方法。

簡要的說，就是生成一對公鑰 (public key) 與私鑰 (private key)，然後將公鑰從用戶端傳送到伺服器端，讓伺服器認識這個用戶，未來此用戶不用輸入密碼就可以登入該伺服器。

##### 如何產生鑰匙對
使用指令 `ssh-keygen`。接著，系統會詢問兩個選擇性設定，但也可以都直接 Enter 略過。

第一個設定是決定這對鑰匙放在哪個資料夾以及鑰匙名稱，預設資料夾會是 /home/user/.ssh/；私鑰名稱是 id_rsa；公鑰名稱則是 id_rsa.pub。準確來說，公鑰名稱固定是私鑰名稱 + .pub。

第二個設定則是 passphrase，根據指令 `man ssh-keygen` 顯示的手冊所寫：
- passphrase 類似於 password，但可以容納更多字元。
- 較佳的 passphrase 應該包含 10-30 個字元。
其他就跟密碼的設定一樣老生常談，
- 應該設定複雜又不容易被猜到的句子。
- 應該設定為大寫字母、小寫字母、數字以及特殊符號的組合。

但有一個重點，**passphrase 一旦遺失就無法復原**！
如果真的不幸遺失或忘記，手冊表示只能重新產生一對新鑰匙，並且需要重新複製到其他機器上……還是好好保管比較實在。

此外，如果以前產生過鑰匙又沒有設定名稱，導致新產生的檔案重名時，系統會詢問是否要覆寫 (overwrite)，建議先將舊的備份後再輸入 y，或者就直接改個名稱吧！

爬文時，有些人建議在產生鑰匙前，先自行建立資料夾 ~/.ssh 並且設定權限為僅檔案擁有者可讀、可寫、可執行，分別就是 `mkdir -p ~/.ssh` 和 `chmod 700 ~/.ssh` 這兩個指令。不過我沒事先做這些動作也順利完成，可能在出現什麼不可預期的錯誤時，可以往這方面下手看看吧！

以下是全部設定都直接 Enter 略過的紀錄：
```
    chihyu@chihyu-demo-ubuntu:~$ ssh-keygen 
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/chihyu/.ssh/id_rsa): 
    Enter passphrase (empty for no passphrase): 
    Enter same passphrase again: 
    Your identification has been saved in /home/chihyu/.ssh/id_rsa.
    Your public key has been saved in /home/chihyu/.ssh/id_rsa.pub.
    The key fingerprint is:
    SHA256:6UKY++YY+SKbRKGCPss3G8U2D/ie+f1+1E0p5S9ueIM chihyu@chihyu-demo-ubuntu
    The key's randomart image is:
    +---[RSA 2048]----+
    |                 |
    |               . |
    |  .           o .|
    |.. .o+   .   . o.|
    |+ ..**+ S     o.o|
    |o. .+O+.     .  o|
    | oo.o.+..   .+ . |
    |o.o*o.o..   E.=  |
    | o+..+.. .oo.o . |
    +----[SHA256]-----+ 
```
##### 複製公鑰
產生鑰匙對後，使用指令 `ssh-copy-id [-i [identity_file]] [user@]hostname` 將公鑰複製到伺服器端，會要求輸入伺服器端的密碼，接著系統就會建議使用者試著經由 ssh 登入看看。
```
    chihyu@chihyu-demo-ubuntu:~$ ssh-copy-id -i ~/.ssh/id_rsa.pub chihyu@172.24.1.111
    /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "id_rsa.pub"
    /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
    /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
    chihyu@172.24.1.111's password: 
    
    Number of key(s) added: 1
    
    Now try logging into the machine, with:   "ssh 'chihyu@172.24.1.111'"
    and check to make sure that only the key(s) you wanted were added.
    
```
##### 操作被拒絕怎麼辦
這時候我遇上問題，伺服器還是不認識我的電腦，依然跟我要密碼……
```
    chihyu@chihyu-demo-ubuntu:~$ ssh chihyu@172.24.1.111
    sign_and_send_pubkey: signing failed: agent refused operation
    chihyu@172.24.1.111's password:
```
直接把錯誤訊息丟去 google 就找到[這篇文章][1]中使用 `eval "$(ssh-agent -s)"` 再 `ssh-add` 的解法。
```
    chihyu@chihyu-demo-ubuntu:~/.ssh$ eval "$(ssh-agent -s)"
    Agent pid 14394
    chihyu@chihyu-demo-ubuntu:~/.ssh$ ssh-add
    Identity added: /home/chihyu/.ssh/id_rsa (/home/chihyu/.ssh/id_rsa)
    chihyu@chihyu-demo-ubuntu:~/.ssh$ ssh chihyu@172.24.1.111
    Last login: Tue May 11 11:27:03 2021 from 172.24.124.2
    chihyu@172.24.1.111:~$ exit
    logout
    Connection to 172.24.1.111 closed.
    chihyu@chihyu-demo-ubuntu:~/.ssh$ 
```
剛下完這兩個指令確實能免密碼登入伺服器，然而，一旦開啟新終端機就又跳出相同的錯誤訊息。重新讀一次前面的參考資料，以為是伺服器端也有叫做 id_rsa 的鑰匙對所導致，因此將原本的 id_rsa.pub 改名為 ubuntu_id_rsa.pub 後，再次 `ssh-copy-id -i ~/.ssh/ubuntu_id_rsa.pub chihyu@172.24.1.111` 複製過去，結果還是出現一樣的問題。

幸好，在[這篇文章的留言處][2]發現檔案權限也需要注意，依建議將伺服器端的 ~/.ssh/authorized_keys 權限設定為 744 後，就沒有再發生錯誤啦！
```
    chihyu@172.24.1.111:~/.ssh$ chmod 744 authorized_keys
    chihyu@172.24.1.111:~/.ssh$ ll
    total 24
    drwx------  2 chihyu chihyu 4096 May 11 14:16 .
    drwxr-xr-x 12 chihyu chihyu 4096 May 11 14:16 ..
    -rwxr--r--  1 chihyu chihyu 1155 May 11 14:16 authorized_keys
    -rw-------  1 chihyu chihyu 3247 Dec 25 17:14 id_rsa
    -rw-------  1 chihyu chihyu  752 Dec 25 17:14 id_rsa.pub
    -rw-------  1 chihyu chihyu 1110 May 10 14:48 known_hosts
```
##### 完成記得測試
輸入 `ssh chihyu@172.24.1.111` 可以直接從用戶端連接到伺服器，而不用輸入密碼，就是設定成功了！聽說使用這樣的鑰匙對連線會比使用密碼還安全，但我只是為了省去打密碼的功夫，哈哈！


[1]: https://it001.pixnet.net/blog/post/345636593-it%E4%BA%8B%E4%BB%B6%E7%B0%BF-%E4%B8%8D%E7%94%A8%E5%AF%86%E7%A2%BC%E7%99%BB%E5%85%A5%E9%81%A0%E7%AB%AF%E4%BC%BA%E6%9C%8D%E5%99%A8 "IT事件簿-不用密碼登入遠端伺服器"
[2]: https://blog.gtwang.org/linux/linux-ssh-public-key-authentication/ "SSH 公開金鑰認證：不用打密碼登入 Linux 設定教學，安全又方便"
