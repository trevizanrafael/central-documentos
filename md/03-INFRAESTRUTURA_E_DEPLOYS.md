# 03. Infraestrutura, Deploys e AWS

Este documento registra como manter o SysFT rodando em produção (Servidores Ubuntu na AWS Lightsail) e as configurações de domínio e deploy.

---

## 1. Arquitetura da Infraestrutura
O SaaS opera em dois servidores separados para garantir segurança de dados e viabilizar desenvolvimento contínuo:

1. **Testes/Homologação (`pactestes.foodtech.net.br`):**
   - Máquina menor na AWS ($12 / 2GB RAM).
   - Usada para desenvolvedores testarem novas funcionalidades antes de levar ao cliente.
   - O banco de dados pode (e deve) ser resetado ou sobrescrito regularmente.
2. **Produção (`pacdigital.foodtech.net.br` / `app.foodtech.net.br`):**
   - Máquina principal na AWS ($24 / 4GB RAM).
   - Onde residem os dados reais dos clientes. O banco de dados nunca deve ser limpo de forma bruta.
   - Operações intensas (Geração de PDFs, Importação XLS) dependem da CPU/RAM dessa máquina.

---

## 2. Deploy Básico e Gerenciador de Processos (PM2)

O aplicativo Node.js é mantido ativo e em monitoramento via `pm2`, que garante que o servidor seja reiniciado em caso de crash (falha no código Node) ou restart físico do servidor.

### Fazendo Deploy de Atualizações
Sempre que fizer push das novidades no GitHub, o processo manual (por enquanto) no servidor para aplicar a atualização é:

```bash
# 1. Entre na pasta do sistema
cd ~/sysft

# 2. Puxe o código do repositório
git pull origin main

# 3. Se instalou novos pacotes no package.json, rode:
npm install

# 4. Reinicie a aplicação via PM2
pm2 restart all
```

*Se houver alteração em variáveis do arquivo `.env` da máquina, você deve rodar `pm2 restart all --update-env` para que o Node carregue as novas chaves.*

---

## 3. Cloudflare e Redes

A comunicação com o cliente é intermediada pelo Cloudflare na nuvem (DNS e Proxy).

- **SSL (Cadeado Verde):** Tratado nativamente pelo Cloudflare em sua modalidade *Flexible* ou *Full*, eliminando a necessidade de instalar Let's Encrypt diretamente no NodeJS/Nginx para tráfego externo.
- **Firewall e DDos:** Regras gratuitas do Cloudflare protegem contra ataques comuns.
- **Problemas de Timeout:** A importação de grandes planilhas XLS (arquivos muito grandes) pode bater no limite de Timeout Gratuito do Cloudflare (100 segundos). Por isso, as rotinas de importação do SysFT quebram os dados em pequenos lotes (batches de 500) enviando `res.write()` para manter a conexão HTTP ativa e evitar erro de Gateway Timeout (524).

---

## 4. Dica de DBA: Setup Novo Banco na AWS
Caso seja necessário formatar e trocar o banco num ambiente em branco:
1. Veja seu usuário atual: `grep DB_USER .env`
2. Conecte como Postgres: `sudo -u postgres psql`
3. Crie: `CREATE DATABASE meu_novo_banco OWNER meu_usuario;`
4. Atualize o `.env` e dê o Restart do PM2.
