# Hands-On Hyperledger Besu - Remix IDE

## Visão Geral do Remix IDE

O Remix IDE é um ambiente de desenvolvimento integrado (IDE) baseado em navegador, amplamente utilizado para criar, testar, implantar e interagir com contratos inteligentes compatíveis com a Ethereum Virtual Machine (EVM). Por funcionar diretamente no navegador, o Remix elimina a necessidade de instalações complexas e oferece uma forma rápida e prática de iniciar o desenvolvimento em blockchain.

A ferramenta possui suporte nativo à linguagem Solidity, permitindo escrever contratos inteligentes, compilá-los, executar testes e realizar implantações em redes locais, públicas ou permissionadas, como o Hyperledger Besu. Além disso, integra-se diretamente com carteiras digitais como a MetaMask, possibilitando a assinatura de transações e a interação com contratos implantados em redes blockchain reais.

Durante este Hands-On, o Remix será utilizado para desenvolver e implantar contratos inteligentes na rede Hyperledger Besu do minicurso, permitindo que os participantes experimentem todo o ciclo de desenvolvimento de aplicações blockchain: criação do contrato, compilação, implantação, execução de transações e consulta dos resultados registrados na rede.

Principais funcionalidades do Remix:

- Desenvolvimento de contratos inteligentes em Solidity;
- Compilação automática e análise de código;
- Implantação em redes Ethereum e Hyperledger Besu;
- Integração direta com MetaMask;
- Execução de transações e chamadas de funções;
- Visualização de logs, eventos e resultados;
- Ambiente totalmente web, sem necessidade de instalação.

Em resumo, o Remix IDE funciona como uma plataforma completa para desenvolvimento e experimentação de aplicações blockchain, sendo uma das ferramentas mais utilizadas para aprendizado, prototipação e testes de contratos inteligentes compatíveis com EVM.

## Objetivo

Neste roteiro você irá conectar a MetaMask à rede do minicurso, criar um contrato no Remix, realizar o deploy na rede Hyperledger Besu e executar transações utilizando a **Account** criada anteriormente.

---

# 1. Acessando o Remix IDE

Acesse:

https://remix.ethereum.org

![Acesso ao Remix](images/01-remix-home.png)

---

# 2. Criando um Workspace

Clique em **Create New Workspace**.

Nome sugerido:

```text
handson-besu
```

## Imagem

![Workspace](images/02-workspace.png)

---

# 3. Criando o Contrato

Arquivo:

```text
contracts/SimpleStorage.sol
```

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleStorage {

    string private message;

    function setMessage(string memory _message) public {
        message = _message;
    }

    function getMessage() public view returns (string memory) {
        return message;
    }
}
```

## Imagem

![Contrato](images/03-contract.png)

---

# 4. Compilando o Contrato

```text
Compiler: 0.8.20
```

Clique em:

```text
Compile SimpleStorage.sol
```

## Imagem

![Compilação](images/04-compile.png)

---

# 5. Conectando a MetaMask

```text
Conta: Account 2

Rede:
iliada-204-rnp-minicurso-m1

RPC URL:
http://200.137.0.26:20414

Chain ID:
10001

Símbolo:
ETH
```

## Imagem

![MetaMask](images/05-metamask-connected.png)

---

# 6. Configurando o Deploy

```text
Environment:
Injected Provider - MetaMask
```

Verifique:

```text
Network:
iliada-204-rnp-minicurso-m1

Account:
Account 2
```

## Imagem

![Deploy Setup](images/06-deploy-setup.png)

---

# 7. Implantando o Contrato

Selecione:

```text
SimpleStorage
```

Clique em:

```text
Deploy
```

Confirme a transação na MetaMask.

## Imagem

![Deploy](images/07-deploy.png)

---

# 8. Contrato Implantado

Exemplo:

```text
SimpleStorage at 0x...
```

## Imagem

![Contrato Implantado](images/08-deployed.png)

---

# 9. Gravando Dados na Blockchain

Digite:

```text
Olá Hands-On ILIADA
```

Clique em:

```text
setMessage
```

## Imagem

![Set Message](images/09-setmessage.png)

---

# 10. Consultando os Dados

Clique em:

```text
getMessage()
```

Resultado esperado:

```text
Olá Hands-On ILIADA
```

## Imagem

![Get Message](images/10-getmessage.png)

---

# 11. Consultando Saldo

Verifique:

```text
Conta: Account 2
Rede: iliada-204-rnp-minicurso-m1
```

## Imagem

![Saldo](images/11-saldo.png)

---

# 12. Realizando uma Transação

Clique em:

```text
Send
```

Informe:

```text
Endereço destino
Quantidade de ETH
```

## Imagem

![Transação](images/12-send.png)

---

# 13. Consultando o Histórico

Abra a aba:

```text
Activity
```

Verifique:

- Hash da transação
- Status
- Valor transferido
- Taxa da rede

## Imagem

![Histórico](images/13-history.png)

---

# Fluxo Completo

```text
MetaMask
    ↓
Conectar Rede Besu
    ↓
Abrir Remix
    ↓
Criar Contrato
    ↓
Compilar
    ↓
Deploy
    ↓
Confirmar Transação
    ↓
Executar Contrato
    ↓
Consultar Resultado
    ↓
Consultar Histórico
```
