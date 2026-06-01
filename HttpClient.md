## HTTPCLIENT

Uma cobertura completa e prática de **java.net.http.HttpClient** (Java 11+) incluindo comparação com **HttpURLConnection** e bibliotecas populares (OkHttp, Apache HttpClient), 
arquitetura, uso (sync/async), builders, BodyHandlers/Publishers, HTTP/2, WebSocket, TLS/SSL customizado, proxies, autenticação, cookies, redirecionamentos, streaming, 
multipart, retries/backoff, performance e boas práticas — com exemplos práticos. Java 17. Estrutura que seguirei:

### 1) Visão geral e motivação
 * java.net.http.HttpClient foi introduzido no Java 11 para substituir a API antiga (HttpURLConnection) oferecendo:
   * API moderna, fluente e tipada.
   * Suporte nativo a HTTP/2 e priorização de requisições.
   * Programação assíncrona com CompletableFuture.
   * BodyPublisher/BodyHandler para tipos variados (String, byte[], InputStream, File, Multipart via publisher).
   * Melhor gerenciamento de conexões, pipelining e multiplexação.
   * WebSocket client integrado.
 * Relação com HttpURLConnection:
   * HttpURLConnection é a API histórica (java.net), baseada em streams blocking e com menos recursos.
   * HttpClient substitui e supera HttpURLConnection em recursos, performance e ergonomia; use HttpURLConnection apenas por compatibilidade ou JDK <11.
 * Relação com bibliotecas externas:
   * OkHttp e Apache HttpClient ainda são muito usados: OkHttp destaca-se por performance, suporte HTTP/2/WebSocket, interceptors; Apache HttpClient é altamente configurável em cenários corporativos. java.net.http.HttpClient é agora suficiente para a maioria dos casos e tem a vantagem de ser parte do JDK. 


### 2) Criação e configuração básica
Exemplo mínimo:

```java
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)            // HTTP_2 or HTTP_1_1
    .connectTimeout(Duration.ofSeconds(10))
    .followRedirects(HttpClient.Redirect.NORMAL)  // NEVER, NORMAL, ALWAYS
    .executor(Executors.newFixedThreadPool(4))    // opcional
    .build();
```
  * Cliente imutável e thread-safe; reutilize instâncias.
  * .version() é preferência; o cliente negociará com o servidor.
  * .followRedirects() controla comportamento.

###  3) Enviar requisições (sync e async)
Construir request:
```java
HttpRequest req = HttpRequest.newBuilder()
    .uri(URI.create("https://example.com/api"))
    .timeout(Duration.ofSeconds(20))
    .header("Accept", "application/json")
    .GET() // ou .POST(BodyPublishers.ofString(json))
    .build();
```

Síncrono:
```java
HttpResponse<String> resp = client.send(req, BodyHandlers.ofString());
int status = resp.statusCode();
String body = resp.body();
```

Assíncrono:
```java
CompletableFuture<HttpResponse<String>> future = client.sendAsync(req, BodyHandlers.ofString());
future.thenAccept(r -> { /* process r */ });
```
### 4) BodyPublishers e BodyHandlers principais
 * BodyPublishers: ofString, ofByteArray, ofInputStream, ofFile, ofPublisher (reactive), noBody, ofLines.
 * BodyHandlers: ofString, ofByteArray, ofInputStream, ofFile, ofLines, discarding. Exemplos:
 * Upload de arquivo:
```java
HttpRequest req = HttpRequest.newBuilder()
    .uri(URI.create("https://example.com/upload"))
    .POST(BodyPublishers.ofFile(Paths.get("path/to/file")))
    .build();
```

 * Download streaming para arquivo sem carregar tudo em memória:
```java
HttpResponse<Path> resp = client.send(req, BodyHandlers.ofFile(Paths.get("out.bin")));
```

### 5) Multipart/form-data (upload de arquivos)
Não há helper built-in para multipart em JDK; construa manualmente ou use BodyPublishers.ofByteArray/ofInputStream/ofPublisher. Exemplo rápido (boundary manual):

 * Montar partes com boundary, cabeçalhos Content-Disposition, Content-Type e concatenar publishers; para simplicidade, use bibliotecas (Apache HttpComponents, OkHttp) em uploads complexos.
### 6) Streaming reativo e backpressure
 * BodyPublishers.ofInputStream e ofPublisher(Flow.Publisher) permitem integrar com APIs reativas.
 * BodyHandlers.ofInputStream retorna InputStream; é útil para processamento incremental.
### 7) Timeouts e cancelamento
 * Connect timeout: no builder via connectTimeout.
 * Per-request timeout: HttpRequest.newBuilder().timeout(...)
 * Cancelamento: para async, future.cancel(true) cancela a operação; para sync, você precisa rodar em outra thread e cancelar via interrupt.
### 8) Redirecionamentos
 * Controlado por client builder (Redirect.NEVER/NORMAL/ALWAYS). NORMAL segue redirecionamentos para GET/HEAD e preserva segurança; comportamento compatível com RFCs.
### 9) HTTP/2 e multiplexação
 * HttpClient suporta HTTP/2 nativamente quando disponível; multiplexação permite múltiplas requisições sobre a mesma conexão, reduz latência.
 * Para HTTP/2 servidores com ALPN via TLS; sem TLS, HTTP/2 com priorização dependente do servidor.
### 10) TLS/SSL customizado e verificação de hostname
 * Para custom TrustStore/KeyStore:
```java
SSLContext sslCtx = SSLContext.getInstance("TLS");
sslCtx.init(km, tm, new SecureRandom());
HttpClient client = HttpClient.newBuilder()
    .sslContext(sslCtx)
    .sslParameters(new SSLParameters()) // opcional
    .build();
```

 * HostnameVerifier não é exposto diretamente; para testes você pode usar custom TrustManager que aceita qualquer hostname (inseguro) — evite em produção.

### 11) Autenticação
 * Basic Auth: adicionar header Authorization com Base64.
 * OAuth: trate via headers Bearer e renove tokens externamente.
 * Kerberos/NTLM: não suportado nativamente — bibliotecas externas/JAAS/Config extra são necessárias.
 * Autenticator (Authenticator): suporte a autenticação básica para proxies e servidores:

```java
Authenticator auth = new Authenticator() {
  protected PasswordAuthentication getPasswordAuthentication() {
    return new PasswordAuthentication("user", "pass".toCharArray());
  }
};
HttpClient.newBuilder().authenticator(auth).build();
```


