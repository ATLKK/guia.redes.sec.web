> [⬅️ 3B. Vulnerabilidades Web (3.10 a 3.19)](./03b-vulnerabilidades-parte2.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [5. Headers de Segurança HTTP ➡️](./05-headers-de-seguranca.md)

---

<a id="s4"></a>
## 4. Redes de Computadores

O funcionamento da Web moderna, a performance percebida no navegador (veja [Aba Network do Chrome DevTools](./02a-devtools-network.md#s2-1)) e a postura defensiva de qualquer aplicação (veja [Vulnerabilidades Web](./03a-vulnerabilidades-parte1.md#s3) e [Headers de Segurança](./05-headers-de-seguranca.md#s5)) repousam diretamente sobre a arquitetura das **Redes de Computadores**. Quando um usuário digita `https://loja-ficticia.test` na barra de endereços, dezenas de protocolos operam de forma coordenada em frações de segundo: nomes são traduzidos em endereços numéricos, quadros elétricos ou ópticos atravessam enlaces locais, pacotes são roteados por múltiplos sistemas autônomos na Internet, sessões de transporte controlam perdas e congestionamento, e túneis criptográficos garantem que nenhum intermediário leia ou adultere a comunicação.

Para a engenharia de software e para a Segurança da Informação (*Cybersecurity*), compreender redes em profundidade permite diagnosticar gargalos de latência, arquitetar infraestruturas resilientes e bloquear vetores de ataque que ocorrem muito antes de a requisição chegar ao código da aplicação.

---

<a id="s4-1"></a>
### 4.1 Modelo OSI e suas sete camadas

O **Modelo OSI** (*Open Systems Interconnection* — Interconexão de Sistemas Abertos), padronizado pela **ISO** (*International Organization for Standardization* — Organização Internacional de Normalização) na norma **ISO/IEC 7498-1** em 1984, é um modelo conceitual e de referência que divide as funções de comunicação de rede em **sete camadas hierárquicas**.

O objetivo primordial do Modelo OSI é promover a **interoperabilidade** entre sistemas heterogêneos por meio da **abstração em camadas**:
1. Cada camada executa um conjunto bem definido de funções de rede.
2. Cada camada consome serviços da camada imediatamente inferior (*Camada $N-1$*) e fornece serviços para a camada imediatamente superior (*Camada $N+1$*).
3. Alterações tecnológicas em uma camada (por exemplo, trocar um cabo metálico Ethernet por Wi-Fi ou fibra óptica na Camada 1/2) não exigem reescrever os protocolos das camadas superiores (como o IP na Camada 3 ou o HTTP na Camada 7).

##### Conceito de PDU, Encapsulamento e Desencapsulamento

À medida que os dados gerados por uma aplicação descem a pilha do transmissor (da Camada 7 até a Camada 1), ocorre o processo de **encapsulamento**: cada camada adiciona informações de controle próprias — geralmente um **cabeçalho** (*header*, ou **PCI** — *Protocol Control Information*) no início dos dados e, no caso da Camada 2, também um **rodapé/trailer** (*Frame Check Sequence* — FCS) no final. Os dados recebidos da camada superior são tratados como carga útil (**SDU** — *Service Data Unit*). A combinação do cabeçalho da camada atual com a SDU forma a **PDU** (*Protocol Data Unit* — Unidade de Dados de Protocolo) daquela camada.

No receptor, ocorre o processo inverso, denominado **desencapsulamento**: o sinal físico sobe da Camada 1 até a Camada 7; cada camada lê e valida seu respectivo cabeçalho, executa sua função lógica, remove o cabeçalho e entrega a carga útil para a camada imediatamente superior.

```text
================================================================================
                FLUXO DE ENCAPSULAMENTO E DESENCAPSULAMENTO OSI
================================================================================

  ORIGEM (Cliente Web)                                   DESTINO (Servidor Web)
  Envia requisição HTTP                                  Processa requisição HTTP
         |                                                         ^
         v (Encapsulamento: desce)                                 | (Desencapsulamento: sobe)
+--------------------+                                   +--------------------+
| 7. Aplicação       |  PDU: Dados (Data / Message)      | 7. Aplicação       |
| 6. Apresentação    |  [ Dados da Aplicação (HTTP) ]    | 6. Apresentação    |
| 5. Sessão          |                                   | 5. Sessão          |
+--------------------+                                   +--------------------+
         |                                                         ^
         v Adiciona Cabeçalho L4 (Portas Origem/Destino, Seq, Ack) | Remove L4
+--------------------+                                   +--------------------+
| 4. Transporte      |  PDU: Segmento (TCP) / Datagrama  | 4. Transporte      |
|                    |  [ Hdr L4 | Dados ]               |                    |
+--------------------+                                   +--------------------+
         |                                                         ^
         v Adiciona Cabeçalho L3 (IP Origem, IP Destino, TTL)      | Remove L3
+--------------------+                                   +--------------------+
| 3. Rede            |  PDU: Pacote (Packet)             | 3. Rede            |
|                    |  [ Hdr L3 | Hdr L4 | Dados ]      |                    |
+--------------------+                                   +--------------------+
         |                                                         ^
         v Adiciona Cabeçalho L2 (MAC Dest/Orig) + Trailer (FCS)   | Remove L2
+--------------------+                                   +--------------------+
| 2. Enlace de Dados |  PDU: Quadro (Frame)              | 2. Enlace de Dados |
|                    |  [ Hdr L2 | Hdr L3 | L4 | D |FCS] |                    |
+--------------------+                                   +--------------------+
         |                                                         ^
         v Codifica em pulsos elétricos, luz ou ondas de rádio     | Decodifica bits
+--------------------+                                   +--------------------+
| 1. Física          |  PDU: Bits (010110010110...)      | 1. Física          |
+--------------------+                                   +--------------------+
         |                                                         ^
         +=====================[ Meio Físico ]=====================+
```

> [!TIP]
> **Mnemônicos clássicos para prova:**
> - De baixo para cima (Camada 1 a 7): **F**ísica, **E**nlace, **R**ede, **T**ransporte, **S**essão, **A**presentação, **A**plicação -> *"Fui Estudar Redes Todo Sábado Às Aulas"* (em inglês: *Please Do Not Throw Sausage Pizza Away* — Physical, Data Link, Network, Transport, Session, Presentation, Application).
> - Para a **PDU** da Camada 1 à 4: **B**its (1) -> **Q**uadro/*Frame* (2) -> **P**acote (3) -> **S**egmento/Datagrama (4) -> **D**ados (5, 6, 7).

##### Análise Detalhada Camada por Camada (Camadas 7 a 1)

###### Camada 7 — Aplicação (*Application Layer*)
- **O que é e para que serve**: É a camada mais próxima do usuário final e dos processos de software. Ela **não é** o aplicativo gráfico em si (como o Google Chrome ou o VS Code), mas sim o conjunto de protocolos que fornecem serviços de rede diretamente às aplicações (transferência de hipermídia, resolução de nomes, envio de correio eletrônico, transferência de arquivos).
- **PDU**: **Dados** (*Data* ou *Message*).
- **Protocolos principais**: **HTTP** (*Hypertext Transfer Protocol*), **HTTPS**, **DNS** (*Domain Name System*), **DHCP** (*Dynamic Host Configuration Protocol*), **FTP** (*File Transfer Protocol*), **SSH** (*Secure Shell*), **SMTP** (*Simple Mail Transfer Protocol*), **POP3** (*Post Office Protocol v3*), **IMAP** (*Internet Message Access Protocol*), **SNMP** (*Simple Network Management Protocol*), **Telnet**, **WebSocket**.
- **Equipamentos e componentes de software**: **WAF** (*Web Application Firewall* — Firewall de Aplicação Web), **Proxies Reversos / API Gateways** (Nginx, Envoy, HAProxy operando em L7), **Balanceadores de Carga L7** (*Application Load Balancers*), navegadores e servidores de aplicação.
- **Problemas de segurança**: Ataques lógicos e de injeção contra aplicações (SQL Injection, Cross-Site Scripting — XSS, Cross-Site Request Forgery — CSRF, Server-Side Request Forgery — SSRF; veja [Seção 3](./03a-vulnerabilidades-parte1.md#s3)), ataques de negação de serviço na camada de aplicação (**HTTP Flood** / **Slowloris**), enumeração de usuários, força bruta de credenciais e *DNS Cache Poisoning*.
- **Configuração segura**: **[REQUISITO]** Validação estrita de entrada e parametrização de consultas no código; **[RECOMENDAÇÃO]** implementação de WAF, *Rate Limiting* (limitação de taxa de requisições, retornando status `429 Too Many Requests`), autenticação forte e cabeçalhos de segurança HTTP (veja [Seção 5](./05-headers-de-seguranca.md#s5)).

###### Camada 6 — Apresentação (*Presentation Layer*)
- **O que é e para que serve**: Atua como o "tradutor" da rede. É responsável pela **formatação e representação sintática dos dados** (garantindo que os dados enviados pela Camada 7 de um sistema possam ser lidos pela Camada 7 de outro sistema com arquitetura interna diferente), **codificação de caracteres** (ASCII, UTF-8, EBCDIC), **serialização de estruturas de dados** (JSON, XML, Protocol Buffers), **compressão de dados** (Gzip, Brotli, Deflate) e, historicamente no modelo conceitual, **criptografia/descriptografia**.
- **PDU**: **Dados** (*Data*).
- **Protocolos, formatos e padrões**: **UTF-8**, **ASCII**, **MIME** (*Multipurpose Internet Mail Extensions*), **TLS/SSL** (*Transport Layer Security / Secure Sockets Layer* — embora na prática do modelo TCP/IP opere entre Transporte e Aplicação), **Gzip**, **Brotli (`br`)**, **JPEG**, **PNG**, **MPEG**.
- **Problemas de segurança**: Ataques baseados em ambiguidade de codificação e desserialização insegura (*Insecure Deserialization*), ataques de canal lateral em compressão combinada com criptografia (como **CRIME** e **BREACH** em TLS/HTTP), *MIME Sniffing* (quando o navegador adivinha incorretamente o tipo de conteúdo e executa HTML/JS disfarçado de imagem) e *downgrade* de algoritmos criptográficos (ex.: forçar SSL 3.0 no ataque POODLE).
- **Configuração segura**: **[REQUISITO]** Uso exclusivo de TLS 1.2 e TLS 1.3 com cifras modernas; **[REQUISITO]** envio do header `X-Content-Type-Options: nosniff` e `Content-Type` explícito com `charset=utf-8`; **[RECOMENDAÇÃO]** evitar comprimir respostas HTTP que contenham simultaneamente segredos (como tokens CSRF) e entradas refletidas controladas pelo usuário.

###### Camada 5 — Sessão (*Session Layer*)
- **O que é e para que serve**: Gerencia o **diálogo** entre aplicações em dispositivos distintos. Suas atribuições incluem estabelecer, manter, sincronizar (por meio de *checkpoints* ou pontos de verificação para retomada em caso de falha) e encerrar sessões de comunicação, além de definir a disciplina de diálogo (*half-duplex* — um fala por vez, ou *full-duplex* — ambos falam simultaneamente).
- **PDU**: **Dados** (*Data*).
- **Protocolos principais**: **NetBIOS** (*Network Basic Input/Output System*), **RPC** (*Remote Procedure Call*), **SMB** (*Server Message Block* — atua entre L5, L6 e L7), **PPTP** (*Point-to-Point Tunneling Protocol*), **SOCKS** (protocolo de proxy), **NFS** (*Network File System*). *(Nota: Na Web moderna, o gerenciamento de sessão de usuários é implementado na Camada 7 via Cookies/Tokens, enquanto a conexão persistente e o túnel criptográfico são geridos entre as Camadas 4 e 6 via TCP e TLS Session Resumption).*
- **Problemas de segurança**: **Sequestro de Sessão** (*Session Hijacking*), **Fixação de Sessão** (*Session Fixation*), ataques de repetição (*Replay Attacks*), exploração de serviços de compartilhamento e chamada remota (como vulnerabilidades críticas em SMBv1, ex.: *EternalBlue* / MS17-010) e enumeração de sessões NetBIOS/RPC.
- **Configuração segura**: **[REQUISITO]** Desativação absoluta de SMBv1 e bloqueio de portas de sessão locais (137–139, 445) no perímetro de Internet; **[REQUISITO]** renovação de identificadores de sessão após autenticação e uso de *nonces*/timestamps contra ataques de repetição.

###### Camada 4 — Transporte (*Transport Layer*)
- **O que é e para que serve**: Responsável pela comunicação lógica **fim a fim (*end-to-end*, ou processo a processo)** entre aplicações rodando em hosts diferentes. Enquanto a Camada 3 entrega pacotes de uma máquina (IP) para outra máquina (IP), a Camada 4 utiliza **números de porta** (16 bits) para entregar os dados ao processo correto dentro da máquina (multiplexação e demultiplexação). Também realiza segmentação de dados grandes, remontagem na ordem correta, detecção de erros, controle de fluxo (evitar que um transmissor rápido afogue um receptor lento) e controle de congestionamento da rede.
- **PDU**: **Segmento** (*Segment*, quando o protocolo é o **TCP**) ou **Datagrama** (*Datagram*, quando o protocolo é o **UDP**).
- **Protocolos principais**: **TCP** (*Transmission Control Protocol* — RFC 9293), **UDP** (*User Datagram Protocol* — RFC 768), **QUIC** (RFC 9000 — protocolo moderno de transporte sobre UDP que incorpora TLS 1.3 nativamente, base do HTTP/3), **SCTP** (*Stream Control Transmission Protocol*).
- **Equipamentos**: **Firewalls Stateful** (que inspecionam o estado de conexões TCP/UDP), **Balanceadores de Carga L4** (*Network Load Balancers*).
- **Problemas de segurança**: **SYN Flood** (exaustão da fila de conexões semiabertas do servidor), **UDP Flood** e ataques de **Amplificação/Reflexão UDP** (usando DNS, NTP ou Memcached com IP de origem falsificado), **TCP Session Hijacking** (adivinhação ou interceptação de números de sequência TCP) e varredura de portas (*Port Scanning* com ferramentas como `nmap` para mapear serviços expostos).
- **Configuração segura**: **[REQUISITO]** Ativação de **SYN Cookies** no sistema operacional do servidor para resistir a SYN Flood; **[REQUISITO]** fechamento de todas as portas TCP/UDP não utilizadas via Firewall (*Default Deny*); **[RECOMENDAÇÃO]** uso de limites de conexões simultâneas por IP de origem.

###### Camada 3 — Rede (*Network Layer*)
- **O que é e para que serve**: Responsável pelo **endereçamento lógico global** e pelo **roteamento (*routing*)** de pacotes através de múltiplas redes interconectadas (inter-rede / Internet). Determina o melhor caminho físico/lógico desde a rede de origem até a rede de destino, mesmo que estejam separadas por dezenas de roteadores ao redor do mundo. Trabalha na modalidade *best-effort* (melhor esforço, sem garantia intrínseca de entrega ou ordem no IP puro).
- **PDU**: **Pacote** (*Packet* ou *IP Datagram*).
- **Protocolos principais**: **IPv4** (*Internet Protocol version 4* — RFC 791), **IPv6** (*Internet Protocol version 6* — RFC 8200), **ICMP** (*Internet Control Message Protocol* — RFC 792) e **ICMPv6** (RFC 4443), **IPsec** (*Internet Protocol Security* — suíte de criptografia e autenticação na camada IP: AH e ESP), e protocolos de roteamento como **OSPF** (*Open Shortest Path First*), **BGP** (*Border Gateway Protocol* — que frequentemente usa TCP como transporte, mas exerce função de plano de controle da Camada 3) e **RIP**.
- **Equipamentos**: **Roteadores** (*Routers*), **Switches Layer 3** (*Multilayer Switches*), **Firewalls de Filtragem de Pacotes**.
- **Problemas de segurança**: **IP Spoofing** (falsificação do endereço IP de origem no cabeçalho do pacote), **BGP Hijacking** (anúncio malicioso ou acidental de rotas BGP desviando tráfego de grandes blocos da Internet), **Smurf Attack** / **Ping Flood** / **Ping of Death** (abuso de ICMP), ataques de fragmentação de pacotes IP (*Teardrop* / evasão de IDS) e roteamento na fonte (*Source Routing*).
- **Configuração segura**: **[REQUISITO]** Implementação de filtragem de ingresso/egresso (**BCP 38 / RFC 2827**) nos provedores e bordas de rede para descartar pacotes com IP de origem falsificado; **[RECOMENDAÇÃO]** adoção de **RPKI** (*Resource Public Key Infrastructure*) para validação criptográfica de rotas BGP, desativação de *IP Source Routing* e de *Directed Broadcast* nos roteadores.

###### Camada 2 — Enlace de Dados (*Data Link Layer*)
- **O que é e para que serve**: Garante a transferência confiável de dados **nó a nó (*node-to-node* / *hop-to-hop*)** dentro do **mesmo enlace físico ou rede local (LAN)**. Organiza os bits da Camada 1 em **quadros (*frames*)**, realiza o **endereçamento físico** via **MAC Address** (*Media Access Control*), controla o acesso ao meio compartilhado (evitando ou tratando colisões) e detecta erros de transmissão física por meio do cálculo de **CRC** (*Cyclic Redundancy Check*) no campo **FCS** (*Frame Check Sequence*). Divide-se em duas subcamadas definidas pelo IEEE 802:
  - **LLC** (*Logical Link Control* — IEEE 802.2): faz a interface com a Camada 3 (identificando qual protocolo de rede está encapsulado, ex.: IPv4 ou IPv6) e controle de fluxo/erros no enlace.
  - **MAC** (*Media Access Control*): gerencia o endereçamento físico (MAC de 48 bits) e a arbitragem de acesso ao meio físico (CSMA/CD no Ethernet legado, CSMA/CA no Wi-Fi).
- **PDU**: **Quadro** (*Frame*).
- **Protocolos e padrões principais**: **Ethernet** (IEEE 802.3), **Wi-Fi / WLAN** (IEEE 802.11), **ARP** (*Address Resolution Protocol* — protocolo de transição que mapeia endereços IP da Camada 3 em endereços MAC da Camada 2), **VLAN** (IEEE 802.1Q — redes locais virtuais), **STP** (*Spanning Tree Protocol* — evita loops em switches), **PPP** (*Point-to-Point Protocol*), **LACP** (agregação de links), **IEEE 802.1X** (controle de acesso à rede baseado em porta).
- **Equipamentos**: **Switches** (comutadores Layer 2), **Placas de Rede** (**NIC** — *Network Interface Card*), **Bridges** (pontes), **Access Points (AP)** Wi-Fi.
- **Problemas de segurança**: **ARP Spoofing / ARP Poisoning** (envenenamento da tabela ARP para interceptar tráfego em ataques *Man-in-the-Middle* — MitM na rede local), **MAC Flooding** (inundar a tabela CAM — *Content Addressable Memory* — do switch com milhares de MACs falsos para forçá-lo a se comportar como um *hub* e transmitir quadros para todas as portas — *fail-open*), **VLAN Hopping** (*Switch Spoofing* ou *Double Tagging* 802.1Q para enviar tráfego a uma VLAN isolada), **STP Manipulation** (forjar BPDU para tornar a máquina do atacante a *Root Bridge*) e captura passiva em redes Wi-Fi abertas ou com WEP/WPA fraco.
- **Configuração segura**: **[REQUISITO]** Uso de **WPA3-Enterprise** (ou WPA2-Enterprise com **IEEE 802.1X / RADIUS**) em redes corporativas sem fio; **[RECOMENDAÇÃO]** ativação de **Port Security** (limita o número de endereços MAC por porta do switch), **DAI** (*Dynamic ARP Inspection*) combinado com **DHCP Snooping** contra ARP Spoofing, **BPDU Guard** nas portas de acesso e desativação de negociação automática de tronco (**DTP**), mantendo a VLAN nativa separada de VLANs de usuários.

###### Camada 1 — Física (*Physical Layer*)
- **O que é e para que serve**: Trata das características **mecânicas, elétricas, ópticas, funcionais e de radiofrequência** necessárias para transmitir e receber um fluxo bruto de bits (`0` e `1`) através de um meio físico de comunicação. Define tensões elétricas, comprimentos de onda de luz, frequências de rádio, taxas de transmissão (*bit rate*), codificação de linha (NRZ, Manchester, PAM4), pinagem de conectores (RJ-45, LC/SC) e topologia física.
- **PDU**: **Bit** (ou símbolo físico).
- **Padrões e meios físicos**: Cabos de par trançado de cobre (**UTP/STP** — Categorias Cat5e, Cat6, Cat6A, Cat8 com padrão **1000BASE-T**, **10GBASE-T**), **Fibra Óptica** monomodo (SMF) e multimodo (MMF), espectro de radiofrequência (**RF** do Wi-Fi 2.4 GHz, 5 GHz, 6 GHz, Bluetooth, 4G/5G), **USB**, **SONET/SDH**, **DSL**.
- **Equipamentos**: **Cabos**, **Conectores**, **Transceptores (SFP/SFP+)**, **Repetidores** (*Repeaters*), **Conversores de Mídia**, **Modems** e **Hubs** (concentradores multiporta puramente elétricos — hoje **`[Obsoleto]`**, substituídos por switches porque os hubs replicavam todos os sinais em todas as portas, criando um único **domínio de colisão**).
- **Problemas de segurança**: **Interceptação física (*Wiretapping*)** por grampeamento de cabos de cobre ou curvatura de fibra óptica, **Jamming** (interferência eletromagnética intencional em sinais Wi-Fi/rádio causando negação de serviço), desconexão ou corte de cabos (indisponibilidade física), e acesso físico não autorizado a salas de telecomunicações/racks para conectar dispositivos espiões (*LAN Taps*).
- **Configuração segura**: **[REQUISITO]** Controle de acesso físico rigoroso a datacenters, racks e *patch panels*; **[BOA PRÁTICA]** preferência por fibra óptica em enlaces troncais críticos (mais difícil de grampear sem causar atenuação detectável do que cabos metálicos), desativação administrativa (`shutdown`) de portas físicas de parede não utilizadas e monitoramento de espectro de RF.

##### Tabela Consolidada do Modelo OSI

| Camada | Nome | PDU | Endereçamento | Função Essencial | Protocolos / Padrões Exemplos | Equipamentos Típicos | Ataques Clássicos |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **7** | **Aplicação** | Dados | URL / URI / Nome de Host | Serviços de rede para aplicações do usuário | HTTP, HTTPS, DNS, DHCP, SSH, SMTP, FTP, IMAP | WAF, Proxy L7, API Gateway | SQLi, XSS, CSRF, SSRF, HTTP Flood, Brute Force |
| **6** | **Apresentação** | Dados | — | Formatação, codificação, compressão, criptografia | UTF-8, JSON, XML, MIME, Gzip, Brotli, TLS/SSL | Gateways TLS, Offloaders | Insecure Deserialization, MIME Sniffing, CRIME/BREACH |
| **5** | **Sessão** | Dados | Session ID (conceitual) | Controle de diálogo, abertura e retomada de sessão | NetBIOS, RPC, SMB, SOCKS, PPTP | Gateways de Sessão | Session Hijacking, Session Fixation, Replay, EternalBlue |
| **4** | **Transporte** | Segmento (TCP) / Datagrama (UDP) | **Porta** (16 bits: `0–65535`) | Comunicação fim a fim (processo a processo), confiabilidade | TCP, UDP, QUIC, SCTP | Firewall Stateful, Load Balancer L4 | SYN Flood, UDP Amplification, Port Scan, TCP Hijacking |
| **3** | **Rede** | Pacote | **Endereço IP** (IPv4 32 bits / IPv6 128 bits) | Endereçamento lógico global e roteamento entre redes | IPv4, IPv6, ICMP, IPsec, OSPF, BGP | Roteador, Switch Layer 3 | IP Spoofing, BGP Hijacking, Ping Flood, Smurf Attack |
| **2** | **Enlace de Dados** | Quadro (*Frame*) | **Endereço MAC** (48 bits) | Transferência nó a nó na rede local, detecção de erro físico | Ethernet (802.3), Wi-Fi (802.11), ARP, VLAN (802.1Q), STP | Switch Layer 2, Placa de Rede (NIC), AP | ARP Spoofing/Poisoning, MAC Flooding, VLAN Hopping |
| **1** | **Física** | Bit | — | Transmissão de sinais elétricos, ópticos ou de rádio no meio | RJ-45, Cat6, Fibra Óptica, 1000BASE-T, RF | Cabo, Hub (`[Obsoleto]`), Repetidor, Transceptor | Wiretapping (grampo), RF Jamming, Corte de cabos |

---

<a id="s4-2"></a>
### 4.2 Modelo TCP/IP e comparativo com o Modelo OSI

Enquanto o Modelo OSI foi desenhado por comitês internacionais como um padrão teórico **antes** que seus protocolos fossem amplamente adotados, o **Modelo TCP/IP** (também conhecido como **Modelo DoD** — *Department of Defense* dos EUA — ou **Arquitetura da Internet**, formalizado na **RFC 1122**) nasceu da implementação prática da ARPANET na década de 1970 e 1980. O protocolo da Internet real é o **TCP/IP**, enquanto o Modelo OSI é utilizado universalmente como vocabulário de diagnóstico e ensino (daí expressões cotidianas como *"Switch Layer 2"*, *"Roteador Layer 3"*, *"Load Balancer Layer 4"* e *"Firewall Layer 7"*).

##### As Camadas do Modelo TCP/IP

Na sua formulação clássica de **4 camadas** (RFC 1122), o Modelo TCP/IP agrupa funções das extremidades superior e inferior do Modelo OSI:

1. **Camada de Aplicação (*Application Layer*)**: Consolida as **Camadas 5 (Sessão), 6 (Apresentação) e 7 (Aplicação)** do Modelo OSI. No modelo TCP/IP, considera-se que se uma aplicação precisa de gerenciamento de sessão, codificação de dados ou criptografia, esses detalhes são resolvidos pelos próprios protocolos de aplicação (como HTTP sobre TLS, SSH, SMTP, DNS).
2. **Camada de Transporte (*Transport Layer* ou *Host-to-Host*)**: Corresponde diretamente à **Camada 4 (Transporte)** do Modelo OSI. É responsável pela entrega fim a fim entre processos por meio dos protocolos **TCP**, **UDP** e **QUIC**.
3. **Camada de Internet (*Internet Layer*)**: Corresponde diretamente à **Camada 3 (Rede)** do Modelo OSI. Responsável pelo endereçamento lógico e roteamento de pacotes inter-redes usando **IPv4**, **IPv6**, **ICMP** e **IPsec**.
4. **Camada de Acesso à Rede (*Network Access Layer* ou *Link Layer*)**: Consolida as **Camadas 1 (Física) e 2 (Enlace de Dados)** do Modelo OSI. Descreve como o pacote IP é formatado em um quadro e transmitido fisicamente pelo meio local (Ethernet, Wi-Fi, Fibra).

> [!NOTE]
> **Modelo TCP/IP de 4 camadas vs. 5 camadas em provas acadêmicas:**
> Autores clássicos de redes de computadores (como **Andrew S. Tanenbaum** e **James F. Kurose / Keith W. Ross**) adotam frequentemente uma versão híbrida didática de **5 camadas** (Aplicação, Transporte, Rede, Enlace e Física), separando a camada de Acesso à Rede em Enlace (L2) e Física (L1). Em questões de prova, fique atento ao enunciado: se a questão citar a arquitetura original TCP/IP (RFC 1122), são **4 camadas**; se citar o modelo híbrido/moderno de livros-texto, são **5 camadas**. No Modelo OSI, são sempre **7 camadas**.

##### Tabela Comparativa: Modelo OSI vs. Modelo TCP/IP

```text
+-----------------------------------+-----------------------------------+-----------------------------------+
|      MODELO OSI (7 Camadas)       |    MODELO TCP/IP (4 Camadas)      |  MODELO HÍBRIDO DIDÁTICO (5 Cam.) |
+-----------------------------------+-----------------------------------+-----------------------------------+
|  7. Aplicação                     |                                   |                                   |
+-----------------------------------+                                   |                                   |
|  6. Apresentação                  |  4. Aplicação                     |  5. Aplicação                     |
+-----------------------------------+                                   |                                   |
|  5. Sessão                        |                                   |                                   |
+-----------------------------------+-----------------------------------+-----------------------------------+
|  4. Transporte                    |  3. Transporte (Host-to-Host)     |  4. Transporte                    |
+-----------------------------------+-----------------------------------+-----------------------------------+
|  3. Rede                          |  2. Internet                      |  3. Rede (ou Internet)            |
+-----------------------------------+-----------------------------------+-----------------------------------+
|  2. Enlace de Dados               |                                   |  2. Enlace de Dados (Link)        |
+-----------------------------------+  1. Acesso à Rede (Link/Network)  +-----------------------------------+
|  1. Física                        |                                   |  1. Física                        |
+-----------------------------------+-----------------------------------+-----------------------------------+
```

| Critério de Comparação | Modelo OSI (ISO/IEC 7498-1) | Modelo TCP/IP (RFC 1122) |
| :--- | :--- | :--- |
| **Número de Camadas** | **7 camadas** | **4 camadas** (ou 5 no modelo híbrido didático) |
| **Origem Histórica** | Modelo teórico de referência criado pela ISO antes da implementação dos protocolos. | Modelo prático criado pelo DoD/DARPA derivado dos protocolos que já operavam na ARPANET/Internet. |
| **Dependência de Protocolos** | Independente de protocolos (genérico; serve para explicar qualquer pilha de rede). | Específico para a suíte de protocolos da Internet (IP, TCP, UDP, HTTP, etc.). |
| **Sessão e Apresentação** | Camadas separadas e explícitas (Camadas 5 e 6). | Incorporadas dentro da **Camada de Aplicação**. |
| **Enlace e Física** | Camadas separadas e explícitas (Camadas 2 e 1). | Agrupadas na **Camada de Acesso à Rede (Link)** na definição de 4 camadas. |
| **Uso Principal Hoje** | Ensino, certificação, padronização de vocabulário técnico (L2, L3, L4, L7) e diagnóstico modular (*troubleshooting*). | Arquitetura real em execução em todos os sistemas operacionais, roteadores e servidores da Internet. |

---

<a id="s4-3"></a>
### 4.3 Camada de Transporte: TCP, UDP e Handshake

A Camada de Transporte (Camada 4) é a ponte entre o mundo dos pacotes de rede (que só conhecem máquinas identificadas por endereços IP) e o mundo das aplicações (que precisam de canais lógicos dedicados para processos específicos). Os dois pilares históricos desta camada são o **TCP** e o **UDP**.

<a id="s4-3-1"></a>
#### 4.3.1 Diferenças entre TCP e UDP

##### TCP (Transmission Control Protocol — Protocolo de Controle de Transmissão, RFC 9293)
O **TCP** é um protocolo **orientado à conexão** (*connection-oriented*) e **confiável** (*reliable*). Antes que qualquer byte da aplicação seja transmitido, cliente e servidor precisam negociar e estabelecer um estado compartilhado (o *Three-Way Handshake*). O TCP oferece um fluxo contínuo de bytes (*byte stream*) ordenado e livre de duplicatas:
- **Confirmação e Retransmissão (*ACK & Retransmission*)**: Cada byte enviado recebe um número de sequência (*Sequence Number*). O receptor confirma o recebimento enviando um pacote com o número do próximo byte esperado (*Acknowledgment Number* — ACK). Se o transmissor não receber o ACK dentro de um tempo limite dinâmico (**RTO** — *Retransmission Timeout*), ou se receber múltiplos ACKs duplicados (*Fast Retransmit*), ele retransmite o segmento perdido automaticamente.
- **Ordenação**: Pacotes IP podem viajar por rotas diferentes na Internet e chegar fora de ordem. O TCP no destinatário usa o *Sequence Number* para reordenar os segmentos antes de entregá-los à aplicação (evitando que uma página HTML ou arquivo JSON chegue embaralhado).
- **Controle de Fluxo (*Flow Control*)**: Usa o campo **Window Size** (Janela Deslizante / *Sliding Window*) no cabeçalho TCP para que o receptor informe ao transmissor quantos bytes livres ainda possui em seu *buffer* de recepção, evitando sobrecarga local.
- **Controle de Congestionamento (*Congestion Control*)**: Algoritmos como *Slow Start*, *Congestion Avoidance*, *CUBIC* e *BBR* monitoram perdas e variações de latência (**RTT** — *Round-Trip Time*) para ajustar a taxa de envio e evitar o colapso dos roteadores intermediários da Internet.

##### Cabeçalho TCP e as Flags de Controle
O cabeçalho TCP possui um tamanho mínimo de **20 bytes** (podendo chegar a 60 bytes com opções como *Maximum Segment Size* — MSS, *Window Scale* e *Selective Acknowledgments* — SACK).

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        | (4 bytes)
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        | (4 bytes)
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      | (4 bytes)
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |       |C|E|U|A|P|R|S|F|                               |
| Offset| Rsrvd |W|C|R|C|S|S|Y|I|            Window             | (4 bytes)
|       |       |R|E|G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        | (4 bytes)
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (0 a 40 bytes)                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

As **6 flags clássicas do TCP** (além das 2 flags modernas de notificação explícita de congestionamento, `CWR` e `ECE`) controlam a máquina de estados da conexão e são presença garantida em provas de redes e segurança:

| Flag TCP | Nome Completo | Significado e Função Prática |
| :--- | :--- | :--- |
| **`SYN`** | *Synchronize* | **Sincronizar números de sequência**. Usada apenas nos dois primeiros passos do estabelecimento de conexão (*Three-Way Handshake*) para negociar os números de sequência iniciais (ISN). |
| **`ACK`** | *Acknowledgment* | **Confirmação**. Indica que o campo *Acknowledgment Number* do cabeçalho é válido. Após o primeiro pacote `SYN` puro, praticamente todos os pacotes de uma conexão TCP normal carregam a flag `ACK` ativa. |
| **`FIN`** | *Finish* | **Finalizar graciosamente**. Indica que o emissor não tem mais dados de aplicação para enviar e deseja encerrar sua metade da conexão de forma limpa e ordenada. |
| **`RST`** | *Reset* | **Reiniciar/Abortar imediatamente**. Encerra a conexão de forma abrupta, sem esperar confirmação. É enviada quando um pacote chega para uma porta fechada, quando uma conexão inválida/expirada recebe dados, ou quando um firewall/aplicação aborta a comunicação. |
| **`PSH`** | *Push* | **Empurrar dados imediatamente**. Instrui a pilha TCP do receptor a entregar os dados recebidos imediatamente para o processo da Camada 7 (como o navegador ou servidor HTTP), sem esperar que o *buffer* de recepção do sistema operacional encha. |
| **`URG`** | *Urgent* | **Dados urgentes**. Sinaliza que o campo *Urgent Pointer* é válido e aponta para dados prioritários (fora de banda). Pouco utilizado em protocolos web modernos. |

##### UDP (User Datagram Protocol — Protocolo de Datagrama de Usuário, RFC 768)
O **UDP** é um protocolo **sem conexão** (*connectionless*) e **não confiável** (*unreliable / best-effort*). Ele funciona como o envio de uma carta simples pelo correio: a aplicação empacota os dados, coloca a porta de origem e a porta de destino, e dispara o datagrama na rede sem qualquer *handshake* prévio.
- Não há confirmação de recebimento (`ACK`), nem retransmissão automática na Camada 4.
- Não há garantia de ordem de chegada nem controle de fluxo/congestionamento nativo no UDP.
- Seu cabeçalho é extremamente enxuto: apenas **8 bytes fixos** (Porta de Origem: 2 bytes; Porta de Destino: 2 bytes; Comprimento: 2 bytes; Checksum: 2 bytes).

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        | (4 bytes)
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Length             |           Checksum            | (4 bytes)
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Por que usar UDP se ele não garante entrega?**
1. **Zero latência de estabelecimento de conexão**: Não gasta 1 RTT inteiro com *Three-Way Handshake* antes de enviar o primeiro byte (ideal para consultas rápidas de pergunta/resposta como **DNS** na porta 53).
2. **Ausência de *Head-of-Line (HoL) Blocking* na camada de transporte**: No TCP, se o segmento #2 se perder no caminho, mas os segmentos #3, #4 e #5 chegarem perfeitamente, o kernel do sistema operacional retém os segmentos #3, #4 e #5 no *buffer* até que o #2 seja retransmitido. Em uma chamada de vídeo ao vivo (**WebRTC** / **VoIP**), jogos online competitivos ou no protocolo **HTTP/3 (sobre QUIC/UDP)**, esperar por um pacote antigo atrasa todo o fluxo ou entrega um quadro de áudio que já perdeu sua utilidade temporal. No HTTP/3, o QUIC usa UDP na rede e implementa confiabilidade independente por fluxo (*stream*), eliminando o bloqueio de cabeça de fila do TCP!

##### Tabela Comparativa Completa: TCP vs. UDP

| Característica | TCP (RFC 9293) | UDP (RFC 768) |
| :--- | :--- | :--- |
| **Orientação à Conexão** | **Orientado à conexão** (exige *Three-Way Handshake* prévio). | **Sem conexão** (*Connectionless* — envia dados diretamente). |
| **Nome da PDU** | **Segmento** (*Segment*). | **Datagrama** (*Datagram*). |
| **Confiabilidade / Entrega** | **Garantida** (usa `ACK`, *timeouts* e retransmissão de pacotes perdidos). | **Não garantida** (*Best-effort*; pacotes perdidos são ignorados pelo UDP). |
| **Ordenação de Dados** | **Garantida** (reordena segmentos via *Sequence Number*). | **Não garantida** (datagramas podem chegar fora da ordem de envio). |
| **Controle de Fluxo e Congestionamento** | **Sim** (Janela deslizante, *Slow Start*, CUBIC, BBR). | **Não** (salvo se implementado pela aplicação acima, como faz o QUIC). |
| **Tamanho do Cabeçalho** | **20 a 60 bytes** (variável). | **8 bytes** (fixo). |
| **Sobrecarga (*Overhead*) e Latência** | Maior sobrecarga de CPU/memória e latência inicial (1 RTT do handshake). | Mínima sobrecarga e latência imediata. |
| **Bloqueio de Cabeça de Fila (*HoL Blocking*)** | Sofre de *HoL Blocking* no nível de conexão se um único pacote cair. | Não retém pacotes subsequentes no nível do sistema operacional. |
| **Suporte a Broadcast / Multicast** | **Não** (estritamente ponto a ponto / *unicast*: 1 origem para 1 destino). | **Sim** (suporta *unicast*, *multicast* e *broadcast*). |
| **Casos de Uso Clássicos** | **HTTP/1.1**, **HTTP/2**, **HTTPS (TLS sobre TCP)**, **SSH**, **FTP**, **SMTP**, **IMAP**, bancos de dados (**MySQL**, **PostgreSQL**). | **DNS** (consultas padrão), **DHCP**, **HTTP/3 (QUIC)**, **VoIP / WebRTC**, **Streaming ao vivo**, **Jogos Online**, **NTP**, **SNMP**. |

---

<a id="s4-3-2"></a>
#### 4.3.2 Handshake TCP (Three-Way Handshake) e encerramento de conexão

Para estabelecer uma comunicação bidirecional (*full-duplex*) confiável sobre uma rede IP não confiável, o TCP executa o famoso **Handshake de Três Vias (*Three-Way Handshake*)**. No Chrome DevTools (aba Network -> sub-aba **Timing**, veja [Fases de Timing](./02a-devtools-network.md#s2-1-6)), o tempo gasto exatamente nesta negociação corresponde à barra **Initial Connection**.

##### Passo a Passo do Three-Way Handshake

Imagine que o cliente (`192.0.2.10`, porta efêmera `54321`) deseja abrir uma conexão HTTPS com o servidor `loja-ficticia.test` (`203.0.113.50`, porta `443`):

1. **Passo 1 — `SYN` (Cliente $\rightarrow$ Servidor)**:
   - O cliente envia um segmento com a flag **`SYN=1`** (e `ACK=0`).
   - Ele escolhe um **Número de Sequência Inicial** (**ISN** — *Initial Sequence Number*) pseudoaleatório criptograficamente seguro, digamos `Seq = 1000` (em ferramentas como o Wireshark, costuma ser exibido como número relativo `Seq = 0` para facilitar a leitura).
   - O estado do cliente passa de `CLOSED` para `SYN_SENT`.
2. **Passo 2 — `SYN-ACK` (Servidor $\rightarrow$ Cliente)**:
   - O servidor, que estava escutando na porta `443` (estado `LISTEN`), recebe o `SYN`.
   - Ele responde com um segmento contendo **duas flags ativas**: **`SYN=1`** e **`ACK=1`**.
   - Para confirmar que recebeu o `SYN` do cliente, o servidor preenche o campo de confirmação somando `1` ao número de sequência do cliente: **`Ack = 1001`** (pois a flag `SYN` consome 1 número lógico de sequência, chamado de *byte fantasma*).
   - Simultaneamente, o servidor anuncia seu próprio Número de Sequência Inicial aleatório: **`Seq = 5000`** (relativo `0`).
   - O estado do servidor passa para `SYN_RCVD`.
3. **Passo 3 — `ACK` (Cliente $\rightarrow$ Servidor)**:
   - O cliente recebe o `SYN-ACK` e envia o terceiro segmento com a flag **`ACK=1`** (e `SYN=0`).
   - Ele usa seu próximo número de sequência **`Seq = 1001`** (relativo `1`) e confirma o `SYN` do servidor somando `1` ao ISN dele: **`Ack = 5001`** (relativo `1`).
   - Ambos os lados entram no estado **`ESTABLISHED`** (Conexão Estabelecida) e o envio de dados (como o *ClientHello* do TLS ou a requisição `GET / HTTP/1.1`) pode começar imediatamente!

```text
================================================================================
              ESTABELECIMENTO DE CONEXÃO TCP (THREE-WAY HANDSHAKE)
================================================================================

  CLIENTE (192.0.2.10:54321)                         SERVIDOR (203.0.113.50:443)
  Estado: CLOSED                                     Estado: LISTEN
        |                                                  |
        |  1. [SYN]          Seq=1000 (Rel: 0)             |
        |------------------------------------------------->|
  Estado: SYN_SENT                                   Estado: SYN_RCVD
        |                                                  |
        |  2. [SYN, ACK]     Seq=5000 (Rel: 0), Ack=1001   |
        |<-------------------------------------------------|
        |                                                  |
        |  3. [ACK]          Seq=1001 (Rel: 1), Ack=5001   |
        |------------------------------------------------->|
  Estado: ESTABLISHED                                Estado: ESTABLISHED
        |                                                  |
        |  4. [PSH, ACK]     Seq=1001, Ack=5001 (Len=517)  |  <-- Envia 517 bytes
        |------------------------------------------------->|      (ex: HTTP GET)
        |                                                  |
        |  5. [ACK]          Seq=5001, Ack=1518            |  <-- Confirma 1001+517
        |<-------------------------------------------------|
```

##### Encerramento Gracioso da Conexão (Four-Way Handshake) e Estado `TIME_WAIT`

Como uma conexão TCP é **full-duplex** (composta por dois fluxos independentes: Cliente $\rightarrow$ Servidor e Servidor $\rightarrow$ Cliente), cada lado deve encerrar sua direção de envio individualmente, resultando tradicionalmente em **4 passos** (*Four-Way Handshake*):

1. **Passo 1 — `FIN, ACK` (Lado A $\rightarrow$ Lado B)**: O lado que decide fechar a conexão (ex.: cliente) envia um segmento com a flag **`FIN=1`** (e `ACK=1`), indicando que não enviará mais dados. Seu estado muda para `FIN_WAIT_1`.
2. **Passo 2 — `ACK` (Lado B $\rightarrow$ Lado A)**: O servidor confirma o recebimento do `FIN` enviando um **`ACK`** (`Ack = Seq_do_FIN + 1`). O servidor entra em `CLOSE_WAIT` (ainda pode terminar de enviar dados pendentes, se houver) e o cliente entra em `FIN_WAIT_2`.
3. **Passo 3 — `FIN, ACK` (Lado B $\rightarrow$ Lado A)**: Quando o servidor também termina de enviar tudo, ele emite seu próprio segmento **`FIN=1, ACK=1`** e entra no estado `LAST_ACK`. *(Nota: Se o servidor não tiver mais nada para enviar já no momento em que recebe o primeiro `FIN`, ele pode combinar os Passos 2 e 3 em um único pacote `FIN, ACK`, reduzindo o fechamento para 3 pacotes).*
4. **Passo 4 — `ACK` (Lado A $\rightarrow$ Lado B)**: O cliente responde com o **`ACK`** final. O servidor, ao recebê-lo, encerra a conexão (`CLOSED`). O cliente, por sua vez, entra no estado **`TIME_WAIT`** (aguardando tipicamente $2 \times \text{MSL}$ — *Maximum Segment Lifetime*, cerca de 60 a 120 segundos) antes de ir para `CLOSED`, para garantir que: (a) se o último `ACK` se perder, ele ainda possa retransmiti-lo caso receba outro `FIN`; e (b) pacotes atrasados ("fantasmas") da conexão antiga morram na rede e não sejam confundidos com uma nova conexão que reutilize o mesmo par de portas.

```text
  CLIENTE                                            SERVIDOR
  Estado: ESTABLISHED                                Estado: ESTABLISHED
        |                                                  |
        |  1. [FIN, ACK]     Seq=X, Ack=Y                  |
        |------------------------------------------------->|
  Estado: FIN_WAIT_1                                 Estado: CLOSE_WAIT
        |                                                  |
        |  2. [ACK]          Seq=Y, Ack=X+1                |
        |<-------------------------------------------------|
  Estado: FIN_WAIT_2                                       |
        |                                                  |
        |  3. [FIN, ACK]     Seq=Y, Ack=X+1                |
        |<-------------------------------------------------|
  Estado: TIME_WAIT (aguarda 2*MSL)                  Estado: LAST_ACK
        |                                                  |
        |  4. [ACK]          Seq=X+1, Ack=Y+1              |
        |------------------------------------------------->|
        |                                            Estado: CLOSED
  Estado: CLOSED (após expirar TIME_WAIT)
```

##### Problemas de Segurança no Handshake TCP e Configuração Segura

1. **Ataque de SYN Flood (Inundação de SYN)**:
   - **Como funciona**: Um atacante (ou *botnet*) envia milhares ou milhões de segmentos `SYN` por segundo para a porta `443` do servidor, usando endereços IP de origem falsificados (*IP Spoofing*). O servidor aloca memória na sua fila de conexões semiabertas (*SYN Backlog Queue*), responde com `SYN-ACK` para os IPs falsos e fica esperando no estado `SYN_RCVD` pelo terceiro passo (`ACK`) que **nunca virá**. Quando a fila enche, o servidor deixa de aceitar novas conexões TCP de usuários legítimos (Negação de Serviço — **DoS/DDoS**).
   - **Defesa — SYN Cookies (`[REQUISITO]`)**: Com **SYN Cookies** ativados no kernel (ex.: `net.ipv4.tcp_syncookies = 1` no Linux), quando a fila de backlog atinge um limite, o servidor **não aloca memória** ao receber o `SYN`. Em vez disso, ele codifica os metadados da conexão (IP/porta de origem e destino, *timestamp* e MSS) dentro do próprio Número de Sequência Inicial (`Seq`) do `SYN-ACK` usando uma função de hash criptográfica com chave secreta local. Se — e somente se — o cliente for real e devolver o terceiro pacote (`ACK` com `Ack = ISN + 1`), o servidor recalcula o hash, valida o cookie e só então aloca a estrutura de memória da conexão!
2. **Previsão de Número de Sequência e TCP Hijacking**:
   - **Como funciona**: Se um sistema operacional gerasse Números de Sequência Iniciais (`ISN`) previsíveis (ex.: incrementando de 1 em 1), um atacante conseguiria adivinhar o `Seq` de uma conexão ativa e injetar pacotes falsificados (`RST` para derrubar a conexão ou dados maliciosos).
   - **Defesa (`[REQUISITO]`)**: Sistemas operacionais modernos implementam a **RFC 6528**, gerando o ISN por meio de uma função pseudoaleatória criptográfica baseada na quádrupla da conexão `(IP Origem, Porta Origem, IP Destino, Porta Destino)` somada a um relógio de alta frequência.
3. **Varredura de Portas (*TCP SYN Scan / Half-Open Scan*)**:
   - **Como funciona**: Em testes de intrusão autorizados, ferramentas como `nmap -sS` enviam apenas o pacote `SYN`. Se o alvo responder `SYN-ACK`, o scanner sabe que a porta está **aberta (`open`)** e envia imediatamente um `RST` para não completar o handshake; se o alvo responder `RST, ACK`, a porta está **fechada (`closed`)**; se não houver resposta (ou retornar erro ICMP de bloqueio), a porta está **filtrada (`filtered`)** por um firewall.
   - **Defesa (`[RECOMENDAÇÃO]`)**: Configurar firewalls para descartar silenciosamente (`DROP`) pacotes destinados a portas fechadas no perímetro externo e monitorar anomalias de varredura via **IDS/IPS** (*Intrusion Detection/Prevention System*).

---

<a id="s4-4"></a>
### 4.4 Endereçamento IP, Sub-redes e Roteamento

Na Camada 3 (Rede / Internet), cada interface conectada precisa de um identificador lógico que indique **em qual rede** ela está localizada e **qual é o host específico** dentro daquela rede. Esse papel cabe ao **Endereço IP** (*Internet Protocol Address*).

<a id="s4-4-1"></a>
#### 4.4.1 IPv4 e IPv6

Atualmente, duas versões do protocolo IP coexistem na Internet (frequentemente operando em **Pilha Dupla / *Dual Stack***, onde o dispositivo possui simultaneamente um endereço IPv4 e um IPv6): o **IPv4** (RFC 791, de 1981) e o **IPv6** (RFC 8200, que atualizou a RFC 2460).

##### IPv4 (Internet Protocol version 4 — RFC 791)
- **Tamanho e formato**: Possui **32 bits** (4 bytes), representados em notação **decimal pontuada** (*dotted-decimal*): quatro octetos de 8 bits separados por pontos, onde cada octeto varia de `0` (`00000000` em binário) a `255` (`11111111` em binário).
  - Exemplo: `192.0.2.10` $\rightarrow$ `11000000.00000000.00000010.00001010`.
- **Espaço total de endereços**: $2^{32} = 4.294.967.296$ endereços (~4,29 bilhões). Devido ao crescimento explosivo de computadores, smartphones, servidores em nuvem e dispositivos IoT (*Internet of Things*), os blocos livres de IPv4 nas autoridades regionais de registro (como o Registro.br / LACNIC) esgotaram-se na década de 2010, mantendo-se vivos graças ao uso intensivo de **NAT** (veja [Seção 4.5.4](#s4-5-4)).
- **Cabeçalho IPv4**: Possui tamanho variável entre **20 e 60 bytes**, com 12 campos obrigatórios, incluindo: *Version* (4), *IHL* (comprimento do cabeçalho), *Total Length*, *Identification / Flags / Fragment Offset* (usados para fragmentação), **TTL** (*Time To Live*), **Protocol** (indica o protocolo da Camada 4 encapsulado: `6` = TCP, `17` = UDP, `1` = ICMP), *Header Checksum*, **Source IP Address** (32 bits) e **Destination IP Address** (32 bits).

##### IPv6 (Internet Protocol version 6 — RFC 8200)
- **Tamanho e formato**: Possui **128 bits** (16 bytes) — quatro vezes a quantidade de bits do IPv4, mas exponencialmente maior em capacidade! É representado por **8 grupos (hextetos) de 4 dígitos hexadecimais** (`0–9` e `a–f`), separados por dois-pontos (`:`), onde cada grupo equivale a 16 bits.
  - Exemplo completo: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`.
- **Espaço total de endereços**: $2^{128} \approx 3,4 \times 10^{38}$ endereços (340 undecilhões de endereços — suficiente para atribuir bilhões de IPs públicos para cada milímetro quadrado da superfície da Terra).
- **Cabeçalho IPv6 simplificado**: Mesmo tendo endereços 4 vezes maiores, o cabeçalho base do IPv6 é **fixo em 40 bytes** e possui apenas **8 campos**, projetado para processamento ultrarrápido em roteadores:
  1. Não há *Header Checksum* na Camada 3 (a verificação de integridade já é feita na Camada 2 pelo Ethernet FCS e na Camada 4 pelo TCP/UDP, evitando que cada roteador precise recalcular o checksum a cada salto).
  2. Roteadores intermediários **não fragmentam pacotes IPv6** (se um pacote for maior que o **MTU** — *Maximum Transmission Unit* do enlace, o roteador o descarta e devolve uma mensagem ICMPv6 *Packet Too Big* para que o próprio host de origem ajuste o tamanho via *Path MTU Discovery*).
  3. O antigo campo *TTL* passa a se chamar **Hop Limit** (Limite de Saltos), que reflete com precisão o seu funcionamento real.
  4. O antigo campo *Protocol* passa a se chamar **Next Header** (Próximo Cabeçalho), permitindo encadear cabeçalhos de extensão opcionais apenas quando necessário.

##### Regras Oficiais de Abreviação do IPv6 (RFC 5952)
Como escrever 32 caracteres hexadecimais é trabalhoso, duas regras permitem comprimir endereços IPv6 (muito cobradas em provas!):
1. **Regra 1 — Omitir zeros à esquerda (*Leading Zeros*)**: Dentro de qualquer um dos 8 grupos de 4 dígitos, todos os zeros à esquerda podem ser removidos (mas deve restar pelo menos um dígito em um grupo de quatro zeros: `0000` vira `0`). Zeros à direita **nunca** podem ser omitidos (`0db0` vira `db0`, nunca `db`).
2. **Regra 2 — Duplo dois-pontos (`::`) para sequência contígua de grupos zero**: Uma sequência de um ou mais grupos consecutivos compostos inteiramente por zeros (`:0000:0000:...`) pode ser substituída por **`::`**.
   - **Atenção (`[REQUISITO]` da RFC)**: O símbolo `::` só pode ser utilizado **UMA ÚNICA VEZ** em todo o endereço IPv6! Se fosse usado duas vezes (ex.: `2001::8a2e::7334`), o computador não saberia quantos grupos de zeros colocar na primeira posição e quantos na segunda para completar os 8 hextetos.

```text
Exemplo de abreviação passo a passo:
Endereço original (128 bits):  2001:0db8:0000:0000:0000:00ff:0000:0042
Passo 1 (zeros à esquerda):    2001:db8:0:0:0:ff:0:42
Passo 2 (comprimir com ::):    2001:db8::ff:0:42

Exemplo do endereço de Loopback IPv6:
Original:                      0000:0000:0000:0000:0000:0000:0000:0001
Abreviado:                     ::1
```

##### Tipos de Endereços e Endereços Especiais (IPv4 e IPv6)

- **Unicast**: Identifica uma única interface de rede (comunicação **1 para 1**). Ex.: uma requisição HTTP entre seu navegador e um servidor web.
- **Multicast**: Identifica um grupo de interfaces interessadas (comunicação **1 para muitos específicos**). O pacote é entregue apenas aos hosts que se inscreveram naquele grupo multicast. Faixa IPv4: `224.0.0.0/4` (`224.0.0.0` a `239.255.255.255`); Faixa IPv6: `ff00::/8`.
- **Broadcast (exclusivo do IPv4)**: Entrega um pacote para **todos os nós** de um domínio de sub-rede (**1 para todos**). Exemplo de broadcast limitado: `255.255.255.255`. **O IPv6 eliminou o Broadcast**, substituindo todas as suas funções por grupos **Multicast** direcionados (o que evita tempestades de broadcast que degradavam redes locais grandes).
- **Anycast**: Um mesmo endereço IP é atribuído a múltiplos servidores distribuídos geograficamente, e o protocolo de roteamento (BGP) entrega o pacote ao nó **topologicamente mais próximo** (**1 para o mais próximo de muitos**). Amplamente utilizado em servidores DNS Raiz, `1.1.1.1` (Cloudflare), `8.8.8.8` (Google) e redes **CDN** (*Content Delivery Network*).
- **Loopback (Endereço de Retorno Local)**: Usado para que um host envie tráfego de rede para si mesmo sem que o pacote saia pela placa de rede física (comum ao acessar `http://localhost:3000` durante o desenvolvimento).
  - No **IPv4**: Todo o bloco **`127.0.0.0/8`** (`127.0.0.1` até `127.255.255.254`), sendo **`127.0.0.1`** o padrão universal.
  - No **IPv6**: Apenas o endereço único **`::1/128`**.
- **Link-Local (Endereçamento Automático de Enlace Local)**: Endereços válidos apenas dentro do enlace local imediato (não são roteados por nenhum roteador).
  - No **IPv4**: Bloco **`169.254.0.0/16`** (**APIPA** — *Automatic Private IP Addressing*, RFC 3927). Quando um computador está configurado para obter IP via DHCP, mas nenhum servidor DHCP responde na rede, o sistema operacional se autoatribui um IP `169.254.x.x` para permitir comunicação mínima no cabo local. *(Nota de Segurança: Em provedores de nuvem como AWS, GCP e Azure, o IP link-local **`169.254.169.254`** hospeda o **Serviço de Metadados da Instância — IMDS**, alvo número 1 de ataques de [SSRF](./03a-vulnerabilidades-parte1.md#s3-4)).*
  - No **IPv6**: Bloco **`fe80::/10`**. Diferente do IPv4, no IPv6 toda interface habilitada **obrigatoriamente** possui um endereço *link-local* `fe80::...` sempre ativo, usado para descoberta de vizinhos (**NDP** — *Neighbor Discovery Protocol*) e comunicação com o gateway padrão.
- **Endereço Não Especificado (*Unspecified Address*)**: **`0.0.0.0`** (IPv4) e **`::`** (IPv6). Representa a ausência de endereço (usado por um cliente DHCP antes de receber seu IP) ou, quando configurado em um servidor de aplicação (ex.: `app.listen(8080, '0.0.0.0')`), instrui o sistema operacional a escutar conexões em **todas as interfaces de rede** da máquina.

##### Tabela Comparativa: IPv4 vs. IPv6

| Característica | IPv4 (RFC 791) | IPv6 (RFC 8200) |
| :--- | :--- | :--- |
| **Tamanho do Endereço** | **32 bits** (4 bytes) | **128 bits** (16 bytes) |
| **Notação Textual** | Decimal pontuada (ex.: `192.0.2.1`) | Hexadecimal separada por dois-pontos (ex.: `2001:db8::1`) |
| **Quantidade de Endereços** | $2^{32} \approx 4,29 \times 10^9$ (~4,3 bilhões) | $2^{128} \approx 3,4 \times 10^{38}$ (~340 undecilhões) |
| **Tamanho do Cabeçalho L3** | **Variável: 20 a 60 bytes** (12 campos base + opções) | **Fixo: 40 bytes** (8 campos base; opções via *Extension Headers*) |
| **Broadcast** | **Sim** (usa endereços de broadcast na sub-rede) | **Não existe** (substituído por **Multicast** eficiente) |
| **Resolução de Endereço Físico (MAC)** | **ARP** (*Address Resolution Protocol* via broadcast L2) | **NDP** (*Neighbor Discovery Protocol* via ICMPv6 Multicast) |
| **Fragmentação de Pacotes** | Realizada pelo host emissor **e** por roteadores intermediários | Realizada **apenas pelo host emissor** (roteadores nunca fragmentam) |
| **Checksum no Cabeçalho IP** | **Sim** (recalculado a cada salto de roteador) | **Não** (delegado às Camadas 2 e 4 para maior performance) |
| **Configuração Automática Sem Estado** | Apenas APIPA local (`169.254.0.0/16`) ou exige servidor DHCPv4 | **SLAAC** (*Stateless Address Autoconfiguration*) nativo via *Router Advertisements*, além de DHCPv6 |
| **Endereço de Loopback** | **`127.0.0.1`** (bloco `127.0.0.0/8`) | **`::1`** (`/128`) |
| **Endereço Link-Local** | **`169.254.0.0/16`** (quando o DHCP falha) | **`fe80::/10`** (obrigatório em toda interface IPv6) |

---

<a id="s4-4-2"></a>
#### 4.4.2 Endereço IP público e privado (RFC 1918 e RFC 4193)

Nem todo endereço IP pode trafegar livremente na Internet global. A arquitetura da Internet divide o espaço de endereçamento em duas categorias fundamentais de escopo:

1. **Endereço IP Público (*Public / Global IP Address*)**:
   - É globalmente único e **roteável na Internet pública**.
   - É alocado pela **IANA** (*Internet Assigned Numbers Authority*) aos registros regionais (**RIRs**, como o **LACNIC / Registro.br** na América Latina/Brasil, ARIN na América do Norte, RIPE NCC na Europa) e repassado a provedores de Internet (**ISPs**), empresas e provedores de nuvem.
   - Qualquer servidor diretamente acessível pela Internet (ou a borda de um roteador doméstico/empresarial) precisa de pelo menos um endereço IP público.
2. **Endereço IP Privado (*Private / Local IP Address*)**:
   - Reservado exclusivamente para uso dentro de **redes internas privadas** (residências, escritórios, datacenters, VPCs em nuvem e clusters Kubernetes).
   - **Não é roteável na Internet pública**: roteadores de borda da Internet descartam imediatamente qualquer pacote cujo endereço de destino (ou de origem, via filtro BCP 38) pertença a uma faixa privada.
   - Como essas redes são isoladas umas das outras, milhões de casas e empresas ao redor do mundo podem usar simultaneamente o mesmo IP privado (como `192.168.1.10`) sem qualquer conflito! Para acessar a Internet, o roteador de borda traduz o IP privado em um IP público por meio de **NAT** (veja [Seção 4.5.4](#s4-5-4)).

##### Faixas de Endereços Privados e Especiais (Tabela Oficial para Prova)

A **RFC 1918** define os três blocos clássicos de endereços IPv4 privados, enquanto a **RFC 6598** define o bloco de **CGNAT** (*Carrier-Grade NAT*, usado por operadoras de banda larga e telefonia móvel) e a **RFC 4193** define os endereços privados do IPv6 (**ULA** — *Unique Local Addresses*):

| Padrão / RFC | Nome da Faixa / Uso | Prefixo CIDR | Intervalo Completo de Endereços | Quantidade de Endereços |
| :--- | :--- | :--- | :--- | :--- |
| **RFC 1918** | **IPv4 Privado — Bloco de 24 bits** (Antiga Classe A privada) | **`10.0.0.0/8`** | `10.0.0.0` a `10.255.255.255` | $2^{24} = 16.777.216$ |
| **RFC 1918** | **IPv4 Privado — Bloco de 20 bits** (Antiga Classe B privada: 16 redes `/16`) | **`172.16.0.0/12`** | `172.16.0.0` a `172.31.255.255` | $2^{20} = 1.048.576$ |
| **RFC 1918** | **IPv4 Privado — Bloco de 16 bits** (Antiga Classe C privada: 256 redes `/24`) | **`192.168.0.0/16`** | `192.168.0.0` a `192.168.255.255` | $2^{16} = 65.536$ |
| **RFC 6598** | **CGNAT / Shared Address Space** (NAT de operadora entre ISP e modem do cliente) | **`100.64.0.0/10`** | `100.64.0.0` a `100.127.255.255` | $2^{22} = 4.194.304$ |
| **RFC 5737** | **Documentação e Exemplos (TEST-NET-1, 2 e 3)** (Reservados para livros/provas) | **`192.0.2.0/24`**<br>**`198.51.100.0/24`**<br>**`203.0.113.0/24`** | `192.0.2.0–255`<br>`198.51.100.0–255`<br>`203.0.113.0–255` | 256 cada |
| **RFC 4193** | **IPv6 ULA (*Unique Local Address*)** (Equivalente ao RFC 1918 no IPv6) | **`fc00::/7`**<br>(Na prática: **`fd00::/8`**) | `fc00::` a `fdff:ffff:ffff:ffff:ffff:ffff:ffff:ffff` | $2^{121}$ |
| **IANA IPv6** | **IPv6 GUA (*Global Unicast Address*)** (IPv6 Público roteável na Internet) | **`2000::/3`** | `2000::` a `3fff:ffff:ffff:ffff:ffff:ffff:ffff:ffff` | $2^{125}$ |

> [!WARNING]
> **Pegadinha clássica de prova na faixa `172.16.0.0/12`:**
> Muitos estudantes memorizam incorretamente que "qualquer IP começando com `172.` é privado". **Falso!** Apenas os endereços cujo segundo octeto está entre **`16` e `31`** (`172.16.x.x` até `172.31.x.x`) são privados pela RFC 1918. Um endereço como `172.32.10.5` ou `172.15.0.1` é um **endereço IP público** legítimo na Internet! De forma semelhante, `100.64.0.0/10` (CGNAT) vai apenas de `100.64.x.x` a `100.127.x.x`.

##### Problemas de Segurança Relacionados a IPs Públicos e Privados
- **Bypass de proteção contra SSRF (*Server-Side Request Forgery*)**: Quando um servidor back-end aceita buscar uma URL fornecida pelo usuário (ex.: gerar prévia de um link ou importar um webhook), um atacante pode informar `http://10.0.0.15:8080/admin`, `http://192.168.1.1/` ou `http://169.254.169.254/latest/meta-data/` para fazer o servidor varrer a rede interna privada que estaria inacessível diretamente da Internet. **[REQUISITO]** Toda rotina que realiza requisições HTTP a partir de URLs externas no servidor deve resolver o DNS primeiro e **bloquear conexões** cujo IP resolvido pertença às faixas privadas/reservadas (`127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.0.0/16`, `100.64.0.0/10`, `0.0.0.0/8`, `::1/128`, `fc00::/7`, `fe80::/10`).
- **Vazamento de topologia interna (*Internal IP Disclosure*)**: Cabeçalhos HTTP de proxies mal configurados (`X-Forwarded-For`, `Via`, `X-Real-IP`) ou mensagens de erro de banco de dados frequentemente revelam endereços `10.x.x.x` na aba **Network** do DevTools (veja [Análise de Segurança na Aba Network](./02a-devtools-network.md#s2-1-9)), auxiliando um atacante a mapear sub-redes internas. **[BOA PRÁTICA]** Sanitizar cabeçalhos de roteamento interno no proxy reverso de borda antes de enviar a resposta ao cliente.

---

<a id="s4-4-3"></a>
#### 4.4.3 Máscara de rede, Sub-redes e CIDR

Um endereço IP sozinho não diz onde termina a identificação da **Rede** e onde começa a identificação do **Host** (máquina). Quem define essa fronteira é a **Máscara de Sub-rede (*Subnet Mask*)** ou o **Prefixo CIDR**.

##### Como funciona a Máscara de Rede e a Notação CIDR (RFC 4632)
No IPv4, uma máscara de sub-rede também possui **32 bits**, composta por uma sequência contígua de bits **`1` à esquerda** (que representam a porção de **Rede** / Sub-rede) seguida por uma sequência contígua de bits **`0` à direita** (que representam a porção de **Host**).

Historicamente (nos anos 1980), utilizava-se o endereçamento com classes fixas (**Classful Addressing** — Classes A `/8`, B `/16` e C `/24`, hoje **`[Obsoleto]`**), que desperdiçava milhões de endereços (uma empresa que precisasse de 300 IPs não cabia numa Classe C de 254 hosts e recebia uma Classe B inteira com 65.534 hosts!). Em 1993, a **RFC 1519** (atualizada pela **RFC 4632**) introduziu o **CIDR** (*Classless Inter-Domain Routing* — Roteamento Entre Domínios Sem Classes), permitindo cortar a máscara em qualquer bit de `/0` a `/32`.

A notação CIDR escreve uma barra seguida do número de bits `1` da máscara:
- **`/24`** significa **24 bits `1`** (rede) e $32 - 24 = \mathbf{8\text{ bits }0}$ (host):
  - Binário: `11111111.11111111.11111111.00000000`
  - Decimal: **`255.255.255.0`**

##### As 4 Regras de Ouro do Cálculo de Sub-rede IPv4

Dado um endereço IP e um prefixo `/p`, onde **$h = 32 - p$** é o número de bits reservados para **Host**:

1. **Endereço de Rede (*Network Address*)**: Obtido aplicando a operação lógica **`AND` bit a bit** entre o endereço IP e a máscara de sub-rede (ou simplesmente **zerando todos os $h$ bits de host**). É o **primeiro endereço** do bloco (sempre par) e identifica a sub-rede em tabelas de roteamento; **não pode ser atribuído a um computador**.
2. **Endereço de Broadcast (*Broadcast Address*)**: Obtido **preenchendo todos os $h$ bits de host com `1`**. É o **último endereço** do bloco (sempre ímpar) e serve para enviar pacotes a todos os membros daquela sub-rede; **não pode ser atribuído a um computador**.
3. **Total de Endereços IP no bloco**:
   $$\text{Total de IPs} = 2^h = 2^{(32 - p)}$$
4. **Total de Hosts Utilizáveis (Válidos)**: Descontam-se os 2 endereços reservados (1 de Rede + 1 de Broadcast):
   $$\text{Hosts Utilizáveis} = 2^h - 2 = 2^{(32 - p)} - 2$$
   - **Primeiro Host Útil**: $\text{Endereço de Rede} + 1$.
   - **Último Host Útil**: $\text{Endereço de Broadcast} - 1$.

> [!NOTE]
> **Exceções à fórmula $2^h - 2$ (`/32` e `/31`):**
> - Um prefixo **`/32`** ($h=0$, máscara `255.255.255.255`) representa **um único host exato** ($2^0 = 1$ endereço, usado em rotas de host, interfaces de loopback e regras de firewall para um IP específico).
> - Um prefixo **`/31`** ($h=1$, máscara `255.255.255.254`), padronizado pela **RFC 3021**, possui 2 endereços totais e é utilizado exclusivamente em **links ponto a ponto** diretos entre dois roteadores (onde não há necessidade de broadcast, aproveitando-se ambos os IPs para as duas pontas do cabo).

##### Exemplo Prático Resolvido (Passo a Passo de Prova)

**Questão de Exemplo**: Um servidor web possui o endereço IP **`192.168.10.150/26`**. Calcule: (a) a máscara em decimal, (b) o endereço de rede, (c) o endereço de broadcast, (d) a faixa de hosts úteis e (e) a quantidade de hosts utilizáveis.

```text
PASSO 1: Analisar o prefixo /26
- Bits de Rede (p) = 26 bits '1' -> 11111111.11111111.11111111.11000000
- Bits de Host (h) = 32 - 26 = 6 bits '0'
- Os 3 primeiros octetos são 255.255.255. O corte acontece no 4º octeto!
- Valor do 4º octeto da máscara: 128 + 64 = 192.
  => (a) Máscara decimal: 255.255.255.192

PASSO 2: Encontrar o tamanho do bloco ("salto") e a quantidade de hosts
- Tamanho do bloco no 4º octeto = 2^h = 2^6 = 64 endereços por sub-rede.
- As sub-redes /26 começam nos múltiplos de 64:
  Sub-rede 0: 192.168.10.0   a 192.168.10.63
  Sub-rede 1: 192.168.10.64  a 192.168.10.127
  Sub-rede 2: 192.168.10.128 a 192.168.10.191  <-- O IP .150 está aqui! (128 <= 150 <= 191)
  Sub-rede 3: 192.168.10.192 a 192.168.10.255

PASSO 3: Extrair os resultados da Sub-rede 2 (192.168.10.128/26)
  => (b) Endereço de Rede:         192.168.10.128  (em binário no 4º octeto: 10000000)
  => (c) Endereço de Broadcast:    192.168.10.191  (próxima rede 192 menos 1; binário: 10111111)
  => (d) Primeiro Host Útil:       192.168.10.129  (Rede + 1)
         Último Host Útil:         192.168.10.190  (Broadcast - 1)
  => (e) Quantidade de Hosts Úteis: 2^6 - 2 = 64 - 2 = 62 hosts utilizáveis.
```

##### VLSM (Variable Length Subnet Masking — Máscara de Sub-rede de Comprimento Variável)
O **VLSM** é a técnica de subdividir uma sub-rede em sub-redes menores com **tamanhos diferentes (prefixos diferentes)** de acordo com a necessidade real de cada departamento ou VLAN. Por exemplo, a partir de um bloco `192.168.1.0/24` (256 IPs):
- Aloca-se `192.168.1.0/25` (126 hosts úteis) para a VLAN de Usuários Corporativos;
- Divide-se a metade restante (`192.168.1.128/25`) em duas `/26`: `192.168.1.128/26` (62 hosts úteis) para a VLAN de Servidores de Aplicação;
- Divide-se `192.168.1.192/26` em `/28` (`192.168.1.192/28`, 14 hosts) para a VLAN de Bancos de Dados e `/30` (`192.168.1.208/30`, 2 hosts) para o enlace ponto a ponto com o Firewall.
- **Benefício de Segurança (`[BOA PRÁTICA]`)**: Segmentar redes via VLANs e sub-redes VLSM permite aplicar listas de controle de acesso (**ACLs**) no roteador/firewall entre sub-redes, impedindo a movimentação lateral direta de um atacante entre a rede de visitantes e a sub-rede de bancos de dados.

##### Tabela de Referência Rápida CIDR (`/8` a `/32`)

| Prefixo CIDR | Máscara de Sub-rede Decimal | Bits de Host ($h$) | Total de IPs ($2^h$) | Hosts Utilizáveis ($2^h - 2$) | Observação / Equivalência |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`/8`** | `255.0.0.0` | 24 | $16.777.216$ | $16.777.214$ | Antiga Classe A (ex.: `10.0.0.0/8`) |
| **`/12`** | `255.240.0.0` | 20 | $1.048.576$ | $1.048.574$ | Bloco privado `172.16.0.0/12` |
| **`/16`** | `255.255.0.0` | 16 | $65.536$ | $65.534$ | Antiga Classe B / Bloco `192.168.0.0/16` |
| **`/20`** | `255.255.240.0` | 12 | $4.096$ | $4.094$ | 16 redes `/24` contíguas (comum em VPCs) |
| **`/24`** | `255.255.255.0` | 8 | $256$ | **$254$** | Antiga Classe C (sub-rede LAN mais comum) |
| **`/25`** | `255.255.255.128` | 7 | $128$ | **$126$** | Metade de uma `/24` (2 sub-redes de 128 IPs) |
| **`/26`** | `255.255.255.192` | 6 | $64$ | **$62$** | Um quarto de uma `/24` (4 sub-redes de 64 IPs) |
| **`/27`** | `255.255.255.224` | 5 | $32$ | **$30$** | 8 sub-redes de 32 IPs dentro de uma `/24` |
| **`/28`** | `255.255.255.240` | 4 | $16$ | **$14$** | 16 sub-redes de 16 IPs dentro de uma `/24` |
| **`/29`** | `255.255.255.248` | 3 | $8$ | **$6$** | Pequenas DMZs ou enlaces com redundância (VRRP) |
| **`/30`** | `255.255.255.252` | 2 | $4$ | **$2$** | Clássica para links ponto a ponto entre 2 roteadores |
| **`/31`** | `255.255.255.254` | 1 | $2$ | $2$ (RFC 3021) | Links ponto a ponto sem endereço de broadcast |
| **`/32`** | `255.255.255.255` | 0 | $1$ | $1$ (Host único) | Rota de host único / Regra de firewall para 1 IP |

---

<a id="s4-4-4"></a>
#### 4.4.4 Gateway Padrão e Roteamento

##### O que é o Gateway Padrão (*Default Gateway*) e como o Host toma decisões de envio
Quando o seu computador (`192.168.1.10/24`) precisa enviar um pacote IP para um destino qualquer, a pilha TCP/IP do sistema operacional faz uma pergunta matemática simples antes de colocar o pacote no cabo: **"O endereço IP de destino pertence à minha própria sub-rede local ou está em outra rede?"**

Para responder, o sistema operacional aplica a sua própria máscara (`255.255.255.0`) ao seu IP e ao IP de destino:
- **Cenário A — Destino na MESMA sub-rede** (ex.: enviar para a impressora `192.168.1.50`):
  - Como ambos resultam na rede `192.168.1.0/24`, a entrega é **direta (local)**. O computador usa o protocolo **ARP** para descobrir o endereço MAC da impressora (`192.168.1.50`) e envia o quadro Ethernet diretamente para ela através do switch L2, **sem passar pelo roteador**.
- **Cenário B — Destino em OUTRA sub-rede** (ex.: acessar `203.0.113.50` na Internet):
  - Como `203.0.113.50` não pertence a `192.168.1.0/24`, a entrega é **remota (indireta)**. O computador consulta sua **Tabela de Roteamento** local em busca de uma rota e encontra a **Rota Padrão (`0.0.0.0/0`)**, que aponta para o endereço IP do **Gateway Padrão** (o roteador local na mesma sub-rede, ex.: `192.168.1.1`).

> [!IMPORTANT]
> **Conceito fundamental para provas (O que muda e o que NÃO muda a cada salto de roteador):**
> Quando o computador envia um pacote para um servidor na Internet (`203.0.113.50`) através do Gateway Padrão (`192.168.1.1`):
> 1. No **Pacote IP (Camada 3)**, o **IP de Destino continua sendo o servidor final (`203.0.113.50`)**! (Exceto quando há tradução NAT, os endereços IP de origem e destino permanecem os mesmos do início ao fim da jornada).
> 2. No **Quadro Ethernet (Camada 2)**, o **MAC de Destino é o endereço MAC do Gateway Padrão (`192.168.1.1`)**!
> A cada **salto (*hop*)** por um roteador no caminho da Internet, o roteador desencapsula e descarta o cabeçalho da Camada 2 antigo, consulta sua tabela de roteamento, **decrementa o campo TTL em 1** no cabeçalho da Camada 3, e cria um **novo quadro da Camada 2** com o seu próprio MAC de saída como origem e o MAC do próximo roteador (*next-hop*) como destino.

##### Tabela de Roteamento, Rota Padrão (`0.0.0.0/0`) e *Longest Prefix Match*
Todo roteador (e todo computador) mantém uma **Tabela de Roteamento**. Quando um pacote chega e existem múltiplas rotas na tabela que englobam o endereço IP de destino, o roteador utiliza a regra determinística do **Casamento de Prefixo Mais Longo (*Longest Prefix Match* — LPM)**: **vence sempre a rota que tiver a máscara mais específica (o maior número após a barra `/`)**.

Exemplo de Tabela de Roteamento em um Roteador Corporativo:

```text
Destino (Prefixo)      Próximo Salto (Next-Hop)   Interface   Explicação
--------------------   ------------------------   ---------   ---------------------------------------------------
10.20.30.100/32        10.0.0.2                   eth1        Rota de host específico (/32 é o mais específico possível)
10.20.30.0/24          10.0.0.1                   eth1        Rota para a sub-rede de servidores
10.20.0.0/16           10.0.0.5                   eth2        Rota sumarizada da filial
192.168.1.0/24         Diretamente Conectado      eth0        Rede local (LAN) conectada diretamente ao roteador
0.0.0.0/0              198.51.100.1               wan0        Rota Padrão (Gateway de Último Recurso — Internet)
```

- Se chegar um pacote para **`10.20.30.55`**, ele casa com `0.0.0.0/0` (0 bits), com `10.20.0.0/16` (16 bits) e com **`10.20.30.0/24`** (24 bits). Pelo *Longest Prefix Match*, `/24 > /16 > /0`, então o pacote é enviado para `10.0.0.1`.
- Se chegar um pacote para **`203.0.113.50`**, ele não casa com nenhuma rota interna específica, caindo na **Rota Padrão (`0.0.0.0/0` no IPv4 ou `::/0` no IPv6)**, que encaminha tudo o que sobra para o provedor de Internet (`198.51.100.1`).

##### Salto (*Hop*) e TTL (*Time To Live* / *Hop Limit*)
O campo **TTL** (8 bits no cabeçalho IPv4, variando de `0` a `255`; chamado de **Hop Limit** no IPv6) não mede segundos no relógio, mas sim a **quantidade máxima de roteadores (saltos)** pelos quais o pacote pode passar antes de ser descartado.
- **Para que serve**: Evitar que um pacote fique circulando infinitamente pela Internet caso ocorra um **loop de roteamento** (ex.: Roteador A manda para B, que manda de volta para A).
- **Como funciona**: O sistema operacional de origem cria o pacote com um TTL inicial padrão (tipicamente `64` no Linux/macOS/Android ou `128` no Windows). Cada roteador que encaminha o pacote subtrai `1` do TTL (`TTL = TTL - 1`). Se o TTL chegar a **`0`** em um roteador, o roteador **descarta o pacote** e envia de volta ao IP de origem uma mensagem **ICMP Tipo 11, Código 0 (*Time Exceeded in Transit*)**.

##### Roteamento Estático vs. Roteamento Dinâmico (OSPF e BGP)
- **Roteamento Estático**: As rotas são inseridas manualmente pelo administrador de rede (`ip route add ...`). É previsível, não consome largura de banda nem CPU com anúncios de rota, sendo ideal para redes pequenas ou rotas padrão de borda, mas não se adapta sozinho se um cabo romper.
- **Roteamento Dinâmico**: Os roteadores trocam informações de topologia automaticamente e recalculam caminhos alternativos em segundos caso um link caia. Divide-se em duas famílias:
  1. **IGP (*Interior Gateway Protocol*) — Dentro de um mesmo Sistema Autônomo (empresa/ISP)**: O padrão de mercado é o **OSPF** (*Open Shortest Path First*), um protocolo de estado de enlace (*Link-State*) que usa o algoritmo de **Dijkstra** para calcular a árvore de caminho mais curto com base no custo (largura de banda) das interfaces.
  2. **EGP (*Exterior Gateway Protocol*) — Entre Sistemas Autônomos diferentes na Internet**: O protocolo que cola toda a Internet mundial é o **BGP** (*Border Gateway Protocol*, operando na porta **TCP 179**). O BGP é um protocolo de vetor de caminho (*Path-Vector*) baseado em políticas comerciais e de segurança entre Sistemas Autônomos identificados por seu **ASN** (*Autonomous System Number*, ex.: AS15169 da Google, AS13335 da Cloudflare).

---

<a id="s4-5"></a>
### 4.5 Protocolos Essenciais de Rede e Enlace

Para que um pacote IP consiga sair da placa de rede do cliente, atravessar o switch local, encontrar o roteador, descobrir o IP de um domínio e alcançar a Internet, uma engrenagem de protocolos auxiliares opera silenciosamente nas Camadas 2, 3 e 7: **ARP**, **DHCP**, **DNS**, **NAT** e **ICMP**.

<a id="s4-5-1"></a>
#### 4.5.1 MAC Address e ARP (Address Resolution Protocol)

##### O Endereço MAC (Media Access Control)
Enquanto o endereço IP (Camada 3) é um endereço **lógico e hierárquico** que muda conforme o dispositivo se conecta a redes diferentes, o **MAC Address** (Camada 2) é o endereço **físico** (também chamado de *Hardware Address* ou *Burned-In Address* — BIA) gravado na interface de rede (**NIC**).
- **Formato e Tamanho**: Possui **48 bits** (6 bytes), representados por **12 dígitos hexadecimais** agrupados de dois em dois (separados por dois-pontos `:`, hífens `-` ou em três grupos de quatro dígitos separados por ponto `.` em equipamentos Cisco):
  - Exemplo: `00:1A:2B:3C:4D:5E` ou `00-1A-2B-3C-4D-5E`.
- **Estrutura (OUI + NIC Specific)**:
  - **Primeiros 24 bits (3 primeiros bytes — ex.: `00:1A:2B`)**: Chamados de **OUI** (*Organizationally Unique Identifier* — Identificador Único Organizacional), atribuídos pelo **IEEE** ao fabricante da placa de rede (ex.: Intel, Realtek, Apple, Dell, VMware). *(Isso permite que ferramentas de inventário ou scanners de rede identifiquem a marca do dispositivo apenas lendo os 3 primeiros bytes do MAC).*
  - **Últimos 24 bits (3 últimos bytes — ex.: `3C:4D:5E`)**: Identificador sequencial exclusivo atribuído pelo fabricante àquela placa específica.
- **Endereço MAC de Broadcast**: Quando todos os 48 bits são preenchidos com `1`, obtém-se **`FF:FF:FF:FF:FF:FF`**. Todo switch Layer 2 que recebe um quadro com destino `FF:FF:FF:FF:FF:FF` replica e envia esse quadro por **todas as portas ativas** daquela mesma VLAN (exceto a porta por onde o quadro entrou).

> [!NOTE]
> **Privacidade e Randomização de MAC (`[BOA PRÁTICA]`):**
> Embora o MAC gravado no chip seja fixo, sistemas operacionais modernos (iOS, Android, Windows 11, macOS e Linux NetworkManager) ativam por padrão a **Randomização de Endereço MAC** ao se conectarem a redes Wi-Fi, evitando que estabelecimentos comerciais ou terceiros rastreiem o deslocamento físico de uma pessoa pelo endereço de hardware do seu smartphone.

##### Como funciona o ARP (Address Resolution Protocol — RFC 826)
Imagine que o Host A (`192.168.1.10`) quer enviar um pacote para o Gateway Padrão (`192.168.1.1`). O Host A sabe o **IP** de destino (`192.168.1.1`), mas a placa de rede Ethernet (Camada 2) não entende endereços IP — ela precisa preencher o campo **MAC de Destino** do quadro Ethernet!

Para descobrir qual é o endereço MAC associado a um endereço IPv4 conhecido na mesma rede local, utiliza-se o protocolo **ARP**:

1. **Consulta à Tabela ARP (Cache ARP)**: Antes de perguntar na rede, o Host A consulta sua própria **Tabela ARP** em memória RAM (visível com o comando `arp -a` ou `ip neigh`). Se o mapeamento `192.168.1.1 -> AA:BB:CC:00:00:01` já estiver lá, ele monta o quadro imediatamente.
2. **ARP Request (Broadcast L2)**: Se o IP não estiver na tabela ARP, o Host A pausa o envio do pacote e dispara um **ARP Request** em **Broadcast (`FF:FF:FF:FF:FF:FF`)** para todas as máquinas da sub-rede:
   - *"Atenção todos (`FF:FF:FF:FF:FF:FF`): Quem possui o endereço IP `192.168.1.1`? Por favor, responda para `192.168.1.10` no MAC `00:11:22:33:44:55`!"*
3. **ARP Reply (Unicast L2)**: Todos os computadores da VLAN recebem o broadcast, mas apenas o dono legítimo do IP `192.168.1.1` responde, enviando um **ARP Reply** diretamente em **Unicast** para `00:11:22:33:44:55`:
   - *"Olá `192.168.1.10`, o IP `192.168.1.1` pertence a mim e meu endereço MAC é `AA:BB:CC:00:00:01`."*
4. **Atualização do Cache**: O Host A grava essa entrada na sua Tabela ARP (com um tempo de expiração típico de alguns minutos) e finalmente transmite o pacote IP encapsulado no quadro Ethernet.

```text
================================================================================
                  FUNCIONAMENTO DO PROTOCOLO ARP (CAMADA 2/3)
================================================================================

  HOST A (Cliente)                                       ROTEADOR (Gateway)
  IP:  192.168.1.10                                      IP:  192.168.1.1
  MAC: 00:11:22:33:44:55                                 MAC: AA:BB:CC:00:00:01
        |                                                       |
        |  1. ARP Request (BROADCAST: Dest MAC = FF:FF:FF:FF:FF:FF)
        |     "Quem tem o IP 192.168.1.1? Diga a 192.168.1.10"  |
        |======================================================>| (Vai para todos na LAN)
        |                                                       |
        |  2. ARP Reply (UNICAST: Dest MAC = 00:11:22:33:44:55) |
        |     "192.168.1.1 está no MAC AA:BB:CC:00:00:01"       |
        |<------------------------------------------------------| (Vai só para o Host A)
```

##### Problema de Segurança: ARP Spoofing / ARP Poisoning e Mitigação
- **Por que acontece**: O protocolo ARP, desenhado em 1982 para redes confiáveis, é completamente **sem estado (*stateless*) e sem autenticação**. Um computador aceita e grava em sua Tabela ARP qualquer *ARP Reply* que chegue pela rede, **mesmo que ele nunca tenha enviado um *ARP Request* antes**!
- **O Ataque (*Man-in-the-Middle* na LAN)**: Um atacante conectado à mesma rede local (VLAN) envia pacotes *ARP Reply* falsificados continuamente para a vítima (`192.168.1.10`) dizendo *"O IP do Gateway `192.168.1.1` está no meu MAC (MAC do Atacante)"*, e simultaneamente envia *ARP Replies* falsos para o Gateway dizendo *"O IP da vítima `192.168.1.10` está no meu MAC"*. Com as duas tabelas ARP envenenadas, todo o tráfego entre a vítima e a Internet passa a atravessar silenciosamente a máquina do atacante!
- **Defesa e Configuração Segura**:
  - **[REQUISITO]** Uso de criptografia nas camadas superiores (**HTTPS / TLS 1.3**, **SSH**, **VPN**): mesmo que o atacante intercepte os quadros na Camada 2 via ARP Spoofing, ele verá apenas dados cifrados e não conseguirá ler senhas ou cookies `Secure` sem gerar um alerta crítico de certificado inválido no navegador.
  - **[RECOMENDAÇÃO]** Ativar **DAI (*Dynamic ARP Inspection*)** em conjunto com **DHCP Snooping** nos switches corporativos gerenciáveis: o switch mantém uma tabela confiável de associação `(MAC <-> IP <-> Porta Física)` construída durante a concessão do DHCP e **descarta automaticamente** qualquer pacote ARP cujo par IP/MAC não corresponda ao registro oficial daquela porta.
  - **[BOA PRÁTICA]** Ativar **Client Isolation** (Isolamento de Clientes / *Private VLAN*) em redes Wi-Fi de visitantes, impedindo que um dispositivo sem fio envie quadros diretamente para outro dispositivo sem fio no mesmo Access Point.

---

<a id="s4-5-2"></a>
#### 4.5.2 DHCP (Dynamic Host Configuration Protocol)

Para que um computador recém-conectado ao cabo ou ao Wi-Fi possa navegar na Web sem que o usuário precise digitar manualmente endereço IP, máscara, gateway e servidor DNS, utiliza-se o **DHCP** (RFC 2131), que opera na Camada de Aplicação sobre **UDP** usando duas portas bem conhecidas:
- **Porta UDP 67**: Porta na qual o **Servidor DHCP** escuta requisições.
- **Porta UDP 68**: Porta na qual o **Cliente DHCP** recebe respostas.

##### O Fluxo DORA (Discover, Offer, Request, Acknowledge)
A negociação de um endereço IP via DHCP ocorre em **4 mensagens sequenciais**, imortalizadas pelo acrônimo **DORA**:

1. **D — DHCP Discover (Cliente $\rightarrow$ Broadcast)**:
   - O cliente acaba de ligar a interface de rede e ainda **não tem endereço IP**.
   - Ele monta um datagrama UDP com **IP de Origem `0.0.0.0`** (porta `68`) e **IP de Destino `255.255.255.255`** (porta `67`), encapsulado em um quadro Ethernet com MAC de Destino `FF:FF:FF:FF:FF:FF`, perguntando: *"Existe algum servidor DHCP nesta rede?"*
2. **O — DHCP Offer (Servidor $\rightarrow$ Cliente)**:
   - O servidor DHCP recebe o *Discover*, separa temporariamente um endereço IP livre de seu *pool* (ex.: `192.168.1.105`) e envia um **DHCP Offer** oferecendo esse IP, a máscara de sub-rede e o tempo de concessão (*Lease Time*).
3. **R — DHCP Request (Cliente $\rightarrow$ Broadcast)**:
   - Como pode haver mais de um servidor DHCP na rede (para alta disponibilidade) e o cliente pode ter recebido múltiplas ofertas, o cliente envia um **DHCP Request** em **Broadcast** dizendo: *"Eu aceito formalmente a oferta do IP `192.168.1.105` feita pelo Servidor DHCP `192.168.1.1`"* (permitindo que eventuais outros servidores DHCP liberem os IPs que haviam ofertado).
4. **A — DHCP Acknowledge / `ACK` (Servidor $\rightarrow$ Cliente)**:
   - O servidor escolhido confirma a alocação enviando o **DHCP ACK** final com todos os parâmetros de configuração (as chamadas **DHCP Options**):
     - **Endereço IP e Máscara de Sub-rede** (*Option 1*, ex.: `255.255.255.0`);
     - **Gateway Padrão / Roteador** (*Option 3*, ex.: `192.168.1.1`);
     - **Servidores DNS** (*Option 6*, ex.: `192.168.1.2` e `1.1.1.1`);
     - **Nome de Domínio Local** (*Option 15*, ex.: `corp.exemplo.test`);
     - **Tempo de Concessão (*Lease Time*, *Option 51*)**: por quanto tempo o cliente pode usar aquele IP (ex.: 8 horas ou 24 horas; ao atingir **50%** desse tempo — timer **T1** —, o cliente envia um *DHCP Request* em *unicast* direto ao servidor para renovar o aluguel).

```text
  CLIENTE DHCP (Porta UDP 68)                        SERVIDOR DHCP (Porta UDP 67)
  IP Inicial: 0.0.0.0                                IP: 192.168.1.1
        |                                                  |
        |  1. [D] DHCP DISCOVER (Src: 0.0.0.0 -> Dst: 255.255.255.255)
        |=================================================>| (Broadcast)
        |                                                  |
        |  2. [O] DHCP OFFER    (Oferece IP 192.168.1.105) |
        |<-------------------------------------------------|
        |                                                  |
        |  3. [R] DHCP REQUEST  (Aceita IP 192.168.1.105)  |
        |=================================================>| (Broadcast)
        |                                                  |
        |  4. [A] DHCP ACK      (Confirma IP, Mask, GW, DNS, Lease=24h)
        |<-------------------------------------------------|
  Configura IP: 192.168.1.105/24
```

##### Problemas de Segurança no DHCP e Configuração Segura
1. **DHCP Starvation (Inanição de DHCP)**: Um atacante na rede local dispara milhares de pedidos *DHCP Discover* falsificando um endereço MAC diferente em cada pedido (usando ferramentas de laboratório como *Gobbler* ou *Yersinia*). Em poucos segundos, todo o *pool* de endereços IP do servidor DHCP legítimo se esgota, impedindo que novos usuários reais se conectem à rede (DoS).
2. **Rogue DHCP Server (Servidor DHCP Falso / Pirata)**: Frequentemente combinado com o *DHCP Starvation*, o atacante sobe um servidor DHCP malicioso na própria máquina. Quando um usuário legítimo envia um *DHCP Discover*, o servidor pirata responde mais rápido com um *DHCP Offer* que entrega ao usuário o **IP da máquina do atacante como Gateway Padrão e como Servidor DNS**! A partir desse instante, o atacante controla tanto o roteamento quanto a resolução de nomes da vítima.
3. **Defesa — DHCP Snooping (`[REQUISITO]` em redes corporativas)**:
   - Recurso de segurança configurado nos switches Layer 2 que divide as portas físicas do switch em duas categorias:
     - **Portas Confiáveis (*Trusted Ports*)**: Apenas os *uplinks* onde o servidor DHCP oficial está conectado. Só elas têm permissão para enviar mensagens de servidor (*DHCP Offer*, *DHCP ACK*, *DHCP NAK*).
     - **Portas Não Confiáveis (*Untrusted Ports*)**: Todas as portas de mesas de usuários e pontos de acesso. Se um computador conectado a uma porta *Untrusted* tentar responder com um *DHCP Offer* ou *DHCP ACK*, o switch **bloqueia o pacote imediatamente e desativa a porta**, neutralizando por completo o ataque de *Rogue DHCP Server*, além de aplicar *rate limit* de pedidos DHCP por porta para impedir *DHCP Starvation*.

---

<a id="s4-5-3"></a>
#### 4.5.3 DNS (Domain Name System) na infraestrutura de rede

Enquanto na [Seção 1.3.1](./01a-fundamentos-web-parte1.md#s1) estudamos os tipos de registros DNS (`A`, `AAAA`, `CNAME`, `MX`, `TXT`) sob a ótica da aplicação web, aqui examinamos como o **DNS** (RFC 1034 / RFC 1035) funciona como infraestrutura crítica de rede.

##### Hierarquia Distribuída e Consulta Recursiva vs. Iterativa
Nenhum servidor DNS único no mundo guarda a lista de todos os domínios da Internet. O DNS é um banco de dados distribuído em árvore invertida:

1. **Consulta Recursiva (Cliente / *Stub Resolver* $\rightarrow$ Resolvedor Recursivo)**:
   - Quando o seu sistema operacional pergunta ao servidor DNS configurado na rede (ex.: o DNS da empresa ou um resolvedor público como `1.1.1.1` / `8.8.8.8`) qual é o IP de `api.loja-ficticia.test`, ele envia uma consulta com a flag **`RD=1`** (*Recursion Desired* — Recursão Desejada). O cliente diz: *"Me entregue a resposta final pronta; faça o trabalho de procurar por mim"*.
2. **Consultas Iterativas (Resolvedor Recursivo $\rightarrow$ Hierarquia Mundial)**:
   - Se a resposta não estiver no cache do Resolvedor Recursivo, ele assume a busca fazendo consultas **iterativas** (onde cada servidor responde com a melhor referência que conhece):
     - **Passo A (Root Servers `.`)**: O resolvedor pergunta a um dos 13 grupos de Servidores Raiz (`a.root-servers.net` a `m.root-servers.net`, distribuídos via Anycast). O servidor Raiz responde: *"Não sei o IP final, mas aqui estão os servidores responsáveis pelo TLD `.test`"*.
     - **Passo B (TLD Servers — *Top-Level Domain*)**: O resolvedor pergunta ao servidor TLD (ex.: `.com`, `.org`, `.br`, `.test`). O TLD responde: *"Não sei o subdomínio `api`, mas os servidores autoritativos do domínio `loja-ficticia.test` são `ns1.loja-ficticia.test`"* (registros `NS`).
     - **Passo C (Authoritative Name Servers)**: O resolvedor pergunta finalmente ao **Servidor Autoritativo** da empresa, que detém a zona oficial e devolve a resposta definitiva com a flag `AA=1` (*Authoritative Answer*): `api.loja-ficticia.test IN A 203.0.113.50` (TTL = 300s).
   - O Resolvedor Recursivo armazena a resposta em seu cache durante os 300 segundos do **TTL** e a devolve ao cliente.

##### Por que o DNS utiliza TANTO a porta 53 UDP QUANTO a porta 53 TCP?

> [!IMPORTANT]
> **Questão recorrente em provas de Redes e Concursos:**
> É um erro comum afirmar que "o DNS usa apenas UDP". Na realidade, o padrão oficial do DNS exige suporte **tanto a UDP quanto a TCP na porta 53**:
> 1. **DNS sobre UDP (Porta 53) — Consultas Padrão**: Usado na imensa maioria das consultas normais do dia a dia porque resolve um nome em apenas **1 RTT** (1 pacote de pergunta + 1 pacote de resposta), sem gastar tempo com o *Three-Way Handshake* do TCP. No padrão original (RFC 1035), pacotes DNS sobre UDP eram limitados a **512 bytes**; com a extensão **EDNS0** (RFC 6891), cliente e servidor negociam payloads UDP maiores (tipicamente **1232 a 4096 bytes**) para acomodar múltiplos registros e assinaturas **DNSSEC**.
> 2. **DNS sobre TCP (Porta 53) — Respostas Grandes e Transferência de Zona**: O DNS opera obrigatoriamente sobre **TCP na porta 53** em duas situações:
>    - **Truncamento (`TC=1`)**: Quando a resposta DNS excede o tamanho máximo suportado no UDP (ou quando um servidor DNS sob ataque ativa a flag `TC=1` — *Truncated* — para confirmar que o cliente não está falsificando o IP de origem), o cliente recebe a resposta UDP truncada e **repete automaticamente a consulta via TCP na porta 53**.
>    - **Transferência de Zona (`AXFR` / `IXFR`)**: Quando um servidor DNS Autoritativo Secundário (*Slave*) precisa copiar toda a base de registros de um domínio a partir do Servidor Primário (*Master*), essa sincronização em massa é feita exclusivamente via **TCP na porta 53**, pois exige confiabilidade absoluta e transfere grande volume de dados.

##### Segurança de Infraestrutura DNS
- **Bloqueio de Transferência de Zona não autorizada (`AXFR`) (`[REQUISITO]`)**: Se um servidor DNS autoritativo permitir que qualquer IP da Internet execute uma consulta `dig @ns1.loja-ficticia.test loja-ficticia.test AXFR` na porta **53/TCP**, um atacante baixa instantaneamente a lista completa de todos os servidores internos, subdomínios de homologação e endereços IP da organização! A transferência de zona deve ser restrita em firewall e na ACL do servidor DNS (`allow-transfer`) estritamente aos IPs dos servidores DNS secundários legítimos.
- **Split-Horizon DNS (ou *Split-Brain DNS*) (`[BOA PRÁTICA]`)**: Arquitetura na qual a organização mantém duas visões de zona DNS separadas: servidores DNS internos (acessíveis apenas na LAN/VPN) que resolvem nomes privados como `gitlab.interno.empresa.test -> 10.10.5.20`, e servidores DNS autoritativos externos que contêm apenas os registros públicos necessários na Internet (`www`, `api`, `MX`), evitando expor a nomenclatura interna da rede ao mundo externo.
- **DNSSEC, DoT e DoH (`[RECOMENDAÇÃO]`)**:
  - **DNSSEC** (*DNS Security Extensions* — RFC 4033/4034/4035) adiciona **assinaturas digitais criptográficas** (`RRSIG`, `DNSKEY`, `DS`) aos registros DNS, garantindo **autenticidade e integridade** (impede *DNS Cache Poisoning*), mas **não cifra** a consulta.
  - **DoT** (*DNS over TLS* — RFC 7858, porta **TCP 853**) e **DoH** (*DNS over HTTPS* — RFC 8484, porta **TCP 443**) adicionam **confidencialidade (criptografia em trânsito)** entre o cliente e o resolvedor recursivo, impedindo que bisbilhoteiros na rede local (como em um Wi-Fi público) vejam quais domínios o usuário está acessando.

---

<a id="s4-5-4"></a>
#### 4.5.4 NAT (Network Address Translation)

Como vimos na [Seção 4.4.2](#s4-4-2), endereços IP privados (`192.168.x.x`, `10.x.x.x`, `172.16–31.x.x`) não podem circular na Internet pública. Como, então, 50 computadores de um escritório com IPs `192.168.1.10` a `192.168.1.60` conseguem acessar a Web simultaneamente quando o provedor entregou apenas **um único endereço IP público** (`198.51.100.7`) no roteador de borda? A resposta é o **NAT** (*Network Address Translation* — Tradução de Endereços de Rede, RFC 2663 / RFC 3022).

##### Modalidades de NAT: SNAT, DNAT e PAT (NAPT)

1. **SNAT (*Source NAT* — NAT de Origem)**:
   - Modifica o **endereço IP de origem** dos pacotes que saem da rede interna privada em direção à Internet pública. Quando o servidor web responde para o IP público, o roteador desfaz a tradução e entrega ao IP privado interno.
2. **PAT (*Port Address Translation*) / NAPT (*Network Address Port Translation*) / *NAT Overload***:
   - É a forma de SNAT usada em 99,9% das residências e empresas. Para que dezenas de máquinas internas compartilhem **um único IP público** simultaneamente, o roteador reescreve tanto o **IP de Origem** quanto a **Porta de Origem (Camada 4)**, mantendo uma **Tabela de Estado de Tradução NAT (*Connection Tracking Table*)**.
3. **DNAT (*Destination NAT* — NAT de Destino / *Port Forwarding*)**:
   - Modifica o **endereço IP de destino** (e opcionalmente a porta de destino) de pacotes que chegam da Internet pública para dentro da rede privada. É utilizado quando queremos publicar um servidor web interno (`10.0.1.50:8080`) atrás do IP público do firewall/roteador (`198.51.100.7:443`).

##### Como funciona a Tabela de Tradução PAT no Roteador (Passo a Passo)

```text
================================================================================
         FUNCIONAMENTO DO PAT / NAPT (MUITOS IPs PRIVADOS -> 1 IP PÚBLICO)
================================================================================

  REDE INTERNA (LAN Privada)                ROTEADOR NAT                INTERNET PÚBLICA
                                      (LAN: 192.168.1.1)
                                      (WAN: 198.51.100.7)
  Host A: 192.168.1.10                        |
  Envia: Src 192.168.1.10:50001 ------------->| Reescreve Src para      Servidor Web
         Dst 203.0.113.50:443                 | 198.51.100.7:61001 ---> 203.0.113.50:443
                                              |
  Host B: 192.168.1.20                        |
  Envia: Src 192.168.1.20:50001 ------------->| Reescreve Src para      Servidor Web
         Dst 203.0.113.50:443                 | 198.51.100.7:61002 ---> 203.0.113.50:443

  +-----------------------------------------------------------------------------------+
  |                    TABELA DE TRADUÇÃO NAT NO ROTEADOR                             |
  +------+-----------------------+-----------------------+----------------------------+
  | Prot | Interno (LAN Privada) | Externo (WAN Pública) | Destino Remoto             |
  +------+-----------------------+-----------------------+----------------------------+
  | TCP  | 192.168.1.10 : 50001  | 198.51.100.7 : 61001  | 203.0.113.50 : 443         |
  | TCP  | 192.168.1.20 : 50001  | 198.51.100.7 : 61002  | 203.0.113.50 : 443         |
  +------+-----------------------+-----------------------+----------------------------+
```

Observe no diagrama acima: mesmo que o Host A e o Host B escolham coincidentemente a mesma porta efêmera interna (`50001`) para acessar o mesmo servidor web (`203.0.113.50:443`), o roteador NAT atribui portas externas distintas (`61001` e `61002`) no IP público `198.51.100.7`. Quando o servidor `203.0.113.50:443` devolve um pacote destinado a `198.51.100.7:61002`, o roteador consulta sua tabela, vê que a porta `61002` pertence ao Host B, troca o destino do pacote para `192.168.1.20:50001`, recalcula os *checksums* IP/TCP e o encaminha para a LAN!

> [!CAUTION]
> **NAT NÃO substitui um Firewall!**
> Embora o SNAT/PAT crie um efeito colateral benéfico — pois um pacote vindo da Internet que não corresponda a uma sessão previamente aberta na tabela de tradução é descartado por não ter para quem ser traduzido —, o NAT é um mecanismo de tradução de endereços, e não um controle de segurança. No IPv6, onde cada máquina pode ter um endereço público global (`2000::/3`) sem NAT, um **Firewall Stateful** explícito na borda é indispensável (**[REQUISITO]**) para bloquear conexões de entrada não solicitadas.

---

<a id="s4-5-5"></a>
#### 4.5.5 ICMP (Internet Control Message Protocol)

O protocolo IP (Camada 3) não possui um mecanismo interno para avisar o remetente quando algo dá errado no caminho (como um roteador sem rota para o destino, um link com MTU menor que o pacote ou um TTL que expirou). Para suprir essa necessidade, foi criado o **ICMP** (*Internet Control Message Protocol* — RFC 792 para IPv4, e **ICMPv6** na RFC 4443 para IPv6).

- **Em qual camada o ICMP opera?** Embora seus dados viajem encapsulados dentro de um pacote IP (`Protocol = 1` no IPv4, `Next Header = 58` no IPv6), o ICMP é considerado parte integrante da **Camada 3 (Rede / Internet)**.
- **Atenção para provas**: O ICMP **NÃO utiliza portas TCP nem UDP**! Perguntas de prova que mencionam *"qual é a porta do comando ping / ICMP"* são pegadinhas clássicas: o ICMP não tem número de porta; ele utiliza campos próprios de **Tipo (*Type*)** e **Código (*Code*)** no seu cabeçalho de 8 bytes.

##### Principais Tipos e Códigos ICMP e Ferramentas de Diagnóstico (`ping` e `traceroute`)

| Tipo ICMPv4 | Código | Nome da Mensagem | Uso Prático e Ferramenta Associada |
| :--- | :--- | :--- | :--- |
| **Tipo `8`** | `0` | **Echo Request** | Pedido de teste de conectividade enviado pelo comando **`ping`**. |
| **Tipo `0`** | `0` | **Echo Reply** | Resposta enviada pelo host de destino confirmando que está ativo e acessível (**`ping`**). |
| **Tipo `3`** | `0` a `15` | **Destination Unreachable** | Informa que o pacote não pôde ser entregue:<ul><li>**Código `0`**: *Network Unreachable* (Roteador não tem rota para a rede).</li><li>**Código `1`**: *Host Unreachable* (O roteador final não conseguiu achar o host via ARP).</li><li>**Código `3`**: *Port Unreachable* (O host recebeu um datagrama **UDP** em uma porta fechada).</li><li>**Código `4`**: *Fragmentation Needed and Don't Fragment (DF) was set* (Essencial para **PMTUD** — *Path MTU Discovery*!).</li><li>**Código `13`**: *Communication Administratively Prohibited* (Bloqueado por filtro/firewall).</li></ul> |
| **Tipo `5`** | `0` a `3` | **Redirect** | Usado por um roteador para avisar a um host local que existe outro roteador mais direto na mesma sub-rede. |
| **Tipo `11`** | `0` | **Time Exceeded (TTL expired in transit)** | Enviado por um roteador intermediário quando o campo **TTL** do pacote IP chega a `0`. É a base de funcionamento do comando **`traceroute`** (Linux/macOS) / **`tracert`** (Windows)! |

**Como o `traceroute` / `tracert` descobre todos os roteadores do caminho usando ICMP?**
1. O `traceroute` envia um primeiro pacote para o destino final, mas força propositalmente o cabeçalho IP a sair com **`TTL = 1`**.
2. O **1º roteador** (Gateway Padrão) recebe o pacote, subtrai 1 do TTL (`1 - 1 = 0`), descarta o pacote e devolve uma mensagem **ICMP Tipo 11 (*Time Exceeded*)** com o seu próprio IP de origem! Assim, o `traceroute` descobre o IP e a latência do Salto 1.
3. Em seguida, o `traceroute` envia um pacote com **`TTL = 2`**, que passa pelo 1º roteador (`TTL=1`) e morre no **2º roteador** (`TTL=0`), gerando um ICMP Tipo 11 do Salto 2. O processo se repete (`TTL = 3, 4, 5...`) até alcançar o servidor final!

##### Segurança e Filtragem Consciente de ICMP
- **Ataques baseados em ICMP**:
  - **ICMP Flood / Ping Flood**: Inundar o alvo com pacotes *Echo Request* para saturar a largura de banda.
  - **Smurf Attack**: Enviar *ICMP Echo Request* com o **IP de origem falsificado** (colocando o IP da vítima como origem) para o endereço de **Broadcast** de uma grande rede (`x.x.x.255`), fazendo centenas de máquinas responderem *Echo Reply* simultaneamente contra a vítima (amplificação).
  - **ICMP Redirect Abuse**: Enviar mensagens *ICMP Tipo 5* falsas na rede local para convencer um host a mudar sua rota para um gateway malicioso.
  - **ICMP Tunneling (*Ping Tunnel*)**: Esconder comandos de *Command & Control (C2)* ou exfiltrar dados confidenciais dentro do campo de *payload* de pacotes *Echo Request/Reply* quando o firewall não inspeciona o conteúdo do ICMP.
- **Configuração Segura (Por que NÃO se deve bloquear 100% do ICMP cegamente)**:
  - **[REQUISITO]** Desativar aceitação e envio de **ICMP Redirects (Tipo 5)** em todos os servidores e estações (`net.ipv4.conf.all.accept_redirects = 0`) e bloquear *Directed Broadcasts* nos roteadores (`no ip directed-broadcast`, RFC 2644).
  - **[RECOMENDAÇÃO]** Aplicar *rate limiting* em **ICMP Echo Request (Tipo 8)** no perímetro externo ou permiti-lo apenas a partir de redes de monitoramento/gerência.
  - **[REQUISITO]** **Permitir sempre ICMP Tipo 3 Código 4 (*Fragmentation Needed*) no IPv4 e pacotes essenciais do ICMPv6 (como *Packet Too Big*, *Router Solicitation/Advertisement* e *Neighbor Solicitation/Advertisement*)!** Se um administrador bloquear todo o protocolo ICMP indiscriminadamente no firewall, a descoberta de MTU do caminho (**PMTUD**) para de funcionar (conexões HTTPS/TLS que enviam certificados grandes travam silenciosamente no meio do handshake — o famoso *PMTUD Black Hole*) e, no **IPv6**, a rede inteira para de operar, pois o IPv6 substituiu o ARP pelo **NDP sobre ICMPv6**.

---

<a id="s4-6"></a>
### 4.6 Segurança de Perímetro e Criptografia em Trânsito

Compreendidos os protocolos que movem pacotes pela rede, passamos aos mecanismos arquiteturais que **filtram, intermediam, isolam e criptografam** esse tráfego: **Firewalls**, **Proxies**, **VPNs** e o protocolo **TLS/HTTPS**.

<a id="s4-6-1"></a>
#### 4.6.1 Firewall

Um **Firewall** é um dispositivo de segurança de rede (físico ou virtual/software, como `nftables`/`iptables` no Linux, *Windows Filtering Platform* ou *Security Groups* em nuvem) posicionado na fronteira entre duas ou mais redes com níveis de confiança distintos (ex.: entre a Internet pública não confiável e a rede interna corporativa), cuja missão é **inspecionar o tráfego que o atravessa e permitir (`ALLOW`/`ACCEPT`) ou bloquear (`DENY`/`DROP`/`REJECT`) pacotes** com base em um conjunto estrito de regras de segurança.

##### Evolução e Categorias de Firewalls (Stateless, Stateful, NGFW e WAF)

| Tipo de Firewall | Camada OSI | Como Funciona | Limitações Principais |
| :--- | :--- | :--- | :--- |
| **1. Stateless (*Packet Filtering* / Sem Estado)** | Camadas 3 e 4 | Avalia cada pacote **individualmente e de forma isolada**, olhando apenas cabeçalhos IP/TCP/UDP (IP origem/destino, porta origem/destino, flags TCP) contra uma lista estática (**ACL** — *Access Control List*). Não guarda memória de pacotes anteriores. | Fácil de enganar com pacotes forjados (ex.: enviar um pacote com flag `ACK` ativa sem ter aberto conexão antes) e exige abrir faixas inteiras de portas altas (`1024–65535`) para permitir o retorno de respostas. |
| **2. Stateful (*Stateful Inspection* / Com Estado)** | Camadas 3 e 4 | Mantém uma **Tabela de Estado de Conexões (*State Table*)** em memória (`NEW`, `ESTABLISHED`, `RELATED`, `INVALID`). Só permite que pacotes de resposta entrem se corresponderem exatamente a uma sessão legítima previamente iniciada e registrada na tabela de estado. | Cego para o conteúdo da Camada 7: se a porta `443` estiver liberada, ele deixa passar um ataque de *SQL Injection* ou *XSS* porque, no nível TCP/IP, a conexão é perfeitamente válida! |
| **3. NGFW (*Next-Generation Firewall*)** | Camadas 3 a 7 | Evolução do *Stateful Firewall* que adiciona **DPI (*Deep Packet Inspection*)**, identificação de aplicações independente da porta (ex.: detectar que alguém está rodando BitTorrent ou SSH escondido na porta 80), **IPS (*Intrusion Prevention System*)** integrado, inspeção TLS e políticas baseadas na identidade do usuário (Active Directory/SSO). | Não substitui um WAF especializado na análise profunda de lógica de negócios, schemas de APIs JSON/GraphQL e regras customizadas de aplicações web. |
| **4. WAF (*Web Application Firewall*)** | **Camada 7** (HTTP / HTTPS) | Especializado exclusivamente no protocolo HTTP/HTTPS. Opera como um proxy reverso na frente dos servidores web, analisando URLs, *Query Strings*, *Headers*, *Cookies* e o corpo (*Payload* JSON/Form/Multipart) para bloquear ataques do **OWASP Top 10** (SQLi, XSS, Command Injection, Path Traversal), *bots* automatizados e DDoS L7. | Protege apenas tráfego HTTP/HTTPS; não protege portas de infraestrutura de rede (como SSH, RDP ou bancos de dados). |

> [!TIP]
> **Diferença entre `DROP` e `REJECT` em regras de Firewall:**
> - **`DROP` (Descarte silencioso)**: O firewall joga o pacote fora sem enviar nenhuma resposta ao remetente. O atacante precisa esperar o *timeout* expirar para perceber que não houve resposta (dificulta e desacelera varreduras de portas externas).
> - **`REJECT` (Rejeição explícita)**: O firewall descarta o pacote e envia imediatamente um pacote de erro de volta (`TCP RST` ou `ICMP Destination Unreachable`). É útil dentro de redes internas para que aplicações falhem rápido (*fail-fast*) em vez de ficarem travadas esperando *timeout*.

##### Política *Default Deny* e Arquitetura de DMZ (*Demilitarized Zone*)

- **Política *Default Deny* (Negar por Padrão / *Implicit Deny*) (`[REQUISITO]`)**: A regra fundamental de qualquer firewall seguro é **"Tudo o que não for explicitamente permitido é proibido"**. O firewall processa as regras de cima para baixo (*top-down*, *first-match*): liberam-se estritamente as portas e origens necessárias (ex.: liberar entrada TCP `443` da Internet para o Load Balancer) e a última regra da tabela bloqueia absolutamente todo o resto (`DENY ALL`).
- **DMZ (*Demilitarized Zone* — Zona Desmilitarizada / Rede de Perímetro) (`[RECOMENDAÇÃO]`)**:
  - É uma sub-rede física ou lógica isolada que fica **no meio do caminho** entre a Internet pública e a Rede Interna (LAN / Rede Privada de Bancos de Dados).
  - **Por que a DMZ existe?** Servidores que precisam receber conexões diretas de desconhecidos da Internet (como **Proxies Reversos**, **Servidores Web** e **Gateways de E-mail**) são os alvos mais expostos a invasões. Se esses servidores ficassem na mesma rede interna dos **Bancos de Dados** e dos computadores dos funcionários, a invasão de um servidor web daria ao atacante acesso imediato ao coração da empresa.
  - Na arquitetura com **DMZ**, os servidores públicos ficam confinados na DMZ: a Internet acessa apenas a DMZ (na porta 443); a DMZ só consegue conversar com a Rede Interna na porta estrita do serviço necessário (ex.: o servidor de aplicação aciona o banco de dados na porta 5432); e nenhuma conexão direta da Internet para a Rede Interna é permitida!

```text
================================================================================
              ARQUITETURA DE PERÍMETRO COM DMZ (FIREWALL DUPLO)
================================================================================

  [ INTERNET PÚBLICA ]
           |
           | Permite APENAS: TCP 443 (HTTPS) -> IP do Proxy Reverso na DMZ
           v
  +------------------+
  | FIREWALL EXTERNO |  (Bloqueia qualquer acesso direto da Internet à LAN)
  +------------------+
           |
           v
  +-------------------------------------------------------+
  |                DMZ (Zona Desmilitarizada)             |
  |  [ WAF / Proxy Reverso Nginx ]  (10.10.1.10:443)      |
  +-------------------------------------------------------+
           |
           | Permite APENAS: DMZ (10.10.1.10) -> App/DB em portas específicas
           v
  +------------------+
  | FIREWALL INTERNO |  (Impede que a DMZ varra o restante da rede interna)
  +------------------+
           |
           v
  +-------------------------------------------------------+
  |             REDE INTERNA PRIVADA (LAN / DATA)         |
  |  [ Servidores de Aplicação ] ---> [ Banco de Dados ]  |
  |        (10.20.1.50:8080)            (10.30.1.99:5432) |
  +-------------------------------------------------------+
```

---

<a id="s4-6-2"></a>
#### 4.6.2 Proxy (Forward Proxy vs Reverse Proxy)

Em redes de computadores, um **Proxy** ("procurador" ou intermediário) é um servidor que atua como ponte nas requisições entre clientes e servidores, encerrando a conexão TCP de um lado e abrindo uma nova conexão TCP do outro. Na prova e na prática profissional, é essencial distinguir os dois tipos opostos de proxy pela pergunta: **"De qual lado da rede o proxy está e quem ele protege?"**

##### 1. Forward Proxy (Proxy Direto ou Proxy de Saída)
- **Quem ele representa**: Fica do lado do **Cliente** (dentro da rede interna corporativa ou configurado no navegador) e age em nome dos **usuários internos** para acessar servidores na Internet.
- **Para que serve**:
  1. **Controle de acesso e filtragem de conteúdo de saída (*Egress Filtering*)**: A empresa bloqueia o acesso direto dos computadores à Internet no firewall e obriga todo o tráfego web a passar pelo *Forward Proxy* (ex.: Squid, Zscaler, Netskope), que verifica se o funcionário tem permissão para acessar determinado domínio, bloqueia sites de *phishing*/malware e aplica políticas de **DLP** (*Data Loss Prevention* — prevenção de vazamento de dados).
  2. **Ocultação/Anonimização do Cliente**: O servidor de destino na Internet vê apenas o endereço IP do *Forward Proxy*, e não o IP individual da estação de trabalho que originou o pedido.
  3. **Cache de Saída**: Armazena localmente arquivos baixados repetidamente pelos usuários da rede (como atualizações de pacotes de sistema).

##### 2. Reverse Proxy (Proxy Reverso ou Proxy de Entrada)
- **Quem ele representa**: Fica do lado do **Servidor** (na borda do datacenter/nuvem ou em uma CDN) e age em nome dos **servidores de back-end** para atender clientes vindos da Internet.
- **Exemplos de mercado**: **Nginx**, **HAProxy**, **Envoy**, **Apache (`mod_proxy`)**, **Traefik**, **Cloudflare**, **AWS Application Load Balancer (ALB)**.
- **Para que serve (Funções essenciais na Arquitetura Web Moderna)**:
  1. **Ocultação e Proteção dos Servidores de Origem**: O cliente na Internet resolve o DNS de `loja-ficticia.test` e obtém apenas o IP público do *Reverse Proxy*. Os servidores reais de aplicação (Node.js, Python, Java, Go) ficam escondidos em sub-redes privadas (`10.x.x.x`), inacessíveis diretamente.
  2. **Balanceamento de Carga (*Load Balancing*)**: Recebe milhões de requisições e as distribui entre dezenas de instâncias de back-end saudáveis (usando algoritmos como *Round Robin*, *Least Connections* ou *IP Hash*) com verificação contínua de saúde (*Health Checks*).
  3. **Terminação TLS (*TLS Offloading / Termination*)**: Realiza o trabalho pesado de criptografia/descriptografia do certificado HTTPS na borda, repassando a requisição limpa (ou re-criptografada internamente) para o back-end.
  4. **Injeção Centralizada de Segurança e Performance**: Adiciona cabeçalhos de segurança (`Strict-Transport-Security`, `Content-Security-Policy`, `X-Content-Type-Options`), aplica **Rate Limiting**, comprime respostas (`gzip`/`br`) e serve ativos estáticos em cache.

```text
================================================================================
                COMPARATIVO: FORWARD PROXY vs. REVERSE PROXY
================================================================================

  [1] FORWARD PROXY (Representa os CLIENTES internos perante a Internet)
  +------------+
  | Cliente 1  |---\      +---------------+                     +--------------+
  +------------+    \     |               |                     | Servidor Web |
                     +--->| FORWARD PROXY | ===(Internet)===>   |   Externo    |
  +------------+    /     | (Rede Cliente)|                     | (Não sabe    |
  | Cliente 2  |---/      +---------------+                     |  quem é C1)  |
  +------------+

  [2] REVERSE PROXY (Representa os SERVIDORES de back-end perante a Internet)
  +------------+                                                +--------------+
  | Navegador  |          +---------------+      /------------->| App Server 1 |
  | do Usuário | =======> | REVERSE PROXY | ----+               +--------------+
  | (Não vê os | (HTTPS)  | (Nginx / CDN) |      \------------->| App Server 2 |
  | AppServers)|          +---------------+                     +--------------+
  +------------+
```

##### Tabela Comparativa: Forward Proxy vs. Reverse Proxy

| Critério | Forward Proxy (Proxy Direto) | Reverse Proxy (Proxy Reverso) |
| :--- | :--- | :--- |
| **Posição na Arquitetura** | Na borda da rede do **Cliente** (saída para a Internet). | Na borda da rede do **Servidor** (entrada vinda da Internet). |
| **Quem ele representa?** | Representa os **clientes** perante os servidores externos. | Representa os **servidores de origem** perante os clientes externos. |
| **Quem fica oculto?** | Oculta a identidade/IP dos **clientes internos** para o servidor web. | Oculta a identidade/IP e a topologia dos **servidores de back-end** para o cliente. |
| **Consciência do Cliente** | O cliente geralmente sabe que está usando um proxy (ou é interceptado na LAN corporativa). | Totalmente **transparente** para o cliente (o navegador acha que o Proxy Reverso é o próprio servidor web). |
| **Objetivos de Segurança** | Filtragem de URLs maliciosas, controle de navegação de funcionários, bloqueio de exfiltração (DLP). | WAF, mitigação de DDoS, *Rate Limiting*, terminação TLS, ocultação do servidor real, injeção de Security Headers. |

---

<a id="s4-6-3"></a>
#### 4.6.3 VPN (Virtual Private Network)

Uma **VPN** (*Virtual Private Network* — Rede Privada Virtual) é uma tecnologia que estende uma rede privada através de uma rede pública não confiável (como a Internet), criando um **túnel lógico criptografado e autenticado** entre dois pontos.

##### Como funciona o Tunelamento (*Tunneling*) e Criptografia
Quando um computador conectado a uma VPN envia um pacote IP destinado a um servidor interno da empresa (`10.20.1.50`), o software cliente da VPN intercepta esse pacote na interface virtual de rede (`tun0` / `wg0`):
1. **Criptografia e Autenticação**: O pacote original inteiro (incluindo o cabeçalho IP privado original e os dados) é **criptografado** e recebe um código de autenticação de integridade (MAC/AEAD).
2. **Encapsulamento (Tunelamento)**: Esse bloco cifrado é colocado como carga útil dentro de um **novo pacote IP externo** (com o IP público do usuário como origem e o IP público do Gateway VPN da empresa como destino).
3. **Desencapsulamento no Destino**: O pacote viaja pela Internet pública (onde provedores e atacantes em redes Wi-Fi veem apenas tráfego cifrado indo para o Gateway VPN). Ao chegar ao Gateway VPN da empresa, ele valida a autenticidade, descriptografa o pacote externo, extrai o pacote IP privado original intacto e o entrega na rede interna `10.20.1.0/24`!

##### Principais Protocolos de VPN

- **IPsec (*Internet Protocol Security* — RFC 4301)**: Padrão clássico da Camada 3, amplamente usado para interligar datacenters e nuvens. Opera em duas fases: primeiro o protocolo **IKEv2** (*Internet Key Exchange v2*, portas **UDP 500** e **UDP 4500** para *NAT Traversal*) autentica as partes e negocia chaves; depois o protocolo **ESP** (*Encapsulating Security Payload*, protocolo IP `50`) cifra e autentica cada pacote. Possui dois modos de operação:
  - **Modo Transporte**: Cifra apenas a carga útil (Camada 4 em diante), mantendo o cabeçalho IP original visível (usado entre dois hosts diretamente).
  - **Modo Túnel**: Cifra o **pacote IP original inteiro** (cabeçalho + dados) e adiciona um novo cabeçalho IP externo (padrão usado em Gateways VPN).
- **WireGuard**: Protocolo de VPN moderno, código enxuto (~4.000 linhas, integrado diretamente ao kernel do Linux e disponível para Windows/macOS/iOS/Android), operando exclusivamente sobre **UDP** com primitivas criptográficas de última geração (*ChaCha20-Poly1305* para criptografia autenticada e *Curve25519* para troca de chaves), oferecendo reconexão instantânea em redes móveis e latência mínima.
- **OpenVPN**: Solução madura de código aberto que utiliza a pilha **TLS (OpenSSL)** para autenticação por certificados e troca de chaves. Pode operar sobre **UDP** (recomendado para evitar o problema de *TCP-over-TCP meltdown*, onde duas camadas TCP tentam retransmitir pacotes perdidos ao mesmo tempo) ou sobre **TCP na porta 443** (útil para atravessar redes de hotéis/aeroportos que bloqueiam portas UDP).

##### Topologias e Modos de Roteamento em VPN
1. **Site-to-Site VPN (Rede a Rede)**: Conecta permanentemente dois roteadores/firewalls de borda (ex.: o Firewall da Matriz em São Paulo ao Virtual Private Gateway da AWS VPC em `sa-east-1`), unindo duas redes locais inteiras de forma transparente para os servidores.
2. **Remote Access VPN (Cliente a Rede)**: Um software instalado no notebook/celular do colaborador em trabalho remoto cria um túnel individual sob demanda (exigindo **MFA** — Autenticação Multifator) até o concentrador VPN da empresa. *(Nota moderna: Muitas organizações vêm substituindo ou complementando VPNs de acesso remoto amplo por arquiteturas **ZTNA — Zero Trust Network Access**, onde o usuário autenticado ganha acesso apenas à aplicação específica autorizada, sem receber um IP com acesso livre à rede interna inteira).*
3. **Full Tunneling vs. Split Tunneling**:
   - **Full Tunneling (Túnel Completo)**: A rota padrão (`0.0.0.0/0`) do notebook do funcionário é apontada para dentro da VPN. **100% do tráfego** (tanto o acesso ao sistema interno `10.x.x.x` quanto o acesso ao YouTube ou Google) vai até a matriz da empresa, passa pelo Firewall/Proxy corporativo e só então sai para a Internet. **[RECOMENDAÇÃO]** Mais seguro (permite inspeção centralizada e impede que a máquina sirva de ponte simultânea entre a Internet doméstica e a LAN corporativa), porém consome mais banda do concentrador VPN.
   - **Split Tunneling (Túnel Dividido)**: Apenas as rotas das sub-redes privadas da empresa (ex.: `10.0.0.0/8`) são enviadas pelo túnel VPN; o tráfego comum de Internet sai diretamente pelo roteador doméstico do usuário. Economiza largura de banda na empresa e melhora a latência em videoconferências, mas exige controles rigorosos de segurança no *endpoint* (**EDR** e firewall local).

---

<a id="s4-6-4"></a>
#### 4.6.4 TLS/SSL, HTTP e HTTPS

Na [Seção 1.3.5 e 1.3.6](./01a-fundamentos-web-parte1.md#s1), vimos os conceitos de HTTP e HTTPS do ponto de vista da Web. Aqui aprofundamos o protocolo criptográfico que torna o HTTPS possível na rede: o **TLS** (*Transport Layer Security* — Segurança da Camada de Transporte).

##### A Relação Exata entre HTTP, TLS e HTTPS
O **HTTPS** (*Hypertext Transfer Protocol Secure*) **não é um protocolo de aplicação separado do HTTP**. Ele é exatamente o protocolo HTTP (HTTP/1.1, HTTP/2 na porta **TCP 443**, ou HTTP/3 sobre QUIC na porta **UDP 443**) trafegando **dentro de um túnel criptográfico TLS**:

$$\text{HTTPS} = \text{HTTP} + \text{TLS} \quad (\text{sobre TCP ou QUIC/UDP})$$

Enquanto no **HTTP puro (porta 80)** os cabeçalhos, cookies de sessão, senhas e o corpo JSON viajam em **texto claro (*plaintext*)** — permitindo que qualquer switch, roteador Wi-Fi, provedor de Internet ou atacante realizando *ARP Spoofing* leia e modifique tudo em tempo real —, o **TLS** fornece três garantias matemáticas invioláveis (**Tríade do TLS**):
1. **Confidencialidade (*Confidentiality*)**: Os dados são cifrados com criptografia simétrica de alta velocidade (**AES-GCM** ou **ChaCha20-Poly1305**). Ninguém no caminho consegue ler URLs completas (apenas o nome do domínio no SNI tradicional), cabeçalhos, cookies ou o corpo da mensagem.
2. **Integridade (*Integrity*)**: Cada registro TLS carrega uma tag de autenticação criptográfica (**AEAD** — *Authenticated Encryption with Associated Data*). Se um atacante no meio do caminho alterar um único bit do pacote em trânsito, a verificação falha imediatamente e a conexão é abortada.
3. **Autenticidade (*Authenticity*)**: Durante o *handshake*, o servidor prova matematicamente sua identidade apresentando um **Certificado Digital X.509** assinado por uma **Autoridade Certificadora (CA)** confiável e assinando os parâmetros da conexão com sua **Chave Privada** assimétrica (**RSA** ou **ECDSA** — *Elliptic Curve Digital Signature Algorithm*).

##### Evolução Histórica: Do SSL 2.0 ao TLS 1.3 (Tabela de Status de Segurança)

Por razões históricas, o termo **"SSL"** (*Secure Sockets Layer*, criado pela Netscape nos anos 1990) ainda aparece em menus (como a fase *"SSL"* na aba *Timing* do Chrome DevTools), mas **todas as versões do SSL estão mortas e proibidas há anos**:

| Protocolo / Versão | Ano / RFC | Status Atual | Motivo Técnico e Riscos |
| :--- | :--- | :--- | :--- |
| **SSL 2.0 / SSL 3.0** | 1995 / 1996 (RFC 6101) | **`[Obsoleto]` — PROIBIDO** (RFC 6176 / RFC 7568) | Falhas estruturais graves no design criptográfico; SSL 3.0 é vulnerável ao ataque **POODLE** (que permite decifrar cookies HTTPS). |
| **TLS 1.0 / TLS 1.1** | 1999 (RFC 2246) / 2006 (RFC 4346) | **`[Obsoleto]` — PROIBIDO** (Deprecados pela **RFC 8996** em 2021) | Uso de algoritmos antigos (MD5/SHA-1, CBC vulnerável a **BEAST** e *Padding Oracle* como **LUCKY13**), ausência de suporte a cifras AEAD modernas. Removidos dos navegadores modernos. |
| **TLS 1.2** | 2008 (**RFC 5246**) | **SEGURO (se bem configurado)** (`[REQUISITO]` restringir cifras) | Ainda amplamente suportado por compatibilidade, mas **exige** desativar cifras fracas (RC4, 3DES, CBC, RSA key exchange estático) e manter apenas cifras **ECDHE + AEAD** (`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`, etc.). |
| **TLS 1.3** | 2018 (**RFC 8446**) | **PADRÃO OURO ATUAL** (`[RECOMENDAÇÃO]` prioritária) | Removeu todas as primitivas legadas inseguras, tornou **Forward Secrecy (ECDHE)** e **AEAD** obrigatórios por design, cifrou os certificados durante o handshake e reduziu a latência de abertura de **2-RTT para apenas 1-RTT**! |

##### Como Funciona o Handshake do TLS 1.3 (1-RTT) Passo a Passo

No antigo TLS 1.2, após o *Three-Way Handshake* do TCP (1 RTT), eram necessárias mais **duas viagens completas de ida e volta (2 RTTs)** para negociar as chaves TLS antes de enviar a requisição HTTP. O **TLS 1.3 (RFC 8446)** revolucionou esse fluxo reduzindo o handshake criptográfico para apenas **1 RTT** (e suportando até *0-RTT Resumption* em conexões repetidas):

1. **Passo 1 — `ClientHello` (Cliente $\rightarrow$ Servidor)**:
   - Logo após enviar o `ACK` final do TCP, o navegador envia a mensagem `ClientHello`.
   - Para economizar uma viagem de rede, o TLS 1.3 já envia dentro do `ClientHello`:
     - As versões e suítes de cifras suportadas (`TLS_AES_128_GCM_SHA256`, `TLS_AES_256_GCM_SHA384`, `TLS_CHACHA20_POLY1305_SHA256`);
     - A extensão **SNI (*Server Name Indication*)**: informa em texto claro (ou cifrado caso use **ECH** — *Encrypted Client Hello*) qual domínio o cliente quer acessar (ex.: `loja-ficticia.test`), permitindo que um mesmo IP/Proxy Reverso sirva certificados diferentes para múltiplos sites;
     - A extensão **ALPN (*Application-Layer Protocol Negotiation*)**: negocia se a conversa dentro do túnel será `h2` (HTTP/2) ou `http/1.1`;
     - A extensão **`key_share`**: o cliente já gera e envia sua **chave pública efêmera Diffie-Hellman (ECDHE)** na primeira mensagem!
2. **Passo 2 — `ServerHello` + Certificado e Finalização Cifrados (Servidor $\rightarrow$ Cliente)**:
   - O servidor recebe a chave pública efêmera do cliente, gera o seu próprio par de chaves efêmeras **ECDHE** e, nesse exato instante, **já consegue calcular o Segredo Compartilhado (*Shared Secret*)** da sessão!
   - O servidor responde com `ServerHello` (contendo a sua chave pública efêmera `key_share`).
   - A partir da linha seguinte do mesmo pacote, **tudo já viaja criptografado**:
     - `{EncryptedExtensions}`: confirma o protocolo ALPN (ex.: `h2`);
     - `{Certificate}`: envia a cadeia de certificados X.509 do servidor;
     - `{CertificateVerify}`: envia uma assinatura digital sobre todo o histórico do handshake usando a **Chave Privada do Certificado**, provando que o servidor realmente é o dono legítimo daquele domínio;
     - `{Finished}`: envia um MAC de verificação de integridade de todo o handshake.
3. **Passo 3 — `{Finished}` + Requisição HTTP Cifrada (Cliente $\rightarrow$ Servidor)**:
   - O navegador calcula o mesmo Segredo Compartilhado usando a `key_share` do servidor, decifra a resposta, valida o Certificado Digital contra a lista de CAs confiáveis do sistema operacional/navegador, verifica a assinatura do `CertificateVerify`, envia seu `{Finished}` e, **no mesmo voo de pacotes**, já dispara a requisição `GET / HTTP/2` totalmente criptografada!

```text
================================================================================
         FLUXO COMPLETO: TCP HANDSHAKE (1 RTT) + TLS 1.3 HANDSHAKE (1 RTT)
================================================================================

  NAVEGADOR (Cliente)                                SERVIDOR HTTPS (Porta 443)
        |                                                  |
        |--- 1. TCP [SYN] -------------------------------->| \
        |<-- 2. TCP [SYN, ACK] ----------------------------|  |-- 1º RTT (TCP: Initial Connection)
        |--- 3. TCP [ACK] -------------------------------->| /
        |                                                  |
        |--- 4. TLS 1.3 ClientHello ---------------------->| \
        |       + SNI (loja-ficticia.test) + ALPN (h2)     |  |
        |       + key_share (Chave Pública Efêmera Cliente)|  |
        |                                                  |  |-- 2º RTT (TLS: "SSL" no DevTools)
        |<-- 5. TLS 1.3 ServerHello + key_share (Servidor) |  |
        |       + {EncryptedExtensions}                    |  |
        |       + {Certificate} (X.509)                    |  |
        |       + {CertificateVerify} (Assinatura Privada) |  |
        |       + {Finished}                               | /
        |                                                  |
        |--- 6. TLS {Finished} + [HTTP/2 GET / Cifrado] -->|  <-- Aplicação já envia dados!
        |<-- 7. [HTTP/2 200 OK Cifrado com AES-GCM] -------|
```

##### Forward Secrecy (PFS — Sigilo Direto Perfeito)
Por que o TLS 1.3 obriga o uso de troca de chaves efêmera (**ECDHE** — *Elliptic Curve Diffie-Hellman Ephemeral*)?
- Em configurações antigas de TLS 1.2 sem *Forward Secrecy* (usando troca de chaves RSA estática), o cliente cifrava o segredo da sessão usando a **Chave Pública do Certificado do Servidor**. Se um adversário gravasse todo o tráfego cifrado da rede hoje e, daqui a 2 anos, invadisse o servidor (ou subornasse alguém) para roubar o arquivo da **Chave Privada (`.key`)** do certificado, ele conseguiria **descriptografar retroativamente todas as sessões gravadas no passado**!
- Com **Forward Secrecy (ECDHE)** (**[REQUISITO]**), a Chave Privada do certificado é usada **apenas para assinar** o handshake no momento da conexão, **nunca para cifrar a chave de sessão**. Para cada nova conexão é gerado um par de chaves matemáticas temporárias (*efêmeras*) em memória RAM que são destruídas assim que a sessão acaba. Mesmo que a chave privada do servidor vaze amanhã, é matematicamente impossível decifrar qualquer tráfego capturado no passado!

---

<a id="s4-7"></a>
### 4.7 Portas de Rede e Tabela Completa de Serviços

Se o **Endereço IP** (Camada 3) equivale ao endereço do prédio de uma grande empresa, a **Porta de Rede** (Camada 4 — Transporte) equivale ao **número da sala ou ramal interno** onde trabalha o departamento específico com o qual você deseja falar. Sem o conceito de portas, um mesmo servidor não conseguiria rodar simultaneamente um servidor web (HTTPS), um servidor de acesso remoto (SSH) e um agente de monitoramento, pois o sistema operacional não saberia para qual processo em memória entregar os pacotes que chegam na placa de rede.

<a id="s4-7-1"></a>
#### 4.7.1 Conceito de Porta, Socket e Faixas da IANA

##### O que é uma Porta Lógica (16 bits)
Nos cabeçalhos **TCP** e **UDP**, os campos *Source Port* (Porta de Origem) e *Destination Port* (Porta de Destino) possuem exatamente **16 bits** cada um (inteiro sem sinal). Isso significa que os números de porta variam matematicamente de **`0` a `65535`** ($2^{16} - 1 = 65535$ portas possíveis para TCP e outras $65535$ portas independentes para UDP).

##### O Conceito de Socket e a 5-Tupla da Conexão
- **Socket**: É a combinação de um **Endereço IP + Número de Porta** (ex.: `203.0.113.50:443`). Representa uma extremidade (*endpoint*) única de comunicação na rede.
- **Identificação Única de uma Conexão (5-Tupla)**: O kernel do sistema operacional identifica cada sessão de rede ativa através da **5-tupla**:
  $$\text{(Protocolo L4, IP de Origem, Porta de Origem, IP de Destino, Porta de Destino)}$$
  Por exemplo: `(TCP, 192.0.2.10, 54321, 203.0.113.50, 443)`. É graças à porta de origem efêmera diferente (`54321`, `54322`, `54323`...) que você consegue abrir 20 abas no navegador para o mesmo site (`203.0.113.50:443`) simultaneamente sem que as respostas se misturem!

##### As 3 Faixas Oficiais de Portas da IANA (RFC 6335)
A **IANA** (*Internet Assigned Numbers Authority*) divide o espaço de `0` a `65535` em três faixas oficiais (muito cobradas em provas!):

| Faixa de Portas | Nome Oficial (IANA / RFC 6335) | Características e Regras no Sistema Operacional |
| :--- | :--- | :--- |
| **`0` a `1023`** | **Portas Bem Conhecidas (*Well-Known Ports* ou *System Ports*)** | Reservadas para os protocolos fundamentais da Internet (HTTP `80`, HTTPS `443`, SSH `22`, DNS `53`, SMTP `25`, etc.). Em sistemas Unix/Linux, são **portas privilegiadas**: exigem privilégios de superusuário (`root` ou capacidade `CAP_NET_BIND_SERVICE`) para que um processo possa escutar (`bind`) nelas. *(A porta `0` é reservada).* |
| **`1024` a `49151`** | **Portas Registradas (*Registered Ports* ou *User Ports*)** | Registradas na IANA para serviços específicos de mercado, bancos de dados e aplicações corporativas (ex.: MySQL `3306`, RDP `3389`, PostgreSQL `5432`, HTTP alternativo `8080`, Redis `6379`). Qualquer usuário sem privilégio de `root` pode abrir um servidor nessas portas. |
| **`49152` a `65535`** | **Portas Dinâmicas, Privadas ou Efêmeras (*Dynamic / Private / Ephemeral Ports*)** | **Não são registradas para servidores fixos**. São sorteadas temporariamente pelo sistema operacional do **Cliente** como **Porta de Origem** toda vez que uma aplicação inicia uma conexão de saída (ex.: seu navegador usa a porta efêmera `52410` para conectar na porta `443` do servidor). Quando a conexão fecha, a porta é devolvida ao sistema. *(Nota: O kernel Linux usa por padrão a faixa `32768–60999` como efêmera, configurável em `/proc/sys/net/ipv4/ip_local_port_range`).* |

---

<a id="s4-7-2"></a>
#### 4.7.2 Tabela Completa de Portas e Serviços de Rede

A tabela abaixo consolida todas as portas essenciais cobradas em provas de Redes, Desenvolvimento Web e Cibersegurança, relacionando o protocolo de transporte, a criptografia nativa, a finalidade e a diretriz imediata de segurança:

| Porta(s) | Protocolo L4 | Serviço / Sigla | Criptografia Nativa? | Finalidade Principal | Postura de Segurança Recomendada |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`20` e `21`** | **TCP** | **FTP** (*File Transfer Protocol*) | **Não** (Texto claro) | Transferência de arquivos: **21** = Controle/Comandos; **20** = Dados (no modo ativo). | **`[Obsoleto]` para uso sem TLS**. Substituir por **SFTP** (porta 22) ou **FTPS** (FTP sobre TLS). |
| **`22`** | **TCP** | **SSH** (*Secure Shell*) / **SFTP** / **SCP** | **Sim** (Cifrado) | Administração remota segura via linha de comando (CLI) e transferência segura de arquivos. | **[REQUISITO]** Desativar login por senha; usar **Chaves SSH (Ed25519/RSA)**, restringir IPs/VPN e usar Fail2ban. |
| **`23`** | **TCP** | **Telnet** (*Teletype Network*) | **Não** (Texto claro) | Administração remota legada em modo texto. | **`[Obsoleto]` — PROIBIDO**. Desativar sempre e substituir integralmente pelo **SSH (porta 22)**. |
| **`25`**<br>*(+`587`/`465`)* | **TCP** | **SMTP** (*Simple Mail Transfer Protocol*) | Opcional via `STARTTLS` (`465` = TLS implícito) | **Envio** e retransmissão (*relay*) de e-mails entre servidores (**25**) e submissão por clientes (**587/465**). | **[REQUISITO]** Proibir *Open Relay*; exigir autenticação + TLS (`587`/`465`) e configurar **SPF, DKIM e DMARC**. |
| **`53`** | **UDP e TCP** | **DNS** (*Domain Name System*) | **Não** (Usar DoT `853` / DoH `443`) | Resolução de nomes em IPs (**UDP** = consultas padrão; **TCP** = transferência de zona `AXFR` e respostas grandes). | **[REQUISITO]** Bloquear `AXFR` para a Internet, desativar recursão aberta em autoritativos e habilitar **DNSSEC**. |
| **`67` e `68`** | **UDP** | **DHCP** (*Dynamic Host Configuration Protocol*) | **Não** (Protocolo LAN) | Concessão automática de IP na rede local (**67** = Servidor DHCP; **68** = Cliente DHCP). | **[REQUISITO]** Ativar **DHCP Snooping** nos switches L2 e bloquear portas 67/68 no roteador de borda. |
| **`80`** | **TCP** | **HTTP** (*Hypertext Transfer Protocol*) | **Não** (Texto claro) | Tráfego Web não criptografado. | **[REQUISITO]** Usar exclusivamente para redirecionar (`301`/`308`) para **HTTPS (porta 443)** + header **HSTS**. |
| **`110`**<br>*(+`995`)* | **TCP** | **POP3** (*Post Office Protocol v3*) | **Não** na `110` (**Sim** na `995` POP3S) | **Recebimento/download** de e-mails da caixa postal para um único dispositivo local. | **[REQUISITO]** Não usar porta `110` em texto claro; exigir **POP3S (`995` TCP)** com TLS. |
| **`143`**<br>*(+`993`)* | **TCP** | **IMAP** (*Internet Message Access Protocol*) | **Não** na `143` (**Sim** na `993` IMAPS) | **Recebimento e sincronização** de e-mails mantendo mensagens e pastas no servidor para múltiplos dispositivos. | **[REQUISITO]** Não usar porta `143` sem TLS; padronizar em **IMAPS (`993` TCP)** + MFA na conta de e-mail. |
| **`443`** | **TCP e UDP** | **HTTPS** (*HTTP over TLS*) | **Sim** (TLS 1.2 / 1.3) | Tráfego Web e APIs REST seguros (**TCP 443** = HTTP/1.1 e HTTP/2; **UDP 443** = HTTP/3 sobre QUIC). | **[REQUISITO]** Exigir **TLS 1.2 / 1.3**, certificados válidos, Forward Secrecy (ECDHE), **HSTS** e **WAF**. |
| **`445`** | **TCP** | **SMB** (*Server Message Block*) | Opcional (SMBv3 suporta cifragem) | Compartilhamento de arquivos, pastas e impressoras em redes Windows / Active Directory. | **[REQUISITO]** **Jamais expor na Internet**; desativar SMBv1 (EternalBlue), exigir *SMB Signing* e Cifragem SMBv3. |
| **`3306`** | **TCP** | **MySQL / MariaDB** | Opcional (Suporta TLS) | Comunicação cliente-servidor com banco de dados relacional MySQL / MariaDB. | **[REQUISITO]** Isolar em sub-rede privada/`127.0.0.1` (nunca expor na WAN), exigir TLS e usuário com menor privilégio. |
| **`3389`** | **TCP e UDP** | **RDP** (*Remote Desktop Protocol*) | **Sim** (Com NLA/TLS) | Área de trabalho remota gráfica em sistemas Windows / Windows Server. | **[REQUISITO]** **Nunca expor diretamente na Internet**; acessar apenas via **VPN / RD Gateway** com **NLA** e **MFA**. |
| **`5432`** | **TCP** | **PostgreSQL** | Opcional (Suporta TLS) | Comunicação cliente-servidor com banco de dados relacional PostgreSQL. | **[REQUISITO]** Isolar na sub-rede de dados, restringir `pg_hba.conf` com `scram-sha-256` e conexões `hostssl`. |
| **`8080`** | **TCP** | **HTTP Alternativo** (*HTTP Proxy / Alt*) | **Não** por padrão | Servidores de aplicação internos (Tomcat, Spring Boot, Node.js), ambientes de dev/homologação e proxies HTTP. | **[RECOMENDAÇÃO]** Manter apenas em loopback/LAN atrás de um **Proxy Reverso (porta 443)**; nunca expor painéis de dev na WAN. |

---

<a id="s4-7-3"></a>
#### 4.7.3 Análise Detalhada Individual de Portas e Serviços

A seguir, analisamos profundamente cada uma das portas listadas, detalhando sua arquitetura de protocolo, funcionamento prático, vetores de ataque clássicos e como protegê-las em ambientes reais e questões de prova.

##### Portas `20` e `21` (TCP) — FTP (File Transfer Protocol)
- **Protocolo de Transporte**: **TCP** (orientado à conexão).
- **Serviço e Finalidade**: O **FTP** (RFC 959) é um protocolo legado para transferência de arquivos entre cliente e servidor. Diferente da maioria dos protocolos modernos, o FTP utiliza **duas conexões TCP separadas** (*out-of-band control*):
  1. **Porta TCP `21` (Canal de Controle / Comandos)**: Permanece aberta durante toda a sessão para enviar comandos de texto (`USER`, `PASS`, `LIST`, `RETR`, `STOR`) e receber códigos de resposta de 3 dígitos (ex.: `220 Service ready`, `230 User logged in`, `530 Not logged in`).
  2. **Porta TCP `20` (Canal de Dados no Modo Ativo)**: Usada exclusivamente para transferir o conteúdo binário/texto dos arquivos ou a listagem de diretórios.
     - *Modo Ativo vs. Modo Passivo (`PASV`)*: No **Modo Ativo**, o cliente abre uma porta alta e o servidor conecta de volta a partir da **porta de origem `20`** (o que frequentemente é bloqueado pelo NAT/Firewall do cliente). No **Modo Passivo (`PASV`)**, o servidor abre uma porta alta efêmera e o cliente inicia tanto a conexão de controle (`21`) quanto a de dados (resolvendo o problema do NAT).
- **Principais Riscos de Segurança**:
  - **Credenciais e arquivos em texto claro (*Cleartext*)**: No FTP tradicional, o comando `USER admin` e `PASS SenhaSecreta123` viajam sem nenhuma criptografia. Qualquer captura de pacotes (Wireshark / `tcpdump`) na rede revela a senha instantaneamente.
  - **Anonymous FTP habilitado indevidamente**: Servidores mal configurados aceitam login com o usuário `anonymous` sem senha, expondo backups, códigos-fonte (`.env`, `wp-config.php`) ou permitindo que atacantes subam *web shells* para o diretório público do servidor web.
  - **FTP Bounce Attack (`PORT` command)**: No modo ativo, um atacante podia usar o comando `PORT` para instruir o servidor FTP a abrir conexões de dados contra portas internas de terceiros, usando o servidor FTP como proxy de varredura de portas.
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** Desativar o FTP em texto claro. Substituir por **SFTP** (*SSH File Transfer Protocol*, que roda sobre o próprio protocolo **SSH na porta TCP 22**) ou, caso um sistema legado exija o protocolo FTP, forçar obrigatoriamente **FTPS** (*FTP over Explicit TLS* — RFC 4217), desabilitar login anônimo e confinar usuários em diretórios isolados (*chroot jail*).

##### Porta `22` (TCP) — SSH (Secure Shell)
- **Protocolo de Transporte**: **TCP** (porta `22`).
- **Serviço e Finalidade**: O **SSH** (atualmente na versão **SSH-2**, RFC 4251 a 4254) é o padrão universal para **administração remota segura via linha de comando (CLI)** de servidores Linux/Unix, equipamentos de rede (roteadores/switches), servidores Windows modernos e tunelamento de portas (*SSH Port Forwarding*), além de servir de base para transferência segura de arquivos (**SCP** e **SFTP**) e operações do **Git** (`git@github.com:...`).
- **Principais Riscos de Segurança**:
  - **Ataques de Força Bruta e *Credential Stuffing***: Qualquer servidor com a porta `22` aberta diretamente para a Internet pública (`0.0.0.0/0`) começa a receber milhares de tentativas automatizadas de login por minuto testando usuários como `root`, `admin`, `ubuntu`, `postgres` com listas de senhas vazadas.
  - **Chaves privadas SSH desprotegidas ou vazadas**: Desenvolvedores que deixam chaves privadas (`id_rsa` / `id_ed25519`) sem *passphrase* em máquinas comprometidas ou as enviam acidentalmente para repositórios Git públicos.
- **Recomendações Práticas de Proteção (`/etc/ssh/sshd_config`)**:
  - **[REQUISITO]** Desativar autenticação por senha (`PasswordAuthentication no`) e exigir exclusivamente **Autenticação por Par de Chaves Assimétricas** (preferencialmente chaves modernas **Ed25519**: `ssh-keygen -t ed25519`, protegidas por senha local).
  - **[REQUISITO]** Desativar o login direto como superusuário (`PermitRootLogin no`), obrigando administradores a entrarem com usuário nominal e elevarem privilégio via `sudo` auditado.
  - **[RECOMENDAÇÃO]** **Não expor a porta 22 para toda a Internet**: restringir o acesso no Firewall apenas aos IPs da **VPN corporativa** ou de um **Bastion Host / Jump Server**, e utilizar ferramentas de bloqueio dinâmico de força bruta como **Fail2ban** ou **CrowdSec**.

```bash
# Exemplo de endurecimento (hardening) essencial em /etc/ssh/sshd_config:
Protocol 2
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
AllowUsers deploy_admin
```

##### Porta `23` (TCP) — Telnet (Teletype Network)
- **Protocolo de Transporte**: **TCP** (porta `23`).
- **Serviço e Finalidade**: Criado em 1969 (RFC 854), o **Telnet** foi o precursor do SSH para acesso remoto a terminais em modo texto.
- **Principais Riscos de Segurança**:
  - **Ausência total de criptografia e autenticação forte (`[Obsoleto]`)**: Cada tecla digitada pelo administrador (incluindo o login `root` e sua senha) é enviada em **texto puro** pela rede, byte a byte. Um ataque de *ARP Spoofing* ou grampo de rede captura a sessão administrativa inteira e permite **Sequestro de Sessão TCP (*TCP Session Hijacking*)** para injetar comandos maliciosos diretamente no terminal aberto.
  - Dispositivos **IoT** (câmeras IP, DVRs, roteadores domésticos antigos) que saem de fábrica com a porta `23` aberta e senhas padrão (`admin:admin`) são o vetor primário de infecção de *botnets* globais de DDoS (como a *Mirai*).
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** **Desinstalar e desativar completamente o serviço Telnet** em todos os servidores, switches e roteadores, bloqueando a porta `23` em todos os firewalls e substituindo 100% do uso administrativo pelo **SSH (porta 22)**. *(Nota: O comando `telnet ip porta` ou `nc -zv ip porta` às vezes é usado por desenvolvedores no terminal apenas como um cliente TCP genérico para testar se uma porta como a `80` ou `5432` está aberta; isso não significa rodar um servidor Telnet na porta 23).*

##### Porta `25` (TCP) — SMTP (Simple Mail Transfer Protocol) e Portas `587` / `465`
- **Protocolo de Transporte**: **TCP**.
- **Serviço e Finalidade**: O **SMTP** (RFC 5321) é o protocolo responsável pelo **envio e roteamento de e-mails** na Internet. Na arquitetura moderna de correio eletrônico, existe uma separação crucial de portas que é **tema clássico de prova**:
  - **Porta TCP `25` (Retransmissão Servidor-para-Servidor / *MTA-to-MTA Relay*)**: Usada entre servidores de e-mail (MTAs — *Mail Transfer Agents*) de domínios diferentes para entregar mensagens (ex.: o servidor do Gmail entrega um e-mail para o servidor listado no registro DNS `MX` de `loja-ficticia.test` na porta `25`). Usa TLS oportunista via comando `STARTTLS`.
  - **Porta TCP `587` (*Mail Submission* com `STARTTLS` — RFC 6409)** e **Porta TCP `465` (*SMTPS / Implicit TLS* — RFC 8314)**: Usadas quando uma **aplicação web (back-end Node.js/Python)** ou um cliente de e-mail (Outlook, Thunderbird) se autentica para **submeter/disparar** um e-mail através do seu servidor de envio (ex.: Amazon SES, SendGrid, Postmark).
- **Principais Riscos de Segurança**:
  - **Open Relay (Retransmissor Aberto)**: Se um servidor SMTP na porta `25` aceitar enviar e-mails de qualquer remetente para qualquer destinatário externo sem autenticação, spammers e golpistas o utilizarão para disparar milhões de e-mails de *phishing*, colocando o IP da empresa em listas de bloqueio globais (*Blocklists / RBLs*).
  - **Email Spoofing (Falsificação de Remetente)**: O protocolo SMTP original não verifica sozinho se quem diz ser `contato@banco-ficticio.test` tem autorização daquele domínio.
  - **SMTP User Enumeration**: Comandos SMTP como `VRFY` e `EXPN` permitem que um atacante descubra quais endereços de e-mail existem no servidor.
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** Desabilitar *Open Relay* (só aceitar entrega sem autenticação para domínios locais da própria empresa), desativar os comandos `VRFY` e `EXPN`, e obrigar autenticação + TLS nas portas **`465`** ou **`587`** para clientes e aplicações.
  - **[REQUISITO]** Publicar no DNS a tríade de autenticação de e-mail contra *Spoofing* e *Phishing*: **SPF** (*Sender Policy Framework* — lista quais IPs podem enviar e-mails pelo domínio), **DKIM** (*DomainKeys Identified Mail* — assina digitalmente o cabeçalho do e-mail com criptografia assimétrica) e **DMARC** (*Domain-based Message Authentication, Reporting, and Conformance* — define a política `p=quarantine` ou `p=reject` para e-mails que falhem no SPF/DKIM).

##### Porta `53` (UDP e TCP) — DNS (Domain Name System)
- **Protocolo de Transporte**: **UDP** (consultas padrão de alta velocidade) e **TCP** (transferências de zona `AXFR`/`IXFR` e respostas que ultrapassam o limite de tamanho do datagrama UDP com flag `TC=1`).
- **Serviço e Finalidade**: Tradução de nomes de domínio legíveis (`api.exemplo.test`) em endereços IP (`203.0.113.10` / `2001:db8::10`) e vice-versa (veja [Seção 1.3.1](./01a-fundamentos-web-parte1.md#s1) e [Seção 4.5.3](#s4-5-3)).
- **Principais Riscos de Segurança**:
  - **Ataques de Amplificação e Reflexão DNS (DDoS)**: Um atacante envia uma pequena consulta UDP (ex.: `60 bytes` pedindo registros `ANY` ou `TXT`/`DNSSEC`) para milhares de servidores DNS recursivos abertos na Internet (*Open Resolvers*), falsificando o **IP de Origem** para ser o IP da vítima (*IP Spoofing*). Os servidores DNS respondem com pacotes de `3.000` a `4.000 bytes` todos direcionados à vítima — um fator de amplificação de mais de **50 vezes**!
  - **Transferência de Zona (`AXFR`) exposta**: Vazamento completo do mapa de hosts internos da organização via porta `53/TCP`.
  - **DNS Cache Poisoning (Envenenamento de Cache)** e **DNS Tunneling** (exfiltração de dados ou canal de comando e controle codificado em subdomínios de consultas DNS, ex.: `dados-secretos-base64.atacante.test`).
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** **Jamais operar um Resolvedor Recursivo Aberto (*Open Resolver*)** acessível para a Internet pública: servidores recursivos devem responder apenas para IPs da rede interna; servidores autoritativos públicos devem ter a recursão estritamente desativada (`recursion no;` no BIND).
  - **[REQUISITO]** Restringir transferências de zona (`TCP 53`) por IP + chaves criptográficas **TSIG** (*Transaction Signature*).
  - **[RECOMENDAÇÃO]** Habilitar **DNSSEC**, configurar **RRL** (*Response Rate Limiting*) em servidores autoritativos contra amplificação, monitorar consultas anômalas (DGA e *DNS Tunneling*) e utilizar **DoT (`853/TCP`)** ou **DoH (`443/TCP`)** nas estações de trabalho.

##### Portas `67` e `68` (UDP) — DHCP (Dynamic Host Configuration Protocol)
- **Protocolo de Transporte**: **UDP** (**`67`** no Servidor DHCP; **`68`** no Cliente DHCP).
- **Serviço e Finalidade**: Configuração automática de endereço IP, máscara, gateway padrão e servidores DNS para dispositivos que entram na rede local através do fluxo **DORA** (veja [Seção 4.5.2](#s4-5-2)).
- **Principais Riscos de Segurança**:
  - **Rogue DHCP Server**: Atacante na LAN entrega seu próprio IP como Gateway Padrão e Servidor DNS para capturar tráfego (*Man-in-the-Middle*).
  - **DHCP Starvation**: Esgotamento intencional de todos os IPs disponíveis no escopo do servidor DHCP legítimo.
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** Ativar **DHCP Snooping** nos switches de acesso Layer 2 (bloqueando respostas oriundas da porta `67/UDP` em portas de usuários *Untrusted*) em conjunto com *Port Security* e bloquear tráfego das portas `67` e `68` na borda externa do firewall.

##### Porta `80` (TCP) — HTTP (Hypertext Transfer Protocol)
- **Protocolo de Transporte**: **TCP** (porta `80`).
- **Serviço e Finalidade**: Porta padrão histórica da World Wide Web para o protocolo **HTTP** sem criptografia (RFC 9110 / RFC 9112). Quando você digita `http://exemplo.test` no navegador sem especificar porta, o navegador conecta automaticamente na porta **TCP `80`**.
- **Principais Riscos de Segurança**:
  - **Transmissão integral em texto claro (*Plaintext*)**: Qualquer requisição ou resposta que trafegue pela porta `80` pode ser lida (**Sniffing**) e modificada em trânsito (**Man-in-the-Middle / Content Injection**) por qualquer nó intermediário (roteadores Wi-Fi públicos, provedores, atacantes via ARP Spoofing).
  - **Roubo de Sessão e Credenciais**: Se um site enviar formulários de login pela porta `80` ou definir cookies de sessão sem o atributo `Secure`, uma única requisição HTTP na porta `80` (mesmo para carregar uma imagem inofensiva) faz o navegador enviar o cookie de sessão em texto claro pela rede.
  - **Ataques de *SSL Stripping***: Antes de o navegador chegar à porta `443`, um atacante na rede local intercepta a primeira tentativa de acesso na porta `80` (`http://loja-ficticia.test`) e impede o redirecionamento para `https://`, mantendo a vítima presa em HTTP texto claro.
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** **Jamais servir conteúdo de aplicação, páginas de login ou endpoints de API na porta 80**. A porta `80` em um servidor web público deve servir exclusivamente a dois propósitos: (1) responder aos desafios automáticos de validação de certificado (`/.well-known/acme-challenge/` do Let's Encrypt); e (2) retornar imediatamente um **Redirecionamento Permanente (`301 Moved Permanently` ou `308 Permanent Redirect`)** para `https://` na porta **`443`**.
  - **[REQUISITO]** Configurar na resposta da porta `443` o cabeçalho **HSTS (`Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`)** e incluir o domínio na *HSTS Preload List* dos navegadores, garantindo que o navegador **nunca mais tente usar a porta 80** nem na primeira conexão!

```nginx
# Configuração segura da porta 80 no Nginx (Apenas Redirecionamento 301 para HTTPS 443):
server {
    listen 80;
    listen [::]:80;
    server_name loja-ficticia.test www.loja-ficticia.test;

    # Redireciona permanentemente qualquer requisição HTTP para HTTPS
    return 301 https://$host$request_uri;
}
```

##### Porta `110` (TCP) — POP3 (Post Office Protocol v3) e Porta `995` (POP3S)
- **Protocolo de Transporte**: **TCP** (porta `110` texto claro / `STARTTLS`; porta **`995`** para **POP3S** — POP3 sobre TLS implícito).
- **Serviço e Finalidade**: O **POP3** (RFC 1939) é um protocolo clássico de **recebimento de e-mails**. Sua filosofia original foi desenhada para a época da Internet discada: o cliente de e-mail se conecta ao servidor na porta `110` (ou `995`), autentica-se (`USER` / `PASS`), **baixa todas as mensagens da caixa de entrada para o disco local do computador e (por padrão) apaga as mensagens do servidor**.
- **Principais Riscos de Segurança**:
  - **Senhas em texto claro na porta `110`**: O login POP3 tradicional trafega em texto puro caso o TLS não seja forçado, expondo a senha de e-mail do usuário na rede local.
  - **Perda de visibilidade central e backup**: Como os e-mails são baixados e removidos do servidor (no comportamento padrão), se o computador do usuário sofrer falha de disco ou infecção por *ransomware*, o histórico de e-mails é perdido, além de não sincronizar o estado de "lido/não lido" ou pastas entre celular e computador.
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** Desativar o POP3 em texto claro na porta `110`; caso o protocolo ainda seja necessário, permitir exclusivamente **POP3S na porta TCP `995`** (TLS 1.2/1.3) com autenticação moderna (OAuth 2.0 / MFA). **[RECOMENDAÇÃO]** Preferir o protocolo **IMAPS (porta 993)** ou Webmail HTTPS para múltiplos dispositivos.

##### Porta `143` (TCP) — IMAP (Internet Message Access Protocol) e Porta `993` (IMAPS)
- **Protocolo de Transporte**: **TCP** (porta `143` texto claro / `STARTTLS`; porta **`993`** para **IMAPS** — IMAP sobre TLS implícito, RFC 8314).
- **Serviço e Finalidade**: O **IMAP** (RFC 3501 / RFC 9051 — IMAP4rev2) é o protocolo moderno padrão para **recebimento, leitura e sincronização de e-mails**.
  - **Diferença fundamental para prova (SMTP vs. POP3 vs. IMAP)**:
    - **SMTP (`25` / `587` / `465`)**: Serve exclusivamente para **ENVIAR** e-mails (da aplicação/cliente para o servidor e entre servidores).
    - **POP3 (`110` / `995`)**: Serve para **RECEBER** e-mails seguindo o modelo *"baixar e apagar do servidor"* (focado em um único dispositivo offline).
    - **IMAP (`143` / `993`)**: Serve para **RECEBER e SINCRONIZAR** e-mails mantendo todas as mensagens, pastas e marcações (*lido, respondido, sinalizado, rascunho*) **armazenadas centralmente no servidor**, permitindo que o usuário acesse a mesma caixa postal simultaneamente pelo smartphone, notebook e tablet em perfeita sincronia!
- **Principais Riscos de Segurança**:
  - Exposição de credenciais se utilizado na porta `143` sem criptografia TLS.
  - **Ataques de *Password Spraying* e *Credential Stuffing***: Como a caixa de e-mail corporativa é a chave para redefinir senhas de todos os outros sistemas (*"Esqueci minha senha"*), servidores IMAP expostos sem MFA são alvos prioritários de invasores para espionagem corporativa (*Business Email Compromise* — BEC).
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** Bloquear conexões não criptografadas na porta `143` e exigir **IMAPS na porta TCP `993`** (TLS implícito).
  - **[REQUISITO]** Desabilitar autenticação legada por senha simples (*Basic Auth*) no servidor de e-mail, exigindo **Autenticação Moderna (OAuth 2.0 / OpenID Connect) com MFA obrigatório** e alertas de login a partir de localizações geográficas atípicas.

##### Porta `443` (TCP e UDP) — HTTPS (HTTP over TLS / QUIC)
- **Protocolo de Transporte**:
  - **TCP (Porta `443`)**: Utilizado pelo **HTTP/1.1** e pelo **HTTP/2** sobre TLS 1.2 / TLS 1.3.
  - **UDP (Porta `443`)**: Utilizado pelo **HTTP/3** (RFC 9114), que roda sobre o protocolo de transporte **QUIC** (RFC 9000) com TLS 1.3 integrado nativamente ao transporte! *(Atenção para provas recentes: hoje a porta `443` é tanto **TCP** quanto **UDP**!).*
- **Serviço e Finalidade**: É a porta mais importante da Internet moderna. Transporta de forma criptografada, íntegra e autenticada todo o tráfego de páginas Web, Single-Page Applications (SPAs), **APIs REST / GraphQL**, **WebSockets seguros (`wss://`)**, **gRPC** e **DNS over HTTPS (DoH)**.
- **Principais Riscos de Segurança**:
  - **Falsa sensação de segurança na Camada 7 ("Tem cadeado, então o site é seguro")**: A porta `443` com TLS protege os dados **em trânsito** (entre o navegador e o servidor contra interceptação na rede), mas **NÃO protege** o servidor contra vulnerabilidades no código da aplicação! Um ataque de **SQL Injection**, **XSS**, **CSRF**, **IDOR/BOLA** ou **Command Injection** (veja [Seção 3](./03a-vulnerabilidades-parte1.md#s3)) viaja perfeitamente criptografado por dentro do túnel da porta `443` até atingir o back-end vulnerável.
  - **Configuração criptográfica fraca (*Cryptographic Failures*)**: Aceitar protocolos obsoletos (TLS 1.0/1.1), cifras sem *Forward Secrecy* ou certificados expirados/autoassinados.
  - **Abuso por malwares (*C2 over HTTPS*)**: Como a porta `443` de saída está liberada em praticamente todos os firewalls corporativos, malwares usam conexões HTTPS na porta `443` para conversar com servidores de *Command & Control* e exfiltrar dados sem levantar suspeitas em firewalls *stateful* simples.
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** Configurar o servidor web / Proxy Reverso para aceitar **apenas TLS 1.2 (com cifras ECDHE + AEAD) e TLS 1.3**, automatizar a renovação de certificados X.509 (ex.: ACME / Let's Encrypt), ativar **OCSP Stapling** e enviar os [Headers de Segurança HTTP](./05-headers-de-seguranca.md#s5) (`Strict-Transport-Security`, `Content-Security-Policy`, `X-Content-Type-Options: nosniff`).
  - **[RECOMENDAÇÃO]** Posicionar um **WAF (Web Application Firewall)** e proteção **Anti-DDoS** na frente da porta `443`, aplicar *Rate Limiting* nos endpoints de autenticação/API e validar rigorosamente todas as entradas no back-end.

##### Porta `445` (TCP) — SMB (Server Message Block)
- **Protocolo de Transporte**: **TCP** (porta `445` — *Direct Host SMB*, que substituiu o antigo SMB sobre NetBIOS nas portas `137–139`). *(Versões recentíssimas do Windows Server também suportam opcionalmente SMB over QUIC na porta UDP 443).*
- **Serviço e Finalidade**: O **SMB** (também conhecido historicamente como **CIFS** — *Common Internet File System*) é o protocolo padrão de **compartilhamento de arquivos, pastas em rede (`\\servidor\financeiro`), impressoras e administração remota (IPC$ / Named Pipes)** em redes Microsoft Windows, Active Directory e servidores Linux rodando **Samba**.
- **Principais Riscos de Segurança**:
  - **Vetor histórico de Worms e Ransomware devastadores (*EternalBlue* / WannaCry / NotPetya)**: Em 2017, a vulnerabilidade crítica **MS17-010 (*EternalBlue*)** no protocolo **SMBv1** permitiu execução remota de código (*Remote Code Execution* — RCE) sem qualquer autenticação apenas enviando pacotes especialmente forjados para a porta `445`, paralisando hospitais, indústrias e governos no mundo inteiro com os ransomwares *WannaCry* e *NotPetya*.
  - **Movimentação Lateral e *SMB Relay* / *Pass-the-Hash***: Dentro de redes corporativas, quando o *SMB Signing* (assinatura digital de pacotes SMB) não é exigido, um atacante na LAN pode interceptar tentativas de autenticação NTLM na porta `445` (usando ferramentas de laboratório como *Responder* e *ntlmrelayx*) e retransmiti-las para assumir o controle de outros servidores na rede.
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** **JAMAIS expor a porta `445` para a Internet pública (`0.0.0.0/0`)**! Ela deve ser bloqueada incondicionalmente nos firewalls de perímetro (tanto na entrada quanto na saída).
  - **[REQUISITO]** **Desativar e remover completamente o protocolo SMBv1** em todos os sistemas Windows e Samba, permitindo apenas **SMBv3** (versões 3.0 / 3.1.1).
  - **[REQUISITO]** Habilitar obrigatoriamente o **SMB Signing** (assinatura de pacotes para impedir ataques de *NTLM Relay*) e o **SMB Encryption** (cifragem de dados AES-GCM/CCM do SMBv3) via GPO no Active Directory, além de segmentar as estações de trabalho para que um computador de usuário não possa conectar na porta `445` de outro computador de usuário (*Workstation-to-Workstation block*).

##### Porta `3306` (TCP) — MySQL e MariaDB
- **Protocolo de Transporte**: **TCP** (porta `3306`; o MySQL 8+ também possui o *X Protocol* na porta `33060`).
- **Serviço e Finalidade**: Porta padrão para conexões diretas cliente-servidor com sistemas gerenciadores de banco de dados relacionais (**SGBDs**) **MySQL** e **MariaDB** (usada pela aplicação back-end em Node.js, PHP, Python, Java ou por clientes de administração como DBeaver e MySQL Workbench).
- **Principais Riscos de Segurança**:
  - **Exposição direta do banco de dados na Internet**: Deixar a porta `3306` aberta para `0.0.0.0/0` em uma VPS ou instância de nuvem permite ataques massivos de força bruta contra o usuário `root`, exploração de vulnerabilidades de pré-autenticação no binário do SGBD e exfiltração direta de todas as tabelas caso uma credencial vaze em um arquivo `.env`.
  - **Abuso de leitura/escrita no sistema de arquivos (`SECURE_FILE_PRIV`)**: Se o usuário do banco usado pela aplicação web tiver o privilégio excessivo `FILE` (ou `SUPER`) e a variável `secure_file_priv` estiver vazia, uma vulnerabilidade de [SQL Injection](./03a-vulnerabilidades-parte1.md#s3-1) na aplicação web permite que o atacante leia arquivos do servidor (`LOAD_FILE('/etc/passwd')`) ou escreva um *web shell* no disco (`INTO OUTFILE '/var/www/html/shell.php'`).
  - **Tráfego de banco não criptografado na rede interna**: Consultas SQL e resultados contendo CPF, e-mails e dados financeiros trafegando em texto claro entre o servidor de aplicação e o servidor de banco.
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** **Bloquear a porta `3306` para a Internet pública!** Se o banco rodar na mesma máquina da aplicação, fazer *bind* apenas em loopback (`bind-address = 127.0.0.1` no `my.cnf`); se rodar em servidor dedicado/RDS, colocá-lo em uma **sub-rede privada isolada (sem IP público)** e liberar no *Security Group*/Firewall a entrada na porta `3306` **exclusivamente a partir do IP/Security Group do Servidor de Aplicação**.
  - **[REQUISITO]** Aplicar o **Princípio do Menor Privilégio**: a aplicação web nunca deve conectar ao banco com o usuário `root`! Crie um usuário específico (ex.: `app_loja_user`@`10.20.1.50`) com permissão restrita apenas a `SELECT, INSERT, UPDATE, DELETE` no banco da aplicação, e exija conexões criptografadas (`REQUIRE SSL` / TLS).

##### Porta `3389` (TCP e UDP) — RDP (Remote Desktop Protocol)
- **Protocolo de Transporte**: **TCP** (conexão principal na porta `3389`) e **UDP** (porta `3389`, usada em versões modernas do RDP para acelerar a transmissão de vídeo, áudio e interatividade da tela).
- **Serviço e Finalidade**: Protocolo proprietário da Microsoft para **Acesso Remoto via Interface Gráfica (Área de Trabalho Remota)** a estações Windows e servidores Windows Server (`mstsc.exe`).
- **Principais Riscos de Segurança**:
  - **Porta número 1 visada por grupos de Ransomware no mundo**: Estatisticamente, a exposição direta da porta `3389` na Internet pública é um dos maiores vetores iniciais de invasão corporativa e sequestro de servidores por *Ransomware*. Atacantes varrem a Internet em segundos (via Shodan/Censys/Masscan) buscando a porta `3389` aberta para executar força bruta de senhas, usar credenciais vazadas em *infostealers* ou explorar vulnerabilidades críticas de execução remota no protocolo (como a falha **BlueKeep — CVE-2019-0708**).
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** **NUNCA expor a porta `3389` diretamente para a Internet pública** (nem mesmo "trocando a porta para `33389`", pois scanners de rede varrem todas as 65.535 portas rapidamente — *Security by Obscurity* não é defesa!).
  - **[REQUISITO]** O acesso administrativo via RDP deve ser feito exclusivamente por dentro de uma **VPN corporativa autenticada com MFA**, por um **RD Gateway ( sobre HTTPS 443 com MFA)** ou solução de **ZTNA / Bastion Host** (como Azure Bastion ou AWS Systems Manager Fleet Manager).
  - **[REQUISITO]** Manter o **NLA (*Network Level Authentication* — Autenticação no Nível de Rede)** obrigatoriamente ativado (exige que o usuário se autentique via CredSSP/TLS **antes** que o servidor aloque recursos gráficos de uma sessão do Windows), ativar políticas de bloqueio de conta (*Account Lockout Policy*) após poucas tentativas falhas e restringir quais usuários pertencem ao grupo *Remote Desktop Users*.

##### Porta `5432` (TCP) — PostgreSQL
- **Protocolo de Transporte**: **TCP** (porta `5432`).
- **Serviço e Finalidade**: Porta padrão para comunicação cliente-servidor com o **PostgreSQL**, um dos sistemas gerenciadores de banco de dados objeto-relacionais de código aberto mais robustos e utilizados no mundo em aplicações web modernas e microsserviços.
- **Principais Riscos de Segurança**:
  - **Exposição na Internet e configuração permissiva no `pg_hba.conf`**: Se `listen_addresses = '*'` estiver ativo no `postgresql.conf` combinado com uma regra `host all all 0.0.0.0/0 trust` (ou `md5` com senha fraca) no arquivo de controle de acesso `pg_hba.conf`, qualquer atacante na Internet pode se conectar ao banco.
  - **Execução de comandos no sistema operacional via `COPY ... FROM/TO PROGRAM`**: No PostgreSQL (versão 9.3+), um superusuário do banco (`postgres`) ou membros da role `pg_execute_server_program` podem executar comandos arbitrários do sistema operacional diretamente por dentro de uma query SQL (`COPY tbl FROM PROGRAM 'id';`). Se a aplicação web conectar ao banco usando o superusuário `postgres` e sofrer um *SQL Injection*, o atacante ganha controle imediato do servidor (*Remote Code Execution*)!
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** Isolar o servidor PostgreSQL em uma **sub-rede privada de dados**, bloqueando a porta `5432` para qualquer origem que não seja a camada de servidores de aplicação.
  - **[REQUISITO]** Configurar o arquivo `pg_hba.conf` para exigir conexões **TLS obrigatórias (`hostssl`)** e o método de hash de autenticação moderno **`scram-sha-256`** (nunca usar `trust` ou `password` em texto claro, e migrar de `md5` para `scram-sha-256`).
  - **[REQUISITO]** A aplicação web jamais deve usar o superusuário `postgres`; deve utilizar uma *role* sem privilégios de superusuário (`NOSUPERUSER NOCREATEDB NOCREaterole`) e sem permissão na role `pg_execute_server_program`.

```text
# Exemplo de configuração segura no arquivo pg_hba.conf do PostgreSQL:
# TYPE    DATABASE        USER            ADDRESS                 METHOD
local     all             postgres                                peer
hostssl   loja_db         app_loja_user   10.20.1.0/24            scram-sha-256
host      all             all             0.0.0.0/0               reject
```

##### Porta `8080` (TCP) — HTTP Alternativo (HTTP Alt / Proxy / Application Server)
- **Protocolo de Transporte**: **TCP** (porta `8080`; frequentemente acompanhada na mesma família por portas de desenvolvimento como `3000`, `5000`, `8000` e `8443` para HTTPS alternativo).
- **Serviço e Finalidade**: Registrada na IANA como **`http-alt`**, a porta `8080` pertence à faixa de **Portas Registradas (`1024–49151`)**. Por estar acima de `1023`, ela **não exige privilégios de `root`** no Linux para ser aberta. Por isso, é amplamente utilizada para:
  1. Rodar servidores de aplicação back-end (como **Apache Tomcat**, **Spring Boot / Java**, **WildFly**, **Node.js**, **Go**, **Gunicorn/Uvicorn**) com um usuário de sistema sem privilégios na rede interna, ficando atrás de um Proxy Reverso (Nginx/ALB) que escuta na porta `443`.
  2. Rodar servidores de desenvolvimento/homologação, ferramentas de CI/CD (como **Jenkins**, que usa `8080` por padrão) ou **Proxies HTTP / Ferramentas de Interceptação de Segurança** (como **OWASP ZAP** e **Burp Suite** em `127.0.0.1:8080`).
- **Principais Riscos de Segurança**:
  - **Bypass de Proxy Reverso / WAF (*Direct-to-Origin Attack*)**: É um erro clássico de configuração em servidores VPS/Cloud configurar o Nginx na porta `443` com HTTPS, WAF e Rate Limiting repassando tráfego para `localhost:8080`, mas **esquecer a porta `8080` aberta para `0.0.0.0/0` no firewall**! O atacante descobre a porta `8080` aberta, acessa `http://203.0.113.50:8080` diretamente em texto claro (sem TLS) e **contorna 100% das proteções do WAF, do Rate Limiting e das restrições de rota do Nginx**!
  - **Exposição de painéis administrativos de infraestrutura**: Consoles de gerenciamento como *Tomcat Manager* (`/manager/html`), *Jenkins*, *Spring Boot Actuator* (`/actuator/env`, `/actuator/heapdump`), *Traefik Dashboard* ou *phpMyAdmin* esquecidos abertos na porta `8080` sem autenticação forte.
- **Recomendações Práticas de Proteção**:
  - **[REQUISITO]** Quando a aplicação rodar na mesma máquina que o Proxy Reverso (Nginx/Apache), configurar o servidor de aplicação para escutar **exclusivamente na interface de Loopback (`127.0.0.1:8080`)** e **nunca em `0.0.0.0:8080`**.
  - **[REQUISITO]** Bloquear a porta `8080` no firewall externo para a Internet pública (`DROP`), permitindo que os usuários externos acessem a aplicação **unicamente pela porta `443` (HTTPS)** através do Proxy Reverso/WAF.
  - **[BOA PRÁTICA]** Desativar endpoints de depuração (como *Spring Boot Actuator* sensíveis, *Debug Mode* do Django/Flask/Express e consoles de gerenciamento do Tomcat) em ambientes de produção ou isolá-los em uma porta administrativa acessível apenas via rede interna/VPN.

---

---

> [⬅️ 3B. Vulnerabilidades Web (3.10 a 3.19)](./03b-vulnerabilidades-parte2.md) | [🏠 **Voltar ao Sumário Principal**](../README.md) | [5. Headers de Segurança HTTP ➡️](./05-headers-de-seguranca.md)
