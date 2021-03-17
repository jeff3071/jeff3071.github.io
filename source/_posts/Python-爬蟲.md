---
title: '[Python]爬蟲'
date: 2021-02-03 21:43:27
tags: [Python, 爬蟲, 棒球分析, 中職]
categories: 資料視覺化
---
## 前言

這篇文章主要是記錄我的學習歷程，以棒球為主題搭建一個數據查詢的網站，從取得資料到架設網站，過程會使用Python來爬取及架設網站。


<!--more-->
> 本系列是整理之前在鐵人賽的文章
## 取得數據

數據的來源為中職的[官網](http://www.cpbl.com.tw/stats/all.html)
但因為中職官網沒有進階數據如OPS+等，會從[CPBL stat](http://my.cpblstats.com/)來取得比較進階的數據

首先要做的是逐月分的表現，只有中職官網有數據，首先觀察球員個人頁面的url

林安可頁面
`http://www.cpbl.com.tw/player/apart.html?player_id=k429&teamno=L01&year=2020&type=05`

蘇智傑頁面
`http://www.cpbl.com.tw/player/apart.html?player_id=H594&teamno=L01&year=2020&type=05`

比較一下發現type與year是不變的，剩下兩個就是需要的變數，必須想辦法取得player_id與teamno，回到全紀錄查詢，按下F12發現在<td\>下的<a\>可以取得，接著決定要如何儲存資料，而這邊決定用csv來儲存。

```Python
import requests
from bs4 import BeautifulSoup
import csv

def get_all_player():
    url = "http://www.cpbl.com.tw/stats/all.html?&game_type=01&&stat=pbat&year=2020&online=1&per_page="

    fieldnames = ['Name','ID','Team ID']
    with open("player_ID.csv", 'w') as csvfile:
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        writer.writeheader()
        for i in range(5):
            url = url
            
            print("爬取" + url + str(i+1) + "中")
            r = requests.get(url + str(i+1))

            soup = BeautifulSoup(r.text, 'html.parser')

            td = soup.select("td a")

            for j in range(len(td)):
                player_id = {}
                temp = td[j]["href"].split("?")[1].split("&")
                player_id['Name'] = td[j].text.strip()
                player_id['ID'] = temp[0].split("=")[1]
                player_id['Team ID'] = temp[1].split("=")[1]
                writer.writerow(player_id)
```

有了ID後，接著要取得各個月份的數據。

打開選手頁面，這個表就是目標
![](https://i.imgur.com/Lqt1zp4.png)
按下F12觀察，這個表是table來排版的
並且在數據的部分有加class
找到了目標位置就可以開始解析

```python
player_url = "http://www.cpbl.com.tw/players/apart.html?year=2020&type=05&"
with open('player_ID.csv', 'r') as csvfile:
    rows = csv.DictReader(csvfile)
    for row in rows:
        player_url = player_url + 'player_id=' + str(row['ID']) + '&teamno=' + str(row['Team ID'])
        res = requests.get(player_url)
        soup = BeautifulSoup(res.text, 'html.parser')
        player_td = soup.select(".display_a1")
    month = soup.select("td")
    month = soup.select("tr > td:nth-of-type(1)")
    month.remove(month[0])
    month.remove(month[0])

    team = soup.select('span')

    player_info, avg_list, OBP_list, SLG_list, PA_list, AB_list, RBI_list, H_list, HR_list = {
    }, {}, {}, {}, {}, {}, {}, {},{}

    for i in range(len(month)):
        avg_list[month[i].text] = player_td[19 + i*10].text
        OBP_list[month[i].text] = player_td[17 + i*10].text
        SLG_list[month[i].text] = player_td[18 + i*10].text
        PA_list[month[i].text] = player_td[10 + i*10].text
        AB_list[month[i].text] = player_td[11 + i*10].text
        RBI_list[month[i].text] = player_td[12 + i*10].text
        H_list[month[i].text] = player_td[13+i*10].text
        HR_list[month[i].text] = player_td[14+i*10].text

    player_info[player_td[9].text] = avg_list
    player_info[player_td[7].text] = OBP_list
    player_info[player_td[8].text] = SLG_list
    player_info[player_td[0].text] = PA_list
    player_info[player_td[1].text] = AB_list
    player_info[player_td[2].text] = RBI_list
    player_info[player_td[3].text] = H_list
    player_info[player_td[4].text] = HR_list
```
> 這邊寫得很醜，~~但我懶得改了~~
> 下個數據會用好一點的方法


資料爬下來了，但是如果要進行繪圖的話還必須解決月份不一致的問題，有些選手可能一整個月都沒出賽，或是新秀升到一軍，導致長度不一，我這邊用一個長度為8的list來存(因為賽季長度的關係最多8個月)。

```python
total_info = {}
for info_type in player_info:
    info = ['0', '0', '0', '0', '0', '0', '0', '0']
    for data in player_info[info_type]:
        n = attr.index(data)
        info[n] = player_info[info_type][data]
    total_info[info_type] = info
total_info['Name'] = row['Name']
```

到這邊各月份的數據都爬下來了，小缺點是爬取速度還需要再加強，接著會優化這隻爬蟲。

