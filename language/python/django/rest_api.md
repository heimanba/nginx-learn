# rest API
## 原生支持REST API


## 使用 `django-rest-framework`模块支持
1. 下载依赖
```
pip install django-rest-framework django-cors-headers
```
2. 新建一个`core`app用于rest api
```
python manage.py startapp core
```
3. 全局添加配置 `settings.py`
```
# ...

INSTALLED_APPS = [
    # 默认的 App ...

    'rest_framework',
    'corsheaders',
    'core',
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    # 默认的中间件 ...
]

CORS_ORIGIN_WHITELIST = (
    'http://localhost:3000',
)

# ...

LANGUAGE_CODE = 'zh-hans'

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```
4. `core/models.py`定义model
```
from django.db import models


class Recipe(models.Model):
    DIFFICULTY_LEVELS = (
        ('Easy', '容易'),
        ('Medium', '中等'),
        ('Hard', '困难'),
    )
    name = models.CharField(max_length=120, verbose_name='名称')
    ingredients = models.CharField(max_length=400, verbose_name='食材')
    picture = models.FileField(verbose_name='图片')
    difficulty = models.CharField(choices=DIFFICULTY_LEVELS, max_length=10,
                                  verbose_name='制作难度')
    prep_time = models.PositiveIntegerField(verbose_name='准备时间')
    prep_guide = models.TextField(verbose_name='制作指南')

    class Meta:
        verbose_name = '食谱'
        verbose_name_plural = '食谱'

    def __str__(self):
        return '{} 的食谱'.format(self.name)
```
5. `core/admin.py`配置管理`Recipe`字段
```
from django.contrib import admin
from .models import Recipe

# Register your models here.
admin.site.register(Recipe)
```
6. 新增`core/serializers.py`,在Rest API需要时候，提供JSON序列化
```
from rest_framework import serializers
from .models import Recipe


class RecipeSerializer(serializers.ModelSerializer):

    class Meta:
        model = Recipe
        fields = (
            'id', 'name', 'ingredients', 'picture',
            'difficulty', 'prep_time', 'prep_guide'
        )
```
7. `core/views.py`定义视图
```
from rest_framework import viewsets
from .serializers import RecipeSerializer
from .models import Recipe


class RecipeViewSet(viewsets.ModelViewSet):
    serializer_class = RecipeSerializer
    queryset = Recipe.objects.all()
```
8. 定义路由
- `core/urls.py`配置
```
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import RecipeViewSet

router = DefaultRouter()
router.register(r'recipes', RecipeViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```
- 全局路由生效
```
from django.urls import path, include
urlpatterns = [
    # ...
    path('api/', include('core.urls')),
]
```

