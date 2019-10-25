@snap[midpoint span-100 text-05]
# Composer APIを使った動的クラスローディング
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

@snap[north span-100]
## ホワイトプラス
@snapend

@snap[midpoint span-90]
![IMAGE](assets/img/wplogo.png)
@snapend

@snap[south span-100]
https://www.wh-plus.co.jp/
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

### composer.jsonを動的に読み込みたくなった

---

なんで？

---

@snap[north span-100]
特殊なLaravelの使い方をしているせい
@snapend

@snap[south-west span-50 text-08 text-red]
```text
リネット
├── 衣類
│   ├── ...
│   ├── composer.json
│   └── vendor
├── 保管
│   ├── ...
│   ├── composer.json
│   └── vendor
├── 布団
│   ├── ...
│   ├── composer.json
│   └── vendor
├── 靴
│   ├── ...
│   ├── composer.json
│   └── vendor
├── CMS
│   ├── ...
│   ├── composer.json
│   └── vendor
└── Laravel + 共有ライブラリ
    ├── ...
    ├── composer.json    
    └── vendor
```
@snapend

@[2-5]

@snap[east span-50 text-08]
```json
{
    "autoload": {
        "classmap": [
            ...
        ],
        "files": [
            ...
        ]
    }
}
```
@snapend

---

### Composer API

https://getcomposer.org/apidoc/master/index.html

---

<iframe class="stretch" src="https://getcomposer.org/apidoc/master/index.html"></iframe>

---

```
リネット
├── 衣類
│   ├── composer.json
│   └── vendor
├── 保管
│   ├── composer.json
│   └── vendor
├── 布団
│   ├── composer.json
│   └── vendor
├── 靴
│   ├── composer.json
│   └── vendor
├── CMS
│   ├── composer.json
│   └── vendor
└── Laravel + 共有ライブラリ
    ├── composer.json    
    └── vendor
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
