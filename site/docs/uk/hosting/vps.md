# Хостинг: VPS

Віртуальний приватний сервер, відомий переважно як VPS, є віртуальною машиною, яка працює в хмарі, а користувачі мають повний контроль над її системою.

## Підготовка вашого коду

Для початку ви можете скористатися наведеним у [прикладі використання сесії](../plugins/session.md#приклад-використання) ботом.

Далі ми будемо вважати, що бот працює на Node.js.
Чому лише Node.js, а не Deno чи краще одразу для обох середовищ?
Відповідь проста: конвеєр для Deno був би дуже простим.
Конвеєр для Node.js ускладнюється додатковим завданням збірки й налаштуванням звʼязку між двома завданням.
Тож конвеєр CI/CD для Node.js буде кращим прикладом.

## Оренда сервера

У якості хостера ми обрали [Hostinger](https://hostinger.com.ua/).
На те є кілька причин:

1. Він мультимовний, тож вам буде зручно й зрозуміло ним користуватися.
2. У нього дуже багата документація, яка ретельно покриває всі проблемні місця, з якими може стикнутися новачок.
3. Він пропонує потужні сервери за досить низькими цінами.

:::tip Аналог серверу
Якщо ви не можете або не хочете орендувати сервер, ви можете замінити його віртуальною машиною.
Для цього скористайтеся застосунком на кшталт [VirtualBox](https://virtualbox.org/).
Створіть віртуальну машину з потрібним дистрибутивом Linux й користуйтеся нею, як справжнім сервером.
:::

Перейдіть до [каталогу тарифів VPS](https://hostinger.com.ua/vps-hosting/).
Ми скористаємося тарифом "VPS 1".
Нижче ви можете побачити порівняльну таблицу, в якій описані запропоновані серверні ресурси й ліміти.
Ресурсів "VPS 1" достатньо для ботів з невеликою аудиторією, а для нашого тестового бота тим паче.

Поверніться до картки "VPS 1" й натисніть кнопку "Додати".
Вас буде автоматично перенаправлено на сторінку оформлення замовлення, де ви також одразу зареєструєтеся у Hostinger.

:::warning Змініть строк оренди!
Типовий строк оренди - 2 роки (маркетингова хитрість), і коштує це величезних грошей.
Вам цього напевно не треба, тому для початку орендуйте сервер на місяць, що коштує значно дешевше.

У будь-якому разі Hostinger надає 30-денну гарантію повернення грошей.
:::

Після оплати ви зможете налаштувати свій сервер:

1. **Місцезнаходження.**
2. **Тип сервера.**
   Обиріть варіант "Чиста ОС".
3. **Операційна система.**
   Ми скористаємося Ubuntu 22.04.
   Якщо ви обрали іншу систему, деякі кроки можуть відрізнятися, тому будьте пильними.
4. **Назва сервера.**
5. **Root-пароль.**
   Придумайте надійний пароль та зберігайте його у безпечному місця!
6. **SSH-ключ.**
   Пропустіть цей крок.

Після створення сервера ви зможете підключитися до нього за допомогою SSH:

> SSH (_Secure Shell_ - _Безпечна оболонка_) - це мережевий протокол для віддаленого керування компʼютером (сервером) та передачі файлів.

```sh:no-line-numbers
ssh root@<ip-адреса>
```

замінивши `<ip-адреса>` на IP-адресу вашого сервера, яку ви можете знайти на сторінці керування сервером.

## Запуск бота

Тепер у нашому розпорядженні є сервер, на якому ми можемо запустити бота, щоб він працював цілодобово.

> Для спрощення початку статті ми пропустили крок доставки коду на сервер, але це описано [нижче](#ci-cd).

:::tip Не забудьте встановити середовище виконання!
Щоб запустити бота вам потрібно встановити на сервері Node.js або Deno, залежно від того, в якому середовищі виконання працюватиме бот.
Це виходить за межі статті, тому вам потрібно буде самостійно це зробити. :wink:
:::

Далі описано два способи, за допомогою яких ви можете підтримувати безперервну роботу бота.

### systemd

systemd — це потужний менеджер служб, який передвстановлений у багатьох дистрибутивах Linux, переважно на основі Debian.

#### Отримання команди запуску

1. Отримайте повний шлях до вашого середовища виконання:

```sh:no-line-numbers
# Якщо ви використовуєте Deno
which deno

# Якщо ви використовуєте Node.js
which node
```

2. Ви також повинні мати повний шлях до файлу, який є точкою входу вашого застосунку.

3. Ваша команда запуску повинна виглядати наступним чином:

```sh:no-line-numbers
<повний-шлях-до-середовища-виконання> <параметри> <повний-шлях-до-файлу-входу>

# Приклад для Deno:
# /home/user/.deno/bin/deno --allow-all /home/user/bot1/mod.ts

# Приклад для Node.js:
# /home/user/.nvm/versions/node/v16.9.1/bin/node /home/user/bot1/index.js
```

#### Створення служби

1. Перейдіть до каталогу служб:

```sh:no-line-numbers
cd /etc/systemd/system
```

2. Відкрийте ваш новий файл служби за допомогою редактора:

```sh:no-line-numbers
nano <назва-застосунку>.service
```

замінивши `<назва-застосунку>` на будь-який ідентифікатор.

> `<назва-застосунку>.service` буде назвою вашої служби.

3. Додайте наступний вміст:

```text
[Unit]
After=network.target

[Service]
Environment=BOT_TOKEN=<токен-бота>
Type=simple
User=<імʼя-користувача>
ExecStart=<команда-запуску>
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

замінивши `<токен-бота>` на токен вашого бота, `<команда-запуску>` на команду, яку ви отримали [вище](#отримання-команди-запуску), а `<імʼя-користувача>` на імʼя користувача, від імені якого запускається застосунок.

> Для отримання додаткової інформації про файли служб відвідайте [цей сайт](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_basic_system_settings/assembly_working-with-systemd-unit-files_configuring-basic-system-settings).

4. Перезавантажуйте systemd щоразу, як ви редагуєте службу:

```sh:no-line-numbers
systemctl daemon-reload
```

#### Керування службою

```sh:no-line-numbers
# Замініть `<назва-служби>` на назву файла служби, яку ви створили.

# Щоб запустити службу
systemctl start <назва-служби>

# Щоб переглянути журнали служби
systemctl status <назва-служби>

# Щоб перезапустити службу
systemctl restart <назва-служби>

# Щоб зупинити службу
systemctl stop <назва-служби>

# Щоб увімкнути запуск служби при завантаженні сервера
systemctl enable <назва-служби>

# Щоб вимкнути запуск служби при завантаженні сервера
systemctl disable <назва-служби>
```

### PM2

[PM2](https://pm2.keymetrics.io/) — це демонічний менеджер процесів для Node.js, який допоможе вам керувати та тримати ваш застосунок у режимі онлайн 24/7.

> PM2 створений спеціально для запуску застосунків, які написані на Node.js.
> Проте його також можна використовувати для запуску застосунків, які написані на інших мовах або для інших рантаймів.

#### Встановлення

```sh:no-line-numbers
npm install pm2@latest -g

# Якщо ви використовуєте Yarn
yarn global add pm2
```

#### Створення застосунку

PM2 пропонує два способи створення застосунку:

1. За допомогою інтерфейсу командного рядка.
2. За допомогою [файлу конфігурації](https://pm2.keymetrics.io/docs/usage/application-declaration/).

Перший спосіб зручний під час знайомства з PM2.
Проте під час розгортання варто скористатися другим способом, що ми й зробили у нашому випадку.

На нашому сервері в каталозі, де зберігається збірка бота, розташовано файл `ecosystem.config.js` із наступним вмістом:

```js
module.exports = {
  apps: [{
    name: "<назва-застосунку>",
    script: "<команда-запуску>",
  }],
};
```

де `<назва-застосунку>` може бути будь-яким ідентифікатором, а `<команда-запуску>` має бути власне командою запуску бота, наприклад, `node bot.js` або `npm run start` тощо.

#### Керування застосунком

Нижче наведено команди, за допомогою яких можна керувати застосунком.

```sh:no-line-numbers
# Якщо файл `ecosystem.config.js` знаходиться в поточному каталозі,
# ви можете нічого не вказувати, щоб запустити застосунок.
# Якщо застосунок вже запущений, то ця команда перезапустить його.
pm2 start

# Всі наступні команди вимагають вказування назви застосунку
# або файлу `ecosystem.config.js`.
# Щоб застосувати дію до всіх застосунків, вкажіть `all`. 

# Щоб перезапустити застосунок
pm2 restart <назва-застосунку>

# Щоб перезавантажити застосунок
pm2 reload <назва-застосунку>

# Щоб зупинити застосунок
pm2 stop <назва-застосунку>

# Щоб видалити застосунок
pm2 delete <назва-застосунку>
```

#### Збереження роботи

Якщо сервер перезавантажиться, ваш бот не відновить роботу.
Щоб бот відновив роботу, потрібно підготовити PM2 для цього.

На сервері в терміналі виконайте наступну команду:

```sh:no-line-numbers
pm2 startup
```

Вам буде надано команду, яку ви маєте виконати, щоб PM2 автоматично запрацював після перезавантаження серверу.

Після цього виконайте ще одну команду:

```sh:no-line-numbers
pm2 save
```

Ця команда збереже список поточних застосунків, щоб запустити їх після перезавантаження сервера.

Якщо ви створили новий застосунок й хочете його також зберегти, просто виконайте `pm2 save` ще раз.

## Запуск бота на вебхуках

Для запуску бота на вебхуках, вам потрібно буде скористатися серверним фреймворком й не викликати `bot.start()`.

Ось приклад коду для запуску бота за допомогою `fastify`, який ви маєте додати до головного файлу бота:

```ts
import { webhookCallback } from "grammy";
import { fastify } from "fastify";

const server = fastify();

server.post(`/${bot.token}`, webhookCallback(bot, "fastify"));

server.listen();
```

### Оренда домену

Щоб звʼязати бота, який працює на вебхуках, із зовнішнім світом, потрібно придбати домен.

Перейдіть на [сторінку пошуку доменних імен](https://hostinger.com.ua/kupyty-domen).
У полі введення тексту введіть доменне імʼя виду `<назва>.<зона>`.
Наприклад, `example.com`.

Якщо бажаний домен вільний, натисніть кнопку "Додати" поряд із ним.
Вас буде автоматично перенаправлено на сторінку оформлення замовлення, де ви також одразу зареєструєтеся у Hostinger, якщо досі не зареєстровані.
Оплатіть домен.

#### Направлення домена на VPS

Перш ніж ваш домен зможе працювати з вашим VPS, вам потрібно направити домен на свій сервер.
Щоб це зробити, на [панелі керування Hostinger](https://hpanel.hostinger.com/) натисніть кнопку "Керувати" поряд із вашим доменом.
Далі перейдіть на сторінку керування записами DNS, натиснувши кнопку "DNS / Сервери імен" у меню зліва.

> Спершу дізнайтеся IP-адресу свого VPS.

У списку DNS записів знайдіть запис типу `A` з іменем `@`.
Відредагуйте цей запис, змінивши IP-адресу в полі "Вказує на" на IP-адресу вашого VPS, а TTL вкажіть 3600.

Далі знайдіть й видаліть запис типу `CNAME` з іменем `www`.
Замість нього створіть новий запис типу `A` з іменем `www`, який вказуватиме на IP-адресу вашого VPS, а TTL вкажіть 3600.

> У разі винекнення труднощів скористайтеся іншим способом, описаним у [базі знань](https://support.hostinger.com/uk/articles/1583227-%D1%8F%D0%BA-%D0%BD%D0%B0%D0%BF%D1%80%D0%B0%D0%B2%D0%B8%D1%82%D0%B8-%D0%B4%D0%BE%D0%BC%D0%B5%D0%BD-%D0%BD%D0%B0-vps).

### Налаштування вебсервера

Щоб сайт запрацював, а бот почав отримувати оновлення від Telegram, потрібно налаштувати вебсервер.
Ми скористаємося [Caddy](https://caddyserver.com/).

Caddy - це потужний вебсервер з відкритим вихідним кодом з автоматичним HTTPS.
Він написаний на Go, тому він дуже швидкий. :zap:

#### Встановлення

Наступні 5 команд завантажать й автоматично запустять Caddy як службу systemd з назвою `caddy`.

```sh:no-line-numbers
apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list
apt update
apt install caddy
```

Перевірте статус роботи Caddy:

```sh:no-line-numbers
systemctl status caddy
```

:::details Усунення помилок
Деякі хостинг-провайдери надають VPS із передвстановленим вебсервером, наприклад, [Apache](https://httpd.apache.org/).
Кілька вебсерверів не можуть одночасно працювати на одній машині.
Щоб Caddy працював, потрібно зупинити й вимкнути інший вебсервер:

```sh:no-line-numbers
systemctl stop apache2
systemctl disable apache2
```

:::

Тепер, якщо ви відкриєте у браузері IP-адресу свого сервера, то побачите типову сторінку з інструкцією по налаштуванню Caddy.

#### Конфігурування

Щоб Caddy обробляв запити, які надходять до нашого домену, потрібно змінити конфігурацію Caddy.

Виконайте наступну команду, щоб відкрити файл конфігурації Caddy:

```sh:no-line-numbers
nano /etc/caddy/Caddyfile
```

Ви побачите наступну типову конфігурацію:

```text:no-line-numbers
# The Caddyfile is an easy way to configure your Caddy web server.
#
# Unless the file starts with a global options block, the first
# uncommented line is always the address of your site.
#
# To use your own domain name (with automatic HTTPS), first make
# sure your domain's A/AAAA DNS records are properly pointed to
# this machine's public IP, then replace ":80" below with your
# domain name.

:80 {
  # Set this path to your site's directory.
  root * /usr/share/caddy

  # Enable the static file server.
  file_server

  # Another common task is to set up a reverse proxy:
  # reverse_proxy localhost:8080

  # Or serve a PHP site through php-fpm:
  # php_fastcgi localhost:9000
}

# Refer to the Caddy docs for more information:
# https://caddyserver.com/docs/caddyfile
```

Для роботи бота приведіть конфіг до наступного вигляду:

```text:no-line-numbers
<домен> {
  root * /usr/share/caddy
  file_server
  reverse_proxy /<токен-бота> localhost:8000
}
```

замінивши `<домен>` на ваш домен, а `<токен-бота>` на токен вашого бота.

Перезавантажуйте Caddy щоразу, як файл конфігураціїї сайту, за допомогою наступної команди:

```sh:no-line-numbers
systemctl reload caddy
```

Тепер всі запити на адресу `https://<домен>/<токен-бота>` будуть переадресовуватися на адресу `http://localhost:8000/<токен-бота>`, на якій працює вебхук бота.

#### Підключення вебхука до Telegram

Залишилося лише сказати Telegram, куди надсилати оновлення.
Для цього відкрийте браузер й відвідайте сторінку за наступним посиланням:

```text:no-line-numbers
https://api.telegram.org/bot<токен-бота>/setWebhook?url=https://<домен>/<токен-бота>
```

замінивши `<токен-бота>` на токен вашого бота, а `<домен>` на ваш домен.

## CI/CD

[CI/CD](https://about.gitlab.com/topics/ci-cd/) - це важлива складова сучасного процесу розробки програмного забезпечення.
Цей посібник охоплює майже весь [конвеєр CI/CD](https://about.gitlab.com/topics/ci-cd/cicd-pipeline/).

Ми зосередимося на написанні скриптів для GitHub та GitLab.
За поотреби ви з легкістю зможете адаптувати наведені нижче приклади для обраного вами сервісу для CI/CD, як-от Jenkins, Buddy тощо.

:::tip Self-hosted runner
GitHub та GitLab безкоштовно пропонують певну кількість ресурсів для виконання ваших завдань.
Проте в ході налаштування конвеєру ви можете швидко витрати їх всі, через що вам доведеться платити гроші за додаткові ресурси, інакше завдання не виконуватимуться.
Щоб такого не сталося, рекомендуємо встановити на свій компʼютер self-hosted runner, щоб завдання виконувалися у вас на компʼютері.
Тоді ви позбавитеся лімітів й зможете запускати конвеєри будь-якої складності й потужності (майже).
:::

### SSH-ключі

Для доставки файлів на сервер вам потрібно налаштувати безпарольну автентифікацію, яка реалізується за допомогою SSH-ключів.

Наступні команди мають виконуватися на вашому персональному компʼютері.

Перейдіть до каталогу з SSH-ключами:

```sh:no-line-numbers
cd ~/.ssh
```

Згенеруйте нову пару ключів:

<CodeGroup>
  <CodeGroupItem title="GitHub" active>

```sh:no-line-numbers
ssh-keygen -t rsa -m PEM
```

</CodeGroupItem>
  <CodeGroupItem title="GitLab">

```sh:no-line-numbers
ssh-keygen -t ed25519
```

</CodeGroupItem>
</CodeGroup>

Ця команда згенерує публічний та приватний ключ бажаного для GitHub та GitLab типу та формату.
За потреби ви можете вказати власну назву ключа.
Зауважте, що ви маєте пропустити крок створення fingerprint (просто натисніть `Enter`).

Далі надішліть **публічний** ключ на сервер:

```sh:no-line-numbers
ssh-copy-id -i <назва-ключа>.pub root@<ip-адреса>
```

замінивши `<назва-ключа>` на назву згенерованого ключа та `<ip-адреса>` на IP-адресу вашого сервера.

Зауважте, що **публічний** ключ може розташовуватися на багатьох серверах, а **приватний** ключ має бути лише у вас й у GitHub або GitLab.

Тепер ви можете підключитися до сервера без необхідності введення паролю.

### Приклад скрипту

<CodeGroup>
  <CodeGroupItem title="GitHub" active>

```yml
name: Main

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 'latest'
      - run: npm ci
      - name: Build
        run: npm run build
      - uses: actions/upload-artifact@v3
        with:
          name: dist
          path: dist/*.js
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/download-artifact@v3
        with:
          name: dist
          path: dist/
      - name: Deploy
        uses: easingthemes/ssh-deploy@v4
        env:
          SOURCE: "dist package.json package-lock.json"
          ARGS: "--delete -az"
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
          REMOTE_USER: ${{ secrets.REMOTE_USER }}
          TARGET: "<цільовий-каталог>"
          SCRIPT_AFTER: |
            cd <цільовий-каталог>
            npm i --omit=dev
            <команда-запуску>
```

де `<цільовий-каталог>` замініть на назву каталогу, в якому на сервері зберігається збірка бота, а `<команда-запуску>` на команду запуску вашого бота, якою може бути виклик `pm2` або `systemctl`, наприклад.

Цей скрипт послідовно виконує два завдання: `build` та `deploy`.
Після виконання `build` артефакт роботи цього завдання, а саме каталог `dist`, в якому міститься збірка бота, передається до завдання `deploy`.

Доставка файлів на сервер відбувається за допомогою утиліти `rsync`, яка впроваджується `easingthemes/ssh-deploy`.
Після доставки файлів на сервері виконується команда, яка описана у змінній середовища `SCRIPT_AFTER`.
У нашому випадку після доставки файлів ми переходимо в каталог бота, в якому встановлюємо всі залежності, окрім `devDependencies`, й перезапускаємо бота.

Зауважте, що вам потрібно додати три [секретні змінні середовища](https://docs.github.com/en/actions/security-guides/encrypted-secrets/):

1. `SSH_PRIVATE_KEY` - тут має зберігатися приватний SSH-ключ, який ви створили у [попередньому кроці](#ssh-ключі).
2. `REMOTE_HOST` - тут має зберігатися IP-адреса вашого серверу
3. `REMOTE_USER` - тут має зберігатися імʼя користувача, від імені якого запускається бот.

</CodeGroupItem>
  <CodeGroupItem title="GitLab">

```yml
image: node:latest

stages:
  - build
  - deploy

Build:
  stage: build
  before_script: npm ci
  script: npm run build
  artifacts:
    paths:
      - dist/

Deploy:
  stage: deploy
  before_script:
    - "command -v ssh-agent >/dev/null || ( apt-get update -y && apt-get install openssh-client -y )"
    - "command -v rsync >/dev/null || ( apt-get update -y && apt-get install rsync -y )"
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan "$REMOTE_HOST" >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
  script:
    - rsync --delete -az dist package.json package-lock.json $REMOTE_USER@$REMOTE_HOST:<цільовий-каталог>
    - ssh $REMOTE_USER@$REMOTE_HOST "cd <цільовий-каталог> && npm i --omit=dev && <команда-запуску>"
```

де `<цільовий-каталог>` замініть на назву каталогу, в якому на сервері зберігається збірка бота, а `<команда-запуску>` на команду запуску вашого бота, якою може бути виклик `pm2` або `systemctl`, наприклад.

Цей скрипт послідовно виконує два завдання: `build` та `deploy`.
Після виконання `build` артефакт роботи цього завдання, а саме каталог `dist`, в якому міститься збірка бота, передається до завдання `deploy`.

Доставка файлів на сервер відбувається за допомогою утиліти `rsync`, яку ми маємо встановити перед виконаням основного скрипту.
Після доставки файлів ми підключаємося до сервера за допомогою SSH, щоб виконати команду для встановлення всіх залежностей, окрім `devDependencies`, та перезапуску застосунку.

Зауважте, що вам потрібно додати три [змінні середовища](https://docs.gitlab.com/ee/ci/variables/):

1. `SSH_PRIVATE_KEY` - тут має зберігатися приватний SSH-ключ, який ви створили у [попередньому кроці](#ssh-ключі).
2. `REMOTE_HOST` - тут має зберігатися IP-адреса вашого серверу
3. `REMOTE_USER` - тут має зберігатися імʼя користувача, від імені якого запускається бот.

</CodeGroupItem>
</CodeGroup>