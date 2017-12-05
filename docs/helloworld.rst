.. _helloworld:

Hello World
=============


作者背景
---------

作者是一个使用多种语言开发复杂程序并且拥有十多年经验的软件工程师。作者第一次学习 Python 是在为一个 C++ 库创建绑定的时候。

除了 Python，作者曾经用 PHP, Ruby, Smalltalk 甚至 C++ 写过 web 应用。在所有这些中，Python/Flask 组合是作者认为最为自由的一种。

应用程序简介
-------------

作为本教程的一部分--我要开发的应用程序是一个极具特色的微博服务器，我称之为 *microblog* 。

我会随着应用程序的不断地进展将涉及到如下这些主题：

* 用户管理，包括管理登录，会话，用户角色，权限以及用户头像。
* 数据库管理，包括迁移处理。
* Web 表单支持，包括对各个字段的验证。
* 分页处理。
* 全文搜索。
* 用户邮件提醒。
* HTML 模板。
* 支持多国语言。
* 缓存以及其它性能优化技术。
* 开发以及生产服务器的调试技巧。
* 在生产服务器上安装。

我希望这个应用程序将能够成为编写其它类型的 web 应用程序的一个样板，当它完成的时候。

要求
-----

如果你有一台能够运行 Python 的机器，可能你将会很轻松。该教程中的应用程序能够完美地运行在 Windows, OS X 以及 Linux 上。除非另有说明，本系列的文章中提供的代码已经在 Python 2.7 和 3.4 上测试过。

本教程假定你很熟悉操作系统的终端窗口(命令提示符为 Windows 用户)，清楚基本命令行文件管理功能。如果你还不熟悉这些的话，我强烈建议你先学习使用命令行，比如如何创建文件夹等，接着再继续。

最后，你应该还能够很舒服地(熟练地)编写 Python 代码。强烈推荐熟悉 Python 的 `Python 模块和包 <http://docs.python.org/tutorial/modules.html>`_ 。

安装 Flask
------------

好的，让我们开始吧！

现在我们必须开始安装 Flask 以及一些我们会用到的扩展。我首选的方式就是创建一个 `虚拟环境 <http://pypi.python.org/pypi/virtualenv>`_ ,这个环境能够安装所有的东西，而你的主 Python 不会受到影响。另外一个好处就是这种方式不需要你拥有 root 权限。

因此，打开一个终端窗口，选择一个你想要放置应用程序的位置以及创建一个包含它的新的文件夹。让我们把这个应用程序的文件夹称为 **microblog** 。

如果你正在使用 Python 3.4，先进入到 **microblog** 目录中接着使用如下的命令创建一个虚拟环境::
	
	$ python -m venv flask

需要注意地是在某些系统中你可能要使用 python3 来代替 python。上面的命令行在 *flask* 文件夹中创建一个完整的 Python 环境。

如果你使用 Python 3.4 以下的版本(包括 python 2.7)，你需要在创建虚拟环境之前下载以及安装 `virtualenv.py <https://pypi.python.org/pypi/virtualenv/>`_ 。如果你在使用 Mac OS X，请使用下面的命令行安装::
	
	$ sudo easy_install virtualenv

如果你使用 Linux，你需要获取一个包。例如，如果你使用 Ubuntu::
	
	$ sudo apt-get install python-virtualenv 

Windows 用户们在安装 virtualenv 上有些麻烦，因此如果你想省事的话，请直接安装 Python 3.4。在 Windows 上安装 virtualenv 最简单地方式就是先安装 pip，安装方式在 `这里 <https://pip.pypa.io/en/latest/installing.html>`。一旦安装好了 pip 的话，下面的命令可以安装 virtualenv::

	$ pip install virtualenv

为了创建一个虚拟环境，请输入如下的命令行 ::

	$ virtualenv flask

上面的命令行在 *flask* 文件夹中创建一个完整的 Python 环境。

虚拟环境是能够激活以及停用的，如果需要的话，一个激活的环境可以把它的 *bin* 文件夹加入到系统路径。我个人是不喜欢这种特色，所以我从来不激活任何环境相反我会直接输入我想要调用的解释器的路径。

如果你是在 Linux, OS X 或者 Cygwin 上，通过一个接一个输入如下的命令行来安装 flask 以及扩展::

	$ flask/bin/pip install flask
	$ flask/bin/pip install flask-login
	$ flask/bin/pip install flask-openid
	$ flask/bin/pip install flask-mail
	$ flask/bin/pip install flask-sqlalchemy
	$ flask/bin/pip install sqlalchemy-migrate
	$ flask/bin/pip install flask-whooshalchemy
	$ flask/bin/pip install flask-wtf
	$ flask/bin/pip install flask-babel
	$ flask/bin/pip install guess_language
	$ flask/bin/pip install flipflop
	$ flask/bin/pip install coverage

如果是在 Windows 上的话，命令行有些不同 ::

	$ flask\Scripts\pip install flask
	$ flask\Scripts\pip install flask-login
	$ flask\Scripts\pip install flask-openid
	$ flask\Scripts\pip install flask-mail
	$ flask\Scripts\pip install flask-sqlalchemy
	$ flask\Scripts\pip install sqlalchemy-migrate
	$ flask\Scripts\pip install flask-whooshalchemy
	$ flask\Scripts\pip install flask-wtf
	$ flask\Scripts\pip install flask-babel
	$ flask\Scripts\pip install guess_language
	$ flask\Scripts\pip install flipflop
	$ flask\Scripts\pip install coverage

这些命令行将会下载以及安装我们将会在我们的应用程序中使用的所有的包。


在 Flask 中的 "Hello, World"
------------------------------

现在在你的 *microblog* 文件夹中下有一个 *flask* 子文件夹，这里有 Python 解释器以及 Flask 框架以及我们将要在这个应用程序中使用的扩展。 是时候去编写我们第一个 web 应用程序！

在 *cd* 到 *microblog* 文件夹后，我们开始为应用程序创建基本的文件结构::
	
	mkdir app
	mkdir app/static
	mkdir app/templates
	mkdir tmp


我们的应用程序包是放置于 *app* 文件夹中。子文件夹 *static* 是我们存放静态文件像图片，JS文件以及样式文件。子文件夹 *templates* 显然是存放模板文件。

让我们开始为我们的 *app* 包(文件 *app/__init__.py* )创建一个简单的初始化脚本::

	from flask import Flask

	app = Flask(__name__)
	from app import views

上面的脚本简单地创建应用对象，接着导入视图模块，该模块我们暂未编写。

视图是响应来自网页浏览器的请求的处理器。在 Flask 中，视图是编写成 Python 函数。每一个视图函数是映射到一个或多个请求的 URL。

让我们编写第一个视图函数(文件 *app/views.py* )::

	from app import app

	@app.route('/')
	@app.route('/index')
	def index():
	    return "Hello, World!"

其实这个视图是非常简单，它只是返回一个字符串，在客户端的网页浏览器上显示。两个 *route* 装饰器创建了从网址 */* 以及 */index* 到这个函数的映射。

能够完整工作的 Web 应用程序的最后一步是创建一个脚本，启动我们的应用程序的开发 Web 服务器。让我们称这个脚本为 *run.py*，并把它置于根目录::

	#!flask/bin/python
	from app import app
	app.run(debug = True)

这个脚本简单地从我们的 app 包中导入 *app* 变量并且调用它的 *run* 方法来启动服务器。请记住 *app* 变量中含有我们在之前创建的 *Flask* 实例。

要启动应用程序，您只需运行此脚本（*run.py*）。在OS X，Linux 和 Cygwin 上，你必须明确这是一个可执行文件，然后你可以运行它::

	chmod a+x run.py

然后脚本可以简单地按如下方式执行::

	./run.py

在 Windows 上过程可能有些不同。不再需要指明文件是否可执行。相反你必须运行该脚本作为 Python 解释器的一个参数::

	flask/Scripts/python run.py

在服务器初始化后，它将会监听 5000 端口等待着连接。现在打开你的网页浏览器输入如下 URL::

	http://localhost:5000

另外你也可以使用这个 URL::

	http://localhost:5000/index

你看清楚了路由映射是如何工作的吗？第一个 URL 映射到 */*，而第二个 URL 映射到 */index*。这两个路由都关联到我们的视图函数，因此它们的作用是一样的。如果你输入其它的网址，你将会获得一个错误，因为只有这两个 URL 映射到视图函数。

你可以通过 *Ctrl-C* 来终止服务器。到这里，我将会结束这一章的内容。对于不想输入代码的用户，你可以到这里下载代码：`microblog-0.1.zip <https://github.com/miguelgrinberg/microblog/archive/v0.1.zip>`_。


下一步？
--------

下一章我们将会小小修改下我们的应用，使用 HTML 模板。我希望在下一章再见到大家！




