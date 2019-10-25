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

### 一つのプロジェクトで複数のcomposer.jsonを読み込みたくなった

---

なんで？

---

@snap[north span-100]
特殊なLaravelの使い方をしているせい
@snapend

@snap[south-west span-50 text-07]
```text
lenet
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
├── wh-plus.com
│   ├── public
│   │   └── index.php
│   ├── composer.json
│   └── vendor
└── lenet_common
    ├── public
    │   └── index.php
    ├── composer.json
    └── vendor
```
@snapend

@snap[east span-50]
@[2, 3-4]
@[2, 5-6](衣類のコントローラとライブラリをClassmapでオートロードしている)
@[7, 8-9](保管のコントローラとライブラリをClassmapでオートロードしている)
@[7, 10-11](布団のコントローラとライブラリをClassmapでオートロードしている)
@[12, 13-14](靴のコントローラとライブラリをClassmapでオートロードしている)
@[12, 15-16](CMSのコントローラとライブラリをClassmapでオートロードしている)
@[17, 18-19](Laravelと共有ライブラリをPSR-4でオートロードしている)
@[17, 20-21](Laravelと共有ライブラリをPSR-4でオートロードしている)
@[22, 23-24](Laravelと共有ライブラリをPSR-4でオートロードしている)
@[22, 25-26](Laravelと共有ライブラリをPSR-4でオートロードしている)
@snapend

+++

a

+++

b

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

lenet.jpドメインにアクセスした時は共有ライブラリとlenet.jpのコードだけが見える。
他サービスのコードが見えない

---

Composer APIを使えば単純にcomposer.jsonを読み込む以外にも色々できる
（ご利用は計画的に）

---

ご静聴ありがとうございました
