Нет, Samba **не создаёт** лог-файлы автоматически. Вам нужно вручную создать файл и настроить права доступа. Вот как это сделать правильно:

### 1. Создание лог-файла
```bash
sudo touch /var/log/samba/cowork.log
```

### 2. Настройка прав
```bash
sudo chown root:root /var/log/samba/cowork.log
sudo chmod 664 /var/log/samba/cowork.log
```

### 3. Проверка записи
После перезапуска Samba:
```bash
sudo systemctl restart smbd
sudo tail -f /var/log/samba/cowork.log
```

### Важные нюансы:
1. **Ротация логов** (добавьте в `/etc/logrotate.d/samba-cowork`):
   ```bash
   /var/log/samba/cowork.log {
       daily
       missingok
       rotate 7
       compress
       delaycompress
       sharedscripts
       postrotate
           /usr/bin/systemctl reload smbd >/dev/null 2>&1 || true
       endscript
   }
   ```

2. **Если логи не пишутся**:
   - Проверьте AppArmor/SELinux:
     ```bash
     sudo aa-status | grep samba
     ```
   - Добавьте правило в AppArmor при необходимости

3. **Альтернативный вариант** - использовать существующий лог:
   ```ini
   log file = /var/log/samba/log.%m  # Автоматически по имени клиента
   ```

После настройки Samba начнёт писать логи только при активном использовании раздела `[cowork]`.
