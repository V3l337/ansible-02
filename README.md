
## Инструкция по использованию

### 1. Подготовка inventory-файла

Создайте ВМ машину и указал папраметры для подключениея в `inventory`
```yml
ansible_host: 
ansible_user: 
ansible_ssh_private_key_file:
```

### 2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает vector. Конфигурация vector должна деплоиться через template файл jinja2. От вас не требуется использовать все возможности шаблонизатора, просто вставьте стандартный конфиг в template файл. Информация по шаблонам по ссылке. не забудьте сделать handler на перезапуск vector в случае изменения конфигурации!

Описал как устаналвивать vector в отельном файле и проверил (в будущем просто вставил в `site.yml`)
Информацию по установке - https://vector.dev/docs/setup/installation/
Информацию по шаблону - https://vector.dev/docs/reference/configuration/
```yml
# Install Vector
    - name: Install curl
      ansible.builtin.apt:
        name:
          - curl
        state: present
        update_cache: yes

    - name: Script detects platform and installs Vector
      shell: curl --proto '=https' --tlsv1.2 -sSfL https://sh.vector.dev | bash -s -- -y

    - name: Create directory for Vector configuration
      ansible.builtin.file:
        path: /etc/vector
        state: directory
        mode: '0755'

    - name: Copy Vector configuration
      ansible.builtin.template:
        src: /root/ansible/ansible-02/playbook/templates/vector_conf.j2
        dest: /etc/vector/vector.yaml
        owner: root
        group: root
        mode: '0644'

  handlers:
    - name: Restart vector service
      ansible.builtin.systemd:
        name: vector
        state: restarted
```

### 3. При создании tasks рекомендую использовать модули: get_url, template, unarchive, file.
### 4. Tasks должны: скачать дистрибутив нужной версии, выполнить распаковку в выбранную директорию, установить vector.
### 5. Запустите ansible-lint site.yml и исправьте ошибки, если они есть.

Исправил скачивание пакетов и установку с .rpm на .deb (так как Ubuntu) и добавил более новую версию в `vars` 
```yml
ansible.builtin.get_url:
          url: "https://packages.clickhouse.com/deb/pool/main/c/{{ item }}/{{ item }}_{{ clickhouse_version }}_amd64.deb"
          dest: "./{{ item }}_{{ clickhouse_version }}_amd64.deb"
        with_items: "{{ clickhouse_packages }}"
```
```yml
- name: Install clickhouse packages
        ansible.builtin.apt:
          deb: "./{{ item }}_{{ clickhouse_version }}_amd64.deb"
          state: present
        with_items: "{{ clickhouse_packages }}"
        notify: Start clickhouse service
```
```yml
clickhouse_version: "22.8.5.29"
```

### 6. Попробуйте запустить playbook на этом окружении с флагом --check.
![image](https://github.com/user-attachments/assets/069d725a-5200-4b06-a166-953302b2d5d5)

### 7. Запустите playbook на prod.yml окружении с флагом --diff. Убедитесь, что изменения на системе произведены.
![image](https://github.com/user-attachments/assets/ebdb4328-9fb1-4660-a299-1eaeff4558f4)

### 8. Повторно запустите playbook с флагом --diff и убедитесь, что playbook идемпотентен.
![image](https://github.com/user-attachments/assets/57650c95-3b85-4072-97f9-c727f6826b50)




