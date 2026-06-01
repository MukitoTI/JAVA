# Networking

### HttpURLConnection
  * É a API HTTP nativa de **Java** (desde JDK1.1) para abrir conexões HTTP/HTTPS. faz parte de java.net
  * É a Base em blocos de I/O clássicos (InputeStram/OutputStream) e não é reativa/não-assícrona.
  * Adequada para uso simples; para aplicações complexas ou alto desempenho prefira `java.net.http.HttpClient(Java 11+)` ou biblioteca externas (OkHttp, Apache HttpClient).


## Configuração e configuração básica

```java
URL url = new URL("https://exemple.com/path");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");   // "GET", "POST", "PUT", "DELET"...
conn.setConnectTimeout(10_000); // ms
conn.setReadTimeout(10_000);  // ms
conn.setDoInput(true);   // padrão true
conn.setDoOutput(false); // true se enviar corpo (POST/PUT)

```

  * Para HTTPS, a implementação retornada é HttpsURLConnection (subclasse).
  * Não chame connect() explicitamente na maioria dos casos; a conexão é aberta ao chamar getInputStream(), getOutputStream() ou getResponseCode().

## Cabeçalho de requisição
```java
conn.setRequestProperty("Accept", "application/json");
conn.setRequestProperty("User-Agent", "MyApp/1.0");
conn.setRequestProperty("Authorization", "Bearer TOKEN");

```

 * Alguns cabeçalhos são gerenciados pela JVM (e.g., Content-Length quando usar getOutputStream() corretamente) — cuidado ao definir manualmente.


## GET: leitura de requisição
```java
int status = conn.getResponseCode();
InputStream in = (status >= 200 && status < 400) ? conn.getInputStream() : conn.getErrorStream();
try (BufferedReader br = new BufferedReader(new InputStreamReader(in, StandardCharsets.UTF_8))) {
    StringBuilder sb = new StringBuilder();
    String line;
    while ((line = br.readLine()) != null) sb.append(line).append('\n');
    String body = sb.toString();
}
```

 * Use getErrorStream() para corpos de erro (4xx/5xx). getInputStream() lança IOException em respostas de erro.

## POST/PUT com corpo (form-urlencoded, JSON, multipart)
```java
conn.setRequestMethod("POST");
conn.setDoOutput(true);
conn.setRequestProperty("Content-Type", "application/json; charset=UTF-8");
byte[] payload = jsonString.getBytes(StandardCharsets.UTF_8);
conn.setRequestProperty("Content-Length", Integer.toString(payload.length));
try (OutputStream os = conn.getOutputStream()) {
    os.write(payload);
}
int status = conn.getResponseCode();
```
Form URL-encoded: use URLEncoder.encode para pares chave/valor.
Multipart (upload de arquivo): construa boundary, escreva partes manualmente — verboso; bibliotecas como Apache HttpClient/OkHttp facilitam.

## Streaming / uploads e downloads grandes
  *  Para uploads grandes, escreva diretamente no OutputStream sem materializar tudo em memória.
  *  Para downloads grandes, leia o InputStream em blocos (byte[]) e grave no disco.
  *  Configure setChunkedStreamingMode(int chunkSize) para streaming com Transfer-Encoding: chunked sem precisar Content-Length.

## Timeouts e conectividade
  * setConnectTimeout(ms) e setReadTimeout(ms) controlam bloqueios; sem eles, pode bloquear indefinidamente.
  * Não existe timeout para escrita no OutputStream antes do Java 8u... (variável); usar threads ou executors para cancelar se necessário.

## Redirecionamentos
 * Por padrão HttpURLConnection segue redirecionamentos para GET/HEAD automaticamente (setInstanceFollowRedirects(true) ou global via HttpURLConnection.setFollowRedirects(boolean)).
 * Para métodos como POST que retornam 302/303, a JVM pode converter em GET; comportamento pode variar; para controle manual, desative seguir redirecionamentos e trate códigos 3xx lendo header "Location".

## Autenticação 
 * Basic Auth: envie header Authorization: "Basic " + Base64.encode(user:pass).
```java
String encoded = Base64.getEncoder().encodeToString((user + ":" + pass).getBytes(StandardCharsets.UTF_8));
conn.setRequestProperty("Authorization", "Basic " + encoded);
```
* Digest, NTLM: não suportados nativamente — requerem implementações próprias ou bibliotecas.
* Proxy com autenticação: ajustar Proxy(Host, port) ao abrir conexão e setRequestProperty("Proxy-Authorization", ...), ou usar Authenticator.setDefault(...).

## Proxy e Authenticator
```java
Proxy proxy = new Proxy(Proxy.Type.HTTP, new InetSocketAddress("proxy.example.com", 3128));
HttpURLConnection conn = (HttpURLConnection) url.openConnection(proxy);

Authenticator.setDefault(new Authenticator() {
    protected PasswordAuthentication getPasswordAuthentication() {
        return new PasswordAuthentication("user", "pass".toCharArray());
    }
});

```
 * Authenticator é global à JVM.

## SSL/TLS (HttpsURLConnection)
  * Para confiança padrão, não é necessário configurar; para certs próprias, use custom TrustManager/SSLContext e chame ((HttpsURLConnection)conn).setSSLSocketFactory(...).
  * HostnameVerifier pode ser substituído (inseguro) para desenvolvimento.
  * Evite desativar verificação em produção.

## Keep-Alive, pooling e performance
 * JVM habilita HTTP keep-alive por padrão. O pooling é interno ao HTTP client da JVM; limitado comparado a bibliotecas modernas.
 * Para alto throughput/concorrência, usar java.net.http.HttpClient (Java 11+) ou OkHttp para melhor pooling, multiplexação e async.
 * Reaproveite conexões: não feche InputStream antes de consumir todo o corpo; chame disconnect() quando a conexão não for mais necessária (sinaliza que conexão pode ser fechada).

## Erros e códigos de resposta
 * getResponseCode() fornece o código; use getErrorStream() para corpo de erro.
 * Trate códigos 429 (rate limit), 503 (retry-after) com backoff exponencial.
 * Exceções comuns: UnknownHostException (DNS), SocketTimeoutException (timeout), SSLHandshakeException (TLS).

## Limitações conhecidas
 * Sem suporte nativo avançado: HTTP/2, multiplexação, websockets, streaming reativo.
 * API antiga — verbosa para multipart, cookies e autenticação complexa.
 * Comportamentos sutis entre JVMs/versões (redirecionamento de POST, cabeçalhos gerenciados).

## Boas práticas
 * Use timeouts (connect + read).
 * Feche streams e conexões (try-with-resources).
 * Para JSON, defina Content-Type e envie UTF-8.
 * Para upload grande, use chunked streaming.
 * Trate redirecionamentos manualmente quando precisar preservar método/headers.
 * Não armazene credenciais em código; use vaults/env.
 * Prefira HttpClient (Java 11+) ou bibliotecas maduras para casos complexos.
 * Teste com servidores reais e ferramentas (curl, mitmproxy) para verificar cabeçalhos/fluxo.


## Exemplo completo: POST JSON e parse JSON (simples)
```java
URL url = new URL("https://api.example.com/items");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("POST");
conn.setConnectTimeout(10_000);
conn.setReadTimeout(10_000);
conn.setDoOutput(true);
conn.setRequestProperty("Content-Type", "application/json; charset=UTF-8");
conn.setRequestProperty("Accept", "application/json");

String json = "{\"name\":\"x\",\"qty\":1}";
try (OutputStream os = conn.getOutputStream()) {
    os.write(json.getBytes(StandardCharsets.UTF_8));
}

int status = conn.getResponseCode();
InputStream in = (status >= 200 && status < 300) ? conn.getInputStream() : conn.getErrorStream();
try (BufferedReader br = new BufferedReader(new InputStreamReader(in, StandardCharsets.UTF_8))) {
    StringBuilder sb = new StringBuilder();
    String line;
    while ((line = br.readLine()) != null) sb.append(line);
    String responseBody = sb.toString();
    // parse com sua lib JSON preferida (Jackson/Gson)
}
conn.disconnect();

```

## Alternativas e quando migrar
 * java.net.http.HttpClient (Java 11+): HTTP/2, async, fluent builder, melhor performance.
 * OkHttp: popular, HTTP/2, WebSockets, conexão/reciclagem eficiente.
 * Apache HttpClient: rico em recursos, configurável, bom para casos corporativos.
 * Use HttpURLConnection para simplicidade e compatibilidade com Java 8 e anteriores; migre para HttpClient/OkHttp para recursos modernos.








