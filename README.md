# Компактная справка по ZFS ACL

## 📋 **Типы ACL в ZFS**

### 1. **POSIX ACL (традиционные)**
```bash
zfs set acltype=posixacl pool/dataset
```

### 2. **NFSv4 ACL (рекомендуемые)**
```bash
zfs set acltype=nfsv4acl pool/dataset
```

## 🔧 **Основные команды**

### Просмотр ACL
```bash
# POSIX ACL
getfacl /path/to/dataset

# NFSv4 ACL  
nfs4_getfacl /path/to/dataset
# или
getfacl -a /path/to/dataset
```

### Установка ACL
```bash
# POSIX ACL
setfacl -m u:user:permissions /path/to/dataset
setfacl -m g:group:permissions /path/to/dataset

# NFSv4 ACL
nfs4_setfacl -a A::user@domain:permissions /path/to/dataset
```

## 🎯 **Базовые права**

### UNIX права (для POSIX)
- `r` - read (4)
- `w` - write (2) 
- `x` - execute (1)

### NFSv4 права
- `r` - read data
- `w` - write data
- `x` - execute
- `p` - append data
- `d` - delete
- `D` - delete child
- `a` - read attributes
- `A` - write attributes
- `R` - read ACL
- `W` - write ACL
- `C` - change owner
- `S` - synchronize

## 📝 **Примеры использования**

### 1. **Наследование ACL**
```bash
# Рекурсивная установка
setfacl -R -m u:john:rwx /pool/data

# Наследование для новых файлов
setfacl -d -m u:john:rwx /pool/data
```

### 2. **Клонирование прав**
```bash
# Скопировать ACL с одного датасета на другой
getfacl /source | setfacl -f - /destination
```

### 3. **Просмотр всех ACL датасетов**
```bash
zfs get -r acltype pool | grep -v off
```

## ⚙️ **Управление через ZFS**

### Включение/выключение ACL
```bash
# Включить ACL
zfs set acltype=posixacl pool/dataset
zfs set aclinherit=passthrough pool/dataset

# Выключить ACL
zfs set acltype=off pool/dataset
```

### Настройки наследования
```bash
zfs set aclinherit=passthrough pool/dataset  # Полное наследование
zfs set aclinherit=restricted pool/dataset   # Ограниченное наследование
zfs set aclinherit=discard pool/dataset      # Без наследования
```

## 🔍 **Полезные команды**

### Поиск файлов с ACL
```bash
# Найти все файлы с ACL в датасете
find /pool/dataset -exec getfacl {} \; 2>/dev/null | grep "^# file" | cut -d' ' -f3
```

### Сброс ACL
```bash
# Удалить все ACL
setfacl -b /path/to/dataset

# Рекурсивный сброс
setfacl -R -b /path/to/dataset
```

## 💡 **Рекомендации**

1. **Используйте NFSv4 ACL** - более современные и функциональные
2. **Настраивайте наследование** через `aclinherit`
3. **Тестируйте права** перед применением на production
4. **Резервное копирование ACL** важных датасетов

```
# Экспорт ACL для бэкапа
getfacl -R /pool/data > data_acl_backup.txt
```

Эта шпаргалка покрывает 95% повседневных задач по управлению ACL в ZFS! 🚀

## Восстановление ACL из бэкапа

### 1. **Базовое восстановление**
```bash
# Восстановление ACL из файла
setfacl --restore=data_acl_backup.txt

# С проверкой
setfacl --test --restore=data_acl_backup.txt
```

### 2. **Восстановление с фильтрацией**
```bash
# Если нужно восстановить только определенные пути
grep -E "^(# file:|^[^#])" data_acl_backup.txt | setfacl --restore=-
```

### 3. **Для отдельных файлов/папок**
```bash
# Извлечь ACL для конкретного файла
awk '/^# file: filename.txt/,/^$/' data_acl_backup.txt | setfacl --restore=-
```

### 4. **Восстановление с изменением путей**
```bash
# Если структура путей изменилась
sed 's|/old/path/|/new/path/|g' data_acl_backup.txt | setfacl --restore=-
```

## ⚠️ **Важные нюансы:**

1. **Права доступа** - убедитесь, что у вас есть права на изменение ACL
2. **Существование путей** - все файлы/папки должны существовать
3. **Порядок восстановления** - сначала директории, потом файлы

## 🔍 **Проверка восстановления:**
```bash
# Сравнение с бэкaпом
getfacl -R /pool/data > data_acl_new.txt
diff data_acl_backup.txt data_acl_new.txt
```

Команда `setfacl --restore` автоматически обрабатывает формат, созданный `getfacl -R`! ✅
