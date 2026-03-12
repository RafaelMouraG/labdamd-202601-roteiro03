# Bloco de Reflexão - Roteiro 03

## 1. Stubs e Skeletons

O stub (cliente) faz o marshalling, serializa a chamada em bytes e envia pela rede. O skeleton(servidor) faz o inverso, ele recebe, deserializa e despacha para a função correta usando a dispatch table.
Na Tarefa 2, `_stub_chamar` converte a chamada em JSON e envia via socket, enquanto `_skeleton_tratar_conexao` recebe os bytes, consulta o dicionário `registro` e executa a função.
Sem esses componentes, as funções de negócio (`somar`, `obter_info`) teriam que lidar diretamente com sockets e serialização, quebrando a separação de responsabilidades e a transparência.

## 2. REST não é RPC

No RPC (Tarefa 1), o cliente chama `proxy.calcular("soma", 10, 3)`, orientado a ações, onde cada funcionalidade é uma função nova.
No REST (Tarefa 3), o cliente faz `POST /produtos`, orientado a recursos, usando verbos HTTP fixos (`GET`, `POST`, `PUT`, `DELETE`).
Fielding (2000) define o REST com 6 restrições, sendo a mais importante a interface uniforme: `GET /produtos/1` lê, `PUT /produtos/1` atualiza, `DELETE /produtos/1` remove, sempre os mesmos verbos. Outra restrição chave é o stateless: cada requisição carrega toda a informação necessária, sem sessão guardada no servidor, diferente do RPC que pode manter contexto entre chamadas.
Isso permite que intermediários como caches e proxies entendam a semântica das requisições, algo impossível no RPC onde cada serviço inventa seus próprios nomes de função.

## 3. Evolução de Contrato

No Protobuf (gRPC), basta adicionar `string unidade = 3;` ao `.proto`. Clientes antigos ignoram tags desconhecidas, sendo backward compatible por design.
No REST com JSON (Tarefa 3), adicionar um campo novo funciona se o cliente ignorar campos extras, mas não há garantia em tempo de compilação.
A evolução no REST depende de convenção (versionamento de URL, OpenAPI), enquanto no gRPC o compilador garante compatibilidade.
O `.proto` funciona como contrato explícito; no REST o contrato é implícito e mais frágil.

## 4. Escolha de Tecnologia

Para parceiros externos: REST com JSON. HTTP é universal, JSON é legível, e a natureza stateless facilita caches e load balancers. Qualquer dev consome sem ferramenta especial.
Para microsserviços internos: gRPC com Protobuf. Serialização binária compacta, tipagem forte via `.proto` verificada em compilação, e HTTP/2 com multiplexação reduzem latência.
Internamente não precisa de legibilidade humana, e o contrato `.proto` compartilhado impede inconsistências entre serviços.
Conforme a Tarefa 5, cada paradigma domina num contexto: REST para APIs públicas, gRPC para comunicação interna.

## 5. Conexão com Labs Anteriores

A transparência de acesso do RPC faz `proxy.calcular("soma", 10, 3)` parecer local, mas o Lab 04 mostrou que transparência excessiva esconde custos reais.
Uma chamada local leva nanossegundos; uma remota pode levar centenas de milissegundos ou falhar por timeout/rede.
Um dev enganado pela transparência pode colocar chamadas RPC num loop de milhares de iterações, causando latência brutal e falhas parciais.
O correto seria usar batch ou streaming (gRPC suporta), mas a sintaxe idêntica à local não incentiva isso. É preciso lembrar que se está cruzando uma fronteira de rede.
