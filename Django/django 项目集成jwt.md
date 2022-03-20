# django 项目集成jwt

## 1、安装djangorestframework-simplejwt

```
pip install djangorestframework-simplejwt
```

然后需要在django的配置上增加：

```
REST_FRAMEWORK = {
    ...
    'DEFAULT_AUTHENTICATION_CLASSES': (
        ...
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    )
    ...
}

JWT_AUTH = {  # 导包： import datetime
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=1),  # jwt有效时间
}
```

在 `urls.py` 文件中定义获取token和刷新token的路径

```
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    ...
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    ...
]
```

