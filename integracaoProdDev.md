## Configurando Passagem de Parâmetros entre Jobs no Jenkins

### 1. Configurando o Job Principal

No Job principal, você irá configurar a passagem de parâmetros para outros Jobs. Siga os passos abaixo:

1. Acesse as configurações do **Job Principal**.
2. Na seção **Ações de Pós-Build**, selecione a opção **Trigger parameterized build on other projects**.
3. Preencha os campos conforme abaixo:
   - **Projects to build:** Selecione o **Job de Desenvolvimento**.
4. Clique em **Adicionar parâmetro** do tipo **Predefinido** e preencha:
   - **Parameters:** `image=$(image)`
5. Clique em **Salvar** para aplicar as mudanças.

### 2. Ajustando o Job de Desenvolvimento

Agora, você precisará ajustar o Job de Desenvolvimento para chamar o Job de Produção. Acesse as configurações desse Job e adicione o seguinte script no pipeline:

```groovy
pipeline {
    environment {
        dockerImage = "${image}"
    }
    agent any

    stages {
        stage('Carregando o ENV de desenvolvimento') {
            steps {
                configFileProvider([configFile(fileId: '2ed9697c-45fc-4713-a131-53bdbeea2ae6', variable: 'env')]) {
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
                        sh "echo $e"
                    }
                }
            }
        }
        stage('Subindo o container novo') {
            steps {
                script {
                    try {
                        sh 'docker run -d -p 81:8000 -v /var/run/mysqld/mysqld.sock:/var/run/mysqld/mysqld.sock -v /var/lib/jenkins/workspace/todo-list-desenvolvimento/.env:/usr/src/app/to_do/.env --name=django-todolist-dev ' + dockerImage + ':latest'
                    } catch (Exception e) {
                        slackSend(color: 'error', message: "[ FALHA ] Não foi possível subir o container - ${BUILD_URL} em ${currentBuild.duration}s", tokenCredentialId: 'slack-token')
                        sh "echo $e"
                        currentBuild.result = 'ABORTED'
                        error('Erro')
                    }
                }
            }
        }
        stage('Notificando o usuário') {
            steps {
                slackSend(color: 'good', message: '[ Sucesso ] O novo build está disponível em: http://192.168.33.10:81/', tokenCredentialId: 'slack-token')
            }
        }
        stage('Fazer o deploy em produção?') {
            steps {
                script {
                    slackSend(color: 'warning', message: "Para aplicar a mudança em produção, acesse [Janela de 10 minutos]: ${JOB_URL}", tokenCredentialId: 'slack-token')
                    timeout(time: 10, unit: 'MINUTES') {
                        input(id: "Deploy Gate", message: "Deploy em produção?", ok: 'Deploy')
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    try {
                        build job: 'todo-list-producao', parameters: [[$class: 'StringParameterValue', name: 'image', value: dockerImage]]
                    } catch (Exception e) {
                        slackSend(color: 'error', message: "[ FALHA ] Não foi possível subir o container em produção - ${BUILD_URL}", tokenCredentialId: 'slack-token')
                        sh "echo $e"
                        currentBuild.result = 'ABORTED'
                        error('Erro')
                    }
                }
            }
        }
    }
}
```

### 3. Descrição do Script

Este script realiza as seguintes operações:

- **Carregando o ENV de Desenvolvimento:** Carrega as variáveis de ambiente necessárias.
- **Derrubando o Container Antigo:** Remove o container existente, se houver.
- **Subindo o Container Novo:** Inicia um novo container a partir da imagem especificada.
- **Notificando o Usuário:** Envia uma notificação de sucesso pelo Slack com a URL de acesso ao build.
- **Fazendo o Deploy em Produção:** Solicita ao usuário a confirmação para o deploy em produção, com um limite de 10 minutos para a resposta.
- **Deploy:** Se o usuário aceitar, chama o Job de Produção, passando a imagem como parâmetro.
