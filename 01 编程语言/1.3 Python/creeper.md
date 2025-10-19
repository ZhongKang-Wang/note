# 20250729

## HTML 简单了解

超文本`标记`语言（英语：HyperText Markup Language，简称：HTML）

是一种用于创建网页的标准标记语言。

您可以使用 HTML 来建立自己的 WEB 站点，HTML 运行在浏览器上，由浏览器来解析。

```html
<!DOCTYPE html> <!--声明文档类型为html，不区分大小写 -->
<html>
    <head>
    <meta charset="utf-8">
    <title>菜鸟教程(runoob.com)</title>
    </head>
    <body>
    
    <h1>我的第一个标题</h1>
    
    <p>我的第一个段落。</p>

    <table width="200px", height="200px", border="1px"> <!-- 表格 tr叫做行， td叫做列 tabel data cell, px表示像素-->
        <tr>
            <td>姓名</td>
            <td>年龄</td>
            <td>性别</td>
        </tr>
        <tr>
            <td>张三</td>
            <td>18</td>
            <td>男</td>
        </tr>

    </table> 
    <ul> <!--UNordered list-->
        <li>铁锅炖大鹅</li> <!--无序列表 list item-->
        <li>锅包肉</li>
    </ul>

    <ol>
        <li>穿上衣服</li>
        <li>下床</li>
    </ol>

    <a href="http://www.atguigu.com">尚硅谷</a> <!--超链接anchor 传送锚点 Hypertext Reference-->
    
    </body>
</html>
```



前端先置知识就学到这里。



所谓爬虫，就是根据一个域名(http://www.taobao.com)进行爬取网页，获取有效信息

又可以认为，通过程序模拟浏览器，向服务器发送请求，获取响应信息。



## urllib 基本使用

基本使用

```python
# urllib 基本使用
# 使用urllib获取百度首页的源码
import urllib.request

# 1. 定义一个url 你要访问的地址
url = "http://www.baidu.com"
# 2. 模拟浏览器向服务器发送请求
response = urllib.request.urlopen(url)
# 3. 获取响应页面的源码
# read()方法 返回字节对象(bytes类型)
# 我们要将其解码为字符串 decode('utf-8')
content = response.read().decode('utf-8') 
# 4. 打印数据
print(content)
```

urllib

- 一个类型
  - `response`是 `HTTPResponse`类型
- 6个方法`read()`、`readline()`...

```python
content = response.readline() # 读一行
print(content)

content = response.readlines() # 一行一行读
print(content)

print(response.getcode()) # 打印响应的状态码 200 表示没有问题
print(response.geturl()) # 打印返回的url地址
print(response.getheaders()) # 打印一些状态信息
```



# 20250804

Q：如何通过爬虫下载一些资源到本地？

```python
# urllib 下载资源
# 使用urllib获取百度首页的源码
import urllib.request

# 1. 下载网页
urlpage = 'http://www.baidu.com'
# urllib.request.urlretrieve(urlpage, 'baidu.html')

# 2. 下载图片
urlimg = 'https://image.baidu.com/search/detail?adpicid=0&b_applid=7702250056126103141&bdtype=0&commodity=&copyright=&cs=1578633941%2C2541624604&di=7523999300557209601&fr=click-pic&fromurl=http%253A%252F%252Fwww.duitang.com%252Fblog%252F%253Fid%253D1500001316&gsm=3c&hd=&height=0&hot=&ic=&ie=utf-8&imgformat=&imgratio=&imgspn=0&is=0%2C0&isImgSet=&latest=&lid=&lm=&objurl=https%253A%252F%252Fc-ssl.dtstatic.com%252Fuploads%252Fblog%252F202311%252F06%252FM6SewzEvcBaYXXD.thumb.1000_0.jpeg&os=2256289415%2C591559105&pd=image_content&pi=0&pn=38&rn=1&simid=1578633941%2C2541624604&tn=baiduimagedetail&width=0&word=%E7%AB%A0%E9%B1%BC%E5%93%A5&z='
# urllib.request.urlretrieve(urlimg, 'zhangyu.jpg')
# 下载的图片显示已损毁
# 原因是需要找真实图片地址，例如以.jpg结尾
urlimg = 'https://ww2.sinaimg.cn/mw690/00839aAegy1hn659w4vusj30yi22o44t.jpg'
urllib.request.urlretrieve(urlimg, 'juzi.jpg')
# 成功了

# 3. 下载视频
urlvideo = ''
urllib.request.urlretrieve(urlvideo, 'bilibili.mp4')
# B站的视频下不下来
```



## 请求对象的定制

为了解决反爬的一种手段。

`user agent`，用户代理，是一个特殊的字符串头，可以识别客户使用的操作系统及版本、浏览器及版本等信息。



```python
import urllib.request

url = 'https://www.baidu.com'

# url的组成
'''
1. 协议 http/https https加了*，更加安全
2. 主机
3. 端口号
http 80 https 443
mysql 3306 oracle 1521
redis 6379 mongodb 27017
4. 路径
5. 参数
6. 锚点

example:
    https://www.baidu.com/s?wd=%E5%88%98%E4%BA%A6%E8%8F%B2&rsv_spt=1&rsv_iqid=0xf123de0903c52326&issp=1&f=8&rsv_bp=1&rsv_idx=2&ie=utf-8&tn=baiduhome_pg&rsv_enter=1&rsv_dl=tb&rsv_sug3=10&rsv_sug1=10&rsv_sug7=101&rsv_sug2=0&rsv_btype=i&prefixsug=%25E5%2588%2598%25E4%25BA%25A6%25E8%258F%25B2&rsp=4&inputT=1891&rsv_sug4=2740
    协议: https
    主机名: www.baidu.com
    端口号: 默认443
    路径： s
    参数: wd = ...
    锚点: #
'''
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36 Edg/138.0.0.0'
}
# 因为urlopen方法中不能传入字典，所以需要定义request对象
# 请求对象的定制
request = urllib.request.Request(url=url, headers=headers) # 关键字传参
response = urllib.request.urlopen(request)
content = response.read().decode('utf-8')
print(content)
# 如果检索不出来可能是遇到了反爬，需要用headers伪装自己
```



## 编码、解码

1. `quote`方法

```python
'''
https://www.baidu.com/s?wd=%E5%88%98%E4%BA%A6%E8%8F%B2
wd = 刘亦菲

ascii编码 A=65 a=97
编入中文 GB2312
Unicode 将所有语言统一编码,通常两个字节表示一个字符

%E5%88%98%E4%BA%A6%E8%8F%B2 就是unicode编码
'''

# 获取https://www.baidu.com/s?wd=刘亦菲的网页源码
import urllib.request
import urllib.parse
# 要将 刘亦菲三个字 变成unicode编码格式

name = urllib.parse.quote('刘亦菲')
url = 'https://www.baidu.com/s?wd='
url = url + name

headers = {
    'Cookie': 'BIDUPSID=DBF039C1A1CDEC1ED41EBC48132A5D02; PSTM=1720947667; BDUSS=FPVFU4Y204YkJneER0dUI3ZlVTRFN2TUZpNlROQ3hxfm5vb0NOSkFCU1E1OEZtSUFBQUFBJCQAAAAAAAAAAAEAAADB0smOsNm2~sfYtKjW1cr0ztIAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAJBammaQWppmOE; BDUSS_BFESS=FPVFU4Y204YkJneER0dUI3ZlVTRFN2TUZpNlROQ3hxfm5vb0NOSkFCU1E1OEZtSUFBQUFBJCQAAAAAAAAAAAEAAADB0smOsNm2~sfYtKjW1cr0ztIAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAJBammaQWppmOE; MAWEBCUID=web_JWBwAOKtiZpMjqansjUawXLHZEmcUeiPjUTmpMraVwckHdMfyJ; H_WISE_SIDS_BFESS=62325_62831_63243_63351_63381_63442_63458_63509; BAIDUID_BFESS=9F8B91D8950908AF9574D3CAE57F493C:FG=1; ZFY=9rktrk4B9k3P1sU2MnQ6Yt7G19Jlz3ehSj7Rc4kmL00:C; __bid_n=19849ee05e011521a4c710; RT="z=1&dm=baidu.com&si=c924cd81-8cbc-4939-b58f-1ac492c00b19&ss=mdocyn6x&sl=1&tt=atj&bcn=https%3A%2F%2Ffclog.baidu.com%2Flog%2Fweirwood%3Ftype%3Dperf&ld=auj&ul=jub&hd=jvc"; arialoadData=false; ab_sr=1.0.1_ZGMzZTZhNTg2NzMzMjVmNWQyYzU4NzEyMzIxNjdjNmFjYzQzNDEyZmU0NmM2NjZmMTJmMjAzMDA2MzE0MzJiYWE0NWU0OGJlMDA0NzYxZmIwMWQzZjA4MzYxZGUyZmRhZGU1MzJlMjYwYzZmNTZhYjliY2Q5ZmRjZGY1NzcwMDMwNmQ1ZGIxZWU1YTYxZjZiY2VjMDQ3MTEwMjc2YjRjNmI0YzRlYTA3NDEyNjM1ZjQxMmU5ZmY0Mjg4NGVkMzY3; H_PS_PSSID=62325_63141_63325_63881_63948_64009_64125_64140_64159_64173_64222_64218_64234_64246_64253_64258_64260_64294_64318_64366_64361_64364_64373; BD_UPN=12314753; BA_HECTOR=218485ah05010k210l0ga00g8h2g8h1k90onr24; BDRCVFR[feWj1Vr5u3D]=I67x6TjHwwYf0; BD_CK_SAM=1; PSINO=3; delPer=0; BDORZ=B490B5EBF6F3CD402E515D22BCDA1598; H_WISE_SIDS=62325_63141_63325_63881_63948_64009_64125_64140_64159_64173_64222_64218_64234_64246_64253_64258_64260_64294_64318_64366_64361_64364_64373; H_PS_645EC=3238O3e9vXiMwApccKcwg0mQICcTGYEXdy9LouHsy4s%2B%2FqxUCxCE8uvUrXU',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36 Edg/138.0.0.0'
}
request = urllib.request.Request(url=url, headers=headers) # 关键字传参
response = urllib.request.urlopen(request)
content = response.read().decode('utf-8')
print(content)

```



Q: 如何进一步解决输入多个参数进行检索的问题呢？

例如 刘亦菲 + 男



2. `urlencode`方法

应用于多个参数的时候

```python
# https://www.baidu.com/s?wd=%E5%88%98%E4%BA%A6%E8%8F%B2&sex=%E5%A5%B3
# 刘亦菲 女
import urllib.parse
data = {
    'wd': '刘亦菲',
    'sex': '女'
}
a = urllib.parse.urlencode(data) # wd=%E5%88%98%E4%BA%A6%E8%8F%B2&sex=%E5%A5%B3
print(a)
```





# 20250805

## post请求

----

```python
# post 请求
import urllib.request
import urllib.parse
import json

url = 'https://fanyi.baidu.com/sug'

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36 Edg/138.0.0.0'
}

data = {
    'kw': 'spider'
}
# post请求的参数 必须要进行编码
data = urllib.parse.urlencode(data).encode('utf-8')
# post请求的参数，需要放在请求对象定制的参数中
request = urllib.request.Request(url=url, data=data, headers=headers)
# 模拟浏览器向服务器发送请求
response = urllib.request.urlopen(request)
# 获取响应的数据
content = response.read().decode('utf-8')
# 字符串->json对象
obj = json.loads(content)
print(obj)
```

