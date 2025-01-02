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
			"email", "totp"
		],
		"preferred": "totp"
	}
}

```