Управление серверами Linux с помощью Ansible позволяет автоматизировать настройку, развертывание и управление множеством серверов. Ansible использует YAML-файлы для описания задач и не требует установки агентов на целевые серверы, так как работает через SSH. Вот основные шаги для начала работы с Ansible:

### 1. Установка Ansible
Для начала установите Ansible на управляющей машине (control node). Обычно это ваш локальный компьютер или специальный сервер.

#### На Ubuntu/Debian:
```bash
sudo apt update
sudo apt install ansible
```

#### На CentOS/RHEL:
```bash
sudo yum install ansible
```

#### На macOS (с помощью Homebrew):
```bash
brew install ansible
```

### 2. Настройка инвентаря (Inventory)
Инвентарь — это файл, в котором перечислены все серверы, которыми вы хотите управлять. Обычно это файл в формате INI или YAML.

Пример файла инвентаря (`inventory.ini`):
```ini
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com
db2.example.com
```

Вы также можете использовать YAML-формат:
```yaml
all:
  hosts:
    web1.example.com:
    web2.example.com:
  children:
    dbservers:
      hosts:
        db1.example.com:
        db2.example.com:
```

### 3. Создание Playbook
Playbook — это YAML-файл, в котором описываются задачи, которые нужно выполнить на серверах.

Пример простого Playbook (`playbook.yml`):
```yaml
---
- hosts: webservers
  become: yes
  tasks:
    - name: Ensure Apache is installed
      apt:
        name: apache2
        state: present

    - name: Ensure Apache is running
      service:
        name: apache2
        state: started
        enabled: yes
```

### 4. Запуск Playbook
Чтобы применить Playbook к серверам, выполните команду:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

### 5. Использование модулей
Ansible предоставляет множество модулей для выполнения различных задач. Например:

- **apt**: Управление пакетами в Debian/Ubuntu.
- **yum**: Управление пакетами в CentOS/RHEL.
- **service**: Управление службами.
- **copy**: Копирование файлов на серверы.
- **file**: Управление файлами и директориями.
- **command**: Выполнение произвольных команд.

Пример использования модуля `copy`:
```yaml
- name: Copy a file to the server
  copy:
    src: /path/to/local/file
    dest: /path/to/remote/file
    owner: root
    group: root
    mode: '0644'
```

### 6. Использование переменных
Вы можете использовать переменные в Playbook для повышения гибкости. Переменные могут быть определены в инвентаре, Playbook или отдельных файлах.

Пример использования переменных:
```yaml
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  tasks:
    - name: Ensure Apache is installed
      apt:
        name: apache2
        state: present

    - name: Configure Apache
      template:
        src: /path/to/template.j2
        dest: /etc/apache2/sites-available/default.conf
```

### 7. Использование ролей (Roles)
Роли позволяют организовать Playbook в более структурированную форму. Роли содержат задачи, переменные, файлы и шаблоны, которые могут быть повторно использованы.

Пример структуры роли:
```
roles/
  common/
    tasks/
    handlers/
    templates/
    files/
    vars/
    defaults/
    meta/
```

Пример использования роли в Playbook:
```yaml
- hosts: webservers
  roles:
    - common
    - webserver
```

### 8. Дополнительные возможности
- **Handlers**: Задачи, которые выполняются только при изменении состояния (например, перезапуск службы).
- **Tags**: Позволяют запускать только определенные задачи в Playbook.
- **Vault**: Шифрование конфиденциальных данных, таких как пароли и ключи.

### Пример Playbook с использованием Handlers:
```yaml
- hosts: webservers
  tasks:
    - name: Ensure Apache is installed
      apt:
        name: apache2
        state: present
      notify: Restart Apache

  handlers:
    - name: Restart Apache
      service:
        name: apache2
        state: restarted
```

### Заключение
Ansible — это мощный инструмент для автоматизации управления серверами. С его помощью вы можете легко управлять множеством серверов, настраивать их и поддерживать в актуальном состоянии. Начните с простых задач и постепенно осваивайте более сложные возможности Ansible.
