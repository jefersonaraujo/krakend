### Clonando o repositório do ORY Hydra
```bash
git clone https://github.com/ory/hydra.git
cd hydra
```

### Subindo os containers com Docker Compose
```bash
docker compose -f quickstart.yml \
    -f quickstart-jwt.yml \
    up \
    --build
```

### Criando um cliente no Hydra
```bash
docker compose -f quickstart.yml exec hydra \
    hydra create client \
    --endpoint http://127.0.0.1:4445/ \
    --format json \
    --id fluid --secret batata \
    --grant-type client_credentials \
    --scope "read" \
    --access-token-strategy jwt
```

### Exemplo de resposta
```json
{
  "access_token_strategy": "jwt",
  "client_id": "fluid",
  "client_secret": "batata",
  "grant_types": ["client_credentials"],
  "scope": "read",
  "token_endpoint_auth_method": "client_secret_basic"
}
```

### Obtendo um token de acesso
```bash
curl -s -k -X POST \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d grant_type=client_credentials \
    -u 'fluid:batata' \
    http://localhost:4444/oauth2/token
```

### Exemplo de resposta com token
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjhiN...",
  "expires_in": 3599,
  "scope": "",
  "token_type": "bearer"
}
```

## Configuração do KrakenD

### Subindo o KrakenD
Troque o IP `10.1.5.14` pelo IP correto da sua máquina ou configure o Docker para rodar em um serviço adequado.
```bash
docker run -p "8080:8080" --rm \
    -v $PWD:/etc/krakend/ \
    devopsfaith/krakend:watch run -c krakend.json
```

### Exemplo de configuração do `krakend.json`
```json
{
  "backend": [
    {
      "url_pattern": "/api",
      "encoding": "json",
      "sd": "static",
      "method": "GET",
      "host": [
        "https://randomuser.me/"
      ],
      "disable_host_sanitize": false
    }
  ],
  "extra_config": {
    "auth/validator": {
      "alg": "RS256",
      "jwk_url": "http://10.1.5.14:4444/.well-known/jwks.json",
      "scopes": ["read"],
      "scopes_key": "scp",
      "disable_jwk_security": true,
      "operation_debug": true,
      "cache": false
    }
  }
}
```

