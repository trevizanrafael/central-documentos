# Guia: Monitoramento

O **Monitoramento** é a operação mais importante do dia a dia. É onde o operador registra formalmente os resultados de cada inspeção realizada no chão de fábrica — a "assinatura eletrônica" de que o controle foi feito.

---

## 1. Como Registrar um Monitoramento

### Opção A — Pela Home (Recomendado)
Na tela inicial, clique no atalho **"Criar Monitoramento"**. O sistema abrirá a tela diretamente no modo Novo.

### Opção B — Pelo Menu Lateral
Clique em **Monitoramento** no menu lateral.

---

## 2. Preenchendo o Formulário

O formulário de monitoramento é dividido em seções que devem ser preenchidas **em ordem**:

### Seção 1 — Identificação (Cascata de seleção)

Selecione em sequência:

1. **Frequência** — Ex: Diário, Semanal.
2. **PAC** — O plano que está sendo executado.
3. **Monitoramento** — O tópico específico dentro do PAC.
4. **Item Avaliado** — O item sendo medido.
5. **Inspeção / Plano de Ação** — O plano com os critérios de conformidade.

> **Importante:** Cada seleção filtra as opções do campo seguinte. Aguarde o carregamento antes de selecionar o próximo campo.

### Seção 2 — Resultado

Após identificar o que está sendo inspecionado:

- **Conforme / Não Conforme:** Selecione o resultado da inspeção.
- **Resposta:** Campo de texto livre para descrever o resultado numérico ou observação relevante (ex: "Temperatura: 4°C").
- **Observações:** Qualquer anotação adicional para contexto.

---

## 3. Quando Há Não Conformidade

Se o resultado for **Não Conforme**, o sistema exibirá automaticamente a **Seção de Não Conformidade**:

- Os textos de **Ação Corretiva** e **Ação Preventiva** são preenchidos automaticamente com base no Plano de Ação cadastrado, mas podem ser editados.
- **Data Limite Corretiva:** Informe até quando a ação corretiva será executada.
- **Data Limite Preventiva:** Informe até quando a ação preventiva será executada.
- **Destinação do Produto:** Se houver produto envolvido, descreva o que foi feito (ex: descartado, reprocessado, retido).

---

## 4. Salvando o Registro

Após preencher todos os campos obrigatórios:
1. Clique em **💾 Salvar**.
2. O sistema registrará automaticamente:
   - **Responsável:** Seu nome de usuário logado (não pode ser alterado).
   - **Data e Hora:** A data e hora exata do servidor no momento do salvamento.

> **Atenção:** Uma vez salvo, o registro não pode ser excluído. Isso garante a rastreabilidade e a integridade da auditoria.

---

## 5. Executando Ações Corretivas / Preventivas

Quando um monitoramento tiver Não Conformidade registrada, o sistema exibirá um **Bloco de Verificação** com o status da ação.

1. Quando a ação for concluída fisicamente, abra o registro de monitoramento correspondente.
2. Localize o bloco da ação (Corretiva ou Preventiva).
3. Clique em **[ Executar ]**.
4. Confirme no modal de confirmação que aparecerá.
5. O sistema carimbará a data e hora da execução e o status mudará para **Executado ✅**.

> **Atenção:** A ação de "Executar" é irreversível. Confirme apenas quando a ação tiver sido efetivamente realizada.

---

## 6. Modo "Pendentes"

Na parte superior do módulo de Monitoramento, você pode alternar entre:
- **Todos:** Exibe o histórico completo de monitoramentos já realizados.
- **Pendentes:** Exibe **somente** os planos de ação que ainda não foram registrados hoje e estão no prazo ou atrasados. Use este modo para não deixar passar nenhum registro.

---

## 7. Dicas

- Preencha os monitoramentos **no horário correto** em que a inspeção foi realizada.
- Em caso de dúvida sobre o critério de conformidade, consulte o Plano de Ação ou o seu Responsável Técnico.
- Utilize o campo "Observações" para registrar qualquer situação incomum, mesmo que o resultado seja "Conforme".
