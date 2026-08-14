# Guia: PAC (Plano de Análise de Controle)

O **PAC** é o coração do sistema. Ele define **o que** deve ser monitorado, **com que frequência**, **quais são os critérios de conformidade** e **quais ações tomar em caso de não conformidade**.

> Pense no PAC como uma "receita de inspeções" que o sistema usará diariamente para gerar as pendências de monitoramento da sua equipe.

---

## 1. Entendendo a Hierarquia do PAC

Antes de cadastrar, entenda a estrutura em 4 níveis:

```
PAC (o plano geral)
  └── Monitoramento (o módulo sendo avaliado, ex: "Temperatura da Câmara Fria")
        └── Item Avaliado (o que medir, ex: "Temperatura interna")
              └── Plano de Ação (o que fazer, como medir, e o que fazer se falhar)
```

---

## 2. Configurando o PAC

### Passo 1 — Criar o PAC Base

1. Clique em **PAC** no menu lateral.
2. Clique em **+ Novo** na barra de ações.
3. Preencha:
   - **Nome do PAC:** Ex: "PPHO 01 - Potabilidade da Água"
   - **Frequência:** Com que frequência este PAC deve ser executado (Diário, Semanal, Mensal, etc.)
4. Clique em **💾 Salvar**.

### Passo 2 — Adicionar Monitoramentos ao PAC

Com o PAC selecionado, clique na aba **Monitoramentos** e adicione os tópicos a serem avaliados.

### Passo 3 — Adicionar Itens Avaliados

Dentro de cada Monitoramento, clique em **Itens Avaliados** e detalhe o que será medido ou observado.

### Passo 4 — Criar o Plano de Ação

Dentro de cada Item, configure o **Plano de Ação**, que define:
- O que constitui "Conforme" e "Não Conforme"
- O texto padrão para **Ação Corretiva** (o que fazer imediatamente quando há falha)
- O texto padrão para **Ação Preventiva** (o que fazer para evitar nova ocorrência)
- Prazos para execução das ações

---

## 3. Revisões do PAC

O sistema mantém um histórico de **revisões** do PAC. Toda alteração significativa em um PAC deve gerar uma nova revisão, que fica registrada com data e responsável.

- Clique na aba **Revisões** dentro de um PAC para ver ou criar revisões.
- Cada revisão pode ter um documento PDF anexado (ex: versão assinada do plano).

---

## 4. Legalizações e Legislação

O módulo **Legislação** (acessível pelo menu lateral) permite vincular portarias, instrução normativas e leis ao PAC, criando rastreabilidade legal para as práticas adotadas.

---

## 5. Frequências

O módulo **Frequências** (acessível pelo menu lateral) gerencia os períodos de execução dos PACs (Diário, Semanal, Quinzenal, Mensal, etc.). As frequências disponíveis são usadas ao criar um novo PAC.

---

## 6. Dicas Importantes

> **Atenção:** O PAC deve ser configurado **antes** de iniciar os monitoramentos. Sem um PAC ativo e configurado, o sistema não gerará pendências e o operador não conseguirá registrar os apontamentos.

- Comece pelos PACs que possuem monitoramento **diário** (PPHO) pois eles geram o maior volume de pendências.
- Consulte a equipe FoodTech caso precise de ajuda na configuração inicial do PAC Padrão para o seu segmento.
