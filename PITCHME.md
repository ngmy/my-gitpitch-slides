@snap[midpoint span-100 text-05]
# Composer APIで実現する単一リポジトリ複数composer.json
@snapend

@snap[south-east span-100 text-07]
Dec 01, 2019
PHPカンファレンス2019 懇親会LT
永宮　悠大
@snapend

---

@snap[north-west]
## 自己紹介
@snapend

@snap[west span-55]
永宮　悠大
（NAGAMIYA Yuta）

- 株式会社ホワイトプラスのエンジニア
- PHPの下回りを見てます
- @fa[github] [@ngmy](https://github.com/ngmy)
- クライミング8：エンジニアリング2
@snapend

@snap[east span-45]
![IMAGE](assets/img/profile.jpg)
@snapend

---

@snap[north span-100]
## ホワイトプラスを知ってる人？ &#x1f64b;
@snapend

@snap[midpoint span-90]
![IMAGE](assets/img/wplogo.png)
@snapend

@snap[south span-100]
https://www.wh-plus.co.jp/
@snapend

+++

@snap[midpoint span-100]
![IMAGE](assets/img/sponsor.png)
@snapend

---

@snap[north span-100]
## リネット
@snapend

@snap[midpoint span-90]
![IMAGE](assets/img/lenet-service.png)
@snapend

@snap[south span-100 text-05]
- 衣類クリーニング
    - https://www.lenet.jp/
- 衣類クリーニング + 保管
    - https://www.lenet-hokan.jp/
- 布団クリーニング
    - https://www.futonlenet.jp/
- 靴クリーニング
    - https://www.kutsulenet.jp/</dd>
@snapend

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

+++

@snap[midpoint span-100]
![IMAGE](assets/img/lenet-package-3.png)
@snapend

+++

@snap[midpoint span-100]
![IMAGE](assets/img/lenet-package-4.png)
@snapend

+++

@snap[midpoint span-100]
![IMAGE](assets/img/lenet-package-5.png)
@snapend

+++

@snap[midpoint span-100]
![IMAGE](assets/img/lenet-package-6.png)
@snapend

---

Composer APIを使えば単純にcomposer.jsonを読み込む以外にも色々できる
（ご利用は計画的に）

---

ご静聴ありがとうございました
