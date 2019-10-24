# Composer APIを使った動的クラスローディング

---

## 自己紹介

### 永宮　悠大（NAGAMIYA Yuta）

- 株式会社ホワイトプラスのエンジニア
- PHPの下回りを見てます
- @ngmy

---

会社・サービス紹介

---

### composer.jsonを動的に読み込みたくなった

---

なんで？

---

特殊なLaravelの使い方をしているせい

```
lenetリポジトリ
├── リネット衣類
│   └── controllers
├── リネット布団
│   └── controllers
├── リネット保管
│   └── controllers
├── リネット靴
│   └── controllers
├── CMS
│   └── controllers
└── Laravel
    ├── app
    ├── bootstrap
    ├── database
    ├── vendor    
    └── lib
```

---

### Composer API

https://getcomposer.org/apidoc/master/index.html

---

```
lenetリポジトリ
├── リネット衣類
│   ├── composer.json
│   └── vendor
├── リネット布団
│   ├── composer.json
│   └── vendor
├── リネット保管
│   ├── composer.json
│   └── vendor
├── リネット靴
│   ├── composer.json
│   └── vendor
├── CMS
│   ├── composer.json
│   └── vendor
└── Laravel
    ├── app
    ├── bootstrap
    ├── database
    ├── vendor    
    └── lib
```

---

```
$loader = require base_path() . '/vendor/autoload.php';
$serviceLoader = require realpath($_SERVER['DOCUMENT_ROOT']) . '/../vendor/autoload.php';
$loader->addClassMap($serviceLoader->getClassMap()); // クラスマップ
foreach ($serviceLoader->getPrefixesPsr4() as $prefix => $paths) { // PSR-4
    $loader->addPsr4($prefix, $paths);
}
```

---

Composer APIを使えば単純にcomposer.jsonを読み込む以外にも色々できる
（ご利用は計画的に）
