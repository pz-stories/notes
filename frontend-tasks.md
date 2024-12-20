# Сценарии
## Регистрация
Форма:
- email
- логин
- пароль + валидация (8+ символов)

Сервер возвращает ид аккаунта

После регистрации выскакивает поп-ап "Подтвердите почту"
Тут пользователь может:
1. Отправить в окно код с почты на сервер приходит account_id, verify_code
2. Зайти на почту и перейти по ссылке. Условно `pzstories.ru/account/verify?account_id=123&code=XXXXXX`
В варианте 2 мы за него отправляем запрос на сервер на подтверждения
Далее вызов API логина. Либо пользователь заново вводит данные либо достаём их откуда-то из памяти.

## Вход 
Пользователь отправляет логин + пароль
Сервер выдаёт ему через куки сессию 
Проверяем вход через `GET api.pzstories.ru/v1/account` - выводит логин, почту.
Перенаправляем на страницу со списком персонажей

## Страница персонажей
Персонажи берутся с помощью API запроса
Формат ответа:
```
ok: true
result: список {
	id: число / Ид аккаунта
	username: Имя пользователя (Надо бы заменить на имя + фамилия игрока)
	status: LIVING | DEAD | BANNED - статус персонажа для сортировки
}
```

Делаем запрос апи на кол-во свободных мест.
И если они есть добавляем кнопку создать персонажа

## Создание персонажа
Форма:
- Пароль
- Никнейм

## Страница персонажа

```python
@dataclass  
class Character:  
    id: int  
    username: str  
    account_id: int  
  
    game_info: BaseCharacterInfo | None

class BaseCharacterInfo(ABC):  
    full_name: str  
    display_name: str  
    access_level: AccessLevel  
    last_connection: datetime | None  
  
  
@dataclass(frozen=True)  
class GameCharacterInfo(BaseCharacterInfo):  
    full_name: str  
    display_name: str  
    access_level: AccessLevel  
    last_connection: datetime | None  
    traits: list[str]  
    stats: CharacterStats  
    faction: Faction | None  
  
  
@dataclass  
class DeadCharacterInfo(BaseCharacterInfo):  
    full_name: str  
    display_name: str  
    access_level: AccessLevel  
    last_connection: datetime | None  
    dead_at: datetime
     
  
@dataclass(frozen=True)  
class CharacterStats:  
    weight: float  
    health: float  
    zombie_kills: int  
    players_kills: int  
    hours_survived: int  
    perks: list[Perk]  
  
  

@dataclass(frozen=True)  
class Faction:  
    name: str  
    owner: str  
    tag: str  


class AccessLevel(StrEnum):  
    PLAYER = "player"  
    OVERSEER = "overseer"  
    OBSERVER = "observer"  
    MODERATOR = "moderator"  
    ADMIN = "admin"  
  
  
@dataclass(frozen=True)  
class Perk:  
    name: str  
    level: int  

```