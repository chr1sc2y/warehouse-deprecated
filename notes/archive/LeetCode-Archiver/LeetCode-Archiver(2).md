---
title: "LeetCode Archiver(2)：获取题目信息"
date: 2018-12-21T15:06:01+11:00
draft: false
categories: ["Python"]
---

## 创建爬虫

在新建好项目后，用PyCharm或其他IDE打开该项目。进入该项目文件夹，使用```genspider```命令新建一个爬虫：
```
cd scrapy_project
scrapy genspider QuestionSetSpider leetcode.com
```
其中QuestionSetSpider是爬虫的名字，leetcode.com是我们打算爬取的网站的域名。

新建好爬虫之后可以看到在项目的spiders文件夹下新增了一个名为 QuestionSetSpider.py的文件，这就是我们刚才新建的爬虫文件。这个爬虫文件会自动生成以下代码
```
# -*- coding: utf-8 -*-
import scrapy

class QuestionSetSpider(scrapy.Spider):
    name = 'QuestionSetSpider'
    allowed_domains = ['leetcode.com']
    start_urls = ['http://leetcode.com/']

    def parse(self, response):
        pass

```
- QuestionSetSpider类继承自scrapy.Spider，也就是scrapy框架中所有爬虫的基类；
- self.name属性是该爬虫的名字，在该爬虫文件的外部可以通过这个属性获取当前爬虫；
- self.allowed_domains是当前爬虫文件可以访问的域名列表，如果在爬取页面时进入了一个该域名以外的url会抛出错误；
- self.start_urls是一个url列表，基类中定义了start_requests函数，它会遍历self.start_urls，并对每一个url调用scrapy.Request(url, dont_filter=True)，为了实现爬取题目的需求，我们需要重写self.start_urls函数

## 获取题目详细信息

### 分析

LeetCode使用了GraphQL进行数据的查询和传输，大部分页面都是通过JS渲染生成的动态页面，所以无法直接从页面上获取标签，即使使用提供JavaScript渲染服务的库（例如Splash）也无法获取全部的数据，所以只能通过发送请求来获取数据。

为了爬取题目的详细信息，我们首先要从题目列表进入每个题目对应的链接。

首先打开leetcode的[problem](https://leetcode.com/problemset/all/)列表，按F12打开Chrome的开发者工具，进入Network标签栏，勾选上Preserve log，刷新该页面。

![1](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Use-Scrapy-to-Crawl-LeetCode/1.png)

可以看到，网页向 https://leetcode.com/api/problems/all/ 发送了一个名为"all/"的GET类型的Request，这就是获取所有题目链接和相关信息的请求。如果此时已经安装了Toggle JavaScript插件，我们可以直接右键点击“Open in new tab”，查看该请求返回的Response。

![2](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Use-Scrapy-to-Crawl-LeetCode/2.png)

更方便的方法是使用postman向服务器发送一个相同的Request，并将其保存下来，这样如果我们下次需要查看相应的Response的时候就不需要再使用开发者工具了。

![3](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Use-Scrapy-to-Crawl-LeetCode/3.png)

返回的Response是一个json对象，其中的"stat_status_pairs"键所对应的值是所有包含题目信息的list，而列表中的["stat"]["question__title_slug"]就是题目所在的页面。以Largest Perimeter Triangle为例，将其title_slug拼接到https://leetcode.com/problems/ 后，进入页面https://leetcode.com/problems/largest-perimeter-triangle/ 。同样地，打开开发者工具，刷新页面，可以看到服务器返回了很多项graphql的查询数据，通过查看Request Payload可以找到其中operationName为"questionData"的一项，这就是当前题目的详细信息。

![4](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Use-Scrapy-to-Crawl-LeetCode/4.png)

将Payload复制粘贴到postman的Body中，在Headers中设置Content-Type为application/json，发送请求，可以看到返回的是一个json对象，包含了该题目所对应的所有信息。

![5](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Use-Scrapy-to-Crawl-LeetCode/5.png)

接下来我们就可以对该题目的信息进行处理了。

### 实现

为了获取题目列表的json对象，我们需要先重写start_requests函数。
```
def start_requests(self):
        self.Login() # 用户登录，后续会用到
        questionset_url = "https://leetcode.com/api/problems/all/"
        yield scrapy.Request(url=questionset_url, callback=self.ParseQuestionSet)
```
Request是scrapy的一个类对象，功能类似于requests库中的get函数，可以让scrapy框架中的Downloader向url发送一个get请求，并将获取的response交给指定的爬虫文件中的回调函数进行相应的处理，其构造函数如下
```
class Request(object_ref):

    def __init__(self, url, callback=None, method='GET', headers=None, body=None, cookies=None, meta=None, encoding='utf-8', priority=0, dont_filter=False, errback=None, flags=None):
    ...
```
在获取到json对象之后，可以通过遍历"stat_status_pairs"键所对应的列表，并取出["stat"]["question__title_slug"]的值，得到题目的title_slug。此时我们不再需要进行打开题目相关页面的操作，直接向GraphQL发送查询详细信息的request即可。

我们可以从postman直接获取到发送请求相关的代码。因为每个题目的title_slug不同，我们可以将Payload中titleSlug后的字段改为一个不会重复的独特的字符串，在每一次获取到新的title_slug之后用replace函数替换它，发送新的请求，然后再将其替换回独特的字符串。

![6](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Use-Scrapy-to-Crawl-LeetCode/6.png)

准备好Payload和Headers之后，我们可以使用FormRequest发送POST请求向GraphQL查询数据。FormRequest是scrapy的一个类对象，功能类似于requests库中的post函数，让scrapy框架中的Downloader向url发送一个post请求，并将获取的response交给指定的爬虫文件中的回调函数进行相应的处理。此处在发送POST请求之后response被交给ParseQuestionData函数进行处理。

```
    question_payload = "{\n    \"operationName\": \"questionData\",\n    \"variables\": {\n        \"titleSlug\": \"QuestionName\"\n    },\n    \"query\": \"query questionData($titleSlug: String!) {\\n  question(titleSlug: $titleSlug) {\\n    questionId\\n    questionFrontendId\\n    boundTopicId\\n    title\\n    titleSlug\\n    content\\n    translatedTitle\\n    translatedContent\\n    isPaidOnly\\n    difficulty\\n    likes\\n    dislikes\\n    isLiked\\n    similarQuestions\\n    contributors {\\n      username\\n      profileUrl\\n      avatarUrl\\n      __typename\\n    }\\n    langToValidPlayground\\n    topicTags {\\n      name\\n      slug\\n      translatedName\\n      __typename\\n    }\\n    companyTagStats\\n    codeSnippets {\\n      lang\\n      langSlug\\n      code\\n      __typename\\n    }\\n    stats\\n    hints\\n    solution {\\n      id\\n      canSeeDetail\\n      __typename\\n    }\\n    status\\n    sampleTestCase\\n    metaData\\n    judgerAvailable\\n    judgeType\\n    mysqlSchemas\\n    enableRunCode\\n    enableTestMode\\n    envInfo\\n    __typename\\n  }\\n}\\n\"\n}\n"

    graphql_url = "https://leetcode.com/graphql"

    def ParseQuestionSet(self, response):
        headers = {
            "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'",
            "content-type": "application/json"  # necessary
        }
        questionSet = json.loads(response.text)
        questionSet = questionSet["stat_status_pairs"]
        for question in questionSet:
            title_slug = question["stat"]["question__title_slug"]
            self.question_payload = self.question_payload.replace("QuestionName", title_slug)
            yield scrapy.FormRequest(url=self.graphql_url, callback=self.ParseQuestionData,
                                     headers=headers, body=self.question_payload)
            self.question_payload = self.question_payload.replace(title_slug, "QuestionName")
```

现在数据已经获取到了，我们需要在items.py文件中定义一个类用来存储题目的详细信息。items.py文件中的类继承自scrapy.Item类，是提供给scrapy框架中的组件Item Pipeline进行处理的统一的的数据结构。

```
import scrapy

class QuestionDataItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    id = scrapy.Field()
    title = scrapy.Field()
    content = scrapy.Field()
    submission_list = scrapy.Field()
    topics = scrapy.Field()
    difficulty = scrapy.Field()
    ac_rate = scrapy.Field()
    likes = scrapy.Field()
    dislikes = scrapy.Field()
    slug = scrapy.Field()
```
定义了QuestionDataItem类之后可以进入ParseQuestionData函数开始对题目详细信息的提取，我们可以根据需求提取出题目的id，title，content，topics，difficulty等信息，用一个QuestionDataItem对象来存储这些数据，然后进行yield questionDataItem操作，将这个对象交给Item Pipeline进行处理。

```
    def ParseQuestionData(self, response):
        questionData = json.loads(response.text)["data"]["question"]
        questionDataItem = QuestionDataItem()
        questionDataItem["id"] = questionData["questionFrontendId"]
        questionDataItem["title"] = questionData["title"]
        questionDataItem["content"] = questionData["content"]
        topics = []
        for topic in questionData["topicTags"]:
            topics.append(topic["name"])
        if len(topics) == 0:
            topics.append("None")
        questionDataItem["topics"] = topics
        questionDataItem["difficulty"] = questionData["difficulty"]
        stats = json.loads(questionData["stats"])
        questionDataItem["ac_rate"] = stats["acRate"]
        questionDataItem["likes"] = questionData["likes"]
        questionDataItem["dislikes"] = questionData["dislikes"]
        questionDataItem["slug"] = questionData["titleSlug"]
        submission_list = self.GetSubmissionList(questionDataItem["slug"])
        questionDataItem["submission_list"] = submission_list

        yield questionDataItem
```

至此题目信息的爬取就完成了。

## 参考资料

<a href="https://scrapy-chs.readthedocs.io/zh_CN/0.24/" target="_blank">Scrapy官方文档</a>

<a href="https://learning.getpostman.com/docs/postman/launching_postman/installation_and_updates/">Postman官方文档</a>