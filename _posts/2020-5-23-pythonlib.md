---
title: 第三方 Python 库
---

## 环境和打包

### requirements.txt

```bash
pip freeze > requirements.txt # 第三方freeze库：pipreqs能从import产生。pip-tools能从setup或一个.in产生。pigar支持notebook
pip install -r requirements.txt

SomeProject==1.4
SomeProject>=1,<2 # 逗号为且
SomeProject~=1.4.2 # “==1.4.*” that’s also “>=1.4.2”
upper bound限定：不会修改最左边的非0数字，^0.1.11能更新到0.1.19但不会到0.2.0
-e . # 相当于pip install -e .
```

### venv

* 不能脱离本机环境，会在`venv/pyvenv.cfg`中硬编码Python的版本和home位置
* 如果Python in-place升级了版本，可用venv --upgrade .venv，之后记得更新venv里的包(--upgrade-deps)。但如果Python自己的路径变化了，就只能手动改了
* `--system-site-packages`使得虚拟环境可访问系统的包

### Docker镜像

```dockerfile
FROM python:3.11-slim # 不含git
ENV PIP_NO_CACHE_DIR 1
ENV PIP_DISABLE_PIP_VERSION_CHECK 1
ENV PYTHONUNBUFFERED 1
ENV PYTHONDONTWRITEBYTECODE 1

COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . . # 一般配合.dockerignore
```

### 包和模块

* 一个.py文件就是一个模块，模块名`__name__`按目录组织，用点分隔，import时无需也不能加.py后缀
* 一个含有`__init__.py`的目录就是包，import该目录时相当于导入该目录下的`__init__.py`，它的`__name__`等于目录路径对应的模块名
* 对于`x/y/z.py`，`import x.y.z`会依次运行`x/__init__.py`和`x/y/__init__.py`，再运行`c.py`，且只能通过`x.y.z.xx`访问z中的东西（包括z里import的），x的init中声明的变量和import的东西都被限制在x的空间中无法访问（除非z里import了x的）；`import x`无法用`x.y`和`x.y.z`访问y和z，除非x的init里import了
* 模块只初始化一次，所有变量归属于某个模块，import机制是线程安全的，所以模块本身是天然的单例实现。一个函数如果绑定了对应模块内的全局变量，当在别的地方`import *`后修改那个全局变量，函数仍然使用的是原来的变量，与class类似
* python命令行也可运行目录，目标为那**一个**`__main__.py`；运行目标时会把`__name__`变量设为`'__main__'`
* 不用-m会把目标所在的文件夹加到sys.path中，然后按路径直接执行目标，目标就是顶级模块；用-m会把cwd加到sys.path中，按模块名先一层层执行`__init__.py`再执行目标，会先编译成.pyc，会把`__package__`设为模块名的前一部分，cwd是顶级模块；该sys.path与环境变量的path无关，对于环境变量修改PYTHONPATH可更改搜索地点
* 不要自己创建名为`runpy.py`的文件，因为系统存在runpy这个包；site.py类似
* VSC的lint默认是从工作区开始的，在子文件夹中运行存在绝对导入的py时能正常运行，但lint却会报错
* 还存在命名空间包的概念，把多个位置不相关的包算进一个命名空间方便使用
* `runpy.run_module('xxx', run_name='__main__', alter_sys=True)`相当于命令行中-m xxx；不加后两个参数就是在不import那个模块的时候使用它
* `__file__`是当前文件名的绝对路径(3.9+)；命名空间包没有此属性
* 直接运行的目标模块是`__main__`，在其它地方也可以import它，且可用它的`__file__`。但注意通过uvicorn等非直接运行时就不能依赖了
* 没有一种表示“项目根目录”的方法
* `__all__`：在被import *时如果存在此字段，只会导入它指定的，help也只能看到这些。下划线开头变量也不会被导入
* 如果有子目录却没有init文件，在作为系统包时无法import那里面的内容

```py
# 绝对import，以sys.path中的目录开始搜索
import x # 引用顶级目录的x/__init__.py，若不存在再找顶级目录的x.py，再找库模块x；以下将前两者简称x模块
import x.y # 在x文件夹中找y模块，仍会调用x/__init__.py
from x import y # 在x模块中找y对象，若导入*则只可能是这种，若不存在再在x文件夹中找y模块，此时仍会调用x/__init__.py

# 相对import，以当前模块的__package__作为起始位置；顶级模块无法使用，会报ImportError
from . import x # 引用同级目录的__init__.py中的x对象，或同级目录的x模块；不存在import .x
from .x import y # 引用同级目录的x模块中的y对象，或x文件夹中的y模块
from ..x import y # 引用上级目录的x模块中的y对象，或上级目录的x文件夹中的y模块
import ..x # 引用上级目录的x模块

try:
    import simplejson as json
except ImportError:
    import json
```

### 构建前端

* wheel包
* pip wheel . 在当前目录（或用-w outdir）下生成wheel。-vv显示详细过程
* python -m build：相当于setup.py bdist_wheel和pip wheel --no-deps。不过也能--sdist
* pip install .
  * -e 使得编辑源文件后无需install即时生效，仅用于开发，原理是软链接
* python3 setup.py bdist_wheel：生成过程在build文件夹里，生成的东西在dist文件夹里。install生成egg并安装，也会自动安装依赖但不会走pip自定义的源，实际用的是easy_install
* pip download --only-binary :all: 下载包及其依赖方便离线安装
* 上传：twine upload [--repository testpypi] dist/*。pypa/gh-action-pypi-publish
* 检查wheel存在的问题的工具：https://github.com/jwodder/check-wheel-contents
* --global-option "-a" --install-option "-b"相当于setup.py -a install -b。现在global-option被pip废弃了，改用--config-settings
* --no-build-isolation：目前版本的pip在构建时会自动创建虚拟环境，导致即使系统中存在满足依赖的包也不会去使用，此参数禁用这一行为

### setuptools

* 目录结构自动发现
  * src布局：pyproject.toml或setup.py、src/mypkg{`__init__.py`, module.py, subpkg}
  * flat布局：没有src目录，其它不变。但会自动排除某些目录，如bin docs utils test
  * 单模块：根下只有一个.py
  * 手动指定：[tool.setuptools] packages或py_modules = ["mypkg"]
* 显示详细的构建信息：DISTUTILS_DEBUG=1

```py
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"

[project]
name = "mypkg"
#version = "0.0.1"
description = "one-line text"
url= 'homepage'
dynamic = ["version", "readme"]

requires-python = ">=3.8"
dependencies = [
    "requests",
    'importlib-metadata; python_version<"3.10"', # platform_system=='Windows' platform_machine=='x86_64'
]

[tool.setuptools.dynamic]
version = {attr = "mypkg.__version__"}
readme = {file = ["README.md"], content-type = "text/markdown"}

[project.scripts]
cli-name = "mypkg.mymodule:some_func" # 若用proj:_main，用的是init中的对象

包内数据文件 https://setuptools.pypa.io/en/latest/userguide/datafiles.html
①使用MANIFEST.in：`include src/mypkg/*.txt、recursive-include 文件夹名 *`
②[tool.setuptools.package-data] mypkg或"*" = ["*.txt"]
使用包内数据：importlib.resources.files("mypkg")/"data/data.csv"。遍历文件名：importlib.metadata.files("mypkg")，读取：read_text()
数据文件必须要在py包内（有init），否则不受支持，如单个py_modules无法使用
读取dist-info中的版本信息：importlib.metadata.version('mypkg')

---
TODO: setup.cfg
[bdist_wheel] # 对应verb的开关
[build_ext]
```

## Scrapy

* robots.txt中的Crawl-delay指定了抓取延迟
* 通用概念：crawler和spider同义，指通用爬虫，目的是发现URL；scraper是某一网站专用的，目的是提取数据
* 大量依赖，包括Twisted lxml等30多个
* 分析：需要哪些数据、从哪些网站上获取、多久提取一次、如何保证准确、如何消费。法律风险：个人数据、有版权的数据、需要登录的数据
* 难点：限制IP、登录、验证码、复杂的ajax
* Crawler类包括settings signals stats extensions engine spider，属于核心

### CLI

* startproject MyPrj .：产生项目模板。scrapy.cfg所在目录是项目根目录，default的值就是scrapy命令行使用的项目，但此文件基本没用
* genspider [-t crawl] douban url：在MyPrj/spiders中产生douban.py单个文件，类名自动为DoubanSpider
* shell url：交互式爬取指定页面。Ctrl+C退出。如果venv中有ipython，会自动使用
* fetch/view url：根据当前项目的配置（如改了UA）发出请求，内容输出到终端/浏览器上。不会进入REPL
* parse url：获取内容并根据选项进行到哪一步的解析
* check：检查parse的docstring中声明`@url xxx要自动测试的网址; @returns items 2 5最少yield2个最多5个; @scrapes field1返回的item必须有field1`
* crawl spidername -o items.json：进行爬取
  * -o追加，-O覆盖，不加就只会在log中记录
  * items.json中是[{},{}]
  * 支持jl后缀表示jsonlines使得多次爬取没问题，还支持csv
  * -a k=v传命令行参数，默认自动变成Spider的属性
  * -s JOBDIR==crawls/somespider-1：可以随时Ctrl+C，下次再加上相同的参数就能恢复
* runspider xxx.py：支持不在项目中运行单个爬虫。但没必要在这种地方轻量，因为Scrapy已经太重了

### spiders

* request
  * Request(url, callback=self.parse, method, headers:{}, body:str/bytes, cookies:{}) 还可指定errback发送错误时的处理函数，cb_kwargs传给回调的额外参数
  * Request.from_curl()
  * scrapy.FormRequest(url, formdata)
  * scrapy.FormRequest.from_response(response, formcss, formdata, callback) 方便处理input type=hidden
  * scrapy.http.JsonRequest(url, data)
* response
  * body是bytes
  * text是包括doctype和head的str
  * headers.getlist('Set-Cookie')
  * json()
  * url、status、ip_address
* self.state：dict，放需要被持久化的内容
* 回调支持async

```py
import scrapy
from scrapy.http.response.html import HtmlResponse

class MySpider(scrapy.Spider):
    name='xxx' # 必须
    allowed_domains = [] # 允许爬的域名 TODO: 如果为空是都不允许还是都允许？
    start_urls = [] # 起始待爬的链接

    # 还有parse_post
    def parse(self, response: HtmlResponse): # TODO: 确定response有哪些属性
        self.logger.info(xxx)
        val = response.css('xxx')

        yield MyItem(key=val) # 也可返回dict和dataclass

        yield from response.follow_all(response.css('ul.pager a')/css='ul.pager a') # 自动提取所有a元素的href并处理相对路径作为新请求

        # 老式做法
        if next_page := response.css('li.next a::attr(href)').get(): # 单个href字符串
            next_page = response.urljoin(next_page) # 创建绝对路径
            yield scrapy.Request(next_page)

# 先检测是否有更新再下载的做法
def start_requests(self): # 覆盖start_urls
    urls = [...]
    for url in urls:
        yield Request(url, self.check, 'HEAD')
def check(self, response):
    date = response.headers['Last-Modified']
    # 从db中获取上次修改的时间
    if db_date > date:
        yield Request(response.url)

# CrawlSpider通用爬虫，自动跟进链接；不要使用和修改parse()
from scrapy.linkextractors import LinkExtractor # 从HTML中提取a的href。也可以修改tags和attrs属性实现提取任意元素的属性
from scrapy.spiders import CrawlSpider, Rule
class MySpider2(CrawlSpider):
    rules=(
        Rule(LinkExtractor(allow=r'.*/index\.html'/[...], deny=..., allow_domains=..., restrict_css=...), callback='parse_item', follow=True), # follow的默认值，不设置callback时是True，否则是False
        Rule(...)
    )
```

### Items

* 相当于Model
* 自动阻止给未声明字段赋值
* 在spider中使用要自己import
* 自带deepcopy()

```py
from scrapy.item import Item, Field
class Product(Item):
    Price = Field()
item = Product(Price=123)
item['Price'] = 456 # 可当作dict用
d = dict(item)
item2 = Product(d)
item.Price 报错、item['Price2'] 报错

# ItemLoader，用于填充Item
# 但还必须在定义scrapy.Field()的时候指定input/output_processor。itemloaders.processors和w3lib.html中提供了一些工具方法
from scrapy.loader import ItemLoader
def parse(self, response):
    l = ItemLoader(item=Product(), response=response)
    l.add_css('Price', '#price')
    return l.load_item()
```

### Item Pipeline

* 处理item的，可以用来清理HTML数据、验证爬取的数据、查重和丢弃、保存到数据库
* 还需在settings里设置ITEM_PIPELINES才能启用
* 自带scrapy.pipelines.files.FilesPipeline和图像管道，前者设置FILES_STORE，Spider返回时如果有`file_urls`，中间件就会下载文件，并添加files字段，里面有储存到的本地路径等一些元数据

```py
class DoubanPipeline:
    def open_spider(self, spider):
        self.db = 初始化数据库连接
    def close_spider(self, spider):
        self.db.close()
    def process_item(self, item, spider):
        it = itemadapter.ItemAdapter(item) # 能把dataclass包装成类dict一致处理
        self.db.add(it.asdict())
        return item 或 raise scrapy.exceptions.DropItem("Missing price")
```

### Settings

* 使用时可当作dict，不过也有一些方法
* crawler.settings
* 在命令行中指定：-s k=v
* 在Spider中设置：custom_settings={k:v}，读取：self.settings

```py
# 默认设置
USER_AGENT = "Scrapy/VERSION (+https://scrapy.org)"
ROBOTSTXT_OBEY = True
LOG_LEVEL = 'DEBUG' # 命令行用-L指定
CONCURRENT_REQUESTS = 16 # 爬多域名时最好调高
CONCURRENT_REQUESTS_PER_DOMAIN = 8
DEFAULT_REQUEST_HEADERS = { 'Accept': 'text/html, ...', 'Accept-Language': 'en' }
DEPTH_LIMIT = 0 # 不限
DOWNLOAD_DELAY = 0 # 不限，秒数，支持小数，默认还会在此基础上随机乘以0.5-1.5
DOWNLOAD_TIMEOUT = 100 # 秒数。可被Spider和Request.meta的download_timeout覆盖
RETRY_TIMES = 2 # 对于HTTP错误码，只有RETRY_HTTP_CODES中指定的才会重试
HTTPCACHE_ENABLED = False # 启用后会在临时文件夹缓存，不遵循HTTP头的指示
HTTPCACHE_GZIP = False
REDIRECT_MAX_TIMES = 20
COOKIES_ENABLED = true # 有时Cookie会被用来识别机器人，此时可考虑禁用；爬多域名一般也禁用
COOKIES_DEBUG = False # 启用后会把cookie记录到日志中
MEMUSAGE_LIMIT_MB = 0 # 超出设定值后会shutdown，一般还设置邮件通知
AUTOTHROTTLE_ENABLED = False # 启用后就不用考虑DOWNLOAD_DELAY了
AUTOTHROTTLE_TARGET_CONCURRENCY = 1.0
TELNETCONSOLE_ENABLED = True # 一般关闭
SCHEDULER_PRIORITY_QUEUE 默认深度优先，爬多域名推荐用 'scrapy.pqueues.DownloaderAwarePriorityQueue'，这不是完全的广度优先

# 储存数据的后端，支持FTP和S3
FEEDS = {'data-%(time)s.json':{'format': 'json','encoding': 'utf8','store_empty': False}}
```

### Debug

```py
# CWD为scrapy.cfg
from scrapy import cmdline
cmd = 'scrapy crawl MySpider'
cmdline.execute(cmd.split())

# 官方的做法
from scrapy.crawler import CrawlerProcess
from scrapy.utils.project import get_project_settings
process = CrawlerProcess(get_project_settings())
process.crawl(MyCrawler, domain='scrapinghub.com') # 第一个参数是爬虫类本身，也可以用name
process.start()

# parse中启动scrapy shell
from scrapy.shell import inspect_response
if xxx:
    return item
else:
    inspect_response(response, self)
```

### Middleware

* DOWNLOADER_MIDDLEWARES_BASE设置中列出了默认启用的自带中间件
* HttpAuthMiddleware：在spider中设置http_user http_pass http_auth_domain属性，就能自动Basic验证
* 自定义的需要在设置中启用
* Downloader中间件，构造函数接受crawler参数，定义process_request process_response process_exception方法
* 还有Spider中间件

### 相关项目

* https://www.zyte.com 官方平台，所有的Tools都是收费的，只能用Scrapy Cloud，数据最多保留四个月
* https://github.com/Python3WebSpider/ProxyPool 定时抓取免费代理网站的代理池，需要Docker或Python+Redis
* scrapy-redis 分布式爬虫
* https://github.com/scrapinghub/splash 浏览器渲染服务

### Parsel

```py
from parsel import Selector
se = Selector(doc)
se.css('a') # 为SelectorList，可直接进一步.css或.xpath()；也可看作[Selector]
.get() # 返回None或元素对应的字符串，若存在多个则只取第一个
.getall() # 返回[str]
.re(r'xxx') # 返回[str]；re_first()取第一个

css('p::text').get() # 取文本，非标准，不会取到子元素的
css('div *::text').getall() # div的所有子元素的文本节点
css('a::attr(href)').getall() # 取属性
css('a').attrib['href'] # 取属性，只取第一个，可用推导式遍历其中元素再取
```

* 基于cssselect和lxml(.html)，由Scrapy团队开发
* 如果HTML代码有BUG（如标签未闭合）出现了多个根元素，直接用css只会处理第一个根元素
* parsel.utils.flatten()：将多层嵌套的可迭代对象变为list
* parsel.css2xpath()：把css变为xpath
* parselcli库：提供parsel命令行，能repl css处理本地文件和url
* pyquery库：也基于cssselect和lxml，API风格为jQuery的
* Chrome的devtools中对于表格会自动加上tbody

### lxml

* 有时文本节点不会直接为str，这样能判断是不是tail文本，以及找回父节点
* lxml.objectify：像操作Python对象一样操作XML。不能与etree混用，不能用于HTML
* html5parser：把html5lib作为lxml的后端构建etree，用法直接把它传给fromstring的参数即可；还有个soupparser但BS又会用内置的html.parser，所以没有任何意义
* etree对应自带的xml.etree.ElementTree，API基本相同。不要用cElementTree，废弃了。完全不支持自闭合元素
* 自带的html.parser不能直接使用。只有html.unescape()有点用
* lxml-stubs：官方维护，虽然准确，但不全；Pylance自带的几乎无类型，但函数全

```py
from lxml import etree, html

root = etree.Element('root') # 支持取索引和区间，for遍历直接子节点，len()，list()；也支持[]取属性。不带子节点的节点，原版是Falsy但说会在未来改变，lxml不是Falsy
root.append(etree.Element('child1', attrib={})) # 效率更高的函数是c=etree.SubElement(root, 'child1')
c = root[0]; c.tag元素名即child1; c.attrib所有属性的dict; c.text='world'; c.tail某个元素后面的文本
c.getparent()/getprevious()/getnext() # 原版没有
etree.tostring(root, encoding="unicode") # 此处的encoding是避免编码为实体；pretty_print=True格式化
root.iter() # 递归遍历所有子节点，可选指定tag名
root.index(c) # c在root中的位置，原版没有

parser = etree.XMLParser(resolve_entities=False, no_network=True) # 禁用外部实体防止XXE攻击。需验证XMLSchema时不可禁用网络
etree.parse('filename'/filelike, parser) # 返回tree，用.getroot()获得根Element；子元素用.getroottree()获得tree
etree.fromstring('xml literal') # 返回Element，基本相当于etree.XML()
etree.dump(root) # 格式化输出到sysout，应仅用于调试；没有html.dump。原版手动格式化要用xml.dom.minidom.parseString().toprettyxml(indent="  ")
etree.xmlfile() # 类似于open()，用于流式创建xml，原版没有
ET.canonicalize('xml literal', out=filelike) # 以低自由度的方式写入文件，更规范
tree.write('filename', encoding="unicode") # 还有method='html'

html.fromstring() # 尝试同时处理document和fragment
html.document_fromstring() # 有一定处理不完整内容的能力，如没有html和body元素时会添加，会把style放到head中，基本相当于etree.HTML()
html.fragment_fromstring() # 必须是单个元素，fragments_fromstring()

tree/elem.find() findall() findtext() iterfind() # 原版也支持的XPath，一般优先考虑，只用于filter因此不支持text()等特性
doc.body.xpath() # 一般返回[Element]或[str]，原版没有
tree.getpath(e) # 根据tree和e返回xpath。还有个getelementpath()优点是不需要额外的前缀声明，没看懂
p = etree.XPath(...); p(root) # 把XPath编译成可调用的函数
```

### Xpath

* 有层次，如e.xpath(`div`)只会选取e节点的直接子div，相当于`./div`；`*`选择当前节点所有子节点，`*/div`选取孙子节点，node()等于*加文本节点str，测试没有注释
* 能回溯到父节点，`..`为父节点。以`/`开头会从根开始搜索，与当前节点的层次无关。原版最多只支持回溯到当前节点，再..会返回None
* traversal递归下降：`//`从根开始，`.//div`从当前节点下开始，`a//b`从当前节点下的a开始
* 过滤属性：`[@attr]`、`[@attr="val"]`引号不可省、`//*[@id="xxx"]`按id选择、`[@*]`存在任何属性都行；获得属性的值：`./@attr`、`.//@attr`
* 当前节点不含子节点的文本：`text()`，含子节点：`string()`；结果为一个字符串且保留空格，大概就是去掉所有尖括号的内容；一般用//text()获取各个子节点文本的列表
* 取索引：`[n]`，n从1开始。最后一个用`[last()]`，倒数第二个用`[last()-1]`，前两个用`[position()<3]`，与递归一起用一般加括号`(//div)[1]`，否则就相当于:first-child了
* 并集运算符，不重复：`//a | //div`、`(a|div)/span`，但不支持`a/(div|span)`，即要用只能从最外层开始
* 子元素筛选，只支持直接子元素：`div[a]`存在a子元素的div元素，`div[not(a)]`不存在a子元素的div元素，`[a]`存在a子元素的子元素
* 中括号其实是比较一般的条件过滤，可与一些函数结合使用：`[contains(text(), "123") and/or starts-with(@attr, "val")]`
* Axes(轴)语法：加`xxx::`，改变冒号后的意义，不必用在pattern最开头。如`attribute::lang`选取当前节点的lang属性，`ancestor::div`选取当前节点的div祖先。默认是child轴
* 还可进行计算：`+-* div mod >= !=`，也有一大堆函数，包括正则命名空间。但一般来说不用，因为不知道实现情况如何
* 嵌入变量：在调用xpath()时传参k=v，在pattern里用$k获取
* 不适合过滤class，因为可能由空格分隔

## Requests

* Session能连接复用以及保留cookie
* 即使使用了Session，方法级别的参数也不会保留
* 非线程安全，toolbelt提供了简单的多线程
* requests-cache库：透明持久化缓存，支持多种后端

```py
s = requests.session() # with或s.close()能关闭所有连接，但之后仍可以继续使用，又会自动创建。一般用于出现异常时及时释放资源
s.request = functools.partial(s.request, timeout=3) # 设置超时时间，不加会一直等，Session级别的只能这样做；同urllib3也可传递元组，第一个参数控制连接超时，第二个参数控制下载
allow_redirects=True; max_redirects=30 #【默】最后结果是200不是3xx；head默认不跟踪
proxies={"http": "http://10.10.10.10:1080", "https": "http://xxx"} # 默认支持HTTP_PROXY，不直接支持socks
headers={'User-Agent':'python-requests/2.23.0','Accept-Encoding':'gzip','Connection':'keep-alive'} #【基本默】大小写不敏感的dict
auth=('user', 'pass') # Authorization头，如果不放到session里，重定向时会自动去掉
cookies.set(k,v,domain,path) # 类型是RequestsCookieJar，但也可以传dict。另有requests.utils.add_dict_to_cookiejar(cj, cookie_dict)、cookiejar_from_dict、dict_from_cookiejar几个函数；有可能第一次能用dict，之后就要用它们了，不能直接update
```

### 请求和响应

* url必须要有scheme；必须每次写完整url，要不就用requests_toolbelt提供的BaseUrlSession
* get的params会自动变成查询参数，且值为None的不会附加上去
* post的data和json传dict（json还可以是list）会自动编码并设置Content-Type，前者是form
* 传字符串给data是设置body，不要传字符串给json；data还支持filelike且支持流式处理，文件记得以rb打开；data还支持生成器，则会传输分块编码
* post支持files={'filefield': filelike-bin}，requests-toolbelt提供了更多功能
* RFC 2616规定如果Content-Type没指定编码且类型是text/*，那就用ISO-8859-1；又不过RFC 7231去掉了这个限制

```py
r: Response = s.get(url,params={k:v})、post(url,data/json = {k:v}/str)、put/delete/head/options
r.raise_for_status(), r.status_code, r.ok # 后两项分别为200和True
r.json() # 即使解码成功也不一定意味着请求成功，因为有时服务器会在失败时也返回json
r.text # 根据encoding解码的HTTP内容字符串
r.encoding # 可赋值，一般在它等于'ISO-8859-1'时赋r.apparent_encoding
r.content # 二进制，但仍会自动解码gzip，适用于图片等
r.url, r.history # 前者包含查询参数，后者为重定向响应列表
r.headers # 响应头部，可用r.request.headers访问请求头部

# 流式API，get返回时只下了Header，可以进行一些处理，直到访问content才会下载响应体
with requests.get(url, stream=True) as r:
    if r.encoding is None: r.encoding = 'u8'
    for line in r.iter_lines(decode_unicode=True):
        if line: # filter out keep-alive new lines
            print(json.loads(decoded_line))

    with open(filename, 'wb') as fd: # 保存二进制文件的推荐方式
        for chunk in r.iter_content(chunk_size): # 默认为1，但content和iter_lines另有默认值
            fd.write(chunk)

# 缓存，默认是保存在内存中的dict，还支持redis；另有requests-cache库星数更高但不是requests官方推荐的
from cachecontrol import CacheControl
cached_se = CacheControl(requests.session()) # 指定文件缓存：cache=cachecontrol.caches.FileCache('.webcache')
# 一些建议：params最好sorted一下、缓存的响应永远不要流式处理
```

### urllib3

* urllib3.request('GET',url,fields={'k':'v'}); r.data.decode()
* POST和PUT：fields自动编码为body，类型默认为multipart。上传文件：fields={'filefield':('filename', str/bytes [,"text/plain"])}。上传二进制内容：设置body参数和Content-Type
* Headers：UA默认为python-urllib3/1.26.8。PM和request()的headers相当于对它`|=`，但后者若存在会则会完全替换pm的
* 支持自动gzip解码，但默认AE是identity，且Content-Length是解压前的
* 默认retries=3，重定向3次，timeout无限
* 流式处理，可看成io.BytesIO，超时异常在read()处发生：preload_content=False; resp.read(4); resp.release_conn()
* PoolManager：管理ConnectionPool，默认最大10个池
* ConnectionPool：一个域名对应一个，默认maxsize=1只长连接一个，更多的能连接但不会保留长连接，设置block=True可阻止更多连接，这俩参数也能在PM的构造函数中使用。一般用connection_from_url()创建，pool.request(这里的url部分可以是相对路径)
* 不会自动使用HTTP_PROXY

### urllib

* 自带，但urlopen明确不支持keep-alive，无法大量使用。结束时会发RST而不是FIN
* 底层是http.client
* UA默认为Python-urllib/3.9
* POST x-www-form-urlencoded：给urlopen或req传data=parse.urlencode(dict).encode('ascii')，此方法一定程度上也能用于构建GET的查询参数字符串
* 似乎没有办法做出浏览器的URL编码的方式：把空格编码为%20，把中文用UTF8编码后每个加上%，其余的特殊字符不变。urllib3 requests不会对URL自动编码
* 支持HTTP_PROXY
* 默认超时20秒

```py
req = urllib.request.Request(url, headers={...})
with urllib.request.urlopen(req/url) as resp # 返回类型是个无意义的私有变量无法自动推断，经测试是http.client.HTTPResponse
text = resp.read().decode();
resp.getheader('xxx')/getheaders();headers.xxx()有少量提取charset和contenttype等内容的函数且是dict-like且大小写不敏感
resp.getcode()、resp.info().get_content_charset()
urllib.request.urlretrieve(url, outfilename) # 直接下载为文件

urllib.parse：
quote() 用于编码?k=v中的v，会编码所有特殊字符除了斜杠；逆过程为unquote()；还有quote_plus()会把空格编码为+而非%20
parts = urlparse(url) 解析成六部分的命名元组，如parts.netloc域名，修改后可unparse
parse_qs(被urlencode的数据) -> dict
urljoin() 处理相对路径
第三方库：yarl、furl
```

### 其它HTTP库

* httpx
  * api与requests差不多，且支持异步、h2、brotli
  * 长连接用httpx.Client()
  * 默认不自动30x跳转
  * cookie在Client上，不能在请求方法上
  * stream有单独的方法
  * post二进制用content参数，form用data参数
  * 底层用的是同作者的httpcore，依赖h11 sniffio anyio certifi
* requests-html基于bs、pyquery、pyppeteer等构建，超级重，支持asyncio，render()自动用chrome请求ajax，第一次用会下载驱动
* httplib2：和urllib3差不多级别的API，活跃度不高，可用于Py2
* faster-than-requests：新，无依赖，速度快，非纯Py，贡献者极少
* h11：底层库，和http.client同级别
* python-trio/hip：urllib3的fork，添加了异步，开发很早期，没有使用的价值

## Beautiful Soup

* 支持不同的HTML Parser：html5lib最接近真实网页，是纯Python，相对慢；lxml(默认就是lxml.html)容错性中等，速度最快；标准库html.parser容错性差，但现在好像还行
* HTML分为四种对象：bs4.BeautifulSoup（文档）、bs4.element.Tag（标签）、bs4.element.NavigableString（文本）、bs4.element.Comment（注释）；XML还有其他对象
* 有的属性是多值属性，如class，bs会自动处理成list（xml不做处理）。但像id中即使有空格，也只会直接返回字符串
* 支持修改，许多东西可以直接赋值和`del`删除，有一些修改树的方法
* 支持`==`判断结构相同
* BS不支持但存在的解析库
  * html5-parser 基于c，支持解析成lxml和BS的对象，但完全不支持whl，必须连lxml也要用动态链接
  * selectolax 是另外两个c后端的Py绑定，自己的API，支持css选择器

```py
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

* 应在虚拟环境中使用，它自己也装进去
* 默认构建结果在dist文件夹中，build文件夹记录了构建过程，warn-xxx.txt记录了出错内容
* 第一次对入口点文件使用`pyinstaller main.py`，会生成main.spec，之后就对该spec使用
* 会把.py编译成.pyc放到一个文件中，文件夹模式下如果没更新依赖，重新部署就只要更新那个文件即可。不跨平台，不支持交叉编译
* spec
  * datas：资源文件，是个`[('src','dest')]`，其中若src是文件夹而dest是`.`就会解包一层。src允许通配符。有`PyInstaller.utils.hooks.collect_data_files('mylib')`自动添加模块中的非.py且非二进制文件。没有exclude_data的功能
  * binary：二进制依赖，如dll
  * hiddenimports：添加没自动分析出来的模块引用。全添加用`hiddenimports = PyInstaller.utils.hooks.collect_submodules('xxx')`
  * excludes：模块名列表。排除后仍能导入未打包进去的模块
* data
  * CWD的相对路径不变
  * `__file__`意义基本不变，单文件模式下会放到临时文件夹中
  * sys.executable和sys.argv[0]：单文件下是加载器的路径，前者还会解软链接
* -y：再次生成时静默覆盖之前构建的内容
* -Fw：单文件+使用窗口；-i file.ico/exe：改变icon，Linux无效
* --log-level DEBUG：更详细的编译期信息。-d all：原样复制.py，不处理成pyc和打包，运行时对解释器传递-v显示加载模块的详细信息
* 有一定的增量生成功能，但在最初的测试阶段存在依赖问题时最好用--clean重新生成
* --uac-admin：Win限定，会申请UAC。--uac-uiaccess不懂有什么区别，文档说与远程桌面有关。单文件模式不推荐使用管理员权限
* pyi-bindepend：显示打包后的依赖；pyi-archive_viewer：显示打包后的内容
* upx如果在Path里会自动使用，Linux程序还可用-s选项strip
* 会在 %LocalAppData%\Packages\PythonSoftwareFoundation.Python.3.9_qbz5n2kfra8p0\LocalCache\Local\pyinstaller 中产生垃圾文件
* 使用multiprocessing时要调用freeze_support()，但好像它已经patch过了
* pywin32-ctypes：用纯Py重新实现的pywin32，但只有一小部分API
* 报错LoadLibrary找不到python312.dll：不要运行build里的文件，运行dist里的

### 其它打包项目

* PyOxidizer：开发处于早期，只支持3.8+，编译，单文件模式能不释放到临时文件夹
* cx_freeze：扩展了setup.py，也可用简单的命令行。不支持单文件
* Nuitka：编译到C，速度快，可能有兼容性问题，不够成熟。https://zhuanlan.zhihu.com/c_1245860717607686144
* shiv：类似于zipapp创建pyz，能打包依赖，不包含解释器，Linux下借助shebang能看起来直接运行，Win下要用`py`
* pex：类似于可移植的pip+venv，感觉没必要学
* py2exe：Star少，贡献者只有10人，不学
* https://docs.beeware.org 一系列项目，能产生msi

## python-fire

* 功能简单，几乎就是链式调用，类似于把空格变成点。能把函数、类中的函数、模块中的函数（实际上都是对象）变成cli工具
* 选项参数名字中的横杠和下划线等价
* 支持`*args`或`**kwargs`
* 分隔符`-`显式告知之后的为子命令而非参数，例如`- upper`可以把结果变为大写
* `python -m fire xxx`可以对目标模块做任何改变而使用
* BUG：对于对象，--help/-h无法显示verb，必须要用`- -h`才行

```py
import fire
def hi(name='world'): return 'hello'+name
fire.Fire(hello) # python cli.py world或--name=world

class Calculator: # 支持继承
    def __init__(self,offset=1): self.o=offset # 构造函数中的变为Flag；可访问属性o
    def add(self,a,b): return a+b
    def multi(self,a,b): return a*b
fire.Fire(Calculator) # python cli.py add 1 2；python cli.py o --offset=1
```

## IPython

* `In[]`和`Out[]`数组记录了每一次的输入和输出；最后三次输出保存在`_`,`__`,`___`中，Out[n]也等价于`_n`
* 退出用Ctrl+D，python在Win下用的是Ctrl+Z；支持Ctrl+R的类似bash的历史搜索
* 传递参数给文件或模块要用`--`，否则会被认为是传给ipython自己；而python不对`--`特别对待
* 交互式输出对象默认使用pprint
* 普通代码中用IPython.embed()进入IPython环境，但只是进入REPL，也不会读取设置，但对当前会话做的修改等返回到原有Python会话中时能保留；start_ipython()是普通的启动IPython的方法，会读取设置
* 自动括号和引号：`/fun 1 -> fun(1)`，`/fun 1,2 -> fun(1,2)`；`,fun a b` -> `fun('a','b')`；`;fun a b` -> `fun('a b')`
* 默认输入时自动忽略`>>>`和`...`，用于方便输入含有交互式提示符号的语句；doctest_mode专门进入测试模式
* 能直接使用exit()
* get_ipython()不为None就是在ipython中
* ctrl+space不是自动完成，而是进入选择模式，受emacs的影响

### [魔法命令](https://ipython.readthedocs.io/en/latest/interactive/magics.html)

* 单个%是行魔法，回车就执行，默认开启了%automagic，使得无歧义时不加%也能执行命令；两个%%是cell魔法
* 命令的结果可以赋值给Python变量，此时无法省略%，另一种方式是%%capture var。命令的参数支持用`$`或者大括号嵌入Python变量，用`$$`转义一个`$`到shell
* quickref：显示所有命令的简要参考；lsmagic：显示所有支持的命令；?加命令：显示指定命令的参考
* timeit：统计语句运行的时间，自动多次运行取最好的5次结果，-n指定运行次数，有%%；单次运行是time；prun显示每个语句的执行时间，有%%
* who：显示定义了的所有变量，后可跟筛选的变量类型，whos信息更丰富，who_ls返回列表
* run test.py/ipynb：-i继承当前会话的变量，-d启动调试，-t计时，-p启动Profiler，-m调用模块（参数仍要--）
* debug：在刚刚出现过异常后使用，能进入ipdb检视刚才的异常栈；当然也可以一开始就用，设置断点；支持%%，接下来输入代码就可以debug
* edit：启动编辑器来输入交互式代码，关闭后会传到cli里；VSC会自动加一个\n，没啥好办法
* store：持久化储存变量，store -r恢复
* hist：查看历史命令，-n加上序号，-g pattern用grep搜索
* xdel：删除变量并试图清除在其对象上的一切引用
* pip：可以直接运行pip
* macro name n1-n2 n3：把n1到n2及n3这几行代码命名为name的宏，之后输name即可，但那时就不显示具体输入内容了，可用store储存，用edit编辑
* pastebin：自动把东西上传到GitHub的gist里并返回链接
* alias：显示预定义的shell命令，也可设置自己的，不过直接设置不能持久化；rehashx：把path里的可执行文件都导入alias中，使用时就不用加叹号了
* pycat：语法高亮地显示文件
* bookmark、pwd、pushd、popd、dhist、ls、cd：与路径有关的一些操作。不能用!cd因为那些程序执行完就终止了
* env：显示或设置环境变量。load_ext dotenv;dotenv：配合python-dotenv库
* !xxx：执行shell命令，等价于system()，但Win下默认是CMD，且可能有编码问题；!!：捕获输出，看起来!赋给变量也是一样的，返回列表一行一条
* ?加命令：显示docstring但与help()的格式不同，且不会显示函数文档，只显示函数名；??两个问号：还会显示源代码
* ?加带*的对象名：显示匹配的对象名；其实是psearch命令
* save：把指定的行保存到文件中、load把目标文件的内容输进终端且不自动执行、recall把上一次的输出(_)输进终端中且不执行、reset -f清除所有定义了的变量、%%writefile将本单元格保存到文件中、paste粘贴并执行、rerun：重运行指定指定行的代码
* load_ext autoreload; autoreload 2修改源文件后会自动重载，autoreload 1修改通过aimport a,b的文件后自动重载；对C模块无效

### 配置

* ipython profile create [profilename]：创建`~/.ipython/profile_default/ipython_config.py`
* 在`profile_default/startup/`中的.py或.ipy会自动执行，命名可以`10-xxx.py`这样含有优先级
* config命令加不含c.的设置项可以动态读取和设置值

```py
c.TerminalInteractiveShell.confirm_exit = False
c.TerminalInteractiveShell.editor = 'code -w'
c.InlineBackend.figure_format = 'svg' # 矢量图；或用retina表示高像素

# 未更改的设置
c.InteractiveShell.autocall = 0 # 自动括号，设为1时是智能模式，2是完全模式
c.InteractiveShell.logstart = False # 启用后会保存会话，下次就会恢复；但默认是overwrite模式？
c.InteractiveShell.pdb = False # 控制是否出现异常时自动进入ipdb，可用%pdb开关
c.InteractiveShellApp.exec_files/.exec_lines/.extensions = [] # IPython启动时要执行的文件/代码/IPython扩展（load_ext）
c.StoreMagics.autorestore = False # 开启后store能自动持久化
```

## pdb

* VSC调试用的并不是pdb，故先不学了
* python -m ipdb xxx.py
* 输入单个问号能显示功能命令
* Debugging a broken unit test: pytest ... --pdbcls=IPython.terminal.debugger:TerminalPdb --pdb
* breakpoint()：内置，进入pdb。相当于`pdb.set_trace()`？
* 命令教程：https://zhuanlan.zhihu.com/p/37218789 https://zhuanlan.zhihu.com/p/43846098 https://realpython.com/python-debugging-pdb/
* 还有个pdb++(pdbpp)项目
* pdb：w/where 打印调用栈

## jupyter

* pip install jupyter 目前7正在开发，安装时加--pre。之前只装notebook的依赖较少，现在仍会装上jupyterlab。官方还在开发基于wasm的jupyterlite，纯网页端的Lab，但运行Py效率太低了。VSC只要装ipykernel即可
* jupyter notebook --no-browser; jupyer run xxx.ipynb
* 会往`%AppData%\jupyter`里写东西，但在商店版Python里会装到隔离区里
* Docker映像：https://jupyter-docker-stacks.readthedocs.io/
* %%html、%%js、%%bash：将cell的内容渲染成HTML输出、运行JS/bash；IPython.display.IFrame/Image/Vedio/Audio能指定网址或路径嵌入内容，但这些是从本地发起的请求而不是服务器，width一般设为"100%"，HTML/Javascript/JSON能运行相应内容，FileLinks('.')类似于tree且会生成可点击的链接

### 配置

* jupyter notebook --generate-config 在 ~/.jupyter/jupyter_notebook_config.py 下生成默认配置。jupyter server也有此命令
* c.ServerApp.root_dir：启动目录、运行时候的根目录
* c.JupyterNotebookApp.open_browser = False
* c.ServerApp.ip = '*'; c.ServerApp.port = 8888
* 添加密码：c.NotebookApp.allow_password_change=False，命令行jupyter notebook password输入密码（无回显）会在.jupyter中生成一个json，设置c.NotebookApp.password为里面的值即可 TODO: 废弃了
* 在反代之后需配置`c.ServerApp.allow_remote_access`或`c.ServerApp.allow_origin`，否则会报`Blocking Cross Origin API request`或`Blocking request with non-local 'Host'`；`/api/kernels/`和`/terminals/`需要配websocket，各种配置中都设置了`Host`

### 快捷键(H显示所有)

* F在上方添加代码块，B在下方，DD是删除，X是剪切，C是复制，V是在下面粘贴，Z撤销删除
* J上移聚焦代码块，K下移，Shift-K选取扩展到下一个代码块，空格向下滚动，Shift-空格向上滚动
* Ctrl-Enter运行当前代码块，Shift-Enter运行当前并移动到下一块，Alt-Enter运行当前并在下方添加代码块
* Enter编辑当前块，Esc返回一般模式（命令模式）
* F查找和替换，S保存
* M把当前代码块的类型改为MD，Y改为Code
* Shift-M把选中的多个块合为一块，Ctrl-Shift-减号为编辑模式下从光标处分隔成两块
* 编辑模式（用Monaco编辑器时不同，如谷歌Colab）：Tab补全，Shift-Tab提示文档（多按几次更详细），Ctrl-D删除整行
* Kaggle：Z撤销，Shift+Z重做。选中一段内容后用鼠标点执行能只执行片段，但撤销时有bug

### 扩展

* 管理扩展的东西：pip install jupyter_contrib_nbextensions jupyter_nbextensions_configurator; jupyter contrib nbextension install --user
* https://github.com/mwouts/jupytext 同时生成py且修改时能双向同步
* 主题：https://github.com/dunovank/jupyter-themes jt -l列出，jt -t xxx切换，jt -r恢复
* 格式化
* Qgrid：提供对标题筛选的功能。qgrid_df=qgrid.show_grid(df, show_toolbar=True); qgrid_df.get_changed_df()将筛选后的数据取出来
* Hinterland：每次输入都有提示

### Binder

* `https://mybinder.org/v2/gh/<user>/<repo>/HEAD?urlpath=nteract`
* repo2docker构建时自动处理requirements.txt apt.txt postBuild（只会运行一次的脚本） start（相当于EntryPoint） runtime.txt（指定Python-3.9）；或只处理Dockerfile

## Conda

* conda是一款软件管理软件，相当于windows里面的应用商店。miniconda和anaconda中都包含了conda。miniconda只包含了conda、python、和一些必备的软件工具；anaconda包含了数据科学和机器学习要用到的很多软件
* pip只用来安装python的whl和源码，后者有时需要编译器，有的需要操作系统的包管理器安装依赖
* conda用来安装conda package二进制包，大部分是python的，但也支持了不少非python语言的依赖项如mkl、cuda这种c/c++写的包；可以安装gcc且不需要root权限；有些包只能用conda，比如rdkit；但包的总数远少于PyPI
* conda自己可以用来创建虚拟环境，可以很轻松地管理多个版本的python
* conda会检查当前环境下所有包之间的依赖关系，比pip更严格
* conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
* conda create -n myvenv python=3.11 -y; conda info -e; conda activate myvenv; conda remove -n myvenv --all
* mamba用c++重新实现了一遍conda

## Web Server

### FastAPI

* /docs和/redoc能查看api文档，还能进行测试
* 如果路径中需要出现后缀，如`/test.txt`，路径需要声明成`{file_path:path}`
* bool查询参数会自动转换，b=1或者b=True或b=yes都可以

```py
app = fastapi.FastAPI()

@app.get("/", summary='xxx', description='xxx')
async def read_root(): return {"Hello": "World"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: Optional[str] = None): # 自动检测非路径参数；如果不加默认值就必须有
    return {"item_id": item_id, "q": q}

# 查询参数校验；如果不想设默认值但又为必要参数，设为...；还可设置title和description；Path对应路径参数校验，Body对应请求体
q: str = Query('default', min_length=3, max_length=50, regex="^fixedquery$") # 数值可设定gt和lt

from pydantic import BaseModel,Field
class Item(BaseModel):
    name: str
    price: float = Field(..., gt=0)
@app.put("/items/{item_id}")
def update_item(item_id: int, item: Item): # 自动把非路径参数从body中提取；支持Enum
    return {"item_name": item.name, "item_id": item_id}
```

### Starlette

* 也支持`@app.route`，但官网没这么写
* 不以`/`结尾的路由会自动重定向，`/{xxx}`时/后必须有内容否则不会匹配，不匹配时返回Not Found文本
* Route还可以设置name，之后可用request或app.url_for获取那个名字的url
* 使用类：继承starlette.endpoints.HTTPEndpoint，定义get等方法
* 自带一些中间件：gzip、httpsredirect
* Config封装了.env的读取
* taoufik07/responder是一个基于Starlette的类似于Flask的框架，但依赖太多，这么重不如用别的框架，也不活跃
* TODO：https://www.starlette.io/schemas/

```py
from starlette.applications import Starlette
from starlette.requests import Request
from starlette.responses import PlainTextResponse,HTMLResponse,JSONResponse,RedirectResponse,FileResponse
from starlette.routing import Route, Mount
from starlette.staticfiles import StaticFiles # 此项及FileResponse需要装aiofiles

Request: path_params从路由中取出定义的变量, url是string-like对象能取出一部分如.path, query_params多值字典, cookies, body()是bytes, json(), form()['filename'].read(), stream()用async for chunk in消费

def homepage(_): ...
def user(request: Request):
    username = request.path_params['username']
    return PlainTextResponse('Hello, %s!' % username) # .headers, .set_cookie()；必须返回Response，不能直接返回str或dict

routes = [
    Route('/', homepage, methods=['GET']), # 默认只接受GET，实际如果路由存在永远可用HEAD
    Route('/user/{username:str}', user), # 默认str(以.和/分隔)，还可指定int,float,path(剩余的所有部分)
    Mount('/users', routes=[ # 指定共有前缀的路由，可在子模块中定义好routes=Router(...)再import进来
        Route('/', users),
        Route('/{username}', user),
    ]),
    Mount('/static', StaticFiles(directory='static', html=True)), # 也可单独使用赋给app
]

app = Starlette(debug=True, routes=routes) # 开启debug客户端也会收到traceback，否则客户端只有Internal Server Error；还可注册on_startup事件，以及设置exception_handlers指定发生错误时的路由
app.state.ADMIN_EMAIL = 'xxx' # 全局状态，请求中访问用request.app；另外单个请求的状态还可以用request.state

# 测试：之后主动调用此函数，会自动启动app
from starlette.testclient import TestClient
def test_homepage():
    with TestClient(app) as client: # 就是requests.Session
        response = client.get("/")
        assert response.status_code == 200

# 手动模拟FileResponse，必须先全部读完再包装成filelike，且不具有ETag和Content-Length且不会自动猜测media_type
def myfile(_):
    with open('test.txt', 'rb') as f:
        return StreamingResponse(io.BytesIO(f.read()))
```

### ASGI和Uvicorn

* pip install uvicorn：依赖click和h11，[standard]还会装上uvloop（Win不支持）等数个依赖
* uvicorn main:app --host 127.0.0.1 --port 8000：【默】对应main.py的app对象，--reload最小版为轮询默认监视CWD
* --uds指定unix socket，--workers多线程
* --log-level默认info。无法从命令行指定日志文件，客户端不会收到traceback
* 默认启用--proxy-headers处理来自于127.0.0.1的X-Forwarded等头，信任所有用--forwarded-allow-ips '*'
* scope：scheme(https)、method(GET)、path(以/开头，不含域名和查询字符串，百分号编码)、headers((k,v)列表，bytes)、query_string(bytes，百分号编码)、client(有ip)，没有原始uri
* abersheeran/a2wsgi：ASGI于WSGI的app互转
* 默认是http的，直接请求https会报h11._util.RemoteProtocolError: illegal request line，curl为SSL_ERROR_SYSCALL
* 只支持HTTP1.1，直接基于asyncio。hypercorn支持HTTP/2，Daphne依赖twisted。granian是rust实现的效率很高

```py
async def app(scope, receive, send): # 必须是异步的，也可以是定义了__call__的类
    assert scope['type'] == 'http' # 不处理WebSocket和Lifespan
    assert scope['method'] in ('GET', 'HEAD')

    message = await receive()
    body = message['body']
    assert message['more_body'] is False # 不处理TE

    await send({ # 必须要有start；为HEAD会自动不发送body
        'type': 'http.response.start',
        'status': 200,
        'headers': [
            (b'content-type', b'text/plain; encoding=utf-8'),
        ],
    })
    await send({
        'type': 'http.response.body',
        'body': b'Hello, world!',
    })

if __name__ == "__main__":  # 若与app在同一文件中，当心全局变量二次执行，如打开了文件未关闭，因为下一句内部重新运行了main.py。可以把那些内容放在else里；或传app对象
    uvicorn.run("main:app", reload=True)

class App: # 使用类定义的方式，指定运行目标是类名而不是实例；也可以定义一个只有scope的函数，返回参数为receive和send的异步函数
    def __init__(self, scope): ...
    async def __call__(self, receive, send): ...
logging.getLogger('uvicorn.error' if 'uvicorn' in __import__('sys').modules else __name__)

# 使用Starlette简单封装uvicorn请求
request = Request(scope, receive)
response = Response(content)
await response(scope, receive, send)
```

## WSGI

* env是一个dict。它包含os.environ的副本。还加了REQUEST_METHOD、REMOTE_ADDR，除了Content-Type和Length以外的HTTP头以HTTP_的方式提供
* body：env['wsgi.input'] 是filelike
* wsgiref.util.request_uri(environ, include_query=True)
* 服务器：gunicorn、uwsgi（pypi上叫pyuwsgi，不原生支持Win）、phusion/passenger（C++写的，好像带有监控功能）

```py
from wsgiref.simple_server import make_server
from wsgiref.types import * # 3.11，目前几乎无用

def app(env:WSGIEnvironment, start_resp:StartResponse, /) -> Iterable[bytes]:
    resp_headers = [('Content-type', 'text/plain; charset=utf-8')]
    start_resp('200 OK', resp_headers)
    return [b'hello world']

from wsgiref.validate import validator # 中间件，检测一些常见的实现错误
app = validator(app)

with make_server('', 8000, app) as httpd:
    httpd.serve_forever()

wsgiref.headers.Headers：方便创建返回的header，类似于dict，大小写不敏感，基本没用
wsgiref.util.FileWrapper(filelike, 分块大小默认8KB) 将文件对象转换为Iterable
```

## ORM和数据库

* SQLite和Python DB-API见主笔记
* pymssql：connect(host=server, user, password, database="tempdb", autocommit=True)。不支持LocalDB
* peewee
* https://github.com/pudo/dataset：基于sqlalchemy
* SQLAlchemy：等2.0
* https://github.com/marshmallow-code/marshmallow 能把类序列化成json，也有校验功能，但标记类型要用该库提供的
* jsonpickle：把任意类序列化成json
* GINO：目前只支持pg
* https://github.com/encode/orm：基于SQLAlchemy core的查询、databases的异步、typesystem的类型验证，但很不活跃
* ponyorm、tortoise-orm
* tinydb：储存数据到json中，用的并不是sql，看作增强版的dict吧，纯Py
* mashumaro：基于dataclass的序列化库，不够成熟

### MySQL

* pymysql：纯Py，当用gevent或者PyPy时可以用
* mysql-connector-python：Oracle官方实现，之前是纯py，性能貌似比pymysql还要差；现在默认启用C扩展；https://dev.mysql.com/doc/connector-python/en/connector-python-coding.html https://dev.mysql.com/doc/x-devapi-userguide/en/devapi-connection-concepts.html https://dev.mysql.com/doc/dev/connector-python/8.0/tutorials/connection_pooling.html https://dev.mysql.com/doc/dev/connector-python/8.0/
* mysqlclient：带有C扩展，性能最好

### Psycopg

* pip install psycopg[binary,pool]。binary是必须的，除非系统中装了libpq5
* 参数化查询：%s，命名参数：%(id)s。一般都用%s就行，强制二进制用%b。两个%%转义出一个。不能用于表名或字段名，因为绑定在服务端处理，表名要用psycopg.sql.SQL(...).format()在客户端处理。还不支持IN %s，可用=ANY(%s)代替
* 将结果作为字典：连接时或创建游标时指定row_factory=psycopg.rows.dict_row或namedtuple_row。还可以指定class_row(dataclass类)自动创建实例
* 支持异步和连接池
* with con是用来关闭连接的，与2和sqlite3不同，后俩是关闭事务，当然前者也会自动完成事务。3可用with conn.transaction()来开启事务块

```py
import psycopg # 名字就叫这个而非psycopg3
with psycopg.connect("dbname=xxx user=xxx") as con:
    for record in con.execute('select * from tb1 where id = %s', (xxx,)): ...
```

### peewee

* CharField, IntegerField, DataField, DataTimeField(default=datetime.now)
* null=False, index=True, unique=True, primary_key=True
* 线程池版连接在playhouse.pool中，也是本包的一部分

```py
import peewee as pw
db = pw.SqliteDatabase('data.db', pragmas={'journal_mode':'wal', 'synchronous':1})
mysql_db = MySQLDatabase('dbname', user='app', password='xxx', host='10.1.0.8', port=3306)

class BaseModel(pw.Model):
    class Meta: database = db
class Person(BaseModel):
    name = CharField(verbose_name='姓名', max_length=10)
    class Meta:
        table_name = 'people'
class Pet(BaseModel):
    owner = ForeignKeyField(Person, backref='pets')

# db.connect(reuse_if_open=True) # 默认使用时会自动连接；用完了可close()
db.create_tables([Person, Pet])
# 操作对象
p = Person.create(...) # 这样会自动保存
p.save()
p.delete_instance()
p = Person.get(Person.name == 'xxx')
# 操作数据；fn().XXX可调用任何SQL函数，后可跟.alias()命名
query = Person.select().where(Person.birthday.between(...) | (...)).join(Pet, JOIN.LEFT_OUTER) # 也可用get()获得单条数据；可用.prefetch(Pet)替代join
for p in query: print(p.pets.count())
Person.delete().where().execute()
Person.update({Person.name: ...}).where().execute()
```

### walrus

* redis-py的wrapper，peewee的作者
* 基于lua脚本实现了Array，对于索引更友好
* BloomFilter
* 搜索“自动完成”的建议
* 全文搜索
* cache支持装饰器

```py
from walrus import *
db = Walrus()
db['walrus'] = 'tusk' # 普通情况下像dict
l = db.List('names')
z = db.ZSet('z1')
z.add({'huey': 3, 'mickey': 6, 'zaizee': 2.5})
z['huey']; z[:'mickey']; z[-2:]; z[-2:, True] # 分别为取/赋、比mickey小的Key、两个最大的的Key、且返回它们的权重
```

### pyodbc

* 驱动
  * Access：Win自带mdb的32位Jet驱动。2007及之后的accdb格式需要单独下ACE驱动，最新的为2016，或者装Office
  * MSSQL：Win11自带32位和64位的SQLServer2000和2012驱动，没啥用，估计LocalDB都连不上
  * 另一种选择是装ODBC驱动，还有Linux和macOS的版本
  * Access驱动的32/64位必须与Py对应，且若系统中有Office也要对应，不同位的Access创建的mdb也无法混用
  * 列出所有可用驱动：pyodbc.drivers()。或开始菜单搜索odbc
* autocommit默认为False，但只是cursor层级没有，cnn层级是有的，connect的时候打开，cnn.commit时提交，with connect也会提交
* cur.fetchval()：非标准API，适合返回单一值，等于fetchone判断不为None再取0
* row可直接按名称访问列
* 编码好像不需要改，从2000开始字符串就是用的Unicode储存
* 本来Access的like用?表示一个字符，*表示0或多个字符，#表示一个数字；但本模块要用_和%
* 不支持命名参数查询
* pypyodbc：纯Py实现，能直接用win_create_mdb()和win_connect_mdb()指定文件名连接，默认自动使用系统的驱动；当初兼容pyodbc，但后来长时间不积极维护已经变得不那么兼容了；支持2.6导致完全没有type hint

```py
import pyodbc
cnnstr = (
    'DRIVER={Microsoft Access Driver (*.mdb)};'
    'DBQ=./data.mdb;' # 不加./可能出现Not a valid file name. (-1044)的问题
)
cnn = pyodbc.connect(cnnstr)
cur = cnn.cursor()
for row in cur.tables(tableType='table'): # 显示所有用户定义的表名
    print(row.table_name)
# 表的列名用cur.columns('tb'); row.column_name，本次选取了的列名用cur.description; row[0]
```

## PySnooper

* 每一行代码的执行
* 变量的声明和赋值
* 返回值
* 持续时间

```py
@pysnooper.snoop(normalize=True)
def f(): ...
# 在函数中的某一部分上使用：with pysnooper.snoop():

# 参数：
snoop('/my/log/file.log', prefix='ZZZ ')
watch=('foo.bar') # 查看非局部变量的值；watch_explode展开字典的内容
depth=2 # 调用其它函数的跟踪深度，默认为1
```

## Cython

* 纯Python模式
  * 第一种是在对应名字的pxd中写cpdef但不实现，类似于pyi，完全不影响本来的py
  * 也支持直接写type hint，但int要写cython.int否则仍视为object不会有任何提升，且与其它使用typing的库有冲突
  * 还可以用装饰器声明locals(a=xxx), returns, exceptval(-1, check=True), cfunc(等价于cdef), inline, ccall(等价于cpdef)
* Jupyter：%load_ext Cython之后在需要的块中%%cython [-a]，可直接用于非函数定义块；--compile=-O3 --link-args=xxx
* 其它库
  * mypyc：基本类型有运行时类型检查，多继承必须用trait特性，对dataclass优化，尽量隐式用slots
  * 与C++交互：pybind11或同作者后出的nanobind，后者必须修改C++部分的代码以适应库，前者相反
  * 把简单的py编译到可读性强的c：pyccel，win下使用非常麻烦
* 详细教程：https://www.cnblogs.com/traditional/tag/Cython/

### 构建

* pyximport.install()后能不编译就import pyx模块，但只能用于开发因为需要环境里有Cython和编译器，依赖额外的.h时也很麻烦。构建结果在~/.pyxblx
* 在CI中为多个系统编译：cibuildwheel库

```py
[build-system] requires = ["setuptools", "cython"]

# setup.py
from Cython.Build import cythonize
setup(ext_modules=cythonize('demo.pyx' 或下一行))
setuptools.Extension("*", sources=["**/*.pyx"], libraries=["m"], library_dirs=numpy.get_include(), include_dirs, define_macros=[(k,v)], extra_compile_args)

from mypyc.build import mypycify
setup(ext_modules=mypycify(['xxx.py'])) 或 list(set(glob('*.py'))-{'setup.py','main.py'})，不支持自动递归包含

# CLI
python setup.py build_ext --inplace # 生成so/pyd且与pyx位于同一位置，能直接import；已有时不会重建，此时可用-f，-j多线程；不支持-a
cython xxx.pyx -a # 生成c，产生与Py交互的分析。小心文件名写错或忘加后缀时无任何提示
cythonize -i xxx.pyx # 生成so/pyd，也会产生对应的.c .o .def临时文件。之后就只能import使用，无法从命令行调用了
mypyc xxx.py --ignore-missing-imports # 很干净，只有so/pyd。默认会递归处理导入了的，如果那条忽略还不够就再加--follow-imports=skip

# 手动编译
$pybase = $(python -c "print(__import__('sys').base_prefix+'/')");
gcc -shared -pthread -fPIC -fwrapv -fno-strict-aliasing -O3 -I ($pybase+"include") -L $pybase -lpython312 src.c
生成可执行文件，仍依赖整个Py环境：先用cython --embed，再用gcc -municode且不能有-shared，好像可以不用-D_UNICODE和UNICODE
```

### 语法

* nogil时不能使用任何Py对象。Py侧函数中可以用with nogil: 调用nogil的函数，否则也能调用但不会释放gil
* 整除默认用的Py的语义，用`# cython: cdivision=True`或`with cython.cdivision(True)`改成C的语义
* TODO: https://cython.readthedocs.io/en/latest/src/tutorial/strings.html 做字符串拼接时要声明中间变量 、Fused Types（类似模板/泛型）

```py
cimport cython # 导入pyx
from libc.stdlib cimport malloc, free # 自带C标准库和一些posix库。查看：源码的Includes

def primes(int nb_primes): ... # def的函数只能在Py侧调用，但里面可以调用cdef的。cdef的只能在pyx中用，cpdef就都能用
cdef inline int add(int a, int b) nogil: return a+b # 返回值若省略则默认为object。不支持static
预处理指令：DEF、IF、ELIF、ELSE

cdef: # 一次性声明多个变量
    int n = 3 # 不会自动初始化
    int arr[100] # 不支持VLA
    int* arr2 = <int*>malloc(100*cython.sizeof(int)) # 要free，一般用finally。类型转换：尖括号；TODO:<T?>好像能进行检查是否能强转，否则强转失败时还是原值
    char* s = 'abc' # 对应bytes
    bint b # 对应Py的bool
    object o # Py_Object
    指针：声明和&n与C一样，但不能用*p解引用，要用p[0]。访问结构体指针变量的成员用点，不是->。支持assert p is not NULL
cdef struct S: int n # 创建实例：S(123)或cdef S s={'n':123}。还有cdef packed struct、cdef enum
cdef class: # 能在Py侧使用

@cython.boundscheck/wraparound/cdivision/initializedcheck(False) # 关闭下标越界/负索引/除零/内存视图初始化检查；也可注释在开头#cython: xxx=False 用于整个文件，也可用在with中
cython.address()等于&，cython.operator.dereference()等于*

# 数组和内存视图
from cpython cimport array
import array # 教程如此，实际测试不导入这个也行，也许是用于Py侧的传进来
cdef array.array a = array.array('i', lst) # 复制一份，仍视为object，能用一些CAPI如resize_smart、extend、zero
cdef int[:] ca = a # 这种类型更适合作为Cython函数的参数，还可加const；[:,::1]表示二维数组且最后一维连续储存；能用with nogil, parallel()；ca[:]=0能把数组全部赋0。实际是memoryview，主要用于访问缓冲区而非创建，不能直接传list
a.data.as_ints # 变为int*，用于调用C API；用as_voidptr变为void*
cdef int value; for value in values[:count]: ... # 使用for遍历int*；数组转list用.tolist()，视图或int*用列表推导式
另外其实也支持直接写cdef list l，还支持dict set bytes
```

### 使用库

```py
# 最简单的使用C函数的方式
int fun(int a) { return a; } # test.c
# testmod.pyx；必须不能是test.pyx，因为它俩在同一目录，而.pyx会编译成.c，就会冲突
cdef extern from "test.c":
    cpdef int fun(int a) # 可以不写参数名称，但就无法用命名参数了

# C库封装示例。在cqueue.pxd中，对应C语言的头文件，把原内容重写一遍，这样不容易导致命名冲突；不能有def函数；宏定义普通地声明变量：
cdef extern from "queue.h":
    ctypedef struct Queue: pass # 对应typedef struct _Queue Queue;
    Queue* queue_new() # 隐式cdef extern
    void queue_free(Queue* queue)
# 在queue.pyx中写包装类，目的是把C风格的函数变成Py风格的类；基本名必须不同于那个.pxd
# distutils: sources = lib/queue.c, another.c # 静态链接时必须指定
# distutils: include_dirs = lib # 头文件所在文件夹，如果不在同一目录就也是必须的
# distutils: define_macros=A=1 B # 定义宏
cimport cqueue # 导入pxd
cdef class Queue:
    cdef cqueue.Queue* _c_queue
    def __cinit__(self): # 这里面只能给cdef的成员赋值，与c相关的内存分配如malloc()放这里
        self._c_queue = cqueue.queue_new()
        if self._c_queue is NULL: raise MemoryError()
    def __dealloc__(self):
        if self._c_queue is not NULL:
            cqueue.queue_free(self._c_queue)

    cdef extend_ints(self, int* values, size_t count): ... # Py不支持int*，显然不能用cpdef
    cdef int peek(self) except? -1: ... # 当函数体会主动抛异常时必须这样声明，否则会打印异常并忽略。此语法表示返回值是-1时会自动检查是不是出现了异常，应选一个小概率出现的值作为异常值
    # 支持Callbacks传递函数，但太复杂略。如果可能发生异常要加except*
```

## cffi

```py
from cffi import FFI
ffibuilder = FFI() # echo_build.py
ffibuilder.cdef('int echo(int a);') # 要在py中用的函数，支持某些地方使用三个点生成特殊代码

# API编译模式，会生成_echo_cffi.c再编译成py模块，默认使用Release文件夹存放.o，但用了setup.py就也用build文件夹了
ffibuilder.set_source('模块名如_echo_cffi', '#include "echo.h"', libraries=['dll/so无后缀'], library_dirs=['.'], include_dirs=['xxx']) # 第二个参数是C意义上的使用库时的实现；libraries是要链接的库，如Linux下一般要加数学库'm'，位数要和Py的一样；如果库都还没编译，也可以指定sources=['echo.c']
if __name__ == '__main__': ffibuilder.compile(verbose=True)
setup_requires=['cffi']; cffi_modules=['echo_build:ffibuilder']; install_requires=['cffi'] # 必须有运行时，否则报No module named '_cffi_backend'
from _echo_cffi import ffi, lib; lib.echo(...) # 这俩对象是固定的

# ABI模式动态加载，无需编译器，仍需cdef
os.add_dll_directory(os.path.abspath('.')) # 不加则只会搜索系统库位置；要不就下一条写绝对路径，不清楚./有没有效
lib = ffi.dlopen(None或xxx.so/dll) # None为加载标准库但Win不支持，具体库必须加后缀，要不就用ctypes.util.find_library()
# ffi.verify() 废弃了

# 运行时创建C类型
a = ffi.new('int[10]') # 相当于int a[10]
s = ffi.new('char[]', b'hello') # 相当于char s[] = "hello"，可配合二进制模式下的f.readinto
st = ffi.new('St*', {'p': 1}) # 定义结构体，如果内部又有指针，要把子对象加到全局的weakref.WeakKeyDictionary()中
ffi.cast("int", 2)
```

## numba

* 依赖numpy，适合大量循环数学计算，需要一个较大的运行时依赖
* @njit强制nopython模式
* 自动处理模块中的所有函数：jit_module(nopython=True)，只会处理调用前声明的
* parallel=True多线程执行，fastmath=True启用精度较低但更快的浮点运算，cache=True缓存到pycache里，error_model='numpy'除以零时不抛异常而是变为inf或nan
* 可以加速pandas，pd的某些函数用参数engine='numba'
* AOT：运行时不再需要numba，但仍需要numpy
* 函数中用了isinstance()“重载”：@generated_jit
* 特化对数组类似于卷积运算的情形：@stencil

## 定时任务和任务队列

* threading.Timer(秒数, fn, args=None).start()：自带，非阻塞，只执行一次，不易管理
* sched：自带，能管理多个注册的回调。s=sched.scheduler(); s.enter(5秒延迟, 0优先级一般没用, fn, argument=(123,)); s.run(blocking=默认True表示等所有事件执行完后再继续)
* dbader/schedule：every(10).minutes/every().hour.do(fun) 轻量无额外依赖，用法相对简单，有装饰器用法。支持秒级任务，阻塞，有一定管理作业的功能，有日志记录。无自动异常处理，会直接抛出，导致后续所有的作业都中断执行
* celery：分布式任务队列，功能强大 https://zhuanlan.zhihu.com/p/22304455 。kombu：也是celery出的，messaging
* rq：使用redis的任务队列，比celery简单
* huey：peewee作者出的，支持redis,sqlite,in-memory的任务队列
* dramatiq：需用redis或rabbitmq
* APScheduler：支持定时任务，可用redis。感觉设计比较复杂
* https://github.com/Miksus/rocketry 定时任务，比较新

## PDF

* pdfminer.six：纯Py，主要用于提取内容
* pdfplumber：基于pdfminer.six
* reportlab：用于创建，商业版的开源版本
* pikepdf：偏底层
* PyMuPDF：功能全面但是GPL
* pdf2docx：基于PyMuPDF，生成的docx能在一定程度上保留格式
* PyPDF2：曾经不维护了，现在复活了
* 不维护的：pdfrw pdfminer

## 杂项

* colorama：控制台的前、背景色；rich：自动染色和格式化
* icecream：代替print输出调试信息，无参调用会显示被调用时所在文件和行号，有参调用会显示参数内容和返回值，并再返回那个返回值；可一件关闭所有输出；未安装可透明回落不存在
* beeprint：格式化打印dict
* python-dotenv：从`.env`中读取并设置环境变量
* PyYAML：一定要用safe_load；或者用strictyaml
* lazy_import：np = lazy_import.lazy_module("xxx")，且也会将lazy化的模块放到sys.modules里，之后其它模块用的xxx也是lazy的
* watchdog：用于监测文件变化
* attrs：dataclasses的增强版；pydantic也类似，主要支持数据验证
* r1chardj0n3s/parse：f-string的反向，可以捕获到命名字典里，parse完整匹配，search只要求p是str的一部分且是非贪婪的但有BUG(#41)，findall直接返回列表结果也是非贪婪的
* lexer/parser
  * https://github.com/lark-parser/lark 扩展的EBNF，功能最多性能好
  * https://github.com/pyparsing/pyparsing 纯Py语句，自底向上
  * https://github.com/textX/textX 用于构建DSL
  * https://github.com/erikrose/parsimonious  简化了的EBNF，性能好。有几年没维护，2022年又有人接手了，提交数不多
  * https://github.com/neogeny/TatSu EBNF，3.8，star很少
  * https://github.com/pytransitions/transitions FSM
  * https://github.com/dabeaz/sly 源于lex和yacc，作者不维护了
* pretty_errors：精简stacktrace，可全局安装
* amazing-qr：虽然star数很多，但依赖太多，要numpy和Pillow。segno：作者好像水平很高
* 计算圈复杂度：https://github.com/terryyin/lizard 支持多种语言无需编译

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
* https://github.com/HelloGitHub-Team/Article/blob/master/contents/Python/cmdline

## TODO

* PyTest https://realpython.com/learning-paths/test-your-python-apps/ https://zhuanlan.zhihu.com/p/385619511
* PyNaCl https://github.com/pyca/cryptography pyOpenSSL pycryptodome
* 数据可视化：Seaborn(基于matplotlib) bokeh plotly.py plotly/dash(基于plotly.js，用于构建网页) matplotlib altair
* poetry，替代pip+venv：https://zhuanlan.zhihu.com/p/81025311 https://python-poetry.org/
* https://github.com/gevent/gevent https://www.gevent.org/
* 自动化任务工具invoke：https://zhuanlan.zhihu.com/p/105263640；Fabric https://zhuanlan.zhihu.com/p/107633056
* https://github.com/serge-sans-paille/pythran AOT，不支持类
* https://github.com/mahmoud/boltons 纯Py utils大集合，不过社区贡献并不太多
* https://github.com/rthalley/dnspython
* https://github.com/scrapinghub/splash 具有HTTP API的轻型浏览器js渲染引擎
* https://github.com/zopefoundation/ZODB 虽然star数少，但提交数很多，支持事务
* 函数式编程：https://github.com/Suor/funcy https://github.com/JulienPalard/Pipe
* 与外部程序交互：https://github.com/pexpect/pexpect https://github.com/amoffat/sh https://sarge.readthedocs.io/en/latest/
* 解析url：https://github.com/gruns/furl
* https://gitlab.com/mike01/pypacker socket库
* https://github.com/prkumar/uplink 把REST API变成class
* 自动重试：https://github.com/jd/tenacity
* https://github.com/hugapi/hug 基于 falconry/falcon 的WebAPI框架，但hug有一段时间没提交了，falcon比较活跃但更底层，考虑学falcon，还支持ASGI；其他人练手的ASGI框架：abersheeran/index.py almarklein/asgineer
* profiler：https://github.com/benfred/py-spy https://github.com/emeryberger/scalene
* PySimpleGUI，基于tk的。其余的功能强大的GUI库统统不学，毕竟有qt。
* https://github.com/jek/blinker 功能简单的非分布式信号（事件）库
* Pyarmor：混淆源代码，但有运行时依赖
* fuzzywuzzy：字符串模糊匹配
* https://github.com/jpadilla/pyjwt
* memcached：pymemcache pylibmc
* 缓存：python-diskcache cacheout rafalp/async-caches cachetools（提供支持TTL的类dict）
* mkdocs mkdocs-material
* ansible
* Brython 在浏览器中运行的Py，支持一部分标准库；~~Transcrypt Py2JS编译器~~；pyodide 编译到WA，支持第三方纯Py库；PyScript 基于Pyodide；PythonMonkey：互调py和js的库
* decorator：更方便地创建装饰器
* 操控浏览器：playwright-python Splinter pyppeteer selenium crawlee
* pyinstrument：使用简单的profile工具
* multibuild：用于产生多平台的wheel
* birdseye：调试工具，与pysnooper是同类的
* https://github.com/JaidedAI/EasyOCR
* wrapt：方便写装饰器，自动处理方法
* https://github.com/Z4nzu/hackingtool
* joblib：有三个功能，一是透明硬盘缓存，二是并行计算，三是快速二进制序列化
* pyupgrade：自动把老版本语法更新到新版本，但条目不多
* https://hatch.pypa.io/latest/ 包管理
* https://tox.wiki/en/latest/
* https://github.com/charliermarsh/ruff An extremely fast Python linter, written in Rust. 代替Flake8 Black isort
* https://github.com/lcompilers/lpython 编译器
