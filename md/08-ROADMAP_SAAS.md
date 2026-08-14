# 08. Roadmap: Transformação SaaS e Expansão de Negócios

Este documento formaliza as demandas de negócio e visão de produto para transformar o SysFT (atualmente uma infraestrutura técnica sólida) em um verdadeiro **Software as a Service (SaaS)** monetizável, self-service e escalável.

---

## 1. Estrutura de Pacotes e Limites (Tiering)
O sistema precisa reconhecer diferentes "Planos de Assinatura" para as empresas clientes, de modo a limitar o uso e forçar upgrades:
- **Tabela de Planos:** Criar entidade que defina limites duros (Ex: Limite de Usuários = 3, Limite de Produtos = 50, Limite de Armazenamento de Arquivos = 5GB, Limite de Auditorias de IA = 100/mês).
- **Enforcement (Bloqueio):** Middlewares no Node.js ou validações no Controller que retornem `403 Forbidden` quando a empresa tentar dar `POST` em um novo monitoramento ou criar novo usuário além da cota contratada, exibindo modal convidando para Upgrade.

## 2. Onboarding e Auto-serviço (Self-Service)
A jornada de aquisição de um cliente SaaS deve ter o mínimo de atrito humano possível.
- **Fase 1 (Atual - MVP Manual):** O usuário preenche o cadastro, paga (Pix/Cartão externamente), e a equipe FoodTech aprova e manda um email de boas-vindas avisando que a conta está liberada.
- **Fase 2 (Automação):** Checkout via Stripe/Pagar.me onde o pagamento com sucesso via Webhook já libera a conta do usuário instantaneamente e envia o email automático.
- **Landing Page ("Nossa Página"):** Construção de uma vitrine atrativa (fora do painel `/usuario/login`) focada em vendas, explicando os benefícios do PAC Digital, ROI para a indústria e call-to-action para cadastro.

## 3. Clone Mágico do PAC Padrão (O Pulo do Gato)
Para que o auto-serviço funcione, o cliente não pode cair num sistema 100% vazio e ter que criar o PAC do zero (isso gera Churn alto por frustração).
- **O Problema:** Como o cliente inicializa sua matriz de qualidade?
- **A Solução:** Existirá uma empresa invisível no sistema chamada "PAC_MASTER_TEMPLATE". Quando um cliente criar a conta, uma rotina no backend varre todas as frequências, PACs, e inspeções dessa empresa Template, e dá um `INSERT` massivo copiando toda a árvore hierárquica (alterando o `empresa_id` para o novo cliente).
- **Auditoria Registrar Migração:** Esse processo de clone da estrutura padrão para a conta do usuário precisa de logs rigorosos para garantir que não houve perda de dados no meio do caminho.

## 4. Base de Conhecimento e Suporte ao Cliente
Reduzir o volume de suporte humano é crucial para a margem de lucro de um SaaS B2B.
- **Vídeos Explicativos / Help / Manuais:** Uma nova aba no painel do usuário onde ele pode ver tutoriais gravados de como preencher o registro, como anexar PDFs e responder não-conformidades.
- **Suporte Multicanal:** Definição clara no sistema (talvez num modal flutuante) com SLA de resposta para Chamados Internos (tickets no banco de dados), atendimento Telefônico (Planos Premium) e Chat Online (WhatsApp Business ou widget no site).

## 5. Estrutura de E-mails Corporativos
- **SMTP Global:** Configuração de um e-mail com domínio próprio (`suporte@foodtech.net.br` via Zoho ou Google Workspace).
- **Disparos Transacionais:** Implementar o envio de emails via Node.js (usando `nodemailer` ou API como SendGrid/AWS SES) para notificar o usuário nas seguintes ações: Boas-vindas (Onboarding), Reset de Senha e Alertas Críticos de Não-Conformidade ou Pacotes Perto do Limite.

## 6. Ajuste Final de Infraestrutura
- O crescimento do SaaS ditará a infraestrutura. Precisamos garantir a conclusão das subidas de Quota na AWS (vCPUs) para instanciar máquinas Teste ($12) e Prod ($24) conforme a carga de novos clientes que entrarão no modo Auto-Serviço, além do vínculo dos IPs elásticos e domínios definitivos.
