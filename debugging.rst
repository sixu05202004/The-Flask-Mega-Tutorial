.. _debugging:


调试，测试以及优化
====================

我们小型的 *microblog* 应用程序已经足够的完善了，因此是时候准备尽可能地清理不用的东西。近来，一个读者反映了一个奇怪的数据库问题，我们今天将会调试它。这也提醒我们不论我们是多小心以及测试我们应用程序多仔细，我们还是会遗漏一些 bug。用户是很擅长发现它们的！

不是仅仅修复此错误，然后忘记它，直到我们遇到另一个。我们会采取一些积极的措施，以更好地准备下一个。

在本章的第一部分，我们将会涉及到 *调试*，我将会展示一些我平时调试复杂问题的技巧。

接着我们想要看看如何衡量我们的测试策略的效果。我们想要清楚地知道我们的测试能够覆盖到应用程序的多少，有时候称之为 *测试覆盖率*。

最后，我们将会研究下许多应用程序遭受过的一类问题，性能糟糕。我们将会看到 *调优* 技术，并且发现我们应用程序中哪些部分最慢。

听起来不错吧？让我们开始吧！


Bug
-------

这个问题是由本系列的一个读者发现，他实现了一个允许用户删除自己的 blog 的新函数后遇到这个问题。我们正式的 *microblog* 版本是没有这个功能的，因此我们快速的加上它，以便我们可以调试这个问题。

我们使用删除 blog 的视图函数如下(文件 *app/views.py*)::

    @app.route('/delete/<int:id>')
    @login_required
    def delete(id):
        post = Post.query.get(id)
        if post == None:
            flash('Post not found.')
            return redirect(url_for('index'))
        if post.author.id != g.user.id:
            flash('You cannot delete this post.')
            return redirect(url_for('index'))
        db.session.delete(post)
        db.session.commit()
        flash('Your post has been deleted.')
        return redirect(url_for('index'))

为了调用这个函数，我们将会在模板中添加这个删除链接(文件 *app/templates/post.html*)::

    {% if post.author.id == g.user.id %}
    <div><a href="{{ url_for('delete', id = post.id) }}">{{ _('Delete') }}</a></div>
    {% endif %}

现在我们接着继续，在生产模式下运行我们的应用程序。Linux 和 Mac 用户可以这么做::

    $ ./runp.py

Windows 用户这么做::

    flask/Scripts/python runp.py

现在作为一个用户，撰写一篇 blog，接着改变主意删除它。当你点击删除链接的时候，错误出现了！

我们得到了一个简短的消息说，应用程序已经遇到了一个错误，并已通知管理员。其实这个消息就是我们的 *500.html* 模版。在生产模式下，当在处理请求的时候出现一个异常错误，Flask 会返回一个 500 错误模版给客户端。因为我们是处于生产模式下，我们看不到错误信息或者堆栈轨迹。


现场调试问题
--------------

回想 :ref:`testing` 这一章，我们开启了一些调试服务在应用程序中的生产模式版本。当时我们创造了一个写入到日志文件的日志记录器，以便应用程序可以在运行时写入调试或诊断信息。Flask 自身会在结束一个错误 500 代码请求之前写入不能处理的任何异常的堆栈轨迹。作为一个额外的选项，我们也开启了基于日志的邮件通知服务，它将会向管理员列表发送日志信息邮件当一个错误写入日志信息的时候。

因此像上面的 bug，我们能够从两个地方，日志文件和发送的邮件，获得捕获的调试信息。

一个堆栈轨迹并不足够，但是总比没有好吧。假设我们对问题一点都不知道，我们需要单从堆栈轨迹中之处发生些什么。这是这个特别的堆栈轨迹的副本::

    127.0.0.1 - - [03/Mar/2013 23:57:39] "GET /delete/12 HTTP/1.1" 500 -
    Traceback (most recent call last):
      File "/home/microblog/flask/lib/python2.7/site-packages/flask/app.py", line 1701, in __call__
        return self.wsgi_app(environ, start_response)
      File "/home/microblog/flask/lib/python2.7/site-packages/flask/app.py", line 1689, in wsgi_app
        response = self.make_response(self.handle_exception(e))
      File "/home/microblog/flask/lib/python2.7/site-packages/flask/app.py", line 1687, in wsgi_app
        response = self.full_dispatch_request()
      File "/home/microblog/flask/lib/python2.7/site-packages/flask/app.py", line 1360, in full_dispatch_request
        rv = self.handle_user_exception(e)
      File "/home/microblog/flask/lib/python2.7/site-packages/flask/app.py", line 1358, in full_dispatch_request
        rv = self.dispatch_request()
      File "/home/microblog/flask/lib/python2.7/site-packages/flask/app.py", line 1344, in dispatch_request
        return self.view_functions[rule.endpoint](**req.view_args)
      File "/home/microblog/flask/lib/python2.7/site-packages/flask_login.py", line 496, in decorated_view
        return fn(*args, **kwargs)
      File "/home/microblog/app/views.py", line 195, in delete
        db.session.delete(post)
      File "/home/microblog/flask/lib/python2.7/site-packages/sqlalchemy/orm/scoping.py", line 114, in do
        return getattr(self.registry(), name)(*args, **kwargs)
      File "/home/microblog/flask/lib/python2.7/site-packages/sqlalchemy/orm/session.py", line 1400, in delete
        self._attach(state)
      File "/home/microblog/flask/lib/python2.7/site-packages/sqlalchemy/orm/session.py", line 1656, in _attach
        state.session_id, self.hash_key))
    InvalidRequestError: Object '<Post at 0xff35e7ac>' is already attached to session '1' (this is '3')

如果你习惯读别的语言的堆栈轨迹，请注意 Python 的堆栈轨迹是反序的，最下面的是引起异常的发生点。

从上面的堆栈轨迹中我们可以看到异常是被 SQLAlchemy 会话处理代码触发的。在堆栈轨迹中，对找出我们代码最后的执行语句也是十分有帮助的。我们会最后定位到我们的 *app/views.py* 文件中的 *delete()* 函数中的 *db.session.delete(post)* 语句。

我们从上面的情况中可以知道 SQLAlchemy 是不能在数据库会话中注册删除操作。但是我们不知道为什么。

如果你仔细看看堆栈轨迹的最下面，你会发现问题好像在 *Post* 对象最开始绑定在会话 *'1'*，现在我们试着绑定同一对象到会话 *'3'* 中。

如果你搜索这个问题的话，你会发现很多用户都会遇到这个问题，尤其是使用一个多线程网页服务器，它们得到两个请求尝试同时把同一对象添加到不同的会话。但是我们使用的是 Python 开发网页服务器，它是一个单线程服务器，因此这不是我们问题的原因。这可能的原因是不知道什么操作导致两个会话在同一时间活跃。

如果我们想要了解更多的问题，我们应该尝试以一种更可控的环境下重现错误。幸运的是，在开发模式下的应用程序试图重现这个问题，它确实重现了。在开发模式中，当发生异常时，我们得到是 Flask web 的堆栈轨迹，而不是 *500.html* 模板。

web 的堆栈轨迹是十分好的，因为它允许你检查代码并且从服务器上计算表达式。没有真正理解在这段代码中到底为什么会发生这种事情，我们只是知道某些原因导致一个请求在结束的时候没有正常删除会话。因此更好的方案就是找出到底是谁创建这个会话。


使用 Python 调试器
--------------------

最容易找出谁创建一个对象的方式就是在对象构建中设置断点。断点是一个当满足一定条件的时候中断程序的命令。此时此刻，这是能够检查程序，比如获取中断时的堆栈轨迹，检查或者甚至改变变量值，等等。断点是 *调试器* 的特色之一。这次我们将会使用 Python 内置模块，叫做 *pdb*。

但是我们该检查哪个类？让我们回到基于 Web 的堆栈轨迹，再仔细找找。在最底层的堆栈帧中，我们能使用代码浏览器和 Python 控制台来找出使用会话的类。在代码中，我们看到我们是在 *Session* 类中。这像是 SQLAlchemy 中的数据库会话的基础类。因为现在在最底层的堆栈帧正是在会话对象里，我们能够在控制台中得到会话实际的类，通过运行::

  >>> print self
  <flask_sqlalchemy._SignallingSession object at 0xff34914c>

现在我们知道使用中的会话是通过 Flask-SQLAlchemy 定义的，因此这个扩展可能定义自己的会话类，作为 SQLAlchemy 会话的一个子类。

现在我们可以到 Flask-SQLAlchemy 扩展的 *flask/lib/python2.7/site-packages/flask_sqlalchemy.py* 中检查源代码并且定位到类 *_SignallingSession* 和它的 *__init__()* 构造函数，现在我们准备用调试器工作。

有很多方式在 Python 应用程序中设置断点。最简单的一种就是在我们想要中断的程序中写入如下代码::

  import pdb; pdb.set_trace()

因此我们继续向前并且暂时在 *_SignallingSession* 类的构造函数插入断点(文件 *flask/lib/python2.7/site-packages/flask_sqlalchemy.py*)::

  class _SignallingSession(Session):

      def __init__(self, db, autocommit=False, autoflush=False, **options):
          import pdb; pdb.set_trace() # <-- this is temporary!
          self.app = db.get_app()
          self._model_changes = {}
          Session.__init__(self, autocommit=autocommit, autoflush=autoflush,
                           extension=db.session_extensions,
                           bind=db.engine,
                           binds=db.get_binds(self.app), **options)

      # ...

让我们继续运行看看会发生什么::

  $ ./run.py
  > /home/microblog/flask/lib/python2.7/site-packages/flask_sqlalchemy.py(198)__init__()
  -> self.app = db.get_app()
  (Pdb)

因为没有打印出 “Running on ...” 的信息我们知道服务器实际上并没有完成启动过程。中断可能已经发生了在内部某些神秘的代码里面。

最重要的问题是我们需要回答应用程序现在处于哪里，因为这将会告诉我们谁在请求创建会话 *'1'*。我们将会使用 *bt* 来获取堆栈轨迹::

  (Pdb) bt
    /home/microblog/run.py(2)<module>()
  -> from app import app
    /home/microblog/app/__init__.py(44)<module>()
  -> from app import views, models
    /home/microblog/app/views.py(6)<module>()
  -> from forms import LoginForm, EditForm, PostForm, SearchForm
    /home/microblog/app/forms.py(4)<module>()
  -> from app.models import User
    /home/microblog/app/models.py(92)<module>()
  -> whooshalchemy.whoosh_index(app, Post)
    /home/microblog/flask/lib/python2.6/site-packages/flask_whooshalchemy.py(168)whoosh_index()
  -> _create_index(app, model))
    /home/microblog/flask/lib/python2.6/site-packages/flask_whooshalchemy.py(199)_create_index()
  -> model.query = _QueryProxy(model.query, primary_key,
    /home/microblog/flask/lib/python2.6/site-packages/flask_sqlalchemy.py(397)__get__()
  -> return type.query_class(mapper, session=self.sa.session())
    /home/microblog/flask/lib/python2.6/site-packages/sqlalchemy/orm/scoping.py(54)__call__()
  -> return self.registry()
    /home/microblog/flask/lib/python2.6/site-packages/sqlalchemy/util/_collections.py(852)__call__()
  -> return self.registry.setdefault(key, self.createfunc())
  > /home/microblog/flask/lib/python2.6/site-packages/flask_sqlalchemy.py(198)__init__()
  -> self.app = db.get_app()
  (Pdb)

像之前做的，我们会发现在 *models.py* 的 92 行中存在问题，那里是我们全文搜索引擎初始化的地方::

  whooshalchemy.whoosh_index(app, Post)

奇怪，在这个阶段我们并没有做触发数据库会话创建的事情，这看起来好像是初始化 Flask-WhooshAlchemy 的行为，它创建了一个数据库会话。

这感觉就像这毕竟不是我们的错误，也许某种形式的交互在两个 Flask 扩展 SQLAlchemy 和 Whoosh 之间。我们能停留在这里并且寻求两个扩展的开发者的帮助。或者是我们继续调试看能不能找出问题的真正所在。我将会继续，如果大家不感兴趣的话，可以跳过下面的内容。

让我们多看这个堆栈轨迹一眼。我们调用了 *whoosh_index()*，它反过来调用了 *_create_index()*。在 *_create_index()* 中的一行代码是这样的::

  model.query = _QueryProxy(model.query, primary_key,
              searcher, model)

在这里的 *model* 的变量被设置成我们的 *Post* 类，我们在调用 *whoosh_index()* 的时候传入的 *Post* 类。考虑到这一点，这看起来像是 Flask-WhooshAlchemy 创建了一个 *Post.query* 封装，它把原始的 *Post.query* 作为参数，并且附加些其它的 Whoosh 特别的东西。接着是最让人感兴趣的一部分。根据上面的堆栈轨迹，下一个调用的函数是 *__get__()*，这是一个 Python 的 `描述符 <http://docs.python.org/2/howto/descriptor.html>`_。

*__get__()* 方法是用于实现描述符，它是一个与它们行为关联的属性而不只是一个值。每次被引用,描述符 *__get__()* 的函数被调用。函数被支持返回属性的值。在这行代码中唯一被提及的的属性就是 *query*，





