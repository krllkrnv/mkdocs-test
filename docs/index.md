# Отчет по развертыванию статического сайта

## Цель проекта
Создание и развертывание статического сайта с использованием MkDocs и автоматизация процесса деплоя на GitHub Pages.

## Выполненные действия

### 1. Подготовка окружения

#### Установка Python и проверка pip
```bash
python --version  # Python 3.12.x
pip --version     # pip 25.0.1
```

#### Создание виртуального окружения
```bash
# Создание каталога проекта
mkdir mkdocs-test
cd mkdocs-test

# Создание виртуального окружения
python -m venv env

# Активация (Windows PowerShell)
.\env\Scripts\Activate.ps1
```

#### Установка MkDocs 
```bash
pip install mkdocs mkdocs-material
```

### 2. Создание проекта MkDocs

#### Инициализация проекта
```bash
mkdocs new .
```

#### Структура проекта
```
mkdocs-test/
├── docs/
│   ├── index.md
│   ├── about.md
│   ├── technical.md
│   └── research.md
├── mkdocs.yml
└── env/
```

### 3. Настройка автоматического деплоя

#### Создание GitHub Actions workflow
Файл: `.github/workflows/docs.yml`

```yaml
name: Deploy Docs

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: 3.12
      - run: pip install mkdocs mkdocs-material
      - run: mkdocs build
      - uses: actions/upload-pages-artifact@v2
        with:
          path: ./site

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

### 4. Настройка GitHub Pages

1. Переход в Settings → Pages
2. Source: "GitHub Actions"
3. Автоматический деплой при push в main

## Исследование альтернативных решений

### Возможности использования отечественных CDN для ускорения доставки контента

#### Яндекс.Облако CDN

**Преимущества:**
- Низкая задержка в России (5-15 мс)
- Интеграция с экосистемой Яндекса
- Конкурентные цены (от 0.5₽ за ГБ)
- Поддержка HTTP/2 и HTTP/3
- Автоматическое сжатие контента

**Технические возможности:**
- Кэширование статических ресурсов
- Поддержка SSL/TLS
- Географическое распределение
- Аналитика трафика
- API для управления

**Пример настройки:**
```yaml
# Конфигурация для статического сайта
cdn_config:
  origin: "https://your-site.com"
  cache_rules:
    - path: "*.css,*.js,*.png,*.jpg"
      ttl: 86400  # 24 часа
    - path: "*.html"
      ttl: 3600   # 1 час
```

### Возможности GitVerse для реализации CI/CD

#### Обзор GitVerse

**GitVerse** - российская альтернатива GitHub с фокусом на соответствие российскому законодательству.

#### Основные возможности:

1. **Интеграция с российскими сервисами:**
   - Яндекс.Облако
   - VK Cloud
   - Selectel
   - Timeweb

2. **CI/CD пайплайны:**
   - Автоматическая сборка
   - Тестирование
   - Деплой на различные платформы
   - Уведомления в Telegram

3. **Безопасность:**
   - Шифрование данных
   - Соответствие 152-ФЗ
   - Аудит доступа
   - Резервное копирование

#### Пример конфигурации GitVerse CI/CD:

```yaml
# .gitverse-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  PYTHON_VERSION: "3.12"
  MKDOCS_VERSION: "1.5.3"

build:
  stage: build
  image: python:3.12-slim
  script:
    - pip install mkdocs==$MKDOCS_VERSION mkdocs-material
    - mkdocs build
  artifacts:
    paths:
      - site/
    expire_in: 1 hour

test:
  stage: test
  image: python:3.12-slim
  script:
    - pip install mkdocs==$MKDOCS_VERSION mkdocs-material
    - mkdocs build --strict
    - echo "Build successful"

deploy_helios:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache openssh-client rsync
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - echo "$SSH_KNOWN_HOSTS" >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
    - rsync -avz --delete site/ $HELIOS_USER@$HELIOS_HOST:$HELIOS_PATH/
  only:
    - main
  when: manual
```

#### Преимущества GitVerse для российских проектов:

1. **Соответствие законодательству:**
   - Хранение данных в России
   - Соответствие 152-ФЗ
   - Аудит и логирование

2. **Интеграция с экосистемой:**
   - Прямая интеграция с российскими хостингами
   - Поддержка российских платежных систем
   - Локализованная поддержка

3. **Безопасность:**
   - Шифрование трафика
   - Защита от утечек
   - Контроль доступа

### Варианты деплоя статического сайта в продакшен среду

#### 1. GitHub Pages

**Технические инструменты:**
- GitHub Actions
- Jekyll (встроенный)
- MkDocs, Sphinx, Docusaurus

**Пример конфигурации:**
```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: 3.12
      - run: pip install mkdocs
      - run: mkdocs build
      - uses: actions/deploy-pages@v2
```

**Преимущества:**
- Бесплатно
- Автоматический SSL
- Интеграция с Git
- CDN от GitHub

**Недостатки:**
- Ограничения на размер (1 ГБ)
- Только статические сайты
- Зависимость от GitHub

#### 2. Netlify

**Технические инструменты:**
- Netlify CLI
- Netlify Functions
- Edge Functions

**Пример конфигурации:**
```toml
# netlify.toml
[build]
  command = "mkdocs build"
  publish = "site"

[build.environment]
  PYTHON_VERSION = "3.12"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

**Преимущества:**
- Простая настройка
- Превью для PR
- Формы из коробки
- Edge Functions

**Недостатки:**
- Ограничения на бесплатном тарифе
- Зависимость от сервиса

#### 3. Vercel

**Технические инструменты:**
- Vercel CLI
- Serverless Functions
- Edge Runtime

**Пример конфигурации:**
```json
{
  "buildCommand": "mkdocs build",
  "outputDirectory": "site",
  "framework": null,
  "installCommand": "pip install mkdocs"
}
```

**Преимущества:**
- Отличная производительность
- Автоматический деплой
- Edge Functions
- Аналитика

**Недостатки:**
- Ограничения на бесплатном тарифе
- Сложность для больших проектов

#### 4. Собственный сервер

**Технические инструменты:**
- Nginx/Apache
- Docker
- rsync/scp
- Let's Encrypt

**Пример настройки Nginx:**
```nginx
server {
    listen 80;
    server_name your-site.com;
    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

**Пример Dockerfile:**
```dockerfile
FROM nginx:alpine
COPY site/ /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Преимущества:**
- Полный контроль
- Неограниченные возможности
- Кастомизация
- Безопасность

**Недостатки:**
- Необходимость администрирования
- Затраты на сервер
- Настройка мониторинга

#### 5. Яндекс.Облако

**Технические инструменты:**
- Object Storage
- CDN
- Compute Cloud
- Terraform

**Пример конфигурации Terraform:**
```hcl
resource "yandex_storage_bucket" "website" {
  bucket = "my-static-website"
  acl    = "public-read"
  
  website {
    index_document = "index.html"
    error_document = "error.html"
  }
}

resource "yandex_cdn_origin_group" "my_group" {
  name = "my-origin-group"
  
  origin {
    source = yandex_storage_bucket.website.website_endpoint
  }
}
```

**Преимущества:**
- Российская юрисдикция
- Интеграция с экосистемой
- CDN
- Object Storage

**Недостатки:**
- Сложность настройки
- Платный сервис

#### 6. Helios (российский хостинг)

**Технические инструменты:**
- SSH доступ
- rsync
- Git hooks
- Cron jobs

**Пример деплоя через rsync:**
```bash
#!/bin/bash
# deploy.sh
rsync -avz --delete \
  --exclude='.git' \
  --exclude='.env' \
  ./site/ \
  user@helios-server:/var/www/html/
```

**Преимущества:**
- Российская юрисдикция
- Поддержка PHP, Python, Node.js
- Домены .ru
- Техподдержка на русском

**Недостатки:**
- Платный сервис
- Ограниченные возможности

## Технические инструменты для деплоя

### CI/CD платформы

1. **GitHub Actions**
   - Интеграция с GitHub
   - Богатая экосистема
   - Бесплатно для публичных репозиториев

2. **GitLab CI**
   - Мощные возможности
   - Встроенный Docker registry
   - Бесплатно для публичных проектов

3. **Jenkins**
   - Гибкость
   - Множество плагинов
   - Локальная установка

4. **GitVerse**
   - Российская альтернатива
   - Соответствие законодательству
   - Интеграция с российскими сервисами

### Инструменты деплоя

1. **rsync**
   - Синхронизация файлов
   - Инкрементальные обновления
   - Поддержка SSH

2. **scp**
   - Безопасное копирование
   - Простота использования
   - Интеграция с SSH

3. **Docker**
   - Контейнеризация
   - Изоляция окружения
   - Масштабируемость

4. **Terraform**
   - Инфраструктура как код
   - Автоматизация
   - Версионирование

### Мониторинг и аналитика

1. **Google Analytics**
   - Аналитика трафика
   - Поведенческие метрики
   - Интеграция с AdSense

2. **Яндекс.Метрика**
   - Российская альтернатива
   - Локализованная аналитика
   - Интеграция с Яндекс.Директ

3. **Uptime Robot**
   - Мониторинг доступности
   - Уведомления
   - Статистика uptime

## Рекомендации по выбору решения

### Для российских проектов

1. **Хостинг**: Яндекс.Облако или Helios
2. **CDN**: Яндекс.Облако CDN или Qrator
3. **CI/CD**: GitVerse или GitLab
4. **Аналитика**: Яндекс.Метрика

### Для международных проектов

1. **Хостинг**: GitHub Pages, Netlify, Vercel
2. **CDN**: Cloudflare, AWS CloudFront
3. **CI/CD**: GitHub Actions, GitLab CI
4. **Аналитика**: Google Analytics

### Для корпоративных проектов

1. **Хостинг**: Собственный сервер или облако
2. **CDN**: Корпоративное решение
3. **CI/CD**: Jenkins или GitLab
4. **Мониторинг**: Prometheus + Grafana

## Заключение

Проект успешно демонстрирует возможности современных инструментов для создания и развертывания статических сайтов. Автоматизация процесса деплоя значительно упрощает поддержку и обновление контента.

### Ключевые достижения:

✅ Создан статический сайт на MkDocs  
✅ Настроен автоматический деплой на GitHub Pages  
✅ Исследованы альтернативные платформы и инструменты  
✅ Проанализированы возможности отечественных CDN  
✅ Изучены возможности GitVerse для CI/CD  
✅ Документированы варианты деплоя в продакшен  

**Ссылка на сайт**: https://krllkrnv.github.io/mkdocs-test