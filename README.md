# 🛠️ Ansible konfigurācijas automatizācija – Majasdarbs

Šis projekts automatizē vairākus uzdevumus uz pieciem Linux hostiem, kas atrodas divos dažādos datu centros (AAA un ZZZ), izmantojot Ansible. Projekts sastāv no trim galvenajām lomām (roles), kas katra pilda noteiktu funkcionalitāti:

---

## 👤 Lietotāja izveide ar sudo tiesībām

- Tiek pievienots jauns lietotājs visiem hostiem ar norādītu lietotājvārdu.
- Lietotājam automātiski tiek piešķirtas **sudo** tiesības.
- Ir iespējams pievienot arī SSH publisko atslēgu (izvēles iespēja).
- Konfigurācija atrodas `roles/user`.

---

## 🕒 Laika sinhronizācija ar Chrony

- Tiek uzstādīts un konfigurēts `chrony` NTP klients katrā hostā.
- Atkarībā no hosta piederības datu centram:
  - Hostiem datu centrā **AAA** (`hostname1`, `hostname4`) tiek izmantots: `ntp.aaa.local`
  - Hostiem datu centrā **ZZZ** (`hostname2`, `hostname3`, `hostname5`) tiek izmantots: `ntp.zzz.local`
- Konfigurācijas fails tiek ģenerēts ar Jinja2 šablonu (`chrony.conf.j2`).
- Konfigurācija atrodas `roles/chrony`.

---

## 📈 Zabbix Agent2 uzstādīšana un konfigurācija

- Tiek lejupielādēta un uzstādīta `zabbix-agent2` pakotne, izmantojot oficiālos `.rpm` un `.deb` failus atkarībā no OS.
- Aģents tiek konfigurēts automātiski, iestatot savienojumu ar norādīto Zabbix servera IP adresi.
- Konfigurācijas fails (`zabbix_agent2.conf`) tiek ģenerēts ar šablonu (`zabbix_agent2.conf.j2`).
- Konfigurācija atrodas `roles/zabbix_agent`.

---


## ✅ Priekšnosacījumi

- Ansible uzstādīts serverī no kura tiks darbināti playbooki.
- Ir izveidota SSH piekļuve visiem hostiem.
- Hostu saraksts ievietots `inventory.ini`.

<br><br><br>

# 👤 Lietotāja pievienošana ar sudo tiesībām

## 📋 Apraksts

Šis Ansible projekts veic automatizētu jauna lietotāja izveidi norādītajos Linux hostos. Lietotājs tiek pievienots attiecīgajai administrēšanas grupai (`sudo` vai `wheel` atkarībā no sistēmas), lai piešķirtu **sudo** tiesības.

## 🛠️ Funkcionalitāte

`user` loma veic šādus uzdevumus:

- Izveido jaunu lietotāju ar norādīto vārdu un komentāru
- Piešķir lietotājam administrēšanas tiesības, pievienojot to `sudo` vai `wheel` grupai
- Izveido mājas direktoriju
- Iestata (`/bin/bash`)
- (Pēc izvēles) Pievieno SSH publisko atslēgu pie lietotāja `~/.ssh/authorized_keys`

## 📁 Fukcijas struktūra

```
ansible-project/
├── add_user.yml
├── inventory.ini
└── roles/
    └── user/
        ├── tasks/
        │   └── main.yml
        └── vars/
            └── main.yml
```

## ▶️ Piemērs – `add_user.yml`

```yaml
- name: Add sudo user to all hosts
  hosts: all
  become: yes
  vars:
    new_user: adminuser
    new_user_comment: Admin User
    # new_user_ssh_pubkey: "ssh-rsa AAAAB3NzaC1..." 
  roles:
    - user
```

## 🚀 Izpilde

Palaidiet šo komandu, lai pievienotu jauno lietotāju visos hostos:

```bash
ansible-playbook -i inventory.ini add-user.yml
```

<br><br><br>

# 📡 Zabbix Agent2 Uzstādīšana un Konfigurācija


Šī Ansible loma uzstāda Zabbix Agent2 visos mērķa hostos un konfigurē to atbilstoši norādītajam Zabbix serverim.

---

## 🔧 Funkcionalitāte

- Lejupielādē un uzstāda Zabbix Agent2 pakotni atkarībā no operētājsistēmas:
  - **Debian / Ubuntu**: `.deb` pakotne  
    [zabbix-agent2_7.2.7-1+ubuntu24.04_arm64.deb](https://repo.zabbix.com/zabbix/7.2/stable/ubuntu/pool/main/z/zabbix/zabbix-agent2_7.2.7-1+ubuntu24.04_arm64.deb)
  - **RedHat / CentOS**: `.rpm` pakotne  
    [zabbix-agent2-7.2.7-release1.el9.x86_64.rpm](https://repo.zabbix.com/zabbix/7.2/stable/rhel/9/x86_64/zabbix-agent2-7.2.7-release1.el9.x86_64.rpm)

- Izveido konfigurāciju no `zabbix_agent2.conf.j2`
  - `Server` – Zabbix servera IP vai hostname
  - `ServerActive` – Zabbix servera IP vai hostname
  - `Hostname` – mērķa hosta hostname

- Startē un iespējo `zabbix-agent2` servisu

---

## 📁 Fukcijas struktūra

```
roles/
└── zabbix_agent/
    ├── tasks/
    │   └── main.yml
    └── templates/
        └── zabbix_agent2.conf.j2
```

---

## 📝 Playbook Piemērs (`zabbix_agent2.yml`)

```yaml
- hosts: all
  become: yes
  vars:
    zabbix_server: "10.0.0.1"  # aizvieto ar Zabbix servera IP vai hostname
  roles:
    - zabbix_agent
```

---

## 🧩 Šablons: `zabbix_agent2.conf.j2`

```jinja
PidFile=/run/zabbix/zabbix_agent2.pid
LogType=file
LogFile=/var/log/zabbix/zabbix_agent2.log
LogFileSize=0
Server={ zabbix_server }
ServerActive={ zabbix_server }
Hostname={ inventory_hostname }
Include=/etc/zabbix/zabbix_agent2.d/*.conf
```

---

## ▶️ Izpilde

```bash
ansible-playbook -i inventory.ini zabbix_agent2.yml
```

---

## 📌 Piezīmes

- Pārliecinies, ka mērķa hostiem ir piekļuve internetam Zabbix pakotņu lejupielādei.
- Hostiem jābūt ar `sudo` piekļuvi.

<br><br><br>

## 🕒 Chrony Laika Sinhronizācija

Šī loma uzstāda un konfigurē `chrony` laika sinhronizācijas pakotni uz visiem hostiem.

- Hostiem datu centrā **AAA** (`hostname1`, `hostname4`) tiek izmantots NTP serveris: `ntp.aaa.local`
- Hostiem datu centrā **ZZZ** (`hostname2`, `hostname3`, `hostname5`) tiek izmantots NTP serveris: `ntp.zzz.local`

Failā `chrony.conf.j2` tiek ģenerēta konfigurācija atbilstoši hostname.

## 📁 Fukcijas struktūra

```

roles/
└── chrony/
    ├── tasks/
    │   └── main.yml
    └── templates/
        └── chrony.conf.j2
```

## 📝 Playbook Piemērs (`chrony.yml`)

```yaml
---
- name: Konfigure Chrony NTP sinhronizaciju uz visiem hostiem
  hosts: all
  become: yes
  roles:
    - chrony
```

## ▶️ Izpilde

```bash
ansible-playbook -i inventory.ini chrony.yml
```


# 🤖 Gala Izpildes fails (`alltasks.yml`)
Šis fails palaid'is visus trīs uzdevumus uz visiem hostiem.

```yaml
---
- name: Izpilda visas konfigurācijas darbības uz visiem hostiem
  hosts: all
  become: yes
  vars:
    new_user: adminuser
    new_user_comment: Admin User
    #new_user_ssh_pubkey: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC..."  # opcional, atkomentē ja nepieciešams
    zabbix_server: "10.0.0.1"  # Nomaini ar īsto Zabbix servera IP
  roles:
    - user
    - zabbix_agent
    - chrony
```

## ▶️ Izpilde

```bash
ansible-playbook -i inventory.ini alltasks.yml
```