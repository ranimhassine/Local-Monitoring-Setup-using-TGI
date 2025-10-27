# Local-Monitoring-Setup-using-TGI
This repository contains documentation and configuration for a local monitoring setup using InfluxDB and a dashboard (likely Grafana).
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Setting Up TIG (Telegraf • InfluxDB • Grafana) on Ubuntu</title>
  <style>
    :root{--bg:#0f1724;--card:#0b1220;--accent:#4f46e5;--muted:#9aa7bf;--mono: Menlo, Monaco, Consolas, "Courier New", monospace}
    html,body{height:100%;margin:0;font-family:Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;background:linear-gradient(180deg,#071029, #081426);color:#e6eef8}
    .wrap{max-width:980px;margin:28px auto;padding:28px}
    header{display:flex;align-items:center;gap:16px}
    h1{margin:0;font-size:1.6rem}
    p.lead{color:var(--muted);margin:6px 0 20px}
    .card{background:rgba(255,255,255,0.02);border-radius:12px;padding:18px;box-shadow:0 6px 18px rgba(2,6,23,0.6);margin-bottom:18px}
    pre{background:#021226;padding:12px;border-radius:8px;overflow:auto;font-family:var(--mono);font-size:0.95rem;color:#cfe8ff}
    code{font-family:var(--mono);font-size:0.95rem}
    .cmd{display:flex;gap:8px;align-items:center}
    button.copy{background:transparent;border:1px solid rgba(255,255,255,0.06);color:var(--muted);padding:6px 10px;border-radius:8px;cursor:pointer}
    .note{background:rgba(79,70,229,0.08);border-left:4px solid var(--accent);padding:10px;border-radius:6px;color:var(--muted);margin:10px 0}
    table{width:100%;border-collapse:collapse;color:#dbeafe}
    td,th{padding:8px;border-bottom:1px solid rgba(255,255,255,0.03);text-align:left}
    footer{color:var(--muted);font-size:0.85rem;margin-top:12px}
    .placeholder{color:#ffa;opacity:0.95}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <svg width="44" height="44" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><rect x="2" y="2" width="20" height="20" rx="4" fill="#111827" stroke="#4f46e5" stroke-width="0.8"></rect><path d="M7 8h10M7 12h10M7 16h6" stroke="#9fb8ff" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
      <div>
        <h1>Setting Up a Monitoring &amp; Dashboarding System — TIG on Ubuntu</h1>
        <p class="lead">Telegraf ➜ InfluxDB 2.x ➜ Grafana — single-machine setup for development or small-scale monitoring (Ubuntu 22.04 / 24.04).</p>
      </div>
    </header>

    <section class="card">
      <h2>Prerequisites</h2>
      <ul>
        <li>Ubuntu machine (22.04 or 24.04 recommended) with a user that has <code>sudo</code> privileges.</li>
        <li>Internet access to download packages.</li>
        <li>Ports used: <code>8086</code> (InfluxDB), <code>3000</code> (Grafana).</li>
      </ul>
    </section>

    <section class="card">
      <h2>Step 1 — Update your system</h2>
      <div class="cmd">
        <pre id="cmd-update">sudo apt update && sudo apt upgrade -y</pre>
        <button class="copy" data-target="cmd-update">Copy</button>
      </div>
    </section>

    <section class="card">
      <h2>Step 2 — Install InfluxDB 2.x</h2>
      <p class="note">This uses the InfluxData repo. If you prefer InfluxDB 3.x, review official docs and update repository URLs accordingly.</p>

      <h3>Add repository key &amp; list</h3>
      <div class="cmd">
        <pre id="cmd-influx-repo">curl --silent --location -O https://repos.influxdata.com/influxdata-archive.key

# verify fingerprint then add to apt keyrings
gpg --show-keys --with-fingerprint --with-colons ./influxdata-archive.key 2>&1 | grep -q '^fpr:\+24C975CBA61A024EE1B631787C3D57159FC2F927:$' && cat influxdata-archive.key | gpg --dearmor | sudo tee /etc/apt/keyrings/influxdata-archive.gpg &gt;/dev/null

echo 'deb [signed-by=/etc/apt/keyrings/influxdata-archive.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list</pre>
        <button class="copy" data-target="cmd-influx-repo">Copy</button>
      </div>

      <h3>Install</h3>
      <div class="cmd">
        <pre id="cmd-influx-install">sudo apt-get update && sudo apt-get install influxdb2 -y

sudo systemctl start influxdb
sudo systemctl enable influxdb
sudo systemctl status influxdb</pre>
        <button class="copy" data-target="cmd-influx-install">Copy</button>
      </div>

      <p class="note">You should see <code>active (running)</code> when checking the service status.</p>
    </section>

    <section class="card">
      <h2>Step 3 — Set up InfluxDB</h2>
      <p>Open <a href="#" onclick="alert('Open http://localhost:8086 in your browser (or replace localhost with your server IP)');return false;">http://localhost:8086</a> and complete the web UI setup to create a username, password, organization, bucket (e.g., <code>monitoring</code>), and copy the generated API token.</p>

      <h3>Or use the CLI</h3>
      <div class="cmd">
        <pre id="cmd-influx-setup">influx setup --username your_username --password your_password --org "MyOrg" --bucket monitoring --force</pre>
        <button class="copy" data-target="cmd-influx-setup">Copy</button>
      </div>

      <p class="note">Save the API token somewhere safe — you'll need it for Telegraf and Grafana.</p>
    </section>

    <section class="card">
      <h2>Step 4 — Install Telegraf</h2>
      <div class="cmd">
        <pre id="cmd-telegraf-install">sudo apt-get install telegraf -y

sudo systemctl start telegraf
sudo systemctl enable telegraf
sudo systemctl status telegraf</pre>
        <button class="copy" data-target="cmd-telegraf-install">Copy</button>
      </div>

      <p>Telegraf will be installed from the repository added earlier.</p>
    </section>

    <section class="card">
      <h2>Step 5 — Configure Telegraf</h2>
      <p>Edit the Telegraf config:</p>
      <div class="cmd">
        <pre id="cmd-telegraf-edit">sudo nano /etc/telegraf/telegraf.conf</pre>
        <button class="copy" data-target="cmd-telegraf-edit">Copy</button>
      </div>

      <p>In the <code>[[outputs.influxdb_v2]]</code> section, set these values (replace placeholders):</p>
      <pre id="cfg-out">[[outputs.influxdb_v2]]
  urls = ["http://127.0.0.1:8086"]
  token = "YOUR_INFLUXDB_API_TOKEN"    # &lt;-- replace
  organization = "MyOrg"              # &lt;-- replace
  bucket = "monitoring"               # &lt;-- replace</pre>
      <button class="copy" data-target="cfg-out">Copy</button>

      <p>Enable the system input plugins (uncomment these blocks or add them):</p>
      <pre id="cfg-inputs">[[inputs.cpu]]
[[inputs.disk]]
[[inputs.diskio]]
[[inputs.mem]]
[[inputs.net]]
[[inputs.processes]]
[[inputs.swap]]
[[inputs.system]]</pre>
      <button class="copy" data-target="cfg-inputs">Copy</button>

      <p class="note">After editing, restart Telegraf:</p>
      <div class="cmd">
        <pre id="cmd-telegraf-restart">sudo systemctl restart telegraf</pre>
        <button class="copy" data-target="cmd-telegraf-restart">Copy</button>
      </div>

      <p>Default collection interval is typically 10s; confirm in the config if you need a different frequency.</p>
    </section>

    <section class="card">
      <h2>Step 6 — Install Grafana</h2>
      <h3>Prerequisites &amp; repository</h3>
      <div class="cmd">
        <pre id="cmd-grafana-prereq">sudo apt-get install -y apt-transport-https software-properties-common wget

sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg &gt;/dev/null

echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

sudo apt-get update
sudo apt-get install grafana -y</pre>
        <button class="copy" data-target="cmd-grafana-prereq">Copy</button>
      </div>

      <div class="cmd">
        <pre id="cmd-grafana-start">sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server</pre>
        <button class="copy" data-target="cmd-grafana-start">Copy</button>
      </div>

      <p>Open <code>http://localhost:3000</code> and log in (default <code>admin</code>/<code>admin</code>). Change the password at first login.</p>
    </section>

    <section class="card">
      <h2>Step 7 — Connect Grafana to InfluxDB</h2>
      <ol>
        <li>In Grafana UI, go to <strong>Configuration &gt; Data Sources &gt; Add data source</strong>.</li>
        <li>Search for <code>InfluxDB</code> (InfluxDB 2.x plugin).</li>
        <li>Set URL to <code>http://127.0.0.1:8086</code>, set <code>Organization</code>, and paste the API token from InfluxDB setup. Choose the bucket (e.g., <code>monitoring</code>).</li>
        <li>Save &amp; test — Grafana should connect successfully.</li>
      </ol>
    </section>

    <section class="card">
      <h2>Step 8 — Import or build dashboards</h2>
      <p>You can create dashboards manually or import community dashboards. Example common metrics to show:</p>
      <table>
        <thead><tr><th>Metric</th><th>Measurement / Field</th></tr></thead>
        <tbody>
          <tr><td>CPU</td><td>usage_user / usage_system / usage_idle</td></tr>
          <tr><td>Memory</td><td>used_percent / total</td></tr>
          <tr><td>Disk</td><td>used_percent / free</td></tr>
          <tr><td>Network</td><td>bytes_recv / bytes_sent</td></tr>
        </tbody>
      </table>

      <p class="note">Grafana also supports provisioning dashboards from JSON, which is handy for reproducible deployments.</p>
    </section>

    <section class="card">
      <h2>Troubleshooting &amp; Tips</h2>
      <ul>
        <li>If you see permission or token errors — verify the InfluxDB token has correct bucket/org permissions.</li>
        <li>Check logs: <code>sudo journalctl -u influxdb -f</code>, <code>sudo journalctl -u telegraf -f</code>, <code>sudo journalctl -u grafana-server -f</code>.</li>
        <li>If metrics don't appear, ensure Telegraf is writing to the same bucket that Grafana is querying.</li>
        <li>For remote collection, adjust <code>urls</code> and firewall rules to allow access between hosts — secure traffic with TLS where possible.</li>
      </ul>
    </section>

    <section class="card">
      <h2>Quick reference — Placeholders to replace</h2>
      <pre class="placeholder">YOUR_INFLUXDB_API_TOKEN
MyOrg
monitoring</pre>
    </section>

    <footer>
      <div>Created for: single-machine TIG stack (development / small-scale monitoring). Review official docs for production hardening (TLS, RBAC, backups, retention policies).</div>
    </footer>
  </div>

  <script>
    // copy-to-clipboard for code blocks
    document.querySelectorAll('button.copy').forEach(btn => {
      btn.addEventListener('click', () => {
        const id = btn.getAttribute('data-target');
        const el = document.getElementById(id);
        if(!el) return;
        navigator.clipboard.writeText(el.innerText).then(()=>{
          const old = btn.innerText;
          btn.innerText = 'Copied!';
          setTimeout(()=> btn.innerText = old,1500);
        }).catch(()=> alert('Copy failed — select and copy manually.'));
      });
    });
  </script>
</body>
</html>
