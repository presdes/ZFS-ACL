Для просмотра прав ZFS датасетов с UNIX и ACL правами в виде дерева можно использовать следующие команды:

## 1. Просмотр дерева датасетов с основными правами

```bash
# Просмотр структуры датасетов
zfs list -t filesystem -o name,mountpoint,quota,compression,acltype -r poolname

# Пример с форматированием
zfs list -t filesystem -o name,mountpoint,quota,compression,acltype -r poolname | tree --fromfile
```

## 2. Рекурсивный просмотр UNIX прав

```bash
# Для конкретного датасета
zfs get all poolname/dataset | grep -E "(permission|acltype|mode)"

# Рекурсивно для всего пула
zfs get -r permission,mode,acltype poolname
```

## 3. Просмотр расширенных ACL прав

```bash
# Рекурсивный просмотр ACL для конкретного датасета
getfacl -R /path/to/dataset

# С выводом в виде дерева
getfacl -R /path/to/dataset | awk '/^# file:/ {print $2} /^[^#]/ {print "  "$0}'
```

## 4. Комплексный скрипт для отображения дерева прав

Создайте скрипт `zfs_acl_tree.sh`:

```bash
#!/bin/bash

POOL_NAME="$1"

echo "ZFS Datasets Tree with ACL permissions for pool: $POOL_NAME"
echo "=========================================================="

zfs list -t filesystem -o name -r "$POOL_NAME" | while read dataset; do
    if [ "$dataset" != "NAME" ]; then
        # Отступы для дерева
        depth=$(echo "$dataset" | tr -cd '/' | wc -c)
        indent=$(printf '%*s' $((depth*2)) "")
        
        # Основные свойства
        mountpoint=$(zfs get -H -o value mountpoint "$dataset")
        permission=$(zfs get -H -o value permission "$dataset")
        acltype=$(zfs get -H -o value acltype "$dataset")
        
        echo "${indent}📁 $dataset"
        echo "${indent}  ├─ Mountpoint: $mountpoint"
        echo "${indent}  ├─ UNIX Perm: $permission"
        echo "${indent}  └─ ACL Type: $acltype"
        
        # Показываем ACL если они есть
        if [ "$acltype" != "off" ] && [ "$acltype" != "discard" ]; then
            echo "${indent}  └─ ACL Entries:"
            getfacl "$mountpoint" 2>/dev/null | grep -v "^#" | grep -v "^$" | while read acl_line; do
                echo "${indent}      └─ $acl_line"
            done
        fi
        echo
    fi
done
```

**Использование:**
```bash
chmod +x zfs_acl_tree.sh
./zfs_acl_tree.sh poolname
```

## 5. Альтернатива с утилитой `tree` и `getfacl`

```bash
# Установите tree если нет
sudo apt install tree

# Просмотр структуры с ACL
tree -a -L 3 -d /path/to/pool | while read line; do
    if [[ $line == *"📁"* ]]; then
        dir=$(echo $line | awk '{print $2}')
        echo "$line - $(getfacl -c "$dir" 2>/dev/null | head -5)"
    else
        echo "$line"
    fi
done
```

## 6. Быстрый просмотр только ACL датасетов

```bash
# Найти все датасеты с включенными ACL
zfs get -r acltype poolname | grep -v "off" | grep -v "discard"

# Показать ACL для этих датасетов
zfs get -r acltype poolname | grep -v "off" | awk '{print $1}' | while read dataset; do
    mountpoint=$(zfs get -H -o value mountpoint "$dataset")
    echo "=== $dataset ($mountpoint) ==="
    getfacl -c "$mountpoint" 2>/dev/null
    echo
done
```

## 7. Для NFSv4 ACL (более современный формат)

```bash
# Если используете nfs4acltype
zfs get -r nfs4acltype poolname

# Просмотр NFSv4 ACL
nfs4_getfacl /path/to/dataset
```

**Пример вывода:**
```
📁 poolname
  ├─ Mountpoint: /poolname
  ├─ UNIX Perm: 755
  └─ ACL Type: posixacl
  └─ ACL Entries:
      └─ user:john:rwx
      └─ group:users:r-x

📁 poolname/dataset1
  ├─ Mountpoint: /poolname/dataset1
  ├─ UNIX Perm: 770
  └─ ACL Type: nfsv4
```

Эти команды покажут вам полную иерархию прав от корневого датасета к дочерним с учетом как UNIX, так и ACL прав.
