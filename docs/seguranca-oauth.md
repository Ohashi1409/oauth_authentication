# Segurança no OAuth 2.0

A segurança é fundamental na implementação do OAuth. Este documento cobre as principais considerações de segurança e melhores práticas.

## Vulnerabilidades Comuns

### 1. CSRF (Cross-Site Request Forgery)

#### Descrição
Um atacante engana o usuário para autorizar uma aplicação maliciosa sem seu conhecimento.

#### Proteção
Use o parâmetro **state** no fluxo de autorização:
```
1. Cliente gera valor aleatório único (state)
2. Cliente armazena state na sessão
3. Cliente inclui state na URL de autorização
4. Servidor retorna state no redirect
5. Cliente valida que state recebido == state armazenado
```

#### Exemplo
```javascript
// Gerar state
const state = crypto.randomBytes(32).toString('hex');
sessionStorage.setItem('oauth_state', state);

// Incluir na URL de autorização
const authUrl = `https://provider.com/oauth/authorize?
  client_id=${CLIENT_ID}&
  redirect_uri=${REDIRECT_URI}&
  state=${state}`;

// Validar no callback
const receivedState = urlParams.get('state');
const storedState = sessionStorage.getItem('oauth_state');
if (receivedState !== storedState) {
  throw new Error('Invalid state parameter');
}
```

### 2. Authorization Code Interception

#### Descrição
Um atacante intercepta o código de autorização durante o redirect.

#### Proteção
- **PKCE (Proof Key for Code Exchange)**: Obrigatório para aplicações públicas
- **HTTPS**: Use sempre em todos os endpoints
- **Redirect URI exato**: Não use wildcards ou URI parciais

### 3. Token Leakage

#### Descrição
Tokens são expostos através de logs, URLs, ou armazenamento inseguro.

#### Proteção
- **Nunca coloque tokens em URLs**
- **Use refresh tokens** para minimizar tempo de vida do access token
- **Armazenamento seguro**:
  - Backend: Variáveis de ambiente, secret managers
  - Frontend: httpOnly cookies (se possível) ou sessionStorage (com cuidado)
- **Não registre tokens em logs**

### 4. Open Redirect

#### Descrição
Atacante manipula o redirect_uri para direcionar o usuário para um site malicioso.

#### Proteção
- **Whitelist de redirect URIs**: Configure URIs exatos no provider
- **Valide redirect_uri** rigorosamente no servidor
- **Não use wildcards** em redirect URIs

### 5. Replay Attacks

#### Descrição
Um atacante reutiliza um código de autorização ou token capturado.

#### Proteção
- **Códigos de autorização de uso único**: Servidor deve invalidar após primeiro uso
- **Tokens de vida curta**: Access tokens devem expirar rapidamente (15-60 minutos)
- **Token binding**: Vincula tokens a características do cliente

## Melhores Práticas de Segurança

### 1. HTTPS Everywhere
```
✅ SEMPRE use HTTPS
❌ NUNCA use HTTP para OAuth
```
- Protege contra man-in-the-middle attacks
- Obrigatório para produção
- Use também em desenvolvimento (ex: localhost com certificado self-signed)

### 2. Validação de Redirect URI

#### No Registro da Aplicação
```javascript
// ✅ BOM - URI exato
redirect_uri: "https://myapp.com/callback"

// ❌ RUIM - Wildcards
redirect_uri: "https://myapp.com/*"
redirect_uri: "https://*.myapp.com/callback"
```

#### No Servidor de Autorização
```javascript
// Validar correspondência exata
if (request.redirect_uri !== registeredRedirectUri) {
  throw new Error('Invalid redirect_uri');
}
```

### 3. Scope Granular

#### Princípio do Menor Privilégio
```javascript
// ✅ BOM - Scopes específicos
scopes: ["read:profile", "read:email"]

// ❌ RUIM - Scopes amplos demais
scopes: ["read", "write", "admin"]
```

#### Validação de Scopes
- Servidor deve validar e limitar scopes solicitados
- Usuário deve ver claramente quais permissões está concedendo
- Aplicação deve solicitar apenas scopes necessários

### 4. Token Management

#### Access Token
```javascript
// Configuração recomendada
{
  type: "Bearer",
  expires_in: 3600,        // 1 hora
  scope: "read:profile"
}
```

#### Refresh Token
```javascript
{
  expires_in: 2592000,     // 30 dias
  rotation: true,          // Emitir novo refresh token a cada uso
  revocable: true          // Permitir revogação
}
```

#### Armazenamento Seguro
```javascript
// ✅ BOM - Backend
process.env.ACCESS_TOKEN
// ou
secretManager.getSecret('oauth_token')

// ✅ ACEITÁVEL - Frontend (com precauções)
sessionStorage.setItem('token', accessToken)  // Melhor que localStorage

// ❌ RUIM - Frontend
localStorage.setItem('token', accessToken)    // Vulnerável a XSS
// Token em cookie sem httpOnly                // Vulnerável a XSS
```

### 5. Client Authentication

#### Confidential Clients (Backend)
```javascript
// Use client_secret
POST /token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTH_CODE&
client_id=CLIENT_ID&
client_secret=CLIENT_SECRET&
redirect_uri=REDIRECT_URI
```

#### Public Clients (Frontend, Mobile)
```javascript
// Use PKCE (sem client_secret)
POST /token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTH_CODE&
client_id=CLIENT_ID&
code_verifier=CODE_VERIFIER&
redirect_uri=REDIRECT_URI
```

### 6. Implementação de PKCE

```javascript
// 1. Gerar code_verifier
const codeVerifier = base64UrlEncode(crypto.randomBytes(32));

// 2. Gerar code_challenge
const codeChallenge = base64UrlEncode(
  crypto.createHash('sha256').update(codeVerifier).digest()
);

// 3. Armazenar code_verifier
sessionStorage.setItem('code_verifier', codeVerifier);

// 4. Incluir code_challenge na autorização
const authUrl = `https://provider.com/oauth/authorize?
  client_id=${CLIENT_ID}&
  redirect_uri=${REDIRECT_URI}&
  code_challenge=${codeChallenge}&
  code_challenge_method=S256`;

// 5. Enviar code_verifier na troca de token
const tokenResponse = await fetch('https://provider.com/oauth/token', {
  method: 'POST',
  body: new URLSearchParams({
    grant_type: 'authorization_code',
    code: authorizationCode,
    client_id: CLIENT_ID,
    redirect_uri: REDIRECT_URI,
    code_verifier: sessionStorage.getItem('code_verifier')
  })
});
```

### 7. Token Revocation

Implemente endpoints para revogação de tokens:

```javascript
// Endpoint de revogação
POST /oauth/revoke
Content-Type: application/x-www-form-urlencoded

token=ACCESS_TOKEN&
token_type_hint=access_token&
client_id=CLIENT_ID&
client_secret=CLIENT_SECRET
```

### 8. Auditoria e Monitoramento

#### Logs de Segurança
```javascript
// Eventos a registrar:
- Tentativas de login
- Autorizações concedidas
- Tokens emitidos
- Tokens revogados
- Tentativas de acesso negado
- Padrões suspeitos
```

#### Alertas
- Múltiplas tentativas de autorização falhadas
- Uso de tokens expirados
- Acessos de localizações incomuns
- Padrões de uso anormais

## Checklist de Segurança OAuth

### Implementação do Cliente
- [ ] HTTPS em todos os endpoints
- [ ] Validação do parâmetro state
- [ ] PKCE implementado (para clientes públicos)
- [ ] Redirect URI exato registrado
- [ ] Armazenamento seguro de tokens
- [ ] Tokens não expostos em URLs ou logs
- [ ] Solicitação de scopes mínimos necessários
- [ ] Implementação de token refresh
- [ ] Tratamento adequado de erros
- [ ] Timeout de sessão implementado

### Implementação do Servidor
- [ ] Validação estrita de redirect_uri
- [ ] Códigos de autorização de uso único
- [ ] Tokens com tempo de expiração
- [ ] Suporte a PKCE
- [ ] Rate limiting implementado
- [ ] Auditoria de eventos de segurança
- [ ] Endpoint de revogação de tokens
- [ ] Validação de scopes
- [ ] Proteção contra CSRF
- [ ] Proteção contra replay attacks

## Recursos Adicionais

### Documentos de Referência
- [OAuth 2.0 Security Best Current Practice](https://tools.ietf.org/html/draft-ietf-oauth-security-topics)
- [OAuth 2.0 Threat Model and Security Considerations](https://tools.ietf.org/html/rfc6819)
- [OWASP OAuth Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html)

### Ferramentas de Teste
- [OAuth.tools](https://oauth.tools/) - Ferramentas de debug OAuth
- [JWT.io](https://jwt.io/) - Debug de JWT tokens
- [Burp Suite](https://portswigger.net/burp) - Teste de segurança

## Conclusão

A segurança no OAuth requer atenção cuidadosa aos detalhes e implementação correta das melhores práticas. Sempre:
- Mantenha bibliotecas atualizadas
- Siga as recomendações da comunidade
- Realize testes de segurança regulares
- Monitore e audite o uso de OAuth
