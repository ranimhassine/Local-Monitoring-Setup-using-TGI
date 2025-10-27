
<body>
  <h1>Setting Up TIG Stack (Telegraf, InfluxDB, Grafana) on Ubuntu</h1>
  <div class="section">
    <h2>Step 1: Update Your System</h2>
    <pre><code>sudo apt update && sudo apt upgrade -y</code></pre>
  </div>

  <div class="section">
    <h2>Step 2: Install InfluxDB</h2>
    <p>Add InfluxData repository key and list:</p>
    <pre><code>curl -sL -O https://repos.influxdata.com/influxdata-archive.key
sudo gpg --dearmor -o /etc/apt/keyrings/influxdata-archive.gpg influxdata-archive.key
echo "deb [signed-by=/etc/apt/keyrings/influxdata-archive.gpg] https://repos.influxdata.com/debian stable main" | sudo tee /etc/apt/sources.list.d/influxdata.list</code></pre>
    <p>Update and install InfluxDB:</p>
    <pre><code>sudo apt update && sudo apt install influxdb2 -y
sudo systemctl start influxdb
sudo systemctl enable influxdb</code></pre>
  </div>

  <div class="section">
    <h2>Step 3: Set Up InfluxDB</h2>
    <p>Visit <code>http://localhost:8086</code> in a browser and complete setup. Or use CLI:</p>
    <pre><code>influx setup --username your_username --password your_password --org your_org --bucket monitoring --force</code></pre>
  </div>
  <p align="center">
  <img src="Influxdb.png" alt="Setup commands" width="600">
</p>
<p align="center">
  <img src="Influxdb-dashboard.png" alt="influxdb" width="600">
</p>


  <div class="section">
    <h2>Step 4: Install Telegraf</h2>
    <pre><code>sudo apt install telegraf -y
sudo systemctl start telegraf
sudo systemctl enable telegraf</code></pre>
  </div>

  <div class="section">
    <h2>Step 5: Configure Telegraf</h2>
    <p>Edit configuration:</p>
    <pre><code>sudo nano /etc/telegraf/telegraf.conf</code></pre>
    <p>Set the following values in <code>[[outputs.influxdb_v2]]</code>:</p>
    <pre><code>urls = ["http://127.0.0.1:8086"]
token = "YOUR_TOKEN"
organization = "your_org"
bucket = "monitoring"</code></pre>
  </div>
  <p align="center">
  <img src="telegraf-config.png" alt="telegraf configurations file" width="600">
</p>

  <div class="section">
    <h2>Step 6: Install Grafana</h2>
    <pre><code>sudo apt install -y apt-transport-https software-properties-common wget
wget -q -O - https://apt.grafana.com/gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/grafana.gpg
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install grafana -y
sudo systemctl start grafana-server
sudo systemctl enable grafana-server</code></pre>
  </div>

  <div class="section">
    <h2>Access Grafana</h2>
    <p>Visit <code>http://localhost:3000</code>. Default login: <code>admin/admin</code>.</p>
  </div>
</body>
</html>
<p align="center">
  <img src="Grafana.png" alt="Grafana" width="600">
</p>
