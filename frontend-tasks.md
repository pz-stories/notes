# Хосты
Сайт: `pzstories.ru`
API: `api.pzstories.ru/v1`

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

Если пользователь попробует залогиниться то показываем ему тоже просто поп-ап

## Кнопка отправить заново
Появляется в поп-апе. В запросе требуется account_id и пароль.

## Вход 
Пользователь отправляет логин + пароль
Сервер выдаёт ему через куки сессию 
Проверяем вход через `GET api.pzstories.ru/v1/account` - выводит логин, почту.
Перенаправляем на страницу со списком персонажей

Если у пользователя не подтверждена почта, то сервер отправляет в ответе ид аккаунта для возможности отправки нового сообщения.

Но пользователь не будет авторизован пока не подтвердит почту.

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
<details>
<summary>Структура</summary>

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
</details>

На наличию поля `dead_at` определяем умер ли игрок и показываем соответственный ответ

На странице должны быть кнопки:
- Удалить персонажа
- Поменять пароль

## Страница настроек аккаунта
Пока только изменить пароль

## Восстановление пароля
Пользователь вводит email. 
На почту отправляется ссылка вида `pzstories.ru/reset-password?access_token={jwt_token}`

При переходе на ссылку открывается страница
Где пользователь вводит новый пароль и подтверждает его
Далее отправляется запрос на `PATCH api.pzstories.ru/v1/account/reset-password`
В теле передаётся новый пароль
В заголовках `Authorization: Bearer {jwt_token}`
После успешного восстановления пользователю сразу выдаётся токен и он входит в систему

## Обработка ошибок
В общем случае непредвиденные ошибки должны выводиться как поп-ап или в виде уведомления.

В случае когда ошибка может быть связана с логикой работы - зависит от контекста.

Большинство ошибок модульные и используются в нескольких местах, то есть способ их обработки должен зависеть от того что делает пользователь.
Ошибки в следующем формате:
```json
{
	"ok": false, // показатель, что результат не успешный
	"error": {
		// Можно использовать для определения типа ошибки
		// Для каждой ошибки уникален
		"code": "КОД_ОШИБКИ",
		"title": "Читаемое описание ошибки. В теории можно его выводить пользователю, но нет гаранитий что тут безопасное содержание. пока что оно на английском",
		"status": 404, // HTTP Код, дубликат кода ответа
		"data": {
			"поле": "Здесь может быть данные о ошибке, а может и не быть",
			// Пример. Основываясь на коде ошибки можно использовать это поле при 
			// отображении данных пользователю.
			"username": "ukiru",
		}
	}
}
```

Пример при неправильно вводе данных для входа
```json
{
	"ok": false,
	"error": {
		"code": "INVALID_CREDENTIALS",
		"title": "Invalid credentials provided",
		"status": 401,
		"data": {}
	}
}
```