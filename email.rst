.. _email:


邮件支持
===========


回顾
--------

在近来的几篇教程中，我们一直在与数据库打交道。

今天我们打算让数据库休息下，相反我们今天准备完成网页应用程序中一项重要的功能:能够给用户发送邮件。

在我们小型 *microblog* 应用程序，我们将要实现一个与邮件有关的功能，我们将会给用户发送一封邮件当他或者她被人关注的时候。实现邮件有很多方式，因此我们需要设计一个通用的框架，以便重用。


安装 Flask-Mail
-----------------

幸运地，Flask 已经存在处理邮件的扩展，尽管不是 100% 支持我们想要的功能，但是已经很好了。

在我们的虚拟环境上安装 Flask-Mail 是相当简单的。在 Windows 以外的其它系统上的用户可以按照如下命令::

    flask/bin/pip install flask-mail

Windows 上的用户稍微有些不同，因为 Flask-Mail 中使用的一个模块不支持该系统，用户需要按照如下命令::

    flask\Scripts\pip install --no-deps lamson chardet flask-mail


配置
-------

当我们在 :ref:`testing` 中，已经为了能够发送错误信息。配置了邮件相关的信息，实际上它也能用于发送用于相关的邮件。

作为提醒，我必须重复下配置中需要的信息:

* 用于发送邮件的邮件服务器
* 管理员的邮箱地址

下面就是 :ref:`testing` 中的配置(文件 *config.py*)::

    # email server
    MAIL_SERVER = 'your.mailserver.com'
    MAIL_PORT = 25
    MAIL_USE_TLS = False
    MAIL_USE_SSL = False
    MAIL_USERNAME = 'you'
    MAIL_PASSWORD = 'your-password'

    # administrator list
    ADMINS = ['you@example.com']

