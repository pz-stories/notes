Сайт:
- Frontend
- Backend

## Сервис статистики игровых персонажей
Отвечает за получение данных о статистики персонажей с сервера и передачу в нормальном виде.
По запросу должен возвращать информацию о персонаже

## Сервис наказаний
Продвинутая система наказаний, которая подразумевает дополнительные виды наказаний, передачу их в другие системы, временные наказаний.
### Сервис внутриигровых наказаний
Так же система отвечает за выдачу/снятию/отмену наказаний.
Должна мониторить наказания выданные через игры для сохранения информации о них.


# Консоль администрации
- Поиск игроков
- Поиск аккаунтов
- Манипуляция с наказаниями
- Просмотр логов действия на сайте
- Журнал сервера (WasteLand Log Searcher)
- Управление сервером


```
Frontend -> Backend
Backend:
	Accounts API:  (работа)
	Characters API:
		GameCharacter API
		PunishmentService
	PunishmentService:
		PunishmentRepository
		PunishmentHistoryStorage
		GamePunishment API:
			GamePunishmentMonitor (backlog)
			GamePunishmentTool (backlog)
	
	GameServerManager (что-то есть)
```

# Архитектура
```
                                       [Punishment Worker] <-------+
                                                                   |
                                                                   |
     [ GameServerManager ] <---(GRPC)--->   [API GATEWAY ]         |
         |        |                     +-------------------+      |
         |        |                     | AccountsDB        |      |
         ↓        |                     | CharactersDB      |      |
      [ LGSM ]  (RCON)                  | PunishmentDB      | <----+ 
         |        |                     | PunishmentHistory |
         |        |                     +-------------------+ 
         ↓        ↓                               ↑
[Project Zomboid Server]                          |
         ↑                                        |
         |                                        |
		 |				  +-----------------------+  
		 |				  |	
		 |		    ( HTTP / WS )           
		 |		       	  ↓                 
+--------|------------------------------------------+
|        |                                          |
|	  [ Mod ]----> [GameAPIServer]                  |
|				   /      |      \                  |
|				  /       |       \                 |
| PunishmentsMonitor   Characters   ServerStateInfo |
|                                                   |
+---------------------------------------------------+
```