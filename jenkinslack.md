## Integração do Jenkins com o Slack para Envio de Notificações

### 1. Configurando o Slack

1. No Slack, acesse a página de **Aplicativos** e instale a extensão **Jenkins CI**.
2. Após a instalação, configure o **canal** ao qual deseja que as notificações sejam enviadas.
3. Gere o **Token de Autenticação** fornecido pelo Slack, que será necessário para configurar o Jenkins.

---

### 2. Instalando o Plugin Slack Notification no Jenkins

1. No Jenkins, acesse o **Painel de Controle** e vá para **Gerenciar Jenkins**.
2. Selecione **Gerenciar Plugins**.
3. Na aba **Disponíveis**, pesquise por **Slack Notification** e instale o plugin.

---

### 3. Configurando o Slack no Jenkins

Após a instalação do plugin, siga os passos abaixo para configurar o Jenkins para enviar notificações ao Slack:

1. No Jenkins, vá até **Gerenciar Jenkins** e selecione **Configurar o Sistema**.
2. Localize a seção **Slack** e preencha os campos:

   - **Workspace:** A URL do seu workspace Slack (ex.: `https://seu-workspace.slack.com`).
   
3. Clique em **Adicionar** na seção de credenciais e selecione **Jenkins**.
   
4. Configure as credenciais:
   - **Tipo:** Selecione **Secret Text**.
   - **Escopo:** Escolha **Global**.
   - **Senha:** Insira o **Token gerado pelo Slack** após a instalação da extensão Jenkins CI.
   - **ID:** Defina um identificador para as credenciais (ex.: `slack-jenkins-token`).

5. Salve as credenciais e selecione o token criado na configuração do Slack no Jenkins.

---

### 4. Configurando o Canal e Testando a Conexão

1. No campo **Default channel / member id**, insira o nome do **canal** no Slack para onde deseja enviar as notificações (ex.: `#canal-jenkins`).
2. Após configurar o canal, clique em **Testar Conexão** para garantir que o Jenkins está se comunicando corretamente com o Slack.

---
