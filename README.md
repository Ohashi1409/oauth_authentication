# OAuth Authentication

Repositório completo sobre OAuth 2.0 com documentação em markdown, exemplos de código e espaço para apresentação.

## 📚 Documentação

Explore os conceitos fundamentais do OAuth 2.0:

### Conteúdo Principal

**[📋 Guia Rápido de Referência](docs/guia-rapido.md)** - Cheat sheet com os conceitos essenciais

1. **[O que é OAuth?](docs/o-que-e-oauth.md)**
   - Introdução ao OAuth
   - História e evolução
   - Conceitos principais (Resource Owner, Client, Authorization Server, etc.)
   - Por que usar OAuth?
   - Diferença entre OAuth e OpenID Connect

2. **[Fluxos de Autorização OAuth](docs/fluxos-oauth.md)**
   - Authorization Code Flow
   - Implicit Flow (legado)
   - Authorization Code Flow com PKCE
   - Client Credentials Flow
   - Resource Owner Password Credentials Flow
   - Device Code Flow
   - Comparação e melhores práticas

3. **[Segurança no OAuth](docs/seguranca-oauth.md)**
   - Vulnerabilidades comuns (CSRF, Token Leakage, etc.)
   - Melhores práticas de segurança
   - Implementação de PKCE
   - Token management
   - Checklist de segurança

4. **[Implementação Prática](docs/implementacao-oauth.md)**
   - Exemplos em Node.js + Express
   - Exemplos em Python + Flask
   - Exemplos em React (SPA)
   - Providers OAuth populares (Google, GitHub, Microsoft)
   - Bibliotecas recomendadas

## 🎯 Estrutura do Repositório

```
oauth_authentication/
├── docs/                          # Documentação completa em markdown
│   ├── o-que-e-oauth.md          # Conceitos fundamentais
│   ├── fluxos-oauth.md           # Fluxos de autorização
│   ├── seguranca-oauth.md        # Segurança e melhores práticas
│   └── implementacao-oauth.md    # Exemplos práticos de código
├── presentation/                  # Apresentação em PDF
│   └── README.md                 # Instruções para adicionar apresentação
└── README.md                     # Este arquivo
```

## 🎤 Apresentação

O diretório [`presentation/`](presentation/) está reservado para armazenar a apresentação sobre OAuth em formato PDF.

Para adicionar sua apresentação:
1. Coloque o arquivo PDF no diretório `presentation/`
2. Consulte o [README da apresentação](presentation/README.md) para mais detalhes

## 🚀 Começando

Se você é novo no OAuth, recomendamos seguir esta ordem:

1. Veja o **[Guia Rápido](docs/guia-rapido.md)** para uma visão geral dos conceitos
2. Comece com **[O que é OAuth?](docs/o-que-e-oauth.md)** para entender os conceitos básicos
3. Explore os **[Fluxos de Autorização](docs/fluxos-oauth.md)** para conhecer as diferentes formas de implementar OAuth
4. Leia sobre **[Segurança](docs/seguranca-oauth.md)** para entender como proteger sua implementação
5. Veja os **[Exemplos Práticos](docs/implementacao-oauth.md)** para implementar OAuth em seus projetos

## 📖 Recursos Adicionais

### Especificações Oficiais
- [RFC 6749 - The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)
- [RFC 6750 - Bearer Token Usage](https://tools.ietf.org/html/rfc6750)
- [RFC 7636 - PKCE](https://tools.ietf.org/html/rfc7636)

### Sites Úteis
- [OAuth.net](https://oauth.net/) - Site oficial do OAuth
- [OAuth 2.0 Simplified](https://www.oauth.com/) - Guia completo
- [Auth0 Docs](https://auth0.com/docs/authenticate) - Documentação Auth0

### Ferramentas
- [OAuth 2.0 Playground](https://www.oauth.com/playground/)
- [JWT.io](https://jwt.io/) - Debug de tokens JWT
- [OAuth Debugger](https://oauthdebugger.com/)

## 🤝 Contribuindo

Sinta-se à vontade para contribuir com este repositório:
- Adicione mais exemplos de código
- Melhore a documentação existente
- Corrija erros ou typos
- Adicione recursos adicionais

## 📝 Licença

Este repositório é de código aberto e está disponível para fins educacionais.
Repositório para armazenar links de apresentação e códigos desenvolvidos para serem apresentados durante a apresentação sobre OAuth Authentication
