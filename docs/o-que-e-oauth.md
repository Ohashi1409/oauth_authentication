# O que é OAuth?

## Introdução

OAuth (Open Authorization) é um padrão aberto de autorização que permite que aplicações obtenham acesso limitado a recursos de usuários em outros serviços, sem expor suas credenciais (senha). É amplamente utilizado na internet para permitir que você faça login em sites usando suas contas do Google, Facebook, GitHub, entre outros.

## História

- **OAuth 1.0**: Lançado em 2007, foi o primeiro padrão OAuth
- **OAuth 1.0a**: Revisão de segurança lançada em 2009
- **OAuth 2.0**: Versão atual, lançada em 2012 (RFC 6749), mais simples e flexível

## Conceitos Principais

### 1. Resource Owner (Proprietário do Recurso)
O usuário que possui os dados e pode conceder acesso a eles. Por exemplo, você é o proprietário dos seus dados no Google.

### 2. Client (Cliente)
A aplicação que deseja acessar os recursos protegidos em nome do usuário. Por exemplo, um aplicativo de gerenciamento de tarefas que quer acessar seu Google Calendar.

### 3. Authorization Server (Servidor de Autorização)
O servidor que autentica o proprietário do recurso e emite tokens de acesso após obter autorização. Por exemplo, os servidores de autenticação do Google.

### 4. Resource Server (Servidor de Recursos)
O servidor que hospeda os recursos protegidos. Aceita e responde a requisições que incluem tokens de acesso válidos.

### 5. Access Token
Um token de credencial usado para acessar recursos protegidos. É uma string que representa a autorização concedida ao cliente.

### 6. Refresh Token
Um token usado para obter um novo access token quando o atual expira, sem precisar da intervenção do usuário.

## Por que usar OAuth?

### Benefícios

1. **Segurança**: Os usuários não precisam compartilhar suas senhas com aplicações terceiras
2. **Controle Granular**: Os usuários podem conceder acesso limitado a recursos específicos
3. **Revogação Fácil**: Os usuários podem revogar o acesso a qualquer momento sem alterar suas senhas
4. **Padrão da Indústria**: Amplamente adotado e suportado por grandes empresas
5. **Melhor Experiência do Usuário**: Login único (SSO) facilita o acesso a múltiplos serviços

### Casos de Uso Comuns

- Login social (Sign in with Google, Facebook, GitHub)
- Integração entre aplicações (ex: postar automaticamente no Twitter)
- Acesso a APIs (ex: acessar dados do Google Drive)
- Aplicações mobile que precisam acessar recursos de backend

## Diferença entre OAuth e OpenID Connect

- **OAuth 2.0**: Protocolo de **autorização** (permite acesso a recursos)
- **OpenID Connect**: Protocolo de **autenticação** construído sobre OAuth 2.0 (verifica identidade do usuário)

Muitas implementações modernas usam ambos em conjunto para fornecer autenticação e autorização.

## Próximos Passos

Para entender melhor o OAuth, explore os seguintes documentos:
- [Fluxos de Autorização OAuth](fluxos-oauth.md)
- [Segurança no OAuth](seguranca-oauth.md)
- [Implementação Prática](implementacao-oauth.md)

## Referências

- [RFC 6749 - The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)
- [RFC 6750 - The OAuth 2.0 Authorization Framework: Bearer Token Usage](https://tools.ietf.org/html/rfc6750)
- [OAuth.net - Official OAuth Website](https://oauth.net/)
