# تطبيق المحاضرة الثانية يدويًا في مشروع FastAPI

هذا دليل للانتقال من المحاضرة الأولى إلى الحالة النهائية المنشورة للمحاضرة الثانية من Corey Schafer. المشروع المستخدم هو `C:\Users\Pc\Desktop\FastAPI-Lesson-01`.

المصدر: https://github.com/CoreyMSchafer/FastAPI-02-Templates

الفيديو: https://www.youtube.com/watch?v=G4NIB9Rx9Qs

Commit المصدر: `c44eb650eceaeb163e79c99dd2f6337ad6fd8870`.

ملف التطبيق والقوالب وملفات الواجهة تطابق هذه النسخة المنشورة. لم تُضف قاعدة بيانات أومصادقة أومسارات لمحاضرات لاحقة. Python بقي 3.13 ليتوافق مع البيئة المثبتة.

## ما الذي تغير

- تغير `main.py` لعرض صفحة باستخدام Jinja2 بدل إرجاع عنوان واحد كـHTML مباشرة.
- أضيف `templates/layout.html` للإطار العام و`templates/home.html` لمحتوى الصفحة.
- أضيف مجلد `static` بكل ملفاته الأصلية: CSS وصور وأيقونات وManifest وملف JavaScript فارغ.
- أضيف الملفان المساعدان `home_finished.html` و`layout_finished.html` وحدث `snippets.txt` كما في المستودع؛ ليست هذه الملفات قوالب التطبيق النشطة.
- لم يتغير `pyproject.toml` أو`uv.lock`؛ Jinja2 موجودة ضمن الاعتمادات المثبتة.

## 1 إيقاف جلسة التطوير

إذا كانت نافذة Uvicorn تعمل من تجربة سابقة، اضغط `Ctrl+C` داخلها قبل تعديل الملفات. لم أوقف أي عملية خاصة بك؛ استُخدم منفذ اختبار مستقل عند التحقق.

## 2 دخول المشروع

```powershell
Set-Location -LiteralPath 'C:\Users\Pc\Desktop\FastAPI-Lesson-01'
```

## 3 نسخة احتياطية

هذه الأوامر تحفظ ملفات المحاضرة الأولى الأساسية في أرشيف جديد دون نسخ البيئة الافتراضية:

```powershell
New-Item -ItemType Directory -Path .\backups -Force | Out-Null
$backupPath = Join-Path (Join-Path (Get-Location) 'backups') ('lesson-01-' + (Get-Date -Format 'yyyyMMdd-HHmmss-fff') + '.zip')
$backupFiles = @('main.py','pyproject.toml','uv.lock','.python-version','.gitignore','snippets.txt','README.md','README_AR.md','SOURCE.md') | Where-Object { Test-Path -LiteralPath $_ }
Compress-Archive -LiteralPath $backupFiles -DestinationPath $backupPath
```

وقت تنفيذ التحديث الفعلي حُفظت نسخة احتياطية تشمل ملفات المشروع الحالية، مع استثناء `.venv` و`__pycache__` و`.git` والنسخ الاحتياطية نفسها. إذا طبقت الأوامر بعد التحديث فستحفظ الحالة الحالية لا حالة قديمة لم تعد موجودة؛ نسخة ما قبل التحديث محفوظة مسبقًا داخل `backups`.

## 4 إنشاء المجلدات

```powershell
New-Item -ItemType Directory -Path .\templates -Force | Out-Null
New-Item -ItemType Directory -Path .\static\css -Force | Out-Null
New-Item -ItemType Directory -Path .\static\js -Force | Out-Null
New-Item -ItemType Directory -Path .\static\icons -Force | Out-Null
New-Item -ItemType Directory -Path .\static\profile_pics -Force | Out-Null
```

## 5 كتابة الملفات النصية

لكل ملف أدناه: شغل أمر المحرر، واكتب المحتوى الكامل، ثم احفظ باستخدام `Ctrl+S`. اختر UTF-8 عند الحفظ. المسافات البادئة في Python مهمة. الصور والملفات الثنائية لها خطوة منفصلة لاحقًا.

### الملف main.py

ينشئ التطبيق، ويربط /static بالمجلد المحلي، ويحدد محرك Jinja2، ويمرر posts وtitle إلى home.html. مسار JSON بقي كما كان.

```powershell
notepad .\main.py
```

```python
from fastapi import FastAPI, Request
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

app = FastAPI()

app.mount("/static", StaticFiles(directory="static"), name="static")

templates = Jinja2Templates(directory="templates")

posts: list[dict] = [
    {
        "id": 1,
        "author": "Corey Schafer",
        "title": "FastAPI is Awesome",
        "content": "This framework is really easy to use and super fast.",
        "date_posted": "April 20, 2025",
    },
    {
        "id": 2,
        "author": "Jane Doe",
        "title": "Python is Great for Web Development",
        "content": "Python is a great language for web development, and FastAPI makes it even better.",
        "date_posted": "April 21, 2025",
    },
]


@app.get("/", include_in_schema=False, name="home")
@app.get("/posts", include_in_schema=False, name="posts")
def home(request: Request):
    return templates.TemplateResponse(
        request,
        "home.html",
        {"posts": posts, "title": "Home"},
    )


@app.get("/api/posts")
def get_posts():
    return posts
```

### الملف templates/home.html

يرث layout.html، ويملأ block content، ويكرر بطاقات المنشورات. النقطة في post.title تصل هنا إلى بيانات القاموس بواسطة Jinja2.

```powershell
notepad .\templates\home.html
```

```html
{% extends "layout.html" %}
{% block content %}
  {% for post in posts %}
    <article class="content-section py-3 px-4 mb-4">
      <div class="d-flex align-items-start gap-4">
        <img class="rounded-circle article-img flex-shrink-0"
             src="{{ url_for('static', path='profile_pics/default.jpg') }}"
             alt="{{ post.author }}'s profile picture"
             width="64"
             height="64"
             loading="lazy">
        <div class="flex-grow-1">
          <div class="article-metadata mb-2">
            <a class="me-2" href="#">{{ post.author }}</a>
            <small class="text-body-secondary">{{ post.date_posted }}</small>
          </div>
          <h2>
            <a class="article-title" href="#">{{ post.title }}</a>
          </h2>
          <p class="article-content">{{ post.content }}</p>
        </div>
      </div>
    </article>
  {% endfor %}
{% endblock content %}
```

### الملف templates/layout.html

الهيكل العام: head، وBootstrap والخطوط، وشريط التنقل، ومنطقة المحتوى، والشريط الجانبي، والتذييل، وتبديل الثيم. توجد روابط Login وRegister شكلية فقط.

```powershell
notepad .\templates\layout.html
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- Site Info -->
    {% if title %}
      <title>FastAPI Blog - {{ title }}</title>
    {% else %}
      <title>FastAPI Blog</title>
    {% endif %}
    <meta name="description" content="FastAPI Tutorial">
    <meta name="author" content="Corey Schafer">

    <!-- Open Graph Tags: The title of the page for social media sharing. It can match the title tag or be more descriptive. -->
    <meta property="og:title" content="FastAPI Blog">

    <!-- Open Graph Tags: Typically set to "website" for static sites or "article" for content-heavy pages. -->
    <meta property="og:type" content="website">

    <!-- Open Graph Tags: The URL of the page, used to ensure link previews resolve to the correct page. -->
    <meta property="og:url" content="">

    <!-- Open Graph Tags: URL of an image that represents the page. Useful for link previews. -->
    <meta property="og:image" content="">

    <!-- Open Graph Tags: Provides an alternative text for the image to improve accessibility. -->
    <meta property="og:image:alt" content="">

    <!-- Preconnect for Google Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

    <!-- Custom Font -->
    <link href="https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100..900;1,100..900&family=Nunito:ital,wght@0,200..1000;1,200..1000&display=swap"
          rel="stylesheet">

    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css"
          rel="stylesheet"
          integrity="sha384-sRIl4kxILFvY47J16cr9ZwB07vP4J8+LH7qKQnuqkuIAvNWLzeN8tE5YBujZqJLB"
          crossorigin="anonymous">

    <!-- Stylesheet -->
    <link rel="stylesheet"
          type="text/css"
          href="{{ url_for("static", path='css/main.css') }}">

    <!-- Set a theme color that matches your website's primary color -->
    <meta name="theme-color" content="#527c9f">

    <!-- Favicon for all browsers -->
    <link rel="icon"
          href="{{ url_for('static', path='icons/favicon.ico') }}"
          sizes="any">
    <link rel="icon"
          href="{{ url_for('static', path='icons/icon.svg') }}"
          type="image/svg+xml">

    <!-- Apple touch icon for iOS devices -->
    <link rel="apple-touch-icon"
          sizes="180x180"
          href="{{ url_for('static', path='icons/icon.png') }}">
    <!-- Web app manifest for Progressive Web Apps -->
    <link rel="manifest"
          href="{{ url_for('static', path='site.webmanifest') }}">

    <!-- Content Security Policy: Uncomment to enhance security by restricting where content can be loaded from (useful for preventing certain attacks like XSS). Update if adding external sources (e.g., Google Fonts, Bootstrap CDN, analytics, etc). -->
    <!-- <meta http-equiv="Content-Security-Policy" content=" default-src 'self'; script-src 'self' code.jquery.com; style-src 'self' fonts.googleapis.com; font-src fonts.gstatic.com; img-src 'self' images.examplecdn.com; "> -->
  </head>
  <body class="d-flex flex-column min-vh-100">
    <header class="site-header">
      <nav class="navbar navbar-expand-md bg-steel fixed-top"
           data-bs-theme="dark">
        <div class="container">
          <a class="navbar-brand me-4" href="{{ url_for("home") }}">FastAPI Blog</a>
          <button class="navbar-toggler"
                  type="button"
                  data-bs-toggle="collapse"
                  data-bs-target="#navbarToggle"
                  aria-controls="navbarToggle"
                  aria-expanded="false"
                  aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
          </button>
          <div class="collapse navbar-collapse" id="navbarToggle">
            <div class="navbar-nav me-auto">
              <a class="nav-link active"
                 aria-current="page"
                 href="{{ url_for("home") }}">Home</a>
            </div>
            <!-- Navbar Right Side -->
            <div class="navbar-nav">
              <a class="btn btn-outline-light mb-2 mb-md-0 me-md-2" href="#">Login</a>
              <a class="btn btn-light mb-2 mb-md-0 me-md-3" href="#">Register</a>
              <div class="nav-item dropdown">
                <a class="nav-link dropdown-toggle"
                   href="#"
                   role="button"
                   id="bd-theme"
                   data-bs-toggle="dropdown"
                   aria-expanded="false">
                  <span id="bd-theme-text">🌗 Auto</span>
                </a>
                <ul class="dropdown-menu dropdown-menu-end" aria-labelledby="bd-theme">
                  <li>
                    <button class="dropdown-item" type="button" data-bs-theme-value="light">🌝 Light</button>
                  </li>
                  <li>
                    <button class="dropdown-item" type="button" data-bs-theme-value="dark">🌚 Dark</button>
                  </li>
                  <li>
                    <button class="dropdown-item" type="button" data-bs-theme-value="auto">🌗 Auto</button>
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </div>
      </nav>
    </header>

    <main role="main" class="container">
      <div class="row">
        <div class="col-md-8">
          {% block content %}
          {% endblock content %}
        </div>
        <aside class="col-md-4">
          <div class="content-section py-3 px-4 mb-4">
            <h3>Our Sidebar</h3>
            <p class="text-body-secondary">You can put any information here you'd like.</p>
            <ul class="list-group">
              <li class="list-group-item">Latest Posts</li>
              <li class="list-group-item">Announcements</li>
              <li class="list-group-item">Calendars</li>
              <li class="list-group-item">etc</li>
            </ul>
          </div>
        </aside>
      </div>
    </main>

    <footer class="mt-auto py-3 bg-body-tertiary border-top">
      <div class="container text-center">
        <p class="text-body-secondary mb-0">
          © <span id="year"></span> Corey Schafer
        </p>
      </div>
    </footer>

    <!-- Set the current year in the footer -->
    <script>document.getElementById('year').textContent = new Date().getFullYear();</script>

    <!-- Bootstrap Bundle with Popper -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/js/bootstrap.bundle.min.js"
            integrity="sha384-FKyoEForCGlyvwx9Hj09JcYn3nv7wiPVlz7YYwJrWVcXK/BmnVDxM+D2scQbITxI"
            crossorigin="anonymous"></script>

    <!-- Dark Mode Toggle -->
    <script>
      const setTheme = (theme) => {
        if (theme === 'auto') {
          document.documentElement.setAttribute('data-bs-theme', 
            window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
        } else {
          document.documentElement.setAttribute('data-bs-theme', theme);
        }
        localStorage.setItem('theme', theme);
        
        // Update button text
        const themeText = { light: '🌝 Light', dark: '🌚 Dark', auto: '🌗 Auto' };
        document.getElementById('bd-theme-text').textContent = themeText[theme];
      };

      // Set theme on click
      document.querySelectorAll('[data-bs-theme-value]').forEach(button => {
        button.addEventListener('click', () => setTheme(button.getAttribute('data-bs-theme-value')));
      });

      // Initialize
      setTheme(localStorage.getItem('theme') || 'auto');
    </script>
  </body>
</html>
```

### الملف static/css/main.css

تنسيقات المصدر الأصلية، بما فيها الألوان والبطاقات والصور والثيم الداكن وقواعد مشتركة قد تستعمل لاحقًا. لا تمثل إضافة وظائف backend.

```powershell
notepad .\static\css\main.css
```

```css
:root {
  /* Fonts */
  --font-heading: "Montserrat", system-ui, -apple-system, BlinkMacSystemFont,
    sans-serif;
  --font-body: "Nunito", system-ui, -apple-system, BlinkMacSystemFont,
    sans-serif;

  /* Colors */
  --color-bg-body: #fafafa;
  --color-bg-content: #ffffff;
  --color-text-primary: #333333;
  --color-text-heading: #444444;
  --color-steel: #527c9f;
  --color-nav-link: #cbd5db;
  --color-nav-link-hover: #ffffff;
  --color-link-hover: #428bca;
  --color-border: #dddddd;
  --color-border-light: #e3e3e3;

  /* Spacing */
  --spacing-xl: 5rem;

  /* Sizing */
  --article-img-size: 4rem; /* 64px converted to rem */
  --account-img-size: 7.813rem; /* 125px converted to rem */
}

/* Base Styles */
body {
  font-family: var(--font-body);
  background: var(--color-bg-body);
  color: var(--color-text-primary);
}

main {
  padding-top: var(--spacing-xl);
}

h1,
h2,
h3,
h4,
h5,
h6 {
  font-family: var(--font-heading);
  color: var(--color-text-heading);
}

.navbar-brand {
  font-family: var(--font-heading);
}

a {
  text-decoration: none;
}

a:focus-visible,
button:focus-visible {
  outline: 2px solid var(--color-link-hover);
  outline-offset: 2px;
}

/* Headers and Navigation */
.bg-steel {
  background-color: var(--color-steel);
}

.site-header .navbar-nav .nav-link {
  color: var(--color-nav-link);
  transition: color 0.2s ease-in-out;
}

.site-header .navbar-nav .nav-link:hover,
.site-header .navbar-nav .nav-link:focus {
  color: var(--color-nav-link-hover);
}

.site-header .navbar-nav .nav-link.active {
  font-weight: 600;
}

/* Content Sections */
.content-section {
  background: var(--color-bg-content);
  border: 1px solid var(--color-border);
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(15, 23, 42, 0.05);
  transition: box-shadow 0.15s ease, transform 0.15s ease,
    border-color 0.15s ease;
}

.content-section:hover {
  box-shadow: 0 4px 10px rgba(15, 23, 42, 0.1);
  border-color: var(--color-border-light);
}

/* Articles */
.article-title {
  color: var(--color-text-heading);
  transition: color 0.2s ease-in-out;
}

a.article-title:hover,
a.article-title:focus {
  color: var(--color-link-hover);
  text-decoration: none;
}

.article-content {
  white-space: pre-line;
  font-size: 1.25rem;
}

.article-img {
  height: var(--article-img-size);
  width: var(--article-img-size);
  flex-shrink: 0;
  object-fit: cover;
  border: 1px solid var(--color-border-light);
  box-shadow: 0 0 0 2px var(--color-bg-content);
}

.article-metadata {
  padding-bottom: 0.063rem; /* 1px converted to rem */
  border-bottom: 1px solid var(--color-border-light);
}

.article-metadata a {
  transition: color 0.2s ease-in-out;
}

.article-metadata a:hover,
.article-metadata a:focus {
  color: var(--color-text-primary);
  text-decoration: none;
}

.article-svg {
  width: 1.563rem; /* 25px converted to rem */
  height: 1.563rem;
  vertical-align: middle;
}

/* Account/Profile */
.account-img {
  height: var(--account-img-size);
  width: var(--account-img-size);
  object-fit: cover;
}

.account-heading {
  font-size: 2.5rem;
}

.file-input-sm {
  max-width: 300px;
}

/* Custom Button Colors - Light Mode */
.btn-outline-primary {
  --bs-btn-color: #527c9f;
  --bs-btn-border-color: #527c9f;
  --bs-btn-hover-color: #fff;
  --bs-btn-hover-bg: #466a87;
  --bs-btn-hover-border-color: #3f6179;
  --bs-btn-active-color: #fff;
  --bs-btn-active-bg: #3f6179;
  --bs-btn-active-border-color: #3a5b72;
}

.btn-outline-secondary {
  --bs-btn-color: #495057;
  --bs-btn-border-color: #6c757d;
  --bs-btn-hover-color: #fff;
  --bs-btn-hover-bg: #5a6268;
  --bs-btn-hover-border-color: #545b62;
  --bs-btn-active-color: #fff;
  --bs-btn-active-bg: #545b62;
  --bs-btn-active-border-color: #4e555b;
}

.btn-outline-danger {
  --bs-btn-color: #a85b5b;
  --bs-btn-border-color: #a85b5b;
  --bs-btn-hover-color: #fff;
  --bs-btn-hover-bg: #924e4e;
  --bs-btn-hover-border-color: #874848;
  --bs-btn-active-color: #fff;
  --bs-btn-active-bg: #874848;
  --bs-btn-active-border-color: #7c4242;
}

.btn-primary {
  --bs-btn-color: #fff;
  --bs-btn-bg: #527c9f;
  --bs-btn-border-color: #527c9f;
  --bs-btn-hover-color: #fff;
  --bs-btn-hover-bg: #466a87;
  --bs-btn-hover-border-color: #3f6179;
  --bs-btn-active-color: #fff;
  --bs-btn-active-bg: #3f6179;
  --bs-btn-active-border-color: #3a5b72;
}

.btn-danger {
  --bs-btn-color: #fff;
  --bs-btn-bg: #a85b5b;
  --bs-btn-border-color: #a85b5b;
  --bs-btn-hover-color: #fff;
  --bs-btn-hover-bg: #924e4e;
  --bs-btn-hover-border-color: #874848;
  --bs-btn-active-color: #fff;
  --bs-btn-active-bg: #874848;
  --bs-btn-active-border-color: #7c4242;
}

/* Custom Button Colors - Dark Mode */
[data-bs-theme="dark"] .btn-outline-primary {
  --bs-btn-color: #8aadc8;
  --bs-btn-border-color: #8aadc8;
  --bs-btn-hover-color: #000;
  --bs-btn-hover-bg: #8aadc8;
  --bs-btn-hover-border-color: #9fbdd4;
  --bs-btn-active-color: #000;
  --bs-btn-active-bg: #9fbdd4;
  --bs-btn-active-border-color: #b4cde0;
}

[data-bs-theme="dark"] .btn-outline-secondary {
  --bs-btn-color: #b8bfc6;
  --bs-btn-border-color: #8a939b;
  --bs-btn-hover-color: #000;
  --bs-btn-hover-bg: #b8bfc6;
  --bs-btn-hover-border-color: #adb5bd;
  --bs-btn-active-color: #000;
  --bs-btn-active-bg: #adb5bd;
  --bs-btn-active-border-color: #a2aab2;
}

[data-bs-theme="dark"] .btn-outline-danger {
  --bs-btn-color: #d89090;
  --bs-btn-border-color: #d89090;
  --bs-btn-hover-color: #000;
  --bs-btn-hover-bg: #d89090;
  --bs-btn-hover-border-color: #e0a8a8;
  --bs-btn-active-color: #000;
  --bs-btn-active-bg: #e0a8a8;
  --bs-btn-active-border-color: #e8bcbc;
}

/* Responsive Adjustments */
@media (max-width: 768px) {
  .account-heading {
    font-size: 2rem;
  }

  .article-img {
    height: 3rem;
    width: 3rem;
  }
}

/* Dark Mode Features */

[data-bs-theme="dark"] {
  --color-bg-body: #1a1a1a;
  --color-bg-content: #2d2d2d;
  --color-text-primary: #e0e0e0;
  --color-text-heading: #f0f0f0;
  --color-border: #404040;
  --color-border-light: #4a4a4a;
}

[data-bs-theme="dark"] .content-section {
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
}

[data-bs-theme="dark"] article.content-section:hover {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.5);
}

/* Custom Modal Colors */
.modal-header.bg-success {
  background-color: #4a7c5d !important;
}

.modal-header.bg-danger {
  background-color: #a85b5b !important;
}

/* Ensure modal header text stays white in both themes */
.modal-header.bg-success .modal-title,
.modal-header.bg-danger .modal-title {
  color: #ffffff !important;
}
```

### الملف static/site.webmanifest

بيانات اسم التطبيق والأيقونات ومظهره المقترح عند الإضافة إلى الشاشة الرئيسية. وجوده وحده لا يجعل التطبيق يعمل دون اتصال.

```powershell
notepad .\static\site.webmanifest
```

```json
{
  "name": "FastAPI Blog",
  "short_name": "FastAPI Blog",
  "description": "FastAPI Tutorial",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#527c9f",
  "icons": [
    {
      "src": "/static/icons/icon.png",
      "sizes": "180x180",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/static/icons/icon.svg",
      "sizes": "any",
      "type": "image/svg+xml",
      "purpose": "any"
    },
    {
      "src": "/static/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/static/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ]
}
```

### الملف snippets.txt

مثال قالب مبسط مساعد في مواد الدرس. لا يقرأه التطبيق عند معالجة الطلب.

```powershell
notepad .\snippets.txt
```

```html
## Home Template
<!DOCTYPE html>
<html>
  <head>
    <title>FastAPI Blog</title>
  </head>
  <body>
    <h1>Home Page</h1>
    <p>This is using a template!</p>
  </body>
</html>
```

### الملف home_finished.html

ملف مساعد نهائي من المصدر. القالب النشط فعليًا هو templates/home.html.

```powershell
notepad .\home_finished.html
```

```html
{% extends "layout.html" %}
{% block content %}
  {% for post in posts %}
    <article class="content-section py-3 px-4 mb-4">
      <div class="d-flex align-items-start gap-4">
        <img class="rounded-circle article-img flex-shrink-0"
             src="/static/profile_pics/default.jpg"
             alt="{{ post.author }}'s profile picture"
             width="64"
             height="64"
             loading="lazy">
        <div class="flex-grow-1">
          <div class="article-metadata mb-2">
            <a class="me-2" href="#">{{ post.author }}</a>
            <small class="text-body-secondary">{{ post.date_posted }}</small>
          </div>
          <h2>
            <a class="article-title" href="#">{{ post.title }}</a>
          </h2>
          <p class="article-content">{{ post.content }}</p>
        </div>
      </div>
    </article>
  {% endfor %}
{% endblock content %}
```

### الملف layout_finished.html

ملف مساعد نهائي من المصدر. القالب النشط فعليًا هو templates/layout.html.

```powershell
notepad .\layout_finished.html
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- Site Info -->
    {% if title %}
      <title>FastAPI Blog - {{ title }}</title>
    {% else %}
      <title>FastAPI Blog</title>
    {% endif %}
    <meta name="description" content="FastAPI Tutorial">
    <meta name="author" content="Corey Schafer">

    <!-- Open Graph Tags: The title of the page for social media sharing. It can match the title tag or be more descriptive. -->
    <meta property="og:title" content="FastAPI Blog">

    <!-- Open Graph Tags: Typically set to "website" for static sites or "article" for content-heavy pages. -->
    <meta property="og:type" content="website">

    <!-- Open Graph Tags: The URL of the page, used to ensure link previews resolve to the correct page. -->
    <meta property="og:url" content="">

    <!-- Open Graph Tags: URL of an image that represents the page. Useful for link previews. -->
    <meta property="og:image" content="">

    <!-- Open Graph Tags: Provides an alternative text for the image to improve accessibility. -->
    <meta property="og:image:alt" content="">

    <!-- Preconnect for Google Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

    <!-- Custom Font -->
    <link href="https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100..900;1,100..900&family=Nunito:ital,wght@0,200..1000;1,200..1000&display=swap"
          rel="stylesheet">

    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css"
          rel="stylesheet"
          integrity="sha384-sRIl4kxILFvY47J16cr9ZwB07vP4J8+LH7qKQnuqkuIAvNWLzeN8tE5YBujZqJLB"
          crossorigin="anonymous">

    <!-- Stylesheet -->
    <link rel="stylesheet" type="text/css" href="/static/css/main.css">

    <!-- Set a theme color that matches your website's primary color -->
    <meta name="theme-color" content="#527c9f">

    <!-- Favicon for all browsers -->
    <link rel="icon" href="/static/icons/favicon.ico" sizes="any">
    <link rel="icon" href="/static/icons/icon.svg" type="image/svg+xml">

    <!-- Apple touch icon for iOS devices -->
    <link rel="apple-touch-icon" sizes="180x180" href="/static/icons/icon.png">
    <!-- Web app manifest for Progressive Web Apps -->
    <link rel="manifest" href="/static/site.webmanifest">

    <!-- Content Security Policy: Uncomment to enhance security by restricting where content can be loaded from (useful for preventing certain attacks like XSS). Update if adding external sources (e.g., Google Fonts, Bootstrap CDN, analytics, etc). -->
    <!-- <meta http-equiv="Content-Security-Policy" content=" default-src 'self'; script-src 'self' code.jquery.com; style-src 'self' fonts.googleapis.com; font-src fonts.gstatic.com; img-src 'self' images.examplecdn.com; "> -->
  </head>
  <body class="d-flex flex-column min-vh-100">
    <header class="site-header">
      <nav class="navbar navbar-expand-md bg-steel fixed-top"
           data-bs-theme="dark">
        <div class="container">
          <a class="navbar-brand me-4" href="#">FastAPI Blog</a>
          <button class="navbar-toggler"
                  type="button"
                  data-bs-toggle="collapse"
                  data-bs-target="#navbarToggle"
                  aria-controls="navbarToggle"
                  aria-expanded="false"
                  aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
          </button>
          <div class="collapse navbar-collapse" id="navbarToggle">
            <div class="navbar-nav me-auto">
              <a class="nav-link active" aria-current="page" href="#">Home</a>
            </div>
            <!-- Navbar Right Side -->
            <div class="navbar-nav">
              <a class="btn btn-outline-light mb-2 mb-md-0 me-md-2" href="#">Login</a>
              <a class="btn btn-light mb-2 mb-md-0 me-md-3" href="#">Register</a>
              <div class="nav-item dropdown">
                <a class="nav-link dropdown-toggle"
                   href="#"
                   role="button"
                   id="bd-theme"
                   data-bs-toggle="dropdown"
                   aria-expanded="false">
                  <span id="bd-theme-text">🌗 Auto</span>
                </a>
                <ul class="dropdown-menu dropdown-menu-end" aria-labelledby="bd-theme">
                  <li>
                    <button class="dropdown-item" type="button" data-bs-theme-value="light">🌝 Light</button>
                  </li>
                  <li>
                    <button class="dropdown-item" type="button" data-bs-theme-value="dark">🌚 Dark</button>
                  </li>
                  <li>
                    <button class="dropdown-item" type="button" data-bs-theme-value="auto">🌗 Auto</button>
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </div>
      </nav>
    </header>

    <main role="main" class="container">
      <div class="row">
        <div class="col-md-8">
          {% block content %}
          {% endblock content %}
        </div>
        <aside class="col-md-4">
          <div class="content-section py-3 px-4 mb-4">
            <h3>Our Sidebar</h3>
            <p class="text-body-secondary">You can put any information here you'd like.</p>
            <ul class="list-group">
              <li class="list-group-item">Latest Posts</li>
              <li class="list-group-item">Announcements</li>
              <li class="list-group-item">Calendars</li>
              <li class="list-group-item">etc</li>
            </ul>
          </div>
        </aside>
      </div>
    </main>

    <footer class="mt-auto py-3 bg-body-tertiary border-top">
      <div class="container text-center">
        <p class="text-body-secondary mb-0">
          © <span id="year"></span> Corey Schafer
        </p>
      </div>
    </footer>

    <!-- Set the current year in the footer -->
    <script>document.getElementById('year').textContent = new Date().getFullYear();</script>

    <!-- Bootstrap Bundle with Popper -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/js/bootstrap.bundle.min.js"
            integrity="sha384-FKyoEForCGlyvwx9Hj09JcYn3nv7wiPVlz7YYwJrWVcXK/BmnVDxM+D2scQbITxI"
            crossorigin="anonymous"></script>

    <!-- Dark Mode Toggle -->
    <script>
      const setTheme = (theme) => {
        if (theme === 'auto') {
          document.documentElement.setAttribute('data-bs-theme', 
            window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
        } else {
          document.documentElement.setAttribute('data-bs-theme', theme);
        }
        localStorage.setItem('theme', theme);
        
        // Update button text
        const themeText = { light: '🌝 Light', dark: '🌚 Dark', auto: '🌗 Auto' };
        document.getElementById('bd-theme-text').textContent = themeText[theme];
      };

      // Set theme on click
      document.querySelectorAll('[data-bs-theme-value]').forEach(button => {
        button.addEventListener('click', () => setTheme(button.getAttribute('data-bs-theme-value')));
      });

      // Initialize
      setTheme(localStorage.getItem('theme') || 'auto');
    </script>
  </body>
</html>
```

## 6 الصور والأيقونات وملف JavaScript

الصور ليست كودًا نصيًا يمكن كتابته داخل Notepad. الملف `FastAPI-Lesson-02-Assets.zip` المرفق يحتوي مجلد static الأصلي كاملًا. إذا حفظته في Downloads، انقله إلى المشروع بهذه الأوامر:

```powershell
$assetsPath = Join-Path $env:USERPROFILE 'Downloads\FastAPI-Lesson-02-Assets.zip'
Expand-Archive -LiteralPath $assetsPath -DestinationPath (Get-Location).Path -Force
```

عدل المسار إذا حفظت الأرشيف في مكان آخر. هذا استخراج أصول الواجهة المرفقة محليًا، وليس تنزيل مستودع برمجي. قد يعيد استخراج CSS وManifest المكتوبين في الخطوة السابقة؛ المحتوى هو نفسه. يتضمن الأرشيف أيضًا SVG وملف `static/js/utils.js` الفارغ، والأيقونات المتعددة و`original.png` وصورة الحساب الافتراضية.

في نسخة سطح المكتب التي حدثتها، هذه الملفات موجودة بالفعل ولا تحتاج إلى استخراجها مجددًا. يمكنك فحصها:

```powershell
Get-ChildItem -LiteralPath .\static -Recurse -File
```

## 7 الاعتمادات

لا تضف إصدارًا جديدًا من Jinja2 عشوائيًا؛ هي مثبتة بالفعل ضمن fastapi[standard] في ملف القفل الأصلي. للتحقق ومزامنة البيئة:

```powershell
uv sync --locked
.\.venv\Scripts\python.exe -c "import fastapi, jinja2; print('FastAPI:', fastapi.__version__); print('Jinja2:', jinja2.__version__)"
```

## 8 التشغيل

```powershell
.\.venv\Scripts\python.exe -m uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

إذا كان المنفذ مستخدمًا بسبب جلسة لك، أوقفها بـCtrl+C من نافذتها أوشغل على منفذ آخر تختاره. لا توقف عملية مجهولة. افتح `/` و`/posts` و`/api/posts` و`/docs` على عنوان الخادم.

## 9 اختبار يدوي من نافذة PowerShell ثانية

```powershell
$response = Invoke-WebRequest -Uri 'http://127.0.0.1:8000/'
$response.StatusCode
$response.Content.Contains('FastAPI is Awesome')
$response.Content.Contains('Python is Great for Web Development')
$posts = Invoke-RestMethod -Uri 'http://127.0.0.1:8000/api/posts'
$posts.Count
(Invoke-WebRequest -Uri 'http://127.0.0.1:8000/static/css/main.css').StatusCode
(Invoke-WebRequest -Uri 'http://127.0.0.1:8000/static/profile_pics/default.jpg').StatusCode
```

المتوقع بالترتيب: `200` ثم `True` ثم `True` ثم `2` ثم `200` ثم `200`. راجع الواجهة في المتصفح أيضًا، وافتح قائمة الثيم لتجربة Light وDark. تستخدم الصفحة Bootstrap وخطوط Google من الإنترنت؛ قد تتأثر هذه الأجزاء عند انقطاع الاتصال أوحجب CDN، حتى لو استجاب HTML المحلي بنجاح.

## تسلسل التنفيذ

1. يستورد Uvicorn الكائن app من main.py.
2. يُربط /static بمجلد static، ويُجهز محرك Jinja2 بمجلد templates.
3. تسجل الدوال والمسارات، وتجهز قائمة posts في الذاكرة.
4. يصل GET / أوGET /posts إلى home مع كائن Request.
5. تنشئ TemplateResponse استجابة من home.html مع posts وtitle.
6. يجد Jinja2 عبارة extends في home.html، فيستخدم layout.html هيكلًا عامًا ويملأ block content ببطاقات المنشورات.
7. تتحول حلقات for وتعبيرات القيم إلى HTML عادي. وتولد url_for عناوين الموارد والمسارات من أسمائها.
8. يستقبل المتصفح HTML، ثم يطلب CSS والصور والأيقونات والموارد الخارجية في طلبات منفصلة.
9. تنفذ JavaScript الموجودة داخل layout.html تعيين السنة وإدارة اختيار الثيم. ملف utils.js الأصلي فارغ ولا يحتوي هذا المنطق.
10. يبقى GET /api/posts مسارًا يعيد JSON دون الحاجة إلى القوالب.

## شرح المفاهيم الأساسية

- `Request`: معلومات الطلب الحالي التي يحتاجها سياق القالب وتوليد العناوين.
- `StaticFiles`: خدمة ملفات من مجلد تحت بادئة URL محددة. هي لا تعالج Jinja داخل تلك الملفات.
- `Jinja2Templates`: إعداد بيئة البحث عن القوالب وعرضها.
- `TemplateResponse`: ربط الطلب واسم القالب وبيانات العرض لإنتاج استجابة HTML.
- `{{ ... }}`: إظهار قيمة أوتعبير في القالب.
- `{% ... %}`: تعليمات القالب، مثل if وfor وextends وblock.
- `extends`: اختيار قالب أب مشترك.
- `block content`: موضع تستبدله الصفحة الفرعية بمحتواها.
- `url_for`: إنشاء عنوان من اسم مسار، مثل home أوstatic، مع معاملات المورد.
- `name='home'` و`name='posts'`: اسمان مستقلان لمسارين مرتبطين بالدالة نفسها.
- `include_in_schema=False`: إخفاء مساري HTML من مخطط API، مع بقائهما متاحين.

التصميم لا ينفذ تسجيل الدخول أوإنشاء حساب بعد؛ الأزرار وروابط عناوين المنشورات والمؤلفين ذات `href='#'` عناصر عرض في هذه المرحلة. لا توجد قاعدة بيانات ولاCRUD إضافية.

## التنفيذ الفعلي والأوامر القابلة للمراجعة

لضمان مطابقة النسخة المنشورة، جُلب مرجع الدرس للمراجعة بهذا الأمر داخل مجلد العمل:

```powershell
git clone --depth 1 https://github.com/CoreyMSchafer/FastAPI-02-Templates.git work/fastapi-lesson-02-source
```

ثم استُخدم ملف `Apply-FastAPI-Lesson-02.ps1` المرفق لتطبيق المحتوى في المشروع. هذا الملف يحتوي أوامر إنشاء المجلدات وكتابة الملفات النصية كاملة، والأصول الثنائية مشفرة Base64، وإنشاء النسخة الاحتياطية، وتشغيل uv sync --locked. لا ينزل المستودع عند تشغيله. يمكنك قراءته قبل الاستخدام. كتابة الملفات من المحرر بالخطوات السابقة بديل تعليمي لنفس التغييرات، وليست ادعاءً بأن عملية التحديث تمت عبر Notepad.

لا تحتاج إلى تشغيله بعد تطبيق الخطوات اليدوية. التحديث مطبق بالفعل في مشروع سطح المكتب. لإعادة تطبيقه في نسخة تدريبية من المحاضرة الأولى، استخدم المسار المناسب:

```powershell
& 'C:\المسار-الذي-حفظت-فيه\Apply-FastAPI-Lesson-02.ps1' -ProjectPath 'C:\Users\Pc\Desktop\FastAPI-Lesson-01'
```

إذا منعت سياسة PowerShell تشغيله فلا يلزم تغيير إعدادات النظام؛ استخدم خطوات كتابة الملفات يدويًا أعلاه. كما أن Script تتحقق من أن main.py ما زالت نسخة الدرس المتوقعة قبل الكتابة.

## نتائج التحقق بعد التطبيق

- طابقت 20 ملفًا من ملفات المصدر بايتًا ببايت؛ الاستثناء المقصود هو اختيار Python 3.13 المحلي.
- نجحت طلبات HTTP لصفحتي / و/posts، وظهرت بطاقتا المنشورات والعنوان Home دون بقاء تعليمات Jinja غير معالجة.
- نجحت طلبات CSS وManifest وجميع الصور والأيقونات وملف JavaScript، وبقيت بيانات /api/posts صحيحة.
- نجحت صفحات التوثيق ومخطط OpenAPI، وأعاد المسار غير الموجود 404.
- فُحصت الصفحة بصريًا في المتصفح وعمل التبديل بين Light وDark.
- أُعيد تطبيق Script على نسخة مستقلة من المحاضرة الأولى، وطابقت ملفاتها المرجع أيضًا.
- أُوقفت جلسة خادم الاختبار بعد التحقق. شغّل المشروع على المنفذ 8000 بالأمر المذكور لتبدأ جلستك.
