---
title: 第三方 Python 库
---

## 环境和打包

预编译的Win下的包：https://www.lfd.uci.edu/~gohlke/pythonlibs/

### requirements.txt

* pipreqs能从import产生本文件，替代freeze；pip-tools能从setup或一个.in产生本文件并能同步版本更新；pigar支持notebook；基本上只有需要锁定依赖时才用

```bash
pip freeze > requirements.txt # 需在venv中运行否则会把全局的写进去；类似于pip list --format=freeze，只是verb不同，freeze参数少
pip install -r requirements.txt

--index https://xxx
SomeProject==1.4
SomeProject>=1,<2 # 逗号为且；在CLI中运行需加引号否则大于号会被认为是重定向
SomeProject~=1.4.2 # install any version “==1.4.*” that’s also “>=1.4.2”
-e . # 相当于pip install -e .
```

### venv

* pip install 不需要--user
* 不能脱离本机环境，会在`venv/pyvenv.cfg`中硬编码Python的版本和home位置；如果Python是in-place升级了版本，可用venv --upgrade .venv，之后记得更新venv里的包；但如果Python自己的路径变化了，就只能手动改了；手动改之前shell就不要进venv了，否则报Permission denied
* `--system-site-packages`使得虚拟环境可访问系统的包，install仍不影响系统，freeze的时候就要加--local

```bash
python3 -m venv .venv

alias activate=". .venv/bin/activate"

if not exist .venv python -m venv .venv --upgrade-deps
.venv\Scripts\activate.bat

if(!(Test-Path .venv)) {python -m venv .venv --upgrade-deps}
& .venv\Scripts\activate.ps1
```

### 包和模块

* 一个.py文件就是一个模块，模块名`__name__`按目录组织，用点分隔，import时无需也不能加.py后缀
* 一个含有`__init__.py`的目录就是包，import该目录时相当于导入该目录下的`__init__.py`，它的`__name__`等于目录路径对应的模块名
* 对于`x/y/z.py`，`import x.y.z`会依次运行`x/__init__.py`和`x/y/__init__.py`，再运行`c.py`，且只能通过`x.y.z.xx`访问z中的东西（包括z里import的），x的init中声明的变量和import的东西都被限制在x的空间中无法访问（除非z里import了x的）；`import x`无法用`x.y`和`x.y.z`访问y和z，除非x的init里import了
* 模块只初始化一次，所有变量归属于某个模块，import机制是线程安全的，所以模块本身是天然的单例实现。一个函数如果绑定了对应模块内的全局变量，当在别的地方`import *`后修改那个全局变量，函数仍然使用的是原来的变量，与class类似
* python命令行也可运行目录，目标为那**一个**`__main__.py`；运行目标时会把`__name__`变量设为`'__main__'`
* 不用-m会把目标所在的文件夹加到sys.path中，然后按路径直接执行目标，目标就是顶级模块；用-m会把cwd加到sys.path中，按模块名先一层层执行`__init__.py`再执行目标，会先编译成.pyc，会把`__package__`设为模块名的前一部分，cwd是顶级模块；该sys.path与环境变量的path无关，对于环境变量修改PYTHONPATH可更改搜索地点
* 不要自己创建名为`runpy.py`的文件，因为系统存在runpy这个包
* VSC的lint默认是从工作区开始的，在子文件夹中运行存在绝对导入的py时能正常运行，但lint却会报错
* 还存在命名空间包的概念，把多个位置不想关的包算进一个命名空间方便使用
* `runpy.run_module('xxx', run_name='__main__', alter_sys=True)`相当于命令行中-m xxx；不加后两个参数就是在不import那个模块的时候使用它

```python
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

### setuptools

* 一般结构为：仓库根目录下`setup.py`, `setup.cfg`, `readme`, `tests`, `mypkg/__init__.py`, `mypkg/data/xxx.json`, `mypkg/xx.py`。装好后就能`import mypkg`和`import mypkg.xx`了。如果有子目录却没有init文件，在作为系统包时无法import那里面的内容
* pip的包名与模块无关
* python3 setup.py --help-commands；生成whl用bdist_wheel，需先装好wheel，生成过程在build文件夹里，生成的东西在dist文件夹里；install自动生成egg并安装模块，也会自动安装依赖但不会走pip自定义的源，实际上用的是easy_install；build_ext --inplace能生成c扩展，可用--compiler指定编译器；bdist --help-formats
* twine upload [--repository testpypi] dist/*；pypa/gh-action-pypi-publish
* pip install . 可以识别setup.py和那个toml，无需-f就能覆盖；加-e可以在编辑源文件后无需install即时生效，仅用于开发，原理是软连接，当然setup自己除外；setup develop [--uninstall]效果类似一样但后者不会删入口点exe
* pip wheel . 可以在pwd生成wheel，还是需要setup.py；注意不是python -m wheel
* pip download --only-binary :all: --dest . xxx 下载whl
* 还有一个pbr模块可用在setup_requires，好像能从requirements.txt自动生成依赖
* 检查wheel存在的问题的项目：https://github.com/jwodder/check-wheel-contents
* MANIFEST.in额外控制sdist的内容，默认包含和不包含：https://packaging.python.org/guides/using-manifest-in/#how-files-are-included-in-an-sdist
* 使用包内数据：importlib.resources.files("mypkg")/"data/data.csv" https://importlib-resources.readthedocs.io/en/latest/using.html；单个py_modules无法使用

```python
# __init__.py；必须有此文件才能自动发现
from impl import fun # 从实现中公开函数
__version__ = '0.0.1' # 默认0.0.0
__all__ = ('fun',) # 在被import *时如果存在此字段，只会导入它指定的，help也只能看到这些

# __main__.py
from . import xxx
def _main(): # 即使不存在__all__也不会被import *
    xxx()
if __name__ == '__main__': # 理论上本文件就是设计成直接运行的，但保不齐被别人import
    _main()

# setup.py：https://packaging.python.org/guides/distributing-packages-using-setuptools/
import setuptools
setuptools.setup( # 也可无参调用，参数能覆盖cfg，写错没有警告
    name = 'xxx',
    packages=['mypkg'], # 也可用find_packages()自动搜索存在__init__.py的文件夹
    package_dir={'mypkg': 'src/mypkg'}, # 如果位置不对可以手动映射，key也可以是''则用最后一部分作为包名
    package_data={'mypkg': ['data/*.dat']}, # 即src/mypkg/data/*.dat，key必须是包名或''即任意包，*.dat只会包含mypkg同级目录下的，或者用*/*.dat
    py_modules=['test'], # 对应与setup.py同级的module.py，对于sub.test也可用且不会flat；或用[x.stem for x in Path().glob('*.py')]动态获取
    entry_points={"console_scripts": ["foo = foo.__main__:_main"],},
)

# setup.cfg：https://setuptools.readthedocs.io/en/latest/userguide/declarative_config.html
[metadata]
name = xxx
version = attr: mypkg.__version__
author = xxx
author_email = xxx
description = xxx
long_description = file: README # long_description_content_type = text/markdown
keywords = one, two
license = MIT # license_file = LICENSE 3rdparty/*.txt （多个需换行）
url = xxx
platform = any
classifiers = # https://pypi.org/pypi?%3Aaction=list_classifiers
    Development Status :: 5 - Production/Stable # 3 - Alpha
    Intended Audience :: Developers
    License :: OSI Approved :: MIT License
    Operating System :: OS Independent
    Programming Language :: Python :: 3
    Topic :: Internet :: WWW/HTTP
project_urls =
    Bug Tracker = https://github.com/user/repo/issues
    Changelog = https://github.com/user/repo/blob/master/CHANGELOG.md

[options]
packages = find: # 还有一种find_namespace:
install_requires =
    requests;python_version<'3.4' # https://www.python.org/dev/peps/pep-0508/
    pywin32 >= 1.0;platform_system=='Windows'
python_requires = >=2.7, !=3.0.*
include_package_data = True # 将MANIFEST.in的内容打包进bdist，还可指定exclude_package_data优先去除bdist的内容
scripts =
    bin/script
    scripts/script
zip_safe = False # 不启用时作为源码安装，方便调试，兼容性好；启用时作为.egg压缩包安装，性能好；对wheel无任何影响
# setup_requires可以加一个wheel；test_suite = tests；tests_require废弃了

[options.entry_points]
console_scripts = # 还支持gui_scripts，关闭父console还能运行；如果某一项需要额外的依赖，用方括号声明名字并在extras里写内容
    myexe = proj.__main__:_main # 若用proj:_main，得到的是init中的对象，而不是__main__.py的

[options.extras_require] # pip安装时或entry_points用中括号才会装上
tests = tox; pytest # 不明白为什么列表变成分号了但是就是这样，也可分行写

[options.packages.find]
where = src
include = pkg*
exclude = tests

[options.package_data] # 不要与include_package_data共用；还有个data_files能把文件安装到指定位置，但should be相对路径，相对于sys.prefix
* = *.txt, *.rst
hello = *.msg

[bdist_wheel] # 对应verb的开关
universal = True # 能在py2和3上不做任何改变就运行且无C扩展才能开，命名上是py2.py3-none-any

# pyproject.toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"
https://www.python.org/dev/peps/pep-0518/
https://github.com/takluyver/flit
https://www.python.org/dev/peps/pep-0621/
https://pypa-build.readthedocs.io/en/latest/
版本支持upper bound限定，不会修改最左边的非0数字：^0.1.11能更新到0.1.19但不会到0.2.0

# ~/.pypirc；chmod 600
[distutils]
index-servers =
    pypi
    pypitest

[pypi]
username:
password:

[pypitest]
repository: https://test.pypi.org/legacy/
username:
password:
```

## Scrapy

TO READ：我的一个gist

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

* 在cssselect和lxml上构建的库；pyquery也是这两这上构建的。但是cssselect和Parsel都是scrapy开发的
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

### 其它库

* urllib.request.urlopen不提供复用，http.client更加底层，两者都无法实际使用
* urllib3是第三方库，线程安全；urllib3.PoolManager().request('GET',url)，只能下到bytes，要自己手动解码`r.data.decode('u8')`，不默认发gzip但能自动解码
* httpx的api差不多，且支持异步、h2、brotli。长连接用httpx.Client()；底层用的是同作者的httpcore
* requests-html基于bs、pyquery、pyppeteer等构建，超级重，支持asyncio，.render()自动用chrome请求ajax，第一次用会下载
* httplib2：和urllib3差不多级别的API，活跃度不高，可用于Py2
* faster-than-requests：新，无依赖，速度快，非纯Py，贡献者极少
* h11：底层库，和http.client同级别
* python-trio/hip：urllib3的fork，添加了异步，开发很早期，没有使用的价值

### Session

* 能连接复用以及保留cookie
* 即使使用了会话，方法级别的参数也不会保留
* 非线程安全的，toolbelt提供了简单的多线程

```python
s = requests.session() # 其实最好用with，这样发生了异常也能关闭
s.request = functools.partial(s.request, timeout=3) # 连接超时时间，可为小数，默认无穷大，不加会一直等；直接赋值只影响connect超时时间，可传递元组，第二个参数控制下载超时；Session级别的只能这样设置，是故意的
allow_redirects=True; max_redirects=30 # 前者默认为True（head除外），会自动跟踪3xx因此结果直接是200；后者是默认值
verify=True #【默】，也可设为CA文件的路径，默认用Mozilla的；cert参数是客户端验证的证书
proxies={"http": "http://10.10.1.10:1080", "https": "http://10.10.1.10:1080"} # 默认会检测环境变量HTTP_PROXY，不直接支持socks
headers={'User-Agent':'python-requests/2.23.0','Accept-Encoding':'gzip','Connection':'keep-alive'}.update({'Referer': referer}) #【基本默】普通dict，大小写不敏感
auth=('user', 'pass') # Authorization头，如果不放到session里，重定向时会自动去掉
cookies.set(k,v,domain,path) # 类型是RequestsCookieJar，但也可以传dict。另有requests.utils.add_dict_to_cookiejar(cj, cookie_dict)、cookiejar_from_dict、dict_from_cookiejar几个函数；有可能第一次能用dict，之后就要用它们了，不能直接update
```

### 请求和响应

* url必须要有scheme；必须每次写完整url，要不就用requests_toolbelt提供的BaseUrlSession
* get的params会自动变成查询参数，且值为None的不会附加上去，值为list的会自动与k展开
* post的data和json传dict（json还可以是list）会自动编码并设置Content-Type，也因后者故最好不要传字符串形式的json给data，可以先loads一下
* 传字符串给data不会有额外变化，就是设置body；传字符串给json无意义；data还支持file-like-objects且支持流式处理，文件记得以rb打开；data还支持生成器，则会传输分块编码
* RFC 2616规定如果Content-Type没指定编码且类型是text/*，那就用ISO-8859-1；又不过RFC 7231去掉了这个限制

```python
r: Response = s.get(url,params={k:v})、post(url,data/json = {k:v}/str)、put/delete/head/options
r.raise_for_status(), r.status_code # 200，== requests.codes.ok
r.json() # 即使解码成功也不一定意味着请求成功，因为有时服务器会在失败时也返回json
r.text # 根据encoding解码的HTTP内容字符串
r.encoding # 可赋值，一般在它等于'ISO-8859-1'时赋r.apparent_encoding
r.content # HTTP内容二进制，但会自动解码gzip，适用于图片等
r.url, r.history # 后者为重定向响应列表
r.headers # 字典，此时为响应头部，仍可用r.request.headers访问请求头部

# 保存二进制文件的推荐方式，会自动gzip解码
with open(filename, 'wb') as fd:
    for chunk in r.iter_content(chunk_size):
        fd.write(chunk)

# 缓存，默认是保存在内存中的dict，还支持redis；另有requests-cache库星数更高但不是requests官方推荐的
from cachecontrol import CacheControl
cached_se = CacheControl(requests.session()) # 指定文件缓存：cache=cachecontrol.caches.FileCache('.webcache')
# 一些建议：params最好sorted一下、缓存的响应永远不要流式处理

# stream=True，则get返回时只有Header被下载下来了，可以进行一些处理，直到访问content才会下载响应体；还可以使用raw属性，是urllib3里的原始响应，未gzip解码？
with requests.get(url, stream=True) as r:
    if r.encoding is None: r.encoding = 'utf-8'
    for line in r.iter_lines(decode_unicode=True): # 迭代流式API
        if line: # filter out keep-alive new lines
            decoded_line = line.decode('utf-8')
            print(json.loads(decoded_line))

# post的files={'filename': file-like-objects}用于multipart/form-data类型的请求，但默认不支持流式处理，要用requests_toolbelt才行
```

## Beautiful Soup

* 支持不同的HTML Parser，其中html5lib最接近真实网页，是纯Python，相对慢；lxml（其实是lxml.html）容错性性中游，速度最快；自带的html.parser容错性差；另外还有一个html5-parser，基于c更快但star只有11，不考虑
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

## [PyInstaller](https://pyinstaller.readthedocs.io/en/stable/usage.html)

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
* TODO: https://zhuanlan.zhihu.com/p/40716095 https://mp.weixin.qq.com/s/rL84_hBqH4CX-SmUXnjKAQ https://www.zhihu.com/question/281858271
* 好像有个PyOxidizer是相同功能，但是需要装Rust且开发非常早期

## python-fire

* 功能简单，几乎就是链式调用，类似于把空格变成点。能把函数、类中的函数、模块中的函数（实际上都是对象）变成cli工具
* 选项参数名字中的横杠和下划线等价
* 支持`*args`或`**kwargs`
* 分隔符`-`显式告知之后的为子命令而非参数，例如`- upper`可以把结果变为大写
* `python -m fire xxx`可以对目标模块做任何改变而使用
* BUG：对于对象，--help/-h无法显示verb，必须要用`- -h`才行

```python
import fire
def hi(name='world'): return 'hello'+name
fire.Fire(hello) # python cli.py world或--name=world

class Calculator: # 支持继承
    def __init__(self,offset=1): self.o=offset # 构造函数中的变为Flag；可访问属性o
    def add(self,a,b): return a+b
    def multi(self,a,b): return a*b
fire.Fire(Calculator) # python cli.py add 1 2；python cli.py o --offset=1
```

## ipython

* `In[]`和`Out[]`记录了每一次的输入和输出，关键是可以用中括号获取它们；最后三次输出保存在`_`,`__`,`___`中，输入保存在`_i`中
* 退出用的是Ctrl+D，python在Win下用的是Ctrl+Z；支持Ctrl+r的与bash类似的历史搜索
* 传递参数给文件或模块要用`--`，否则会被认为是传给ipython自己；而python不对`--`特别对待
* 用pipx装的时候记得加`--system-site-packages`，且不要在里面的pip更新系统包
* 交互式输出对象默认使用pprint，可以用%pprint关闭；在语句后加上分号会不显示交互式输出结果（不显示Out）
* 普通代码中用`IPython.embed()`进入IPython环境，但只能手动交互，并不是之后的代码就由IPython自动执行了，也不会读取设置，好处是修改了全局变量等退出到原有Python环境中时能保留；start_ipython()是普通的启动IPython的方法，会读取设置
* `from IPython.display import display`能输出html、jpeg、js、md、json
* 自动括号和引号：`/fun 1 -> fun(1)`，`/fun 1,2 -> fun(1,2)`；`,fun a b` -> `fun('a','b')`；`;fun a b` -> `fun('a b')`
* 默认输入时自动忽略`>>>`和`...`，用于方便输入含有交互式提示符号的语句；doctest_mode可以改变这一行为但不懂怎么用
* exit()的行为和python不一致
* `get_ipython()`不为None就是在ipython中
* ctrl+space不是自动完成，而是进入选择模式，受emacs的影响
* TODO: https://github.com/ipython/ipython-in-depth

### [魔法命令](https://ipython.readthedocs.io/en/latest/interactive/magics.html)

* 单个%是行魔法，回车就执行，默认开启了%automagic，无歧义时不加%也行；两个%%是cell魔法，会提示`...`允许多行输入，无法省略百分号
* 命令的结果可以赋值给Python变量，此时无法省略%；命令的参数支持用`$`或者大括号嵌入Python变量，用`$$`转义一个`$`
* quickref：显示所有魔法命令的简要参考；lsmagic：显示所有支持的魔法命令；?加魔法命令：显示指定命令的参考
* run test.py：运行脚本，-i可以继承当前会话的变量，-d启动调试，-t计时，-p或%prun启动Profiler，-m调用模块（参数仍要--）
* debug：在刚刚出现过异常后使用，能进入ipdb检视刚才的异常栈；当然也可以一开始就用，设置断点；支持%%，接下来输入代码就可以debug
* edit：启动编辑器来输入交互式代码，关闭后会传到cli里；VSC会自动加一个\n，没啥好办法
* store：持久化储存变量，store -r恢复
* hist：查看历史命令，-n加上序号，-g pattern用grep搜索
* timeit：统计语句运行的时间，会多次运行取平均值或手动指定-n；有%%；单次运行是time
* xdel：删除变量并试图清除在其对象上的一切引用
* who：显示当前所有变量，可指定要显示的类型；whos信息更丰富
* pip：可以直接运行pip
* macro name n1-n2 n3：把n1到n2及n3这几行代码命名为name的宏，之后输name即可，但那时就不显示具体输入内容了，可用store储存，用edit编辑
* pastebin：自动把东西上传到GitHub的gist里并返回链接
* alias：显示预定义的shell命令，也可设置自己的，不过直接设置不能持久化；rehashx：把path里的可执行文件都导入alias中，使用时就不用加叹号了
* pycat：语法高亮地显示文件
* bookmark、pwd、pushd、popd、dhist：与路径有关的一些操作
* env：显示环境变量
* !ls执行shell命令，但Win下默认是CMD，且编码为gbk；!!：没看懂和一个的区别
* ?加命令：内省，会显示docstring但与help()的格式不同，且不会显示函数文档，只显示函数名；??两个问号：还会显示源代码
* ?加带有`*`的对象：显示匹配到的对象名；其实是psearch命令
* save：把指定的行保存到文件中、load把目标文件的内容输进终端且不自动执行、recall把上一次的输出放进输入中且不执行、reset -f清除所有定义了的变量、%%writefile将本单元格保存到文件中、paste粘贴并执行、rerun：重运行指定指定行的代码
* %%HTML、%%js、%%latex、%%markdown：将cell渲染成HTML输出；%%js运行JS
* autoreload 2：需要load_ext加载。import模块后修改源文件能自动变化
* matplotlib inline

### 配置

* ipython profile create [profilename]：创建`~/.ipython/profile_default/ipython_config.py`
* 在`profile_default/startup/`中的.py或.ipy会自动执行，命名可以`10-xxx.py`这样含有优先级
* config加不含c.的设置项可以动态读取和设置值
* 使用`import os; os.environ['comspec']='powershell.exe'`更改`!`的shell；或者创建自己的魔法命令，从`-c -`读取

```python
c.TerminalInteractiveShell.confirm_exit = False
c.TerminalInteractiveShell.editor = 'code -w'

# 未更改的设置
c.InteractiveShell.autocall = 0 # 启动自动括号，设为1时是智能模式，2是完全模式
c.InteractiveShell.logstart = False # 启用后会保存会话，下次就会恢复；但默认是overwrite模式？
c.InteractiveShell.pdb = False # 控制是否出现异常时自动进入ipdb，可用%pdb开关
c.InteractiveShellApp.exec_files/.exec_lines/.extensions = [] # IPython启动时要执行的文件/代码/IPython扩展（load_ext）
c.StoreMagics.autorestore = False # 开启后store能自动持久化
```

## pdb

* VSC调试用的并不是pdb，故先不学了
* python -m ipdb
* 输入单个问号能显示功能命令
* Debugging a broken unit test: pytest ... --pdbcls=IPython.terminal.debugger:TerminalPdb --pdb
* breakpoint()：进入pdb，是原版python就有的功能，相当于`pdb.set_trace()`？
* 命令教程：https://zhuanlan.zhihu.com/p/37218789 https://zhuanlan.zhihu.com/p/43846098
* 还有个pdb++(pdbpp)项目

## jupyter

* pip install jupyter; jupyter notebook --no-browser --allow-root
* 在反代之后需要配置`NotebookApp.allow_remote_access`或`c.NotebookApp.allow_origin`，否则会报`Blocking Cross Origin API request`或`Blocking request with non-local 'Host'`；`/api/kernels/`和`/terminals/`需要配websocket，各种配置中都设置了`Host`
* 会往`%AppData%\jupyter`里写东西，但在商店的Python里会装到沙盘里
* https://www.zhihu.com/search?type=content&q=jupyter https://www.zhihu.com/question/59392251
* Docker映像文档：https://jupyter-docker-stacks.readthedocs.io/
* 输出富文本：https://nbviewer.jupyter.org/github/ipython/ipython/blob/master/examples/IPython%20Kernel/Rich%20Output.ipynb
* TODO: https://www.dataquest.io/blog/jupyter-notebook-tips-tricks-shortcuts/

### 配置

* jupyter notebook --generate-config 在 .jupyter/jupyter_notebook_config.py 下生成默认配置
* c.NotebookApp.notebook_dir：指定启动目录
* c.NotebookApp.open_browser = False
* c.NotebookApp.ip = '*'
* c.NotebookApp.port = 8888

### 快捷键

* a在上方添加代码块，b在下方，dd是删除
* shift enter运行当前代码块并移动到下一块
* j上移聚焦代码块，k下移
* enter编辑当前块，esc返回一般模式

## Conda

* conda是一款软件管理软件，相当于windows里面的应用商店。miniconda和anaconda中都包含了conda。miniconda只包含了conda、python、和一些必备的软件工具；anaconda包含了数据科学和机器学习要用到的很多软件
* pip只用来安装python的whl和源码，后者有时需要编译器，有的需要操作系统的包管理器安装依赖
* conda用来安装conda package二进制包，大部分是python的，但也支持了不少非python语言的依赖项如mkl、cuda这种c/c++写的包；有些包只能用conda，比如rdkit；但包的总数远少于PyPI
* conda自己可以用来创建虚拟环境，可以很轻松地管理多个版本的python
* conda比pip更加严格，conda会检查当前环境下所有包之间的依赖关系
* conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
* conda create -n venv python=3.8; conda info -e; conda activate venv; conda remove -n venv --all

## Web Server

### FastAPI

* /docs和/redoc能查看api文档，还能进行测试
* 如果路径中需要出现后缀，如`/test.txt`，路径需要声明成`{file_path:path}`
* bool查询参数会自动转换，b=1或者b=True或b=yes都可以

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/", summary='xxx', description='xxx')
async def read_root():
    return {"Hello": "World"}

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: Optional[str] = None): # 自动检测非路径参数；如果不加默认值就必须有
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
* 不以`/`结尾的路由会自动重定向
* taoufik07/responder是一个基于Starlette的类似于Flask的框架，但依赖很多
* Route还可以设置name，之后可用request或app.url_for获取那个名字的url
* 使用类：继承starlette.endpoints.HTTPEndpoint，定义get等方法
* 自带一些中间件：gzip、httpsredirect
* Config封装了.env的读取
* TODO：https://github.com/Redocly/redoc https://github.com/swagger-api/swagger-ui https://www.starlette.io/schemas/

```python
from starlette.applications import Starlette
from starlette.requests import Request
from starlette.responses import PlainTextResponse,HTMLResponse,JSONResponse,RedirectResponse,FileResponse
from starlette.routing import Route, Mount
from starlette.staticfiles import StaticFiles # 此项及FileResponse需要装aiofiles

def homepage(_): ...
def user(request: Request):
    username = request.path_params['username'] # query_params, cookies, body()是bytes, json(), form()['filename'].read()
    return PlainTextResponse('Hello, %s!' % username) # headers, set_cookie()

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

# 手动模拟FileResponse，必须先全部读完再包装成file-like obj，且不具有ETag和Content-Length且不会自动猜测media_type
def myfile(_):
    with open('test.txt', 'rb') as f:
        return StreamingResponse(io.BytesIO(f.read()))
```

### Uvicorn

* pip install uvicorn[standard]：最小需要click和h11，标准版会装上基于Cython的uvloop和watchdog
* uvicorn main:app --host 127.0.0.1 --port 8000：【默】对应main.py的app对象，--reload最小版为轮询
* --uds指定unix socket；--workers多线程
* 默认处理来自于127.0.0.1的X-Forwarded等头，可用--forwarded-allow-ips *信任所有
* scope中有scheme、method、path、headers

```python
async def app(scope, receive, send):
    while more_body:
        message = await receive()
        body += message.get('body', b'')
        more_body = message.get('more_body', False)
    await send({
        'type': 'http.response.start',
        'status': 200,
        'headers': [
            [b'content-type', b'text/plain'],
        ],
    })
    await send({
        'type': 'http.response.body',
        'body': b'Hello, world!',
    })

if __name__ == "__main__": # 从脚本运行
    uvicorn.run("main:app",reload=True)

async def app(scope, receive, send): # 使用Starlette简单封装uvicorn请求
    request = Request(scope, receive)
    response = Response(content, media_type='text/plain')
    await response(scope, receive, send)
```

## ORM和数据库

* peewee
* https://github.com/pudo/dataset：基于sqlalchemy
* SQLAlchemy：等2.0
* https://github.com/marshmallow-code/marshmallow 能把类序列化成json，但标记类型要用该库提供的
* jsonpickle：把任意类序列化成json
* GINO：目前只支持pg
* https://github.com/encode/orm：基于SQLAlchemy core的查询、databases的异步、typesystem的类型验证，但很不活跃
* ponyorm、tortoise-orm
* tinydb：储存数据到json中，用的并不是sql，看作增强版的dict吧，纯Py
* edgedb：自创DML的关系型数据库，但好像是基于pg的

### MySQL

* pymysql：纯Py，当用gevent或者PyPy时可以用
* mysql-connector-python：纯Py，Oracle官方实现，性能貌似比pymysql还要差
* mysqlclient-python：带有C扩展，性能最好

## PySnooper

* 每一行代码的执行
* 变量的声明和赋值
* 返回值
* 持续时间

```python
@pysnooper.snoop(normalize=True)
def f(): ...
# 在函数中的某一部分上使用：with pysnooper.snoop():

# 参数：
snoop('/my/log/file.log', prefix='ZZZ ')
watch=('foo.bar') # 查看非局部变量的值；watch_explode展开字典的内容
depth=2 # 调用其它函数的跟踪深度，默认为1
```

## 杂项

* colorama：控制台的前、背景色；rich：自动染色和格式化
* icecream：用来代替print输出调试信息的，比如无参调用会显示被调用时所在文件和行号，有参调用会显示参数内容和返回值，并再返回那个返回值，用disable()可关掉所有输出，install()可使得无需import也能用
* beeprint：格式化打印dict
* python-dotenv：从`.env`中读取并设置环境变量
* PyYAML：一定要用safe_load；或者用strictyaml
* lazy_import：np = lazy_import.lazy_module("numpy")，且也会将lazy化的模块放到sys.modules里，之后其它模块用的numpy也是lazy的
* chardet：猜测编码
* watchdog：用于监测文件变化
* celery：分布式任务队列，功能强大 https://zhuanlan.zhihu.com/p/22304455；rq：使用Redis的任务队列，简单；dramatiq
* attrs：dataclasses的增强版；pydantic也类似，主要支持数据验证
* r1chardj0n3s/parse：f-string的反向，可以捕获到命名字典里，parse完整匹配，search只要求p是str的一部分且是非贪婪的但有BUG(#41)，findall直接返回列表结果也是非贪婪的
* lexer/parser：https://github.com/lark-parser/lark (扩展的EBNF，功能最多性能好) https://github.com/pyparsing/pyparsing (纯Py语句，自底向上) https://github.com/erikrose/parsimonious (简化了的EBNF，性能好) https://github.com/dabeaz/sly (源于lex/yacc虽为3.6更新了但仍很麻烦，lexer和parser分开) https://github.com/neogeny/TatSu (EBNF，3.8，star很少)；FSM：https://github.com/pytransitions/transitions；支持命令的DSL（感觉不如直接写Py）：https://github.com/textX/textX
* pretty_errors：精简stacktrace，可全局安装
* abersheeran/a2wsgi：ASGI于WSGI的app互转

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

* PyTest https://realpython.com/learning-paths/test-your-python-apps/
* PyNaCl https://github.com/pyca/cryptography
* Seaborn bokeh plotly.py plotly/dash 数据可视化
* https://github.com/dbader/schedule，据说自带的很不好用
* 命令行选项创建工具：https://github.com/docopt/docopt （很久没更新了） https://github.com/pallets/click/ 很复杂但最好 https://github.com/tiangolo/typer；把命令行程序变成GUI：https://github.com/chriskiehl/Gooey
* https://github.com/harelba/q：Run SQL directly on CSV or TSV files；csvkit
* poetry，替代pip+venv：https://zhuanlan.zhihu.com/p/81025311 https://python-poetry.org/
* Nuitka：https://zhuanlan.zhihu.com/p/31721250 https://zhuanlan.zhihu.com/c_1245860717607686144 性能有提高，跨平台差；好像不能单文件
* pytagcloud 中文分词 生成标签云 https://zhuanlan.zhihu.com/p/20432734
* https://github.com/grantjenks/python-diskcache
* https://github.com/Delgan/loguru 日志
* https://github.com/mitmproxy/mitmproxy
* https://github.com/gevent/gevent
* 自动化任务工具invoke：https://zhuanlan.zhihu.com/p/105263640；Fabric https://zhuanlan.zhihu.com/p/107633056
* https://github.com/serge-sans-paille/pythran https://github.com/numba/numba
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
* https://github.com/hugapi/hug 基于 falconry/falcon 的WebAPI框架，但hug有一段时间没提交了，falcon比较活跃但更底层，考虑学falcon，还支持ASGI
* profiler：https://github.com/benfred/py-spy https://github.com/emeryberger/scalene
* GUI：PySimpleGUI kivy DearPyGui
* 和C++交互：pybind11
* streamlit：从程序生成网页，不过主要是为机器学习设计的
* https://github.com/jek/blinker 功能简单的非分布式信号（事件）库
