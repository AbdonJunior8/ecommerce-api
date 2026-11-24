# E-commerce API

API REST em Node.js, Express, Sequelize e MySQL para gerenciamento de usuários, categorias e produtos, com autenticação JWT, migrations e documentação Swagger.

## Requisitos

- Node.js 18 ou superior
- MySQL 8+
- npm

## Configuração

1. Crie o banco no MySQL:

```sql
CREATE DATABASE ecommerce_api CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

2. Instale as dependências:

```bash
npm install
```

3. Copie `.env.example` para `.env` e ajuste os dados de conexão e o segredo JWT:

```bash
cp .env.example .env
```

No Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

4. Execute as migrations:

```bash
npm run db:migrate
```

5. Inicie o servidor:

```bash
npm run dev
```

A API estará em `http://localhost:3000`. Documentação: `http://localhost:3000/api-docs`. Health check: `http://localhost:3000/health`.

## Autenticação

Cadastre um usuário em `POST /v1/user` (esse endpoint é público para permitir o primeiro cadastro). Depois faça login em `POST /v1/user/token` com `email` e `password`. Envie o token retornado nos endpoints protegidos:

```http
Authorization: Bearer SEU_TOKEN
```

POST, PUT e DELETE de categorias e produtos, além de PUT e DELETE de usuários, exigem JWT. GET e cadastro inicial de usuário e login são públicos.

> Os requisitos fornecidos divergem entre status 400 e 401 para token ausente/inválido. Esta implementação usa **401 Unauthorized**, padrão HTTP para falha de autenticação e também indicado nos requisitos individuais de CRUD.

## Endpoints

### Usuários
- `POST /v1/user` — cadastro (`firstname`, `surname`, `email`, `password`, `confirmPassword`)
- `POST /v1/user/token` — login e geração JWT
- `GET /v1/user/:id` — consulta
- `PUT /v1/user/:id` — atualização protegida
- `DELETE /v1/user/:id` — exclusão protegida

### Categorias
- `GET /v1/category/search?limit=12&page=1&fields=name,slug&use_in_menu=true`
- `GET /v1/category/:id`
- `POST /v1/category`
- `PUT /v1/category/:id`
- `DELETE /v1/category/:id`

### Produtos
- `GET /v1/product/search?limit=12&page=1&match=tenis&category_ids=1,2&price-range=100-200`
- `GET /v1/product/:id`
- `POST /v1/product`
- `PUT /v1/product/:id`
- `DELETE /v1/product/:id`

O parâmetro `limit=-1` desativa a paginação. `fields` permite selecionar campos. Em produtos, também há suporte a `option[ID]=valor1,valor2`.

## Exemplo de produto

```json
{
  "enabled": true,
  "name": "Tênis casual",
  "slug": "tenis-casual",
  "stock": 10,
  "description": "Tênis confortável",
  "price": 119.9,
  "price_with_discount": 99.9,
  "category_ids": [1],
  "images": [
    { "type": "image/png", "content": "data:image/png;base64,BASE64_DA_IMAGEM" }
  ],
  "options": [
    { "title": "Tamanho", "shape": "square", "type": "text", "values": ["P", "M", "G"] }
  ]
}
```

Imagens base64 devem ser enviadas como Data URI (`data:image/png;base64,...`). URLs HTTP(S) também são aceitas. Arquivos são salvos em `uploads/products/:productId` e servidos pela rota `/uploads`. A pasta `uploads/` é ignorada pelo Git.

## Testes

```bash
npm test
```

Os testes incluídos cobrem helpers de paginação e seleção de campos. Para testes de integração completos, configure um banco de teste e amplie a suíte com Supertest.

## Estrutura

```text
src/
  config/
  controllers/
  database/migrations/
  middleware/
  models/
  routes/
  services/
  app.js
  server.js
  swagger.js
tests/
```

## Observações técnicas

- Senhas são armazenadas com hash bcrypt (12 rounds), nunca em texto puro.
- Sequelize usa migrations; não há `sync({ force: true })`.
- Preços usam `DECIMAL(10,2)` em vez de FLOAT para evitar imprecisão monetária.
- Associações de produto/categoria são muitos-para-muitos; imagens e opções pertencem a um produto.
- Respostas de exclusão/atualização usam `204` sem corpo.
- Exclusão de produto remove imagens e opções relacionadas via `ON DELETE CASCADE`.
- Para produção, configure HTTPS, segredo JWT forte, limites de payload, validação adicional, política de CORS e armazenamento externo de imagens.
# ecommerce-api
