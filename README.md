
# Lietotāja pievienošana ar sudo tiesībām

## 📋 Apraksts

Šis Ansible projekts veic automatizētu jauna lietotāja izveidi norādītajos Linux hostos. Lietotājs tiek pievienots attiecīgajai administrēšanas grupai (`sudo` vai `wheel` atkarībā no sistēmas), lai piešķirtu **sudo** tiesības.

## 🛠️ Funkcionalitāte

`user` loma veic šādus uzdevumus:

- Izveido jaunu lietotāju ar norādīto vārdu un komentāru
- Piešķir lietotājam administrēšanas tiesības, pievienojot to `sudo` vai `wheel` grupai
- Izveido mājas direktoriju
- Iestata (`/bin/bash`)
- (Pēc izvēles) Pievieno SSH publisko atslēgu pie lietotāja `~/.ssh/authorized_keys`

## 📁 Projekta struktūra

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
    # new_user_ssh_pubkey: "ssh-rsa AAAAB3NzaC1..."  # pēc izvēles
  roles:
    - user
```

## 🚀 Izpilde

Palaidiet šo komandu, lai pievienotu jauno lietotāju visos hostos:

```bash
ansible-playbook -i inventory.ini add-user.yml
```