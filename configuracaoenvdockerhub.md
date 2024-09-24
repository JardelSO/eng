## 1. Instalando o Plugin "Config File Provider" no Jenkins

Instale o plugin **Config File Provider** para gerenciar arquivos de configuração da sua aplicação, como conexões com o banco de dados de produção e desenvolvimento:

1. No Jenkins, vá para o **Painel de Controle** e clique em **Gerenciar Jenkins**.
2. Selecione **Gerenciar Plugins**.
3. Na aba **Disponíveis**, pesquise por **Config File Provider**.
4. Marque o **Config File Provider** e clique em **Instalar sem reiniciar**.

---

## 2. Configurando Arquivos de Conexão com Banco de Dados

Agora vamos criar arquivos de configuração para os ambientes de desenvolvimento e produção:

1. No **Painel de Controle**, clique em **Gerenciar Jenkins** novamente.
2. Selecione **Config File Management**.
3. Clique em **Novo Arquivo** e selecione a opção **Arquivo customizado**.
4. Preencha os campos:
   - **Nome:** Um identificador para o arquivo (ex.: `db-config-dev`).
   - **Conteúdo:** Insira o conteúdo de conexão com o banco de dados de desenvolvimento (ex.: variáveis de ambiente ou string de conexão).

5. Anote o **ID** do arquivo criado.

6. Repita o processo para o arquivo de configuração de produção, alterando o nome e o conteúdo de acordo com o ambiente.

---

## 3. Utilizando os Arquivos de Configuração no Job

Agora, iremos aplicar essas configurações no Job criado anteriormente:

1. No Job de codinome "principal", acesse as configurações.
2. Marque a opção **Provide Configuration files**.
3. Selecione o arquivo de desenvolvimento criado anteriormente na opção **File**.
4. Defina o local onde a aplicação espera esse arquivo de configuração (ex.: no diretório raiz `.env`) no campo **Target**.

---

## 4. Adicionando um Shell Script para Subir a Aplicação

Nos passos de construção do Job, adicione um **Execute Shell** para subir e testar sua aplicação. Por exemplo, para uma aplicação Python, o script seria:

```bash
# Subindo o container de teste
docker run -d -p 82:8000 -v /var/run/mysqld/mysqld.sock:/var/run/mysqld/mysqld.sock -v /var/lib/jenkins/workspace/jenkins-todo-list-principal/to_do/.env:/usr/src/app/to_do/.env --name=todo-list-teste django_todolist_image_build

# Testando a imagem
docker exec -i todo-list-teste python manage.py test --keep
exit_code=$?

# Derrubando o container antigo
docker rm -f todo-list-teste

if [ $exit_code -ne 0 ]; then
    exit 1
fi
```

Esse script irá rodar testes no container, remover o container antigo e, se os testes falharem, o job será marcado como falho.

---

## 5. Configurando Publicação Automática no DockerHub

Agora vamos configurar a publicação automática da imagem Docker no DockerHub. Primeiro, instale o plugin **Parameterized Trigger**:

1. No **Painel de Controle**, clique em **Gerenciar Jenkins**.
2. Selecione **Gerenciar Plugins**.
3. Na aba **Disponíveis**, pesquise por **Parameterized Trigger**.
4. Marque o **Parameterized Trigger** e clique em **Instalar sem reiniciar**.

### 5.1 Configurando Parâmetros para Publicação

1. No Job "principal", vá para as configurações.
2. Marque a opção **Esta construção é parametrizada**.
3. Adicione um **Parâmetro String**:
   - **Nome:** `image`
   - **Valor padrão:** `usuarioDockerHub/NomeDaImagem`.

4. Adicione outro **Parâmetro String** para definir o host Docker:
   - **Nome:** `DOCKER_HOST`
   - **Valor padrão:** `tcp://127.0.0.1:2376`.

### 5.2 Publicando a Imagem no DockerHub

1. Em **Construir/publicar imagem Docker**, marque a opção **Push Image**.
2. Clique em **Adicionar** e escolha **Jenkins** para adicionar suas credenciais DockerHub:
   - **Tipo:** Usuário e senha.
   - **Usuário:** Seu nome de usuário do DockerHub.
   - **Senha:** Sua senha do DockerHub.

Após isso, o Jenkins estará configurado para publicar automaticamente a imagem no DockerHub após a execução bem-sucedida do Job.

--- 
