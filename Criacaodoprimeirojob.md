## 1. Concedendo permissões para gerenciamento do Docker

Acesse o servidor via SSH e execute os comandos abaixo para conceder as permissões necessárias para que tanto o Jenkins quanto o seu usuário possam gerenciar o Docker sem precisar de permissões adicionais:

```bash
sudo usermod -aG docker $USER
sudo usermod -aG docker jenkins
```

Após isso, é recomendável fazer logout e login novamente no terminal para que as mudanças de grupo sejam aplicadas.

---

## 2. Adicionando credenciais SSH no Jenkins para integração com GitHub

Agora, configure o Jenkins para interagir com o GitHub via SSH. Siga os passos:

1. No Jenkins, navegue até:  
   **Gerenciar Jenkins** → **Credenciais** → **System** → **Credenciais globais** → **Adicionar Credenciais**.
   
2. Preencha os campos da seguinte forma:
   - **Tipo:** SSH Usuário com chave privada
   - **ID:** Identificador/nome para a credencial (ex.: `github-ssh-key`)
   - **Usuário:** `git` (usuário padrão para acesso SSH ao GitHub)
   - **Chave Privada:** Insira a chave privada gerada anteriormente.

3. Clique em **OK** para salvar.

---

## 3. Instalando o plugin Docker no Jenkins

Instale o plugin Docker no Jenkins para gerenciar os contêineres:

1. No Jenkins, vá para o **Painel de Controle** e clique em **Gerenciar Jenkins**.
2. Selecione **Gerenciar Plugins**.
3. Na aba **Disponíveis**, pesquise por "Docker".
4. Marque o **Docker Plugin** e clique em **Instalar sem reiniciar** ou **Instalar e Reiniciar**, se necessário.

---

## 4. Configuração da Nuvem Docker

Agora, configure o Jenkins para usar o Docker como uma nuvem:

1. No **Painel de Controle**, vá para **Gerenciar Jenkins**.
2. Selecione **Configurar o Sistema** e role até encontrar a seção **Cloud**.
3. Clique em **Adicionar uma nova cloud** e selecione **Docker**.
4. Na nova configuração de Docker, preencha os seguintes campos:
   - **Docker Host URI:** `tcp://IP_DO_SERVIDOR:PORTA_DAEMON`
   - **Habilitar:** Marque essa opção.
   
5. Expanda os detalhes e configure conforme necessário. Quando terminar, clique em **Testar Conexão** para garantir que o Jenkins consegue se conectar ao Docker.

6. Após a configuração, clique em **Salvar**.

---

## 5. Criando o primeiro Job no Jenkins

Agora vamos criar um novo Job no Jenkins:

1. No **Painel de Controle**, clique em **Nova Tarefa**.
2. Dê um nome ao Job e selecione **Construir um projeto de software de estilo livre**.
3. Clique em **OK** para prosseguir.

### 5.1 Configurando o Job

Após a criação do Job, configure-o conforme abaixo:

#### 5.1.1 Gerenciamento de Código Fonte

- **Gerenciamento de código fonte:** Selecione **Git**.
- **URL do repositório:** Preencha com o link SSH do seu repositório GitHub.
- **Credenciais:** Selecione as credenciais SSH cadastradas anteriormente.

#### 5.1.2 Gatilho de Disparo

Em **Gatilho de disparo para construções**:
- Marque **Gatilho de gancho do GitHub para polling de SCM**.
- No campo **Agenda**, insira: `* * * * *`.

> Esse padrão de crontab faz com que o Jenkins verifique o GitHub a cada minuto em busca de alterações no repositório. Para mais detalhes sobre o crontab, consulte [este guia](https://guialinux.uniriotec.br/crontab/).

#### 5.1.3 Configuração do Build

No ambiente **Build**, faça as seguintes configurações:

1. Marque a opção **Excluir o espaço de trabalho antes do início da compilação** para garantir que arquivos antigos não sejam reutilizados.
   
2. Adicione um **Executar Shell** para verificar o Dockerfile com Hadolint (um linter para Docker):

```bash
docker run --rm -i hadolint/hadolint < Dockerfile
```

Isso verifica se o Dockerfile segue as convenções recomendadas.

3. Adicione um passo de **Construir/publicar imagem Docker**:
   - **Diretório para Dockerfile:** Caminho onde o Dockerfile está localizado.
   - **Cloud:** Selecione a Cloud Docker configurada anteriormente.
   - **Imagem:** Defina o nome da imagem no formato `usuarioDockerHub/nomeImagem`.

4. Clique em **Salvar** para concluir a configuração do Job.

---
