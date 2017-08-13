# Crawler-sina
对 http://news.sina.com.cn/china/ 中新闻进行爬取

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

news = 'http://news.sina.com.cn/c/nd/2017-07-03/doc-ifyhrxtp6459083.shtml'
getDetails(news)
