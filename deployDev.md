## Criando um Job de Deploy Automático no Ambiente de Desenvolvimento

### 1. Criar um Novo Pipeline no Jenkins

1. No **Painel de Controle** do Jenkins, clique em **Nova Tarefa**.
2. Selecione **Pipeline** e insira um nome para o Job.
3. Clique em **OK** para prosseguir.

### 2. Configuração de Parâmetros

Para facilitar a configuração do Job e evitar a necessidade de configurar a imagem e o Docker Host manualmente, faremos o seguinte:

1. Marque a opção **Esta construção é parametrizada**.
2. Adicione dois parâmetros do tipo **String**:
   
   - **Parâmetro 1:**
     - **Nome:** `image`
     - **Valor padrão:** Deixe vazio, pois o parâmetro será preenchido com o nome do Job anterior (pipeline de build da imagem).
   
   - **Parâmetro 2:**
     - **Nome:** `DOCKER_HOST`
     - **Valor padrão:** `tcp://127.0.0.1:2376` (ou o endereço do host Docker que você está usando).

### 3. Script do Pipeline em Groovy

Na seção de **Pipeline Script**, usaremos a linguagem Groovy para definir as etapas do deploy. Para mais informações sobre a sintaxe Groovy, consulte [o site oficial](https://groovy-lang.org/).

#### Exemplo de Pipeline:

```groovy
pipeline {
    environment {
        dockerImage = "${image}"
    }
    agent any

    stages {
        stage('Carregando o ENV de desenvolvimento') {
            steps {
                configFileProvider([configFile(fileId: '<id do seu arquivo de desenvolvimento>', variable: 'env')]) {
                    sh 'cat $env > .env'
                }
            }
        }
        stage('Derrubando o container antigo') {
            steps {
                script {
                    try {
                        sh 'docker rm -f django-todolist-dev'
                    } catch (Exception e) {
                        echo "Container antigo não encontrado ou erro ao removê-lo: $e"
                    }
                }
            }
        }
        stage('Subindo o container novo') {
            steps {
                script {
                    try {
                        sh '''
                        docker run -d \
                        -p 81:8000 \
                        -v /var/run/mysqld/mysqld.sock:/var/run/mysqld/mysqld.sock \
                        -v /var/lib/jenkins/workspace/jenkins-todo-list-desenvolvimento/.env:/usr/src/app/to_do/.env \
                        --name=django-todolist-dev ${dockerImage}:latest
                        '''
                    } catch (Exception e) {
                        slackSend (
                            color: 'danger', 
                            message: "[ FALHA ] Não foi possível subir o container - ${BUILD_URL} em ${currentBuild.duration}s", 
                            tokenCredentialId: 'slack-token'
                        )
                        echo "Erro ao subir o container: $e"
                        currentBuild.result = 'ABORTED'
                        error('Erro ao subir o container')
                    }
                }
            }
        }
        stage('Notificando o usuário') {
            steps {
                slackSend (
                    color: 'good', 
                    message: "[ SUCESSO ] O novo build está disponível em: http://192.168.33.10:81/", 
                    tokenCredentialId: 'slack-token'
                )
            }
        }
    }
}
```

### 4. Explicação dos Estágios do Pipeline

- **Carregando o ENV de desenvolvimento:**  
  Este estágio utiliza o **Config File Provider** para carregar o arquivo de configuração de ambiente (por exemplo, `.env`) para o container.
  
- **Derrubando o container antigo:**  
  Antes de subir o novo container, o pipeline tenta remover o container antigo (`django-todolist-dev`). Se o container não existir ou ocorrer um erro, ele será ignorado, mas registrado no log.

- **Subindo o container novo:**  
  Neste estágio, o novo container é iniciado utilizando a imagem Docker gerada no job anterior. Ele monta os volumes necessários, como o arquivo `.env` e o socket do MySQL, e mapeia a porta 81 para a aplicação.

- **Notificando o usuário:**  
  Após a conclusão bem-sucedida do deploy, o Jenkins envia uma notificação para o Slack, informando o sucesso e o link para acessar a aplicação.
