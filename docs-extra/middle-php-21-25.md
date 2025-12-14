---
### 21. Зачем нужно ключевое слово final?

<details>
<summary>Раскрыть:</summary>

`final` в PHP используется, чтобы **запретить наследование** от класса или **переопределение** конкретного метода.

#### 1. `final class` — запрет наследования
```php
final class Money {
    public function __construct(public int $amount) {}
}
```
* Никто не сможет сделать `class MyMoney extends Money {}`.
* Полезно, когда класс — законченная единица, которую нельзя безопасно расширять.

#### 2. `final method` — запрет переопределения метода
```php
class BaseService {
    final public function execute(): void {
        $this->validate();
        $this->handle();
    }

    protected function validate(): void {}
    protected function handle(): void {}
}
```
* `execute()` нельзя переопределить, но можно переопределять “хуки” (`validate`, `handle`).
* Это часто используется в паттерне **Template Method**.

#### 3. Зачем применять `final`
* **Безопасность инвариантов**: запрещаем ломать правила домена через наследование.
* **Ясный API**: класс/метод “не для расширения”.
* **Проще поддерживать**: меньше неожиданных наследников и side effects.
* **Оптимизации**: теоретически рантайм может оптимизировать вызовы, но обычно это не главный мотив.

#### 4. Когда не стоит злоупотреблять
* Если библиотека/фреймворк предполагает расширяемость через наследование.
* Если проект использует наследование как основной механизм расширения (хотя предпочтительнее композиция).

#### 5. Практическая рекомендация
В современных проектах часто:
* доменные value-objects/DTO делают `final`
* сервисы и хендлеры тоже часто `final`
* расширяемость достигается через **интерфейсы + DI** вместо наследования

</details>

---
### 22. Что нового в PHP 7 и PHP 8?

<details>
<summary>Раскрыть:</summary>

Ниже — ключевые изменения, которые часто спрашивают на собеседованиях (без мелких нюансов).

#### PHP 7 (7.0–7.4): основные изменения
##### 1. Производительность
* Существенный прирост скорости и снижение потребления памяти (Zend Engine 3).

##### 2. Типизация
* Скаляры: `int`, `float`, `string`, `bool` (type hints).
* Возвращаемые типы (`: string`, `: int`).
* `declare(strict_types=1)` — строгая типизация параметров/возвратов.

##### 3. Null Coalescing Operator
```php
$val = $arr['key'] ?? 'default';
```

##### 4. Spaceship operator
```php
$a <=> $b; // -1, 0, 1
```

##### 5. Anonymous classes
```php
$obj = new class { public function hi() { return 'hi'; } };
```

##### 6. Throwable
* Ошибки и исключения унифицированы через `Throwable`.

##### 7. PHP 7.4 (часто важное)
* Typed properties:
```php
public int $id;
```
* Arrow functions:
```php
fn($x) => $x * 2
```
* Preloading (OPcache):
  * позволяет загрузить классы заранее (полезно в больших приложениях)

---

#### PHP 8 (8.0+): основные изменения
##### 1. Named arguments
```php
foo(b: 2, a: 1);
```

##### 2. Attributes
```php
#[Route('/api')]
public function index() {}
```

##### 3. Union types
```php
function f(int|string $x): int|string {}
```

##### 4. Match expression
```php
$res = match($type) {
  'a' => 1,
  'b' => 2,
  default => 0,
};
```

##### 5. Nullsafe operator
```php
$name = $user?->profile?->name;
```

##### 6. Constructor property promotion
```php
public function __construct(
  private int $id,
  private string $email
) {}
```

##### 7. JIT (через OPcache)
* может ускорять CPU-bound задачи, но обычно умеренно влияет на веб.

##### 8. Более строгие ошибки
* много warning/notice стали исключениями/фаталами → лучше качество кода, но возможны breaking changes.

---

#### Как правильно отвечать на собесе
* PHP 7: производительность + типизация.
* PHP 8: удобные фичи (attributes/match/union/nullsafe) + строгий рантайм.
* Для веба чаще важнее архитектура, БД/IO и OPcache, чем JIT.

</details>

---
### 23. SOLID, DRY, KISS, YAGNI.

<details>
<summary>Раскрыть:</summary>

Это базовые принципы проектирования кода, которые помогают писать поддерживаемые системы.

#### 1. SOLID

##### S — Single Responsibility Principle
* Класс имеет одну ответственность и одну причину для изменений.

##### O — Open/Closed Principle
* Расширяем функциональность без изменения существующего кода (через интерфейсы/стратегии/декораторы).

##### L — Liskov Substitution Principle
* Наследники должны быть взаимозаменяемы без нарушения контрактов.

##### I — Interface Segregation Principle
* Много маленьких интерфейсов лучше, чем один огромный.

##### D — Dependency Inversion Principle
* Зависим от абстракций, а не от конкретных классов (DI, интерфейсы).

---

#### 2. DRY (Don’t Repeat Yourself)
* Не повторяем бизнес-правила/знания в разных местах.
* Важно не “вынести всё в абстракции”, а удерживать баланс.

---

#### 3. KISS (Keep It Simple)
* Решение должно быть максимально простым, без лишней магии.

---

#### 4. YAGNI (You Aren’t Gonna Need It)
* Не добавляем функциональность “на будущее”, если её нет в требованиях сейчас.

---

#### 5. Практика
* SOLID помогает строить слои (Controller → Service/UseCase → Repository).
* DRY снижает дублирование бизнес-логики.
* KISS/YAGNI защищают от оверинжиниринга.

</details>

---
### 24. Паттерны проектирования, с которыми приходилось работать.

<details>
<summary>Раскрыть:</summary>

На собеседовании лучше перечислять паттерны через “проблема → решение → где применял”.

#### 1. Creational
* **Factory / Abstract Factory** — создание объектов без привязки к конкретным классам.
* **Builder** — сборка сложных объектов/DTO шаг за шагом.
* **Prototype** — создание через клонирование.
* **Singleton** — встречается, но часто считают антипаттерном в приложениях.

#### 2. Structural
* **Adapter** — адаптация внешнего API/SDK к внутреннему интерфейсу.
* **Decorator** — добавление поведения (кеш/логирование) без наследования.
* **Facade** — простой интерфейс к сложной подсистеме.
* **Proxy** — проксирование вызовов (кеш, lazy, remote calls).

#### 3. Behavioral
* **Strategy** — выбор алгоритма (комиссии, правила).
* **Observer / Event Dispatcher** — события (Symfony events, Doctrine subscribers).
* **Command** — действие как объект (handlers, async jobs).
* **Chain of Responsibility** — цепочка обработчиков (middleware).
* **Template Method** — общий алгоритм + переопределяемые шаги.

#### 4. Примеры “из жизни”
* Middleware (PSR-15) → Chain of Responsibility.
* CacheTagHelper/кеширование поверх репозитория → Decorator/Proxy.
* Разные способы расчёта комиссии → Strategy.

</details>

---
### 25. Что такое простая фабрика?

<details>
<summary>Раскрыть:</summary>

**Простая фабрика (Simple Factory)** — практический приём: класс/метод, который **создаёт нужную реализацию** по параметру.

#### 1. Зачем нужна
* убирает `new` из бизнес-логики
* централизует выбор реализаций
* упрощает тестирование и поддержку

#### 2. Пример
```php
interface NotifierInterface {
    public function send(string $msg): void;
}

final class EmailNotifier implements NotifierInterface {
    public function send(string $msg): void {}
}

final class SmsNotifier implements NotifierInterface {
    public function send(string $msg): void {}
}

final class NotifierFactory {
    public static function make(string $type): NotifierInterface
    {
        return match ($type) {
            'email' => new EmailNotifier(),
            'sms'   => new SmsNotifier(),
            default => throw new InvalidArgumentException('Unknown notifier type'),
        };
    }
}
```

#### 3. Плюсы
* простота
* быстро внедряется
* уменьшает дублирование создания объектов

#### 4. Минусы
* фабрика может вырасти в большой `switch/match` (нарушение OCP)
* при росте вариантов лучше:
  * DI container + autowiring
  * registry стратегий
  * factory method / abstract factory

#### 5. “Middle” формулировка
* “Simple Factory — это централизованное создание объектов по входному параметру. Удобно на старте, но при расширении лучше уходить в DI/стратегии, чтобы не разрастался switch.”

</details>

---
