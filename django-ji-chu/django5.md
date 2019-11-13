# url详细,传递参数给views

from django.conf.urls import url

from . import views

urlpatterns = \[ url\(r'^blog/$', views.page\), url\(r'^blog/page\(?P\[0-9\]+\)/$', views.page\), \]

## View \(in blog/views.py\)

def page\(request, num="1"\):

```text
# Output the appropriate page of blog entries, according to num.
...
```

