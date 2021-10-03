## 概念
1. `STATIC_URL`
静态文件的URL前缀
2. `STATIC_ROOT`
该目录下面文件会当成静态文件处理，`STATIC_ROOT`主要用来方面部署`Django App`的，由于写`App`经常会遇到和该App相关静态文件。
通过运行`python manage.py collectstatic`，会自动将各个`App`下面静态文件复制到`STATIC_ROOT`路径。

3.`STATICFILES_DIRS`
`STATICFILES_DIRS`指定了一个工程下面哪个目录存放改工程相关静态文件，所以使用`collectstatic`也会将`STATICFILES_DIRS`指定文件收集到`STATIC_ROOT`目录下面。所以`STATICFILES_DIRS`下面是不能包含`STATIC_ROOT`这个路径的

## 实践
最佳实践的配置方式是将所有`App`下面静态文件统一放置到一个目录下面。然后将改目录设置为
`STATICFILES_DIRS`，`STATIC_ROOT`则设置为另外的目录
```
STATIC_URL = '/static/'
# 开发阶段放置项目自己的静态文件
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'staticfiles'),
)
# 执行collectstatic命令后会将项目中的静态文件收集到该目录下面来（所以不应该在该目录下面放置自己的一些静态文件，因为会覆盖掉）
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```

## 注意
#### 静态文件部署
1. 开发环境
在`Debug = True`时候，`Diango`会把`/static`映射到`django.contrib.staticfiles`这个应用。`staticfiles`应用汇自动搜集各个APP的`static`目录下面搜索静态文件。

2. 线上环境
`staticfiles`会失效，如果希望提供的文件服务，还存放在当前的服务上。
- 收集所有的静态文件
```
python manage.py collectstatic
```
- 配置`settings.py`
```
# 加上文件处理路径
STATIC_ROOT = './static'
```
- 通过` django.views.static.serve()`服务来处理静态文件
配置`urls.py`
```
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # ... the rest of your URLconf goes here ...
] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```