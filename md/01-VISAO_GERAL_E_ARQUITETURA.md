# 01. Visão Geral e Arquitetura

> **SysFT (Sistema FoodTech)** - Plataforma Web B2B para Gestão de Conformidade Regulatória em Indústrias de Alimentos.

---

## 1. Stack Tecnológica

O sistema segue uma arquitetura tradicional MVC (Model-View-Controller) renderizada no lado do servidor, sem frameworks frontend complexos tipo React/Vue (Single Page Applications), maximizando a performance inicial, o SEO e a simplicidade de desenvolvimento.

| Camada | Tecnologia |
|---|---|
| **Runtime** | Node.js (CommonJS) |
| **Framework Web** | Express.js |
| **Template Engine** | EJS (Embedded JavaScript) |
| **Banco de Dados** | PostgreSQL |
| **Driver BD** | `pg` (node-postgres) puro, usando Pool de conexões (sem ORM) |
| **Sessões e Auth** | `express-session` com cookies (Baseado na memória ou BD) |
| **Upload de Arquivos** | `multer` (usando `diskStorage`) |
| **Processamento IA** | OpenRouter API (modelo `openai/gpt-4o`) para visão e auditorias |
| **Manipulação de PDF** | `pdfjs-dist` + `canvas` para converter páginas em imagens e submeter à IA |
| **Frontend Markdown** | `marked.js` carregado via CDN para renderizar textos da IA |

---

## 2. Padrão Arquitetural MVC

O repositório é organizado para separar claramente responsabilidades de Roteamento, Lógica de Negócios e Acesso a Dados:

```
src/
├── app.js                     # Entry point: configura Express, sessões, middlewares, rotas globais
├── config/
│   ├── db.js                  # Pool de conexões PostgreSQL (usado em todos os Models)
│   ├── upload*.js             # Configurações do Multer por entidade (Logo, Contatos, etc)
│   └── prompt*.js             # Instruções base (System Prompts) para as chamadas de IA
├── controllers/               # Handlers HTTP, Lógica de Negócios e tratamento de request/response
├── middleware/
│   ├── authMiddleware.js      # Bloqueia rotas restritas a Administradores (Mestre)
│   └── userAuthMiddleware.js  # Bloqueia rotas restritas a Usuários (Clientes SaaS)
├── models/                    # Padrão Data Mapper (Executam SQL puro via pg)
├── routes/
│   ├── adminRoutes.js         # Endpoints para /admin
│   └── userRoutes.js          # Endpoints para /usuario
├── utils/
│   └── renderUserApp.js       # Wrapper para renderizar views injetando dados globais necessários
└── views/
    ├── admin/                 # Templates EJS (Painel Mestre FoodTech)
    └── user/                  # Templates EJS (Painel Cliente SaaS)
```

---

## 3. Padrões de Autenticação e Multitenancy

O sistema possui dois universos de acesso totalmente isolados no código e nas rotas, garantindo segurança entre quem administra o sistema e quem o utiliza:

### 3.1. Painel Master (Admin)
- **Base URL:** `/admin/*`
- **Tabela:** `admins`
- **Sessão:** `req.session.adminId`
- **Middleware:** `requireAdminAuth`
- **Casos de Uso:** Criação de empresas clientes, perfis globais e permissões, usuários masters.

### 3.2. Painel Cliente (User / SaaS)
- **Base URL:** `/usuario/*`
- **Tabela:** `usuarios`
- **Sessão:** `req.session.usuarioId`
- **Middleware:** `requireUserAuth`
- **Casos de Uso:** Todo o business logic do cliente (PAC, Monitoramentos, Produtos, Estabelecimentos).
- **Isolamento de Tenant:** Toda query de um usuário logado **DEVE SEMPRE** filtrar pelo `empresa_id` associado ao usuário. Nunca confie apenas nos IDs passados por rotas; garanta no WHERE da query SQL que aquele dado pertence à empresa do usuário logado.

---

## 4. Convenções e Padrões de Código

### 4.1 Nomenclaturas
- **Arquivos JS:** `camelCase` (ex: `userAuthMiddleware.js`, `importacaoController.js`)
- **Models:** Nomenclatura baseada no negócio + Sufixo `Model` (ex: `ProdutoRotuloModel.js`)
- **Tabelas de Banco:** `snake_case` e pluralizadas (ex: `produtos`, `produto_rotulos`, `empresas`)
- **CSS e HTML IDs:** `kebab-case` para classes (ex: `btn-primary`, `tab-processos`) e `camelCase` ou `kebab-case` para IDs de controle.

### 4.2 Respostas de API (AJAX)
Qualquer rota destinada a ser consumida via AJAX (fetch) do lado do cliente deve seguir um contrato JSON rígido:

**Sucesso:**
```json
{
  "success": true,
  "data": { "id": 1, "nome": "Exemplo" },
  "message": "Mensagem opcional de sucesso"
}
```

**Erro:**
```json
{
  "success": false,
  "message": "Motivo claro do erro para ser renderizado no alert do cliente"
}
```

### 4.3 Rotas REST Padrão
Para manter o Controller limpo, siga o CRUD clássico no mapeamento de rotas (no arquivo `userRoutes.js`):
- `GET /api/entidade` → Mapeia para `Controller.list`
- `POST /api/entidade` → Mapeia para `Controller.create`
- `PUT /api/entidade/:id` → Mapeia para `Controller.update`
- `DELETE /api/entidade/:id` → Mapeia para `Controller.deleteSingle` (Ou Soft Delete)
