---
### 11. Клонирование объектов и вложенные объекты (shallow / deep clone).

<details>
<summary>Раскрыть:</summary>

В PHP оператор `clone` создаёт **копию объекта**, но важно понимать, что это копирование **по умолчанию поверхностное (shallow)**:  
сам объект копируется, а **вложенные объекты внутри свойств** продолжают ссылаться на те же экземпляры, пока вы не выполните “глубокое” копирование вручную.

#### 1. Shallow clone (поверхностное клонирование)
* Создаётся новый объект (новый идентификатор).
* **Скалярные** свойства копируются как значения.
* **Объекты внутри свойств** остаются теми же (копируются ссылки на них).

Пример:
```php
final class Address {
    public function __construct(public string $city) {}
}

final class UserProfile {
    public function __construct(
        public string $name,
        public Address $address
    ) {}
}

$u1 = new UserProfile('Ihor', new Address('Warsaw'));
$u2 = clone $u1;

$u2->name = 'Max';
$u2->address->city = 'Krakow';

echo $u1->name;            // Ihor (скаляр отдельно)
echo $u1->address->city;   // Krakow (вложенный объект общий!)
```

#### 2. Deep clone (глубокое клонирование)
Чтобы клон был “независимым”, нужно вручную клонировать вложенные объекты — обычно через `__clone()`.

```php
final class UserProfile {
    public function __construct(
        public string $name,
        public Address $address
    ) {}

    public function __clone(): void
    {
        $this->address = clone $this->address; // deep для вложенного объекта
    }
}
```

#### 3. Что происходит при `clone`
* Вызывается “внутренний” механизм копирования объекта.
* Затем, если определён метод `__clone()`, он вызывается **на новом объекте**.

#### 4. Частые нюансы на практике
* **ORM (Doctrine)**: клонирование Entity может быть опасным (id, UnitOfWork, коллекции). Обычно делают отдельные DTO/Factory для “копирования”.
* **Коллекции/массивы с объектами**: массив копируется, но элементы-объекты остаются общими → нужен deep clone для каждого элемента.
* `DateTimeImmutable` безопаснее для “шеринга”, чем `DateTime`, потому что он неизменяемый.

#### 5. Когда использовать
* Прототипирование объектов (паттерн Prototype).
* Подготовка “черновика” данных перед изменениями.
* Копирование конфигураций/настроек (при условии корректного deep clone).

</details>

---
### 12. Что такое Mock? Где используется и зачем?

<details>
<summary>Раскрыть:</summary>

**Mock (мок)** — это тестовый двойник (test double), который:
* имитирует зависимость (например, репозиторий, HTTP-клиент),
* позволяет **задавать ожидания**: какие методы должны быть вызваны, сколько раз и с какими аргументами.

#### 1. Зачем нужен Mock
* Изолировать unit-тест от внешних зависимостей: БД, сеть, очередь, время.
* Проверить **поведение** (behavior): “вызвали ли мы нужный метод?”
* Сделать тесты быстрыми, стабильными и предсказуемыми.

#### 2. Mock vs Stub (важное отличие)
* **Stub** — возвращает заранее подготовленные данные, без проверки “как его вызывали”.
* **Mock** — кроме возврата данных, ещё и **проверяет взаимодействие** (expectations).

#### 3. Пример в PHPUnit (простая идея)
```php
$repo = $this->createMock(UserRepository::class);

$repo->expects($this->once())
    ->method('save')
    ->with($this->isInstanceOf(User::class));

$service = new UserService($repo);
$service->register('test@example.com');
```

#### 4. Где обычно используют
* Unit-тесты сервисов (Service Layer, UseCase/Handler).
* Тестирование контроллеров (реже — если это не интеграционный тест).
* Тестирование кода, который дергает внешние API (HTTP client) или очередь.

#### 5. Типичные ошибки
* Слишком много моков → тест становится “тестом реализации”, ломается при рефакторинге.
* Мокать “свою” бизнес-логику вместо границ системы — обычно мокают **инфраструктуру** (репозитории, клиенты, логгеры).
* Не различать unit и интеграционные тесты.

</details>

---
### 13. Что такое PSR?

<details>
<summary>Раскрыть:</summary>

**PSR (PHP Standards Recommendations)** — набор рекомендаций/стандартов, публикуемых группой **PHP-FIG**, чтобы разные библиотеки и фреймворки были совместимы по интерфейсам и стилю.

#### 1. Зачем это нужно
* Унификация кода в проектах и командах.
* Совместимость компонентов (например, любой PSR-3 Logger можно подменить другим).
* Упрощение интеграции пакетов из Composer-экосистемы.

#### 2. Наиболее важные PSR (то, что часто спрашивают)
* **PSR-4** — автозагрузка классов (namespace → путь).
* **PSR-3** — интерфейс логгера (`LoggerInterface`).
* **PSR-7** — HTTP сообщения (Request/Response/Stream).
* **PSR-11** — контейнер зависимостей (`ContainerInterface`).
* **PSR-15** — HTTP middleware (request handler + middleware).
* **PSR-18** — HTTP client.
* **PSR-6 / PSR-16** — кеширование (pool/simple-cache).
* **PSR-12** — код-стайл (расширение PSR-2).

#### 3. Пример: PSR-3 Logger
```php
use Psr\Log\LoggerInterface;

final class PaymentService {
    public function __construct(private LoggerInterface $logger) {}

    public function pay(): void {
        $this->logger->info('Payment started');
    }
}
```
Тут важно: сервису всё равно, какой именно логгер внутри (Monolog, кастомный и т.д.) — главное, что он PSR-3.

#### 4. Как это проявляется в Symfony
* Многие компоненты Symfony “наружу” дают PSR-интерфейсы (логгер, кеш, контейнер, HTTP).
* Это облегчает замену реализации без переписывания кода.

</details>

---
### 14. Опишите реализацию одного из паттернов проектирования.

<details>
<summary>Раскрыть:</summary>

Опишем **Strategy (Стратегия)** — паттерн, который позволяет **заменять алгоритм поведения** во время выполнения через общий интерфейс.

#### 1. Идея паттерна
* Есть “контекст”, которому нужно выполнить действие (например, рассчитать комиссию).
* Алгоритмы разные (для разных провайдеров/стран/режимов).
* Мы выносим алгоритмы в отдельные классы-стратегии.

#### 2. Пример (PHP)
```php
interface FeeStrategyInterface {
    public function calculate(int $amount): int;
}

final class FixedFeeStrategy implements FeeStrategyInterface {
    public function __construct(private int $fee) {}
    public function calculate(int $amount): int { return $amount + $this->fee; }
}

final class PercentFeeStrategy implements FeeStrategyInterface {
    public function __construct(private int $percent) {}
    public function calculate(int $amount): int {
        return $amount + (int) round($amount * $this->percent / 100);
    }
}

final class PaymentCalculator {
    public function __construct(private FeeStrategyInterface $strategy) {}

    public function total(int $amount): int {
        return $this->strategy->calculate($amount);
    }
}
```

#### 3. Что даёт Strategy
* Убирает большие `if/else` или `switch` по типам.
* Упрощает расширение: добавили новую стратегию — не ломаем старые.
* Легче тестировать: каждая стратегия тестируется отдельно.

#### 4. Где часто используют в реальных проектах
* Тарифы/комиссии, расчёты, валидации по разным правилам.
* Выбор провайдера/интеграции (Stripe/Adyen/PayPal).
* В Symfony удобно через DI + теги/Registry (собираем набор стратегий).

</details>

---
### 15. Что такое Redis?

<details>
<summary>Раскрыть:</summary>

**Redis** — высокопроизводительное in-memory хранилище данных (key-value), которое поддерживает разные структуры данных и часто используется как кеш, брокер событий и быстрое хранилище.

#### 1. Ключевые особенности
* Данные хранятся в памяти → очень быстро.
* Поддерживает структуры:
  * Strings
  * Hashes
  * Lists
  * Sets / Sorted Sets
  * Streams
* Может сохранять данные на диск (persistency):
  * **RDB** (снимки)
  * **AOF** (журнал операций)

#### 2. Типовые кейсы использования
* **Кеширование** (страницы/запросы/DTO/флаги)
* **Сессии** (особенно при масштабировании)
* **Rate limiting** (счётчики запросов)
* **Distributed locks** (защита от параллельных запусков)
* **Очереди / стримы** (Streams, списки)
* **Pub/Sub** (уведомления)

#### 3. Важные концепции для продакшна
* **TTL/expiration** — время жизни ключей.
* **Eviction policy** — что делать при нехватке памяти (LRU/LFU и т.д.).
* **Replication / Sentinel / Cluster** — отказоустойчивость и масштабирование.
* **Проблемы “кеш-штампа”** — когда много запросов одновременно пытаются пересоздать истёкший кеш (решают locks, jitter TTL, soft TTL).

#### 4. Пример: простое кеширование (идея)
```php
// Псевдо-логика: положить значение с TTL
SET user:123 "{...json...}" EX 300
```

#### 5. Redis vs Memcached (коротко)
* Redis богаче по структурам и возможностям (locks, streams, persistence).
* Memcached проще и очень быстрый для простого кеша, но функциональность меньше.

</details>

---
