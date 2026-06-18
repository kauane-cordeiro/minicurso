# Desafio Rede Besu — Adicionando um Novo Validador à Rede

## Objetivo

Neste desafio, você atuará como uma nova organização que deseja ingressar em uma rede blockchain permissionada Hyperledger Besu já existente.

Diferentemente do laboratório anterior, não será necessário alterar o arquivo `genesis.json` nem modificar o arquivo `static-nodes.json` da rede.

O novo nó será criado de forma independente, conectado manualmente à rede existente, sincronizado com a blockchain e, posteriormente, submetido ao processo de governança QBFT para ingressar no conjunto de validadores.

Ao final deste desafio você será capaz de:

* Criar o material criptográfico de um novo nó;
* Configurar um novo nó Besu;
* Iniciar um nó externo à rede;
* Conectá-lo manualmente aos peers existentes;
* Sincronizar a blockchain;
* Solicitar a entrada do nó no consenso QBFT;
* Realizar a votação para aprovação do novo validador.

---

# Arquitetura do Desafio

```text
                    Rede Existente

     ┌─────────────┐    ┌─────────────┐
     │ Org1 Node1  │────│ Org1 Node2  │
     └─────────────┘    └─────────────┘
             │
             │
     ┌─────────────┐    ┌─────────────┐
     │ Org2 Node1  │────│ Org2 Node2  │
     └─────────────┘    └─────────────┘
             │
             │
     ┌─────────────┐    ┌─────────────┐
     │ Org3 Node1  │────│ Org3 Node2  │
     └─────────────┘    └─────────────┘

                ▲
                │
                │ Solicita ingresso
                │
     ┌──────────────────────┐
     │ Nova Organização     │
     │ Org4 Node1           │
     └──────────────────────┘
```

---

# 1. Preparação do Ambiente

Configure as variáveis utilizadas durante o laboratório.

Execute:

```bash
BASEDIR=/home/iliada/iliada/rede-besu
CONFIGDIR=/home/iliada/iliada/rede-besu/config

cd $BASEDIR
```

---

# 2. Criar a Estrutura do Novo Nó

Neste desafio será criada uma nova organização contendo um único nó:

```text
org4-node1
```

Crie a estrutura de diretórios.

Execute:

```bash
mkdir -p $CONFIGDIR/nodes/org4-node1
mkdir -p $BASEDIR/volumes/org4-node1
```

Verifique:

```bash
ls -la $CONFIGDIR/nodes
ls -la $BASEDIR/volumes
```

---

# 3. Gerar o Material Criptográfico

Acesse o diretório dos binários do Besu.

Execute:

```bash
cd $BASEDIR/bin
```

Gere a chave pública.

Execute:

```bash
besu-24.5.4/bin/besu \
  --data-path=$CONFIGDIR/nodes/org4-node1 \
  public-key export \
  --to=$CONFIGDIR/nodes/org4-node1/key.pub
```

Gere o endereço do nó.

Execute:

```bash
besu-24.5.4/bin/besu \
  --data-path=$CONFIGDIR/nodes/org4-node1 \
  public-key export-address \
  --to=$CONFIGDIR/nodes/org4-node1/node.id
```

Verifique os arquivos criados.

Execute:

```bash
ls -la $CONFIGDIR/nodes/org4-node1
```

Visualize a chave pública.

Execute:

```bash
cat $CONFIGDIR/nodes/org4-node1/key.pub
```

Visualize o endereço do nó.

Execute:

```bash
cat $CONFIGDIR/nodes/org4-node1/node.id
```

---

# 4. Criar o Docker Compose do Novo Nó

Crie o arquivo de configuração.

Execute:

```bash
nano $CONFIGDIR/nodes/org4-node1/docker-compose.yml
```

Cole o conteúdo abaixo:

```yaml
services:
  org4-node1:
    image: hyperledger/besu:24.5.4
    restart: unless-stopped

    environment:
      TZ: America/Sao_Paulo
      LANG: en_US.UTF-8

      LOG4J_CONFIGURATION_FILE: "/var/lib/besu/log.xml"

      BESU_DATA_PATH: "/var/lib/besu"
      BESU_GENESIS_FILE: "/var/lib/besu/genesis.json"

      BESU_MAX_PEERS: "50"
      BESU_REMOTE_CONNECTIONS_LIMIT_ENABLED: "false"

      BESU_MIN_GAS_PRICE: "0"

      BESU_HOST_ALLOWLIST: "*"

      BESU_RPC_HTTP_ENABLED: "true"
      BESU_RPC_HTTP_HOST: "0.0.0.0"
      BESU_RPC_HTTP_PORT: "8545"

      BESU_RPC_HTTP_API: "ADMIN,ETH,TXPOOL,NET,QBFT,WEB3,DEBUG,TRACE,PERM"

      BESU_RPC_HTTP_CORS_ORIGINS: "*"

      BESU_METRICS_ENABLED: "true"
      BESU_METRICS_HOST: "0.0.0.0"
      BESU_METRICS_PORT: "9545"

      BESU_P2P_HOST: "0.0.0.0"
      BESU_P2P_PORT: "30399"

    user: "0"

    command: >-
      --Xdns-enabled=true
      --Xdns-update-enabled=true
      --rpc-http-port=8545
      --logging=DEBUG

    volumes:
      - /home/iliada/iliada/rede-besu/volumes/org4-node1/:/var/lib/besu/
      - /home/iliada/iliada/rede-besu/config/nodes/org4-node1/key:/var/lib/besu/key
      - /home/iliada/iliada/rede-besu/config/besu/log.xml:/var/lib/besu/log.xml
      - /home/iliada/iliada/rede-besu/config/besu/genesis.json:/var/lib/besu/genesis.json

    ports:
      - 30399:30399
      - 85499:8545
      - 95499:9545
```

Salve o arquivo.

```text
Ctrl + O
Enter
Ctrl + X
```

Verifique:

```bash
cat $CONFIGDIR/nodes/org4-node1/docker-compose.yml
```

---

# 5. Iniciar o Novo Nó

Acesse a pasta do novo nó.

Execute:

```bash
cd $CONFIGDIR/nodes/org4-node1
```

Inicie o container.

Execute:

```bash
docker-compose up -d
```

Verifique:

```bash
docker ps
```

Acompanhe os logs.

Execute:

```bash
docker-compose logs -f
```

Para sair dos logs:

```text
Ctrl + C
```

---

# 6. Verificar o Estado Inicial do Nó

Neste momento o nó ainda não conhece nenhum peer da rede.

Verifique a quantidade de peers.

Execute:

```bash
curl -X POST http://localhost:85499 \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc":"2.0",
  "method":"net_peerCount",
  "params":[],
  "id":1
}'
```

Resultado esperado:

```json
{
  "result":"0x0"
}
```

---

# 7. Descobrir um Peer da Rede Existente

Agora será necessário obter o endereço ENODE de um dos validadores da rede.

Execute em um dos nós da rede existente:

```bash
curl -X POST http://localhost:8545 \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc":"2.0",
  "method":"admin_nodeInfo",
  "params":[],
  "id":1
}'
```

Localize o campo:

```json
"enode":"enode://..."
```

Copie o valor completo.

Exemplo:

```text
enode://abc123...@10.0.0.10:30301
```

---

# 8. Conectar Manualmente o Novo Nó

Utilize o ENODE obtido anteriormente.

Execute:

```bash
curl -X POST http://localhost:85499 \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc":"2.0",
  "method":"admin_addPeer",
  "params":[
    "enode://SEU_ENODE_AQUI"
  ],
  "id":1
}'
```

Resultado esperado:

```json
{
  "result": true
}
```

---

# 9. Confirmar a Conexão com a Rede

Verifique novamente os peers.

Execute:

```bash
curl -X POST http://localhost:85499 \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc":"2.0",
  "method":"net_peerCount",
  "params":[],
  "id":1
}'
```

Resultado esperado:

```json
{
  "result":"0x1"
}
```

ou superior.

---

# 10. Sincronizar a Blockchain

Verifique o bloco atual da rede.

Execute:

```bash
curl -X POST http://localhost:8545 \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc":"2.0",
  "method":"eth_blockNumber",
  "params":[],
  "id":1
}'
```

Verifique o bloco atual do novo nó.

Execute:

```bash
curl -X POST http://localhost:85499 \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc":"2.0",
  "method":"eth_blockNumber",
  "params":[],
  "id":1
}'
```

Aguarde até que os valores sejam iguais ou muito próximos.

---

# 11. Consultar os Validadores Atuais

Execute:

```bash
curl -X POST http://localhost:8545 \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc":"2.0",
  "method":"qbft_getValidatorsByBlockNumber",
  "params":["latest"],
  "id":1
}'
```

Observe que o endereço do novo nó ainda não aparece entre os validadores.

---

# 12. Obter o Endereço do Novo Nó

Execute:

```bash
cat $CONFIGDIR/nodes/org4-node1/node.id
```

Armazene o valor em uma variável.

Execute:

```bash
NEW_VALIDATOR=$(cat $CONFIGDIR/nodes/org4-node1/node.id)

echo $NEW_VALIDATOR
```

---

# 13. Solicitar Entrada no Consenso

Envie a proposta de inclusão do novo validador.

Execute:

```bash
curl -X POST http://localhost:8545 \
-H "Content-Type: application/json" \
-d "{
  \"jsonrpc\":\"2.0\",
  \"method\":\"qbft_proposeValidatorVote\",
  \"params\":[
    \"$NEW_VALIDATOR\",
    true
  ],
  \"id\":1
}"
```

Resultado esperado:

```json
{
  "result": true
}
```

---

# 14. Realizar a Votação

Repita a votação a partir dos demais validadores da rede.

Exemplo:

```bash
curl -X POST http://localhost:8546 \
-H "Content-Type: application/json" \
-d "{
  \"jsonrpc\":\"2.0\",
  \"method\":\"qbft_proposeValidatorVote\",
  \"params\":[
    \"$NEW_VALIDATOR\",
    true
  ],
  \"id\":1
}"
```

```bash
curl -X POST http://localhost:8547 \
-H "Content-Type: application/json" \
-d "{
  \"jsonrpc\":\"2.0\",
  \"method\":\"qbft_proposeValidatorVote\",
  \"params\":[
    \"$NEW_VALIDATOR\",
    true
  ],
  \"id\":1
}"
```

Após atingir o quórum necessário, o nó será promovido a validador.

---

# 15. Confirmar a Entrada no Consenso

Execute:

```bash
curl -X POST http://localhost:8545 \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc":"2.0",
  "method":"qbft_getValidatorsByBlockNumber",
  "params":["latest"],
  "id":1
}'
```

O endereço do novo nó deverá aparecer na lista.

---

# 16. Consultar Métricas de Assinatura

Execute:

```bash
curl -X POST http://localhost:8545 \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc":"2.0",
  "method":"qbft_getSignerMetrics",
  "params":["earliest","latest"],
  "id":1
}'
```

Após alguns blocos, o novo validador começará a aparecer nas métricas de assinatura.

---

# 17. Validar o Novo Nó

Consultar Chain ID.

Execute:

```bash
curl -X POST http://localhost:85499 \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc":"2.0",
  "method":"eth_chainId",
  "params":[],
  "id":1
}'
```

Consultar altura da blockchain.

Execute:

```bash
curl -X POST http://localhost:85499 \
-H "Content-Type: application/json" \
-d '{
  "jsonrpc":"2.0",
  "method":"eth_blockNumber",
  "params":[],
  "id":1
}'
```

Consultar métricas Prometheus.

Execute:

```bash
curl http://localhost:95499/metrics
```

---

# Resultado Esperado

Ao final deste desafio:

* Uma nova organização foi criada.
* Um novo nó Besu foi configurado.
* O nó foi conectado manualmente à rede.
* O nó sincronizou toda a blockchain.
* Os validadores votaram sua entrada.
* O novo nó passou a participar do consenso QBFT.
* A rede foi expandida sem reinicialização e sem alteração do genesis.

---

# Fim do Desafio

Parabéns! Você adicionou uma nova organização à rede Hyperledger Besu e executou o processo completo de governança QBFT para inclusão de um novo validador.
