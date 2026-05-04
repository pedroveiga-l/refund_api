# Refund API

API para solicitação e consulta de reembolsos com autenticação JWT, upload de comprovantes e persistência em SQLite.

## 🎯 Funcionalidades

- ✅ Cadastro e autenticação de usuários com JWT
- ✅ Upload de comprovantes (imagens)
- ✅ Criação e listagem de solicitações de reembolso
- ✅ Controle de acesso por perfil (employee/manager)
- ✅ Publicação de arquivos em `/uploads`

## 🛠️ Tecnologias

- Node.js, TypeScript, Express
- Prisma ORM, SQLite
- JWT, bcrypt, Zod, multer

## 📥 Como Testar

Arquivos com rotas prontas inclusos:
- `Rotas-API-Insomnia` → Importar no Insomnia
- `Rotas-API-HAR` → Importar em outras ferramentas

## 📋 Perfis de Usuário

| Perfil | Permissões |
|--------|-----------|
| **employee** | Criar reembolsos, enviar comprovantes, consultar por ID |
| **manager** | Listar todos os reembolsos, consultar por ID |

## 🔐 Autenticação

Endpoints protegidos exigem o header:
```
Authorization: Bearer TOKEN_JWT
```

## 📡 Principais Endpoints

### Usuários
- `POST /users` — Criar conta
- `POST /sessions` — Login (retorna token)

### Reembolsos
- `POST /refunds` — Criar reembolso (employee)
- `GET /refunds` — Listar reembolsos com paginação (manager)
- `GET /refunds/:id` — Consultar por ID (employee/manager)

### Uploads
- `POST /uploads` — Enviar comprovante (employee)
- `GET /uploads/:filename` — Baixar arquivo (público)

## 📝 Detalhes dos Endpoints

### POST /users
Criar um novo usuário.
```json
{
  "name": "João Silva",
  "email": "joao@email.com",
  "password": "123456",
  "role": "employee"  // opcional: employee (padrão) ou manager
}
```
**Resposta:** 201 Created (sem corpo)

### POST /sessions
Autenticar e obter token.
```json
{
  "email": "joao@email.com",
  "password": "123456"
}
```
**Resposta:** Token JWT + dados do usuário

### POST /refunds
Criar solicitação de reembolso (requer token).
```json
{
  "name": "Almoço com cliente",
  "category": "food",  // food, services, transport, accomodation, others
  "amount": 89.90,
  "filename": "arquivo_comprovante_1234567890.jpg"
}
```
**Resposta:** 201 Created com dados do reembolso

### GET /refunds
Listar reembolsos (requer token de manager).
- Query params: `?name=João&page=1&perPage=10`
- **Resposta:** Lista paginada

### GET /refunds/:id
Consultar reembolso específico (requer token).
- **Resposta:** Dados do reembolso com informações do usuário

### POST /uploads
Enviar comprovante (requer token de employee).
- **Content-Type:** multipart/form-data
- **Campo:** file (imagem)
- **Aceita:** JPEG, JPG, PNG (máx 3MB)
- **Resposta:** Nome do arquivo salvo

### GET /uploads/:filename
Baixar arquivo (acesso público).
- **Resposta:** Arquivo binário

## 🚀 Como Rodar

```bash
# Instalar dependências
npm install

# Gerar Prisma Client
npx prisma generate

# Rodar migrations
npx prisma migrate dev

# Iniciar servidor (dev)
npm run dev
```

Servidor rodará em `http://localhost:3333`

## 🛢️ Banco de Dados

- **SQLite** local em `dev.db`
- Modelos: `User`, `Refund`
- Migrações em `prisma/migrations`

## ❌ Erros Comuns

<<<<<<< HEAD
| Status | Mensagem | Solução |
|--------|----------|---------|
| 401 | JWT token not found | Enviar token no header Authorization |
| 401 | Invalid JWT token | Token expirado ou inválido |
| 401 | Unauthorized | Seu perfil não tem permissão |
| 400 | Validation failed | Dados enviados inválidos |
| 400 | Email already exists | Usuário já cadastrado |