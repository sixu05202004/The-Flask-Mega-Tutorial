.. _webforms:

web 表单
=========

Web 表单是一个在任何 Web 应用程序中最基本的部分，因此本章节准备介绍下 Flask 中的表单的处理方式。在 Flask 中，处理表单是使用了扩展 `Flask-WTF <http://packages.python.org/Flask-WTF>`_。接下来我会专门对该扩展进行讲述。


回顾
------

在上一章节中，我们定义了一个简单的模板，使用占位符来虚拟了暂未实现的部分，比如用户以及文章等。

在本章我们将要讲述应用程序的特性之一--表单，我们将会详细讨论如何使用 web 表单。

Web 表单是在任何一个 web 应用程序中最基本的一部分。我们将使用表单允许用户写文章，以及登录到应用程序中。

我们接下来讲述的正是从我们离开的地方，所以你可能要确保以上的应用程序正确地安装和工作。


配置
------

为了能够处理 web 表单，我们将使用 `Flask-WTF <http://packages.python.org/Flask-WTF>`_ ，该扩展封装了 `WTForms <http://wtforms.simplecodes.com/docs/dev>`_ 是的恰当地集成进 Flask 中。

许多 Flask 扩展需要大量的扩展，因此我们将要在 *microblog* 文件夹的根目录下创建一个配置文件以至于容易被编辑。这就是我们将要开始的(文件 *config.py*)::

	CSRF_ENABLED = True
	SECRET_KEY = 'you-will-never-guess'

十分简单吧，我们的 Flaks-WTF 扩展只需要两个配置。 *CSRF_ENABLED* 配置是为了激活 `跨站点请求伪造 <http://en.wikipedia.org/wiki/Cross-site_request_forgery>`_ 保护。在大多数情况下，你需要激活该配置使得你的应用程序更安全些。

*SECRET_KEY* 配置仅仅当 CSRF 激活的时候才需要，它是用来建立一个加密的令牌，用于验证一个表单。当你编写自己的应用程序的时候，请务必设置很难被猜测到密钥。

既然我们有了配置文件，我们需要告诉 Flask 去读取以及使用它。我们可以在 FLask 应用程序对象被创建后去做，方式如下(文件 *app/__init__.py*)::

	from flask import Flask

	app = Flask(__name__)
	app.config.from_object('config')

	from app import views


用户登录表单
------------

在 Flask-WTF 中，表单是表示成对象，*Form* 类的子类。一个表单子类简单地把表单的域定义成类的变量。

我们将要创建一个登录表单，用户用于认证系统。在我们应用程序中支持的登录机制不是标准的用户名/密码类型，我们将使用 `OpenID <http://openid.net/>`_。OpenIDs 的好处就是认证是由 OpenID 的提供者完成的，因此我们不需要验证密码，这会让我们的网站对用户而言更加安全。

OpenID 登录仅仅需要一个字符串，被称为 OpenID。我们将在表单上提供一个 ‘remember me’ 的选择框，以至于用户可以选择在他们的网页浏览器上种植 cookie ，当他们再次访问的时候，浏览器能够记住他们的登录。

所以让own编写第一个表单(文件 *app/forms.py*)::

	from flask.ext.wtf import Form, TextField, BooleanField
	from flask.ext.wtf import Required

	class LoginForm(Form):
	    openid = TextField('openid', validators = [Required()])
	    remember_me = BooleanField('remember_me', default = False)

