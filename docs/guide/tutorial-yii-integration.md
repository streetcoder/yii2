Working with Third-Party Code
=============================

From time to time, you may need to use some third-party code in your Yii applications. Or you may want to
use Yii as a library in some third-party systems. In this section, we will show how to achieve these goals.


## Using Third-Party Libraries in Yii <a name="using-libs-in-yii"></a>

To use a third-party library in a Yii application, you mainly need to make sure the classes in the library
are properly included or can be autoloaded.


### Using Composer Packages <a name="using-composer-packages"></a>

Many third-party libraries are released in terms of [Composer](https://getcomposer.org/) packages.
You can install such libraries by taking the following two simple steps:

1. modify the `composer.json` file of your application and specify which Composer packages you want to install.
2. run `php composer.phar install` to install the specified packages.

The classes in the installed Composer packages can be autoloaded using the Composer autoloader. Make sure
the [entry script](structure-entry-scripts.md) of your application contains the following lines to install
the Composer autoloader:

```php
// install Composer autoloader
require(__DIR__ . '/../vendor/autoload.php');

// include Yii class file
require(__DIR__ . '/../vendor/yiisoft/yii2/Yii.php');
```


### Using Downloaded Libraries <a name="using-downloaded-libs"></a>

If a library is not released as a Composer package, you should follow its installation instructions to install it.
In most cases, you will need to download a release file manually and unpack it in the `BasePath/vendor` directory,
where `BasePath` represents the [base path](structure-applications.md#basePath) of your application.

If a library carries its own class autoloader, you may install it in the [entry script](structure-entry-scripts.md)
of your application. It is recommended the installation is done before you include the `Yii.php` file so that
the Yii class autoloader can take precedence in autoloading classes.

If a library does not provide a class autoloader, but its class naming follows [PSR-4](http://www.php-fig.org/psr/psr-4/),
you may use the Yii class autoloader to autoload the classes. All you need to do is just to declare a
[root alias](concept-aliases.md#defining-aliases) for each root namespace used in its classes. For example,
assume you have installed a library in the directory `vendor/foo/bar`, and the library classes are under
the `xyz` root namespace. You can include the following code in your application configuration:

```php
[
    'aliases' => [
        '@xyz' => '@vendor/foo/bar',
    ],
]
```

If neither of the above is the case, it is likely that the library relies on PHP include path configuration to
correctly locate and include class files. Simply follow its instruction on how to configure the PHP include path.

In the worst case when the library requires explicitly including every class file, you can use the following method
to include the classes on demand:

* Identify which classes the library contains.
* List the classes and the corresponding file paths in `Yii::$classMap` in the [entry script](structure-entry-scripts.md)
  of the application. For example,
```php
Yii::$classMap['Class1'] = 'path/to/Class1.php';
Yii::$classMap['Class2'] = 'path/to/Class2.php';
```


## Using Yii in Third-Party Systems <a name="using-yii-in-others"></a>

Because Yii provides many excellent features, sometimes you may want to use some of its features to support
developing or enhancing 3rd-party systems, such as WordPress, Joomla, or applications developed using other PHP
frameworks. For example, you may want to use the [[yii\helpers\ArrayHelper]] class or use the
[Active Record](db-active-record.md) feature in a third-party system. To achieve this goal, you mainly need to
take two steps: install Yii, and bootstrap Yii.

If the third-party system uses Composer to manage its dependencies, you can simply run the following commands
to install Yii:

```
php composer.phar require yiisoft/yii2-framework:*
php composer.phar install
```

Otherwise, you can [download](http://www.yiiframework.com/download/) the Yii release file and unpack it in
the `BasePath/vendor` directory.

Next, you should modify the entry script of the 3rd-party system by including the following code at the beginning:

```php
require(__DIR__ . '/../vendor/yiisoft/yii2/Yii.php');

$yiiConfig = require(__DIR__ . '/../config/yii/web.php');
new yii\web\Application($yiiConfig); // Do NOT call run() here
```

As you can see, the code above is very similar to that in the [entry script](structure-entry-scripts.md) of
a typical Yii application. The only difference is that after the application instance is created, the `run()` method
is not called. This is because by calling `run()`, Yii will take over the control of the request handling workflow.

Like in a Yii application, you should configure the application instance based on the environment running
the third-party system. For example, to use the [Active Record](db-active-record.md) feature, you need to configure
the `db` application component with the DB connection setting used by the third-party system.

Now you can use most features provided by Yii. For example, you can create Active Record classes and use them
to work with databases.


## Using Yii 2 with Yii 1 <a name="using-both-yii2-yii1"></a>

If you were using Yii 1 previously, it is likely you have a running Yii 1 application. Instead of rewriting
the whole application in Yii 2, you may just want to enhance it using some of the features only available in Yii 2.
This can be achieved as described below.

> Note: Yii 2 requires PHP 5.4 or above. You should make sure that both your server and the existing application
  support this.

First, install Yii 2 in your existing application by following the instructions given in the last subsection.

Second, modify the entry script of the application as follows,

```php
// include the customized Yii class described below
require(__DIR__ . '/../components/Yii.php');

// configuration for Yii 2 application
$yii2Config = require(__DIR__ . '/../config/yii2/web.php');
new yii\web\Application($yii2Config); // Do NOT call run()

// configuration for Yii 1 application
$yii1Config = require(__DIR__ . '/../config/yii1/main.php');
Yii::createWebApplication($yii1Config)->run();
```

Because both Yii 1 and Yii 2 have the `Yii` class, you should create a customized version to combine them.
The above code includes the customized `Yii` class file, which can be created as follows.

```php
$yii2path = '/path/to/yii2';
require($yii2path . '/BaseYii.php'); // Yii 2.x

$yii1path = '/path/to/yii1';
require($yii1path . '/YiiBase.php'); // Yii 1.x

class Yii extends \yii\BaseYii
{
    // copy-paste the code in YiiBase (1.x) here
}

Yii::$classMap = include($yii2path . '/classes.php');

// register Yii2 autoloader via Yii1
Yii::registerAutoloader(['Yii', 'autoload']);
```

That's all! Now in any part of your code, you can use `Yii::$app` to access the Yii 2 application instance, while
`Yii::app()` will give you the Yii 1 application instance:

```php
echo get_class(Yii::app()); // outputs 'CWebApplication'
echo get_class(Yii::$app);  // outputs 'yii\web\Application'
```