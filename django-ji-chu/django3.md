# views详解

## 函数视图

1. 基础视图分析: 1. request 里面有所有的信息,他的信息是我们的wsgi解析之后的报文,回想一下http协议,他所有的信息都在报文里面,是一个很长的字符串,但是我们看到的request里面是dict和object,说明django框架已经帮我们解析好了报文里面的信息 2. HttpResponse是django帮我们封装响应报文,返回我们想给浏览器返回的信息,当然也有其他的返回值,他们都是django帮我们封装好的

   ```text
    Http404
    HttpResponseForbidden
    ...
   ```

   > movies/views.py

   ```text
        def test1(request):
            return HttpResponse("Hello, world!")
   ```

2. 快捷函数
   1. 我们来写一个能够渲染模板的函数,那么他需要使用HttpResponse来给用户返回信息

      > movies/views.py

      ```text
           def test5(request):
           from django.template import loader
           t = loader.get_template('movies/index.html')
           return HttpResponse(t.render({},request))
      ```

   2. 如果我们觉得这样写很麻烦,那么我们可以使用快捷函数 render来帮我们做这个事情,凡是在 from django.shortcuts 中定义的函数,都是帮我们快捷的完成一些功能的函数

      > movies/views.py

      ```text
           def test2(request):

           return render(request,'movies/index.html')
      ```

   3. 查看一下render的源码,我们发现 其实就是我们刚才的原始函数,不过他帮我们做了一层封装

      ```text
           def render(request, template_name, context=None, content_type=None, status=None, using=None):
               """
               Returns a HttpResponse whose content is filled with the result of calling
               django.template.loader.render_to_string() with the passed arguments.
               """
               content = loader.render_to_string(template_name, context, request, using=using)
               return HttpResponse(content, content_type, status)
      ```

   4. 其他快捷函数
      1. redirec 重定向,在访问一个网页的时候,帮助我们重定向

         ```text
          def test6(request):

              return redirect('https://www.baidu.com')
         ```

      2. get\_object\_or\_404 帮助我们查找数据,如果没有数据则返回404的错误

         ```text
              def test7(request):
                  obj = get_object_or_404(Test,pk=1)

                  return HttpResponse(obj.name)
         ```

      3. 课堂练习 1. 如果不使用 redict快捷函数那么要怎么实现 2. 如果不使用get\_object\_or\_404 那么要怎么实现
3. 装饰器
   1. 使用装饰器去控制函数视图的行为

      \`\`\`

      ```text
       from django.views.decorators.http import require_http_methods
      ```

```text
        @require_http_methods(['GET'])
        def test8(request):
            return HttpResponse('hello')

    ```

    1. require_http_methods 控制行为,list传递
    2. require_GET get行为
    3. require_POST() post行为
```

## 类视图

```text
lietview
deleteview
```

