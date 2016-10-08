---
title: yii2-asset-manager
date: 2016-10-08 18:17:45
desc:
tags:
---

新公司使用了yii2, 工作需要, 查阅了yii2的asset管理, 解决很多css, js, 管理问题. 做点记录[原文](http://www.yiiframework.com/doc-2.0/guide-structure-assets.html).

<!-- more -->

## Yii2 Asset可以理解为一个集合, 把对应的css, js圈起来, 当asset在view注册时候, 会把对应的css, js引入

### 快速使用

* 定义Asset `框架根目录/assets/AppAsset.php`

    ```php
    namespace app\assets;
    use yii\web\AssetBundle;
    class AppAsset extends AssetBundle
    {
        public $basePath = '@webroot'; // 文件路径
        public $baseUrl = '@web'; // 访问url

        public $css = [
            'css/site.css', // 引入site.css
        ];

        public $js = [
            'js/site.js' // 引入site.js
        ];
        public $depends = [
            'yii\web\YiiAsset', // 依赖YiiAsset And BootstrapAsset
            'yii\bootstrap\BootstrapAsset',
        ];

        public $jsOptions = [
            condition' => 'lte IE9'
        ]; // css 配置, css引入位置,设置浏览器版本等

        public $cssOptions = [
            'position' => '\yii\web\View::POS_HEAD',
        ]; // js配置, js引入配置等等.
    } 
    ```

* 随便找个视图 `框架根目录/view`

    ```php
        <?php \app\assets\AppAsset::register($this); ?>
        <?php $this -> title = "Yii2 Asset Demo";?>
        <div class="container">
            <h1> Hello Yii2 Asset </h1>
        </div>
    ```

* 访问页面, 发现css, js, jquery, bootstrap...都引入到页面了.


### 动态引入对应的js文件

    ```php
        $bundle->js[] = 'i18n/' . \Yii::$app->language . '.js'; // 根据framework设置的语言, 引入i18n目录下对应语言的js文件
    ```
**不推荐这样使用, 语言对应最好是放在js处理.**


### 合理的使用asset(个人理解, 如有不妥, 欢迎拍砖)

1. 架设一个应用分前台和后台, 有必要新建三个 `CommonAsset`(前后台都需要用的css, js), `BackendAsset`(后台用的公共css, js), `FrontendAsset`(前台公共使用的css, js)

    ```php

        // common asset define
        namespace app\assets;
        class CommonAsset extend AssetBundle
        {
            public $basePath = "common/path";
            public $baseUrl = "/";

            public $css = [
                'css/common.css'
            ];

            public $js = [
                'js/common.js',
            ];
        }


        // backend asset define
        namespace app\assets;
        class BackendAsset extends AssetBundle
        {
            public $basePath = "backend/path";
            public $baseUrl = "/";

            public $depends = [
                'app\assets\CommonAsset'
            ];
        }


        // frontend asset define
        namespace app\Assets;
        class FrontendAsset extends AssetBundle
        {
            public $basePath = "frontend/path";
            public $baseUrl = "/";
            public $depends = [
                'app\assets\CommonAsset'
            ];
        }

    ```
2. 在对应的前台独立的view, 对应依赖 `FrontendAsset`, 对应的后台独立view 依赖对应`BackendAsset`. 实现了asset管理


### 压缩优化

#### 压缩策略

1. 将所有的css压缩成单个app.min.css, 所有的js文件压缩成单个app.min.js
2. 分组, 前台css, js, 压缩成对应的frontend.min.css,frontend.min.js. 后台的css, js对应压缩成backend.min.css, backend.min.js


#### 使用`yii asset/template assets-tpl.php`先生成配置模版

1. 模版配置好后, 通过命令生成`yii asset assets-tpl.php assets-prod.php `, 生成用于生产环境的配置文件.

#### 配置策略1

* 配置如下

    ```php

        // 声明webroot, web目录, 根据自己应用目录, 在console环境下, 必须声明
        Yii::setAlias('@webroot', __DIR__ . '/../../web');
        Yii::setAlias('@web', '/');

        return [
            'jsCompressor' => 'java -jar compiler.jar --js {from} --js_output_file {to}',
            'cssCompressor' => 'java -jar yuicompressor.jar --type css {from} -o {to}',
            'deleteSource' => false,
            'bundles' => [ // 将所有assets/下的asset都写在这个数组中
                'yii\web\YiiAsset',
                'yii\web\JqueryAsset',
            ],
            'targets' => [
                'all' => [
                    'class' => 'yii\web\AssetBundle',
                    'basePath' => '@webroot/assets',
                    'baseUrl' => '@web/assets',
                    'js' => 'js/all-{hash}.js',
                    'css' => 'css/all-{hash}.css',
                ],
            ],
            // Asset manager configuration:
            'assetManager' => [
                'basePath' => '@webroot/assets',
                'baseUrl' => '@web/assets',
            ],
        ];
    ```

#### 配置策略2

* 配置如下

    ```php

    Yii::setAlias('@webroot', __DIR__ . '/../../web');
    Yii::setAlias('@web', '/');

    return [
        // Adjust command/callback for JavaScript files compressing:
        'jsCompressor' => 'java -jar compiler.jar --language_in=ECMASCRIPT5 --js {from} --js_output_file {to}',
        // Adjust command/callback for CSS files compressing:
        'cssCompressor' => 'java -jar yuicompressor.jar --type css {from} -o {to}',
        // The list of asset bundles to compress:
        'bundles' => [
            'app\assets\BackendAsset',
            'app\assets\FrontendAsset',
            'app\assets\MapAsset',
            'app\assets\AppAsset',
            'app\assets\MapMonitorAsset',
            /*
            'yii\web\YiiAsset',
            'yii\web\JqueryAsset',
            'yii\bootstrap\BootstrapAsset',
            */
        ],
        // Asset bundle for compression output:
        'targets' => [
            /*
             *
            'appAseets' => [
                'class' => 'yii\web\AssetBundle',
                'basePath' => '@webroot',
                'baseUrl' => '@web',
                'js' => 'js/all-{hash}.js',
                'css' => 'css/all-{hash}.css',
            ],
             */

            'share' => [
                'class' => 'app\assets\AppAsset',
                'basePath' => '@webroot',
                'baseUrl' => '@web',
                'js' => 'dist/js/share-{hash}.js',
                'css' => 'dist/css/share-{hash}.css',
                'depends' => [
                    'app\assets\AppAsset',
                ]
            ],

            'frontend' => [
                'class' => 'app\assets\FrontendAsset',
                'basePath' => '@webroot',
                'baseUrl' => '@web',
                'js' => 'dist/js/frontend-{hash}.js',
                'css' => 'dist/css/frontend-{hash}.css',
                'depends' => []
            ],

            'backend' => [
                'class' => 'app\assets\BackendAsset',
                'basePath' => '@webroot',
                'baseUrl' => '@web',
                'js' => 'dist/js/backend-{hash}.js',
                'css' => 'dist/css/backend-{hash}.css',
                'depends' => [
                    'app\assets\MapAsset',
                ]

            ]
        ],
        // Asset manager configuration:

        'assetManager' => [
            'basePath' => '@webroot/dist',
            'baseUrl' => '@web/dist',
        ],
    ];
    
    ```

#### 使用`yii asset assets-tpl.php`生成对应使用的线上环境的配置文件.

前提: 压缩和合并文件, 框架默认使用[compiler.jar](https://github.com/google/closure-compiler) 和 [yuicompressor.jar](http://yui.github.io/yuicompressor/), 先安装好java环境, 在下载对应的`compiler.jar` 和 `yuiicompressor.jar`到`yii|yii.bat`的目录下, 安装好环境, 方可执行如下命令.

    ```php
        yii asset assets-tpl.php assets-prod.php
    ```

* 将生成的`assets-prod.php`配置文件配置在`components => ['assetManager']`段. 如下

    ```php
    components' => [
        'assetManager' => [
            'appendTimestamp' => true, // 将引入的css, js 末尾添加v=timestamp. 使用`filectime($filename)`实现.
            'bundles' => require(__DIR__ . '/' . (YII_ENV_PROD ? 'assets-prod.php' : 'assets-dev.php')),  
        ],
    ],
    ```
**对应`assets-dev.php` 返回一个空数组, 或对应的的assetManager配置信息.**


### 总结

* **对于web端, 策略2是更合适的, 针对hybrid应用, 首选策略1**.
* **压缩也是有条件的, 之前提到的动态映入js,以及映入cdn的js都部不可以的**
* **yuicompressor.jar下在2.4.7或一下版本, 2.4.8会java.io.FileNotFoundException.**





