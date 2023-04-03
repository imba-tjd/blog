# Django

* manage.py与django-admin命令行和python -m django做的事一样，只是manage.py还设定了settings的位置。当使用单一项目只有一个设置时，用manage.py更方便。否则要用--settings指定
* 创建项目：django-admin startproject mysite .
  * 会在当前目录下创建manage.py和mysite文件夹
  * mysite含有settings和urls。asgi如果用不到可以删，wsgi在runserver时会用到
* 运行：manage.py runserver [addrport] 会自动重载
  * VSC调试：目标是manage.py，参数是runserver，再加"django": true
* 创建应用。业务上的子模块，也可以打包复用。会在manage.py的同级创建：manage.py startapp 应用名。
* 交互式命令行，代替普通的PyREPL，因为它会读取必要的设置：manage.py shell
* 部署前的设置检查：manage.py check --deploy

## App

* 修改App名字后要修改：settings、总urls、templates文件夹相关，包括render()、extends，静态文件同理

```py
# news/model.py ORM
from django.db import models
class Article(models.Model):
    name = models.CharField(max_length=10)
    text = models.TextField()
    def __str__(self):
        return self.name

# news/admin.py 添加Model进管理面板
from django.contrib import admin
from . import models
admin.site.register(models.Article)

# news/views.py 其实是Controller
from django.http import HttpResponse
from django.shortcuts import render
def index(request):
    return HttpResponse('Hello world')

def year_archive(request, year):
    a_list = Article.objects.filter(pub_date__year=year)
    context = {'year': year, 'article_list': a_list}
    return render(request, 'news/year_archive.html', context) # 寻找news/templates/news/year_archive.html

# news/urls.py 子路由
from django.urls import path
from . import views
app_name = 'news' # 可选，若有，则urlpatterns中的路由名将添加 news: 前缀
urlpatterns = [
    path('', views.index, name='路由名'),
    path('articles/<int:year>/', views.year_archive [, {行内指定参数}])
    转换器还支持str、slug(如a-b_1)、path(匹配非空字段，包括/)、命名正则捕获组re_path('^prefix/(?P<name>pattern)$')
    默认值：给view的参数加。两个路由规则都指向它，一个有路由参数，一个没有
]

# mysite/urls.py 总路由
from django.contrib import admin
from django.urls import include, path
urlpatterns = [
    path('news/', include('news.urls')), 若捕获了参数，会传递给子路由。include还可接收list[path]，用于处理公共前缀
    path('admin/', admin.site.urls), admin是唯一不需要include()的
]
handler404 = 自定义视图对象或'mysite...'

# mysite/settings.py
INSTALLED_APPS = ['news.apps.NewsConfig'] # 必须手动注册，否则就和没有一样
# 也可以写'news'和删掉app.py
```

## [settings](https://docs.djangoproject.com/zh-hans/4.1/ref/settings/)

* 在代码中读取值：from django.conf import settings

```py
DEBUG 默认为False，模板默认True
ALLOWED_HOSTS 在DEBUG=False时必须设置，可为['*']，'.a.com'将匹配b.a.com和a.com。默认为[]只支持DEBUG下本地访问
USE_X_FORWARDED_HOST 默认False

WSGI_APPLICATION 可以去掉，会按约定值检测
USE_I18N 默认True。设为False后能提高性能
TIME_ZONE 默认America/Chicago，模板默认UTC，应手动改为当前系统时区如Asia/Shanghai，无法设为UTC+8
USE_TZ 5.0默认True，表示内部用UTC，再根据TIME_ZONE转化显示。设为False时总是使用当地时间

CommonMiddleware：APPEND_SLASH 默认True，当不匹配已知路由且不以/结尾时，自动发出重定向请求加/。PREPEND_WWW 默认False

LANGUAGE_CODE 默认en-us。只有启用USE_I18N此项才有效果。启用语言中间件（能检测用户请求要用的语言）后此项表示Fallback，不启用时就用此项的
```

## django.http 请求和响应

* HttpRequest
  * method 如'POST'
  * GET URL参数，类dict
  * POST 表单提交的字符串数据，类dict
* 响应
  * HttpResponseRedirect(reverse('路由名', args=(xxx,)))
  * raise Http404('message')
  * JsonResponse({})
* @django.views.decorators.http.require_http_methods(["GET", "POST"]) / require_POST / require_safe表示只接受GET和HEAD

## [模板](https://docs.djangoproject.com/zh-hans/4.1/ref/templates/builtins/)

* 放在mysite/appname/templates/appname/下。不推荐不再写一遍appname，因为不同应用的tempalte没有区分，只有再写一遍才能保持前缀
* VSC：支持极差
* 变量：{{ xxx }}
  * dict和list可用d.key和l.0代替[]
  * “点”运算符也支持无参方法不加括号
* 注释：{# #}、{% comment %} 多行注释 {% endcomment %}
* 循环
  * {% for e in arr %} e.prop {% empty %} arr为Falsy时输出 {% endfor %}
  * 解包：for a,b in arr
  * forloop变量：counter表示从1开始的索引，counter0从0开始
  * ifchanged：检查一个值是否在循环的最后一次迭代中发生了变化
* 判断：{% if xxx %} {% else %} {% endif %}
  * xxx可以先使用filter：if messages|length >= 100
  * 不支持 a > b > c
  * 输出第一个不是Falsy的值：firstof var1 var2 "fallback"
* 模板继承
  * base.html：{% block 块名 %} 默认内容 {% endblock (可选块名) %}
  * 子模板：第一行 extends "base.html"，body里用 block 相同的块名 重写内容
* 模板组合：include "xxx.html" 会自动传递context；可用with k=v k2=v2传递额外的，only表示仅使用行内提供的
* tag：{% xxx arg1 arg2 %}
  * url '路由名' 路由参数：URL反向解析，读取路由名，生成对应的链接，用在href中避免模板中硬编码应用名。Py代码中用django.urls.reverse()
  * load a b 加载自定义tag集
  * now "pattern" 规则见模板参考的date部分。可用"DATE_FORMAT"等四项表示从设置里取预定义的
  * csrf_token：放在form里，原生只用于表格POST。view如果开了缓存要在@cache_page下面加@csrf_protect
  * verbatim; end：停止渲染此部分内容
  * with var=xxx; end：相当于声明变量，当多次访问xxx很昂贵时使用
* filter：{{ xxx | 过滤器名:"arg" }}
  * default:"no" 当变量未定义或为Falsy时显示为no。还有default_if_none。设置的TEMPLATE_STRING_IF_INVALID用于未定义时显示的值，默认不显示。里面的内容没有自动转义
  * add:"数字"或变量 如果两者都能转换为数字，则执行数学上的相加，否则对于字符串或列表也执行它们对应的+
  * join:", "
  * truncatewords:"100"
  * filesizeformat 把bytes转换成nMB等人类可读格式
  * length、length_is:"n"
  * dictsort:"key" 对dict按key排序
  * escapejs 当xxx含有\n时转义成\\n等，用于提供给js字符串
  * floatformat 默认将小数四舍五入到一位
  * urlencode 将: ? & = 中文 进行URL编码
  * json_script:"id" 将对象输出为JSON放到script id="id"中。JS中用JSON.parse(document.getElementById('id').textContent)
  * safe 避免自动转义> < ' " &
* 设置的OPTIONS的context_processors：给context添加一些属性

## ORM

* 创建数据库db.sqlite3：manage.py makemigrations; migrate。查看会执行什么sql语句：sqlmigrate 应用名 0001
* 官方支持PG MySQL
* 会自动创建名为id的bigint主键
* 字段：https://docs.djangoproject.com/zh-hans/4.1/ref/models/fields
  * 类型：DateField() ForeignKey(Reporter, on_delete=models.CASCADE) IntegerField(default=0)
  * 选项：null默认False blank用于表单验证默认False default unique help_text随表单显示 choices=models.TextChoices('选项1','选项2').choices
  * ManyToManyField OneToOneField

p = Article(name=xxx); p.save() 或 Article.objects.create()
Article.objects.all() / get(id=1/name__startswith=xxx/name__contains=xxx) / filter() / order_by(减号表示反向排序)
get()不存在时抛 那个Model类.DoesNotExist
o = django.shortcuts.get_object_or_404(Person, pk=art_id) get_list_or_404() 如果不存在则自动返回404

values_list

### [设置](https://docs.djangoproject.com/zh-hans/4.1/ref/databases/)

```py
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "mydatabase",
        "USER": "mydatabaseuser",
        "PASSWORD": "mypassword",
        "HOST": "127.0.0.1",
        "PORT": "5432",
    }
}
DEFAULT_AUTO_FIELD 模板中默认为BigAutoField，本身默认为AutoField。修改此值不会迁移，去掉会产生警告
数据库设置中的TIME_ZONE 只有当连接非DJ管理的数据库时才设置
CONN_MAX_AGE 持久连接。当连接本身很耗时时可以开，设为None表示无限，必须小于服务器的持久连接限制。还可以开CONN_HEALTH_CHECKS
ATOMIC_REQUESTS
```

## 类视图、通用视图

* template_name默认为appname/modelname_detail.html
* 自动添加context，不同通用视图的名称不同，如DetailView是model转小写，ListView是加_list。或用context_object_name指定名称
* DetailView
  * 需要URL中存在名为pk的参数，代表主键值

```py
from django.views import generic

class IndexView(generic.ListView):
    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]

class DetailView(generic.DetailView):
    model = Question

.as_view()
```

## 测试

* 运行：manage.py test appname。testserver：使用给定的fixture启动服务器
* 手动：DJshell中django.test.utils.setup_test_environment()
* HTTP Client：django.test.Client() API风格与requests类似，但实现完全不同。resp有context

```py
class QuestionModelTests(TestCase):
    def test_was_published_recently_with_future_question(self): # 要以test开头
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)
```

## 静态文件

* 放在mysite/appname/static/appname/下
* 模板中使用：load static、static 'appname/xxx.jpg'
* 部署
  * 必须设置STATIC_ROOT=某绝对路径，可用`BASE_DIR/'static'`
  * 在生产环境机器上manage.py collectstatic --noinput，会把所有静态文件收集复制到那个文件夹中，在nginx里设置访问/static时使用它们
  * -l产生软链接
* 由django响应静态文件
  * runserver无需先收集，加--insecure则非DEBUG也无需收集
  * 自定义静态文件路由：`urlpatterns.append(path(settings.STATIC_URL.lstrip('/') + '<path:path>', django.views.static.serve, {'document_root': settings.STATIC_ROOT}))`
  * 自定义静态文件，且DEBUG下由django响应，非DEBUG和没有一样，方便开发，也适用于其它WSGI服务器运行：urlpatterns += django.contrib.staticfiles.urls.staticfiles_urlpatterns('static2/') 无参时默认为static且runserver提前拦截处理该路由了
* STATIC_URL：静态文件的路由前缀，如有需要可设为 http://static.xxx.com/ 这样的绝对路径
* WhiteNoise：响应静态文件，支持gzip和brotli压缩
* django-compressor：minify css和js

## 文件存储

* 当ORM使用FileField或ImageField时，表示用户上传的文件，它们是django.core.files.File对象，储存在MEDIA_URL下
* 默认情况下文件储存用自带的FileSystemStorage，需设置MEDIA_ROOT，要与静态文件的不同
* 4.2新增内存储存
* 不会自动serve它们。仅在开发阶段serve：urlpatterns += django.conf.urls.static.static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
* 修改存储后端：4.2的设置里改STORAGES，类似于数据库的设置

## 缓存

* 支持memcached、redis、数据库(配合createcachetable命令)、本地目录。不设置时默认用本地内存
* 缓存本身超时时间默认5分钟。Expires头受 CACHE_MIDDLEWARE_SECONDS影响，默认10分钟
* 全站缓存：添加UpdateCache和FetchFromCache中间件，前者需放在任何可能添加Vary头的中间件之前，包括Session GZip Locale，后者放在最后
* 视图缓存：@django.views.decorators.cache.cache_page
* 默认根据fully-qualified URL进行缓存，包括查询字符串。@django.views.decorators.vary.vary_on_headers()/@vary_on_cookie添加基于请求头的差异而缓存不同内容
* @cache_control(private=True)、@never_cache
* 在代码中按类dict使用：django.core.cache.cache
* 条件视图装饰器：@django.views.decorators.http.condition(etag_func=xxx, last_modified_func=xxx) 其中xxx是一个计算需要的东西的函数，如果满足，则返回304。如果只需要其中一个，可用@etag或@last_modified，但两个都需要时不能叠加使用，只能用condition
* ConditionalGetMiddleware：需等生成完了视图才计算添加ETag，不能减少计算，只能减少网络流量。而装饰器可使用指定的低消耗函数判断是否更改了，没到生成视图的地方

## 日志

```
That's a very simple logging, that should output everything to console from all modules with level of DEBUG, which means output everything it can.
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler'
        },
    },
    'loggers': {
        '': {  # 'catch all' loggers by referencing it with the empty string
            'handlers': ['console'],
            'level': 'DEBUG',
        },
    },
}
```

## Admin

* 创建管理员账号：manage.py createsuperuser --user admin --email admin@example.com
* 会产生静态文件，也需要迁移数据库
* 依赖auth
* https://docs.djangoproject.com/zh-hans/4.1/intro/tutorial07/
* https://docs.djangoproject.com/zh-hans/4.1/ref/contrib/admin/

## Auth

* https://docs.djangoproject.com/zh-hans/4.1/topics/auth/

## Session

* https://docs.djangoproject.com/zh-hans/4.1/topics/http/sessions/

## [中间件](https://docs.djangoproject.com/zh-hans/4.1/ref/middleware)

* Common：设置非流响应的Content-Length头
* GZip：响应较小时自动不压缩，能处理客户端的Accept-Encoding，若响应已有Content-Encoding则不压缩。提供@gzip_page单个视图
* Security：设置Content-Type为nosniff、CORS为same-origin、referrer-policy为same-origin（TODO:说是HTTPS下CSRF中间件需要此头，需测试外部是HTTPS但django和网关是http的情形是否需要）、一些默认不开的HSTS设置
* ConditionalGet

## 其它contrib

* messages消息
  * 应用场景：处理完一个表单或一些其他类型的用户输入后，向用户显示一个一次性的通知消息
  * 依赖MIDDLEWARE：Session、Message。被admin依赖
* contenttypes
  * 用于跟踪Django项目中安装的所有model，为我们提供更高级的模型接口。是一些“额外的数据表”
  * 被admin、auth依赖
* humanize：如4500变成4,500，某个时间变成an hour ago、yesterday，1000000变成1.0 million
* redirects：管理重定向
* sitemap

## 单文件运行

```py
import html
import os
import sys

from django.conf import settings
from django.core.wsgi import get_wsgi_application
from django.http import HttpResponse
from django.urls import path
from django.utils.crypto import get_random_string

settings.configure(
    DEBUG=(os.getenv("DEBUG") == "1"),
    ALLOWED_HOSTS=["*"],
    ROOT_URLCONF=__name__,
    SECRET_KEY=get_random_string(50),
)

def index(request):
    name = request.GET.get("name", "World")
    return HttpResponse(f"Hello, {html.escape(name)}!")

urlpatterns = [
    path("", index),
]

app = get_wsgi_application()

if __name__ == "__main__":
    from django.core.management import execute_from_command_line
    execute_from_command_line(sys.argv)
# DEBUG=1 python app.py runserver
```

## 其它项目

* REST：django-rest-framework(https://appliku.com/post/django-rest-framework-openapi-3 djangorestframework-simplejwt)、django-ninja
* CMS：Wagtail、Django-CMS
* django-allauth 一些OAuth登录，国内的只有微博
* django-environ
* django-storages 支持S3、FTP、Azure、Google
* django-cors-headers
* django-braces 类视图Mixin
* django-extensions
* django-post-office 邮件后端
* django-crispy-forms 美化表单
* django-import-export 操纵数据
* djangoql 增强Admin的搜索功能，最后更新2022年1月
* django-revproxy 针对单一站点的反代

## Jinja

* VSC：Better Jinja。不要用叫做Jinja的扩展，早就不更新了

### 手动渲染

```py
from jinja2 import Environment, PackageLoader, select_autoescape
env = Environment(
    loader=PackageLoader("yourapp"), # 会在yourapp/templates/中搜索，还可用FileSystemLoader('templates')
    autoescape=select_autoescape(),
    enable_async=True # 之后可用render_async
)
template = env.get_template("mytemplate.html")
print(template.render(the="variables", go="here"))
```

## 遇到的错误

* `TemplateDoesNotExist; django.template.loaders.app_directories.Loader: .venv\Lib\site-packages\django\contrib\auth\templates\appname\index.html (Source does not exist)`：看看是不是templates文件夹名称打错了
* `ModuleNotFoundError: No module named 'site.settings'; 'site' is not a package`：项目设置文件夹不能叫site，这名字已经被内置包占用了

## wsgi服务器

* uwsgi --module=mysite.wsgi --http=0.0.0.0:80
* gunicorn --bind=0.0.0.0:80 --forwarded-allow-ips="*"

## djangorestframework

```py
INSTALLED_APPS = ['rest_framework']
REST_FRAMEWORK = {
    # Use Django's standard `django.contrib.auth` permissions,
    # or allow read-only access for unauthenticated users.
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly'
    ]
}

from django.urls import path, include
from django.contrib.auth.models import User
from rest_framework import routers, serializers, viewsets
# Serializers define the API representation.
class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = ['url', 'username', 'email', 'is_staff']

# ViewSets define the view behavior.
class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer

router = routers.DefaultRouter()
router.register(r'users', UserViewSet)

urlpatterns = [
    path('', include(router.urls)),
    path('api-auth/', include('rest_framework.urls')) # browsable API
]
```

* browsable API：urlpatterns加path('api-auth/', include('rest_framework.urls'))


## TODO

CSRF：
https://docs.djangoproject.com/zh-hans/4.1/ref/csrf/
https://docs.djangoproject.com/zh-hans/4.1/howto/csrf
https://www.squarefree.com/securitytips/web-developers.html


from django.views.decorators.debug import sensitive_post_parameters @sensitive_post_parameters('password', 'credit_card_number') view 存在多个装饰器时应放在最上面

发生500错误时，会向ADMINS列表中的用户发送邮件。但需要先配置EMAIL_XXX

https://docs.djangoproject.com/zh-hans/4.1/
https://docs.djangoproject.com/zh-hans/4.1/topics/
https://docs.djangoproject.com/zh-hans/4.1/ref/templates/builtins/#linebreaks
https://docs.djangoproject.com/zh-hans/4.1/topics/http/file-uploads/

https://tutorial.djangogirls.org/zh/django_forms/
https://www.liujiangblog.com/course/django/95
https://realpython.com/learning-paths/django-web-development/
https://appliku.com/post/django-project-tutorial-beginners-settings-docker#django-custom-user-model


[auth]
    changepassword

[contenttypes]
    remove_stale_contenttypes

[django]
    check
    compilemessages
    dbshell
    dumpdata
    flush
    inspectdb 从数据库生成py
    loaddata
    makemessages
    optimizemigration
    sendtestemail
    showmigrations
    sqlflush
    sqlmigrate
    sqlsequencereset
    squashmigrations

[sessions]
    clearsessions
