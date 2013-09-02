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
*