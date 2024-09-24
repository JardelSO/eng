## Criar uma Nova Tarefa no Jenkins para Deploy de Produção

### 1. Criando a Nova Tarefa

1. No **Painel de Controle** do Jenkins, clique em **Nova Tarefa**.
2. Dê um nome ao Job e selecione a opção **Construir um projeto de software de estilo livre**.
3. Clique em **OK** para prosseguir.

### 2. Configuração de Parâmetros

Para facilitar a configuração do Job e evitar a necessidade de configurar a imagem e o Docker Host manualmente, siga os passos abaixo:

1. Marque a opção **Esta construção é parametrizada**.
2. Adicione dois parâmetros do tipo **String**:
   
   - **Parâmetro 1:**
     - **Nome:** `image`
     - **Valor padrão:** Deixe vazio, pois este parâmetro será preenchido com o nome da imagem gerada pelo Job anterior (pipeline de build da imagem Docker).
   
   - **Parâmetro 2:**
     - **Nome:** `DOCKER_HOST`
     - **Valor padrão:** `tcp://127.0.0.1:2376` (ou insira o endereço do host Docker que você está utilizando).

### 3. Configuração de Arquivos de Configuração

1. Marque a opção **Provide Configuration files**.
2. Selecione o arquivo de produção que você criou anteriormente na opção **File**.
3. No campo **Target**, defina o caminho onde sua aplicação espera o arquivo de configuração (por exemplo, no diretório raiz: `.env`).

### 4. Script de Construção (Execute Shell)

Nos passos de construção do Job, adicione um **Execute Shell** para subir e testar a aplicação. Abaixo está um exemplo para uma aplicação Python utilizando Docker:

```bash
docker run -d -p 80:8000 \
-v /var/run/mysqld/mysqld.sock:/var/run/mysqld/mysqld.sock \
-v /var/lib/jenkins/workspace/todo-list-producao/.env:/usr/src/app/to_do/.env \
--name=django-todolist-prod $image:latest \
|| { 
    # Em caso de falha, remover container antigo e tentar novamente
    docker rm -f django-todolist-prod
    docker run -d -p 80:8000 \
    -v /var/run/mysqld/mysqld.sock:/var/run/mysqld/mysqld.sock \
    -v /var/lib/jenkins/workspace/todo-list-producao/.env:/usr/src/app/to_do/.env \
    --name=django-todolist-prod $image:latest
}
```

Este script tenta iniciar o container da aplicação com a nova imagem. Se o container falhar ao iniciar, ele remove o container antigo e tenta novamente.

### 5. Configuração de Ações de Pós-Build

1. Na seção **Ações de Pós-Build**, adicione a ação **Slack Notifications** para enviar notificações sobre o status da build:
   - Marque a opção **Notify Success** para notificar quando a build for bem-sucedida.
   - Marque a opção **Notify Every Failure** para ser notificado sempre que a build falhar.

---
