## 1. Concedendo permissões para gerenciamento do Docker

Acesse o servidor via SSH e execute os seguintes comandos para adicionar as permissões necessárias para que o Jenkins e o seu usuário possam gerenciar o Docker:

```bash
sudo usermod -aG docker $USER
sudo usermod -aG docker jenkins
```

Isso permite que tanto o seu usuário quanto o Jenkins utilizem o Docker sem precisar de permissões adicionais.

---

## 2. Adicionando credenciais SSH no Jenkins para integração com GitHub

Agora, precisamos configurar o Jenkins para interagir com o GitHub via SSH. Siga os passos abaixo:

1. Acesse o Jenkins e navegue até:  
   **Gerenciar Jenkins** → **Credenciais** → **System** → **Credenciais globais** → **Adicionar Credenciais**.

2. Preencha os campos da seguinte forma:
   - **Tipo:** SSH Usuário com chave privada
   - **ID:** Identificador/nome para a credencial
   - **Usuário:** `git` (o usuário padrão para acesso via SSH ao GitHub)
   - **Chave Privada:** Insira a chave privada gerada anteriormente

3. Clique em **Criar**.

---

## 3. Criando o primeiro Job no Jenkins

Agora, vamos criar o primeiro Job no Jenkins:

1. Volte para o **Painel de Controle** do Jenkins e clique em **Nova Tarefa**.
2. Insira o nome do job no campo **Nome do item**.
3. Selecione a opção **Construir um projeto de software de estilo livre**.
4. Clique em **OK**.

### 3.1 Configurando o Job

Após criar o Job, faça as seguintes configurações:

#### 3.1.1 Gerenciamento de Código Fonte
- **Gerenciamento de código fonte:** Selecione **Git**.
- **URL do repositório:** Preencha com o link de acesso SSH do seu repositório GitHub.
- **Credenciais:** Selecione as credenciais SSH que foram cadastradas anteriormente.

#### 3.1.2 Gatilho de Disparo

No ambiente de **Gatilho de disparo para construções**:
- Marque a opção **Gatilho de gancho do GitHub para pesquisa do GITScm**.
- No campo **Agenda**, preencha com `* * * * *`.

> Esse padrão de sintaxe é do `crontab` e indica que o Jenkins irá consultar o GitHub a cada 1 minuto para verificar alterações no repositório. Para mais informações, consulte [este guia sobre crontab](https://guialinux.uniriotec.br/crontab/).

#### 3.1.3 Configuração do Build

No ambiente de **Build**, marque a opção **Excluir o espaço de trabalho antes do início da compilação**. Isso evita que arquivos antigos ou indesejados sejam utilizados na construção do projeto.

---

### 4. Conclusão

Agora o Jenkins está configurado para:
- Monitorar seu repositório GitHub a cada minuto.
- Excluir o espaço de trabalho antes de cada build, garantindo uma construção limpa.
```
