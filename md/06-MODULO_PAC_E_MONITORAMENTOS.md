# 06. Módulo PAC e Monitoramentos

Este módulo é onde ocorre a operação diária da indústria. Onde os planos de ação desenhados são efetivamente monitorados e checados na fábrica.

---

## 1. Visão Geral da Arquitetura de Monitoramento
- **Controller:** `src/controllers/monitoramentoController.js`
- **Tabela:** `registros_monitoramento`

O Monitoramento não é um cadastro avulso, ele é **derivado** de um Plano de Ação (PAC) previamente configurado no sistema. A tabela guarda a checagem que o operador fez no chão de fábrica e registra sua conformidade, amarrando a autoria da inspeção para fins de auditoria do MAPA.

### Dois Modos de Operação no Frontend
1. **Modo "Todos":** Exibe o histórico de tudo que já foi monitorado, com ID real salvo no banco.
2. **Modo "Pendentes":** Um cenário especial onde a query (`CTE PENDING_CTE` no banco) varre os planos de ação que estão *atrasados* ou *dentro do vencimento* mas que ainda **não possuem** um registro de monitoramento hoje. Esses itens aparecem na UI com `id === null` e só viram registro real no banco quando o usuário clica em "Salvar".

---

## 2. A Mágica do "Cascade Select"
A criação de um monitoramento exige que o operador navegue numa hierarquia profunda de 5 níveis. Para manter a performance e não carregar um JSON gigantesco, o sistema usa rotas `/api/monitoramento/cascade/*`.

**A Hierarquia:**
1. Seleciona a **Frequência** (Diário, Semanal...)
2. Filtra os **Planos PAC** daquela frequência.
3. Filtra o **Monitoramento** (a aba do PAC).
4. Filtra o **Item Avaliado**.
5. Filtra a **Inspeção / Plano de Ação**.

Cada seleção desabilita os de baixo temporariamente e faz um fetch AJAX em cadeia.

---

## 3. Gestão de Conformidade (Seções Dinâmicas)
O formulário de Monitoramento é reativo ao resultado da inspeção:

- Se **"Conforme"**: O operador preenche observações e salva. O formulário tem apenas as Seções 1 (Identificação), 2 (Resultado) e a última de Rastreabilidade.
- Se **"Não Conforme"**: O Javascript injeta instantaneamente a **Seção 3 (Não Conformidade)**. Esta seção puxa os textos de "Ação Corretiva" e "Ação Preventiva" configurados no Plano Base, e exige que o operador defina Prazos (Datas Limite).

---

## 4. Blocos de Verificação (Assinatura Eletrônica Simples)
Quando há Não Conformidade, as ações corretivas e preventivas geram pendências.
O painel exibe um Bloco de Verificação com um botão **[ Executar ]**.

1. Ao clicar, um modal de confirmação aparece.
2. Confirmando, o sistema injeta um carimbo de data via `new Date().toISOString()`.
3. O status vai para **"Executado" (Verde)** e o botão fica **disabled**.
4. **Segurança:** O clique dispara um `PUT` imediato. É uma operação irreversível. A data exata da execução da corretiva é salva em `data_execucao_corretiva` no Postgres.

---

## 5. Rastreabilidade e Auditoria (Injeção de Responsável)
Em sistemas regulatórios, o operador não pode digitar o próprio nome.

- O campo `responsavel_nome` **NUNCA** vem do `req.body` do frontend.
- O controller injeta a autoria no servidor resgatando a variável de sessão de login: `const responsavelNome = req.session.usuarioNome || 'Desconhecido';`
- Isso impede que requisições forjadas via API Postman mascarem a autoria do apontamento.

---

## 6. Débitos Técnicos (Pendências Futuras)
*(Este módulo possui dívidas técnicas documentadas que devem ser sanadas em Sprints futuras)*:
1. **Remover Exclusão:** Monitoramentos assinados devem ser imutáveis. O botão "Excluir" deve sumir da UI após o primeiro salvamento.
2. **Campos Readonly:** Após salvo, os campos de identificação (Cascata e Não Conformidades base) devem ser travados no front, permitindo editar apenas "Observações" ou clicar em "Executar" corretiva.
3. **Bloqueio de Duplicidade:** Evitar que no modo "Pendentes" cliques duplos gerem 2 registros de monitoramento para a mesma inspeção no mesmo dia.
