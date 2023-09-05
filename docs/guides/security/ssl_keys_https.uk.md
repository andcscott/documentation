---
title: Генерація ключів SSL
author: Steven Spencer
contributors: Ezequiel Bruni
tested_with: 8.5
tags:
  - безпека
  - ssl
  - openssl
---
  
# Генерація ключів SSL

## Передумови

* Робоча станція та сервер під керуванням Rocky Linux
* _OpenSSL_, інстальований на машині, на якій ви збираєтеся генерувати закритий ключ і CSR, а також на сервері, де ви в кінцевому підсумку встановлюватимете свій ключ і сертифікати
* Здатність зручно виконувати команди з командного рядка
* Корисно: знання команд SSL і OpenSSL


## Вступ

Майже кожен веб-сайт сьогодні _має_ працювати з сертифікатом SSL (рівня захищених сокетів). Ця процедура допоможе вам створити закритий ключ для вашого веб-сайту, а потім розробити CSR (запит на підписання сертифіката), який ви використовуватимете для придбання нового сертифіката.

## Згенеруйте закритий ключ

Для непосвячених, закриті ключі SSL можуть мати різні розміри, виміряні в бітах, що визначає, наскільки важко їх зламати.

Станом на 2021 рік рекомендований розмір закритого ключа веб-сайту все ще становить 2048 біт. Ви можете піти вище, але подвоєння розміру ключа з 2048 біт до 4096 біт лише приблизно на 16% безпечніше, займає більше місця для зберігання ключа та спричиняє більші навантаження на ЦП під час обробки ключа.

Це сповільнює продуктивність вашого веб-сайту без значної безпеки. Дотримуйтеся розміру ключа 2048 і завжди стежте за рекомендаціями.

Для початку давайте переконаємося, що OpenSSL встановлено як на робочій станції, так і на сервері:

`dnf install openssl`

Для початку переконайтеся, що OpenSSL встановлено як на робочій станції, так і на сервері.

Наш приклад домену example.com. Пам’ятайте, що вам потрібно буде придбати та зареєструвати свій домен заздалегідь. Ви можете придбати домени через кілька «Реєстраторів».

Якщо у вас не використовується DNS (система доменних імен), ви часто можете використовувати тих самих постачальників для хостингу DNS. DNS перетворює ваш названий домен на числа (IP-адреси, IPv4 або IPv6), які може зрозуміти Інтернет. Ці IP-адреси будуть місцем розміщення веб-сайту.

Давайте згенеруємо ключ за допомогою openssl:

`openssl genrsa -des3 -out example.com.key.pass 2048`

Зверніть увагу, що ми назвали ключ із розширенням .pass. Це тому, що щойно ми виконаємо цю команду, вона попросить вас ввести парольну фразу. Введіть просту парольну фразу, яку ви можете запам’ятати, оскільки ми скоро видалимо це:

```
Enter pass phrase for example.com.key.pass:
Verifying - Enter pass phrase for example.com.key.pass:
```

Далі видалімо цю парольну фразу. Це тому, що якщо ви не видалите його, щоразу, коли ваш веб-сервер перезавантажиться та завантажить ваш ключ, вам потрібно буде вводити цю парольну фразу.

Ви можете навіть не бути поруч, щоб увійти в нього, або ще гірше, у вас може бути не готова консоль. Видаліть його зараз, щоб уникнути всього цього:

`openssl rsa -in example.com.key.pass -out example.com.key`

Це ще раз запитає цю парольну фразу, щоб видалити її з ключа:

`Введіть парольну фразу, наприклад.com.key.pass:`

Тепер, коли ви ввели парольну фразу втретє, її було видалено з файлу ключа та збережено як example.com.key

## Створення CSR

Далі нам потрібно створити CSR (запит на підписання сертифіката), який ми будемо використовувати для придбання нашого сертифіката.

Під час генерації CSR вам буде запропоновано ввести кілька частин інформації. Це атрибути X.509 сертифіката.

Одним із запитів буде «Загальне ім’я (наприклад, ВАШЕ ім’я)». Це поле має бути заповнене повним доменним іменем сервера, який буде захищено SSL. Якщо веб-сайт, який потрібно захистити, буде https://www.example.com, тоді введіть www.example.com у цьому запиті:

`openssl req -new -key example.com.key -out example.com.csr`

Це відкриває діалогове вікно:

`Назва країни (2-літерний код) [XX]:` введіть двозначний код країни, у якій знаходиться ваш сайт, наприклад "US"

`Назва штату або провінції (повна назва) []:` введіть повну офіційну назву вашого штату або провінції, наприклад "Nebraska"

`Назва населеного пункту (наприклад, місто) [Місто за замовчуванням]:` введіть повну назву міста, наприклад "Omaha"

`Назва організації (наприклад, компанія) [Company Ltd за замовчуванням]:` За бажанням ви можете ввести організацію, частиною якої є цей домен, або просто натиснути «Enter», щоб пропустити.

`Назва організаційного підрозділу (наприклад, розділ) []:` Це буде опис підрозділу організації, до якого належить ваш домен. Знову ж таки, ви можете просто натиснути «Enter», щоб пропустити.

`Загальна назва (наприклад, ваше ім’я або ім’я хосту вашого сервера) []:` Тут ми маємо ввести ім’я хосту нашого сайту, наприклад, «www.example.com»

`Адреса електронної пошти []:` Це поле необов’язкове, ви можете заповнити його або просто натиснути «Enter», щоб пропустити.

Далі вам буде запропоновано ввести додаткові атрибути, які можна пропустити, натиснувши «Enter» в обох випадках:

```
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

Тепер ви повинні створити свій CSR.

## Придбання Сертифікату

Кожен постачальник сертифікатів матиме однакову процедуру. Ви купуєте SSL і термін (1 або 2 роки тощо), а потім надсилаєте свій CSR. Для цього вам потрібно буде використати команду `more`, а потім скопіювати вміст вашого файлу CSR.

`more example.com.csr`

Що має показати вам щось на зразок цього:

```
-----BEGIN CERTIFICATE REQUEST-----
MIICrTCCAZUCAQAwaDELMAkGA1UEBhMCVVMxETAPBgNVBAgMCE5lYnJhc2thMQ4w
DAYDVQQHDAVPbWFoYTEcMBoGA1UECgwTRGVmYXVsdCBDb21wYW55IEx0ZDEYMBYG
A1UEAwwPd3d3Lm91cndpa2kuY29tMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIB
CgKCAQEAzwN02erkv9JDhpR8NsJ9eNSm/bLW/jNsZxlxOS3BSOOfQDdUkX0rAt4G
nFyBAHRAyxyRvxag13O1rVdKtxUv96E+v76KaEBtXTIZOEZgV1visZoih6U44xGr
wcrNnotMB5F/T92zYsK2+GG8F1p9zA8UxO5VrKRL7RL3DtcUwJ8GSbuudAnBhueT
nLlPk2LB6g6jCaYbSF7RcK9OL304varo6Uk0zSFprrg/Cze8lxNAxbFzfhOBIsTo
PafcA1E8f6y522L9Vaen21XsHyUuZBpooopNqXsG62dcpLy7sOXeBnta4LbHsTLb
hOmLrK8RummygUB8NKErpXz3RCEn6wIDAQABoAAwDQYJKoZIhvcNAQELBQADggEB
ABMLz/omVg8BbbKYNZRevsSZ80leyV8TXpmP+KaSAWhMcGm/bzx8aVAyqOMLR+rC
V7B68BqOdBtkj9g3u8IerKNRwv00pu2O/LOsOznphFrRQUaarQwAvKQKaNEG/UPL
gArmKdlDilXBcUFaC2WxBWgxXI6tsE40v4y1zJNZSWsCbjZj4Xj41SB7FemB4SAR
RhuaGAOwZnzJBjX60OVzDCZHsfokNobHiAZhRWldVNct0jfFmoRXb4EvWVcbLHnS
E5feDUgu+YQ6ThliTrj2VJRLOAv0Qsum5Yl1uF+FZF9x6/nU/SurUhoSYHQ6Co93
HFOltYOnfvz6tOEP39T/wMo=
-----END CERTIFICATE REQUEST-----
```

Ви хочете скопіювати все, включно з рядками «ПОЧАТИ ЗАПИТУ НА СЕРТИФІКАТ» і «КІНЕЦЬ ЗАПИТУ НА СЕРТИФІКАТ». Потім вставте їх у поле CSR на веб-сайті, де ви купуєте сертифікат.

Перед видачею сертифіката вам, можливо, доведеться виконати інші дії перевірки, залежно від власності на домен, реєстратора, яким ви користуєтеся, тощо. Коли його видають, його слід надати разом із проміжним сертифікатом від постачальника, який ви також використовуватимете в конфігурації.

## Висновок

Згенерувати всі деталі для придбання сертифіката веб-сайту не складно, і це може виконати системний адміністратор або адміністратор веб-сайту за допомогою описаної вище процедури.