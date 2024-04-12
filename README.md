# Archict/Brick

[![Tests](https://github.com/Archict/brick/actions/workflows/tests.yml/badge.svg?branch=master)](https://github.com/Archict/brick/actions/workflows/tests.yml)

> To build something, you need bricks

Base library of Archict framework

## What is a Brick?

A Brick is the base component of Archict. It consists on a collection of [Services](#services) which can
have a [Configuration](#services-configuration) and listen to [Event](#events).

## How to create a Brick?

In technical terms, a Brick is a composer package with the `archict-brick` type. The easiest way to create your own
Brick
is to use our [template](https://github.com/Archict/brick-template).

If you bring a look at the `composer.json` content, you can see this line:

```json
{
  "type": "archict-brick"
}
```

This line is *super mega* important, it's the one telling to Archict that your package is a Brick. Without it, your
Services will never be loaded.

You also need to depend on this package to have all the necessary classes to create your Brick.

### Services

Creating a Service is pretty straight forward, just put the attribute Service on your class:

```php
<?php

use Archict\Brick\Service;

#[Service]
final class MyService {}
```

One of the feature you can have with Services, is dependencies injection. Your Service can depend on some other
Services (from other Bricks for example). For that, just add them in your constructor and Archict will inject them just
for you.

```php
<?php

use Archict\Brick\Service;

#[Service]
final readonly class MyService 
{
    public function __construct(
        private AnotherService $another_service,
    ) {}
}
```

### Services configuration

Sometimes, you need your Service to be configurable by another developer. For that you can give them configuration file.
For that you need to do several things.

First you need to create a data class which store the configuration variables:

```php
<?php

use Archict\Brick\ServiceConfiguration;

#[ServiceConfiguration]
final readonly class MyConfiguration 
{
    public function __construct(
        public int $nb_workers,
    ) {}
}
```

Please note that only `public` members will be considered as part of the configuration.

Then specify to your Service it can use this class as configuration (instance of your configuration can also be injected
in the Service constructor):

```php
<?php

use Archict\Brick\Service;

#[Service(MyConfiguration::class)]
final readonly class MyService 
{
    public function __construct(
        private MyConfiguration $config,
    ) {}
}
```

Finally, provide a default config file (in YAML format) in the `config` folder at the root of your package. This file
will be used as default config unless another config file is provided to Archict. By default, the file is named with
your Service class name lowercased (`myservice.yml`), you can change this behavior by specifying the filename:

```php
<?php

use Archict\Brick\Service;

#[Service(MyConfiguration::class, 'foo.yml')]
final readonly class MyService 
{
}
```

### Events

To bring some life to your Services, they can listen to Events and even dispatch some.

Listening to an Event is pretty easy, just add the `ListeningEvent` attribute to a public method of your Service.
Archict will know which Event you are listening by getting the argument type of the method (so you can name your method
as you want):

```php
<?php

use Archict\Brick\ListeningEvent;
use Archict\Brick\Service;

#[Service]
final class MyService
{
    #[ListeningEvent]
    public function fooBarBaz(MyEvent $event): void
    {
        // Do something with $event
    }
}
```

Dispatching an Event need some steps. First you need to have an Event class:

```php
<?php

use Archict\Brick\Event;

#[Event]
final class MyEvent {}
```

Then add the package `archict/core` to your dependencies. It contains the Service `EventManager` which is necessary to
dispatch an Event. You can now use it in your Service to dispatch your Event.

```php
<?php

use Archict\Brick\Service;
use Archict\Core\EventManager;

#[Service]
final readonly class MyService
{
    public function __construct(
        private EventManager $event_manager,
    ) {}
    
    public function someMethod(): void
    {
        $this->event_manager->dispatch(new MyEvent());
    }
}
```

## How to use Bricks?

For that, let's go see [Archict/core](https://github.com/Archict/core).
