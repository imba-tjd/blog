---
title: 第三方 Python 库
---

## 环境

预编译的Win下的包：https://www.lfd.uci.edu/~gohlke/pythonlibs/

### requirements.txt

```bash
pip freeze > requirements.txt # 一定要在venv中运行否则会把全局的写进去
pip install -r requirements.txt

SomeProject==1.4
SomeProject>=1,<2 # 逗号为且；在CLI中运行需加引号否则大于号会被认为是重定向
SomeProject~=1.4.2 # install any version “==1.4.*” that’s also “>=1.4.2”

# 另外还有pipreqs、pigar、pip-tools几个程序也是用来管理依赖的
```

### venv

* pip install 不需要--user
* 不能脱离本机环境，会在`venv/pyvenv.cfg`中硬编码Python的版本和home位置；如果Python是in-place升级了版本，可用venv --upgrade .venv，之后记得更新venv里的包；但如果Python自己的路径变化了，就只能手动改了；手动改之前shell就不要进venv了，否则报Permission denied
* `--system-site-packages`使得虚拟环境可访问系统的包，install仍不影响系统，freeze的时候就要加--local

```bash
python3 -m venv .venv
. .venv/bin/activate
deactivate
```

快捷方式：

```bash
alias activate=". .venv/bin/activate"

if not exist .venv python -m venv .venv --upgrade-deps --system-site-packages
.venv\Scripts\activate.bat

if(!(Test-Path .venv)) {python -m venv .venv --upgrade-deps --system-site-packages}
& .venv\Scripts\activate.ps1
```

### 模块

* 一个.py文件就是一个模块，模块名`__name__`按目录组织，用点分隔，import时无需也不能加.py后缀
* import运行：对于`a/b/c.py`，`import a.b.c`会依次运行a和b目录下的`__init__.py`，再运行`c.py`；但如果import的直接是目录，只会执行`__init__.py`，不会执行`__main__.py`
* python命令行运行：既可以运行文件，也可以运行目录，对于目录就是运行`__main__.py`；会把目标中的`__name__`设为`'__main__'`
* python命令行加不加-m的区别：不加-m会把目标所在的文件夹加到`sys.path`中，然后直接执行目标；加-m会把当前工作目录加到`sys.path`中，对于目录会先执行`__init__.py`再执行`__main__.py`，对于文件不能加.py后缀；`runpy.py`在其中起到了作用，不要自己创建该名字的文件
* 模块只初始化一次，所有变量归属于某个模块，import机制是线程安全的。所以模块本身是天然的单例实现
* 包的目录下必须要有`__init__.py`文件，内容可以为空，如果没有就不是包？

```python
__title__
__author__ = xxx
if __name__=='__main__': # 只有直接运行才会成立，import时不成立
    ...
```

## Scrapy

### CLI

* startproject prjname; cd prjname
* genspider -t crawl myspider url：用模板创建目标爬虫，只会有单个py文件，需要在项目中使用
* shell url：交互式爬取页面；fetch/view url：使用当前项目的设置来爬取并把内容输出到终端/浏览器上，有助于发现是否存在AJAX请求
* check：检查有没有错误
* crawl prjname -o items.json：进行爬取，可以用-a传递k=v，在爬虫的构造函数中获取；runspider myspider.py：不在项目中运行单个爬虫文件

### spiders

* 在spiders文件夹下创建
* response.body默认是bytes，如果要find中文，用body_as_unicode()
* 可把response.text传给bs4手动用非lxml解析HTML，或者改下载器中间件

```python
import scrapy
class XXXSpider(scrapy.Spider): # 必须继承
    name='xxx'
    allowed_domains=[] # 允许爬的域名；必须是list
    start_urls=[] # 或def start_requests(self):yield scrapy.Request(url=url, callback=self.parse, method = 'POST')
    def parse(self, response): # 还有parse_post
        item = Douban250Item()
        self.log(xxx)
        v = response.css('xxx')
        item['key'] = v # 也可在Item的构造函数里赋值
        yield item # 也可以直接返回字典
        next_page=xxx
        if next_page:
            yield response.follow(next_page, callback=self.parse) # 相当于创建Request，支持相对路径无需urljoin，且对于<a>会自动用href属性，可直接用css='ul.pager a'作为第一个参数；还有follow_all
    # 传递命令行参数
    def __init__(self, category=None, *args, **kwargs):
        super(XXXSpider, self).__init__(*args, **kwargs)
        self.start_urls = ['http://www.example.com/categories/%s' % category]

#先检测是否有更新再下载，还需处理起始url；无法用于Rule
def parse(self, response):
    urls = []
    for url in urls:
        yield Request(url, method='HEAD', self.check)
def check(self, response):
    date = response.headers[' Last-Modified']
    #check date to your db
    if db_date > date:  # or whatever is your case
        yield Request(response.url, self.success)
def success(self, response):
    yield item

#CrawlSpider通用爬虫，自动跟进链接
from scrapy.spiders import CrawlSpider, Rule
from scrapy.linkextractors import LinkExtractor
class XXXSpider(CrawlSpider):
    rules=( # LinkExtractor的默认tags=('a','area'), attrs='href'
        Rule(LinkExtractor(allow=r'category\.php',deny=xxx,allow_domains=xxx), # 这几个参数都可以是列表；follow的默认值，不设置回调时是True，否则是False
        Rule(LinkExtractor(allow=r'.*/index\.html'), callback='parse_item', follow=True), # 回调不能是也不要改parse
    )
```

### Items

* 相当于Model，可防止给未声明变量赋值
* 在spider中from ..items import xxxItem
* 传入dict()就变成字典，把字典传给构造函数就变成Item

```
from scrapy.item import Item,Field
class Douban250Item(Item):
    ranking = Field()
```

### Pipeline

* 处理item的，可以用来清理HTML数据、验证爬取的数据、查重和丢弃、保存到数据库
* 还需在settings.py里设置ITEM_PIPELINES

```python
import pymongo
from scrapy.conf import settings
class Douban250Pipeline(object):
    def open_spider(self, spider):
        self.client = pymongo.MongoClient(host=settings["MONGODB_HOST"], port=settings["MONGODB_PORT"])
        self.mydb = self.client[settings["MONGODB_DBNAME"]]
        self.post = self.mydb[settings["MONGODB_SHEETNAME"]]
    def close_spider(self, spider):
        self.client.close()
    def process_item(self, item, spider):
        data = dict(item)
        self.post.insert(data)
        return item # 或raise DropItem("Missing price")
```

### Middleware

```python
class BeautifulSoupMiddleware(object):
    def __init__(self, crawler):
        super(BeautifulSoupMiddleware, self).__init__()
        self.parser = crawler.settings.get('BEAUTIFULSOUP_PARSER', "html.parser")

    @classmethod
    def from_crawler(cls, crawler): # 好像如果在__init__()里使用crawler.settings，就必须重新设置一遍该方法
        return cls(crawler)

    def process_response(self, request, response, spider):
        return response.replace(body=str(BeautifulSoup(response.body, self.parser)))
```

### Settings

* TODO：如何读取设置

```python
USER_AGENT='xxx'
ROBOTSTXT_OBEY=True # 默认
LOG_LEVEL='WARNING' # 默认DEBUG
FEEDS = {'items.json':{'format': 'json','encoding': 'utf8','store_empty': False}}
```

### Debug

```python
#在IDE中debug，在scrapy.cfg同级下创建run.py；对于VSC还要"cwd": "${fileDirname}"
from scrapy import cmdline
cmd = 'scrapy crawl MySpider'
cmdline.execute(cmd.split())

#官方的方法，在自己的爬虫类里创建Twisted
from scrapy.crawler import CrawlerProcess
from scrapy.utils.project import get_project_settings
process = CrawlerProcess(get_project_settings())
process.crawl(MyCrawler, domain='scrapinghub.com') # 理论上第一个参数也可以是爬虫的name，但是我测试会报找不到失败，只能用类
process.start()
```

### Parsel

* 在cssselect和lxml上构建的库
* get()以后就变为普通的字符串了
* getall()永远返回列表
* css提供::text取文本，返回值仍为Selector；不会取到子元素的，理论上用` *::text`可获取，但实际失败了，最好用xpath的string()
* .attrib['href']及非标准的::attr可取属性，前者的结果是普通的字符串，且只会取第一个的
* 如果HTML代码有BUG（如标签未闭合），出现了多个根元素，可用`len(se.xpath('/*')) > 1`检测；直接用css只会处理第一个根元素
* xpath可以用`$k`嵌入变量，在后续参数中传入`k=v`替换
* css不允许回溯到父元素
* parsel.utils.flatten()：将多层嵌套的可迭代对象变为list
* parsel.css2xpath()：把css变为xpath
* 安装parselcli包，可使用parsel命令行，可直接输入css选择器，输`-xpath`切换到xpath选择器，输`+strip`就能自动过滤空白的，`-help`显示帮助，`-embed`启动python解释器
* 只能以lxml解析HTML，有人开了PR添加`lxml.html.html5lib`的后端，但是没合并最后关了

```python
from parsel import Selector
se = Selector(s)
se.css('a').xpath('xxx').re(r'xxx').get()/.getall()
```

### Xpath

* `.`代表当前节点，`..`代表父节点
* 选择当前节点下的某一元素直接用元素名，`xxx`相当于`./xxx`；选择某一层所有节点用`*`，不能不用否则什么都选不到
* 以`/`开头的选择器会从创建Xpath的根重新开始搜索，即使当前已经在子节点了
* `//`进行traversal，注意如果要从当前节点下搜索仍要加`.`
* `|`：表示取或/并集
* 获取属性的值：`@href`；过滤属性：`li[contains/starts-with(@class, "list") and/or @name="item"`，其中contains用于值有多个时满足一个即可；选取存在指定属性名但不限值的元素：`[@xxx]`，选取只要存在任何属性的元素：`[@*]`
* 选择当前节点不含子节点的文本：`text()`，含子节点：`string()`；只会是一个字符串且保留空格，大概就是把所有尖括号里的去掉了
* 列表位置以1开始，最后一个用`li[last()]`，前两个用`li[position()<3]`
* 有个`node()`表示匹配任何类型的节点，不知和`*`比有什么区别
* 存在Axes(轴)的语法，用于选择兄弟元素和父元素等；还有一些其它函数：concat、not、string-length等很多

## Requests

```python
import requests

# Session，能连接复用（urllib3提供连接池）以及保留cookie；即使使用了会话，方法级别的参数也不会保留
s = requests.session() # 其实最好用with，这样发生了异常也能关闭
s.request = functools.partial(s.request, timeout=3) # 连接超时时间，可为小数，默认无穷大，不加会一直等；直接赋值只影响connect超时时间，可传递元组，第二个参数控制下载超时；Session级别的只能这样设置，是故意的
allow_redirects=True; max_redirects=30 # 前者默认为True（head除外），会自动跟踪3xx因此结果直接是200；后者是默认值
verify=True #【默】，也可设为CA文件的路径，默认用Mozilla的；cert参数是客户端验证的证书
proxies={"http": "http://10.10.1.10:1080", "https": "http://10.10.1.10:1080"} # 也支持环境变量HTTP_PROXY，支持Basic Auth;不直接支持socks
headers={'User-Agent':'python-requests/2.23.0','Accept-Encoding':'gzip','Connection':'keep-alive'}.update({'Referer': referer}) #【基本默】普通dict，大小写不敏感
auth=('user', 'pass') # Authorization头，如果不放到session里，重定向时会自动去掉
cookies.set(k,v,domain,path) # 类型是RequestsCookieJar，但好像也可以按dict的方式使用。另有requests.utils.add_dict_to_cookiejar(cj, cookie_dict)、cookiejar_from_dict、dict_from_cookiejar几个函数，曾经我测试起来必须用它们才行，不能直接update

# url必须要有scheme；必须每次写完整url，要不就用requests_toolbelt提供的BaseUrlSession
# get的params会自动变成查询参数，且值为None的不会附加上去，值为list的会自动与k展开
# post的data和json传dict（json还可以是list）会自动编码并设置Content-Type，也因后者故最好不要传字符串形式的json给data，可以先loads一下；传字符串给data不会有额外变化，就是设置body；传字符串给json无意义；data还支持file-like-objects且支持流式处理，文件记得以rb打开；data还支持生成器，则会传输分块编码
r: Response = s.get(url,params={k:v})、post(url,data/json = {k:v}/str)、put/delete/head/options
# Response：
r.status_code（200，== requests.codes.ok）、raise_for_status()、json()（即使解码成功也不一定意味着请求成功，因为有时服务器会在失败时也返回json）、text（根据encoding解码的HTTP内容字符串）、url、encoding（可赋值，一般赋r.apparent_encoding或'utf-8'）、content（HTTP内容二进制，但会自动解码gzip，适用于图片等）、history（记录重定向响应列表）、headers（此时为响应头部，但仍可用r.request.headers访问请求头部）
# 只有当Content-Type包含text且不存在charset时，根据RFC此时默认字符集必须是ISO-8859-1；其它时候会进行猜测

# 保存二进制文件的推荐方式，已gzip解码
with open(filename, 'wb') as fd:
    for chunk in r.iter_content(chunk_size):
        fd.write(chunk)

# 缓存，默认是保存在内存中的dict；还支持redis：
from cachecontrol import CacheControl
cached_se = CacheControl(requests.session()) # 指定文件缓存：cache=cachecontrol.caches.FileCache('.webcache')
# 一些建议：params最好sorted一下；缓存的响应永远不要流式处理

# stream=True，则get返回时只有Header被下载下来了，可以进行一些处理，直到访问content才会下载响应体；还可以使用raw属性，是urllib3里的原始响应，未gzip解码
with requests.get(url, stream=True) as r:
    if r.encoding is None: r.encoding = 'utf-8'
    for line in r.iter_lines(decode_unicode=True): # 迭代流式API
        if line: # filter out keep-alive new lines
            decoded_line = line.decode('utf-8')
            print(json.loads(decoded_line))

# post的files={'filename': file-like-objects}用于multipart/form-data类型的请求，但默认不支持流式处理，要用requests_toolbelt才行
```

## Beautiful Soup

* 支持不同的HTML Parser，其中html5lib最接近真实网页，是纯Python，相对慢；lxml（其实是lxml.html）容错性性中游，速度最快；自带的html.parser容错性差
* HTML分为四种对象：bs4.BeautifulSoup（文档）、bs4.element.Tag（标签）、bs4.element.NavigableString（文本）、bs4.element.Comment（注释）；XML还有其他对象
* 有的属性是多值属性，如class，bs会自动处理成list（xml不做处理）。但像id中即使有空格，也只会直接返回字符串
* 支持修改，许多东西可以直接赋值和`del`删除；还有一些其它的用于修改树的方法，暂时不学
* `==`判断结构是否相同，如果要严格判断是否是同一对象，用is

```python
pip install html5lib beautifulsoup4
from bs4 import BeautifulSoup

soup = BeautifulSoup(html_doc, 'html5lib', from_encoding='utf-8') # 无需import；默认自动猜测编码但可能猜错且影响速度，soup.original_encoding记录了猜测结果
tag = soup.select/select_one('css selector') # 支持limit=2限制返回元素的数量，不是递归深度
tag.name：标签名；html的根元素的.name为[document]，类型为BeautifulSoup，因为最外层元素不止一个，除此之外和Tag差不多
tag.attrs：该节点所有属性的dict；单个属性的获取和修改直接把tag当作dict用
tag.contents：直接子节点的列表；.children：生成器；.descendants：递归子节点的生成器
tag.string：如果能递归地唯一确定一个NavigableString，则此属性可以直接获得它，否则获得的是None；Comment也可用.string，获得的是不含注释符号的注释内容
tag.strings：递归取文本生成器；.stripped_strings：每一项去掉空格和换行的strings；.text：join了的strings；.get_text()：可指定join分隔符和是否strip的text，直接用和text没区别
tag.元素：递归取第一个指定的元素；.parent：父节点，BeautifulSoup的parent是None；.parents：递归向上取所有父节点一直到BeautifulSoup；.next_sibling、.previous_sibling：下/上一个兄弟节点，关键是它们可能是“空白”，其实是'\n'这样的文本；.next_element、.previous_element：下一个解析对象，如进入子节点或者父节点的下一个兄弟；.next_siblings、.previous_sibling、.next_elements、.previous_elements：生成器
find_all(name,attr=value)：还支持传入re.compile来搜索tag名，还可以接受返回bool的回调，可设置recursive=False；直接call对象本身相当于调用find_all，find()相当于设置depth=1；find_all_next()、find_next()、find_all_previous()、find_previous()：支持过滤的next_element
tag.prettify(formatter=)：带有缩进的格式化；普通输出：str(tag)；.encode(formatter=)：变为bytes
对于输入时的html实体，进入bs时都会解析一遍，但默认formatter="minimal"在输出时只有&、<、>会再次转化成实体。若用formatter="html5"会尽可能多地把unicode字符变成实体。
但对于输入时href中正常的&，输出时也会变为实体而出错，官方的理由是这样能最大程度保证输出的是有效的HTML。用formatter=None可以避免但是其它地方又不会转换了。
```

## PyInstaller

* pyinstaller opts file.py/file.spec
* -F：打包成单一的exe；默认-D创建dir，结果在dist文件夹中；build文件夹记录了构建过程，warn-xxx.txt记录了出错内容
* -c：使用控制台，默认；-w：使用窗口
* -i：改变icon；-y：重新生成时静默覆盖
* -d：显示debug级别的信息
* --uac-admin：Win下才有效，会申请UAC
* 第一次运行pyinstaller时会生成`.spec`配置文件：exclude是要排除的包；datas是要添加的任意文件，允许通配符；pathex是运行时的工作目录；binary用于添加二进制依赖，如dll
* pyi-bindepend：显示打包后的依赖，pyi-archive_viewer：显示打包后的内容，pyi-makespec：通过命令行选项生成`.spec`
* 最好把它自己装到虚拟环境里，否则结果很大
* 好像如果upx可用就会自动使用，Linux程序还可用-s(strip)
* 控制import的内容能减少大小
* 如果有共同的依赖，有方法合并，但好像有点复杂，且文档未更新

## 参考

* https://zhuanlan.zhihu.com/p/29436838
* https://zhuanlan.zhihu.com/p/55002383
* https://zhuanlan.zhihu.com/p/55002383
* https://zhuanlan.zhihu.com/p/48047813
* https://docs.scrapy.org
* https://zhuanlan.zhihu.com/p/57341319
* https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/
* https://zhuanlan.zhihu.com/p/21976757
* https://zhuanlan.zhihu.com/p/21976757
* https://requests.readthedocs.io/zh_CN

## TODO

* PyTest；自带的叫testsuit
* PySnooper：用于调试
* PyNaCl
* Numpy scipy https://zhuanlan.zhihu.com/p/30011154 https://mp.weixin.qq.com/s?__biz=MzIxMjM4MjkwMw==&mid=2247483920&idx=1&sn=96b11616cf48c83f54ac76c6687a20af https://cs231n.github.io/python-numpy-tutorial/
* Seaborn 数据可视化
* Pandas https://zhuanlan.zhihu.com/c_1142118980302032896 https://mp.weixin.qq.com/s?__biz=MzIxMjM4MjkwMw==&mid=2247483970&idx=1&sn=8028f7582597e0023f0fa02f84db57f1
* 哪些 Python 库让你相见恨晚:https://www.zhihu.com/question/24590883
* https://github.com/dbader/schedule，据说自带的很不好用
* colorama：彩色输出
* https://github.com/docopt/docopt：命令行选项创建工具，也有C#端；https://github.com/pallets/click/ 另一个工具；https://github.com/chriskiehl/Gooey：把命令行程序变成GUI
* https://github.com/harelba/q：Run SQL directly on CSV or TSV files
* gzip
* bokeh
* poetry，替代pip+venv：https://zhuanlan.zhihu.com/p/81025311 https://python-poetry.org/
* Nuitka：https://zhuanlan.zhihu.com/p/133303836 https://zhuanlan.zhihu.com/p/31721250 性能有提高，跨平台差；好像不能单文件
* fastapi：https://fastapi.tiangolo.com/
* https://typer.tiangolo.com/ https://github.com/pallets/click/ 用于解析命令行参数
* pytagcloud 中文分词 生成标签云 https://zhuanlan.zhihu.com/p/20432734
* pyright
* plotly、plotly/dash
* httpx
* https://github.com/grantjenks/python-diskcache
