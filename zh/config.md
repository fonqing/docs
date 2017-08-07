<div class='article-menu'>
  <ul>
    <li>
      <a href="#overview">读取配置</a> 
      <ul>
        <li><a href="#factory">配置工厂</a></li>
        <li><a href="#native-arrays">原生数组</a></li>
        <li><a href="#file-adapter">文件适配器</a></li>
        <li><a href="#ini-files">解析ini文件</a></li>
        <li><a href="#merging">合并配置</a></li>
        <li><a href="#nested-configuration">嵌套配置</a></li>
        <li><a href="#injecting-into-di">依赖注入配置</a></li>
      </ul>
    </li>
  </ul>
</div>

<a name='overview'></a>

# 读取配置

`Phalcon\Config` 是用来将应用中用到的各种配置转换为PHP对象的组件。

<a name='factory'></a>

## 配置工厂

配置载入适配器类使用`adapter`选项，如果没有提供扩展，它将被添加到`filePath`。

```php
<?php

use Phalcon\Config;

$config = new Config(
    [
        'test' => [
            'parent' => [
                'property'  => 1,
                'property2' => 'yeah',
            ],
        ],  
    ]
);

echo $config->get('test')->get('parent')->get('property');  // displays 1
echo $config->test->parent->property;                       // displays 1
echo $config->path('test.parent.property');                 // displays 1
```

<a name='factory'></a>

## 配置工厂

配置载入适配器类使用`adapter`选项，如果没有提供扩展，它将被添加到`filePath`。

```php
<?php

use Phalcon\Config\Factory;

$options = [
    'filePath' => 'path/config',
    'adapter'  => 'php',
 ];

 $config = Factory::load($options);
 ```

<a name='native-arrays'></a>
## 内置数组
第一个示例展示如何转换内置配置数组为`Phalcon\Config`对象，此方式性能最佳，因为在请求期间不会读取额外的文件。

```php
<?php

use Phalcon\Config;

$settings = [
    'database' => [
        'adapter'  => 'Mysql',
        'host'     => 'localhost',
        'username' => 'scott',
        'password' => 'cheetah',
        'dbname'   => 'test_db'
    ],
     'app' => [
        'controllersDir' => '../app/controllers/',
        'modelsDir'      => '../app/models/',
        'viewsDir'       => '../app/views/'
    ],
    'mysetting' => 'the-value'
];

$config = new Config($settings);

echo $config->app->controllersDir, "\n";
echo $config->database->username, "\n";
echo $config->mysetting, "\n";
```

如果您想更好地组织项目，可以将数组保存到另一个文件中，然后读取它。

```php
<?php

use Phalcon\Config;

require 'config/config.php';

$config = new Config($settings);
```

<a name='file-adapter'></a>

## 文件适配器

支持的适配器如下:

| 类                            | 描述                                                                                      |
| -------------------------------- | ------------------------------------------------------------------------------------------------ |
| `Phalcon\Config\Adapter\Ini`  | 使用INI文件存储配置信息，使用PHP内置的函数`parse_ini_file`解析。|
| `Phalcon\Config\Adapter\Json` | 使用JSON存储配置信息。 |
| `Phalcon\Config\Adapter\Php`  | 使用  原生PHP数组存储配置信息，性能最好。  |
| `Phalcon\Config\Adapter\Yaml` | 使用YAML文件存储配置信息。                                                               |

<a name='ini-files'></a>

## 读取INI文件

很多应用使用INI文件存储配置信息，Phalcon也可以使用。 `Phalcon\Config` 使用PHP内置的函数 `parse_ini_file` 读取和解析文件。配置文件中每个小节被解析为子设置以方便访问。

```ini
[database]
adapter  = Mysql
host     = localhost
username = scott
password = cheetah
dbname   = test_db

[phalcon]
controllersDir = '../app/controllers/'
modelsDir      = '../app/models/'
viewsDir       = '../app/views/'

[models]
metadata.adapter  = 'Memory'
```

以上INI文件可以用如下方法读取:

```php
<?php

use Phalcon\Config\Adapter\Ini as ConfigIni;

$config = new ConfigIni('path/config.ini');

echo $config->phalcon->controllersDir, "\n";
echo $config->database->username, "\n";
echo $config->models->metadata->adapter, "\n";
```

<a name='merging'></a>

## 合并配置

`Phalcon\Config` 能够递归合并配置到另一个配置。新的配置项会被添加，已有的配置项会更新。

```php
<?php

use Phalcon\Config;

$config = new Config(
    [
        'database' => [
            'host'   => 'localhost',
            'dbname' => 'test_db',
        ],
        'debug' => 1,
    ]
);

$config2 = new Config(
    [
        'database' => [
            'dbname'   => 'production_db',
            'username' => 'scott',
            'password' => 'secret',
        ],
        'logging' => 1,
    ]
);

$config->merge($config2);

print_r($config);
```

以上代码生成如下结构:

```bash
Phalcon\Config Object
(
    [database] => Phalcon\Config Object
        (
            [host] => localhost
            [dbname]   => production_db
            [username] => scott
            [password] => secret
        )
    [debug] => 1
    [logging] => 1
)
```

另外，在 [Phalcon Incubator](https://github.com/phalcon/incubator) 项目中，还有部分其他适配器可用。

<a name='nested-configuration'></a>

## 嵌套配置

还可以通过 `Phalcon\Config::path` 方法使用嵌套配置. 这种方法允许您建立嵌套配置，而不用修改各处的配置. 例子如下:

```php
<?php

use Phalcon\Config;

$config = new Config(
   [
        'phalcon' => [
            'baseuri' => '/phalcon/'
        ],
        'models' => [
            'metadata' => 'memory'
        ],
        'database' => [
            'adapter'  => 'mysql',
            'host'     => 'localhost',
            'username' => 'user',
            'password' => 'passwd',
            'name'     => 'demo'
        ],
        'test' => [
            'parent' => [
                'property' => 1,
                'property2' => 'yeah'
            ],
        ],
   ]
);

//使用“.”分割配置
$config->path('test.parent.property2');    // yeah
$config->path('database.host', null, '.'); // localhost

$config->path('test.parent'); // Phalcon\Config

//使用“/”分割配置
$config->path('test/parent/property3', 'no', '/'); // no

Config::setPathDelimiter('/');
$config->path('test/parent/property2'); // yeah
```

<a name='injecting-into-di'></a>

## 依赖注入配置

您可以通过添加一个服务的方式将配置注入到控制器中，具体方法代码如下。

```php
<?php

use Phalcon\Di\FactoryDefault;
use Phalcon\Config;

// Create a DI
$di = new FactoryDefault();

$di->set(
    'config',
    function () {
        $configData = require 'config/config.php';

        return new Config($configData);
    }
);
```

这样就可以在控制器中通过依赖注入来读取配置， 通过`config`读取配置代码如下:

```php
<?php

use Phalcon\Mvc\Controller;

class MyController extends Controller
{
    private function getDatabaseName()
    {
        return $this->config->database->dbname;
    }
}
```
