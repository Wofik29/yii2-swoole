# yii2-swoole

For example the Yii2 framework coroutine asynchronous capabilities.

This plugin is based on the  coroutine implementation of the underlying [swoole](https://github.com/swoole/swoole-src)
 transforming the core code of Yii2, making the developer non-aware, and using the asynchronous IO capabilities of swoole without changing the business code.

## Characteristic

- Coroutine MySQL client, connection pool, support for master-slave, transaction.
- Coroutine Redis client, connection pool, cache (currently not intended to support transactions)
- Coroutine HttpClient, dependent on Swoft implementation
- Swoole_table cache component
- Asynchronous file log component
- Business code and swoole main process separation


## Installation

#### Environmental requirements

1. hiredis
2. composer
3. PHP7.X
4. Swoole2.1 and open coroutine and asynchronous Redis

#### Swoole Install

- see https://www.swoole.co.uk/#get-started

#### Composer install

- In the project `composer.json` file, add a dependency:

```json
{
  "require": {
      "deepziyu/yii2-swoole": "*"
  }
}
```

- Execution `$ php composer.phar update` or `$ composer update` installation.



## Configuration

You can refer to  [the sample project](https://gitee.com/lizhenju/yii2-swoole-demo)

Create a new startup file.

The startup file clearly shows the working and process principle of the plug-in. Handwriting this file will help you understand the plugin more.

The swoole.php example is as follows:

```php

defined('YII_DEBUG') or define('YII_DEBUG', true);
defined('YII_ENV') or define('YII_ENV', 'dev');
defined('WEB_ROOT') or define('WEB_ROOT', dirname(__DIR__) . '/web'); //web目录的路径，用户访问的静态文件都放这里

require(__DIR__ . '/../../vendor/autoload.php');

$config = [
    'id' => 'api-test-hello',
    'setting' => [
        // swoole_server configuration. 
        // @see other configuration items see https://wiki.swoole.com/wiki/page/274.html
        'daemonize'=>0,
        'worker_num'=>2,
        'task_worker_num' => 1,
        'log_file' => __DIR__.'/../runtime/logs/swoole.log',
        'log_level'=> 0,
        'chroot' => '/',
    ],
    'cacheTable' => function (){
        // swoole_table need to be started in advance, the size of 2 returns
        return deepziyu\yii\swoole\cache\SwooleCache::initCacheTable(1024);
    },
    'bootstrap' => [
        'class' => 'deepziyu\yii\swoole\bootstrap\YiiWeb',
        'config' => function(){
            // The closure is used to delay loading
            // returning the configuration of each component of Yii            
            require_once(__DIR__ . '/../../vendor/autoload.php');
            require_once(__DIR__ . '/../../yii-swoole/Yii.php');
            require(__DIR__ . '/../config/bootstrap.php');
            $config = yii\helpers\ArrayHelper::merge(
                require(__DIR__ . '/../config/main.php'),
                require(__DIR__ . '/../config/main-local.php'),
                [
                    'components' => [
                      'errorHandler' => [
                           'class'=>'deepziyu\yii\swoole\web\ErrorHandler'
                        ],
                        'cache' => [
                            'class' => 'deepziyu\yii\swoole\cache\SwooleCache',
                        ],
                    ],
                ]
            );

            return $config;
        },
    ],

];

deepziyu\yii\swoole\server\Server::run($config);

```

## Start up

```
php swoole.php start|stop|reload|reload-task
```

## Usage HttpClient 

See [Swotf documentation](https://doc.swoft.org/http.html) for use the HTTP client.

## TODO

- MysqlPool does not currently support transactions. (realized)
- The MysqlPool and RedisPool connection pools are full. Currently, sleep() is used to queue up. After the number of waits is exceeded, an exception is reported.
- MysqlPool does not currently support master-slave. (realized)

## Known Bug

- The iterator will cause the coroutine to hang.

  BUG code:
  ```php
      $models = User::find()->each(10);
      foreach ($models as $model) { // Hang up in this row
           $data[] = $model->toArray();
      }
  ```

## Bug fixed

- new ActiveRecord([]); will trigger __set() in the magic method to call the coroutine Client, resulting in two problems:
  1、  The first instantiation will cause the coroutine to hang.
  2、 If the SQL error directly leads to the termination of the work process.
  BUG code:
  ```php
      class OneModel extend ActiveRecord{
          public static function tableName()
          {
             return 'some-table do not exist';
          }
      }
      // Causes the process to terminate
      $model = new OneModel([
          'some-att'=>'some-value',
      ]);
  ```

## link

[gitee](https://gitee.com/lizhenju/yii2-swoole)

[github](https://github.com/deepziyu/yii2-swoole)

## Chat && Help

Swoft frame QQ exchange group:548173319
