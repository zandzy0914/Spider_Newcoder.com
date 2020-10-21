# Spider_Newcoder.com
python爬虫爬取牛客网
# #8 提前(了解)996社畜(相关信息)

## 一、背景

```
鲨鲨作为刚入这行的萌新，很想知道之后实习的岗位信息和相关工资。但这时候如果自己一个个去看去查，感觉不够Hacker，么得逼格，于是鲨鲨快乐地去点亮我们可爱的爬虫技能点了(ding~)
```

**网络爬虫** （web crawler），也叫网络蜘蛛（spider），是一种用来自动浏览[万维网](https://zh.wikipedia.org/wiki/万维网)的[网络机器人](https://zh.wikipedia.org/w/index.php?title=网络机器人&action=edit&redlink=1)。其目的一般为编纂[网络索引](https://zh.wikipedia.org/w/index.php?title=网络索引&action=edit&redlink=1) 这听起来并不像人话 让我们通俗点解释一下，如果将因特网比作一张错综复杂的交错巨网，那么爬虫就是这张网上的一只只小蜘蛛，这些蜘蛛会将他们探下的路的踪迹和在路上抓到的资源反馈给我们。

总而言之，爬虫就是一段可以自动抓取互联网上对于我们有价值的信息的程序。
既然说到了程序和编程，接下来就开启冒险，让我们来谈谈这次的通关任务吧。

## 二、主线任务

1. **爬取数据** ：经典找实习网站 [牛客网 ](https://www.nowcoder.com/intern/center)，爬取相关信息并存到 **数据库** （可以先从存在本地开始）；

2. 请将尝试爬取的过程整理到**markdown**中，连同代码**github链接**和爬到的**数据**，打包发给我们；

## 三、任务分析

1. 爬取牛客网有关实习岗位信息和相关工资的信息
2. 将这些信息保存至数据库
3. 导出数据
4. 优化

## 四、任务实现

### 4.1网页分析

打开 [牛客网实习广场](https://www.nowcoder.com/intern/center?recruitType=1&page=1) 按下F12调出开发者工具，查看该页面的HTML信息，检查实习岗位信息后可以发现，有关内容都放在了形如下的HTML块内：

```html
<li class="clearfix">
	<div class="reco-job-main">
	 <div class="reco-job-pic">
	 <div class="reco-job-cont">...</div>
     <div class="reco-job-info">...</div>
     </div>
</li>
```

其中包括***公司信息***

```html
<div class="reco-job-com">
  <a href="/intern/39418?jobIds=34516&amp;ncsr=" target="_blank">腾讯</a>
</div>
```

***岗位信息***

```
<div class="reco-job-cont">
  <a href="/intern/39418?jobIds=34516&amp;ncsr=" class="reco-job-title" target="_blank">腾讯音乐算法实习生（QQ音乐 全民K歌）</a>
```

***工作地点***

```
<span class="nk-txt-ellipsis job-address" data-title="深圳" data-tips-index="11"><i class="icon-map-marker"></i>深圳</span>
```

***相关工资***

```
<span><span class="ico-nb">¥</span>薪酬面议</span>
```

故我们要是想要爬取实习信息，只需在HTML文件定位到相关语句的位置，即可获取所需信息。

### 4.2数据库建立

1. 通过图形化管理MYSQL数据库的工具，如 *SQLyog* 等软件直接建立数据库 ***INTERNSHIP_INFORMATION*** ，并在该数据库中新建名为 *internship_information* 的表，采用的是 *utf-8* 的字符集。表中需要有 *ID* , *company_name* , *job_info* , *address* , *pay* 这五项，对应的类型分别是 *int* , *varchar*(100) , *varchar*(100) , *varchar*(100) , *varchar*(100)。

2. 通过命令提示符新建

   ```
   mysql> CREATE DATABASE INTERNSHIP_INFORMATION;
   
   mysql> USE INTERNSHIP_INFORMATION;
   mysql> CREATE TABLE IF NOT EXISIS `internship_information`(
   	->     `ID` INT UNSIGNED AUTO_INCREMENT,
   	->     `company_name` VARCHAR(100),
   	->     `job_info` VARCHAR(100),
   	->     `address` VARCHAR(100),
   	->     `pay` VARCHAR(100),
   	->     PRIMARY KEY(`ID`)
   	-> )ENGINE=InnoDB DEFAULT CHARSET=utf8;
   ```

### 4.3代码实现

```python
import pymysql
import requests
from bs4 import BeautifulSoup

#连接数据库
connect = pymysql.Connect(
    host = 'localhost',
    port = 3306,
    user = 'root',
    password = '140918',   #数据库的密码
    db = 'INTERNSHIP_INFORMATION',    #建立的数据库名称
    charset = 'utf8'    #采用的字符编码方式
)

#获取游标
cursor = connect.cursor()

#将信息插入数据库
def put_in(company_name,job_info,address,pay):
    sql = "INSERT INTO internship_information (company_name,job_info,address,pay) VALUES ('{company_name}','{job_info}','{address}','{pay}')".format(company_name=company_name,job_info=job_info,address=address,pay=pay)    #SQL语句
    cursor.execute(sql)    #执行SQL语句
    connect.commit()    #把事务所做的修改保存到数据库

url =  'https://www.nowcoder.com/intern/center?recruitType=1&page={}'    #要爬取的链接主体
headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36"}    #更改头域

#爬取实习岗位信息
def get_information(url,n):
    url = 'https://www.nowcoder.com/intern/center?recruitType=1&page={}'.format(n)    #爬取链接
    html = requests.get(url, headers=headers)    #获取html文件
    
    soup = BeautifulSoup(html.content,'lxml')    #转化成对象类型
    Info = soup.find_all('li',class_="clearfix")    #寻找有关实习信息的语句
   
    for li in Info:
        job_name = str(li.select('a')[1].string)    #获取岗位信息
        company_name = str(li.select('a')[2].string)    #获取公司信息
        #company_name = str(li.select('div')[3].string)
        address = str(li.select('span')[0].text)    #获取工作地址
        pay = str(li.select('span')[1].text)    #获取薪资信息
        
        print(company_name,job_name,address,pay)
    
        put_in(company_name,job_name,address,pay)    #将获取的信息插入数据库

if __name__ == "__main__":
    for n in range(1,11):    #循环操作，爬取1-10页内容
        get_information(url,n)
```

- [github链接]()

### 4.4代码分析

#### 4.4.1库函数分析

1. ```python
   import requests
   ```

   - 通过调用 *requests* 库，来实现爬虫功能，其中要注意选择请求网页的方式： *get* 和 *post* 。

2. ```python
   from bs4 import BeautifulSoup
   ```

   - 通过调用 *BeautifulSoup* 库，来实现对HTML文件的分析，简化寻找相关语句的流程和方法。

3. ```python
   import pymysql
   ```

   - 通过调用 *pymysql* 库，来实现与数据库的连接，以存入数据。

#### 4.4.2重点分析

1. ```python
   soup = BeautifulSoup(html.content,'lxml')    #转化成对象类型
   Info = soup.find_all('li',class_="clearfix")    #寻找有关实习信息的语句
   ```

   - 第一句通过调用 *BeautifulSoup* 库来生成 *soup* 对象，以便之后对内容的解析。

   - 第二句则是通过 *soup.find_all()* 函数来寻找所需信息并返回内容，其中 *'li’* 表示要查找的 *value* ，可以是string，list，function，真值或者re正则表达式，*class_="clearfix"* 则是查找的 *value* 的一些属性，class等。

2. ```python
   for li in Info:
           job_name = str(li.select('a')[1].string)    #获取岗位信息
           company_name = str(li.select('a')[2].string)    #获取公司信息
           #company_name = str(li.select('div')[3].string)
           address = str(li.select('span')[0].text)    #获取工作地址
           pay = str(li.select('span')[1].text)    #获取薪资信息
   ```

   - 通过遍历 ***Info*** ，对其中的所有 *li* 对象都做 *select* 操作，比如要寻找 *job_name* 可以通过分析其在HTML文件中所处位置的前后语句特点，通过 *li.select()函数* 定位并获取字符串，再通过字符串的操作取得所需的有效信息。
   - 注意不同参数的选择可能导致通过 *li.select()函数* 得到的字符串不同

### 4.5数据呈现

- 部分数据：

| ID   | company_name         | job_info                   | address                                                      | pay           |
| ---- | -------------------- | -------------------------- | ------------------------------------------------------------ | ------------- |
| 1    | 动次科技             | 前端实习生                 | 北京                                                         | ¥薪酬面议     |
| 2    | 旷视科技             | 前端实习生                 | 武汉                                                         | ¥200-250元/天 |
| 3    | 平安科技             | 平安科技前端开发实习生     | 深圳                                                         | ¥250-300元/天 |
| 4    | 爱钛技术             | 前端工程师-实习生          | 上海                                                         | ¥300-400元/天 |
| 5    | 元戎启行             | 前端开发实习生             | 深圳                                                         | ¥薪酬面议     |
| 6    | 快手                 | 快手-数据架构-前端开发实习 | 北京                                                         | ¥薪酬面议     |
| 7    | 百度                 | web开发实习生（深圳）      | 深圳                                                         | ¥200-250元/天 |
| 8    | Momenta魔门塔        | 前端研发工程师             | 北京                                                         | ¥400-500元/天 |
| 9    | 深圳互想科技有限公司 | UI实习生 全国远程          | 北京,上海,广州,深圳,杭州,南京,成都,厦门,武汉,西安,长沙,哈尔滨,合肥,长春,其他 | ¥200-250元/天 |
| 10   | 深圳互想科技有限公司 | 前端开发实习生——远程       | 北京,上海,广州,深圳,杭州,南京,成都,厦门,武汉,西安,长沙,哈尔滨,合肥,长春,其他 | ¥250-300元/天 |

## 五、成果评估

#### 5.1优缺点

1. 优点：基本实现需求，获取有关实习岗位的信息，满足需要，同时将数据存储在数据库里，便于增删改查。
2. 缺点：获得的信息较为简单，有效信息较少，同时信息的排序没有规律，也未对信息进行比较操作和分类。

#### 5.2改进

1. 在代码中增加对工作地点、工作内容的比较，使得爬取的信息可以根据地点、工作内容来进行区分、存储。
2. 通过数据可视化基本操作，可以按城市，企业，薪资各方面进行交叉分析，获取最优解。
