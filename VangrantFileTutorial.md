## Criar arquivo VagrantFile

### 1. Definir caixa base
```ruby
Vagrant.configure("2") do |config|
config.vm.box = "ubuntu/focal64"
```

Este comando define a imagem de sistema operacional da VM. Aqui, estamos usando a imagem do Ubuntu 20.04 LTS (Focal Fossa). O Vagrant irá baixar essa caixa automaticamente, caso não esteja presente no seu sistema.

### 2. Configuração de Disco

```ruby
config.disksize.size = '40GB'
```

A configuração acima aumenta o tamanho do disco da máquina virtual para 40GB. Isso garante que a VM tenha espaço suficiente para o servidor web e outros pacotes que possam ser instalados.

### 3. Redirecionamento de Portas

```ruby
config.vm.network "forwarded_port", guest: 80, host: 80
config.vm.network "forwarded_port", guest: 81, host: 81
config.vm.network "forwarded_port", guest: 3306, host: 3306
config.vm.network "forwarded_port", guest: 8080, host: 8080
config.vm.network "forwarded_port", guest: 9000, host: 9000
config.vm.network "forwarded_port", guest: 19999, host: 19999
config.vm.network "forwarded_port", guest: 9001, host: 9001
```

Aqui, estamos redirecionando várias portas da máquina virtual para a máquina hospedeira. Isso permite que você acesse os serviços da VM (como o servidor web ou banco de dados) diretamente do seu computador:

- Porta 80: Servidor web (Apache)
- Porta 3306: Banco de dados MySQL
- Portas 8080, 9000, 9001, etc.: Outros serviços ou aplicativos web.

### 4. Configuração de Rede Privada

```ruby
config.vm.network "private_network", ip: "192.168.33.88"
```

Essa linha configura a VM com uma rede privada, atribuindo a ela o IP `192.168.33.88`. Isso permite que a máquina hospedeira acesse a VM diretamente por esse IP, facilitando o desenvolvimento e a configuração de servidores.

### 5. Configuração de Memória

```ruby
config.vm.provider "virtualbox" do |vb|
  vb.memory = "2048"
end
```

Aqui, alocamos 2GB de RAM (2048MB) para a máquina virtual, garantindo que ela tenha memória suficiente para rodar o sistema operacional e os serviços instalados.

### 6. Provisionamento do Servidor

O provisionamento é o processo de configurar e instalar software automaticamente dentro da VM. Abaixo, cada passo do script é detalhado:

#### a. Atualizando os Pacotes

```bash
config.vm.provision "shell", inline: <<-SHELL
sudo apt-get update
```

Este comando atualiza a lista de pacotes disponíveis e suas versões no sistema.

#### b. Instalando o Apache

```bash
sudo apt-get install -y apache2
```

Aqui, o servidor web Apache é instalado na máquina virtual. Ele será usado para hospedar os dois domínios.

#### c. Criando Diretórios para os Domínios

```bash
sudo mkdir -p /var/www/lab.com/public_html
sudo mkdir -p /var/www/motomoto.com/public_html
```

Esses comandos criam os diretórios onde os arquivos HTML de cada domínio serão armazenados.

#### d. Configurando Permissões

```bash
sudo chown -R $USER:$USER /var/www/lab.com/public_html
sudo chown -R $USER:$USER /var/www/motomoto.com/public_html
```

Aqui, ajustamos as permissões dos diretórios criados para garantir que o usuário atual tenha controle sobre eles.

#### e. Copiando Conteúdo para os Diretórios

```bash
sudo cp -r /vagrant/Forca/* /var/www/lab.com/public_html/
sudo cp -r /vagrant/freeway/* /var/www/motomoto.com/public_html/
```

Esses comandos copiam os arquivos HTML (presumivelmente dos diretórios `Forca` e `freeway` no seu projeto) para os diretórios do servidor web.

#### f. Criando os Virtual Hosts

Cada domínio (ou "site") terá uma configuração de Virtual Host no Apache, permitindo que o servidor responda por diferentes URLs.

##### Configuração do `lab.com`

```bash
sudo bash -c 'cat > /etc/apache2/sites-available/lab.com.conf <<EOF
<VirtualHost *:80>
  ServerAdmin admin@lab.com
  ServerName lab.com
  ServerAlias www.lab.com
  DocumentRoot /var/www/lab.com/public_html
  ErrorLog ${APACHE_LOG_DIR}/boladoida_error.log
  CustomLog ${APACHE_LOG_DIR}/boladoida_access.log combined
</VirtualHost>
EOF'
```

Esta configuração define o site `lab.com`, incluindo o endereço de e-mail do administrador, o domínio, o diretório raiz do site, e onde os logs de erros e acessos serão armazenados.

##### Configuração do `motomoto.com`

```bash
sudo bash -c 'cat > /etc/apache2/sites-available/motomoto.com.conf <<EOF
<VirtualHost *:80>
  ServerAdmin admin@motomoto.com
  ServerName motomoto.com
  ServerAlias www.motomoto.com
  DocumentRoot /var/www/motomoto.com/public_html
  ErrorLog ${APACHE_LOG_DIR}/cavalodoido_error.log
  CustomLog ${APACHE_LOG_DIR}/cavalodoido_access.log combined
</VirtualHost>
EOF'
```

Da mesma forma, essa configuração cria o Virtual Host para o domínio `motomoto.com`.

#### g. Habilitando os Sites

```bash
sudo a2ensite lab.com.conf
sudo a2ensite motomoto.com.conf
```

Com esses comandos, habilitamos os arquivos de configuração dos dois domínios (`lab.com` e `motomoto.com`) no Apache.

#### h. Desabilitando o Site Padrão

```bash
sudo a2dissite 000-default.conf
```

Aqui, desabilitamos o site padrão do Apache, garantindo que apenas os dois domínios personalizados sejam servidos.

#### i. Reiniciando o Apache

```bash
sudo systemctl restart apache2
SHELL
end
```

Finalmente, o servidor Apache é reiniciado para aplicar todas as novas configurações.

---

## Ao final, você terá um documento igual a [este aqui](https://github.com/JardelSO/eng/blob/main/VagrantFile)
