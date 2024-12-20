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
При запросе данных о игроке получаем ответ типа `Character`

```python
# @dataclass - игнорируй, ничего не значит
# класс - описание структуры
# поле: тип
# str - строка, int - число, float - дробь, None - ничего (null)
# datetime - время (будет в виде строки на самом деле)
# list[str] - массив строк
# Тип | Тип2 - Один из типо (Или)


@dataclass  
class Character:  
    id: int  
    username: str  
    account_id: int  
	# Здесь будет тип GameCharacterInfo или DeadCharacterInfo
    game_info: BaseCharacterInfo | None

class BaseCharacterInfo(ABC):  
    full_name: str  
    display_name: str  # Возвможно уйдет в будуем
    access_level: AccessLevel  
    last_connection: datetime | None  
  
  
@dataclass(frozen=True)  
class GameCharacterInfo(BaseCharacterInfo):  
	"""Информация о живом персонаже"""
	
    full_name: str  
    display_name: str  
    access_level: AccessLevel  
    last_connection: datetime | None  
    traits: list[str]  
    stats: CharacterStats  
    faction: Faction | None  
  
  
@dataclass  
class DeadCharacterInfo(BaseCharacterInfo):  
	"""Информация о мертвом персонаже"""
    full_name: str  
    display_name: str  
    access_level: AccessLevel  
    last_connection: datetime | None  
    dead_at: datetime
     
  
@dataclass(frozen=True)  
class CharacterStats:  
	"""Статистика персонажа"""
    weight: float  
    health: float  
    zombie_kills: int  
    players_kills: int  
    hours_survived: int  
    perks: list[Perk]  
  
  

@dataclass(frozen=True)  
class Faction:  
	"""
	Фракция
	"""
    name: str  
    owner: str  
    tag: str  



class AccessLevel(StrEnum):  
	"""
	Это просто перечисление уровней доступа чтобы было удобнее
	"""
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