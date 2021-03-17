---
title: Cos相似度實作
date: 2020-01-18 01:28:39
tags: [python, NLP]
categories: [資料科學]
---

## 方法設計
1. 使用jieba斷詞，設置自定義字典
2. 每篇文章斷詞後，計算每篇文章對於這個集合中的詞的詞頻生成兩篇文章各自的詞頻向量
3. 計算兩個向量的餘弦相似度，值越大就表示越相似


## 實現說明

### 爬取文章
    
我們利用爬蟲爬取分析目標的粉絲專頁，並將其內容寫入MySQL裡的facebook資料表，並將jieba斷詞後不重複的結果寫入MySQL的dictionary資料表中word欄位

<!--more-->
> 我們選取怪奇事務所的粉專進行

```python
#Facebook粉絲頁網址
url = "https://mbasic.facebook.com/IncredivilleTW/?refid=13&__tn__=%2Cg"

resp = req.get(url, cookies=cookies)

html_doc = resp.text

soup = BeautifulSoup(html_doc, 'html.parser')
headers = {
    'cookie': 'datr=lsdAXbRR26aO9ePicNOFFkF6; sb=v8dAXdIceUJHQYtxfH5yJghe; _fbp=fb.1.1568882205233.239187278; locale=zh_TW; c_user=531045890; xs=37%3AqDyc_aZQWbZqJQ%3A2%3A1570409723%3A17284%3A8670%3A4bOHmE1AjVm-UQ; presence=EDvF3EtimeF1570574420EuserFA2531045890A2EstateFDt3F_5bDiFA2user_3a506028681A2EoF1EfF1C_5dEutc3F1570409759903G570574420183Elm3FnullCEchFDp_5f531045890F1CC; wd=1280x648; fr=0POPFYG9NYhEAyYKy.AWUwzziefaPVhk_qeNUq-ORj_XI.BdQMTz.zg.F2a.0.0.BdnqEu.AWV_r9K_; spin=r.1001274804_b.trunk_t.1570680542_s.1_v.2_; dpr=2; act=1570684244969%2F1'
}
docs = []
filter = re.compile("[^\u4e00-\u9fa5A-Za-z]+")
titles = soup.find_all("a")
for t in titles:
    doc = []
    if t.has_attr("href"):
    if "story.php" in t["href"] and "https://" not in t["href"]:
      #發文url
      post_url = "https://mbasic.facebook.com"+t["href"]
      #文章id
      url_rex = re.compile("story_fbid=(.*?)(?:&|$)");
      post_id = url_rex.findall(post_url)[0]
      resp_1 = req.get(post_url, cookies=cookies,headers=headers)
      #Facebook發文文章
      html_doc_1 = resp_1.text
  
      soup_1 = BeautifulSoup(html_doc_1, 'html.parser')
      #存放文章內容的list
      docs_1 = []
 
      #取得所有文章，包含留言
      titles_1 = soup_1.find_all("div")
      reply_id_array = []
      post_id_array = []
      for t_1 in titles_1:
            if(t_1.find("h3")):
                  name = t_1.find("h3").getText() #發文者
                  #發文時間,有可能我們說到的結構沒有包含abbr，要判斷以濾掉
                  if  t_1.find("abbr") is None:
                        publish_date = None;
                  else:
                        publish_date = t_1.find("abbr").getText()
                  
                  #內文
                  sub_content= t_1.find("h3").find_next("div").getText()

                  #發文id,如果沒有id，就代表是原始文章
                  reply_id= t_1.find("h3").find_parent("div").find_parent("div").get("id")

                  if reply_id is None and post_id in post_id_array:
                        continue;
                  else:
                        post_id_array.append(post_id)

                  if reply_id is not None and reply_id in reply_id_array:
                        continue;
                  else:
                        reply_id_array.append(reply_id)
                  #將發文內容斷詞
                  seg_list = jieba.cut(sub_content, cut_all=False)
                  insertDictionary(seg_list)
                  item = {
                  "post_id":post_id,
                  "author":name,
                  "content":remove_emoji(sub_content),
                  "reply_id":t_1.find("h3").find_parent("div").find_parent("div").get("id"),
                  "publish_date":publish_date,
                  "url":post_url,
                  }
                  insertContent(item)
                  docs_1.append(item)
```
    
### 製成字典
將dictionary資料表word欄位中的資料取出，並製成字典

```python
dicts = []
sql = "select * from dictionary"
mycursor = mydb.cursor()
mycursor.execute(sql)
myresult = mycursor.fetchall()

for i in myresult:
dicts.append(i[1])

```
    
### 生成詞頻向量並且計算餘弦相似度
        
    取出欲分析之兩篇文章後，再次使用jieba斷詞，並且依照字典產生詞頻向量，計算兩篇文章之餘弦相似度，值愈大代表愈相似
    
    ```python
    sql = "select post_id,content from facebook limit 2"
    mycursor = mydb.cursor()
    mycursor.execute(sql)
    myresult = mycursor.fetchall()
    post_ids = []
    post_contents = []
    post_array = []
    
    for i in myresult:
      post_ids.append(i[0])
      post_contents.append(i[1])
      seg_list = jieba.cut(i[1], cut_all=False)
      docA = np.zeros(len(dicts))
      for j in seg_list:
          p = dicts.index(j)
        docA[p:p+1] = docA[p:p+1]+1
          post_array.append(docA)
    c = np.dot(post_array[0],post_array[1])
    a_s = np.sqrt(np.sum(post_array[0]**2))
    b_s = np.sqrt(np.sum(post_array[1]**2))
    
    result = c / (a_s * b_s)
    ```
