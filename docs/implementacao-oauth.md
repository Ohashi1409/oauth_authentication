# Implementação Prática do OAuth 2.0

Este documento fornece exemplos práticos de implementação OAuth em diferentes linguagens e frameworks.

## Exemplo 1: Node.js + Express (Authorization Code Flow com PKCE)

### Instalação
```bash
npm install express crypto axios
```

### Código do Cliente

```javascript
const express = require('express');
const crypto = require('crypto');
const axios = require('axios');

const app = express();
const PORT = 3000;

// Configurações OAuth
const OAUTH_CONFIG = {
  clientId: 'YOUR_CLIENT_ID',
  clientSecret: 'YOUR_CLIENT_SECRET', // Opcional para PKCE
  authorizationUrl: 'https://provider.com/oauth/authorize',
  tokenUrl: 'https://provider.com/oauth/token',
  redirectUri: 'http://localhost:3000/callback',
  scope: 'read:profile read:email'
};

// Armazenamento temporário (em produção, use Redis ou banco de dados)
const sessions = new Map();

// Função auxiliar para gerar strings aleatórias
function generateRandomString(length = 32) {
  return crypto.randomBytes(length).toString('hex');
}

// Função auxiliar para PKCE
function base64URLEncode(buffer) {
  return buffer.toString('base64')
    .replace(/\+/g, '-')
    .replace(/\//g, '_')
    .replace(/=/g, '');
}

function generateCodeChallenge(verifier) {
  return base64URLEncode(
    crypto.createHash('sha256').update(verifier).digest()
  );
}

// Rota inicial - inicia o fluxo OAuth
app.get('/login', (req, res) => {
  // Gerar state para proteção CSRF
  const state = generateRandomString();
  
  // Gerar PKCE code_verifier e code_challenge
  const codeVerifier = generateRandomString(43);
  const codeChallenge = generateCodeChallenge(codeVerifier);
  
  // Armazenar state e code_verifier na sessão
  const sessionId = generateRandomString();
  sessions.set(sessionId, {
    state,
    codeVerifier
  });
  
  // Construir URL de autorização
  const params = new URLSearchParams({
    response_type: 'code',
    client_id: OAUTH_CONFIG.clientId,
    redirect_uri: OAUTH_CONFIG.redirectUri,
    scope: OAUTH_CONFIG.scope,
    state: state,
    code_challenge: codeChallenge,
    code_challenge_method: 'S256'
  });
  
  const authUrl = `${OAUTH_CONFIG.authorizationUrl}?${params.toString()}`;
  
  // Enviar sessionId como cookie
  res.cookie('session_id', sessionId, { httpOnly: true });
  res.redirect(authUrl);
});

// Rota de callback - recebe o código de autorização
app.get('/callback', async (req, res) => {
  try {
    const { code, state } = req.query;
    const sessionId = req.cookies?.session_id;
    
    // Validar state para proteção CSRF
    const session = sessions.get(sessionId);
    if (!session || session.state !== state) {
      return res.status(400).send('Invalid state parameter');
    }
    
    // Trocar código por token
    const tokenResponse = await axios.post(OAUTH_CONFIG.tokenUrl, {
      grant_type: 'authorization_code',
      code: code,
      redirect_uri: OAUTH_CONFIG.redirectUri,
      client_id: OAUTH_CONFIG.clientId,
      client_secret: OAUTH_CONFIG.clientSecret, // Opcional para PKCE
      code_verifier: session.codeVerifier
    }, {
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    });
    
    const { access_token, refresh_token, expires_in } = tokenResponse.data;
    
    // Armazenar tokens (em produção, use armazenamento seguro)
    session.accessToken = access_token;
    session.refreshToken = refresh_token;
    session.expiresAt = Date.now() + (expires_in * 1000);
    
    res.send('Login successful! Token received.');
  } catch (error) {
    console.error('Error during token exchange:', error);
    res.status(500).send('Authentication failed');
  }
});

// Rota protegida - usa o access token
app.get('/profile', async (req, res) => {
  try {
    const sessionId = req.cookies?.session_id;
    const session = sessions.get(sessionId);
    
    if (!session || !session.accessToken) {
      return res.status(401).send('Not authenticated');
    }
    
    // Verificar se o token expirou
    if (Date.now() > session.expiresAt) {
      // Renovar token usando refresh_token
      await refreshAccessToken(session);
    }
    
    // Fazer requisição à API protegida
    const profileResponse = await axios.get('https://provider.com/api/profile', {
      headers: {
        'Authorization': `Bearer ${session.accessToken}`
      }
    });
    
    res.json(profileResponse.data);
  } catch (error) {
    console.error('Error fetching profile:', error);
    res.status(500).send('Error fetching profile');
  }
});

// Função para renovar access token
async function refreshAccessToken(session) {
  try {
    const tokenResponse = await axios.post(OAUTH_CONFIG.tokenUrl, {
      grant_type: 'refresh_token',
      refresh_token: session.refreshToken,
      client_id: OAUTH_CONFIG.clientId,
      client_secret: OAUTH_CONFIG.clientSecret
    }, {
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    });
    
    const { access_token, refresh_token, expires_in } = tokenResponse.data;
    
    session.accessToken = access_token;
    if (refresh_token) {
      session.refreshToken = refresh_token; // Refresh token rotation
    }
    session.expiresAt = Date.now() + (expires_in * 1000);
  } catch (error) {
    console.error('Error refreshing token:', error);
    throw error;
  }
}

// Rota de logout
app.get('/logout', (req, res) => {
  const sessionId = req.cookies?.session_id;
  sessions.delete(sessionId);
  res.clearCookie('session_id');
  res.send('Logged out successfully');
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

## Exemplo 2: Python + Flask (Authorization Code Flow)

### Instalação
```bash
pip install flask requests python-dotenv
```

### Código do Cliente

```python
from flask import Flask, redirect, request, session, jsonify
import requests
import secrets
import hashlib
import base64
from urllib.parse import urlencode

app = Flask(__name__)
app.secret_key = 'your-secret-key-here'  # Em produção, use variável de ambiente

# Configurações OAuth
OAUTH_CONFIG = {
    'client_id': 'YOUR_CLIENT_ID',
    'client_secret': 'YOUR_CLIENT_SECRET',
    'authorization_url': 'https://provider.com/oauth/authorize',
    'token_url': 'https://provider.com/oauth/token',
    'redirect_uri': 'http://localhost:5000/callback',
    'scope': 'read:profile read:email'
}

def generate_code_verifier():
    """Gera code_verifier para PKCE"""
    return base64.urlsafe_b64encode(secrets.token_bytes(32)).decode('utf-8').rstrip('=')

def generate_code_challenge(verifier):
    """Gera code_challenge a partir do verifier"""
    digest = hashlib.sha256(verifier.encode('utf-8')).digest()
    return base64.urlsafe_b64encode(digest).decode('utf-8').rstrip('=')

@app.route('/login')
def login():
    # Gerar state para CSRF protection
    state = secrets.token_urlsafe(32)
    session['oauth_state'] = state
    
    # Gerar PKCE parameters
    code_verifier = generate_code_verifier()
    code_challenge = generate_code_challenge(code_verifier)
    session['code_verifier'] = code_verifier
    
    # Construir URL de autorização
    params = {
        'response_type': 'code',
        'client_id': OAUTH_CONFIG['client_id'],
        'redirect_uri': OAUTH_CONFIG['redirect_uri'],
        'scope': OAUTH_CONFIG['scope'],
        'state': state,
        'code_challenge': code_challenge,
        'code_challenge_method': 'S256'
    }
    
    auth_url = f"{OAUTH_CONFIG['authorization_url']}?{urlencode(params)}"
    return redirect(auth_url)

@app.route('/callback')
def callback():
    # Verificar state para CSRF protection
    if request.args.get('state') != session.get('oauth_state'):
        return 'Invalid state parameter', 400
    
    code = request.args.get('code')
    if not code:
        return 'No authorization code received', 400
    
    # Trocar código por token
    token_data = {
        'grant_type': 'authorization_code',
        'code': code,
        'redirect_uri': OAUTH_CONFIG['redirect_uri'],
        'client_id': OAUTH_CONFIG['client_id'],
        'client_secret': OAUTH_CONFIG['client_secret'],
        'code_verifier': session.get('code_verifier')
    }
    
    try:
        response = requests.post(OAUTH_CONFIG['token_url'], data=token_data)
        response.raise_for_status()
        tokens = response.json()
        
        # Armazenar tokens na sessão
        session['access_token'] = tokens['access_token']
        session['refresh_token'] = tokens.get('refresh_token')
        
        return 'Login successful! Token received.'
    except requests.exceptions.RequestException as e:
        return f'Authentication failed: {str(e)}', 500

@app.route('/profile')
def profile():
    access_token = session.get('access_token')
    if not access_token:
        return 'Not authenticated', 401
    
    # Fazer requisição à API protegida
    headers = {'Authorization': f'Bearer {access_token}'}
    
    try:
        response = requests.get('https://provider.com/api/profile', headers=headers)
        response.raise_for_status()
        return jsonify(response.json())
    except requests.exceptions.RequestException as e:
        return f'Error fetching profile: {str(e)}', 500

@app.route('/logout')
def logout():
    session.clear()
    return 'Logged out successfully'

if __name__ == '__main__':
    app.run(debug=True, port=5000)
```

## Exemplo 3: React (SPA com PKCE)

### Instalação
```bash
npm install react axios
```

### Código do Cliente

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const OAUTH_CONFIG = {
  clientId: 'YOUR_CLIENT_ID',
  authorizationUrl: 'https://provider.com/oauth/authorize',
  tokenUrl: 'https://provider.com/oauth/token',
  redirectUri: 'http://localhost:3000/callback',
  scope: 'read:profile read:email'
};

// Funções auxiliares
function generateRandomString(length = 32) {
  const charset = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-._~';
  const values = new Uint8Array(length);
  crypto.getRandomValues(values);
  return Array.from(values)
    .map(v => charset[v % charset.length])
    .join('');
}

async function generateCodeChallenge(verifier) {
  const encoder = new TextEncoder();
  const data = encoder.encode(verifier);
  const hash = await crypto.subtle.digest('SHA-256', data);
  return base64URLEncode(hash);
}

function base64URLEncode(buffer) {
  const bytes = new Uint8Array(buffer);
  let binary = '';
  bytes.forEach(b => binary += String.fromCharCode(b));
  return btoa(binary)
    .replace(/\+/g, '-')
    .replace(/\//g, '_')
    .replace(/=/g, '');
}

function App() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    // Verificar se estamos na rota de callback
    const urlParams = new URLSearchParams(window.location.search);
    const code = urlParams.get('code');
    const state = urlParams.get('state');
    
    if (code) {
      handleCallback(code, state);
    }
  }, []);

  const handleLogin = async () => {
    try {
      // Gerar state para CSRF protection
      const state = generateRandomString();
      sessionStorage.setItem('oauth_state', state);
      
      // Gerar PKCE parameters
      const codeVerifier = generateRandomString(43);
      const codeChallenge = await generateCodeChallenge(codeVerifier);
      sessionStorage.setItem('code_verifier', codeVerifier);
      
      // Construir URL de autorização
      const params = new URLSearchParams({
        response_type: 'code',
        client_id: OAUTH_CONFIG.clientId,
        redirect_uri: OAUTH_CONFIG.redirectUri,
        scope: OAUTH_CONFIG.scope,
        state: state,
        code_challenge: codeChallenge,
        code_challenge_method: 'S256'
      });
      
      const authUrl = `${OAUTH_CONFIG.authorizationUrl}?${params.toString()}`;
      window.location.href = authUrl;
    } catch (error) {
      console.error('Error initiating login:', error);
    }
  };

  const handleCallback = async (code, state) => {
    try {
      setLoading(true);
      
      // Verificar state
      const savedState = sessionStorage.getItem('oauth_state');
      if (state !== savedState) {
        throw new Error('Invalid state parameter');
      }
      
      // Obter code_verifier
      const codeVerifier = sessionStorage.getItem('code_verifier');
      
      // Trocar código por token
      const tokenResponse = await axios.post(OAUTH_CONFIG.tokenUrl, {
        grant_type: 'authorization_code',
        code: code,
        redirect_uri: OAUTH_CONFIG.redirectUri,
        client_id: OAUTH_CONFIG.clientId,
        code_verifier: codeVerifier
      }, {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      });
      
      const { access_token, refresh_token } = tokenResponse.data;
      
      // Armazenar tokens
      sessionStorage.setItem('access_token', access_token);
      if (refresh_token) {
        sessionStorage.setItem('refresh_token', refresh_token);
      }
      
      // Limpar parâmetros da URL
      window.history.replaceState({}, document.title, '/');
      
      // Buscar perfil do usuário
      await fetchUserProfile();
    } catch (error) {
      console.error('Error during callback:', error);
    } finally {
      setLoading(false);
    }
  };

  const fetchUserProfile = async () => {
    try {
      const accessToken = sessionStorage.getItem('access_token');
      const response = await axios.get('https://provider.com/api/profile', {
        headers: {
          'Authorization': `Bearer ${accessToken}`
        }
      });
      setUser(response.data);
    } catch (error) {
      console.error('Error fetching profile:', error);
    }
  };

  const handleLogout = () => {
    sessionStorage.clear();
    setUser(null);
  };

  if (loading) {
    return <div>Loading...</div>;
  }

  return (
    <div>
      <h1>OAuth 2.0 Demo</h1>
      {user ? (
        <div>
          <h2>Welcome, {user.name}!</h2>
          <p>Email: {user.email}</p>
          <button onClick={handleLogout}>Logout</button>
        </div>
      ) : (
        <button onClick={handleLogin}>Login with OAuth</button>
      )}
    </div>
  );
}

export default App;
```

## Providers OAuth Populares

### Google OAuth 2.0
```javascript
const GOOGLE_CONFIG = {
  clientId: 'YOUR_CLIENT_ID.apps.googleusercontent.com',
  authorizationUrl: 'https://accounts.google.com/o/oauth2/v2/auth',
  tokenUrl: 'https://oauth2.googleapis.com/token',
  scope: 'openid profile email'
};
```

### GitHub OAuth
```javascript
const GITHUB_CONFIG = {
  clientId: 'YOUR_CLIENT_ID',
  authorizationUrl: 'https://github.com/login/oauth/authorize',
  tokenUrl: 'https://github.com/login/oauth/access_token',
  scope: 'read:user user:email'
};
```

### Microsoft (Azure AD)
```javascript
const MICROSOFT_CONFIG = {
  clientId: 'YOUR_CLIENT_ID',
  authorizationUrl: 'https://login.microsoftonline.com/common/oauth2/v2.0/authorize',
  tokenUrl: 'https://login.microsoftonline.com/common/oauth2/v2.0/token',
  scope: 'openid profile email'
};
```

## Bibliotecas Recomendadas

### Node.js
- **Passport.js**: Middleware de autenticação completo
- **simple-oauth2**: Cliente OAuth 2.0 simplificado
- **openid-client**: Cliente OpenID Connect certificado

### Python
- **Authlib**: Biblioteca OAuth/OIDC completa
- **requests-oauthlib**: Extensão OAuth para requests
- **Flask-Dance**: OAuth para Flask

### React/JavaScript
- **react-oauth2-code-pkce**: Hook React para OAuth com PKCE
- **oidc-client-ts**: Cliente OpenID Connect
- **Auth0 SDK**: Solução completa de autenticação

## Recursos Adicionais

- [OAuth 2.0 Playground](https://www.oauth.com/playground/) - Teste fluxos OAuth
- [Postman OAuth 2.0](https://learning.postman.com/docs/sending-requests/authorization/#oauth-20) - Teste APIs OAuth
- [OAuth Debugger](https://oauthdebugger.com/) - Debug de implementações OAuth

## Conclusão

A implementação do OAuth requer atenção aos detalhes de segurança. Sempre:
- Use PKCE para aplicações públicas
- Valide o parâmetro state
- Use HTTPS em produção
- Mantenha bibliotecas atualizadas
- Siga as melhores práticas do provider escolhido
