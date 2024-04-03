# Домашнее задание к занятию "`Кластеризация и балансировка нагрузки`" - `Тен Денис`

### Задание 1
- Запустите два simple python сервера на своей виртуальной машине на разных портах
- Установите и настройте HAProxy, воспользуйтесь материалами к лекции по [ссылке](2/)
- Настройте балансировку Round-robin на 4 уровне.
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.

### Решение Задание 1

Устанавливаем HAProxy
```
yum install haproxy && systemctl enable --now haproxy && systemctl status haproxy
```
![image](https://github.com/killakazzak/10-02-slb-cluster-hw/assets/32342205/b52a7428-84cd-4229-840b-fa90347eac9b)

Запускаем http python сервер для порта 8888

```
vim http_index_8888.py
```

```python
import http.server
import socketserver

PORT = 8888
SERVER_NAME = "Сервер 8888"

Handler = http.server.SimpleHTTPRequestHandler
Handler.extensions_map['.html'] = 'text/html'

class MyHandler(Handler):
    def log_message(self, format, *args):
        print(f"[{SERVER_NAME}] {format % args}")

    def do_GET(self):
        if self.path == '/':
            self.path = '/index.html'
        return Handler.do_GET(self)

with socketserver.TCPServer(("", PORT), MyHandler) as httpd:
    print(f"{SERVER_NAME} запущен на порту {PORT}")
    httpd.serve_forever()
```
Запускаем http python сервер для порта 9999  
```
vim http_index_9999.py
```
```python
import http.server
import socketserver

PORT = 9999
SERVER_NAME = "Сервер 9999"

Handler = http.server.SimpleHTTPRequestHandler
Handler.extensions_map['.html'] = 'text/html'

class MyHandler(Handler):
    def log_message(self, format, *args):
        print(f"[{SERVER_NAME}] {format % args}")

    def do_GET(self):
        if self.path == '/':
            self.path = '/index1.html'
        return Handler.do_GET(self)

with socketserver.TCPServer(("", PORT), MyHandler) as httpd:
    print(f"{SERVER_NAME} запущен на порту {PORT}")
    httpd.serve_forever()
```


---
### Задание 2
- Запустите три simple python сервера на своей виртуальной машине на разных портах
- Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
- HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.

### Решение Задание 2

---

## Задания со звёздочкой*
Эти задания дополнительные. Их можно не выполнять. На зачёт это не повлияет. Вы можете их выполнить, если хотите глубже разобраться в материале.

---

### Задание 3*
- Настройте связку HAProxy + Nginx как было показано на лекции.
- Настройте Nginx так, чтобы файлы .jpg выдавались самим Nginx (предварительно разместите несколько тестовых картинок в директории /var/www/), а остальные запросы переадресовывались на HAProxy, который в свою очередь переадресовывал их на два Simple Python server.
- На проверку направьте конфигурационные файлы nginx, HAProxy, скриншоты с запросами jpg картинок и других файлов на Simple Python Server, демонстрирующие корректную настройку.

### Решение Задание 3*

---

### Задание 4*
- Запустите 4 simple python сервера на разных портах.
- Первые два сервера будут выдавать страницу index.html вашего сайта example1.local (в файле index.html напишите example1.local)
- Вторые два сервера будут выдавать страницу index.html вашего сайта example2.local (в файле index.html напишите example2.local)
- Настройте два бэкенда HAProxy
- Настройте фронтенд HAProxy так, чтобы в зависимости от запрашиваемого сайта example1.local или example2.local запросы перенаправлялись на разные бэкенды HAProxy
- На проверку направьте конфигурационный файл HAProxy, скриншоты, демонстрирующие запросы к разным фронтендам и ответам от разных бэкендов.

### Решение Задание 4*


------

