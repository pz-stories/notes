# pz-stories-notes



```
POST api.pzstories.ru/v1/auth/login
{
	"email": "user@example.com",
	"password": "my_password"
}
```

```json
{
	"ok": true,
	"result": {
		"type": "mfa_required",
		"methods": [
			{
				"name": "email",
				"need_request": true,
			},
			{
				"name": "totp",
				"need_request": false,
			},
			{
				"name": "telegram",
				"need_request": true,
			}
		],
		"preferred": "totp"
	},
	"_http_response": {
		"cookie": {
			"ticket": {
				"account_id": 1,
				"scope": "login_2fa"
			}
		}
	}
}
```

```
GET api.pzstories/auth/2fa/email

204 No Content
```

```
POST api.pzstories/auth/login?
Cookie: ticket:fdhdjdj.djdkdsk

{
	"email": "user@example.com",
	"password": "my_password",
	"2fa": {
		"code": "123456"
		"method": "method"
	}
}
```

```json
{
	"http.request": {
		"cookie": "access_token",
		
	} 
}
```


```python

MFA = {
	   before,
	   between,
}

MFA.before


```