# Введение

# Паттерны банды 4-х

# Layered Architecture

# DDD

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


## Domain Model