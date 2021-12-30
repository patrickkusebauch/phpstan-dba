# database handling class reflection extension for PHPStan

This extension provides following features:

* `PDO->query` knows the array shape of the returned results and therefore can return a generic `PDOStatement`
* `mysqli->query` knows the array shape of the returned results and therefore can return a generic `mysqli_result`
* `SyntaxErrorInQueryMethodRule` can inspect sql queries and detect syntax errors

[see the unit-testsuite](https://github.com/staabm/phpstan-dba/tree/main/tests/data) to get a feeling about the current featureset.

__Its really early days... and this libs has a few rough edges.__

## Usage

To get the extension running you need to configure the `phpstan-dba`.

1. Include the [`dba.neon`](https://github.com/staabm/phpstan-dba/blob/main/config/dba.neon) from within your PHPStan configuration.

2. Additionally your `bootstrap` file needs to be [configured within your phpstan configuration](https://phpstan.org/config-reference#bootstrap), so it will be automatically included by PHPStan:

```php
<?php // bootstrap.php

use staabm\PHPStanDba\QueryReflection\MysqliQueryReflector;
use staabm\PHPStanDba\QueryReflection\QueryReflection;
use staabm\PHPStanDba\QueryReflection\RecordingQueryReflector;
use staabm\PHPStanDba\QueryReflection\ReplayQueryReflector;
use staabm\PHPStanDba\QueryReflection\ReflectionCache;

require_once __DIR__ . '/vendor/autoload.php';

$cacheFile = __DIR__.'/.phpstan-dba.cache';

QueryReflection::setupReflector(
    new RecordingQueryReflector(
        ReflectionCache::create(
            $cacheFile
        ),
        // XXX put your database credentials here
        new MysqliQueryReflector(new mysqli('mysql57.ab', 'testuser', 'test', 'phpstan-dba'))
    )
);
```

As you can see, `phpstan-dba` requires a `mysqli` connection to introspect the database.

### Record and Replay

In case you cannot be sure to have a database running at PHPStan analysis time, you can use the [`RecordingQueryReflector`](https://github.com/staabm/phpstan-dba/blob/main/src/QueryReflection/RecordingQueryReflector.php) to record the reflection information:

```php
<?php // bootstrap.php

use staabm\PHPStanDba\QueryReflection\MysqliQueryReflector;
use staabm\PHPStanDba\QueryReflection\QueryReflection;
use staabm\PHPStanDba\QueryReflection\RecordingQueryReflector;
use staabm\PHPStanDba\QueryReflection\ReplayQueryReflector;
use staabm\PHPStanDba\QueryReflection\ReflectionCache;

require_once __DIR__ . '/vendor/autoload.php';

$cacheFile = __DIR__.'/.phpstan-dba.cache';

QueryReflection::setupReflector(
    new ReplayQueryReflector(
        ReflectionCache::load(
            $cacheFile
        )
    )
);
```

This might be usefull if your CI pipeline cannot connect to your development database server for whatever reason.

The GitHubActions setup of `phpstan-dba` is [using this technique to replay the reflection information](https://github.com/staabm/phpstan-dba/blob/main/bootstrap.php).

## Installation

```shell
composer require --dev staabm/phpstan-dba
```

## Todos

- support the [phpstan/extension-installer](https://github.com/phpstan/extension-installer)
- support [more mysql to PHPStan type mappings](https://github.com/staabm/phpstan-dba/blob/b868f40c80afcecd3de408df3801b5a24e220dd8/src/QueryReflection/MysqliQueryReflector.php#L111)
- cover more real world examples and fine tune the [QueryReflection classes](https://github.com/staabm/phpstan-dba/tree/main/src/QueryReflection)
- support a PDO based QueryReflector
- support running `RecordingQueryReflector` in PHPStan-non-debug mode (currently we see concurrency issues while building the cache)
