# 1. Instanciação da Máquina Virtual

O `Vagrantfile` apresentado a seguir corresponde ao modelo validado e utilizado nas edições anteriores do minicurso, servindo como base para a criação do ambiente de laboratório.


### Convenção de Nomenclatura das Máquinas Virtuais

As máquinas virtuais devem seguir o padrão de nomenclatura abaixo:

```text
iliada-<ambiente>-rnp-minicurso-<sigla_estado><numero_vm_host>
```

**Exemplo:**

```text
iliada-prod-rnp-minicurso-rj1
iliada-prod-rnp-minicurso-pr3
iliada-dev-rnp-minicurso-ba2
```

---

### Alocação de Máquinas Virtuais por Host

Cada host pode acomodar até **5 máquinas virtuais por ambiente**, seguindo os intervalos de portas definidos na tabela abaixo:

| Máquina Virtual | Faixa de Portas |
| --------------- | --------------- |
| VM 1            | 01 – 20         |
| VM 2            | 21 – 40         |
| VM 3            | 41 – 60         |
| VM 4            | 61 – 80         |
| VM 5            | 81 – 99         |

---

### Controle de Alocação

A distribuição atual de hosts, máquinas virtuais e faixas de portas pode ser consultada na planilha abaixo:

📊 **[Planilha de Alocação de Ambientes e Portas](https://nasnuvensrnp.sharepoint.com/:x:/r/sites/ILIADA/_layouts/15/Doc.aspx?sourcedoc=%7BDA2E4EC2-0DCC-43E7-B935-391C8E2505CF%7D&file=Minicurso%20HandsOn%20ILIADA-%20WTestbeds%202025.xlsx&action=default&mobileredirect=true)**



Exemplo: Primeira VM (1) do Host Alocado no PoP do estado do Rio de Janeiro (RJ)

#### 1.1 Crie o arquivo Vagrantfile
```bash
nano Vagrantfile
```
```bash
Vagrant.configure("2") do |config|
  # Define a VM com o nome "iliada-204-rnp-minicurso-rj1"
  config.vm.define "iliada-204-rnp-minicurso-rj1" do |vm1|
    # Especifica a imagem base (Ubuntu 22.04 genérica)
    vm1.vm.box = "generic/ubuntu2204"
    # Define o hostname da VM
    vm1.vm.hostname = "iliada-204-rnp-minicurso-rj1"

    # Configurações específicas para o provedor libvirt/KVM
    vm1.vm.provider "libvirt" do |libvirt|
      libvirt.driver = "kvm"                            # Usa o driver KVM
      libvirt.uri = "qemu:///system"                    # URI do hypervisor
      libvirt.cpus  = 8                                 # Número de CPUs
      libvirt.memory = 16400                            # Memória RAM (em MB)
      libvirt.machine_virtual_size = 128                # Tamanho do disco (em GB)
      libvirt.management_network_name = "iliada-204-nat"         # Nome da rede NAT
      libvirt.management_network_mode = "nat"                    # Tipo NAT
      libvirt.management_network_address = "10.100.204.0/24"     # Sub-rede
    end

    # Define uma interface de rede privada com IP fixo
    vm1.vm.network :private_network,
      :libvirt__network_name => "204-iliada",
      :libvirt__netmask => "255.255.0.0",
      :ip => "10.204.21.1"

    # Mapeamento de portas entre host e VMs 1
    vm1.vm.network "forwarded_port", guest: 22, host: 20401, protocol: "tcp"      # SSH
    vm1.vm.network "forwarded_port", guest: 2000, host: 20402, protocol: "tcp"    # Grafana
    vm1.vm.network "forwarded_port", guest: 9090, host: 20403, protocol: "tcp"    # Prometheus
    vm1.vm.network "forwarded_port", guest: 5901, host: 20405, protocol: "tcp"    # VNC
    vm1.vm.network "forwarded_port", guest: 6080, host: 20408, protocol: "tcp"    # noVNC
    vm1.vm.network "forwarded_port", guest: 3000, host: 20410, protocol: "tcp"    # Portal
    vm1.vm.network "forwarded_port", guest: 3001, host: 20411, protocol: "tcp"    # Portal Fabric
    vm1.vm.network "forwarded_port", guest: 3002, host: 20412, protocol: "tcp"    # Portal Besu
    vm1.vm.network "forwarded_port", guest: 8545, host: 20414, protocol: "tcp"    # Hyperledger Besu JSON-RPC
    vm1.vm.network "forwarded_port", guest: 4545, host: 20415, protocol: "tcp"    # Hyperledger Besu JSON-RPC aluno
    vm1.vm.network "forwarded_port", guest: 7151, host: 20416, protocol: "tcp"    # Hyperledger Fabric Peer
    vm1.vm.network "forwarded_port", guest: 7051, host: 20417, protocol: "tcp"    # Hyperledger Fabric Peer aluno
    vm1.vm.network "forwarded_port", guest: 80, host: 20418, protocol: "tcp"      # HTTP cc-tools
    vm1.vm.network "forwarded_port", guest: 7200, host: 20419, protocol: "tcp"    # HTTP cc-tools aluno

    # Script de provisionamento que será executado na criação da VM
    vm1.vm.provision "shell", inline: <<-SHELL
      # Cria o usuário "iliada" com home e bash
      useradd -m -d "/home/iliada" -s /bin/bash "iliada"
      # Adiciona o usuário ao grupo docker e sudo
      sudo usermod -aG docker iliada
      usermod -aG sudo iliada
      # Permite sudo sem senha para "iliada"
      sed -i '51i\\iliada ALL=(ALL:ALL) NOPASSWD:ALL' /etc/sudoers

      # Define a senha do usuário "iliada"
      PASS="sorj3508#"
      echo -e "$PASS\\n$PASS" | sudo passwd "iliada"

      # Atualiza pacotes e instala ferramentas básicas
      su iliada && cd /home/iliada/
      sudo apt update
      sudo apt install -y net-tools git make

      # Configura o repositório do Docker
      sudo mkdir -p /etc/apt/keyrings
      curl -fsSl https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
      echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/u                                                                                buntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      sudo apt update
      sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-compose

      # Garante que o usuário está no grupo docker
      usermod -aG docker iliada

      # Configura o fuso horário
      timedatectl set-timezone America/Sao_Paulo

      # Instala o NVM e Node.js v20 para o usuário "iliada"
      su -c "curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash" iliada
      su -c "NVM_DIR=/home/iliada/" iliada
      sudo -u iliada bash -c 'source /home/iliada/.nvm/nvm.sh && nvm install 20'

      # Instala kubectl
      curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
      curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
      echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
      sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

      # Instala utilitário jq e kind (Kubernetes-in-Docker)
      sudo apt install -y jq
      [ $(uname -m) = x86_64 ] && curl -Lo /home/iliada/kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
      sudo chmod +x /home/iliada/kind
      sudo mv /home/iliada/kind /usr/local/bin/kind

      # Instala XFCE (ambiente gráfico) + tightvnc + ferramentas úteis
      apt install -y cloud-guest-utils xfce4 xfce4-goodies build-essential net-tools python3-pip curl websockify novnc
      apt install -y tightvncserver

      # Instala Firefox e ferramentas de área de transferência
      sudo apt install -y firefox autocutsel

      # Expande automaticamente a partição/disco da VM
      growpart /dev/vda 3
      pvresize /dev/vda3
      lvextend -r -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
    SHELL
  end
end
```

#### 1.2 Criar e iniciar a máquina virtual

Execute o comando abaixo para provisionar e iniciar a máquina virtual definida no `Vagrantfile`:

```bash
vagrant up
```

# 2. Instalação do VNC e Google Chrome

#### 2.1 Acessar a máquina virtual

Conecte-se à máquina virtual utilizando o Vagrant:

```bash
vagrant ssh
```

#### 2.2 Acessar o usuário do laboratório

Altere para o usuário utilizado nos laboratórios:

**Usuário:** iliada  
**Senha:** sorj3508#

```bash
su - iliada
```

#### 2.3 Criar e executar o script de instalação

Crie o arquivo `setup-vnc.sh`, adicione o conteúdo disponibilizado a seguir e execute o script para instalar e configurar o ambiente gráfico, o VNC e o Google Chrome.

```bash
nano setup-vnc.sh
```
Cole o conteúdo abaixo:

```bash
#!/bin/bash

# ==========================================================
# Configuração automática do TigerVNC + XFCE + Google Chrome
# Usuário: iliada
# Senha Linux: iliada2026
# Senha VNC: iliada2026
# ==========================================================

USUARIO="iliada"
SENHA="iliada2026"
DIR_VNC="/home/$USUARIO/.vnc"

if [ "$EUID" -ne 0 ]; then
  echo "Erro: Este script precisa ser executado como root."
  echo "Use: sudo ./setup_vnc.sh"
  exit 1
fi

echo ">>> Alterando senha do usuário $USUARIO..."
echo "$USUARIO:$SENHA" | chpasswd

echo ">>> Instalando pacotes necessários..."
apt update

apt install -y \
    tigervnc-standalone-server \
    tigervnc-common \
    xfce4 \
    xfce4-goodies \
    autocutsel \
    dbus-x11 \
    wget \
    curl \
    ca-certificates \
    fonts-liberation \
    libappindicator3-1 \
    xdg-utils

echo ">>> Instalando Google Chrome..."
wget -q https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb \
    -O /tmp/google-chrome-stable_current_amd64.deb

apt install -y /tmp/google-chrome-stable_current_amd64.deb

rm -f /tmp/google-chrome-stable_current_amd64.deb

echo ">>> Versão do Google Chrome instalada:"
google-chrome --version || true

echo ">>> Criando diretório VNC..."
mkdir -p "$DIR_VNC"

echo ">>> Configurando senha do VNC..."
su - "$USUARIO" -c "
mkdir -p ~/.vnc
echo '$SENHA' | vncpasswd -f > ~/.vnc/passwd
chmod 600 ~/.vnc/passwd
"

echo ">>> Criando arquivo xstartup..."
cat << 'EOF' > "$DIR_VNC/xstartup"
#!/bin/sh

unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS

sleep 2

/usr/bin/autocutsel -s CLIPBOARD -fork
/usr/bin/autocutsel -s PRIMARY -fork

xset s off
xset s noblank
xset -dpms

(
    sleep 10

    xfconf-query \
        -c xfce4-power-manager \
        -p /xfce4-power-manager/dpms-enabled \
        -n -t bool -s false || true

    xfconf-query \
        -c xfce4-power-manager \
        -p /xfce4-power-manager/blank-on-ac \
        -n -t int -s 0 || true

    xfconf-query \
        -c xfce4-power-manager \
        -p /xfce4-power-manager/inactivity-on-ac \
        -n -t int -s 0 || true

    xfconf-query \
        -c xfce4-power-manager \
        -p /xfce4-power-manager/lock-screen-suspend-hibernate \
        -n -t bool -s false || true

    xfconf-query \
        -c xfce4-session \
        -p /startup/screensaver/enabled \
        -n -t bool -s false || true

) &

exec /usr/bin/startxfce4
EOF

echo ">>> Ajustando permissões..."
chmod +x "$DIR_VNC/xstartup"
chown -R "$USUARIO:$USUARIO" "$DIR_VNC"

echo ">>> Criando serviço Systemd..."
cat << 'EOF' > /etc/systemd/system/vncserver@.service
[Unit]
Description=Servidor VNC TigerVNC Display %i
After=network.target

[Service]
Type=forking
User=iliada
Group=iliada
WorkingDirectory=/home/iliada

# PIDFile=/home/iliada/.vnc/%H:%i.pid

ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
ExecStart=/usr/bin/vncserver :%i \
    -geometry 1920x1080 \
    -depth 24 \
    -localhost no

ExecStop=/usr/bin/vncserver -kill :%i

Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

echo ">>> Ajustando permissões do serviço..."
chmod 644 /etc/systemd/system/vncserver@.service

echo ">>> Recarregando Systemd..."
systemctl daemon-reload

echo ">>> Habilitando serviço..."
systemctl enable vncserver@1.service

echo ">>> Reiniciando serviço..."
systemctl restart vncserver@1.service

echo
echo "=============================================="
echo "Configuração concluída com sucesso!"
echo "=============================================="
echo "Usuário Linux : $USUARIO"
echo "Senha Linux   : $SENHA"
echo "Senha VNC     : $SENHA"
echo
echo "Google Chrome:"
google-chrome --version || true
echo
echo "Acesso VNC:"
echo "IP_DA_VM:5901"
echo
echo "Verificar status:"
echo "systemctl status vncserver@1.service"
echo
echo "Verificar logs:"
echo "journalctl -u vncserver@1.service -f"
echo "=============================================="
```


Execute:
```bash
chmod +x setup_vnc.sh
sudo ./setup_vnc.sh

sudo systemctl daemon-reload
sudo systemctl restart vncserver@1.service
sudo systemctl status vncserver@1.service
```

# 3. Clonagem dos Repositórios

#### 3.1 Criar o diretório de trabalho

Crie o diretório que será utilizado para armazenar os repositórios do laboratório e acesse-o:

```bash
mkdir -p ~/iliada
cd ~/iliada
```

#### 3.2 Clonar os repositórios da rede Besu

Clone os repositórios contendo os ambientes do instrutor e dos alunos para os laboratórios baseados em Hyperledger Besu:

```bash
git clone https://user:access_token@git.rnp.br/iliada-blockchain/m2/minicurso-handson-besu.git rede-besu

git clone https://user:access_token@git.rnp.br/iliada-blockchain/m2/minicurso-handson-besu-aluno.git rede-besu-aluno
```

#### 3.3 Clonar os repositórios da rede Fabric

Clone os repositórios contendo os ambientes do instrutor e dos alunos para os laboratórios baseados em Hyperledger Fabric:

```bash
# git clone <repositorio-fabric> rede-fabric

# git clone <repositorio-fabric-aluno> rede-fabric-aluno
```

---

# 4. Instalação do Java 17

O Hyperledger Besu é executado sobre a JVM (Java Virtual Machine). Portanto, é necessário instalar o Java 17 antes de iniciar os laboratórios.

#### 4.1 Instalar o Java

```bash
apt-cache search openjdk | grep openjdk-17

sudo apt install -y openjdk-17-jre
sudo apt install -y openjdk-17-jdk
```

#### 4.2 Validar a instalação

```bash
java --version
```

#### 4.3 Configurar a variável JAVA_HOME

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH

echo $JAVA_HOME
```
# 5 Intalar a Extensão do MetaMask

Acesse:

https://metamask.io/download

Selecione **Chrome** ou o navegador de sua escolha e clique em **Ex: Usar no chrome ou Add Extension**.

![Download MetaMask](images/metamask/01.1-metamask-download.png)
![Add Extension MetaMask](images/metamask/01.2-add-ext.png)
![Add Extension MetaMask](images/metamask/01.3-add-ext.png)

Fixe a extensão no seu navegador para facilitar o acesso:

![Fix Extension MetaMask](images/metamask/01.4-fix-ext.png)
---

# 6. Conectar a Carteira MetaMask

#### 6.1 Importar uma carteira existente

Selecione a opção **Importar carteira existente** e informe a frase de recuperação (Secret Recovery Phrase) abaixo:

```text
wood latin transfer merge champion claw shuffle jelly rebuild castle basic weather
```

![Import Wallet MetaMask](images/metamask/02.2-import-wallet.png)

---

# 7. Importar uma Conta de Teste

No menu da MetaMask, acesse:

```text
Contas
→ Importar Conta
```

Selecione a opção **Chave Privada** e informe a chave da conta de teste:

```text
0x8f2a55949038a9610f50fb23b5883af3b4ecb3c3bb792cbcefbd1542c692be63
```

![Import Account MetaMask](images/metamask/03-import-account.png)

---

# 8. Adicionar a Rede Besu

Preencha os dados da rede conforme o ambiente disponibilizado para o laboratório.

### Exemplo do Minicurso

| Campo | Valor |
|---------|---------|
| Nome da Rede | iliada-204-rnp-minicurso-rn1 |
| RPC URL | http://200.137.0.26:20414 |
| Chain ID | 10001 |
| Símbolo | ETH |

![Connect network Metamask](images/metamask/04-connect-network.png)

---

## Troubleshooting de Conectividade RPC

A MetaMask não conseguiu conectar à rede Besu?

Valide a disponibilidade do serviço RPC executando os testes abaixo.

### 1. Dentro da máquina virtual

```bash
curl -X POST \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":1}' \
http://localhost:8545
```

### 2. No host físico que executa o Vagrant

```bash
curl -X POST \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":1}' \
http://localhost:20414
```

### 3. A partir de uma máquina externa

```bash
curl -X POST \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":1}' \
http://200.159.254.122:20414
```

Se o terceiro teste não retornar uma resposta do Hyperledger Besu, verifique as regras de firewall, o redirecionamento de portas do Vagrant e a conectividade de rede entre o cliente e o host.
# 9. Conectando-se à Rede

```text
MetaMask
→ Selecionar Rede
→ iliada-besu
```
![Connect network and Metamask wallet](images/metamask/05-connect-account-and-network.png)

---

# 10. Adicionar a MetaMask aos Favoritos

Para facilitar o acesso durante os laboratórios, recomenda-se fixar a extensão da MetaMask na barra de ferramentas do navegador e manter a barra de favoritos visível.

#### 10.1 Fixar a MetaMask e Remix na Barra de Ferramentas

Clique no ícone de extensões do navegador e fixe a MetaMask para que ela fique sempre acessível na barra superior.

![Add Favoritos](images/interno/10.1-add-favoritos.png)

#### 10.2 Habilitar a Barra de Favoritos

Caso a barra de favoritos esteja oculta, habilite sua exibição para facilitar o acesso aos links e materiais utilizados durante o minicurso.

![Habilitar Favoritos](images/interno/10.2-habilitar-barra-favoritos.png)

### 10.3 URL Remix IDE:

Acesse e adicione aos favoritos:

```text
https://remix.ethereum.org/
```