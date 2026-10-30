Guia Completo de Estudos: Desenvolvimento Web, Cybersecurity, Chrome DevTools e Redes de Computadores

Apostila Universitária e Manual Técnico de Referência Material didático completo de estudo, revisão e consulta prática para avaliações acadêmicas e auditorias técnicas em Desenvolvimento Web, Inspeção Avançada com Google Chrome DevTools (Network, Application e Console), Segurança de Aplicações Web (Cybersecurity / OWASP) e Redes de Computadores.

Como Utilizar Esta Apostila e Convenções Adotadas

Para garantir precisão técnica e diferenciar claramente exigências estritas de decisões arquiteturais dependentes de contexto, esta apostila utiliza as seguintes marcações padronizadas ao longo de todas as seções:

[REQUISITO]: Controle ou configuração obrigatória para atingir um patamar mínimo aceitável de segurança no cenário descrito (por exemplo, trafegar credenciais exclusivamente sob HTTPS/TLS e utilizar consultas parametrizadas contra SQL Injection).
[RECOMENDAÇÃO]: Prática fortemente aconselhada pelos padrões de mercado (OWASP, IETF, MDN), mas cuja aplicação exata depende da arquitetura do sistema (por exemplo, diretivas específicas de Content-Security-Policy ou escolha entre SameSite=Strict e SameSite=Lax).
[BOA PRÁTICA]: Medida de defesa em profundidade (Defense in Depth), higiene de código, observabilidade ou padronização que reduz a superfície de exposição e facilita a manutenção segura.
[Varia por versão]: Funcionalidade, coluna, filtro ou comportamento do navegador Google Chrome / Chrome DevTools cuja nomenclatura, localização visual ou suporte muda conforme a versão do navegador ou sistema operacional.
[Obsoleto]: Protocolo, algoritmo, cabeçalho ou atributo depreciado por normas técnicas (RFCs/W3C/WHATWG) — mantido na explicação exclusivamente para fins de diagnóstico de sistemas legados e resolução de questões de prova.

IMPORTANT

Nota Ética e Legal sobre Segurança da Informação: Todos os conceitos de vulnerabilidades, vetores de teste e exemplos de análise apresentados nesta apostila têm finalidade estritamente acadêmica, defensiva e educacional. Testes práticos de segurança devem ser realizados única e exclusivamente em ambientes locais de laboratório (como OWASP Juice Shop, OWASP WebGoat, DVWA e PortSwigger Web Security Academy) ou em sistemas para os quais você possua autorização formal e expressa. No Brasil, a invasão ou teste não autorizado de dispositivos informáticos constitui crime previsto no Art. 154-A do Código Penal (Lei nº 12.737/2012 e Lei nº 14.155/2021).

📚 Acesso Rápido aos Capítulos (Renderização Completa no GitHub)

Todos os arquivos abaixo têm menos de 350 KB para garantir que o GitHub exiba 100% do conteúdo na tela sem ocultar nada:

Capítulo	Arquivo Navegável	Conteúdo Principal
01A	01a-fundamentos-web-parte1.md	Arquitetura Cliente-Servidor, URL, DNS, IP, TCP, UDP, HTTP/HTTPS, Métodos e Status HTTP, Headers
01B	01b-fundamentos-web-parte2.md	Cookies, Sessões, JWT, Autenticação, HTML/CSS/JS, DOM/BOM, REST/JSON, CORS/SOP, Storage e Cache
02A	02a-devtools-network.md	Chrome DevTools: Aba Network Completa (Controles, Filtros, Colunas, Timing, TTFB, Segurança)
02B	02b-devtools-application.md	Chrome DevTools: Aba Application Completa (Cookies, Local/Session Storage, IndexedDB, SW, Cache)
02C	02c-devtools-console.md	Chrome DevTools: Aba Console Completa (Níveis, Filtros, API console, Erros e Diagnóstico XSS/CSP)
03A	03a-vulnerabilidades-parte1.md	OWASP Top 10 (2021/2025), SQLi, XSS, CSRF, SSRF, Broken Access Control, Auth, Misconfig, Crypto, IDOR
03B	03b-vulnerabilidades-parte2.md	Directory/Path Traversal, File Upload, Clickjacking, Open Redirect, Session Hijacking/Fixation, Cmd Injection, Race Conditions
04	04-redes-de-computadores.md	Modelo OSI, TCP/IP, Handshake, IPv4/IPv6, CIDR/Sub-redes, ARP, DHCP, DNS, NAT, Firewall, Proxy, VPN, TLS e Portas
05	05-headers-de-seguranca.md	CSP, HSTS, X-Content-Type-Options, X-Frame-Options, Referrer-Policy, Permissions-Policy, COOP/CORP/COEP, CORS
06	06-checklist-e-comparativo.md	Checklist Prático de Auditoria pelo Navegador + Tabelas Comparativas Seguro vs. Inseguro
07	07-glossario-tecnico.md	Glossário Técnico Completo de A a Z (152 Termos Detalhados)
08	08-simulado-e-referencias.md	Simulado Completo para a Prova (65 Questões) + Gabarito Comentado + Fontes Oficiais

Sumário Navegável Completo
1. Fundamentos de Desenvolvimento Web
1.1 Arquitetura cliente-servidor
1.1.1 Conceito e papéis
1.1.2 Front-end e back-end
1.1.3 Camadas e modelos de 2 e 3 camadas
1.1.4 HTTP é stateless: como se mantém estado
1.1.5 Intermediários: proxy reverso, CDN e balanceador de carga
1.2 O que acontece quando acessamos uma URL
1.2.1 Anatomia da URL
1.2.2 Passo a passo da navegação
1.2.3 Relação com a aba Network e o painel Timing
1.3 DNS, IP, TCP, UDP, HTTP e HTTPS
1.3.1 DNS (Domain Name System)
1.3.2 IP (Internet Protocol)
1.3.3 TCP (Transmission Control Protocol)
1.3.4 UDP (User Datagram Protocol)
1.3.5 HTTP (Hypertext Transfer Protocol)
1.3.6 HTTPS (HTTP over TLS)
1.4 Métodos HTTP
1.4.1 GET
1.4.2 POST
1.4.3 PUT
1.4.4 PATCH
1.4.5 DELETE
1.4.6 OPTIONS
1.4.7 HEAD
1.4.8 Outros métodos (CONNECT, TRACE) e tabela comparativa
1.5 Códigos de Status HTTP
1.5.1 Classes de status (1xx a 5xx)
1.5.2 Códigos de status detalhados
1.5.3 Tabela-resumo de códigos HTTP
1.5.4 Pegadinhas clássicas de prova sobre status HTTP
1.6 Headers HTTP de Requisição e Resposta
1.6.1 Estrutura e funcionamento dos headers
1.6.2 Principais headers de requisição
1.6.3 Principais headers de resposta
1.6.4 Problemas de segurança em headers e configuração segura
1.7 Cookies, sessões, tokens e autenticação
1.7.1 Cookies
1.7.2 Sessões no servidor
1.7.3 Tokens e JWT (JSON Web Token)
1.7.4 Autenticação e autorização
1.8 HTML, CSS e JavaScript
1.8.1 HTML (HyperText Markup Language)
1.8.2 CSS (Cascading Style Sheets)
1.8.3 JavaScript
1.9 DOM e BOM
1.9.1 DOM (Document Object Model)
1.9.2 BOM (Browser Object Model)
1.10 APIs REST e JSON
1.11 CORS e Same-Origin Policy (SOP)
1.11.1 Same-Origin Policy (SOP)
1.11.2 CORS (Cross-Origin Resource Sharing)
1.11.3 Problemas de segurança em CORS e configuração segura
1.12 Local Storage, Session Storage e IndexedDB
1.12.1 Local Storage
1.12.2 Session Storage
1.12.3 IndexedDB
1.12.4 Comparativo de armazenamento no cliente e segurança
1.13 Cache do Navegador
1.13.1 Funcionamento e camadas de cache
1.13.2 Controle e validação de cache
1.13.3 Problemas de segurança em cache e configuração segura
2. Guia Completo do Chrome DevTools
2.1 Network
2.1.1 Visão geral da interface da aba Network
2.1.2 Controles da barra superior
2.1.3 Filtros da aba Network
2.1.4 Colunas da lista de requisições
2.1.5 Painel de detalhes da requisição
2.1.6 Performance e rede: fases de Timing
2.1.7 Protocolos HTTP/1.1, HTTP/2 e HTTP/3 no DevTools
2.1.8 Transfer Size, Resource Size, Cache e Service Workers
2.1.9 Análise de Segurança na Aba Network (Casos Críticos)
2.2 Application
2.2.1 Grupo Application
2.2.2 Visão geral do grupo Storage
2.2.3 Local Storage e Session Storage
2.2.4 IndexedDB
2.2.5 Cookies na aba Application
2.2.6 Aprofundamento em Segurança de Cookies
2.2.7 Service Workers e Cache Storage
2.2.8 Demais recursos da aba Application
2.3 Console
2.3.1 Tipos de mensagens no Console
2.3.2 Recursos, controles e filtros da aba Console
2.3.3 JavaScript, métodos do objeto console e execução no REPL
2.3.4 Erros e exceções no Console
2.3.5 Segurança na aba Console (Diagnóstico e Código Vulnerável vs. Corrigido)
3. Cybersecurity: Vulnerabilidades Web
3.0 Panorama do OWASP Top 10
3.0.1 Evolução do OWASP Top 10: de 2021 para 2025
3.0.2 Mapeamento das Vulnerabilidades desta Seção para o OWASP Top 10
3.1 SQL Injection (SQLi)
3.2 Cross-Site Scripting (XSS — Refletido, Armazenado e DOM-Based)
3.3 Cross-Site Request Forgery (CSRF)
3.4 Server-Side Request Forgery (SSRF)
3.5 Broken Access Control (Quebra de Controle de Acesso)
3.6 Authentication Failures (Falhas de Autenticação)
3.7 Security Misconfiguration (Configuração Incorreta de Segurança)
3.8 Cryptographic Failures (Falhas Criptográficas)
3.9 Insecure Direct Object Reference (IDOR / BOLA)
3.10 Directory Traversal (e Directory Listing / Exposição de Diretórios)
3.11 File Upload Vulnerabilities (Vulnerabilidades de Upload de Arquivos)
3.12 Clickjacking (UI Redressing)
3.13 Open Redirect (Redirecionamento Aberto)
3.14 Session Hijacking (Sequestro de Sessão)
3.15 Session Fixation (Fixação de Sessão)
3.16 Command Injection (Injeção de Comandos do Sistema Operacional)
3.17 Path Traversal (Travessia de Caminho e Canonicalização de Arquivos)
3.18 Race Conditions (Condições de Corrida / TOCTOU)
3.19 Matriz Comparativa das 18 Vulnerabilidades Web
4. Redes de Computadores
4.1 Modelo OSI e suas sete camadas
4.2 Modelo TCP/IP e comparativo com o Modelo OSI
4.3 Camada de Transporte: TCP, UDP e Handshake
4.3.1 Diferenças entre TCP e UDP
4.3.2 Handshake TCP (Three-Way Handshake) e encerramento de conexão
4.4 Endereçamento IP, Sub-redes e Roteamento
4.4.1 IPv4 e IPv6
4.4.2 Endereço IP público e privado (RFC 1918 e RFC 4193)
4.4.3 Máscara de rede, Sub-redes e CIDR
4.4.4 Gateway Padrão e Roteamento
4.5 Protocolos Essenciais de Rede e Enlace
4.5.1 MAC Address e ARP (Address Resolution Protocol)
4.5.2 DHCP (Dynamic Host Configuration Protocol)
4.5.3 DNS (Domain Name System) na infraestrutura de rede
4.5.4 NAT (Network Address Translation)
4.5.5 ICMP (Internet Control Message Protocol)
4.6 Segurança de Perímetro e Criptografia em Trânsito
4.6.1 Firewall
4.6.2 Proxy (Forward Proxy vs Reverse Proxy)
4.6.3 VPN (Virtual Private Network)
4.6.4 TLS/SSL, HTTP e HTTPS
4.7 Portas de Rede e Tabela Completa de Serviços
4.7.1 Conceito de Porta, Socket e Faixas da IANA
4.7.2 Tabela Completa de Portas e Serviços de Rede
4.7.3 Análise Detalhada Individual de Portas e Serviços
5. Headers de Segurança HTTP
5.1 Content-Security-Policy (CSP)
5.2 Strict-Transport-Security (HSTS)
5.3 X-Content-Type-Options
5.4 X-Frame-Options
5.5 Referrer-Policy
5.6 Permissions-Policy
5.7 Cross-Origin-Opener-Policy (COOP)
5.8 Cross-Origin-Resource-Policy (CORP)
5.9 Cross-Origin-Embedder-Policy (COEP)
5.10 Access-Control-Allow-Origin
5.11 Access-Control-Allow-Credentials
5.12 Cache-Control
5.13 Tabela-Resumo dos Headers de Segurança HTTP
6. Como Identificar se uma Aplicação Web é Segura (Checklist de Auditoria no Navegador)
6.1 Auditoria na Aba Network
6.1.1 O site utiliza HTTPS em todas as requisições?
6.1.2 Existem dados sensíveis nas URLs (query strings e paths)?
6.1.3 Os cookies enviados e recebidos possuem atributos adequados?
6.1.4 Existem headers de segurança nas respostas?
6.1.5 As APIs retornam somente os dados estritamente necessários?
6.1.6 Existem respostas com informações internas do servidor ou banco?
6.1.7 Há problemas ou permissividade excessiva de CORS?
6.2 Auditoria na Aba Application
6.2.1 Existem tokens sensíveis armazenados no Local Storage, Session Storage ou IndexedDB?
6.2.2 Os cookies estão configurados corretamente na tabela de Cookies?
6.2.3 Existem informações pessoais (PII) desnecessárias persistidas no cliente?
6.2.4 O Service Worker apresenta comportamento esperado e seguro?
6.2.5 Há dados sensíveis armazenados em Cache Storage ou HTTP Cache?
6.3 Auditoria na Aba Console
6.3.1 Existem erros que revelam informações internas ou stack traces?
6.3.2 Há mensagens de log com dados sensíveis, tokens ou credenciais?
6.3.3 Existem violações de Content Security Policy (CSP)?
6.3.4 Existem erros de autenticação, autorização ou sessões expiradas mal tratadas?
6.3.5 Existem problemas de JavaScript que podem afetar a segurança?
6.4 Limitações da Análise pelo Navegador (Caixa-Preta vs. Caixa-Branca)
7. Comparativo: Seguro vs. Inseguro
7.1 Transporte, Protocolos, Métodos e URLs
7.2 Cookies, Sessões, Tokens (JWT) e Autenticação
7.3 Armazenamento no Navegador e Cache
7.4 Headers de Segurança, CORS e Isolamento
7.5 Código Front-End (HTML, CSS, JavaScript e DOM)
7.6 Código Back-End, Banco de Dados, Upload e APIs REST
7.7 Infraestrutura, Portas e Redes de Computadores
8. Glossário Técnico (A–Z)
8.1 Letra A
8.2 Letra B
8.3 Letra C
8.4 Letra D
8.5 Letra E
8.6 Letra F
8.7 Letra G
8.8 Letra H
8.9 Letra I
8.10 Letra J
8.11 Letra K
8.12 Letra L
8.13 Letra M
8.14 Letra N
8.15 Letra O
8.16 Letra P
8.17 Letra Q
8.18 Letra R
8.19 Letra S
8.20 Letra T
8.21 Letra U
8.22 Letra V
8.23 Letra W
8.24 Letra X
8.25 Letras Y e Z
8.26 Quadro Comparativo de Siglas Parecidas (Anti-Pegadinhas de Prova)
9. Simulado Completo para a Prova
9.1 Questões de Múltipla Escolha (Questões 1 a 30)
9.1.1 Nível Básico (Questões 1 a 10)
9.1.2 Nível Intermediário (Questões 11 a 20)
9.1.3 Nível Avançado (Questões 21 a 30)
9.2 Questões de Verdadeiro ou Falso (Questões 31 a 45)
9.3 Questões Discursivas (Questões 46 a 55)
9.4 Questões Práticas de Interpretação do Chrome DevTools (Questões 56 a 65)
9.5 Gabarito Oficial e Resolução Comentada (Questões 1 a 65)
9.5.1 Tabela de Respostas Rápidas (Questões 1 a 45)
9.5.2 Resolução Comentada: Múltipla Escolha — Nível Básico (Questões 1 a 10)
9.5.3 Resolução Comentada: Múltipla Escolha — Nível Intermediário (Questões 11 a 20)
9.5.4 Resolução Comentada: Múltipla Escolha — Nível Avançado (Questões 21 a 30)
9.5.5 Resolução Comentada: Verdadeiro ou Falso (Questões 31 a 45)
9.5.6 Padrão de Resposta da Banca: Questões Discursivas (Questões 46 a 55)
9.5.7 Padrão de Resposta da Banca: Interpretação do Chrome DevTools (Questões 56 a 65)
10. Fontes e Referências Bibliográficas
10.1 Documentação Oficial do Chrome DevTools e Google Chrome
10.2 OWASP (Open Worldwide Application Security Project)
10.3 MDN Web Docs e Google Web Fundamentals (web.dev)
10.4 RFCs Oficiais da IETF (HTTP, TLS, DNS, TCP/IP, Cookies, JWT, OAuth)
10.5 Documentação Oficial da Microsoft, Cisco, W3C e WHATWG
10.6 Livros Clássicos e Ambientes de Laboratório Recomendados
10.6.1 Bibliografia Clássica de Referência
10.6.2 Ambientes de Laboratório Autorizados e Gratuitos para Prática
