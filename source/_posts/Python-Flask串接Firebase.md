---
title: '[Python]Flask串接Firebase'
date: 2021-02-08 19:24:56
tags: [Python, Firebase, 棒球分析, 中職]
categories: 資料視覺化
---

## Flask

首先用pip安裝Flask，接者創建一個`app.py`。

<!-- more -->

`app.py`

```python3
import flask
from flask import render_template, url_for, redirect, request, jsonify, Response #之後會用到的

app = flask.Flask(__name__)
app.config["DEBUG"] = True

@app.route('/', methods=['GET'])
def index():
    return render_template('index.html')
```

接著下`python app.py`就可以執行了，但是這邊還沒創建Html檔，所以會報錯，因為使用了render_template，需要再創建一個templates資料夾，並在資料夾內創建`index.html`

回到網頁，就會看到全白的網站了。

## Header

未來可能會用到許多頁面，而這些頁面通常會共用一個header，所以我這邊將它獨立為一個`header.html`。

`header.html`
```Html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.bootcss.com/jquery/3.0.0/jquery.min.js"></script>
    <title>Document</title>
</head>
    <body>
        <nav class="navbar navbar-expand-lg navbar-light bg-light">
            <a class="navbar-brand" href="/">中職球員數據</a>
            <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarNavAltMarkup"
                aria-controls="navbarNavAltMarkup" aria-expanded="false" aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNavAltMarkup">
                <div class="navbar-nav">
                    <a class="nav-item nav-link" href="/">首頁 <span class="sr-only">(current)</span></a>
                </div>
            </div>
        </nav>
    </body>
</html>
```

> 這邊我使用了bootstrap來進行排版


回到`index.html`，因為Flask支援jinja 2，讓我能直接引入header。 [官方文件](https://jinja.palletsprojects.com/en/2.11.x/)


`index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>數據可視化</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css"
        integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous">
    <script src="https://cdn.bootcss.com/jquery/3.0.0/jquery.min.js"></script>
    <script type="text/javascript" src="https://assets.pyecharts.org/assets/echarts.min.js"></script>
    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js"
        integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous">
    </script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js"
        integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous">
    </script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"
        integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous">
    </script>
</head>
<body>
    <div class="container">
        {% include 'header.html' %}
    </div>
<body>
<html>
```
> 上面引入的script都是之後會用的
> 

## 從Firebase取得資料

前面有介紹到如何連接Firebase，而連接的程式碼就放在`app.py`裡面

```python
import flask
from flask import render_template, url_for, redirect, request, jsonify, Response 

app = flask.Flask(__name__)
app.config["DEBUG"] = True

cred = credentials.Certificate('path/to/serviceAccount.json')
firebase_admin.initialize_app(cred)

...

if __name__ == '__main__':
    app.run()
```

為了不讓`app.py`太過雜亂，所以我把取資料的程式碼獨立為`db_connect.py`，並且連同之前儲存資料的程式碼一起放進來。

取得資料的方法非常簡單，指定好collection和document後，使用get()就能取得，取出來的資料可以用to_dict()轉成dict，讓Python好處理。

`db_connect.py`

```python
import firebase_admin
from firebase_admin import credentials
from firebase_admin import firestore

def get_hitter_stat(player_name):
    db = firestore.client()
    doc_ref = db.collection(u'打者').document(str(player_name))
    doc = doc_ref.get()
    if doc.exists:
        return doc.to_dict()
    else:
        return False
        
def store_month(data):
    db = firestore.client()
    batch = db.batch()
    for player in data:
        doc_ref = db.collection(u'打者').document(player['Name'])
        del player['Name']
        batch.update(doc_ref, {u'月份':player})
    batch.commit()
```

接著把`db_connect.py`import進`app.py`

## 前端設置

接著要先把`index.html`切出我要顯示出來的位置，首先選擇球員需要用兩個下拉式選單`<select>`，並用bootstrap稍微美化一下表單。

```html
<form>
    <div class="row">
        <div class="form-group col-6">
            <select name="team" id="team" class="form-control">
                <option value="中信兄弟">中信兄弟</option>
                <option value="統一獅">統一獅</option>
                <option value="富邦悍將">富邦悍將</option>
                <option value="樂天桃猿">樂天桃猿</option>
            </select>
        </div>
        <div class="form-group col-6">
            <select name="player" id="player" class="form-control">
            </select>
        </div>
     </div>
</form>
```

並且我想要將這兩個下拉式選單連動，也就是當我選擇中信兄弟時，player的選單會顯示出中信兄弟的球員，這邊我需要用到select的onchange觸發ajax讓Flask傳球員名單過來，先寫一個js function。

```javascript
function select_team(){
    let team = $('#team').val();
    $.ajax({
        type: 'POST',
        url:  "/select_team",
        data: {"team":team},
        success: function(result){
            $('#player').empty();
            $('#player').append($("<option/>", {
                value: '選擇球員',
                text: '選擇球員'
            }));
            for(let i = 0; i < result.length ;i++){
                $('#player').append($("<option/>",{
                    value: result[i],
                    text: result[i]
                }));
            }
        }
    });
}
```
這邊我用POST的方式，而data是前端必須傳給後端的資料，讓後端能判斷，**empty很重要**(de很久才想到)，而for迴圈則是加入球員，前端的程式碼寫好了，接著回到`app.py`，我需要設置一個`/select_team`才能接起來。

`app.py`

```python
from select_player import getplayer
@app.route('/select_team', methods=['POST'])
def select_team():
    team = request.form.getlist('team')
    player_list = getplayer(team[0])
    return jsonify(player_list)
```
**getlist進來會是list**

`select_player.py`

```python
# coding: utf-8
import csv

def getplayer(team):
    with open('player_ID.csv', 'r',encoding='utf8') as csvfile:
        rows = csv.DictReader(csvfile)
        player_list = []
        if team == '中信兄弟':
            for row in rows:
                if row['Team ID'] == 'E02':
                    player_list.append(row['Name'])
        elif team == '統一獅':
            for row in rows:
                if row['Team ID'] == 'L01':
                    player_list.append(row['Name'])
        elif team == '富邦悍將':
            for row in rows:
                if row['Team ID'] == 'B04':
                    player_list.append(row['Name'])
        elif team == '樂天桃猿':
            for row in rows:
                if row['Team ID'] == 'AJL011':
                    player_list.append(row['Name'])

        return player_list
```

但是現在在第一次進網頁時，第二欄會是空的，我希望在載入好時能先執行一遍，這邊我用了ready來達成

```javascript
$(document).ready(select_team());
```

