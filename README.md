# CV

Лендинг-резюме Никиты Колосова, ведущего Go-разработчика.

Весь сайт — один статический файл `site/index.html`, без сборки и зависимостей. Блок опыта сделан в виде трейса: каждая компания — span на шкале 2013–2026, клик раскрывает подробности. Длительность текущей работы пересчитывается в браузере.

```
site/                       то, что уезжает на сервер
.github/workflows/deploy.yml  проверка HTML и деплой на VDS
deploy/nginx.conf.example   пример конфига nginx
.htmlvalidate.json          правила проверки HTML
```

## Деплой

При каждом пуше в `master`, который меняет `site/`, GitHub Actions:

1. проверяет разметку через html-validate;
2. по SSH заливает содержимое `site/` на сервер через `rsync` (лишние файлы удаляются, новые подменяются в самом конце загрузки);
3. если задан `SITE_URL`, открывает сайт и проверяет, что он отвечает.

Запустить вручную: **Actions → Deploy → Run workflow**.

### Подготовка сервера (один раз)

```bash
# на сервере: пользователь для деплоя и папка сайта
sudo adduser --disabled-password --gecos "" deploy
sudo mkdir -p /var/www/cv && sudo chown deploy:deploy /var/www/cv
sudo apt install -y rsync nginx

# локально: отдельный ключ только для деплоя
ssh-keygen -t ed25519 -f ~/.ssh/cv_deploy -N "" -C "github-actions cv"
ssh-copy-id -i ~/.ssh/cv_deploy.pub deploy@<host>
```

Конфиг nginx — в `deploy/nginx.conf.example`: скопируйте его в `/etc/nginx/sites-available/`, поменяйте `server_name` и `root`, включите сайт и выпустите сертификат через `certbot --nginx`.

### Настройки репозитория

**Settings → Secrets and variables → Actions.**

Secrets:

| Имя | Что положить |
| --- | --- |
| `DEPLOY_SSH_KEY` | приватный ключ целиком: `cat ~/.ssh/cv_deploy` |
| `DEPLOY_KNOWN_HOSTS` | отпечаток сервера: `ssh-keyscan -p <port> <host>` |

Variables:

| Имя | Пример |
| --- | --- |
| `DEPLOY_HOST` | `203.0.113.10` или `cv.example.com` |
| `DEPLOY_USER` | `deploy` |
| `DEPLOY_PATH` | `/var/www/cv` |
| `DEPLOY_PORT` | `22` (необязательно) |
| `SITE_URL` | `https://cv.example.com` (необязательно, для проверки после деплоя) |

То же через GitHub CLI:

```bash
gh secret set DEPLOY_SSH_KEY < ~/.ssh/cv_deploy
ssh-keyscan -p 22 <host> | gh secret set DEPLOY_KNOWN_HOSTS
gh variable set DEPLOY_HOST --body "<host>"
gh variable set DEPLOY_USER --body "deploy"
gh variable set DEPLOY_PATH --body "/var/www/cv"
gh variable set SITE_URL --body "https://<domain>"
```

Если чего-то не хватает, workflow остановится на шаге «Check configuration» и напишет, что именно не задано.

## Правка

Опыт описан прямо в разметке. Положение полосы задаётся в месяцах от августа 2013: `style="--s:<начало>;--e:<конец>"`.

Проверить HTML локально: `npx html-validate site/`.
