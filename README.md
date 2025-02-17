![image](https://github.com/user-attachments/assets/31746556-dde1-4fde-b7e1-621d1d65d855)# Решение домашнего задания к занятию «Хранение в K8s. Часть 2»
https://github.com/netology-code/kuber-homeworks/blob/main/2.2/2.2.md

## Задание 1.Создать Deployment приложения, использующего локальный PV, созданный вручную.

### Манифест деплоймента
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: depl-volume
spec:
  replicas: 1
  selector:
    matchLabels:
      type: app-pv
  template:
    metadata:
      labels:
        app: multitool
        type: app-pv
    spec:
      containers:
        - name: multitool
          image: praqma/network-multitool:alpine-extra
          volumeMounts:
            - name:  shared-vol
              mountPath: "/data"
        - name: busy-writer
          image: busybox:1.36.1
          command: ['sh', '-c', "until false; do date >> /data/test.txt; sleep 5; done"]
          volumeMounts:
            - name:  shared-vol
              mountPath: "/data"
      volumes:
        - name:  shared-vol
          persistentVolumeClaim:
            claimName: pvc-host
```
### Манифест pv
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: shared-pv
spec:
  capacity:
    storage: 10Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  storageClassName: local
  hostPath:
    path:  /data
```
### Манифест pvc
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-host
spec:
  storageClassName: "local"
  accessModes:
  - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Mi
```
### Multitool читает файл
![image](https://github.com/user-attachments/assets/014416fd-cf8f-42e1-9947-d7938d5eb297)
### Файл лежит на локальном диске
![image](https://github.com/user-attachments/assets/270cf2a4-ee4e-40d8-98a4-350e2f4ab724)
### Удаление Deployment и PVC, проверка файла
![image](https://github.com/user-attachments/assets/4201fca0-0bee-42c5-9678-8cff6dd7f182)
### Удаление PV, проверка файла
![image](https://github.com/user-attachments/assets/8f45e057-74a6-42b2-b10b-90fc2d19c68e)  
После удаления PV локальные файлы остались на месте, все потому, что как видно на скриншоте выше, RECLAIM POLICY для нашего хранилиза имеет значение Retain. Это значение меняется в параметре persistentVolumeReclaimPolicy. Вот три доступных значения для этого параметра:  
1. Retain: При выборе этого значения PV сохраняется даже после удаления PVC. 
2. Recycle: Устарело и не используется.
3. Delete: При использовании этого значения PV будет автоматически удален вместе с его хранилищем. 


## Задание 2. Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

Включить и настроить NFS-сервер на MicroK8S.
Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
Продемонстрировать возможность чтения и записи файла изнутри пода.
Предоставить манифесты, а также скриншоты или вывод необходимых команд.
