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
+ идентичность основана на данных (два dto считаются одинаковыми, если содержат одинаковые данные)

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

Простой объект служащий чаще всего для описания поля или группы полей (уточнить)

## Entity


## Domain Model