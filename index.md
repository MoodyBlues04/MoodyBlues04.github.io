# Введение

# GoF patterns

# Layered Architecture

# MVC

# DDD
...
**Связанные определения:**
+ *Домен* - предметная область проекта, сфера, для которой он создан. Например: e-commerce, банкинг, коммуникации и т п.

## Dto
Наиболее простой способ хранения данных. Простой объект для структуризации данных, не содержащий ни логики, ни валидации.

**Использование:**
+ передача данных между слоями (TODO ссылку на layered) или компонентами приложения
+ структуризация данных из запроса к приложению или ответа от API

**Особенности:**
+ не содержит бизнес логику (содержит только данные)
+ данные не обязательно должны быть провалидированы и иметь смысл с точки зрения бизнес-логики

**Пример:**
```php
class PersonDto
{
    public function __construct(
        public readonly string $name,
        public readonly string $phone,
        public readonly string $birthDate
    ) {
    }
}

class ApiClient
{
    // ... other methods ...

    public function getPerson(int $id): PersonDto
    {
        $response = $this->get("/api/$id");
        return new PersonDto($response['name'], $response['phone'], $response['birth_date']);
    } 
}
```
Заметьте, ни телефон, ни дата рождения не валидируются внутри dto и вполне могут иметь некорректное с точки зрения бизнеса значение.

## ValueObject

Объект, представляющий иммутабельную группу связных данных. Можно сказать, простой [Entity](#entity).

**Использование:**
+ Описание полей entity
+ Группировка связных данных и методов по работе с ними

**Особенности:**
+ Уникальность объекта определяется данными, которые он содержит, а не его идентичностью

**Пример:**

```php
class MoneyAmount
{
    public function __construct(
        private int $amount,
        private Currency $currency
    ) {
    }

    public function getInCurrency(Currency $currency): int
    {
        return $this->currency->transform($this->amount, $currency);
    }

    // ... other methods ...
}

class BankAccount
{
    public function __construct(
        private readonly int $accountId,
        private readonly User $owner,
        private readonly MoneyAmount $amount
    )
}
```
В данном примере `MoneyAmount` -- является типичным примером ValueObject-а. Это абстракция над некоторой простой сущностью (количеством денег), упрощающая и унифицирующая работу с таким типом данных. Что позволяет абстрагировать, например, `BankAccount` от логики работы с количеством денег, переводом их из одной валюты в другую и т п. 

Встроенным в `php` примером является, например, `\DateTime`.

## Entity
Паттерн группирующий некоторые связанные бизнес-данные в объект, представляющий чаще всего некоторую реальную бизнес-сущность со своим lifetime-мом. Уникальность объекта определяется его идентичностью (чаще всего идентиикатором является `primary-key`), таким образом объекты с одинаковым идентификатором будут объявлены одинаковыми независимо от их данных.

**Использование:**
+ Описание бизнес сущностей и связанных с ними логики

**Особенности:**
+ Уникальность определяется идентификатором
+ Представляют реальную бизнес-сущность

**Пример:**

```php
class Customer
{
    /**
     * @param Order[] $orders
     */
    public function __construct (
        public readonly int $id, // primary-key
        public readonly string $name,
        public readonly array $orders
    ) {
    }

    public function cancelOrder(int $orderId): void
    {
        // some business-logic
    }

    public function addOrder(Order $order): void
    {
        $this->orders[] = $order;
    }

    public function leaveFeedback(string $message, int $rating): void
    {
        // business logic
    }
}
```

Также стоит отметить, что данный паттерн тесно связан с понятием модели (из паттерна MVC) и часто модели являются entity (например `Customer` из примера выше вполне может быть реализован как представитель `ActiveRecord` TODO ссылку).

## Domain Model

Доменная модель служит механизмом инкапсуляции сложной бизнес-логики. Она соответствует конкретному объекту домена (например для банкинга это могут быть `Account`, `Customer`, `Bill` etc.) и реализует методы работы с ним.
Понятие доменной модели является одним из главных в [DDD](#ddd) (domain-driven design).

Можно сравнить этот паттерн с концепцией модели из паттерна [MVC](#mvc). Однако модель MVC часто содержит информации о способе хранения данных (или сосрадотачивается на ней, передавая управление бизнес-логикой в сервисы) - что для доменной модели не характерно.

Важно отметить, что определения доменной модели разнятся от источника к источнику. Альтернативная версия определения ([источник](https://stackoverflow.com/questions/15540147/differentiating-between-domain-model-and-entity-with-respect-to-mvc)): доменная модель - абстрактная концепция, описывающая предмет из бизнес логики. В то время как, например [entity](#entity) - ее конкретное представление в коде.

**Использование:**
+ представление сложных доменных сущностей и отражение их поведения

**Особенности:**
+ сосредотачивается на бизнес-логике: все правила, ограничения и логика, связанные с сущностью - живут в доменной модели
+ изолированность от внешней (не доменной) инфраструктуры
+ отсутствие знаний о способе хранения данных

**Пример:**

Пример из раздела про [entity](#entity) подходит и в данном случае (класс `Customer` можно рассматривать и как доменную модель). Однако доменные модели обычно больше и сложнее. Например, могут объединять логику работы с несколькими entity.

## Repository

Repository представляет собой дополнительную абстракцию между хранилищем данных и остальным приложением, инкапсулирует логику получения и изменения данных в хранилище.

Часто возникает вопрос о необходимости паттерна Repository при наличии системы [ORM](#orm), которая также является абстракцией над хранилищем данных, позволяет подменить их источник (на другую SQL базу например). Однако у Repository есть ряд *преимуществ*:
+ большая гибкость: источник данных можно подменить не только на другую базу, но и на файловое хранилище, вызов API
+ использование механизма [DI](#di) (будет разобран в примере), что упрощает написание тестов и понижает связность компонентов системы
+ полное разграничение бизнес-логики и логики получения данных (отсутствие которого при использовании ORM часто остается проблемой)

**Особенности:**
+ в соответствии с [DDD](#ddd) репозиторий создается для каждой доменной сущности (или даже сценария ее использования) отдельно
+ часто может использовать [DataMapper](#datamapper)-ы для преобразования данных из хранилища в доменные сущности

**Пример:**

```php
interface PersonRepositoryInterface
{
    public function findById(int $id): Person;
    /**
     * @return Person[]
     */
    public function findAll(): array;
    public function save(Person $person): bool;
}

readonly class MysqlPersonRepository implements PersonRepositoryInterface
{
    public function __construct(private Query $query)
    {
    }

    public function findById(int $id): Person
    {
        // ...some other data retrieving logic...

        return $this->query->where('id', $id)->first(); // Laravel QueryBuilder syntax
    }

    /**
     * @inheritDoc
     */
    public function findAll(): array
    {
        return $this->query->all();
    }

    public function save(Person $person): bool
    {
        return $person->save();
    }
}

readonly class PersonController
{
    public function __construct(private PersonRepositoryInterface $personRepository)
    {
    }

    // now we don't care about data source, just use repository to retrieve it
    // also we can easily mock this repository in our unit-tests
}

```

## DataMapper
TODO
# Dictionary

## ORM

## DI
TODO в принципы SOLID