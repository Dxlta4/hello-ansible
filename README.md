# Лабораторная работа: Основы Ansible в DevOps

---

## Ход работы

## 1. Установка Ansible на управляющую машину (Linux/WSL)

### Шаг 1.1: Обновление пакетов системы
```bash
sudo apt update
sudo apt upgrade -y
```
<img width="1280" height="687" alt="image" src="https://github.com/user-attachments/assets/43f3438d-8fed-4b69-a3cc-4382406ab9fb" />


### Шаг 1.2: Установили Python и pip, проверили версию

<img width="1280" height="674" alt="image" src="https://github.com/user-attachments/assets/27b4774e-76c4-4ae5-9c2b-1bdefba1b4db" />
<img width="1280" height="668" alt="image" src="https://github.com/user-attachments/assets/55ac28c1-5879-48de-b617-3dc92892ae24" />


### Шаг 1.3: Установили Ansible и проверили версию
```bash
sudo apt install -y ansible
ansible --version
```

<img width="1280" height="671" alt="image" src="https://github.com/user-attachments/assets/e7c91439-decd-43b2-9241-3edfcf6aa883" />


Ожидаемый вывод:
```
ansible [core 2.14.x]
  config file = None
  configured module search path = ['/home/user/.ansible/plugins/modules']
  ...
```

---

## 2. Подготовка SSH ключей для управляемых машин

SSH ключи используются для аутентификации Ansible на управляемых хостах без ввода пароля.

### Шаг 2.1: Генерация SSH ключевой пары на управляющей машине
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/ansible_key -N ""
```

### Шаг 2.2: Установка прав доступа на приватный ключ
```bash
chmod 600 ~/.ssh/ansible_key
chmod 644 ~/.ssh/ansible_key.pub
```
<img width="1280" height="316" alt="image" src="https://github.com/user-attachments/assets/7d54d113-d23f-4269-9e68-38d8c3ef87e2" />


---

## 3. Запуск управляемого контейнера в Docker

### Шаг 3.1: Сборка и запуск контейнера
``bash

## Перейдите в директорию с docker-compose.yml


<img width="629" height="106" alt="image" src="https://github.com/user-attachments/assets/6b8730a9-0998-403f-b175-5b1e9de6af9d" />

<img width="461" height="132" alt="image" src="https://github.com/user-attachments/assets/a2d282b9-0c41-41ca-8267-2cab293ceb5f" />

# Запуск контейнера в фоновом режиме и проверка
docker-compose up -d
``
<img width="1280" height="498" alt="image" src="https://github.com/user-attachments/assets/64b51733-2e2f-4374-879d-f5128737c38f" />

Ожидаемый вывод:
```
NAME                    COMMAND               STATUS      PORTS
ansible-managed-host    "/usr/sbin/sshd -D"   Up 1 min    0.0.0.0:2222->22/tcp
```

## 4. Проверка SSH подключения к контейнеру

### Шаг 4.1: Проверка SSH подключения
```bash
ssh -i ~/.ssh/ansible_key -p 2222 ansible@localhost
```

Ожидаемый результат: вы должны попасть в bash контейнера без ввода пароля.
<img width="1048" height="108" alt="image" src="https://github.com/user-attachments/assets/62b1bf11-5728-43ec-aa20-3d3547e122b1" />
<img width="860" height="471" alt="image" src="https://github.com/user-attachments/assets/ee041e49-3f53-4674-abfe-edd56974e2a6" />


Выход из контейнера:
```bash
exit
```

---

## 5. Проверка инвентарного файла Ansible (inventory)

```bash
ansible-inventory -i inventory.ini --list
```
<img width="743" height="511" alt="image" src="https://github.com/user-attachments/assets/27f59dde-c0a5-4e08-88a8-4189b47f24b4" />

Ожидаемый вывод (JSON формат):
```json
{
    "_meta": {
        "hostvars": {
            "managed1": {
                "ansible_host": "localhost",
                "ansible_port": "2222",
                ...
            }
        }
    },
    "all": {...},
    "managed_hosts": {...},
    "ungrouped": {}
}
```

---

## 6. Проверка подключения Ansible к управляемому хосту

### Шаг 6.1: Тест ping
```bash
ansible -i inventory.ini managed_hosts -m ping
```
<img width="673" height="120" alt="image" src="https://github.com/user-attachments/assets/26bf2500-2f27-4ded-a3f1-f17271f63eed" />


Ожидаемый вывод:
```
managed1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Шаг 6.2: Сбор информации о системе (facts)
```bash
ansible -i inventory.ini managed1 -m setup
```
<img width="831" height="813" alt="image" src="https://github.com/user-attachments/assets/2d6c35b4-361f-4a92-9ec1-d06c5c8612b4" />


Вывело всю информацию о системе управляемого хоста.

### Шаг 6.3: Выполнение простой команды
```bash
ansible -i inventory.ini managed1 -m command -a "uname -a"
```
<img width="1195" height="81" alt="image" src="https://github.com/user-attachments/assets/16206310-1de0-4c53-a522-b7dbc91123af" />


Ожидаемый вывод:
```
managed1 | CHANGED | rc=0 >>
Linux f3a4c8b0c4a2 5.15.0-92-generic #102-Ubuntu SMP Thu Jan 9 10:54:01 UTC 2025 x86_64 GNU/Linux
```

---

## 7. Создание и запуск Ansible Playbook

### Шаг 7.1: Запуск playbook
```bash
# Запуск playbook
ansible-playbook -i inventory.ini playbook.yml
```
<img width="665" height="39" alt="image" src="https://github.com/user-attachments/assets/56fde0ce-2360-4cef-b23b-5f7abba17c51" />

### Шаг 7.2: Вывод playbook

<img width="1280" height="472" alt="image" src="https://github.com/user-attachments/assets/b4e7b71b-4706-4168-8cb0-f2e1a77008e8" />


```
PLAY [managed_hosts] ************************************************************

TASK [Gathering Facts] **********************************************************
ok: [managed1]

TASK [Update package list] ******************************************************
changed: [managed1]

TASK [Install required packages] ************************************************
changed: [managed1]

TASK [Create test directory] ****************************************************
changed: [managed1]

TASK [Create test file with content] ********************************************
changed: [managed1]

TASK [Display file content] *****************************************************
ok: [managed1] => {
    "msg": "File content from managed host: Hello from Ansible!\nThis is a test file created by Ansible playbook."
}

TASK [Get system information] ***************************************************
ok: [managed1] => {
    "msg": "System: Linux, Hostname: f3a4c8b0c4a2, Uptime: 12 min"
}

PLAY RECAP ************************************************************
managed1 : ok=7 changed=4 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

---

## 8. Задания для выполнения

### Задание 1: Базовое подключение
1. Установили Ansible на нашей машине
2. Сгенерировали SSH ключевую пару
3. Создали инвентарный файл `inventory.ini`
4. Проверили подключение командой `ansible-inventory --list`
5. Выполнили ping к управляемому хосту

---

### Задание 2: Базовые ad-hoc команды
1. Получите информацию о ядрах CPU управляемого хоста:
   ```bash
   ansible -i inventory.ini managed1 -m setup -a "filter=ansible_processor_cores"
   ```
<img width="967" height="142" alt="image" src="https://github.com/user-attachments/assets/21dd9136-e948-4e06-9618-8c4e34a4c605" />

2. Проверьте свободное место на диске:
   ```bash
   ansible -i inventory.ini managed1 -m command -a "df -h"
   ```
<img width="768" height="247" alt="image" src="https://github.com/user-attachments/assets/d6f292d9-8ae8-4f44-8422-8d3c66ca346c" />

3. Получите список всех пользователей:
   ```bash
   ansible -i inventory.ini managed1 -m command -a "cat /etc/passwd"
   ```
<img width="859" height="491" alt="image" src="https://github.com/user-attachments/assets/793551a4-ee7e-4cab-8784-e9eb63cf1759" />

4. Измените временную зону хоста на UTC:
   ```bash
   ansible -i inventory.ini managed1 -m command -a "timedatectl set-timezone UTC"
   ```
<img width="948" height="78" alt="image" src="https://github.com/user-attachments/assets/2d6ef582-eb95-45bf-aecb-e604956848e3" />


---

### Задание 3: Работа с файлами
1. Создали и запустиои новый playbook `task3_files.yml`:
    ```bash
   ansible-playbook -i inventory.ini task3_files.yml
   ```

<img width="1280" height="606" alt="image" src="https://github.com/user-attachments/assets/314d179a-2175-43bb-b16a-0faff4a47390" />


---

## 10. Вывод

В ходе выполнения лабораторной работы установили Ansible на управляющую машину в системе linux, создали SSH ключи для управляемых машин,собрали и запустили управляемый контейнер в Docker, проверили SSH подключение к контейнеру, создали инвентарный файл Ansible (inventory), проверили подключение Ansible к управляемому хосту, создали и запустили Ansible Playbook, а также выполнили базовые ad-hoc команды.
---

