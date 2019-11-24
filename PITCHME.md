@snap[midpoint span-100 text-08 title]
# Laravelで、モジュラモノリス
@snapend

@snap[south-east span-100 text-07]
2019年12月1日（日）
PHPカンファレンス2019 懇親会LT
永宮　悠大
@snapend

---

@snap[north text-07]
## 自己紹介
@snapend

@snap[west span-60 text-08]
- 永宮　悠大（NAGAMIYA Yuta）
- 株式会社ホワイトプラスのエンジニア
- PHP・Laravelの面倒を見る人
- @fa[github] [@ngmy](https://github.com/ngmy)
- クライミング8：エンジニアリング2
@snapend

@snap[east span-40]
![IMAGE](assets/img/profile.jpg)
@snapend

---

@snap[north span-100 text-07]
## 株式会社ホワイトプラス
@snapend

@snap[midpoint span-90]
![IMAGE](assets/img/wplogo.png)
@snapend

@snap[south span-100]
https://www.wh-plus.co.jp/
@snapend

---?image=assets/img/sponsor.png&size=contain

---

@snap[north span-100 text-05]
## ネット宅配クリーニングのリネット
@snapend

@snap[west span-50]
![IMAGE](assets/img/cleaning_result.png)
@snapend

@snap[east span-50 text-07]
- PC・スマホから24時間いつでも注文できて、自宅にいたままお洋服の預け、受け取りができるWebサービス
- 衣類クリーニング
    - https://www.lenet.jp/
- 衣類クリーニング保管
    - https://www.lenet-hokan.jp/
- 布団クリーニング
    - https://www.futonlenet.jp/
- 靴クリーニング
    - https://www.kutsulenet.jp/</dd>
- PHP + Laravelで構築
@snapend

---

@snap[north span-100 text-07]
## モノリス
@snapend

@snap[midpoint]
![IMAGE](assets/img/toy_dorodango_kirei.png)
@snapend

---

@snap[north span-100 text-07]
## モジュラモノリス
@snapend

@snap[midpoint]
![IMAGE](assets/img/rubiks_cube.png)
@snapend

---

- モノリスから複数のマイクロサービスを抽出するより、まずアプリ内をモジュラーにしていく
- 独立したサービスとして抽出する前にアーキテクチャ上のよい境界を見つける。スムーズに移行しやすいようにコードを構成しておく

---

## モジュールに分割する

---

@snap[north span-100 text-07]
### リネットリポジトリのディレクトリ構成
@snapend

@snap[west span-40]
```text
lenet
├── lenet_common
├── lenet.jp
├── lenet-hokan.jp
├── futonlenet.jp
├── kutsulenet.jp
└── wh-plus.com
```
@snapend

@snap[east span-60 text-center]
@[1-7](1つの共有コード + 5つのサービス)
@[2](共有コード（Laravel + 共有ライブラリ）)
@[3](サービス1（衣類）)
@[4](サービス2（保管）)
@[5](サービス3（布団）)
@[6](サービス4（靴）)
@[7](サービス5（社内・工場CMS）)
@snapend

---

@snap[north span-100 text-07]
### 共有コードのディレクトリ構成
@snapend

@snap[west span-40]
```text
lenet_common
├── app
├── bootstrap
├── config
├── database
├── lib
├── public
├── resources
├── routes
├── storage
├── tests
├── vendor
├── composer.json
└── phpstan.neon
```
@snapend

@snap[east span-60 text-center]
@[2-5, 7-13](Laravel)
@[6](共有ライブラリ)
@[14](PHPStanの設定ファイル)
@snapend

---

@snap[north span-100 text-07]
### 各サービスのディレクトリ構成
@snapend

@snap[west span-40]
```text
lenet.jp
├── app
├── config
├── controllers
├── lib
├── public
├── templates
├── vendor
├── composer.json
└── phpstan.neon
```
@snapend

@snap[east span-60 text-center]
@[2](ルートファイル、ミドルウェア、例外ハンドラ等)
@[3](サービス固有の設定)
@[4](コントローラ)
@[5](ライブラリ)
@[6](ドキュメントルート)
@[7](テンプレート)
@[8-9](composer.json + vendor)
@[10](PHPStanの設定ファイル)
@snapend

---

@snap[north span-100 text-07]
### 各サービスのindex.php &rarr; Laravelのindex.php
@snapend

#### `lenet.jp/public/index.php`

```php
require $_SERVER['DOCUMENT_ROOT'] . '/../../lenet_common/public/index.php';
```

---

@snap[north span-100 text-07]
### Laravelのルートファイル &rarr; 各サービスのルートファイル
@snapend

#### `lenet_common/routes/web.php`

```php:lenet_common/routes/web.php
require $_SERVER['DOCUMENT_ROOT'] . '/../app/routes.php';
```

---

## 境界を遵守させる

---

Composer APIでオートロードするクラスを制限する

---

### Composer API

https://getcomposer.org/apidoc/master/index.html

---

<iframe class="stretch" src="https://getcomposer.org/apidoc/master/index.html"></iframe>

---

`lenet_common/app/Providers/AppServiceProvider.php`の`register`メソッド

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

## PHPStanで依存の違反を検出する

---

@snap[north span-100 text-07]
### 各サービスのphpstan.neon
@snapend

#### `lenet.jp/phpstan.neon`

```yaml
parameters:
  paths:
    - %rootDir%/../../../../lenet.jp/app
    - %rootDir%/../../../../lenet.jp/controllers
    - %rootDir%/../../../../lenet.jp/lib
  autoload_files:
    - %rootDir%/../../../vendor/autoload.php
    - %rootDir%/../../../_ide_helper.php
    - %rootDir%/../../../../lenet.jp/vendor/autoload.php
```

@snap[south span-100]
@[6-8](共有コードのクラスをオートロード)
@[6,9](自サービスのクラスをオートロード)
@snapend

---

@snap[north span-100 text-07]
### 共有コードのphpstan.neon
@snapend

#### `lenet_common/phpstan.neon`

```yaml
parameters:
  paths:
    - %rootDir%/../../../app
    - %rootDir%/../../../controllers
    - %rootDir%/../../../lib
  autoload_files:
    - %rootDir%/../../../vendor/autoload.php
    - %rootDir%/../../../_ide_helper.php
```

@snap[south span-100]
@[6-8](共有コードのクラスをオートロード)
@snapend

---

@snap[midpoint span-100]
![IMAGE](assets/img/lenet-package-7.png)
@snapend

---

サービスごとを疎結合にしつつ
共通のコードは共有できる

ゆくゆくはマイクロサービスかも目指す？

---

ご静聴ありがとうございました

---

まだ時間がある？

---

@snap[north span-100]
### ミドルウェアの登録
@snapend

`lenet_common/app/Providers/RouteServiceProvider.php`の`boot`メソッド

@snap[span-100]
```php
if ($_SERVER['SERVICE_NAME'] == 'lenet.jp') {
    $this->app->make(Kernel::class)->pushMiddleware(
        \Jp\Lenet\App\Http\Middleware\BeforeMiddleware::class
    );
    // ...
    $this->app['router']->aliasMiddleware(
        'auth',
        \Jp\Lenet\App\Http\Middleware\Authenticate::class
    );
    // ...
}
```
@snapend

---

@snap[north span-100 text-07]
### 環境設定
@snapend

- Laravelの設定はすべて環境変数で流し込むようにしている
    - サービスごとにk8sのyamlがある
- .envファイルは使っていない

#### `lenet-jp.yaml`

```yaml
env:
  - name: APP_ENV
    value: production
  - name: APP_HOST_PREFIX
    value: www
  - name: APP_SERVICE
    value: lenet.jp
  - name: APP_DB_ENDPOINT
  # ...
```

---

### 設定ファイル

各サービスの設定ファイルはヘルパークラスを作って読み混んでる

---

@snap[north span-100 text-07]
### ビューパス
@snapend

#### `lenet_common/config/view.php`

```php
'paths' => [
    realpath(base_path('resources/views')),
    $_SERVER['DOCUMENT_ROOT'] . '/../templates',
],
```

---

@snap[north span-100 text-07]
### マイグレーション・バッチ・ストレージ
@snapend

- 全てのサービスで共有
- Laravelのデフォルトそのまま
- ただし、バッチはこの前DigDagに移行した

@snap[south span-40]
![IMAGE](assets/img/logo-digdag-sq-tr.png)
@snapend

---

@snap[north span-100 text-07]
### 必要なパスの書き換え
@snapend

#### `lenet_common/bootstrap/app.php`

```
 $app->bind('path.public', function () {                                                                                 
     return $_SERVER['DOCUMENT_ROOT'];                                                                                   
 });   
```

---

@snap[north span-100 text-07]
### エラーハンドラの登録
@snapend

#### `lenet_common/app/Providers/AppServiceProvider.php`の`register`メソッド

```php
if ($_SERVER['SERVICE_NAME'] == 'lenet.jp') {
    $this->app->singleton(
        \Illuminate\Contracts\Debug\ExceptionHandler::class,
        \Jp\Lenet\App\Exceptions\Handler::class
    );
}
```

---

サービスごとを疎結合にしつつ
共通のコードは共有できる

ゆくゆくはマイクロサービスかも目指す？

---

ご静聴ありがとうございました
