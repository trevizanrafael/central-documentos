# 04. Módulo Admin (Master)

O Módulo Admin é o painel de controle mestre da FoodTech. Ele é acessado pela rota base `/admin/` e gerencia a criação dos clientes (Empresas), controle de acessos (Perfis) e credenciais (Usuários).

---

## 1. Gestão de Empresas
- **Controller:** `src/controllers/empresasController.js`
- **Tabela:** `empresas`
- **Papel no Sistema:** Representa a entidade pagante do SaaS (Tenant).
- **Relacionamentos:** 1 Empresa possui *N* Usuários. Toda informação no painel do usuário final é escopada pelo `empresa_id`.

**Destaques:**
- **Upload de Logotipo:** O cadastro de empresa permite upload de imagem (configurado em `src/config/upload.js`). Essa imagem é salva em `/uploads/empresas/` e é injetada globalmente no painel do cliente (`user-app.ejs`) para branding (White-label parcial).
- **Segurança (Delete):** Não é possível excluir uma empresa que ainda possui usuários vinculados (A FK constraint 23503 do Postgres bloqueia a ação, e o sistema retorna um erro amigável).

---

## 2. Perfis de Acesso (Controle de Módulos)
- **Controller:** `src/controllers/perfisController.js`
- **Tabela:** `perfis`
- **Papel no Sistema:** Grupo de permissões que dita o que um usuário pode ou não ver no painel operacional.

**Destaques de Arquitetura:**
- Ao invés de criar dezenas de colunas booleanas, o sistema utiliza o tipo de dado nativo do Postgres **`TEXT[]`** (Array de Strings) na coluna `permissions`.
- **Exemplo de Carga:** `['estabelecimento', 'produtos', 'ingredientes']`
- **Impacto no Frontend:** No momento do login do usuário, as permissões são cacheadas e o EJS (`user-app.ejs`) condicionalmente renderiza os botões do menu lateral checando se o módulo existe dentro do array `permissions`.

---

## 3. Gestão de Usuários (Contas Cliente)
- **Controller:** `src/controllers/usuariosController.js`
- **Tabela:** `usuarios`
- **Papel no Sistema:** São as pessoas físicas que vão logar no painel `/usuario/` e operar o software.

**Destaques de Segurança:**
- Senhas são hasheadas obrigatoriamente usando `bcrypt` com *salt rounds* de 10.
- Senhas nuas **nunca** transitam para fora do banco na rota `GET` (o SQL explícito omite a coluna).
- **Upsert na Importação CSV:** O sistema suporta onboarding rápido lendo uma planilha CSV e processando linha a linha via email (Se não existe, cria; Se existe, atualiza).

---

## 4. UI e Funcionalidades Globais do Admin
A interface do painel Admin adota os seguintes padrões de interação AJAX (sem refresh):

- **Tabelas de Dados:** Renderizadas via EJS, com suporte nativo a Checkboxes na primeira coluna.
- **Ações em Lote:** Selecionar múltiplas linhas e clicar em "Excluir Selecionados" dispara a rota de Batch Delete enviando um Array de IDs no body da request JSON.
- **Formulários Modais:** Operações de CRUD (Criar/Editar) abrem deslizando modais laterais, reciclando a mesma estrutura HTML para insert e update via preenchimento de input hidden `id`.
