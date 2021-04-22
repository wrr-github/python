# 一、安装

```shell
pip install Flask
```

# 二、应用

## 1、快速上手

```python
from flask import Flask
app = Flask(__name__)  # 第一个参数是应用模块或者包的名称

@app.route('/')  # route()装饰器：将url绑定到函数
def hello_world():  # 函数名称被用于生成相关联的URL
    return 'Hello, World!'  # 函数最后返回需要在用户浏览器中显示的信息
  
if __name__ == '__main__':
   app.run(host, port, debug, options)
'''    
host：要监听的主机名，默认为127.0.0.1（localhost）
port：监听的端口，默认值为5000
debug：默认为false，如果设置为true，则提供调试信息，如果代码更改，服务器将自行重新加载
options：要转发到底层的Werkzeug服务器
'''
```

## 2、运行

```shell
export FLASK_APP=hello.py
python -m flask run
```

## 3、在url上添加变量

```python
from markupsafe import escape

@app.route('/user/<username>')  # 通过把URL的一部分标记为<variable_name>就可以在URL中添加变量
def show_user_profile(username):
    return 'User %s' % escape(username)

@app.route('/post/<int:post_id>')  # 通过使用<converter:variable_name>，可以选择性的加上一个转换器，为变量指定规则
def show_post(post_id):
    return 'Post %d' % post_id
```

转换器类型：

| 类型   | 规则                                |
| ------ | ----------------------------------- |
| string | （缺省值） 接受任何不包含斜杠的文本 |
| int    | 接受正整数                          |
| float  | 接受正浮点数                        |
| path   | 类似 `string` ，但可以包含斜杠      |
| uuid   | 接受 UUID 字符串                    |

## 4、url尾部/

```python
@app.route('/projects/')  # 访问时用/projects/和/projects会返回相同的输出
def projects():
    return 'The project page'

@app.route('/about')  # 没有尾部斜杠，访问时尾部若带有斜杠则会404(如：/about/)。可以保持URL唯一
def about():
    return 'The about page'
```

## 5、重定向、动态构建、错误

```python
from flask import Flask, redirect, url_for
app = Flask(__name__)
@app.route('/admin')
def hello_admin():
   return 'Hello Admin'

@app.route('/guest/<guest>')
def hello_guest(guest):
   return 'Hello %s as Guest' % guest

@app.route('/user/<name>')
def hello_user(name):
   if name =='admin':
      return redirect(url_for('hello_admin'))  # redirect()：重定向
   else:
      return redirect(url_for('hello_guest', guest = name))  # url_for(args)：构建指定函数名args的URL
    	abort(401)  # abort()：更早退出请求，并返回错误代码
    	this_is_never_executed()

@app.errorhandler(404)  # errorhandler()装饰器：定制出错页面
def page_not_found(error):
    return render_template('page_not_found.html'), 404
```

## 6、http方法

| 方法名 | 备注                                                         |
| ------ | ------------------------------------------------------------ |
| GET    | 以未加密的形式将数据发送到服务器，默认                       |
| HEAD   | 和GET方法相同，但没有响应体                                  |
| POST   | 用于将HTML表单数据发送到服务器，POST方法接收的数据不由服务器缓存 |
| PUT    | 用上传的内容替换目标资源的所有当前表示                       |
| DELETE | 删除由URL给出的目标资源的所有当前表示                        |

## 6、全局对象request

| 属性    | 备注                                      |
| ------- | ----------------------------------------- |
| method  | 获取请求方法                              |
| form    | 获取form表单数据，若无此参数则容易返回400 |
| args    | 获取url上的参数                           |
| files   | 获取上传的文件                            |
| cookies | 获取cookies                               |

```python
from flask import request
from werkzeug.utils import secure_filename
from flask import make_response

@app.route('/login', methods=['GET', 'POST'])  # route()装饰器的methods参数：处理不同的HTTP方法
def login():
    if request.method == 'POST':  # method属性：获取请求方法
      
      	username = request.form['username']  # form属性：获取form表单数据，若无此参数则容易返回400
        password = request.form['password']
        
        searchword = request.args.get('key', '')  # args属性：获取url上的参数
      
      	f = request.files['the_file']  # files属性：获取上传的文件
        f.save('/var/www/uploads/uploaded_file.txt')  # save()：把上传文件保存
        f.save('/var/www/uploads/' + secure_filename(f.filename))  # secure_filename()：获取上传的文件名
        
        cookies = request.cookies.get('username')  # cookies属性：获取cookies
        resp = make_response(render_template(...))  # make_response()：显示转换cookie
    		resp.set_cookie('username', 'the username')
       
        return do_the_login()
    else:
       	return show_the_login_form()

with app.test_request_context('/hello', method='POST'):  # test_request_context()：为环境管理器
    assert request.path == '/hello'  # path属性：获取请求url路径
    assert request.method == 'POST'

with app.request_context(environ):  # 把整个 WSGI 环境传递给 request_context() 方法
    assert request.method == 'POST'
```

## 7、渲染模版

```python
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)  # render_template()：渲染模板，Flask会在templates文件夹内寻找模板
```

## 9、响应

1. 如果视图返回的是一个响应对象，那么就直接返回它。
2. 如果返回的是一个字符串，那么根据这个字符串和缺省参数生成一个用于返回的 响应对象。
3. 如果返回的是一个字典，那么默认调用 `jsonify` 创建一个响应对象。
4. 如果返回的是一个元组，那么元组中的项目可以提供额外的信息。元组中必须至少包含一个项目，且项目应当由 `(response, status)` 、 `(response, headers)` 或者 `(response, status, headers)` 组成。 `status` 的值会重载状态代码， `headers` 是一个由额外头部值组成的列表 或字典。
5. 如果以上都不是，那么 Flask 会假定返回值是一个有效的 WSGI 应用并把它转换为 一个响应对象。

```python
@app.errorhandler(404)
def not_found(error):
    resp = make_response(render_template('error.html'), 404)  # make_response()：生成响应对象
    resp.headers['X-Something'] = 'A value'  # 修改响应对象中的header
    return resp
```

```python
@app.route("/me")
def me_api():
    user = get_current_user()
    return {  # 返回字典，默认会转成json
        "username": user.username,
        "theme": user.theme,
        "image": url_for("user_image", filename=user.image),
    }
  
@app.route("/users")
def users_api():
    users = get_all_users()
    return jsonify([user.to_json() for user in users])  #jsonfy()：序列化json数据类型
```

## 10、session

```python
from flask import Flask, session, redirect, url_for, request
from markupsafe import escape

app = Flask(__name__)

# Set the secret key to some random bytes. Keep this really secret!
app.secret_key = b'_5#y2L"F4Q8z\n\xec]/'

@app.route('/')
def index():
    if 'username' in session:
        return 'Logged in as %s' % escape(session['username'])
    return 'You are not logged in'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''

@app.route('/logout')
def logout():
    # remove the username from the session if it's there
    session.pop('username', None)
    return redirect(url_for('index'))
```

## 11、日志

```python
app.logger.debug('A value for debugging')
app.logger.warning('A warning occurred (%d apples)', 42)
app.logger.error('An error occurred')
```

