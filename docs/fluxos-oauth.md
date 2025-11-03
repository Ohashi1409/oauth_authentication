# Fluxos de Autorização OAuth 2.0

OAuth 2.0 define diferentes fluxos (também chamados de "grant types") para diferentes cenários de uso. Cada fluxo é otimizado para um tipo específico de aplicação.

## 1. Authorization Code Flow

### Descrição
O fluxo mais seguro e recomendado para aplicações web que podem manter um segredo do cliente (client secret) de forma segura no servidor.

### Quando Usar
- Aplicações web tradicionais com backend
- Aplicações que podem armazenar segredos de forma segura

### Passos do Fluxo

```
1. Usuário acessa a aplicação cliente
2. Cliente redireciona usuário para o servidor de autorização
3. Usuário autentica e autoriza a aplicação
4. Servidor de autorização redireciona de volta com um código de autorização
5. Cliente troca o código por um access token (usando client secret)
6. Cliente usa o access token para acessar recursos
```

### Exemplo de URL de Autorização
```
https://authorization-server.com/auth?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=https://client-app.com/callback&
  scope=read_profile&
  state=RANDOM_STRING
```

### Vantagens
- O access token nunca é exposto ao navegador do usuário
- Mais seguro devido ao uso do client secret
- Suporta refresh tokens

## 2. Implicit Flow

### Descrição
Fluxo simplificado onde o access token é retornado diretamente na URL de redirecionamento. **Não é mais recomendado** para aplicações novas.

### Quando Usar
- **Legado**: Aplicações JavaScript antigas (SPA)
- **Atual**: Não recomendado, use Authorization Code Flow com PKCE

### Passos do Fluxo

```
1. Usuário acessa a aplicação cliente
2. Cliente redireciona usuário para o servidor de autorização
3. Usuário autentica e autoriza
4. Servidor redireciona de volta com access token no fragment da URL
5. Cliente extrai o token e usa para acessar recursos
```

### Desvantagens
- Token exposto na URL
- Não suporta refresh tokens
- Menos seguro que outros fluxos

## 3. Authorization Code Flow with PKCE

### Descrição
Extensão do Authorization Code Flow que adiciona proteção contra ataques de interceptação. **Recomendado para aplicações mobile e SPAs**.

### Quando Usar
- Aplicações mobile (iOS, Android)
- Single Page Applications (SPAs)
- Qualquer aplicação que não pode manter um client secret seguro

### PKCE (Proof Key for Code Exchange)

PKCE adiciona dois parâmetros ao fluxo:
- **code_verifier**: String aleatória gerada pelo cliente
- **code_challenge**: Hash SHA-256 do code_verifier

### Passos do Fluxo

```
1. Cliente gera code_verifier e code_challenge
2. Cliente redireciona usuário com code_challenge
3. Usuário autentica e autoriza
4. Servidor retorna código de autorização
5. Cliente troca código por token, enviando code_verifier
6. Servidor valida code_verifier contra code_challenge
7. Cliente recebe access token
```

### Vantagens
- Não requer client secret
- Protege contra ataques de interceptação
- Recomendado pela comunidade de segurança

## 4. Client Credentials Flow

### Descrição
Usado quando a aplicação cliente precisa acessar seus próprios recursos, não recursos de um usuário específico.

### Quando Usar
- Comunicação servidor-para-servidor
- Background jobs
- APIs que não atuam em nome de um usuário

### Passos do Fluxo

```
1. Cliente envia client_id e client_secret para o servidor de autorização
2. Servidor retorna access token
3. Cliente usa token para acessar recursos
```

### Exemplo de Requisição
```http
POST /token HTTP/1.1
Host: authorization-server.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&
client_id=CLIENT_ID&
client_secret=CLIENT_SECRET&
scope=api.read
```

## 5. Resource Owner Password Credentials Flow

### Descrição
O cliente coleta diretamente as credenciais do usuário (username e password). **Não recomendado**, exceto em cenários muito específicos.

### Quando Usar
- **Raramente**: Apenas quando há extrema confiança entre cliente e servidor
- Migração de sistemas legados
- Aplicações de primeira parte (owned pelo mesmo provider)

### Por que Evitar
- Viola o princípio fundamental do OAuth (não compartilhar senhas)
- Usuário não pode revogar acesso sem trocar senha
- Menos seguro

## 6. Device Code Flow

### Descrição
Projetado para dispositivos com entrada limitada (Smart TVs, consoles de jogos, IoT devices).

### Quando Usar
- Dispositivos sem navegador completo
- Dispositivos com entrada limitada

### Passos do Fluxo

```
1. Dispositivo solicita device code ao servidor
2. Servidor retorna device code e user code
3. Dispositivo exibe user code e URL para o usuário
4. Usuário acessa URL em outro dispositivo e insere user code
5. Usuário autentica e autoriza
6. Dispositivo faz polling para verificar autorização
7. Servidor retorna access token quando autorizado
```

## Comparação dos Fluxos

| Fluxo | Segurança | Uso Recomendado | Refresh Token |
|-------|-----------|-----------------|---------------|
| Authorization Code | ⭐⭐⭐ | Aplicações web com backend | ✅ |
| Authorization Code + PKCE | ⭐⭐⭐ | SPAs, Mobile apps | ✅ |
| Implicit | ⭐ | Legado (não use) | ❌ |
| Client Credentials | ⭐⭐⭐ | Servidor-para-servidor | ❌ |
| Password Credentials | ⭐ | Evitar quando possível | ✅ |
| Device Code | ⭐⭐⭐ | Dispositivos com entrada limitada | ✅ |

## Melhores Práticas

1. **Use Authorization Code Flow com PKCE** para aplicações modernas
2. **Evite Implicit Flow** - use PKCE em vez disso
3. **Use HTTPS** sempre
4. **Valide o state parameter** para prevenir CSRF
5. **Implemente refresh token rotation** quando possível
6. **Use tokens de vida curta** para access tokens
7. **Armazene tokens de forma segura** (nunca em localStorage sem criptografia)

## Referências

- [RFC 6749 - OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)
- [RFC 7636 - PKCE](https://tools.ietf.org/html/rfc7636)
- [OAuth 2.0 Security Best Current Practice](https://tools.ietf.org/html/draft-ietf-oauth-security-topics)
