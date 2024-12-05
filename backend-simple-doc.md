# Структура
`HTTP_METHOD ENDPOINT`
```
example body
```
<- RESPONSE_CODE
```
example response
```
# Регистрация
`POST /v1/account/register`
```json
{
	"email": "email@at.com",
	"password": "8-72 symbols"
}
```
<- 202
```json
{
	"ok": true,
	"result": {
		"next-challenge": "VERIFY_EMAIL",
		"args": {
			"user_id": 1011
		}
	}
}
```

`POST /v1/account/verify-email
```json
{
	"user_id": 1011,
	"code": "secret_code",
}
```
<- 201 + Cookie SessionId
```json
{
	"ok": true,
	"resul": {...} // UserObject
}
```

# Вход
`POST /v1/auth/login`
```json
{
	"email": "my.email@site.com",
	"password": "password",
}
```
## Без 2ФА
<- 200
```json
{
	"ok": true,
	"result": {
		// UserObject
	}
}
```
## 2ФА
### Включен только один способ
<-202
```json
{
	"ok": true,
}
```
## Несколько способов
<-202
```json
{
	"ok": true,
	"result": {
		"next-challenge": "2fa.method",
		"args": {
			"methods": [
					"email",
					"totp",
					"telegram"
			],
			"key": "rand-hex"
		}
	}
}
```

### Ответ серверу
`POST /v1/auth/2fa/use/{method}`
```
Headers:
	Authorization: Bearer {token}
```
<- 202
```json
{
	"ok": true,
	"result": {
		"method": "email",
		"timeout": "5m",
	}
}
```

## Ответ серверу
`POST /v1/auth/2fa/{method}`
```
Headers:
	Authorization: Bearer {token}
```

```json
{
	"code": "code from 2fa"
}
```
<- 200 + Cookie: SessionId
```json
{
	"ok": true,
	"result": {...} // UserObject
}
```

