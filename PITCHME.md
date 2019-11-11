@snap[midpoint span-90 text-05]
# Laravelでモジュラモノリス
@snapend

@snap[south-east span-100 text-07]
2019年12月1日（日）
PHPカンファレンス2019 懇親会LT
永宮　悠大
@snapend

---

@snap[north-west]
## 自己紹介
@snapend

@snap[west span-100 text-08]
- 永宮　悠大（NAGAMIYA Yuta）
- 株式会社ホワイトプラスのエンジニア
- PHP、Laravelの面倒を見てます
- @fa[github] [@ngmy](https://github.com/ngmy)
- クライミング8：エンジニアリング2
@snapend

@snap[east span-40]
![IMAGE](assets/img/profile.jpg)
@snapend

---

@snap[north span-100 text-07]
## ホワイトプラスを知ってる人？ &#x1f64b;
@snapend

@snap[midpoint span-90]
![IMAGE](assets/img/wplogo.png)
@snapend

@snap[south span-100]
https://www.wh-plus.co.jp/
@snapend

---

@snap[north span-100 text-07]
## ゴールドスポンサーやってます
@snapend

@snap[south span-90]
![IMAGE](assets/img/sponsor.png)
@snapend

---

@snap[north span-100 text-07]
## リネットやってます
@snapend

@snap[west span-40]
![IMAGE](assets/img/lenet.png)
@snapend

@snap[east span-50 text-07]
- 宅配クリーニングのリネット
- PC・スマホから注文できて、自宅にいたままクリーニングできるWebサービス
- 衣類クリーニング
    - https://www.lenet.jp/
- 衣類クリーニング + 保管
    - https://www.lenet-hokan.jp/
- 布団クリーニング
    - https://www.futonlenet.jp/
- 靴クリーニング
    - https://www.kutsulenet.jp/</dd>
- PHP + Laravelで構築
@snapend

---？image=assets/img/toy_dorodango_kirei.png&size=100% auto

@snap[north span-100 text-07]
## モノリス
@snapend

---？image=assets/img/rubiks_cube.png&size=100% auto

@snap[north span-100 text-07]
## モジュラモノリス
@snapend

---

一つのLaravelアプリケーションをモジュールを分割し、疎結合にする

---

### 境界を強制する

---

Composer APIでオートロードするクラスを制限する

---

@snap[south-west span-50 text-07]
```text
lenet
├── lenet_common
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
├── lenet.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
├── lenet-hokan.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
├── futonlenet.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
├── kutsulenet.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
└── wh-plus.com
    ├── public
    │   └── index.php
    ├── composer.json
    ├── phpstan.neon
    └── vendor
```
@snapend

---

```php
$loader = require base_path() . '/vendor/autoload.php';
$serviceLoader = require realpath($_SERVER['DOCUMENT_ROOT'])
    . '/../vendor/autoload.php';
$loader->addClassMap($serviceLoader->getClassMap());
foreach ($serviceLoader->getPrefixesPsr4() as $prefix => $paths) {
    $loader->addPsr4($prefix, $paths);
}
```

@snap[south span-100]
@[1](Laravel + 共有ライブラリのautoload.phpを読み込む)
@[2-3](ドキュメントルートから応じたサービスのautoload.phpを読み込む)
@[4](Classmapをマージ)
@[5-7](PSR-4をマージ)
@snapend

---

PHPStanで依存の違反を検出する

---

@snap[south-west span-50 text-07]
```text
lenet
├── lenet_common
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
├── lenet.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
├── lenet-hokan.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
├── futonlenet.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
├── kutsulenet.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
└── wh-plus.com
    ├── public
    │   └── index.php
    ├── composer.json
    ├── phpstan.neon
    └── vendor
```
@snapend

---

```
# vim: set ft=yaml:

parameters:
  paths:
    - %rootDir%/../../../../kuritaku.jp/app
    - %rootDir%/../../../../kuritaku.jp/controllers
    - %rootDir%/../../../../kuritaku.jp/lib
  autoload_files:
    - %rootDir%/../../../vendor/autoload.php # 共有ライブラリ + Laravel
    - %rootDir%/../../../_ide_helper.php
    - %rootDir%/../../../../kuritaku.jp/vendor/autoload.php
```

---

```
# vim: set ft=yaml:

parameters:
  paths:
    - %rootDir%/../../../app
    - %rootDir%/../../../controllers
    - %rootDir%/../../../lib
  autoload_files:
    - %rootDir%/../../../vendor/autoload.php
    - %rootDir%/../../../_ide_helper.php
```

---

ドキュメントルート

```
<?php

require $_SERVER['DOCUMENT_ROOT'] . '/../../lenet_common/public/index.php';
```

---

ミドルウェアの登録

```
class RouteServiceProvider extends ServiceProvider
{
    public function boot()
    {
        /*
         * サービスに応じたミドルウェアを登録
         *
         * バッチの場合はwh-plus.comのミドルウェアを登録する。（Laravel 4.2の時代の挙動に合わせている）
         */
        switch ($_SERVER['HOST_SUFFIX']) {
            case 'wh-plus.com':
                $this->app->make(Kernel::class)->pushMiddleware(\Com\WhPlus\App\Http\Middleware\BeforeMiddleware::class);

                $this->app['router']->aliasMiddleware('apiAuth', \Com\WhPlus\App\Http\Middleware\AuthenticateApi::class);
                $this->app['router']->aliasMiddleware('auth', \Com\WhPlus\App\Http\Middleware\Authenticate::class);
                ...
                break;
            case 'kuritaku.jp':
                ...
            case 'lenet-hokan.jp':
                ...
            case 'kuritakufuton.net':
                ...
                break;
            default:
                break;
        }

        parent::boot();
    }
```

---

設定ファイル

---

@snap[south-west span-50 text-07]
```text
lenet
├── lenet_common
│   ├── config
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
├── lenet.jp
│   ├── config
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
├── lenet-hokan.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
├── futonlenet.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
├── kutsulenet.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   ├── phpstan.neon
│   └── vendor
└── wh-plus.com
    ├── public
    │   └── index.php
    ├── composer.json
    ├── phpstan.neon
    └── vendor
```
@snapend

---

各サービスの設定ファイルはヘルパークラスを作って読み混んでる

---

Laravelの設定はすべて環境変数で流し込むようにしている
k8sのyaml

```
          env:
            - name: APP_ENV
              value: production
            - name: APP_HOST_PREFIX
              value: www
            - name: APP_SERVICE
              value: lenet.jp
            - name: APP_DB_ENDPOINT
              ...
```

---

ビューパス

```
<?php

    'paths' => [
        realpath(base_path('resources/views')),
        realpath(base_path('../lenet_common/app/views')), // TODO 移行完了したら削除する
        $_SERVER['DOCUMENT_ROOT'].'/../templates',
    ],

```

---

マイグレーション・バッチ・ストレージ

共通
Laravelのデフォルトそのまま
ただバッチはこの前DigDagに移行した

@snap[south span-40]
![IMAGE](assets/img/logo-digdag-sq-tr.png)
@snapend

---

必要なパスの書き換え

```
 $app->bind('path.public', function () {                                                                                 
     return $_SERVER['DOCUMENT_ROOT'];                                                                                   
 });   
```

---

エラーハンドリング


```
    public function report(Exception $exception)
    {
        ...

        // サービス固有のレポート処理
        $this->reportForService($exception);

        parent::report($exception);
    }

    public function render($request, Exception $exception)
    {
        ...

        // サービス固有のレンダー処理
        $response = $this->renderForService($request, $exception);
        if (!is_null($response)) {
            return $response;
        }

        return parent::render($request, $exception);
    }
```

---

```
<?php

class Handler extends BaseExceptionHandler
{
    /**
     * 例外をレポート、もしくはログする
     *
     * ここは例外をSentryやBugsnagなどへ送るために適した場所
     *
     * @param  \Exception  $exception
     * @return void
     */
    protected function reportForService(Exception $exception)
    {
    }

    /**
     * HTTPレスポンスへ例外をレンダー
     *
     * @param  \Illuminate\Http\Request $request
     * @param  \Exception               $exception
     * @return \Illuminate\Http\Response|null
     */
    public function renderForService($request, Exception $exception)
    {
        if (ErrorResponseUtil::is404($exception) || ErrorResponseUtil::is405($exception)) {
            if (starts_with($request->path(), ['plan', 'sp/plan'])) {
                return Redirect::to('/plan');
            }

            // Error 404
            return Response::stream(function () {
                if (Agent::isMobile()) {
                    include public_path('sp/errordoc/404.php');
                } else {
                    include public_path('errordoc/404.php');
                }
            }, 404);
        }

        // 開発環境のシステムエラーはトレース表示させたいのでレスポンスを作らない
        if (!Config::get('app.debug')) {
            // Error Unknown
            return Response::view('error.' . TemplateUtil::baseDir() . '.index', [], 500);
        }
    }
}
```

---

サービスごとを疎結合にしつつ
共通のコードは共有できる

ゆくゆくはマイクロサービスかも目指す？

---

ご静聴ありがとうございました

---

### 一つのプロジェクトで複数のcomposer.jsonを読み込みたくなった

---

なんで？

---

リネットのLaravelのディレクトリ構成のクセがすごい

---

@snap[north span-100]
リネットのLaravelのディレクトリ構成
@snapend

@snap[south-west span-50 text-07]
```text
lenet
├── lenet_common
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   └── vendor
├── lenet.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   └── vendor
├── lenet-hokan.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   └── vendor
├── futonlenet.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   └── vendor
├── kutsulenet.jp
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   └── vendor
└── wh-plus.com
    ├── public
    │   └── index.php
    ├── composer.json
    └── vendor
```
@snapend

@snap[east span-50]
@[2](Laravelと共有ライブラリ)
@[2, 3-4](Laravelのindex.php)
@[2, 5-6](Laravelのcomposer.jsonとvendor<br>共有ライブラリをpsr-4に登録している)
@[7](衣類のコード)
@[7, 8-9](衣類のドキュメントルート<br>Laravelのindex.phpをrequireしている)
@[7, 10-11](衣類のcomposer.jsonとvendor<br>名前空間なしのレガシーなコードをclassmapに登録している)
@[12-16](保管のコード<br>※衣類と同じ)
@[17-21](布団のコード<br>※衣類と同じ)
@[22-26](靴のコード<br>※衣類と同じ)
@[27-31](社内・工場CMSのコード<br>※衣類と同じ)
@snapend

---

### Composer API

https://getcomposer.org/apidoc/master/index.html

---

<iframe class="stretch" src="https://getcomposer.org/apidoc/master/index.html"></iframe>

---

```php
$loader = require base_path() . '/vendor/autoload.php';
$serviceLoader = require realpath($_SERVER['DOCUMENT_ROOT'])
    . '/../vendor/autoload.php';
$loader->addClassMap($serviceLoader->getClassMap());
foreach ($serviceLoader->getPrefixesPsr4() as $prefix => $paths) {
    $loader->addPsr4($prefix, $paths);
}
```

@snap[south span-100]
@[1](Laravel + 共有ライブラリのautoload.phpを読み込む)
@[2-3](ドキュメントルートから応じたサービスのautoload.phpを読み込む)
@[4](Classmapをマージ)
@[5-7](PSR-4をマージ)
@snapend

---

@snap[midpoint span-100]
![IMAGE](assets/img/lenet-package-1.png)
@snapend

+++

@snap[midpoint span-100]
![IMAGE](assets/img/lenet-package-2.png)
@snapend

@snap[south span-100]
https://www.lenet.jp/ にアクセスした時
@snapend

+++

@snap[midpoint span-100]
![IMAGE](assets/img/lenet-package-3.png)
@snapend

@snap[south span-100]
https://www.lenet-hokan.jp/ にアクセスした時
@snapend

+++

@snap[midpoint span-100]
![IMAGE](assets/img/lenet-package-4.png)
@snapend

@snap[south span-100]
https://www.futonlenet.jp/ にアクセスした時
@snapend

+++

@snap[midpoint span-100]
![IMAGE](assets/img/lenet-package-5.png)
@snapend


@snap[south span-100]
https://www.kutsulenet.jp/ にアクセスした時
@snapend

+++

@snap[midpoint span-100]
![IMAGE](assets/img/lenet-package-6.png)
@snapend

@snap[south span-100]
社内・工場CMSにアクセスした時
@snapend

---

Composer APIを使えば単純にcomposer.jsonを読み込む以外にも色々できる
（ご利用は計画的に）

---

ご静聴ありがとうございました
