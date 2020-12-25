#  Flaskで作る画像判定

## venv環境作成

バーチャル環境にvenvを使います。

### 今回作成アプリの仮想環境設定

flask_vgg16という環境を作るには、以下を実行

```
python -m venv  flask_vgg16
```



環境に入るのは以下`cd flask_vgg16g`で移動した後に次のコマンドを実行

ここはよく失敗するので注意。

作成した環境名のフォルダ内にbinフォルダが作成されます。binフォルダ内にactivateファイルがあるのでそれを実行すれば良いのです。

```python
source bin/activate
```

現在の仮想環境から出る。

```python
deactivate
```

## アプリケーションの環境設定

Flaskの導入

```
pip install Flask
```

### Flaskバージョン確認

Flaskのバージョン確認は次のようにPythonから確認します。

```
$ python
>> import flask
>> flask.__version__
```

'1.1.2'のようにバージョンが確認できます。

今回のFlaskバージョンは 1.1.1 での例です。

### tensorflowインストール

```
pip install tensorflow==2.0.0
```

### werkzeug

```
pip install Jinja2 redis Werkzeug
```

### gunicorn のインストール

```
pip install gunicorn
```

### psycopg2 のインストール

```
pip install psycopg2-binary
```

以下numpyまでは導入すみ

### jinja2のインストール

```
pip install jinja2
```

### h5pyのインストール

```
pip install h5py
```

### numpyのインストール

```
pip install numpy
```

### pillowインストール

```
pip install pillow
```

###matplotlibインストール
```
pip install matplotlib
```



### Flaskドキュメント

[Flask1.1系のドキュメントはこちら](https://flask.palletsprojects.com/en/1.1.x/)



## アプリケーション本体の作成

### FlaskでHello world表示

Flaskアプリケーションを作成する手始めは、Flaskをimportすることから始まります。

```
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
	return 'Hello world!'

if __name__ == "__main__":
    app.run(debug=True)
```

Flaskクラスをインスタンス化しています。パラメータには本来アプリケーション名を指定します。

一般的に`__name__`で属性を指定することが多いです。

`__name__`は Python の特殊なグローバル変数の1つで、Python のプログラムがコマンドラインから直接行われた場合、この値は`__main__`となります。また、importで他のプログラムから呼ばれた場合、ファイル名が格納されます。

`@app.route('/')`はデコレータです。URLがドメインの後`'/'`で終わっているだけのものなら、そのすぐ下に記述された関数を実行する役目です。

Flaskのルータはデコレータを活用しているのですが、デコレータの実例として良い例です。

hello関数は特に説明の必要はないと思いますが、'Hello world!'を返すだけのものです。

```
if __name__ == "__main__":
    app.run(debug=True)
```

この部分は、アプリ起動の処理を書いています。

`__name__`が`__main__`と等しいということは、起動しているモジュールがメインプログラムモジュールだということ、もう少し噛み砕くと、importされただけだから関数を実行するまで実行しないでねというものです。

今回の場合はメインモジュールで実行しているので`__name__`は`__main__`となります。

作成したappインスタンスが持つrunメソッドを`debug=True`で実行するということになります。



## 画像判定アプリにFlaskを適用

Homeページはテンプレートの`index.html`判定ページは`result.html`を使います。

画像ファイルのアップロードは`index.html`で行います。

### ホームページ作成

ルーティングはデコレータで次のように作成します。

```
@app.route("/", methods=["GET"])
def index():
```

`methods=["GET"]`はgetメソッドで通信することを意味します。

getは受信するためのメソッド、postは送信するためのメソッドと捉えておいても良いかもしれません。

index.htmlのページはgetメソッドで送信されたものしか受け付けないのでmethod=["GET"]としました。

一方、resultのページはindexhtmlからpostメソッドで送ったデータを取得しますので、methodをPOSTにしています。この用途では methods=["POST"]でも良いのですが、その下のresult関数でifを使って切り分けをしています。post送信ならデータを受信して画像判定のプログラムに活用します。そうでなければ`redirect(url_for("index"))`でindexのページにリダイレクトしています。

```
@app.route("/result", methods=["GET", "POST"])
def result():
```

次にindex関数はindex,htmlファイルを表示させるものです。

```
def index():
    return render_template("index.html")
```

テンプレートの記述はJinja2というテンプレートエンジンを使っていますが、ほとんどの内容はDjangoと同じ使い方です。

## テンプレート

今回のような単純なレイアウトでは必要ないのですが、各ページ共通で活用できるテンプレートが次のlayout.htmlです。ファイル名は好きなものでよく、base.htmlをDjangoでは使用しました。

layout.html

```
  
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <title>画像分類</title>
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous">
  </head>
  <body>
    {% block header %}
    {% endblock %}
    {% block content %}
    {% endblock %}    
    <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js" integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.1/dist/umd/popper.min.js" integrity="sha384-9/reFTGAW83EW2RDu2S0VKaIzap3H66lZH81PoYlFhbGU+6BZp6G7niu735Sk7lN" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js" integrity="sha384-B4gt1jrGC7Jh4AgTPSdUtOBvfO8shuf57BaghqFfPlYxofvL8/KUEfYiJOMMV+rV" crossorigin="anonymous"></script>
  </body>
</html>
```

ベースのなるテンプレートに継ぎ足すテンプレートは先頭に`{% extends "layout.html" %}`をつけます。

ベーステンプレートの`{% block header %} {% endblock %}`内に入る内容は同じ`{% block header %} {% endblock %}`の中にコードを書きます。

index.html

```
{% extends "layout.html" %}
{% block header %}
<div class="jumbotron">
    <div class="container">
        <h1 class="display-4">画像判定アプリ</h1>
        <p class="lead">アップした画像が10種類のグループのどれに属するかを判定します。</p>
        <hr class="my-4">
    </div>
</div>
{% endblock %}
{% block content %}
<div class="container">
    <form method="post" , enctype="multipart/form-data" , action="/result">
        <div class="custom-file mb-5">
            <input type="file" name="file" class="custom-file-input" id="customFile">
            <label class="custom-file-label" for="customFile"></label>
        </div>
        <div><input type="submit", value="判　定", class="btn btn-secondary btn-lg btn-block" ></div>
    </form>
</div>
{% endblock %}
```

ホームに戻るボタンのリンクはgetメソッドで戻る仕組みを簡単なformで作成していることに注目です。

result.html

```
{% extends "layout.html" %}
{% block header %}
<div class="jumbotron">
	<div class="container">
  <h1 class="display-4">判定結果</h1>
	</div>
</div>
{% endblock %}
{% block content %}
<div class="container">
    {{result}}
    <p><img src="{{filepath}}"></p>
    <form method="get" action="/">
      <button class="submit">戻る</button>
    </form>
  </div>
{% endblock %}
```

## 画像判定のコード

画像判定のコードはresult関数に書かれています。

次のコードは post で送られたものかどうか判定し、違うならindexにリダイレクトします。

```
if request.method == "POST":
```

次の内容はファイルの存在と形式を確認して、ファイルを保存する内容です。

```
if "file" not in request.files:
    print("File doesn't exist!")
    return redirect(url_for("index"))
file = request.files["file"]
if not allowed_file(file.filename):
    print(file.filename + ": File not allowed!")
    return redirect(url_for("index"))

# ファイルの保存
if os.path.isdir(UPLOAD_FOLDER):
    shutil.rmtree(UPLOAD_FOLDER)
os.mkdir(UPLOAD_FOLDER)
filename = secure_filename(file.filename)  # ファイル名を安全なものに
filepath = os.path.join(UPLOAD_FOLDER, filename)
file.save(filepath)
```

続いて画像の読み込みをしています。

```
image = Image.open(filepath)
image = image.convert("RGB")
image = image.resize((img_size, img_size))
x = np.array(image, dtype=float)
x = x.reshape(1, img_size, img_size, 3) / 255
```

h5ファイルの読み込み

```
try:
    model = load_model("./image_class.h5")
except:
    print('Error:HDF5ファイルの読み込みに失敗しました。')
```

予測

```
y = model.predict(x)[0]
sorted_idx = np.argsort(y)[::-1]  # 降順でソート
result = ""
for i in range(n_result):
    idx = sorted_idx[i]
    ratio = y[idx]
    label = labels[idx]
    result += "<p>" + str(round(ratio*100, 1)) + \
              "%の確率で" + label + "です。</p>"
return render_template("result.html", result=Markup(result), filepath=filepath)
```



ポイントは別に作成したColaboファイルでModelを学習します。

今回はcifar-10の画像とVGG16で転移学習しています。その結果をHDF5ファイルに保存しておいて、今度はこのアプリ内で保存しておいたHDF5ファイルで予測を行うものです。

あくまで学習用のサンプルですから、内容は単純ですが、応用すると人物などの判定にも使えるものです。

## ローカル環境でのサーバー起動

guess.pyのファイルがある場所にcdで移動後以下コマンド

```
python guess.py
```

* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)と表示されますのでこのURLでブラウザを開きます。

サーバーを止めるには、表示に記述されている通り、CTRL+C です。



cd コマンドでclassifierフォルダに移動しておきます。

## アプリの実行

コマンドで以下を実行

```
python guess.py
```

\* Serving Flask app "guess" (lazy loading)

 \* Environment: production

  WARNING: This is a development server. Do not use it in a production deployment.

  Use a production WSGI server instead.

 \* Debug mode: off

 \* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

以上の表示でブラウザで確認します。

## デプロイ

### Procfile の準備

heroku 上で実行するコマンドを記載します。あるいは新規でファイルを作成して、Procfileというファイル名にします。

```
vim Procfile
```

vimで以下内容を記述　適当なエディッタで記述してもOK

```
web: gunicorn guess:app --log-file -
```

ここは、vimの使い方が苦手な場合、現在作業を行なっているフォルダ内にProcfileを作成して、その中に記述をしてもよい。

### requirements.txt の準備

`pip freeze` でインストール済パッケージ一覧を表示することが出来ます。
`heroku` は`requirements.txt`を頼りに必要なライブラリをインストールするので準備します。

```
pip freeze > requirements.txt
```

`requirements.txt`が出来上がっているのがわかります。

```
Flask==1.1.2
gunicorn==20.0.4
h5py==2.10.0
Jinja2==2.11.2
numpy==1.18.3
Pillow==7.1.2
tensorflow==2.3.0
Werkzeug==1.0.1
```

## Heroku CLIの導入

Heroku-CLIはターミナル上でHerokuアプリをより簡単に使えるツールです。

[ダウンロード](https://devcenter.heroku.com/articles/heroku-cli)こちらのページから自分のPC環境に合ったものでインストールします。

### ログイン

Herokuにログインは次のコマンドです。ログインはブラウザが立ち上がりますので、そこでログイン操作します。一度Herokuの管理画面を知っておくと良いでしょう。

```
heroku login
```

もし全てコマンドで実行したいなら` -i`オプションをつけてログインします。

```
heroku login -i
```

### Herokuのアプリ名を決めます。

アプリの作成コマンド

```
heroku create flask-vgg16
```

エラーが出たら名前がすでに使われているので、
違う名前でアプリを作成してください。

次の内容が表示されるとOK

Creating ⬢ flask-vgg16... done

https://flask-vgg16.herokuapp.com/ | https://git.heroku.com/flask-vgg16.git



## GitでHeroku コミット

まずはgitにコミットします。

**staticフォルダ内のimagesフォルダにはローカルでテストした画像を入れたままaddします。空の状態だとフォルダがgitに反映されずにHerokuでフォルダが作成されません**

git初期化コマンド（既にgit管理している場合は必要ありません）

```
git init
```

herokuのremotを教えるコマンド

```
heroku git:remote -a flask-vgg16
```

ステージにaddするコマンド

```
git add .
```

コミットするコマンド

```
git commit
```

herokuにpushするコマンド

```
git push heroku master
```

Herokuにプッシュが終わると、requirements.txtに従ってインストールが始まります。

一番のネックはTensorflowのインストールです。2.1以上のバージョンだとコケます。

原因はSlug Sizeの問題です。

無事にインストールできれば、Webアプリは以下のアドレスで開きます。

https://flask-vgg16.herokuapp.com/ 

## インストールがうまくいかない場合

Herokuで作成したアプリには、Slug Sizeというものがあります。

Slug SizeはブラウザからHerokuにログイン後、アプリ名をクリック→Settingsをクリック→Infoから判ります。

Slug Sizeが大きいと、次のような問題が起こります。

特にTensorlowの最新版では500MB制限にかかりインストール出来ません。

Tensorlow2.0.0ではインストールに成功しています。

- アプリの起動が遅くなる
- 500MB制限に引っかかる

## 判定結果でエラーが出る場合

ColaboのTensorlowのバージョンやKerasの使い方次第でうまくHDF5ファイルが読み込めないことがあります。その場合ColaboのTensorlowのバージョンをHerokuにインストールするバージョンに合わせるなどの対策が必要になります。

ColaboのTensorflowをアンインストール

```
pip uninstall -y tensorflow
```

バージョン2.0.0のインストール

```
pip install tensorflow-gpu==2.0.0
```

```
heroku plugins:install heroku-repo
```

```
heroku repo:gc --app <アプリ名>
heroku repo:purge_cache --app <アプリ名>
```

