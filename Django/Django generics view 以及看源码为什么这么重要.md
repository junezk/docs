# Django generics view 以及看源码为什么这么重要

Django generics view 以及看源码为什么这么重要 这对你理解django 以及后续学习django rest framework 很重要的

关于阅读代码
------

我们知道，得益于python的语言特性，python的源码是直接可以看的到的，而django是一种大而全的东东，虽然django的文档看似全面，但实际上有些模块写的确实不怎么样，而且平时遇到的需求也是多变的，有的时候你需要实现某种诡异的功能，当然我们可以用google或者做github的搬运工，但是有的时候框架已经给你做了很多事情，你只需要在多做那么一丢丢就能实现某种需求，这个时候看代码比去扒github是快的多的。

先稍微扯一下Django generics view
--------------------------

这玩意儿通常你百度或者google的时候，大部分时间会跟你讲这是**Class-based views**。

第一次接触的时候，我觉得他牛逼的地方在于只要在子类里面定义get post方法，就不需要写类似这种东西了

        if request.method.lower() == 'get':
            do_something()
        else:
            do_otherthing()

其实与普通的method view不同的是，他的父类定义了一堆有用的方法，当你知道这些的时候你可以少些超多的代码来做一些屌屌的事情。

来看下使用示例
-------

防止文件被删除，贴几段过来

    class BaseMixin(object):
        
        def get_context_data(self,*args,**kwargs):
            context = super(BaseMixin,self).get_context_data(**kwargs)
            try:
                #热门文章
                context['hot_article_list'] = Article.objects.order_by("-view_times")[0:10]
                #导航条
                context['nav_list'] =  Nav.objects.filter(status=0)
                #最新评论
                context['latest_comment_list'] = Comment.objects.order_by("-create_time")[0:10]
    
            except Exception as e:
                logger.error(u'[BaseMixin]加载基本信息出错')
    
            return context


​    
    class IndexView(BaseMixin,ListView):
        template_name = 'blog/index.html'
        context_object_name = 'article_list'
        paginate_by = PAGE_NUM #分页--每页的数目
        
        def get_context_data(self,**kwargs):
            #轮播
            kwargs['carousel_page_list'] = Carousel.objects.all()
            return super(IndexView,self).get_context_data(**kwargs)
    
        def get_queryset(self):
            article_list = Article.objects.filter(status=0)
            return article_list

[这是项目原来的地址](http://git.oschina.net/billvsme/vmaig_blog/blob/master/blog/views.py?dir=0&filepath=blog/views.py&oid=b1aa8b945769be610d9764315dd1ef21cc13141b&sha=d6b84374362791d01f7c9a9933d645387cb189ae)

如果你之前只写过method的view，你可能会觉得很诡异，为毛这段代码定义了几个属性，重写了几个方法就实现了我之前写的那么一大坨长长的代码？这种时候就必须看源码了，比文档来的直观。

generic view代码讲解
----------------

OK先贴一波源码，django的generic view有很多，就讲下最常见的list。

这是 base.py里面的东东

    class View(object):
        """
        Intentionally simple parent class for all views. Only implements
        dispatch-by-method and simple sanity checking.
        """
    
        http_method_names = ['get', 'post', 'put', 'delete', 'head', 'options', 'trace']
    
        def __init__(self, **kwargs):
            """
            Constructor. Called in the URLconf; can contain helpful extra
            keyword arguments, and other things.
            """
            # Go through keyword arguments, and either save their values to our
            # instance, or raise an error.
            for key, value in kwargs.iteritems():
                setattr(self, key, value)
    
        @classonlymethod
        def as_view(cls, **initkwargs):
            """
            Main entry point for a request-response process.
            """
            # sanitize keyword arguments
            for key in initkwargs:
                if key in cls.http_method_names:
                    raise TypeError(u"You tried to pass in the %s method name as a "
                                    u"keyword argument to %s(). Don't do that."
                                    % (key, cls.__name__))
                if not hasattr(cls, key):
                    raise TypeError(u"%s() received an invalid keyword %r" % (
                        cls.__name__, key))
    
            def view(request, *args, **kwargs):
                self = cls(**initkwargs)
                if hasattr(self, 'get') and not hasattr(self, 'head'):
                    self.head = self.get
                return self.dispatch(request, *args, **kwargs)
    
            # take name and docstring from class
            update_wrapper(view, cls, updated=())
    
            # and possible attributes set by decorators
            # like csrf_exempt from dispatch
            update_wrapper(view, cls.dispatch, assigned=())
            return view
    
        def dispatch(self, request, *args, **kwargs):
            # Try to dispatch to the right method; if a method doesn't exist,
            # defer to the error handler. Also defer to the error handler if the
            # request method isn't on the approved list.
            if request.method.lower() in self.http_method_names:
                handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
            else:
                handler = self.http_method_not_allowed
            self.request = request
            self.args = args
            self.kwargs = kwargs
            return handler(request, *args, **kwargs)
    
        def http_method_not_allowed(self, request, *args, **kwargs):
            allowed_methods = [m for m in self.http_method_names if hasattr(self, m)]
            logger.warning('Method Not Allowed (%s): %s', request.method, request.path,
                extra={
                    'status_code': 405,
                    'request': self.request
                }
            )
            return http.HttpResponseNotAllowed(allowed_methods)

这是list.py里面的东东

    class MultipleObjectMixin(object):
        allow_empty = True
        queryset = None
        model = None
        paginate_by = None
        context_object_name = None
        paginator_class = Paginator
    
        def get_queryset(self):
            """
            Get the list of items for this view. This must be an interable, and may
            be a queryset (in which qs-specific behavior will be enabled).
            """
            if self.queryset is not None:
                queryset = self.queryset
                if hasattr(queryset, '_clone'):
                    queryset = queryset._clone()
            elif self.model is not None:
                queryset = self.model._default_manager.all()
            else:
                raise ImproperlyConfigured(u"'%s' must define 'queryset' or 'model'"
                                           % self.__class__.__name__)
            return queryset
    
        def paginate_queryset(self, queryset, page_size):
            """
            Paginate the queryset, if needed.
            """
            paginator = self.get_paginator(queryset, page_size, allow_empty_first_page=self.get_allow_empty())
            page = self.kwargs.get('page') or self.request.GET.get('page') or 1
            try:
                page_number = int(page)
            except ValueError:
                if page == 'last':
                    page_number = paginator.num_pages
                else:
                    raise Http404(_(u"Page is not 'last', nor can it be converted to an int."))
            try:
                page = paginator.page(page_number)
                return (paginator, page, page.object_list, page.has_other_pages())
            except InvalidPage:
                raise Http404(_(u'Invalid page (%(page_number)s)') % {
                                    'page_number': page_number
                })
    
        def get_paginate_by(self, queryset):
            """
            Get the number of items to paginate by, or ``None`` for no pagination.
            """
            return self.paginate_by
    
        def get_paginator(self, queryset, per_page, orphans=0, allow_empty_first_page=True):
            """
            Return an instance of the paginator for this view.
            """
            return self.paginator_class(queryset, per_page, orphans=orphans, allow_empty_first_page=allow_empty_first_page)
    
        def get_allow_empty(self):
            """
            Returns ``True`` if the view should display empty lists, and ``False``
            if a 404 should be raised instead.
            """
            return self.allow_empty
    
        def get_context_object_name(self, object_list):
            """
            Get the name of the item to be used in the context.
            """
            if self.context_object_name:
                return self.context_object_name
            elif hasattr(object_list, 'model'):
                return smart_str('%s_list' % object_list.model._meta.object_name.lower())
            else:
                return None
    
        def get_context_data(self, **kwargs):
            """
            Get the context for this view.
            """
            queryset = kwargs.pop('object_list')
            page_size = self.get_paginate_by(queryset)
            context_object_name = self.get_context_object_name(queryset)
            if page_size:
                paginator, page, queryset, is_paginated = self.paginate_queryset(queryset, page_size)
                context = {
                    'paginator': paginator,
                    'page_obj': page,
                    'is_paginated': is_paginated,
                    'object_list': queryset
                }
            else:
                context = {
                    'paginator': None,
                    'page_obj': None,
                    'is_paginated': False,
                    'object_list': queryset
                }
            context.update(kwargs)
            if context_object_name is not None:
                context[context_object_name] = queryset
            return context


​    
    class BaseListView(MultipleObjectMixin, View):
        def get(self, request, *args, **kwargs):
            self.object_list = self.get_queryset()
            allow_empty = self.get_allow_empty()
            if not allow_empty and len(self.object_list) == 0:
                raise Http404(_(u"Empty list and '%(class_name)s.allow_empty' is False.")
                              % {'class_name': self.__class__.__name__})
            context = self.get_context_data(object_list=self.object_list)
            return self.render_to_response(context)

首先是为什么不需要写if else判断，而在子类里面定义get post即可，可以看到是下面的dispatch方法做了通用的处理。

        def dispatch(self, request, *args, **kwargs):
            # Try to dispatch to the right method; if a method doesn't exist,
            # defer to the error handler. Also defer to the error handler if the
            # request method isn't on the approved list.
            if request.method.lower() in self.http_method_names:
                handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
            else:
                handler = self.http_method_not_allowed
            self.request = request
            self.args = args
            self.kwargs = kwargs
            return handler(request, *args, **kwargs)

而通用的listview实际上就是一个父类实现了get方法，其中的MultipleObjectMixin实现了超多的方法供你重写，基本上大部分时间我们重写MultipleObjectMixin里面的对应方法以及设置合适的属性就可以完成大部分的业务逻辑。

BaseListView首先通过 `self.get_queryset()` 方法拿到了你需要list的对象，中间一坨是分页的东东，下面的context则是最后需要render\_to\_response时传的参数，这样看就跟以前自己通过method view写的东西一一对应了。而我们只需要根据不同的业务逻辑实现里面的覆盖实现里面的方法即可，有些连覆盖都不需要是要在设置合适的属性，到这里我就不需要根据具体的方法具体讲解了，大家自己看。

    class BaseListView(MultipleObjectMixin, View):
        def get(self, request, *args, **kwargs):
            self.object_list = self.get_queryset()
            allow_empty = self.get_allow_empty()
            if not allow_empty and len(self.object_list) == 0:
                raise Http404(_(u"Empty list and '%(class_name)s.allow_empty' is False.")
                              % {'class_name': self.__class__.__name__})
            context = self.get_context_data(object_list=self.object_list)
            return self.render_to_response(context)

一个现实中的例子
--------

上面仅仅是如何使用django默认的各种generic view，但是有时候默认的机制并不能实现你的需求，例如现在在移动端的大潮下，html配饰移动端一般有两种方案：1.响应式；2.两套页面完全分开。两种方案各有利弊，如果我们需要根据根据user agent来做不同终端的页面渲染，甚至是不同的逻辑（web和移动端逻辑不同这很正常），这种时候很容易想到重写dispatch方法，来做到一种通用的处理方式。

    from django.views.generic import View as DjangoView
    
    class View(DjangoView):
    
        def _get_handler(self, request):
            ''' 
            根据ua获取handler
            '''
            handler_name = request.method.lower()
            if handler_name in self.http_method_names:
                handler = getattr(self, handler_name)
                if not handler:
                    return self.http_method_not_allowed
    
            user_agent = request.META.get('HTTP_USER_AGENT', '').lower()
            handler = getattr(self, handler_name, self.http_method_not_allowed)
            user_agents = ['ipad', 'iphone', 'ipod', 'androidtv', 'android']
            for ua in user_agents:
                if ua in user_agent:
                    handler_name = '{}_{}'.format(handler_name, ua) 
                    break
            return getattr(self, handler_name) or handler
            
        def dispatch(self, request, *args, **kwargs):
            # Try to dispatch to the right method; if a method doesn't exist,
            # defer to the error handler. Also defer to the error handler if the
            # request method isn't on the approved list.
            # 支持user agent跳转
            # 如果实现了对应的方法，则直接使用对应ua的规则
            # 例如method=get, ua为iphone的ua，子类实现 get_iphone, 则使用get_iphone进行render
            handler = self._get_handler(request)
            self.request = request
            self.args = args
            self.kwargs = kwargs
            return handler(request, *args, **kwargs)