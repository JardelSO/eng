## Integração entre Jenkins, Prometheus e Grafana

Para configurar a integração entre Jenkins, Prometheus e Grafana, siga os passos abaixo:

### 1. Configuração do Jenkins

1. **Acesse o Painel de Controle do Jenkins:**
   - No seu navegador, abra o Jenkins e clique em **Gerenciar Jenkins**.

2. **Instale o Plugin Prometheus Metrics:**
   - Selecione **Gerenciar Plugins**.
   - Vá até a aba **Disponíveis** e pesquise por **Prometheus metrics**.
   - Marque o plugin **Prometheus metrics** e clique em **Instalar sem reiniciar**.

3. **Verifique as Métricas:**
   - Após a instalação, o Jenkins estará expondo as métricas em: 
     ```
     http://<Public-IP>:8080/prometheus
     ```

### 2. Configuração do Prometheus

1. **Suba uma Imagem do Prometheus:**
   - Execute o seguinte comando no terminal:
     ```bash
     docker run -d -p 9090:9090 --name prometheus -v /home/vagrant/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
     ```

2. **Configuração do Arquivo `prometheus.yml`:**
   - O conteúdo do arquivo `prometheus.yml` é crucial para definir onde o Prometheus irá buscar os dados. Aqui está um exemplo básico:
     ```yaml
     global:
       scrape_interval: 15s

     scrape_configs:
       - job_name: "jenkins"
         metrics_path: /prometheus/
         static_configs:
           - targets: ["192.168.33.10:8080"]
     ```

### 3. Configuração do Grafana

1. **Suba uma Imagem do Grafana:**
   - Execute o seguinte comando:
     ```bash
     docker run -d -p 3000:3000 --name=grafana -e "GF_SECURITY_ADMIN_PASSWORD=admin" grafana/grafana
     ```

2. **Acessando o Grafana:**
   - O usuário e a senha padrão são ambos: `admin`.

3. **Adicionar o Prometheus como Fonte de Dados:**
   - No Grafana, vá até **Configurações** (Settings).
   - Selecione **Data Sources** (Fontes de Dados) e clique em **Add data source**.
   - Escolha **Prometheus** e preencha os campos necessários, incluindo a URL do Prometheus, que será:
     ```
     http://<Public-IP>:9090
     ```

### 4. Criando um Dashboard

- Após adicionar o Prometheus como fonte de dados, você pode construir seu próprio dashboard ou importar um já pronto.
- Para importar dashboards, visite o site oficial do Grafana em: [Grafana Dashboards](https://grafana.com/grafana/dashboards/).
