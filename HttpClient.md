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
