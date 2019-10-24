@snap[north]
# Composer APIを使った動的クラスローディング
@snapned

@snap[south]
Dec 01, 2019
PHPカンファレンス2019 懇親会LT
永宮　悠大
@snapned

---

@snap[north-west]
## 自己紹介
@snapend

@snap[west span-55]
永宮　悠大（NAGAMIYA Yuta）
- 株式会社ホワイトプラスのエンジニア
- PHPの下回りを見てます
- @fa[github] [@ngmy](https://github.com/ngmy)
- クライミング8：エンジニアリング2
@snapend

@snap[east span-45]
![IMAGE](assets/img/profile.jpg)
@snapend

---

## ホワイトプラス

![IMAGE](assets/img/wplogo.png)

https://www.wh-plus.co.jp/

---

## リネット

![IMAGE](assets/img/lenet-service.png)

衣類クリーニング https://www.lenet.jp/
衣類クリーニング + 保管 https://www.lenet-hokan.jp/
布団クリーニング https://www.futonlenet.jp/
靴クリーニング https://www.kutsulenet.jp/



### composer.jsonを動的に読み込みたくなった

---

なんで？

---

特殊なLaravelの使い方をしているせい

```
リネットリポジトリ
├── リネット衣類
│   ├── public
│   │   └── index.php
│   ├── controllers // 衣類のコントローラ
│   └── lib // 衣類のライブラリ
├── 保管
├── 布団
├── 靴
├── CMS
└── Laravel + 共有ライブラリ
    ├── app
    ├── public
    │   └── index.php
    ├── ... more Laravel Directory
    ├── composer.json
    ├── lib // 共有ライブラリ（PSR-4）
    └── vendor
```

@[1-5, zoom-13]

---

### Composer API

https://getcomposer.org/apidoc/master/index.html

---

```
lenetリポジトリ
├── lenet.jp
│   ├── composer.json
│   └── vendor
├── lenet-hokan.jp
│   ├── composer.json
│   └── vendor
├── futonlenet.jp
│   ├── composer.json
│   └── vendor
├── kutsulenet.jp
│   ├── composer.json
│   └── vendor
├── 車内CMS
│   ├── composer.json
│   └── vendor
└── laravel + common
    ├── app
    ├── bootstrap
    ├── database
    ├── vendor    
    └── lib
```

---

```php
$loader = require base_path() . '/vendor/autoload.php';
$serviceLoader = require realpath($_SERVER['DOCUMENT_ROOT']) . '/../vendor/autoload.php';
$loader->addClassMap($serviceLoader->getClassMap()); // クラスマップ
foreach ($serviceLoader->getPrefixesPsr4() as $prefix => $paths) { // PSR-4
    $loader->addPsr4($prefix, $paths);
}
```

@[1]
@[2]
@[3]
@[4-6]

---

lenet.jpドメインにアクセスした時は共有ライブラリとlenet.jpのコードだけが見える。
他サービスのコードが見えない

---

Composer APIを使えば単純にcomposer.jsonを読み込む以外にも色々できる
（ご利用は計画的に）

---

ご静聴ありがとうございました
