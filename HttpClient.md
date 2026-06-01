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

### 12) Proxy
 * Configure ProxySelector (global ou por cliente):
```java
HttpClient.newBuilder()
    .proxy(ProxySelector.of(new InetSocketAddress("proxy.example.com", 3128)))
    .build();
```
 * Proxy authentication via Authenticator.

### 13) Cookies
 * HttpClient não armazena cookies por padrão; forneça CookieHandler (CookieManager) ao builder:

```java
CookieManager cm = new CookieManager();
cm.setCookiePolicy(CookiePolicy.ACCEPT_ALL);
HttpClient.newBuilder().cookieHandler(cm).build();

```

### 14) WebSocket
 * API WebSocket integrada (java.net.http.WebSocket) com modelo assíncrono e listener:

```java
CompletableFuture<WebSocket> wsFuture = client.newWebSocketBuilder()
    .buildAsync(URI.create("wss://example.com/ws"), listener);
```

### 15) Interceptores / filtros
 * Não existe interceptor nativo como OkHttp. Padrões:
   * Construir wrapper em torno de send/sendAsync para aplicar lógica (retry, logging).
   * Usar Executor customizado e factory de requests.
   * Para instrumentação, envolver BodyHandlers/Publishers.


### 16) Retries e backoff
 * Implemente em camadas: wrapper que reenvia em falhas transitórias (IOExceptions, 5xx, 429) com backoff exponencial e jitter; respeite headers Retry-After quando presentes.

### 17) Tratamento de erros e códigos HTTP
 * client.send lança IOException/InterruptedException; validar statusCode via response.statusCode().
 * Use BodyHandlers.ofString() para ler corpo de erro.
 * Trate especificidades: 401/403 (auth), 429/503 (retry-after), 3xx (redirecionamento).

### 18) Performance e pooling
 * HttpClient gerencia conexões internamente e reaproveita conexões HTTP/1.1 keep-alive e HTTP/2 multiplexadas.
 * Reaproveite instâncias de HttpClient.
 * Ajuste Executor para controlar threads de callbacks.
 * Para alto throughput, prefira HTTP/2 onde possível e limite concorrência por host externamente.
### 19) Monitoramento e métricas
 * Não há métricas embutidas; adicione wrappers para:
  * Latência por request (System.nanoTime).
  * Contadores de sucesso/falha.
  * Tamanho de payloads.
  * Integre com micrometer/Prometheus via instrumentação manual.
### 20) Comparativo rápido (quando escolher qual)
 * HttpURLConnection: use apenas se precisar de compatibilidade com JDK <11; API antiga e limitada.
 * java.net.http.HttpClient: escolha padrão para Java 11+ — integração JDK, HTTP/2, async.
 * OkHttp: se precisar de interceptors fáceis, multipart simples, ótima performance em mobile/Android.
 * Apache HttpClient: se precisar de recursos corporativos avançados e compatibilidade com ecossistemas antigos. Tabela resumida (breve):
 * Simplicidade/Disponibilidade: HttpClient (JDK11+)
 * HTTP/2 & WebSocket: HttpClient, OkHttp
 * Interceptors & Multipart fáceis: OkHttp, Apache
 * Alto controle e customização: Apache

### 21) Exemplos práticos (concisos)
GET sync:
```java
HttpRequest r = HttpRequest.newBuilder(URI.create("https://httpbin.org/get")).GET().build();
HttpResponse<String> s = client.send(r, BodyHandlers.ofString());
```
POST JSON async:
```java
HttpRequest r = HttpRequest.newBuilder(URI.create("https://api.example.com"))
    .POST(BodyPublishers.ofString("{\"name\":\"x\"}", StandardCharsets.UTF_8))
    .header("Content-Type","application/json")
    .build();
client.sendAsync(r, BodyHandlers.ofString())
      .thenApply(HttpResponse::body)
      .thenAccept(System.out::println);
```

Download grande streaming:
```java
HttpRequest r = HttpRequest.newBuilder(URI.create(url)).GET().build();
client.send(r, BodyHandlers.ofFile(Paths.get("out.bin")));
```
Retry simples com CompletableFuture (exponencial): implementar wrapper que chama sendAsync e em caso de falha re-chama com delay via ScheduledExecutorService.

### 22) Boas práticas resumidas
 * Reutilize HttpClient.
 * Defina timeouts (connect + per-request).
 * Use BodyHandlers.ofFile/ofInputStream para payloads grandes.
 * Use CookieManager se precisar manter cookies.
 * Não confie em autoverificação de redirecionamento para preservar headers sensíveis.
 * Implementar retries com backoff e respeitar Retry-After.
 * Instrumente latências e taxas de erro.
 * Para multipart/complex uploads, prefira bibliotecas prontas se não quiser construir boundary manualmente.








