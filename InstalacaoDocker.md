# Tutorial de Instalação do Docker no Ubuntu

## 1. Atualizar os pacotes existentes

Antes de instalar o Docker, é importante garantir que o sistema está atualizado. Execute os seguintes comandos no terminal:

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 2. Instalar dependências necessárias

O Docker requer alguns pacotes adicionais para funcionar corretamente. Instale-os com o seguinte comando:

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```

---

## 3. Adicionar a chave GPG oficial do Docker

Para garantir que os pacotes do Docker venham de uma fonte confiável, adicione a chave GPG oficial do Docker:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

---

## 4. Adicionar o repositório do Docker

Adicione o repositório oficial do Docker ao seu sistema:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

## 5. Instalar o Docker

Agora que o repositório foi adicionado, atualize os pacotes e instale o Docker:

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

---

## 6. Verificar a instalação do Docker

Após a instalação, você pode verificar se o Docker foi instalado corretamente executando o seguinte comando:

```bash
sudo docker --version
```

Isso deve exibir a versão do Docker instalada no sistema.

---

## 7. Habilitar o Docker para iniciar automaticamente

Ative o Docker para ser iniciado automaticamente no boot do sistema:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

---

## 8. Executar Docker sem `sudo` (opcional)

Por padrão, o Docker precisa ser executado com `sudo`. Para permitir que o Docker seja usado sem `sudo`, adicione seu usuário ao grupo `docker`:

```bash
sudo usermod -aG docker $USER
```

Após executar esse comando, é necessário fazer logout e login novamente ou reiniciar o sistema para que as alterações entrem em vigor.

---

## 9. Testar a instalação

Execute o seguinte comando para verificar se o Docker está funcionando corretamente:

```bash
docker run hello-world
```

Esse comando faz o download e executa uma imagem de teste chamada `hello-world`, verificando se o Docker foi instalado e está operando corretamente.

---

## 10. Habilitar o Daemon Docker para Gerenciamento Remoto

Se você precisa habilitar o Docker para ser gerenciado remotamente, siga os passos abaixo para configurar o daemon Docker:

### 10.1 Criar diretório de configuração do daemon

Primeiro, crie o diretório onde será colocada a configuração personalizada do serviço Docker:

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d/
```

### 10.2 Criar o arquivo de override de configuração

Em seguida, crie o arquivo de configuração `override.conf`:

```bash
sudo vi /etc/systemd/system/docker.service.d/override.conf
```

### 10.3 Adicionar as configurações de host e porta

Dentro do arquivo `override.conf`, adicione as linhas abaixo para configurar o daemon Docker a fim de escutar tanto no socket padrão quanto na porta TCP 2376 para acesso remoto:

```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376
```

- **`-H fd://`**: Mantém o Docker rodando no socket padrão para comunicação local.
- **`-H tcp://0.0.0.0:2376`**: Permite que o Docker escute em todas as interfaces de rede (`0.0.0.0`), na porta **2376** para conexões remotas.

### 10.4 Reinicializar o Docker e o daemon

Para aplicar as novas configurações, reinicie o daemon do Docker:

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 10.5 Verificar se o Docker está escutando na porta remota

Para garantir que o Docker está configurado corretamente para gerenciamento remoto, use o seguinte comando para verificar se o daemon está escutando na porta configurada:

```bash
sudo netstat -tlpn | grep dockerd
```

Você deverá ver algo semelhante a:

```
tcp        0      0 0.0.0.0:2376            0.0.0.0:*               LISTEN      1234/dockerd
```

Isso confirma que o Docker está pronto para ser acessado remotamente pela porta 2376.

---

### 10.6 Conclusão

Agora, o daemon Docker está configurado para aceitar conexões remotas, permitindo o gerenciamento do Docker a partir de outras máquinas. Lembre-se de garantir a segurança do seu ambiente, configurando certificados SSL e outros mecanismos de autenticação para proteger o acesso remoto ao Docker.
```
