.. _userlogin:

用户登录
==========

这是本系列的第五章，该章主要讲述用户登录的实现，把表单和数据库的内容结合起来，在本章结束后我们就可以看到一个拥有注册新用户、用户登录以及用户注销的小应用程序。

回顾
--------

在上一章中，我们创建了数据库，并且学会了如何使用数据库来存储用户以及文章数据，但是并没有把它结合到我们的应用程序中。

我们接下来讲述的正是我们上一章离开的地方，所以你可能要确保应用程序 *microblog* 正确地安装和工作。


配置
-------

我们首先开始配置我们使用的 Flask 扩展。对于登录系统，我们将会使用两个扩展， Flask-Login 以及 Flask-OpenID。配置如下(文件 *app\__init__.py*)::

	import os
	from flask.ext.login import LoginManager
	from flask.ext.openid import OpenID
	from config import basedir

	lm = LoginManager()
	lm.init_app(app)
	oid = OpenID(app, os.path.join(basedir, 'tmp'))

Flask-OpenID 扩展需要一个路径用来存储临时文件夹。对此我们提供了一个本地的 *tmp* 文件夹。


重新审视用户模型
---------------

Flask-Login 扩展希望我们的 *User* 类实现一些方法，但是没有要求类如何实现方法内的代码。

下面是我们针对 Flask-Login 完成的 *User* 类(文件 *app/models.py*)::

	class User(db.Model):
	    id = db.Column(db.Integer, primary_key = True)
	    nickname = db.Column(db.String(64), unique = True)
	    email = db.Column(db.String(120), unique = True)
	    role = db.Column(db.SmallInteger, default = ROLE_USER)
	    posts = db.relationship('Post', backref = 'author', lazy = 'dynamic')

	    def is_authenticated(self):
	        return True

	    def is_active(self):
	        return True

	    def is_anonymous(self):
	        return False

	    def get_id(self):
	        return unicode(self.id)

	    def __repr__(self):
	        return '<User %r>' % (self.nickname)

