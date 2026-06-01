# JDBC

JDBC (Java Database Connectivity) é a API padrão do Java para interação com bancos de dados relacionais. Fornece um conjunto de interfaces e classes para:

 * conectar-se a um banco de dados,
 * executar consultas SQL (SELECT) e atualizações (INSERT/UPDATE/DELETE),
 * recuperar resultados (ResultSet),
 * gerenciar transações,
 * trabalhar com tipos binários/texto grandes (BLOB/CLOB),
 * usar drivers específicos do fornecedor.
JDBC é agnóstico ao SGBD; cada banco fornece um driver JDBC que implementa as interfaces. A API principal está em java.sql e javax.sql.

### Arquitetura / Componentes
 * Driver JDBC: implementação do fornecedor (Oracle, PostgreSQL, MySQL, SQL Server, MariaDB, etc.). Normalmente um JAR que registra um Driver via ServiceLoader ou DriverManager.
 * DriverManager: gerencia drivers e cria conexões via DriverManager.getConnection(url, user, pass).
 * DataSource: alternativa mais moderna ao DriverManager; pode suportar pooling e ser configurada por container/DI.
 * Connection: objeto que representa uma sessão com o banco; controla autocommit, transações, criação de statements.
 * Statement / PreparedStatement / CallableStatement:
   * Statement: execução de SQL simples (sem parâmetros).
   * PreparedStatement: SQL pré-compilado com parâmetros (mais seguro e eficiente).
   * CallableStatement: chama stored procedures.
 * ResultSet: cursor sobre linhas retornadas por SELECT.
 * ResultSetMetaData, DatabaseMetaData: metadados.
 * SQLException: exceções SQL (com cadeia de causas e SQLState).

### Ciclo básico de uso (padrão)
1. Carregar driver (opcional em drivers modernos): Class.forName("org.postgresql.Driver");
2. Abrir conexão: Connection conn = DriverManager.getConnection(url, user, pass);
3. Criar PreparedStatement/Statement.
4. Executar (executeQuery para SELECT, executeUpdate para INSERT/UPDATE/DELETE, execute for múltiplos resultados).
5. Ler ResultSet.
6. Fechar ResultSet, Statement e Connection (try-with-resources).
7. Gerenciar transações se necessário (conn.setAutoCommit(false); conn.commit()/rollback()).

Exemplo mínimo (SELECT):
```java
String url = "jdbc:postgresql://localhost:5432/mydb";
try (Connection conn = DriverManager.getConnection(url, "user", "pass");
     PreparedStatement ps = conn.prepareStatement("SELECT id, name FROM person WHERE age > ?");
) {
    ps.setInt(1, 30);
    try (ResultSet rs = ps.executeQuery()) {
        while (rs.next()) {
            long id = rs.getLong("id");
            String name = rs.getString("name");
            // processa
        }
    }
} catch (SQLException e) {
    // tratar erro
}

```

### DriverManager vs DataSource
 * DriverManager: simples, bom para aplicações pequenas e scripts.
 * DataSource: recomendado em produção; suportado por containers/app servers e frameworks; pode prover pooling (por exemplo HikariCP, Apache DBCP, c3p0) e integração com JNDI.

Exemplo DataSource com HikariCP:
```java
HikariConfig cfg = new HikariConfig();
cfg.setJdbcUrl("jdbc:postgresql://localhost:5432/mydb");
cfg.setUsername("user");
cfg.setPassword("pass");
cfg.setMaximumPoolSize(10);
HikariDataSource ds = new HikariDataSource(cfg);

try (Connection conn = ds.getConnection()) {
    // ...
}
```

### PreparedStatement e prevenção de SQL Injection
 * Sempre usar PreparedStatement para parâmetros (não concatenar strings).
 * Exemplo:

```java
String sql = "INSERT INTO users(username, email) VALUES (?, ?)";
try (PreparedStatement ps = conn.prepareStatement(sql)) {
    ps.setString(1, username);
    ps.setString(2, email);
    ps.executeUpdate();
}
```

### Batch updates
 * Usar batching para múltiplos INSERT/UPDATE para reduzir round-trips:
```java
String sql = "INSERT INTO logs(msg, created_at) VALUES (?, ?)";
try (PreparedStatement ps = conn.prepareStatement(sql)) {
    conn.setAutoCommit(false);
    for (String m : messages) {
        ps.setString(1, m);
        ps.setTimestamp(2, Timestamp.from(Instant.now()));
        ps.addBatch();
    }
    int[] results = ps.executeBatch();
    conn.commit();
}
```

 * Ajuste de batch size (ex.: 1000) para não estourar memória.

### Transactions, isolation levels e savepoints
 * Autocommit padrão = true. Para controle manual:
```java
conn.setAutoCommit(false);
try {
    // várias operações
    conn.commit();
} catch (SQLException e) {
    conn.rollback();
}
```
* Isolation levels: Connection.TRANSACTION_READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE.

```java
conn.setTransactionIsolation(Connection.TRANSACTION_REPEATABLE_READ);
```
 * Savepoints:

```java
Savepoint sp = conn.setSavepoint("partial");
conn.rollback(sp); // volta até savepoint
```

### Performance: fetch size, auto-commit, prepared statement caching
 * fetchSize: diz ao driver quantas linhas buscar por vez (útil para grandes resultsets).

```java
ps.setFetchSize(1000);
```

 * Desative autocommit para operações em lote.
 * Use PreparedStatement e, se o driver/DB suportar, cache de prepared statements (HikariCP e drivers têm opções).
 * Use indexagem no DB e analise planos com EXPLAIN.

### Trabalhando com grandes objetos (BLOB/CLOB) e streaming
 * Use streams para não carregar tudo em memória:
  * Leitura de BLOB:

```java
try (InputStream in = rs.getBinaryStream("data")) {
    Files.copy(in, Paths.get("out.bin"), StandardCopyOption.REPLACE_EXISTING);
}
```

 * Escrita em BLOB via PreparedStatement.setBinaryStream or setBlob.
 * Para CLOB use getCharacterStream / setCharacterStream.
### ResultSet types e concurrency
 * Tipos: TYPE_FORWARD_ONLY, TYPE_SCROLL_INSENSITIVE, TYPE_SCROLL_SENSITIVE.
 * Concurrency: CONCUR_READ_ONLY, CONCUR_UPDATABLE.
Exemplo de cursor rolante:

```java
try (Statement st = conn.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_READ_ONLY)) {
    ResultSet rs = st.executeQuery("SELECT ...");
    rs.afterLast();
    while (rs.previous()) { ... }
}
```

### Stored procedures e CallableStatement
Chamar procedure:

```java
try (CallableStatement cs = conn.prepareCall("{ call increment_salary(?, ?) }")) {
    cs.setInt(1, empId);
    cs.registerOutParameter(2, Types.INTEGER);
    cs.execute();
    int newSalary = cs.getInt(2);
}
```

### Error handling (SQLException)
 * SQLException fornece:
   * getMessage(), getErrorCode(), getSQLState(), getNextException() (cadeia).
 * Trate erros transacionais com rollback.
 * Faça logs e categorize (deadlock, constraint violation, connection lost).

### Connection pooling (essencial em produção)
 * Use um pool maduro: HikariCP (preferido por performance), Apache DBCP, c3p0.
 * Pooling reduz overhead de abrir/fechar conexões.
 * Configurar:
   * maximumPoolSize, connectionTimeout, idleTimeout, maxLifetime.
 * Exemplo Hikari (já mostrado). Sempre feche Connection (retorna ao pool).

### XA / Distributed Transactions
 * Para transações distribuídas entre múltiplos recursos, use XADataSource e um Transaction Manager (Java EE/JTA).
 * Complexity alta; prefira sagas/compensações se possível.

### Concurrency e Threads
 * Connection geralmente não thread-safe; não compartilhe Connection simultaneamente entre threads.
 * DataSource e pool são thread-safe; cada thread requisita Connection separada.

### Mapeamento manual para objetos (DAO pattern)
 * DAO padrão: obter Connection (via DataSource), executar PreparedStatement, mapear ResultSet para POJO.
 * Exemplo simples:

```java
public User findById(DataSource ds, long id) throws SQLException {
    String sql = "SELECT id, name, email FROM users WHERE id = ?";
    try (Connection c = ds.getConnection();
         PreparedStatement ps = c.prepareStatement(sql)) {
        ps.setLong(1, id);
        try (ResultSet rs = ps.executeQuery()) {
            if (rs.next()) return new User(rs.getLong("id"), rs.getString("name"), rs.getString("email"));
            return null;
        }
    }
}
```

### Migração para ORM (JPA/Hibernate) — quando e porquê
 * Use JDBC direto quando precisar de controle fino, desempenho máximo em queries customizadas, ou para operações em massa.
 * ORM facilita produtividade, mapeamento objeto-relacional, caching de nível 1/2, queries HQL/Criteria, e lifecycle de entidades.
 * Considere usar ORM + acesso JDBC nativo para operações críticas (bulk, ETL).

### Segurança
 * Nunca concatene parametros em SQL — risco de SQL Injection.
 * Valide entradas, use least privilege (usuário DB com permissões mínimas).
 * Armazene credenciais em vaults/secret manager; não em código.
 * Use TLS para conexões remotas se suportado pelo driver/DB (jdbc:postgresql://host:port/db?sslmode=require).

### Boas práticas resumidas
 * Use DataSource + pool (HikariCP).
 * PreparedStatement sempre para parâmetros.
 * Defina timeouts (loginTimeout/connectionTimeout).
 * Controle transações explicitamente quando necessário (autoCommit=false).
 * Use fetchSize para grandes resultsets e Body streaming para blobs.
 * Feche recursos com try-with-resources.
 * Monitore o pool (connections in use, wait times).
 * Faça tratamento de erros e retries seletivos (idempotência).
 * Teste com dados reais e analise planos de execução.

### Exemplos práticos completos
1. INSERT em lote com transação e batch:
```java
String sql = "INSERT INTO orders(user_id, total) VALUES (?, ?)";
try (Connection conn = ds.getConnection();
     PreparedStatement ps = conn.prepareStatement(sql)) {
    conn.setAutoCommit(false);
    int batch = 0;
    for (Order o : orders) {
        ps.setLong(1, o.getUserId());
        ps.setBigDecimal(2, o.getTotal());
        ps.addBatch();
        if (++batch % 500 == 0) {
            ps.executeBatch();
            conn.commit();
        }
    }
    ps.executeBatch();
    conn.commit();
}
```

2. Streaming de grandes ResultSet para processar sem estourar memória (Postgres example):
```java
try (Connection conn = ds.getConnection();
     PreparedStatement ps = conn.prepareStatement("SELECT id, payload FROM big_table", ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY)) {
    ps.setFetchSize(1000);
    try (ResultSet rs = ps.executeQuery()) {
        while (rs.next()) {
            long id = rs.getLong("id");
            InputStream in = rs.getBinaryStream("payload");
            // processa stream
        }
    }
}
```

3. Leitura/Escrita de BLOB:


```java
// escrever
try (PreparedStatement ps = conn.prepareStatement("INSERT INTO files(name, data) VALUES (?, ?)")) {
    ps.setString(1, name);
    try (InputStream in = Files.newInputStream(path)) {
        ps.setBinaryStream(2, in, Files.size(path));
        ps.executeUpdate();
    }
}
// ler
try (PreparedStatement ps = conn.prepareStatement("SELECT data FROM files WHERE id = ?")) {
    ps.setLong(1, id);
    try (ResultSet rs = ps.executeQuery()) {
        if (rs.next()) {
            try (InputStream in = rs.getBinaryStream("data")) {
                Files.copy(in, Paths.get("out.bin"), StandardCopyOption.REPLACE_EXISTING);
            }
        }
    }
}
```
