---
title: 實用細節
date: 2021-05-04 22:00:00
updated: 2021-05-04 22:00:00
categories: 程式
tags: 
- Hexo
- Markdown
---

重看了前一篇文章的參考連結，突然發現一些無意間忽略但很實用的細節！

## 關於 Hexo
比如說，在產生靜態檔之前，可以使用 `$ hexo clean` 清除舊的靜態檔。
難怪之前修改完成也產生新的靜態檔，卻總是看到修改前的畫面⋯⋯
<!-- more -->

另外有些可以組合的指令

    $ hexo d -g    ## hexo generate + hexo deploy
    $ hexo s -g    ## hexo generate + hexo server

雖然 `-g` 看起來在後面，但其實是先執行的動作。

## 關於 Markdown
如果沒有特別設定，部落格的文章內容將全文顯示於首頁。但我覺得這樣有點太冗長，畫面也不甚美觀。才發現有個**閱讀更多**的指令可以用！

    <!-- more -->

在適當的地方插入，就能讓首頁保持僅預覽部份文章的簡潔感。

另外，我還在[Markdown 語法說明][]學到另一種插入連結的方式！
在前一篇文章中，我插入了連續四個連結，使用的是行內形式的連結語法：

    [link text](http://example.com/ "Title")

    [[教學] 使用 GitHub Pages + Hexo 來架設個人部落格](https://ed521.github.io/2019/07/hexo-install/ "[教學] 使用 GitHub Pages + Hexo 來架設個人部落格")
    [如何搭建個人 Blog 使用 Hexo + Gitpage](https://medium.com/@bebebobohaha/%E4%BD%BF%E7%94%A8-hexo-gitpage-%E6%90%AD%E5%BB%BA%E5%80%8B%E4%BA%BA-blog-5c6ed52f23db "如何搭建個人 Blog 使用 Hexo + Gitpage")
    [個人技術站一把罩！部落格建置大全（一）- 使用 Hexo 搭配 Github Page 建置自己的部落格](https://www.muji.dev/2020/02/16/hexo-github-page/ "個人技術站一把罩！部落格建置大全（一）- 使用 Hexo 搭配 Github Page 建置自己的部落格")
    [Git+Hexo-架設個人靜態網站](https://hackmd.io/@king87515/Sy16ckymU "Git+Hexo-架設個人靜態網站")

但其實，也可以使用參考形式的連結語法：

    [link text][id]
    [id]: http://example.com/ "Title"

    [[教學] 使用 GitHub Pages + Hexo 來架設個人部落格] [1]
    [如何搭建個人 Blog 使用 Hexo + Gitpage] [2]
    [個人技術站一把罩！部落格建置大全（一）- 使用 Hexo 搭配 Github Page 建置自己的部落格] [3]
    [Git+Hexo-架設個人靜態網站] [4]

    [1]: https://ed521.github.io/2019/07/hexo-install/
        "[教學] 使用 GitHub Pages + Hexo 來架設個人部落格"
    [2]: https://medium.com/@bebebobohaha/%E4%BD%BF%E7%94%A8-hexo-gitpage-%E6%90%AD%E5%BB%BA%E5%80%8B%E4%BA%BA-blog-5c6ed52f23db
        "如何搭建個人 Blog 使用 Hexo + Gitpage"
    [3]: https://www.muji.dev/2020/02/16/hexo-github-page/
        "個人技術站一把罩！部落格建置大全（一）- 使用 Hexo 搭配 Github Page 建置自己的部落格"
    [4]: https://hackmd.io/@king87515/Sy16ckymU "Git+Hexo-架設個人靜態網站"

`[id]` 前是否有空白並不影響語法。
相較起行內形式，參考形式的原始檔讀起來輕鬆多了。尤其是連結被放在文字中間時，參考形式避免連結網址中斷思緒，可以更舒服地閱讀，也會因為連結被集中於同一處，而更容易修改。甚至於還可以省略標數字的部分：

    [link text][]
    [link text]: http://example.com/ "Title"

    [[教學] 使用 GitHub Pages + Hexo 來架設個人部落格][]
    [如何搭建個人 Blog 使用 Hexo + Gitpage][]
    [個人技術站一把罩！部落格建置大全（一）- 使用 Hexo 搭配 Github Page 建置自己的部落格][]
    [Git+Hexo-架設個人靜態網站][]

    [[教學] 使用 GitHub Pages + Hexo 來架設個人部落格]: https://ed521.github.io/2019/07/hexo-install/ "[教學] 使用 GitHub Pages + Hexo 來架設個人部落格"
    [如何搭建個人 Blog 使用 Hexo + Gitpage]: https://medium.com/@bebebobohaha/%E4%BD%BF%E7%94%A8-hexo-gitpage-%E6%90%AD%E5%BB%BA%E5%80%8B%E4%BA%BA-blog-5c6ed52f23db "如何搭建個人 Blog 使用 Hexo + Gitpage"
    [個人技術站一把罩！部落格建置大全（一）- 使用 Hexo 搭配 Github Page 建置自己的部落格]: https://www.muji.dev/2020/02/16/hexo-github-page/ "個人技術站一把罩！部落格建置大全（一）- 使用 Hexo 搭配 Github Page 建置自己的部落格"
    [Git+Hexo-架設個人靜態網站]: https://hackmd.io/@king87515/Sy16ckymU "Git+Hexo-架設個人靜態網站"

就算 \[\] 之間包含繁體漢字也毫無問題，真的頗實用！不過有些特殊符號似乎需要跳脫字元（反斜線）\\ 協助顯示⋯⋯這部份就下次再研究吧！


[Markdown 語法說明]: https://markdown.tw/ "Markdown 語法說明"
