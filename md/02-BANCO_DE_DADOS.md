# 02. Banco de Dados e Migrations

Este documento detalha o funcionamento do PostgreSQL dentro do SysFT e como interagimos com os dados.

---

## 1. Ausência de ORM e SQL Puro

Diferente de muitas aplicações modernas, o SysFT opta por **não utilizar ORMs** (como Prisma ou Sequelize). Todas as operações de banco de dados são feitas escrevendo SQL puro e seguro (usando consultas parametrizadas do driver `pg`).

### Vantagens dessa escolha no projeto:
- Máxima performance em Queries e Inserts em lote (ex: importações de milhares de monitoramentos).
- Sem curvas de aprendizado com abstrações de ORM complexas.
- Facilidade para escrever migrações "inline".

---

## 2. Padrão de "Auto-Migration" (ensureTable)

Para facilitar deploys e a criação de novas features, o sistema adota um padrão de migração no momento da execução, sem ferramentas externas.

Todo arquivo de Model (ex: `models/empresasModel.js`) possui uma função exportada chamada `createTable()`.

### Como Funciona:
1. Dentro de `createTable()`, sempre se utiliza o modificador `IF NOT EXISTS` para as tabelas.
2. Para lidar com novas colunas adicionadas futuramente a tabelas já existentes, o script faz uso do Bloco `DO $$ ... BEGIN ... EXCEPTION` do PostgreSQL para rodar comandos `ALTER TABLE ... ADD COLUMN` ignorando erros caso a coluna já exista.
3. No lado do Controller, costumamos rodar `await NomeDoModel.createTable()` de forma oportunista (ex: na primeira vez que um recurso é listado ou no momento do Bootstrap da aplicação em `app.js`).

**Exemplo de Padrão do Model:**
```javascript
const pool = require('../config/db');

module.exports = {
  createTable: async () => {
    // 1. Cria a Tabela se não existir
    await pool.query(`
      CREATE TABLE IF NOT EXISTS minha_tabela (
        id SERIAL PRIMARY KEY,
        empresa_id INTEGER REFERENCES empresas(id),
        nome VARCHAR(255) NOT NULL,
        criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `);

    // 2. Garante a adição de uma coluna nova (Update futuro)
    await pool.query(`
      DO $$
      BEGIN
        BEGIN
          ALTER TABLE minha_tabela ADD COLUMN descricao TEXT;
        EXCEPTION WHEN duplicate_column THEN RAISE NOTICE 'coluna descricao já existe';
        END;
      END $$;
    `);
  }
};
```

---

## 3. Prevenção de SQL Injection

**NUNCA** concatene variáveis diretamente na string SQL. O driver `pg` suporta consultas parametrizadas (`$1`, `$2`, etc), o que bloqueia 100% dos ataques de injeção de SQL.

**CORRETO:**
```javascript
const { rows } = await pool.query(
  'SELECT * FROM usuarios WHERE empresa_id = $1 AND email = $2',
  [empresaId, emailCliente]
);
```

**ERRADO (PROIBIDO):**
```javascript
// RISCO CRÍTICO DE SEGURANÇA
const { rows } = await pool.query(
  \`SELECT * FROM usuarios WHERE empresa_id = \${empresaId} AND email = '\${emailCliente}'\`
);
```

---

## 4. Gerenciamento do Pool e Transações

- As rotinas normais do sistema usam o objeto global `pool` (exportado de `config/db.js`), que gerencia automaticamente a abertura e o fechamento da conexão debaixo dos panos.
- Para operações que exigem **Transações** (múltiplas inserções dependentes que devem falhar juntas em caso de erro, ex: Inserir um Produto e logo depois seus Rótulos), você deve resgatar um cliente único (`pool.connect()`) e controlá-lo manualmente.

**Padrão de Transação:**
```javascript
const client = await pool.connect();
try {
  await client.query('BEGIN'); // Inicia a transação
  
  const res1 = await client.query('INSERT INTO tabela1 ... RETURNING id', [...]);
  const novoId = res1.rows[0].id;

  await client.query('INSERT INTO tabela2 (ref_id) VALUES ($1)', [novoId]);

  await client.query('COMMIT'); // Salva permanentemente no banco
} catch (error) {
  await client.query('ROLLBACK'); // Desfaz tudo em caso de erro no meio do caminho
  throw error;
} finally {
  client.release(); // IMPORTANTÍSSIMO: Devolve a conexão pro pool
}
```
