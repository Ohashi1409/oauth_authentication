# Guia Rápido de Referência OAuth 2.0

Este é um guia de referência rápida para conceitos essenciais do OAuth 2.0.

## Conceitos Básicos

| Termo | Descrição |
|-------|-----------|
| **Resource Owner** | Usuário que possui os dados e concede acesso |
| **Client** | Aplicação que deseja acessar os recursos |
| **Authorization Server** | Servidor que emite tokens após autenticação |
| **Resource Server** | Servidor que hospeda os recursos protegidos |
| **Access Token** | Token usado para acessar recursos protegidos |
| **Refresh Token** | Token usado para renovar access tokens |

## Fluxos Principais

### 1. Authorization Code Flow (+ PKCE)
✅ **Recomendado para:** Web apps, Mobile apps, SPAs

```
Usuário → Cliente → Authorization Server → Usuário autentica
→ Código de autorização → Cliente troca por token → Acesso a recursos
```

### 2. Client Credentials Flow
✅ **Recomendado para:** Servidor-para-servidor, Background jobs

```
Cliente → Authorization Server (client_id + client_secret) → Access Token
```

### 3. Device Code Flow
✅ **Recomendado para:** Smart TVs, IoT devices

```
Dispositivo solicita → Usuário autentica em outro device → Token emitido
```

## Parâmetros Importantes

### URL de Autorização
```
https://provider.com/oauth/authorize?
  response_type=code
  &client_id=CLIENT_ID
  &redirect_uri=REDIRECT_URI
  &scope=SCOPES
  &state=RANDOM_STRING        # Proteção CSRF
  &code_challenge=CHALLENGE    # PKCE
  &code_challenge_method=S256  # PKCE
```

### Troca de Token
```
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=AUTHORIZATION_CODE
&redirect_uri=REDIRECT_URI
&client_id=CLIENT_ID
&client_secret=CLIENT_SECRET   # Confidential clients
&code_verifier=VERIFIER        # PKCE
```

## Scopes Comuns

| Provider | Scopes Exemplo |
|----------|----------------|
| Google | `openid profile email` |
| GitHub | `read:user user:email repo` |
| Facebook | `public_profile email` |
| Microsoft | `openid profile email User.Read` |

## Códigos de Resposta HTTP

| Código | Significado |
|--------|-------------|
| 200 | Sucesso |
| 400 | Bad Request (parâmetros inválidos) |
| 401 | Unauthorized (token inválido/expirado) |
| 403 | Forbidden (sem permissão) |
| 404 | Endpoint não encontrado |

## Tipos de Token

### Access Token
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "read:profile"
}
```

### Refresh Token
```json
{
  "refresh_token": "tGzv3JOkF0XG5Qx2TlKWIA",
  "expires_in": 2592000
}
```

## Segurança - Checklist Rápido

### Sempre Faça
- ✅ Use HTTPS
- ✅ Valide o parâmetro `state`
- ✅ Implemente PKCE (para clientes públicos)
- ✅ Use redirect URI exato
- ✅ Tokens de vida curta
- ✅ Armazene tokens de forma segura
- ✅ Solicite apenas scopes necessários

### Nunca Faça
- ❌ Use HTTP em produção
- ❌ Exponha tokens em URLs
- ❌ Use wildcards em redirect_uri
- ❌ Armazene tokens em logs
- ❌ Compartilhe client_secret
- ❌ Use Implicit Flow (legado)

## Erros Comuns

### invalid_grant
**Causa:** Código de autorização inválido ou expirado  
**Solução:** Reinicie o fluxo de autorização

### invalid_client
**Causa:** client_id ou client_secret incorretos  
**Solução:** Verifique as credenciais da aplicação

### invalid_request
**Causa:** Parâmetros obrigatórios faltando  
**Solução:** Verifique todos os parâmetros necessários

### unauthorized_client
**Causa:** Cliente não autorizado para o grant_type  
**Solução:** Verifique as permissões do cliente no provider

## PKCE - Implementação Rápida

### JavaScript
```javascript
// 1. Gerar code_verifier
const codeVerifier = base64URLEncode(crypto.randomBytes(32));

// 2. Gerar code_challenge
const hash = crypto.createHash('sha256').update(codeVerifier).digest();
const codeChallenge = base64URLEncode(hash);

// 3. Armazenar verifier
sessionStorage.setItem('code_verifier', codeVerifier);

// 4. Usar challenge na URL de autorização
// 5. Enviar verifier na troca de token
```

### Python
```python
import secrets
import hashlib
import base64

# 1. Gerar code_verifier
code_verifier = base64.urlsafe_b64encode(secrets.token_bytes(32)).decode('utf-8').rstrip('=')

# 2. Gerar code_challenge
digest = hashlib.sha256(code_verifier.encode('utf-8')).digest()
code_challenge = base64.urlsafe_b64encode(digest).decode('utf-8').rstrip('=')
```

## Usando Access Token

### HTTP Header
```http
GET /api/profile HTTP/1.1
Host: api.provider.com
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

### JavaScript (Axios)
```javascript
const response = await axios.get('https://api.provider.com/profile', {
  headers: {
    'Authorization': `Bearer ${accessToken}`
  }
});
```

### Python (Requests)
```python
headers = {'Authorization': f'Bearer {access_token}'}
response = requests.get('https://api.provider.com/profile', headers=headers)
```

## Renovação de Token

```http
POST /oauth/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&refresh_token=REFRESH_TOKEN
&client_id=CLIENT_ID
&client_secret=CLIENT_SECRET
```

## Revogação de Token

```http
POST /oauth/revoke HTTP/1.1
Content-Type: application/x-www-form-urlencoded

token=ACCESS_TOKEN
&token_type_hint=access_token
&client_id=CLIENT_ID
&client_secret=CLIENT_SECRET
```

## Recursos Úteis

- **[Documentação Completa](o-que-e-oauth.md)** - Conceitos detalhados
- **[Fluxos OAuth](fluxos-oauth.md)** - Todos os grant types
- **[Segurança](seguranca-oauth.md)** - Melhores práticas
- **[Implementação](implementacao-oauth.md)** - Exemplos de código

## Providers OAuth Populares

| Provider | URL de Documentação |
|----------|---------------------|
| Google | https://developers.google.com/identity/protocols/oauth2 |
| GitHub | https://docs.github.com/en/developers/apps/building-oauth-apps |
| Microsoft | https://docs.microsoft.com/en-us/azure/active-directory/develop/ |
| Facebook | https://developers.facebook.com/docs/facebook-login |
| Auth0 | https://auth0.com/docs |

## Ferramentas de Debug

- **[OAuth.tools](https://oauth.tools/)** - Debug OAuth flows
- **[JWT.io](https://jwt.io/)** - Decode JWT tokens
- **[OAuth Debugger](https://oauthdebugger.com/)** - Test OAuth
- **[Postman](https://www.postman.com/)** - Test APIs

---

Para documentação completa, consulte os arquivos em `/docs/`.
