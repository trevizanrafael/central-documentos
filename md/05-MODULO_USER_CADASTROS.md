# 05. Módulo User (Cadastros Base)

Este documento centraliza a arquitetura dos cadastros principais operados pelo usuário final (SaaS), abrangendo **Estabelecimento**, **Produtos** e **Ingredientes**.

---

## 1. Padrão de UI "Sub-records"

Os módulos mais complexos (Estabelecimento e Produtos) adotam um padrão de interface chamado **SubrecordManager**, inspirado em sistemas ERP. 
Em vez de colocar dezenas de campos numa única tela longa, o formulário principal fica no topo e possui **Abas (Tabs)** na parte inferior.
- Clicar numa aba oculta a tela principal e abre um painel 100% focado no Sub-record (ex: Contatos, Endereços, Rótulos).
- Cada painel de Sub-record possui sua própria barra de ações (*Novo, Salvar, Excluir, Próximo, Anterior*) e gerencia seu estado em memória via um array JavaScript antes de enviar ao backend.
- Modais são usados para preview de anexos (PDFs/Imagens) em tela cheia (92vh).

---

## 2. Módulo: Estabelecimento
- **Controller:** `src/controllers/estabelecimentoController.js`
- **Tabelas:** `estabelecimentos` (Pai), `estabelecimento_contatos`, `estabelecimento_enderecos`, `estabelecimento_unidades_inspecao`, `estabelecimento_anexos_documentos`.

Este é o coração burocrático do cliente. Toda conta de usuário possui **apenas 1 Estabelecimento** vinculado (relação 1:1 com o usuário).

**Destaques:**
- **Gate de Habilitação:** A UI inicial só desbloqueia os campos após o usuário responder duas perguntas-chave: "Registrar como CPF ou CNPJ?" e "Pequeno porte?".
- **Integração BrasilAPI:** Dispara fetch no frontend para preencher automaticamente CNPJ e CEP.
- **Anexos Dinâmicos:** A lista de documentos exigidos (Planta Baixa, Alvará, etc) não é fixa. Ela muda dinamicamente via JS com base nas respostas do Gate de Habilitação. Um PDF de até 20MB (`config/uploadDocumento.js`) pode ser enviado para cada item exigido, fazendo `Upsert` caso reenviado.

---

## 3. Módulo: Ingredientes (Dicionário)
- **Controller:** `src/controllers/ingredientesController.js`
- **Tabela:** `ingredientes`

Este módulo é um cadastro muito simples. Serve exclusivamente para criar o "Dicionário de Ingredientes" da empresa. 
Quando o cliente vai cadastrar a fórmula de um Produto, ele não digita o nome do ingrediente à mão; ele seleciona do banco de Ingredientes que ele mesmo cadastrou aqui.

---

## 4. Módulo: Produtos
- **Controller:** `src/controllers/produtosController.js`
- **Tabelas:** `produtos` (Pai), `produto_rotulos`, `produto_ingredientes`, `produto_processo_fabricacao`.

É o core operacional da indústria de alimentos. Um estabelecimento cadastra dezenas de Produtos.

### 4.1 Sub-record: Rótulos e Auditoria de IA
- Cada produto pode ter várias versões de rótulo (ex: 500g, 1Kg).
- O usuário envia a arte do rótulo (PDF ou Imagem) usando `config/uploadRotulo.js`.
- **A Mágica da IA:** Ao clicar em "Avaliar com IA", o backend (`pdfjs-dist` + `canvas`) converte o PDF em imagens JPG de alta resolução, envia essas imagens para o modelo de Visão Computacional (OpenRouter -> GPT-4o) junto com o Prompt técnico (`config/promptRotulo.js`).
- A IA responde um laudo técnico em Markdown, que é salvo na coluna `avaliacao_ia` e renderizado na tela do usuário usando `marked.js`.

### 4.2 Sub-record: Ingredientes e Processo
- **Ingredientes:** Uma lista N:N vinculando o Produto à Tabela de Ingredientes, onde o usuário define a Porcentagem na fórmula.
- **Processo de Fabricação:** Em vez de um array de passos, o banco possui 7 colunas fixas de tipo `TEXT` (Etapa 1: Recebimento... Etapa 7: Expedição). O front puxa os 7 campos numa única tela de formulário (registro único por produto).
