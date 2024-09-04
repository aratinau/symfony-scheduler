# Symfony Scheduler

## Getting Started

1. If not already done, [install Docker Compose](https://docs.docker.com/compose/install/) (v2.10+)
2. Run `docker compose build --no-cache` to build fresh images
3. Run `docker compose up --pull always -d --wait` to set up and start a fresh Symfony project
4. Open `https://localhost` in your favorite web browser and [accept the auto-generated TLS certificate](https://stackoverflow.com/a/15076602/1352334)
5. Run `docker compose down --remove-orphans` to stop the Docker containers.

## LogHello

    docker compose exec php php bin/console messenger:consume -v scheduler_default

```php
<?php
// src/Message/LogHello.php

namespace App\Message;

final class LogHello
{
    /*
     * Add whatever properties and methods you need
     * to hold the data for this message class.
     */

    public function __construct(
        public int $length
    ) {
    }
}
```

### LogHelloHandler

```php
<?php
// src/MessageHandler/LogHelloHandler.php

namespace App\MessageHandler;

use App\Message\LogHello;
use Psr\Log\LoggerInterface;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;

#[AsMessageHandler]
final class LogHelloHandler
{
    public function __construct(
        private LoggerInterface $logger
    ) {
    }

    public function __invoke(LogHello $message): void
    {
        $this->logger->warning(str_repeat('🎸', $message->length).' '.$message->length);
    }
}
```

### MainSchedule

```php
<?php
// src/Scheduler/MainSchedule.php

namespace App\Scheduler;

use App\Message\LogHello;
use Symfony\Component\Scheduler\Attribute\AsSchedule;
use Symfony\Component\Scheduler\RecurringMessage;
use Symfony\Component\Scheduler\Schedule;
use Symfony\Component\Scheduler\ScheduleProviderInterface;

#[AsSchedule]
class MainSchedule implements ScheduleProviderInterface
{
    public function getSchedule(): Schedule
    {
        return (new Schedule())->add(
            RecurringMessage::every('4 seconds', new LogHello(4)),
            RecurringMessage::every('3 seconds', new LogHello(3)),
        );
    }
}
```
