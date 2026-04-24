# 🍸 BarOS — Gestão Inteligente de Bares

SaaS B2B para profissionalizar a operação de bares: fichas técnicas, padronização para bartenders, cálculo automático de CMV, controle de insumos e integração com fornecedores.

---

## 🏗 Stack

- **Frontend** — React 18 + Vite + TailwindCSS + React Router + Axios
- **Backend** — Node.js + NestJS 10 + TypeORM + Passport JWT
- **Banco** — PostgreSQL 16
- **Auth** — JWT (Bearer Token)

---

## 📁 Estrutura

```
baros/
├── backend/                 # API NestJS
│   ├── src/
│   │   ├── auth/            # JWT, login, registro, /auth/me
│   │   ├── users/           # Entidade + service
│   │   ├── recipes/         # Receitas + cálculo de custo/CMV/dashboard
│   │   ├── ingredients/     # CRUD de ingredientes
│   │   ├── suppliers/       # Fornecedores
│   │   ├── products/        # Produtos + recálculo automático
│   │   ├── ocr/             # /ocr/scan (mock para Fase 2)
│   │   ├── common/          # Enums, decorators, guards
│   │   ├── config/          # data-source TypeORM
│   │   └── database/        # seed.ts
│   └── package.json
├── frontend/                # React + Vite
│   ├── src/
│   │   ├── api/             # axios + interceptor JWT
│   │   ├── context/         # AuthContext
│   │   ├── components/      # Layout, Sidebar, Topbar, ProtectedRoute
│   │   └── pages/           # Login, Dashboard, Recipes, Ingredients, Suppliers, Products
│   └── package.json
├── docker-compose.yml       # Postgres
└── README.md
```

---

## 🚀 Subindo o projeto (passo-a-passo)

### 1. Pré-requisitos
- Node.js 18+
- Docker (ou Postgres local)

### 2. Subir o Postgres

```bash
docker compose up -d
```

### 3. Backend

```bash
cd backend
cp .env.example .env
npm install
npm run start:dev          # roda em http://localhost:3333/api
```

Em outro terminal, popular dados de exemplo:

```bash
cd backend
npm run seed
```

Isso cria:
- Admin: `admin@baros.dev` / `admin123`
- Bartender: `bartender@baros.dev` / `bartender123`
- 6 ingredientes
- 1 receita pronta (Negroni)

### 4. Frontend

```bash
cd frontend
npm install
npm run dev                # http://localhost:5173
```

Acesse `http://localhost:5173`, faça login com `admin@baros.dev / admin123`.

---

## 🔑 Tipos de usuário e permissões

| Role        | Permissões                                                         |
| ----------- | ------------------------------------------------------------------ |
| `ADMIN`     | Tudo: cria/edita receitas, ingredientes, fornecedores, vê CMV     |
| `BARTENDER` | Leitura: visualiza receitas e fichas técnicas                      |
| `SUPPLIER`  | Cadastra produtos próprios, atualiza preços                        |

---

## 📚 Endpoints REST

Base: `http://localhost:3333/api` · Auth: `Authorization: Bearer <jwt>`

### Auth (público)
- `POST /auth/register` — cria usuário e retorna token
- `POST /auth/login` — autentica e retorna token
- `POST /auth/forgot-password` — stub seguro (não vaza enumeração)
- `GET  /auth/me` — usuário logado

### Receitas
- `GET    /recipes` — lista
- `POST   /recipes` *(ADMIN)* — cria com cálculo automático de custo e CMV
- `GET    /recipes/:id` — detalhe com ingredientes
- `PATCH  /recipes/:id` *(ADMIN)* — edita (recalcula se ingredientes/preço mudarem)
- `DELETE /recipes/:id` *(ADMIN)*
- `GET    /recipes/:id/suggest-price?targetCmv=25` — sugestão de preço
- `GET    /recipes/dashboard` — métricas agregadas

### Ingredientes
- `GET /ingredients` · `POST /ingredients` *(ADMIN)* · `PATCH /ingredients/:id` *(ADMIN)* · `DELETE /ingredients/:id` *(ADMIN)*

### Fornecedores
- `GET /suppliers` · `POST /suppliers` *(ADMIN/SUPPLIER)* · `PATCH /suppliers/:id` *(ADMIN/SUPPLIER)* · `DELETE /suppliers/:id` *(ADMIN)*

### Produtos
- `GET /products` *(filtro opcional `?supplierId=`)*
- `POST /products` *(ADMIN/SUPPLIER)*
- `PATCH /products/:id/price` *(ADMIN/SUPPLIER)* — **dispara recálculo de todas as receitas que usam o ingrediente**
- `DELETE /products/:id` *(ADMIN)*

### OCR (Fase 2 — mock)
- `POST /ocr/scan` *(ADMIN)* — multipart com campo `file`. Retorna estrutura simulada de receita.

---

## 🧮 Regras de negócio implementadas

1. **Cálculo de custo total**
   `custo_total = Σ (quantidade × custo_base do ingrediente)`

2. **CMV (%)**
   `CMV = (custo_total / preço_venda) × 100`

3. **Sugestão de preço**
   `preço_sugerido = custo_total / (CMV_alvo / 100)`

4. **Recálculo em cascata** — ao atualizar o preço de um produto de fornecedor:
   - O `baseCost` do ingrediente é sincronizado com o **menor preço** entre fornecedores.
   - Todas as receitas que usam esse ingrediente são recalculadas (custo total + CMV + snapshots por linha).

5. **Permissões via decorator `@Roles()` + `RolesGuard`**.

---

## 🗺 Roadmap

### ✅ Fase 1 — MVP
- [x] Autenticação JWT
- [x] CRUD receitas (com cálculo automático)
- [x] CRUD ingredientes
- [x] Cálculo de CMV e sugestão de preço

### ✅ Fase 2 — Gestão
- [x] Dashboard (CMV médio, receita mais rentável, pior CMV)
- [x] Permissões por role (ADMIN/BARTENDER/SUPPLIER)
- [x] Estrutura de OCR pronta (mock em `/ocr/scan`)

### ✅ Fase 3 — Fornecedores
- [x] CRUD fornecedores
- [x] CRUD produtos
- [x] Recálculo automático ao atualizar preço

### 🔜 Próximos passos sugeridos
- Integração OCR real (Tesseract.js / Google Vision / Claude Vision)
- Histórico de preços (tabela `price_history`)
- Pedidos para fornecedores (`Orders`)
- Multi-tenant (1 conta = 1 bar com vários usuários)
- Relatórios em PDF
- Notificações quando CMV ultrapassar limite
- App mobile para bartenders consultarem fichas

---

## 🧪 Testando rapidamente via cURL

```bash
# Login
TOKEN=$(curl -s -X POST http://localhost:3333/api/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"email":"admin@baros.dev","password":"admin123"}' | jq -r .access_token)

# Listar receitas
curl http://localhost:3333/api/recipes -H "Authorization: Bearer $TOKEN"

# Dashboard
curl http://localhost:3333/api/recipes/dashboard -H "Authorization: Bearer $TOKEN"
```

---

## 📝 Notas técnicas

- `synchronize: true` está ativo apenas em `NODE_ENV=development` para acelerar o setup. Em produção, gere migrations com `npm run migration:generate -- src/database/migrations/Init` e aplique com `npm run migration:run`.
- `JWT_SECRET` no `.env.example` é placeholder — **trocar antes de subir para produção**.
- O endpoint `/auth/forgot-password` é um stub seguro: nunca confirma se o email existe (evita user enumeration). Em produção, implementar token de reset com expiração + envio de email.
- O Vite usa proxy `/api → http://localhost:3333` em dev, então o frontend chama `/api/...` direto sem CORS na hora.
