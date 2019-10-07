---
title: 使用Docker 運行 Ubuntu
date: 2019-09-23 13:59:00
tags: 
- docker
- Ubuntu
intro: ''

---
<!--more-->

```
環境：macOS
```

## 建立Container
`$ docker run --name MyUbuntu -dt ubuntu`

`docker run`會由image建立contaniner

參數定義

- `--name`: 命名container，上面那行指令我們將container命名成`MyUbuntu`，若省略，會隨機命名
- `-d`：建立 container 後，就脫離目前 process
- `-t`：預設執行 /bin/bash process，為了讓 container 啟動後不會立即停止
- ubuntu：官方image名字

**一個container一個process**

而process執行完container會自動釋放
因此加上`-t`使container執行`bash`，讓container不會自動釋放

## 進入Ubuntu

`$ docker exec -it MyUbuntu bash`

`docker exec`對執行中的container下指令

- i：interactive，可對 terminal 輸入資料
- t：terminal，可對 terminal 顯示資料
- MyUbuntu : container 名稱
- bash : 對 container 下的指令
