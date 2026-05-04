# refund_api

API REST para solicitação e consulta de reembolsos, com autenticação JWT, controle de acesso por perfil, upload de comprovantes e persistência em SQLite via Prisma.

## Descrição detalhada

Esta API foi criada para apoiar o fluxo de solicitação de reembolsos de despesas. Ela permite cadastrar usuários, autenticar sessões, enviar comprovantes de imagem, registrar solicitações de reembolso e consultar solicitações já cadastradas.

O problema que ela resolve é centralizar o processo de reembolso em uma API simples, com validação de dados, proteção das rotas e separação de permissões entre perfis de usuário.

Principais funcionalidades:

- Cadastro de usuários com perfil `employee` ou `manager`.
- Autenticação com token JWT.
- Upload de arquivos de imagem para comprovantes.
- Criação de solicitações de reembolso.
- Listagem paginada de reembolsos com filtro por nome do usuário.
- Consulta de um reembolso específico por ID.
- Tratamento centralizado de erros e validações.

## Tecnologias utilizadas

- Linguagem: TypeScript
- Runtime: Node.js
- Framework: Express
- ORM: Prisma
- Banco de dados: SQLite
- Validação: Zod
- Autenticação: jsonwebtoken
- Criptografia de senha: bcrypt
- Upload de arquivos: multer
- Middleware de erros assíncronos: express-async-errors
- CORS: cors

## Estrutura do projeto

- `src/app.ts`: configura o servidor Express, middlewares globais, arquivos estáticos e rotas.
- `src/server.ts`: inicia a aplicação na porta `3333`.
- `src/routes/`: concentra as rotas da API.
- `src/controllers/`: contém a lógica de cada endpoint.
- `src/middlewares/`: guarda autenticação, autorização e tratamento de erros.
- `src/configs/`: configurações de autenticação JWT e upload de arquivos.
- `src/database/`: instancia o cliente Prisma.
- `src/providers/`: serviços auxiliares, como armazenamento em disco.
- `src/types/`: extensões de tipos do Express, como `request.user`.
- `src/utils/`: classes utilitárias, como `AppError`.
- `prisma/schema.prisma`: define os modelos, enums e conexão com o banco.
- `prisma/migrations/`: histórico de migrações do banco.
- `tmp/`: pasta temporária usada no fluxo de upload.
- `tmp/uploads/`: destino final dos arquivos enviados.

Responsabilidades principais:

- Controllers: validam dados de entrada, executam consultas no banco e retornam as respostas.
- Middlewares: protegem rotas e padronizam erros.
- Providers: encapsulam regras de armazenamento de arquivos.
- Prisma: faz o mapeamento entre a aplicação e o banco SQLite.

## Como executar o projeto

### Pré-requisitos

- Node.js instalado.
- npm instalado.
- Banco SQLite local gerenciado pelo Prisma.

### Instalação

1. Instale as dependências:

```bash
npm install
```

2. Gere o client do Prisma, se necessário:

```bash
npx prisma generate
```

3. Aplique as migrações no banco local:

```bash
npx prisma migrate dev
```

4. Inicie a aplicação em modo de desenvolvimento:

```bash
npm run dev
```

A API sobe em `http://localhost:3333`.

### Autenticação

As rotas protegidas exigem o header abaixo:

```http
Authorization: Bearer <token>
```

## Rotas da API

### POST /users

Cria um novo usuário.

- Método HTTP: `POST`
- URL: `/users`
- Acesso: público
- Body: JSON
- Parâmetros:
  - `name` obrigatório, string, mínimo de 2 caracteres
  - `email` obrigatório, string, formato de e-mail
  - `password` obrigatório, string, mínimo de 6 caracteres
  - `role` opcional, `employee` ou `manager`, padrão `employee`

Exemplo de requisição:

```bash
curl -X POST http://localhost:3333/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Ana Souza",
    "email": "ana.souza@empresa.com",
    "password": "123456",
    "role": "employee"
  }'
```

Exemplo de resposta:

```http
HTTP/1.1 201 Created
```

Observação: a rota retorna `201` sem corpo de resposta.

---

### POST /sessions

Autentica um usuário e gera um token JWT.

- Método HTTP: `POST`
- URL: `/sessions`
- Acesso: público
- Body: JSON
- Parâmetros:
  - `email` obrigatório, string, formato de e-mail
  - `password` obrigatório, string

Exemplo de requisição:

```bash
curl -X POST http://localhost:3333/sessions \
  -H "Content-Type: application/json" \
  -d '{
    "email": "ana.souza@empresa.com",
    "password": "123456"
  }'
```

Exemplo de resposta:

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "5eb1f1d4-9d13-4ed8-9b1f-9c7d1a5ef001",
    "name": "Ana Souza",
    "email": "ana.souza@empresa.com",
    "role": "employee",
    "createdAt": "2026-05-04T12:00:00.000Z",
    "updatedAt": null
  }
}
```

---

### POST /uploads

Envia uma imagem para armazenamento local e retorna o nome final do arquivo.

- Método HTTP: `POST`
- URL: `/uploads`
- Acesso: privado
- Perfis permitidos: `employee`
- Content-Type: `multipart/form-data`
- Campo do arquivo: `file`
- Parâmetros:
  - arquivo obrigatório
  - formatos aceitos: `image/jpeg`, `image/jpg`, `image/png`
  - tamanho máximo: 3 MB

Exemplo de requisição:

```bash
curl -X POST http://localhost:3333/uploads \
  -H "Authorization: Bearer <token>" \
  -F "file=@/caminho/para/comprovante.png"
```

Exemplo de resposta:

```json
{
  "filename": "a1b2c3d4e5f6a7b8c9d0-comprovante.png"
}
```

Observação: os arquivos enviados ficam acessíveis publicamente em `GET /uploads/<filename>`.

---

### POST /refunds

Cria uma nova solicitação de reembolso vinculada ao usuário autenticado.

- Método HTTP: `POST`
- URL: `/refunds`
- Acesso: privado
- Perfis permitidos: `employee`
- Body: JSON
- Parâmetros:
  - `name` obrigatório, string
  - `category` obrigatório, enum com os valores `food`, `others`, `services`, `transport` e `accomodation`
  - `amount` obrigatório, número positivo
  - `filename` obrigatório, string com o nome retornado em `/uploads`

Exemplo de requisição:

```bash
curl -X POST http://localhost:3333/refunds \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Almoço com cliente",
    "category": "food",
    "amount": 89.9,
    "filename": "a1b2c3d4e5f6a7b8c9d0-comprovante.png"
  }'
```

Exemplo de resposta:

```json
{
  "id": "1c2d3e4f-5678-4abc-9012-3456789def01",
  "name": "Almoço com cliente",
  "amount": 89.9,
  "category": "food",
  "filename": "a1b2c3d4e5f6a7b8c9d0-comprovante.png",
  "userId": "5eb1f1d4-9d13-4ed8-9b1f-9c7d1a5ef001",
  "createdAt": "2026-05-04T12:15:00.000Z",
  "updatedAt": null
}
```

---

### GET /refunds

Lista solicitações de reembolso com paginação e filtro por nome do usuário.

- Método HTTP: `GET`
- URL: `/refunds`
- Acesso: privado
- Perfis permitidos: `manager`
- Query params:
  - `name` opcional, string, filtra pelo nome do usuário relacionado
  - `page` opcional, número, padrão `1`
  - `perPage` opcional, número, padrão `10`

Exemplo de requisição:

```bash
curl "http://localhost:3333/refunds?name=Ana&page=1&perPage=5" \
  -H "Authorization: Bearer <token>"
```

Exemplo de resposta:

```json
{
  "refunds": [
    {
      "id": "1c2d3e4f-5678-4abc-9012-3456789def01",
      "name": "Almoço com cliente",
      "amount": 89.9,
      "category": "food",
      "filename": "a1b2c3d4e5f6a7b8c9d0-comprovante.png",
      "userId": "5eb1f1d4-9d13-4ed8-9b1f-9c7d1a5ef001",
      "createdAt": "2026-05-04T12:15:00.000Z",
      "updatedAt": null,
      "user": {
        "id": "5eb1f1d4-9d13-4ed8-9b1f-9c7d1a5ef001",
        "name": "Ana Souza",
        "email": "ana.souza@empresa.com",
        "password": "$2b$08$...",
        "role": "employee",
        "createdAt": "2026-05-04T12:00:00.000Z",
        "updatedAt": null
      }
    }
  ],
  "pagination": {
    "page": 1,
    "perPage": 5,
    "totalRecords": 1,
    "totalPages": 1
  }
}
```

Observação: a listagem é filtrada pelo nome do usuário vinculado ao reembolso.

---

### GET /refunds/:id

Consulta um reembolso específico pelo ID.

- Método HTTP: `GET`
- URL: `/refunds/:id`
- Acesso: privado
- Perfis permitidos: `employee` e `manager`
- Parâmetros de rota:
  - `id` obrigatório, UUID

Exemplo de requisição:

```bash
curl "http://localhost:3333/refunds/1c2d3e4f-5678-4abc-9012-3456789def01" \
  -H "Authorization: Bearer <token>"
```

Exemplo de resposta:

```json
{
  "id": "1c2d3e4f-5678-4abc-9012-3456789def01",
  "name": "Almoço com cliente",
  "amount": 89.9,
  "category": "food",
  "filename": "a1b2c3d4e5f6a7b8c9d0-comprovante.png",
  "userId": "5eb1f1d4-9d13-4ed8-9b1f-9c7d1a5ef001",
  "createdAt": "2026-05-04T12:15:00.000Z",
  "updatedAt": null,
  "user": {
    "id": "5eb1f1d4-9d13-4ed8-9b1f-9c7d1a5ef001",
    "name": "Ana Souza",
    "email": "ana.souza@empresa.com",
    "password": "$2b$08$...",
    "role": "employee",
    "createdAt": "2026-05-04T12:00:00.000Z",
    "updatedAt": null
  }
}
```

Observação: quando o ID não existe, a implementação atual retorna `null`.

## Exemplos de uso

### Fluxo completo de consumo

1. Crie um usuário.
2. Faça login em `/sessions` e copie o token.
3. Envie o comprovante em `/uploads` usando o token.
4. Crie o reembolso em `/refunds` com o `filename` retornado no upload.
5. Se você for `manager`, consulte a listagem em `/refunds`.

### Exemplo com fetch

```javascript
const loginResponse = await fetch("http://localhost:3333/sessions", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    email: "ana.souza@empresa.com",
    password: "123456",
  }),
});

const { token } = await loginResponse.json();

const uploadForm = new FormData();
uploadForm.append("file", fileInput.files[0]);

const uploadResponse = await fetch("http://localhost:3333/uploads", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${token}`,
  },
  body: uploadForm,
});

const { filename } = await uploadResponse.json();

await fetch("http://localhost:3333/refunds", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${token}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "Almoço com cliente",
    category: "food",
    amount: 89.9,
    filename,
  }),
});
```

## Tratamento de erros

A API usa um tratamento centralizado de erros.

### Formato padrão de erro

- Erro de domínio ou autenticação:

```json
{
  "message": "Mensagem do erro"
}
```

- Erro de validação com Zod:

```json
{
  "message": "Validation failed",
  "issues": {
    "...": "..."
  }
}
```

### Principais erros retornados

- `400 Bad Request`: validação de dados falhou.
- `401 Unauthorized`: token JWT ausente, inválido ou perfil não autorizado.
- `500 Internal Server Error`: erro inesperado no servidor.

Mensagens importantes da implementação atual:

- `JWT token not found`
- `Invalid JWT token`
- `Unauthorized`
- `Já existe um usuário com esse e-mail`
- `E-mail ou senha inválido`

## Autor

Nome do desenvolvedor: seu nome aqui

Se desejar, substitua o texto acima pelo seu nome e mantenha este README como documentação oficial do projeto.