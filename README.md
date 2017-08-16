# Crawler-sina
对 http://news.sina.com.cn/china/ 中新闻进行爬取
```
import requests
from bs4 import BeautifulSoup
import re
import json
commentsurl = 'http://comment5.news.sina.com.cn/page/info?version=1&format=js&\
channel=gn&newsid=comos-{}&group=&compress=0&ie=utf-8&\
oe=utf-8&page=1&page_size=20'
    
def getCommentsCount(newsUrl):
    m = re.search('doc-i(.+).shtml', newsUrl)
    newid = m.group(1)
    comments = requests.get(commentsurl.format(newid))
    jd = json.loads(comments.text.strip('var data='))
    return jd['result']['count']['total']

def getDetails(newsUrl):
    detail={}
    res = requests.get(newsUrl)
    res.encoding = 'utf-8'
    soup = BeautifulSoup(res.text, 'html.parser')
    detail['title'] = soup.select('#artibodyTitle')[0].text
    detail['time'] = soup.select('.time-source')[0].contents[0].strip()
    detail['source'] = soup.select('.time-source span a')[0].text
    detail['article'] = ' '.join([p.text.strip() for p in soup.select('#artibody p')[:-1]])
    detail['editor'] = soup.select('.article-editor')[0].text.lstrip('责任编辑：').strip()
    detail['commentsCount'] = getCommentsCount(newsUrl)
    return detail

def parseListLink(url):
    newsdetails = []
    res = requests.get(url)
    jd = json.loads(res.text.lstrip('  newsloadercallback(').rstrip(');'))
    for ent in jd['result']['data']:
        newsdetails.append(getDetails(ent['url']))
    return newsdetails
    
url = 'http://api.roll.news.sina.com.cn/zt_list?channel=news&cat_1=gnxw&cat_2==gdxw1||=gatxw||=zs\
-pl||=mtjj&level==1||=2&show_ext=1&show_all=1&show_num=22&tag=1&format=json&page={}'
news_total = []
for i in range(3):  #可以改变爬取页数
    newsurl = url.format(i)
    newsary = parseListLink(newsurl)
    news_total.extend(newsary)
    
import pandas
df = pandas.DataFrame(news_total)

import sqlite3
with sqlite3.connect('news.sqlite') as db:
    df.to_sql('news', con = db)
    
with sqlite3.connect('news.sqlite') as db:
    df2 = pandas.read_sql_query('SELECT * FROM news', con = db)
```
