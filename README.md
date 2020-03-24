# LDAP
## Задание
* 1. Установить FreeIPA;
* 2. Написать Ansible playbook для конфигурации клиента;
* 3**. Firewall должен быть включен на сервере и на клиенте.
## Выполнение
в Vagrantfile стенд из 2 виртуальных машин  
ipa - 192.168.16.1  
client - 192.168.16.2  
домен netrunner.lan  
### FreeIPA сервер
демон firewalld запущен и в нем дополнительно разрешается подключение к сервисам ldap и dns
```bash
firewall-cmd --add-service=freeipa-ldap --add-service=freeipa-ldaps --add-service=dns --zone=public --permanent
```
установлены пакеты ipa-server ipa-server-dns и исправлен hosts 
```bash
      yum install ipa-server -y
      yum install ipa-server-dns -y
      sed -i '1s/127.0.0.1.*/192.168.16.1 ipa.netrunner.lan/' /etc/hosts
```

конфиг сервера (закоментирован в vagrantfile, т.к. не всегда проходил при провижене, запускал вручную)
```bash
ipa-server-install -U -r NETRUNNER.LAN -n netrunner.lan -p DM123456 -a AD123456 --mkhomedir --hostname=ipa.netrunner.lan --setup-dns --auto-forwarders --forward-policy=only --no-reverse
```

### FreeIPA клиент
Настраивается с помощью ansible роли freeipa-client
```bash
ansible-playbook playbooks/freeipa-client.yml
```

#### переменные
```bash
domain: netrunner.lan
server: ipa.netrunner.lan
realm: NETRUNNER.LAN
hostname: client.netrunner.lan
principal: admin
password: AD123456
dns: 192.168.16.1
```
#### задачи

##### установка компонента nmcli
```bash
- name: Install nmcli components on client
  yum:
    name: NetworkManager-glib
    state: present
```
##### DNS сервер
```bash
- name: Add IPv4 DNS server address
  nmcli:
    conn_name: System eth1
    type: ethernet
    dns4:
    - "{{ dns }}"
    state: present
```
##### отключаю DNS virtualbox
```bash
- name: Remove auto-dns
  shell: "nmcli con mod 'System eth0' ipv4.ignore-auto-dns yes"
  notify: restart network
```
##### Запускаю демон firewalld
```bash
- name: enable firewalld
  systemd:
    name: firewalld
    state: started
    enabled: yes
```
##### правила firewalld
```bash
- name: firewalld ldap
  firewalld:
    zone: public
    service: freeipa-ldap
    permanent: yes
    state: enabled
    immediate: yes

- name: firewalld ldaps
  firewalld:
    zone: public
    service: freeipa-ldaps
    permanent: yes
    state: enabled
    immediate: yes

- name: firewalld dns
  firewalld:
    zone: public
    service: dns
    permanent: yes
    state: enabled
    immediate: yes
```
##### установка пакета ipa-client
```bash
- name: Install freeIPA on client
  yum:
    name: freeipa-client
    state: present
```
##### настройка клиента
```bash
- name: Setup client
  shell: "ipa-client-install -U --mkhomedir --force-ntpd --domain={{ domain }} --server={{ server }} --realm={{ realm }} --hostname={{ hostname }} --principal={{ principal }} --password={{ password }}"
```

