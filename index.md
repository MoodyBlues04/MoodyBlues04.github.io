# Введение

# GoF patterns

# Layered Architecture

# MVC

# Design Principles
## SOLID
## YAGNI
## DRY

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

Repository представляет собой абстракцию между хранилищем данных и доменной логикой, инкапсулирует логику получения и изменения данных в хранилище. В соответствии с [определением Мартина Фаулера](https://martinfowler.com/eaaCatalog/repository.html) Repository позволяет работать с хранилищем как с простой in-memory коллекцией, предоставляя интерфейс подобный приведенному ниже:
```php
interface RepositoryInterface
{
    public function find(Specification $specification): Item;
    public function create(Item $item): bool;
    public function update(Item $item): bool;
}
```

На практике часто для основных сценариев работы создаются отдельные repository, где используются конкретные методы получения сущностей вместо использования паттерна [Specification](TODO link) (например `findByUsername($username)`, `findByCredentials($email, $parrsword)` и т п).

**Особенности:**
+ репозиторий работает именно с доменными сущностями. Базовый интерфейс располагается на доменном уровне. Конкретные реализации - на инфраструктурном
+ в соответствии с [DDD](#ddd) репозиторий создается для каждой доменной сущности (или даже сценария ее использования) отдельно
+ часто может использовать [DataMapper](#datamapper)-ы для преобразования данных из хранилища в доменные сущности
+ позволяет очень гибко подменять источники данных (не только переключаться с SQL-базы на другую, но и использовать, например, файловое хранилище)

**Использование:**
+ абстракция между инфраструктурным и доменным уровнями для получения и работы с доменными сущностями
+ предоставление интерфейса для работы с объектами домена, как с коллекцией (не всегда на практике используется в таком ключе)
 
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

## DAO
TODO DAO и ORM не из DDD

DAO (Data Access Object), в отличие от [Repository](#repository) работает не с доменными сущностями, а с конкретной технологией (SQL db, файловое хранилище и т п). В DAO содержится вся логика получения/обновления данных сущности (таблицы/файла и т п). Часто DAO используется в реализации конкретного экземпляра repository. [Источник.](https://softwareengineering.stackexchange.com/questions/438348/the-difference-between-repository-and-dao)

**Особенности:**
+ работа на уровне хранилища данных, с конкретной технологией
+ в соответствии с DDD располагается на инфраструктурном уровне

**Использование:**
+ абстракция над хранилищем данных, инкапсулирующая логику работы с хранилищем

**Пример:**
```php

/**
 * Infrastructure level dto.
 */
readonly class MysqlPerson
{
    public function __construct(
        public int $id,
        public string  $name,
        public int $age
    ) {
    }
}

readonly class MysqlPersonDao
{
    public function __construct(
        private Connection $connection
    ) {
    }

    /**
     * @return MysqlPerson[]
     */
    public function selectAll(): array
    {
        $rows = $this->connection
            ->executeQuery("SELECT * from persons")
            ->fetchAllAssociative();
        return array_map(fn ($row) => $this->personFromArray($row), $rows);
    }

    // other retriewe/update data logic
}
```

## ORM

ORM (Object relation mapping) - паттерн, который позволяет взаимодействовать с реляцинными базами данных как с объектами вашего приложения.

Стоит отметить, что многими представителями сообщества [ORM считается анти-паттерном](https://habr.com/ru/articles/667078/). Это частично верно, у ORM есть существенные недостатки. Например, ORM-модели часто совмещают в себе работу с хранилищем данных и бизнес-логику, что противоречит [DRY](#dry). Однако, это не значит, что ORM не стоит использовать - важно делать это с пониманием.    

**Использование:**
+ ORM создает связь таблиц БД и объектов приложения (обычно это модели)
+ ORM предоставляет абстракцию над SQL-запросами, заменяя их методами QueryBuilder-а. Например: `ModelClass::query()->select('name')->where('id', $id)->get()` (синтаксис `Eloquent`)
+ использование ORM способно существенно ускорить разработку засчет дополнительной абстракции над БД и отсутствия необходимости преобразования данных вручную

**Примеры:**
+ [Doctrine](https://www.doctrine-project.org/) - используется в Symfony
+ [Eloquent](https://laravel.su/docs/12.x/eloquent) - наследник Doctrine, используется в Laravel
+ [Hibernate](https://hibernate.org/) - java-фреймворк, прородитель систем ORM
  
Пример работы с `Eloquent`:
```php
    /**
     * @property int $id
     * @property string $name
     * @property int $age
     * @property-read Collection $orders
     */
    class Person extends Model
    {
        protected $fillable = [
            'name',
            'age',
        ];

        public function orders(): HasMany
        {
            return $this->hasMany(Order::class);
        }
        
        // ... other model settings ...
    }

    class PersonController
    {
        public function search(Request $request)
        {
            $persons = Person::query()
                ->where('name', 'like', '%' . $request->get('name') . '%')
                ->whereBetween('age', [$request->get('age_from'), $request->get('age_to')])
                ->all();
            return $this->response($persons);
        }
    }
```

## Сравнение Dao/Orm/Repository
TODO дописать

Часто возникает вопрос о необходимости паттерна Repository при наличии системы [ORM](#orm), которая также является абстракцией над хранилищем данных, позволяет подменить их источник (на другую SQL базу например). Однако у Repository есть ряд *преимуществ*:
+ большая гибкость: источник данных можно подменить не только на другую базу, но и на файловое хранилище, вызов API
+ использование механизма [DI](#di) (будет разобран в примере), что упрощает написание тестов и понижает связность компонентов системы
+ полное разграничение бизнес-логики и логики получения данных (отсутствие которого при использовании ORM часто остается проблемой)

[DAO vs Repository](https://dzone.com/articles/differences-between-repository-and-dao)
[Repository disadvantages](https://ayende.com/blog/3955/repository-is-the-new-singleton)

## Identity Map
[Identity Map](https://martinfowler.com/eaaCatalog/identityMap.html) - механизм, гарантирующий, что каждый загруженный (обычно из БД) объект будет загружен лишь единожды. Identity Map часто является составляющей ORM и представляет собой `Map<PrimaryKey, Model>`.

**Преимущества:**
+ гарантия уникальности объектов позволяет клиенту рассчитывать, что его данные всегда соответствуют актуальным данным БД
+ улучшение производительности благодаря отсутствию повторных запросов к БД за теми же данными

## DataMapper
[DataMapper](https://martinfowler.com/eaaCatalog/dataMapper.html) отвечает за преобразование persistent-object (сущности из хранилища данных) в объект доменной области. Использование этого паттерна позволяет отделить доменную модель от ее способа представления в хранилище.
Например, такой подход решает проблему ORM моделей, которые являются persistance-объектами, но содержат бизнес-логику.

**Использование:**
+ абстракция между хранилищем и доменной моделью. Является прослойкой между уровнями infrastructure и domain (в терминологии DDD)
+ составление сложных [Entity](#entity) из разрозненных данных (таблиц БД)

**Примеры:**

*Замечание:* `MysqlPersonDao` из предыдущего [примера](#dao), если бы возвращало entity - являлось бы в том числе DataMapper-ом, так как из представления данных на уровне БД в DAO формировался доменный объект. [Другой подобный пример](https://designpatternsphp.readthedocs.io/en/latest/Structural/DataMapper/README.html)

```php

class MysqlPersonMapper
{
    public function fromMysqlPerson(MysqlPersonDto $mysqlPerson): PersonEntity
    {
        $personEntity = new PersonEntity($mysqlPerson->id);
        // ... mapping logic ...
        return $personEntity;
    }
    
    public function toMysqlPerson(PersonEntity $personEntity): MysqlPersonDto
    {
        return new MysqlPersonDto(
            $personEntity->getId(),
            $personEntity->getName(),
            // ... other mapping logic ...
        );
    }
}

```

## Usecase

## Transaction Script

## Command/Query


# Dictionary



## DI (dependency injection)