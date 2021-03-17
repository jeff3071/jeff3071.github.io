---
title: '[Python]資料存進Firebase'
date: 2021-02-05 20:53:11
tags: [Python, 爬蟲, 棒球分析, 中職, Firebase]
categories: 資料視覺化
---

## Firebase

這邊只會用到Firebase的Cloud Firestore功能，[官方文件](https://firebase.google.com/docs/firestore?authuser=0)。

<!--more-->

之後搭建網站將會使用Flask，所以在這邊我們使用Python來串接，創建好自己的應用後，取得私鑰的json檔案，接者就可以連接到Firebase了。

```Pythone
import firebase_admin
from firebase_admin import credentials
from firebase_admin import firestore

# Use a service account
cred = credentials.Certificate('path/to/serviceAccount.json')
firebase_admin.initialize_app(cred)

db = firestore.client()
```

Firebase是No SQL，我們需要創建collection和document，這邊我將打者設為一個collection，每位選手為一個document，底下有選手的各個資料。


```Python
def store_month(data):
    batch = db.batch()

    for player in data:
        doc_ref = db.collection(u'打者').document(player['Name'])
        del player['Name']
        batch.update(doc_ref, {u'月份':player})
    batch.commit()
```

這段程式碼我使用了batch，batch的好處在於只需要用一個request就能寫入多筆資料(單次上限為500)，寫入時使用update則是因為有些球員的資料是不需要更新的。

![](https://i.imgur.com/bFtSS8v.png)
> 儲存成功的樣子

接著讓爬蟲呼叫這個function就可以寫入了。
