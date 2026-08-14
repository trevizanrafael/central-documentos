# 07. UI, IDV e Padrões Frontend

Este documento compila a Identidade Visual (IDV) corporativa da FoodTech e os padrões de desenvolvimento Frontend adotados no sistema.

---

## 1. Identidade Visual (IDV)

O SysFT não é um sistema lúdico ou fofinho. Ele é um **software industrial premium para inspeção sanitária**. A aparência deve sempre transparecer robustez, segurança jurídica e tecnologia.

### Cores Principais
- **Cor Primária:** Azul FoodTech `#0A73C9` (Botões principais, links, sidebars ativas).
- **Cor Secundária:** Grafite Escuro `#222222` (Textos, sidebar inativa, títulos).
- **Fundo Global:** Cinza Gelo `#F4F5F7` (Traz leveza e destaca os painéis brancos).
- **Fundo de Paineis:** Branco Neutro `#FFFFFF`.
- **Status:** Verde Sucesso (`#16A34A`), Laranja Atenção (`#F59E0B`), Vermelho Erro (`#DC2626`).

### Tipografia
- Fonte primária: **Inter** (do Google Fonts).
- Sem serifa, geométrica e extremamente limpa para leituras densas de planilhas e formulários.

---

## 2. Padrão de Partials (EJS)

Para manter a interface consistente e não duplicar código, o sistema depende pesadamente de arquivos na pasta `src/views/partials/`. **Sempre** use os partials abaixo em vez de codar botões na mão:

### `module-actionbar.ejs`
- É a barra horizontal exibida no topo de qualquer módulo (Estabelecimento, Produto, Monitoramento, PAC).
- Gera automaticamente os botões `+ Novo`, `💾 Salvar`, `🗑 Excluir` e a navegação `<- 1 / N ->`.
- Passa-se os parâmetros no include:
  ```ejs
  <%- include('../partials/module-actionbar', { modulePrefix: 'pac', tabTitle: 'Cadastro PAC' }) %>
  ```

### `pdf-viewer-modal.ejs`
- É o Modal Fullscreen padrão de 92vh para visualizar PDFs (Alvarás, Plantas, Rótulos).
- Usa Iframe para PDF e tag `<img>` para JPG/PNG (cobrindo a lacuna deixada pelo upload da IA).

---

## 3. Padrão "Sub-records" (SubrecordManager)

O frontend das telas mais pesadas (onde existem relações de 1 para N) foi abstraído numa classe/função JavaScript chamada `SubrecordManager`. 
Essa arquitetura impede a criação de telas infinitas (scroll de 5000px) organizando a visualização em abas isoladas.

**A Regra de Ouro do Sub-record:**
1. A tela começa mostrando a Tabela Pai.
2. Na base da Tabela Pai, existem botões inativos simulando abas.
3. Ao selecionar um registro (ex: "Produto X"), as abas acendem.
4. Ao clicar na Aba (ex: "Rótulos"), o HTML do formulário Pai ganha a classe `hidden` instantaneamente, o Subrecord perde o `hidden`, e um `window.scrollTo(0,0)` ocorre. O usuário foi virtualmente "transportado" para dentro daquele contexto.
5. Há sempre um botão `<- Voltar` no pão-de-migalha (breadcrumb) para refazer o caminho oposto.

### Controle de Estado de Botões no Sub-record
A action bar do sub-record protege o operador de erros:
- **Novo:** Habilitado sempre que o sistema não estiver salvando algo. Ao clicar, o formulário limpa e os botões "Excluir" e navegação de setas ficam desabilitados.
- **Salvar:** Só habilita se o formulário ficar "sujo" (dirty) via evento `input`.
- **Excluir:** Só pode ser clicado num registro salvo e se o form não tiver edições pendentes (evita excluir sem querer quando se tentava apenas limpar o form).

### Zero Registros (Empty State)
Nenhuma tela de Sub-record pode carregar um formulário desabilitado feio quando não existem registros. Sempre deve renderizar uma `div.sr-empty` convidativa com um ícone de vetor e uma instrução "Nenhum documento cadastrado. Clique em NOVO para criar.".

---

## 4. Injeção de Dados Backend -> Frontend
Para injetar variáveis vindas do servidor NodeJS no script do EJS (sem acionar alertas de linter sobre aspas erradas):
Usamos o padrão "JSON Island":

```html
<script id="_serverData" type="application/json">
  { "empresa": <%- JSON.stringify(empresa) %> }
</script>
<script>
  const data = JSON.parse(document.getElementById('_serverData').textContent);
  console.log(data.empresa);
</script>
```
