.. _pagination:


分页
========


回顾
------

在前面的章节(:ref:`followers`)，我们已经完成了所有支持 “关注者” 功能的数据库的修改。今天我们将会让我们应用程序接受用户的真实数据。我们将要告别伪造数据的时候！

我们接下来讲述的正是我们上一章离开的地方，所以你可能要确保应用程序 *microblog* 正确地安装和工作。


提交博客文章
---------------

让我们先以简单的内容开始，主页应该有一个提交新的 blog 的表单。

首先我们定义一个单字段的表单对象(文件 *app/forms.py*)::

    class PostForm(Form):
        post = TextField('post', validators = [Required()])

接着，我们把表单添加到模板中(文件 *app/templates/index.html*)::

    <!-- extend base layout -->
    {% extends "base.html" %}

    {% block content %}
    <h1>Hi, {{g.user.nickname}}!</h1>
    <form action="" method="post" name="post">
        {{form.hidden_tag()}}
        <table>
            <tr>
                <td>Say something:</td>
                <td>{{ form.post(size = 30, maxlength = 140) }}</td>
                <td>
                {% for error in form.errors.post %}
                <span style="color: red;">[{{error}}]</span><br>
                {% endfor %}
                </td>
            </tr>
            <tr>
                <td></td>
                <td><input type="submit" value="Post!"></td>
                <td></td>
            </tr>
        </table>
    </form>
    {% for post in posts %}
    <p>
      {{post.author.nickname}} says: <b>{{post.body}}</b>
    </p>
    {% endfor %}
    {% endblock %}

到目前为止，内容没有什么解释的，都不是新的东西。我们只是简单的添加另外一个表单而已，跟我们之前做的一样。

最后，把这一切联系起来的视图函数需要被扩展用来处理表单(文件 *app/views.py*)::

    from forms import LoginForm, EditForm, PostForm
    from models import User, ROLE_USER, ROLE_ADMIN, Post

    @app.route('/', methods = ['GET', 'POST'])
    @app.route('/index', methods = ['GET', 'POST'])
    @login_required
    def index():
        form = PostForm()
        if form.validate_on_submit():
            post = Post(body = form.post.data, timestamp = datetime.utcnow(), author = g.user)
            db.session.add(post)
            db.session.commit()
            flash('Your post is now live!')
            return redirect(url_for('index'))
        posts = [
            { 
                'author': { 'nickname': 'John' }, 
                'body': 'Beautiful day in Portland!' 
            },
            { 
                'author': { 'nickname': 'Susan' }, 
                'body': 'The Avengers movie was so cool!' 
            }
        ]
        return render_template('index.html',
            title = 'Home',
            form = form,
            posts = posts)

让我们一个一个来回顾下这个函数的修改点:

* 导入 *Post* 和 *PostForm* 类
* 我们接受 POST 请求在与 *index* 视图函数相关联的两个路由上，因为我们需要接受提交的 blog。
* 当接受常规的 GET 请求的时候我们像以前一样的处理。当我们接收到一个表单的提交的时候，我们在数据库中插入一个新的 *Post* 记录。
* 模板现在接受一个附件的参数：*form*。

当我们在数据库中插入一个新的 *Post* 后，我们将会重定向到首页::

    return redirect(url_for('index')) 

我们这里可以轻松地跳过(不使用)重定向，允许函数继续到渲染模板的部分，这将会是更加高效的。因此，为什么需要重定向？考虑如果一个用户正在撰写 blog，接着不小心按到了浏览器的刷新键，会发生些什么。刷新的命令将会做些什么？浏览器将会重新发送上一次的请求作为刷新命令的结果。

没有重定向，上一次的请求是 提交表单的 POST 请求，因此刷新动作将会重新提交表单，导致与第一个相同的第二个 *Post* 记录被写入数据库。这并不好。

有了重定向，我们迫使浏览器在表单提交后发送另外一个请求，即重定向页的请求。这是一个简单的 GET 请求，因此一个刷新动作将会重复 GET 请求而不是多次提交表单。

这个小技巧避免了用户在提交 blog 后不小心触发刷新的动作而导致插入重复的 blog。


显示 blog
----------

我们将要从数据库获取 blog，并展示它们。

如果你还记得前几篇文章中，我们创建了几个伪造的 blog，它们已经在我们主页上展示很长一段时间。在 *index* 视图函数这两个创建的伪造的对象是简单的 Python 列表::

    posts = [
        { 
            'author': { 'nickname': 'John' }, 
            'body': 'Beautiful day in Portland!' 
        },
        { 
            'author': { 'nickname': 'Susan' }, 
            'body': 'The Avengers movie was so cool!' 
        }
    ]

但是在上一章中我们已经创建了一个查询，它允许我们获取关注的用户的所有的 blog，因此我们简单地替换上面这些伪造的数据(文件 *app/views.py*)::

    posts = g.user.followed_posts().all()

当你运行应用程序的时候就会看到来自数据库中的 blog。

*User* 类中的 *followed_posts* 方法