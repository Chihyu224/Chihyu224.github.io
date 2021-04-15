---
title: 介紹 Jenkins Scripted Pipeline
date: 2021-06-09 20:00:00
updated: 2021-06-09 20:00:00
categories: 程式
tags:
- Jenkins
- Pipeline
- scripted pipeline
---

前陣子使用 Jenkins 幫助公司自動化建置程式，也稍微接觸了 CI/CD，畢竟 Jenkins 就是一款 CI/CD 工具。

此處的 CI 是指持續整合（Continuous Integration），而 CD 則是持續交付或持續部屬（Continuous Delivery / Continuous Deployment）。

但這篇文章僅介紹**如何撰寫 Jenkins Pipeline script**。

<!-- more -->

## 前言
Pipeline 有兩種完全不同的語法，分別是 Declarative Pipeline 與 Scripted Pipeline。雖然兩種都是寫給 Jenkins 的 Pipeline，但是無法混在一起使用，否則執行時會有各種錯誤。

不得不提，[官方文件][]寫得很實用，只是 scripted 語法預設隱藏，必須點擊連結 **Toggle Scripted Pipeline** 才會顯示，這點讓我查資料時吃了些苦頭。提醒一下，以下的所有程式碼都是 **Scripted Pipeline**，且不打算介紹另一種語法。

## 基本架構

以下是從官方文件節錄出來的[基本架構][]，想要更詳細的說明可以直接進連結，另外也有簡體中文版本的說明可以參考。個人推薦，若能力許可，直接讀英文文件較佳，比較不會產生誤解或者前後名詞對不上的狀況。

```
node {
    stage('Build') {
        // This is a comment.
    }
    stage('Test') {
        // Do something here...
    }
    stage('Deploy') {
        // Just write some steps!
    }
}
```

雖說非強制性，但官方建議將所有 Pipeline 工作內容都放在 **node**（節點）裡面。實際建置時，也是由節點分配執行器與工作區給 Pipeline，一旦移除節點區塊，Pipeline 就無法做任何事情，所以終究是必須加進去的區塊。

**stage**（階段）也是選擇性的，如果恰當切分階段區塊，那麼在 Jenkins 使用者界面會相當清楚地展現每個階段的運作結果、花費時間與其餘詳細資訊。多次建置後，Jenkins 還會計算每個階段平均建置時間，有助於瞭解整體趨勢。

在 **Scripted Pipeline** 中，不需要 stage**s** 區塊或者是 steps 區塊，直接在階段區塊中撰寫欲執行的步驟即可。

## 步驟

### echo

跟在 Linux 相同用法，`echo` 顯示的文字都會被保存在 Jenkins 的建置日誌檔中，我幾乎都用來確認該階段的狀態 `echo "Stage Build result: $currentBuild.result"`。

其中 `$currentBuild.result` 是 Jenkins 原生變數，預設值是 null。呼叫前需要自行先設定 `currentBuild.result = 'SUCCESS'`、`currentBuild.result = 'FAILURE'` 或者 `currentBuild.result = 'UNSTABLE'`，分別會顯示綠燈、紅燈或黃燈。

### sh

在 Unix/Linux 系統使用終端機，如果要在 Windows 系統下使用，可以替換為 `bat`。能將各種指令包進去，簡單的用法好比說 `sh 'cd;mkdir ~/ExDir;touch ~/ExDir/ExFile.log'`，也能加上其他選項，例如 `returnStdout: true` 可以將終端機的標準輸出作為回傳值另作他用。

```
def FILE_TAIL = null
node{
    stage('Get the tail of log file') {
        FILE_TAIL = sh(script: 'tail ~/ExDir/ExFile.log', returnStdout: true).trim()
        echo "The tail of log file is ${FILE_TAIL}"
    }
}
```

以上範例中，`def FILE_TAIL = null` 這個變數 `FILE_TAIL` 也可以定義在節點區塊或者階段區塊內，就只是全域變數和區域變數的差異。需要注意的是，一旦為 `sh` 增加選項，就必須將預設可忽略的 `script` 選項清楚標示，且選項之間要以逗號區隔。如果覺得選項太長、不易閱讀而想要分行，`sh` 括號內接受換行。如果覺得字串太長，可以使用 `+` 連結。

其實這個範例有點脫褲字放屁，因為不特別使用 `returnStdout: true` 時，`sh` 的標準輸出就會自動被寫入建置日誌檔，現在卻特地擷取 `sh` 的標準輸出，再交給 `echo` 寫入建置日誌檔……沒關係，後面會提到條件控制，就能對變數 `FILE_TAIL` 做些特別處理了！可參閱官方文件獲得[更多 sh 的介紹][]。

另外，單引號 `''` 與雙引號 `""` 內的變數 `${FILE_TAIL}` 會有[不同的結果][]，需要多加留意。

## 語法

在官方文件的[語法說明][]中，提到 **Scripted Pipeline** 才能使用的兩種語法：**條件控制**與**異常處理**。也就是其他程式語言中常見的 `if…else…` 以及 `try…catch…finally…`，後者似乎只有部份語言支援。

### 異常處理

雖然官方文件將異常處理區塊放在階段區塊之中，但我個人在處理**流程出現任何錯誤就取消後續所有動作**時，比較喜歡拉到節點區塊之外，使用一個 `try` 包住所有可能會出錯的動作，如下：

```
try {
    node {
        stage('Example') {
            echo "This is an example."
        }
    }
} catch(Exception err) {
    throw err
}
```

因為在 `try` 區塊中遇到任何錯誤，都會直接跳到 `catch` 區塊，再也不回頭，所以要根據哪些是互相關聯的動作，事先規劃要使用幾個 `try`、各自範圍要多大。

此外還有 `finally` 區塊可以使用，適合放一些無論成功或失敗都必須執行的動作。

有時候，發生某些系統容許，可是基於種種原因而必須中止流程的動作時，能使用 `error 'This is the error message!'` 回傳自定義的錯誤訊息。

### 條件控制

到處都看得到的條件控制，一旦符合 `()` 小括號中的條件，就執行 `{}` 大括號中的動作，否則就繼續確認下一個條件直到結束。最常見的是 `if` 與 `else` 的搭配，就是簡單明瞭的二分法。也能使用 `else if` 增加多個條件，讓動作更分歧，或者只使用 `if` 針對某條件增加動作。

條件可以用 `&&` 表示**兩者皆須成立**，以及 `||` 表示**其中一者成立**，更加細膩地避開各種意料外的現象。以下是個簡單的例子：

```
node {
    stage('Example') {
        if(currentBuild.result == null || currentBuild.result == 'SUCCESS') {
            echo 'OK'
        } else if(currentBuild.result == 'UNSTABLE') {
            echo 'So-so'
        } else {
            echo 'NG'
        }
    }
}
```

## 定義新函式

除了使用 `def` 定義新變數，也可以用來定義新函式，這邊直接放範例，裡面比較特別的只有 `slackSend(channel:'#backup', color: colorCode, message: msg)` 將 `message` 指定的字串作為訊息經 Slack 傳進 `channel` 指定的群組中，並以 `color` 指定的顏色強調該則訊息。

不過要使用 `slackSend` 之前，記得要先在 Jenkins 安裝好 [Slack Plugin][]！當然也要事先下載 Slack，創建指定的群組。另外還有 email 跟 HipChat 的通知套件可以使用，但我沒有實作過，就不野人獻曝了。

```
def costomizedSlackSend(String result = 'UNKNOWN') {
    // Default values
    def colorCode = '#FF0000'
    def message = "${result}: ${env.JOB_NAME} #${env.BUILD_NUMBER} (${env.BUILD_URL})"

    // Override default color code based on result
    if (result == 'SUCCESS') {
        colorCode = '#00FF00'
    } else if (result == 'UNSTABLE') {
        colorCode = '#FFFF00'
    } else if (result == 'UNKNOWN') {
        colorCode = '#C0C0C0'
    }

    // Send notification to Slack
    slackSend(channel:'#backup', color: colorCode, message: message)
}
```

## 總結

為了整理這篇文章，感覺又查了不少資料，對於 Scripted Pipeline 也更熟悉了些。希望下次使用時，不會太惶恐，也不要再發生「我好像有看過這個東西，但又想不起來要怎麼用」的悲劇。

## 附錄

```
def FILE_TAIL = null
try {
    currentBuild.result = 'SUCCESS'

    node {
        stage('Build') {
            echo "This is an example for Build stage"
            sh 'cd; mkdir ~/ExDir;' +
               'touch ~/ExDir/ExFile-1.log;' +
               'touch ~/ExDir/ExFile-2.log'
            echo "Build $currentBuild.result"
        }
        stage('Test-1') {
            FILE_TAIL = sh(script: 'tail ~/ExDir/ExFile-1.log', returnStdout: true).trim()
            if(FILE_TAIL.contains("success")) {
                FILE_TAIL = null
            } else {
                error 'Test-1 ERROR!'
            }
            echo "Test-1 $currentBuild.result"
        }
        stage('Test-2') {
            FILE_TAIL = sh(script: 'tail ~/ExDir/ExFile-2.log', returnStdout: true).trim()
            if(FILE_TAIL.contains("success")) {
                FILE_TAIL = null
            } else {
                error 'Test-2 ERROR!'
            }
            echo "Test-2 $currentBuild.result"
        }
        stage('Deploy') {
            echo "This is an example for Deploy stage"
            costomizedSlackSend(currentBuild.result)
            echo "Deploy $currentBuild.result"
        }
    }
} catch(Exception err) {
    currentBuild.result = 'FAILURE'
    costomizedSlackSend(currentBuild.result)
    throw err
}
```

[官方文件]: https://www.jenkins.io/doc/book/pipeline/ "Jenkins Pipeline"
[基本架構]: https://www.jenkins.io/doc/book/pipeline/#scripted-pipeline-fundamentals "Scripted Pipeline fundamentals"
[更多 sh 的介紹]: https://www.jenkins.io/doc/pipeline/steps/workflow-durable-task-step/#sh-shell-script "sh Shell Script"
[不同的結果]: https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#string-interpolation "String interpolation"
[語法說明]: https://www.jenkins.io/doc/book/pipeline/syntax/#scripted-pipeline "Scripted Pipeline Syntax"
[Slack Plugin]: https://plugins.jenkins.io/slack/ "Slack Notification"