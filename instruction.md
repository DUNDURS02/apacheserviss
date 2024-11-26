# Apache HTTP/HTTPS Server Instalācijas Instrukcija

## Sistēmas Prasības
- **Operētājsistēma:** Linux (Ubuntu 20.04/22.04, CentOS, RHEL), macOS, Windows (ar WAMP/XAMPP).
- **Procesors:** 64-bit.
- **RAM:** Vismaz 1GB (ieteicams 2GB+).
- **Cietā diska vieta:** Vismaz 2GB brīvas vietas.
- **Tīkla piekļuve:** Internets, ja nepieciešama sertifikātu iegūšana vai attālā piekļuve.

---

## Instalācijas Soļi

 - [ ] **1. Atjaunojiet sistēmas pakotnes**
    ```
    sudo apt update && sudo apt upgrade -y
    ```
    > **Ko šī komanda dara?**
    `sudo apt update`: Atjaunina informāciju par pieejamām programmatūras versijām.
    `sudo apt upgrade -y`: Atjaunina visas instalētās programmas uz jaunākajām versijām.

 - [ ] **2. Instalējiet Apache**
    ```
    sudo apt install apache2 -y
    ```
    > **Ko šī komanda dara?**
    `sudo apt install apache2`: Instalē Apache serveri no Ubuntu programmatūras repozitorijiem.
    `-y`: Automātiski apstiprina instalāciju.

 - [ ] **3. Verificējiet instalāciju**
    - Pārbaudiet, vai Apache darbojas:
    ```
    sudo systemctl status apache2
    ```
    > **Ko šī komanda dara?**
    `sudo systemctl status apache2`: Parāda informāciju par Apache servera darbību (vai tas ir ieslēgts, darbojas bez kļūdām).

### Pārbaudiet serveri pārlūkprogrammā
Atveriet pārlūkprogrammu.
Ierakstiet `http://<server-ip>` vai `http://localhost`.
`localhost`: Norāda, ka jūs pārbaudāt serveri uz tās pašas mašīnas.
Ja viss darbojas, jūs redzēsiet noklusējuma Apache sveiciena lapu - `It works!`.


---

## HTTPS Konfigurācija ar Let's Encrypt (SSL)

- [ ] **1. Instalējiet `certbot`**
    ```
    sudo apt install certbot python3-certbot-apache -y
    ```
    > **Ko šī komanda dara?**
    Instalē certbot un tā paplašinājumu, kas ļauj to izmantot kopā ar Apache.

- [ ] **2. Iegūstiet SSL sertifikātu**
    ```
    sudo certbot --apache
    ```
    - Ievadiet domēna nosaukumu un e-pastu.
    - Pēc konfigurācijas pārbaudiet vietni, izmantojot `https://<server-ip>` vai domēna nosaukumu.
    > **Ko šī komanda dara?**
`sudo certbot --apache`: Izsauc certbot, kas konfigurē SSL sertifikātu jūsu Apache serverim.
Tas jūs lūgs ievadīt jūsu domēna nosaukumu (piemēram, example.com) un e-pastu.

- [ ] **3. Atjauniniet sertifikātu (automātiskais režīms)**
SSL sertifikātiem ir derīguma termiņš (parasti 90 dienas). Ar šo komandu jūs pārbaudāt, vai automātiskā atjaunināšana darbojas:
    ```
    sudo certbot renew --dry-run
    ```

---

## Testēšana un Konfigurācija

- [ ] **1. Pārbaudiet Apache darbību**
    - Pārliecinieties, vai serveris darbojas:
    ```
    curl -I http://localhost
    ```
    > **Ko šī komanda dara?**
`curl -I`: Nosūta pieprasījumu uz serveri un parāda tikai atbildes galvenes. Ja viss darbojas, jūs redzēsiet statusu `HTTP/1.1 200 OK`.

- [ ] **2. Pievienojiet virtuālos hostus**
    - Izveidojiet jaunu konfigurāciju failu:
        ```
        sudo nano /etc/apache2/sites-available/example.com.conf
        ```
    - Iekopējiet zemāk esošo saturu:
        ```
        <VirtualHost *:80>
            ServerName example.com
            ServerAlias www.example.com
            DocumentRoot /var/www/example.com
            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
        ```
    - Aktivizējiet virtuālo hostu:
        ```
        sudo a2ensite example.com.conf
        sudo systemctl reload apache2
        ```

- [ ] **3. Drošības konfigurācija**
    - Ieslēdziet ugunsmūri un atļaujiet HTTP/HTTPS:
        ```
        sudo ufw allow 'Apache Full'
        ```

---

## Papildus Konfigurācija
- [ ] **1. Aktivizējiet Apache moduļus**
    - Piemēram, `mod_rewrite`:
        ```
        sudo a2enmod rewrite
        sudo systemctl restart apache2
        ```

- [ ] **2. Integrējiet logging un monitoring**
    - Integrējiet ar monitoring rīkiem, piemēram, Prometheus vai Grafana.

---

## Rezultāts
- [ ] Apache serveris ir instalēts un konfigurēts.
- [ ] HTTPS (SSL/TLS) ir aktivizēts, nodrošinot drošu piekļuvi.
- [ ] Virtuālie hosti ir iestatīti, lai pārvaldītu vairākas tīmekļa vietnes.
- [ ] Drošība ir nodrošināta ar ugunsmūra konfigurāciju.

---
## Prometheus ātrā instalēšana (bez skaidrojumiem)

 - [ ] Lejupielādējiet jaunāko Prometheus versiju no oficiālās mājaslapas:
    ```
    wget https://github.com/prometheus/prometheus/releases/download/v3.0.0/prometheus-2.x.x.linux-amd64.tar.gz
    ```
 - [ ] Izpakojiet arhīvu:
    ```
    tar -xvzf prometheus-2.x.x.linux-amd64.tar.gz
    cd prometheus-2.x.x.linux-amd64
    ```
 - [ ] Palaidiet Prometheus:
    ```
    ./prometheus --config.file=prometheus.yml
    ```
> Prometheus būs pieejams pārlūkprogrammā: `http://localhost:9090`.

### Instalējiet Apache Eksportētāju
 - [ ] Lejupielādējiet apache_exporter:
    ```
    wget https://github.com/Lusitaniae/apache_exporter/releases/download/v0.x.x/apache_exporter-0.x.x.linux-amd64.tar.gz
    ```
 - [ ] Izpakojiet arhīvu:
    ```
    tar -xvzf apache_exporter-0.x.x.linux-amd64.tar.gz
    ```
 - [ ] Palaidiet eksportētāju:
    ```
    ./apache_exporter --scrape_uri="http://localhost/server-status?auto"
    ```




### Aktivizējiet mod_status Apache Serverī lai Apache sniegtu datus eksportētājam, jāiespējo mod_status.

 - Aktivizējiet moduli:
    ```bash
    sudo a2enmod status
    sudo systemctl restart apache2
    ```
 - Pievienojiet šādu konfigurāciju Apache virtuālā hosta konfigurācijas failam:
    ```apache
    <Location "/server-status">
        SetHandler server-status
        Require local
    </Location>
    ```
> Šī konfigurācija ļauj piekļūt `/server-status` tikai no pašas servera mašīnas.

 - Restartējiet Apache:
    ```bash
    sudo systemctl restart apache2
    ```
