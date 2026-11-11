# تنفيذ المحاضرة الثالثة يدوياً — FastAPI Path Parameters

المشروع الموجود: `C:\Users\Pc\Desktop\FastAPI-Lesson-01`.
هذه ترقية للمحاضرة الثانية، وليست مشروعاً جديداً. أوقف خادم التطوير السابق بـ Ctrl+C قبل البدء.
الكود مطابق لمستودع المدرّس للمحاضرة الثالثة. تبقى Python 3.13 المحلية وملفات dependencies كما كانت.
أزرار Edit/Delete مجرد واجهة في هذه المرحلة. البيانات قائمة في الذاكرة، ولا توجد قاعدة بيانات أو تسجيل دخول.

## 1. افتح PowerShell وخذ نسخة احتياطية

نفّذ التالي مرة واحدة قبل تعديل الملفات. الأرشيف يستبعد البيئة الافتراضية والكاش والأرشيفات السابقة.
يحفظ أيضاً دليل المحاضرة الثانية باسم مستقل.

```powershell
$ErrorActionPreference = 'Stop'
Set-Location -LiteralPath 'C:\Users\Pc\Desktop\FastAPI-Lesson-01'
$backupPath = Join-Path (Get-Location) ('backups\before-lesson-03-' + [guid]::NewGuid().ToString('N') + '.zip')
New-Item -ItemType Directory -Path backups -Force | Out-Null
$items = Get-ChildItem -Force | Where-Object { $_.Name -notin @('.venv', '__pycache__', '.git', 'backups') }
Compress-Archive -LiteralPath $items.FullName -DestinationPath $backupPath
Write-Output "Backup: $backupPath"
Copy-Item -LiteralPath README_AR.md -Destination README_LESSON_02_AR.md
$utf8 = [System.Text.UTF8Encoding]::new($false)

```

## 2. اكتب الملفات السبعة التالية

افتح كل ملف بالأمر الموضّح، استبدل محتواه بالكامل بالكود التالي، ثم احفظه بترميز UTF-8.
لا تعدّل باقي ملفات المحاضرة الثانية. ملفات templates موجودة مسبقاً.

### error_finished.html

نسخة تعليمية جاهزة من قالب الخطأ. لا يحمّلها التطبيق أثناء التشغيل.

```powershell
notepad .\error_finished.html
```

```html
{% extends "layout.html" %}
{% block content %}
  <article class="content-section py-3 px-4 mb-4">
    <h1>
      <a class="article-title" href="#">Oops... {{ status_code }} Error</a>
    </h1>
    <p class="article-content">{{ message }}</p>
  </article>
{% endblock content %}

```

### main.py

تعريف المسارات، البحث عن منشور بالمعرّف، وإرجاع HTML أو JSON، وتخصيص استجابات الأخطاء.

```powershell
notepad .\main.py
```

```python
from fastapi import FastAPI, HTTPException, Request, status
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates
from starlette.exceptions import HTTPException as StarletteHTTPException

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


@app.get("/posts/{post_id}", include_in_schema=False)
def post_page(request: Request, post_id: int):
    for post in posts:
        if post.get("id") == post_id:
            title = post["title"][:50]
            return templates.TemplateResponse(
                request,
                "post.html",
                {"post": post, "title": title},
            )
    raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Post not found")


@app.get("/api/posts")
def get_posts():
    return posts


@app.get("/api/posts/{post_id}")
def get_post(post_id: int):
    for post in posts:
        if post.get("id") == post_id:
            return post
    raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Post not found")


@app.exception_handler(StarletteHTTPException)
def general_http_exception_handler(request: Request, exception: StarletteHTTPException):
    message = (
        exception.detail
        if exception.detail
        else "An error occurred. Please check your request and try again."
    )

    if request.url.path.startswith("/api"):
        return JSONResponse(
            status_code=exception.status_code,
            content={"detail": message},
        )

    return templates.TemplateResponse(
        request,
        "error.html",
        {
            "status_code": exception.status_code,
            "title": exception.status_code,
            "message": message,
        },
        status_code=exception.status_code,
    )


@app.exception_handler(RequestValidationError)
def validation_exception_handler(request: Request, exception: RequestValidationError):
    if request.url.path.startswith("/api"):
        return JSONResponse(
            status_code=status.HTTP_422_UNPROCESSABLE_CONTENT,
            content={"detail": exception.errors()},
        )

    return templates.TemplateResponse(
        request,
        "error.html",
        {
            "status_code": status.HTTP_422_UNPROCESSABLE_CONTENT,
            "title": status.HTTP_422_UNPROCESSABLE_CONTENT,
            "message": "Invalid request. Please check your input and try again.",
        },
        status_code=status.HTTP_422_UNPROCESSABLE_CONTENT,
    )

```

### post_finished.html

نسخة تعليمية جاهزة من قالب المنشور. لا يحمّلها التطبيق أثناء التشغيل.

```powershell
notepad .\post_finished.html
```

```html
{% extends "layout.html" %}
{% block content %}
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
        <h2 class="article-title">{{ post.title }}</h2>
        <p class="article-content">{{ post.content }}</p>
        <!-- Only show edit/delete options if the user is the author -->
        <!-- TODO: Add user authentication and authorization -->
        <div class="post-actions mt-3 pt-3 border-top">
          <a class="btn btn-outline-secondary me-1" href="#">Edit Post</a>
          <button type="button"
                  class="btn btn-outline-danger"
                  data-bs-toggle="modal"
                  data-bs-target="#deleteModal">Delete Post</button>
        </div>
        <!-- TODO: Add user authentication and authorization -->
      </div>
    </div>
  </article>

  <!-- Delete Confirmation Modal -->
  <div class="modal fade"
       id="deleteModal"
       tabindex="-1"
       aria-labelledby="deleteModalLabel"
       aria-hidden="true">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="deleteModalLabel">Delete Post?</h5>
          <button type="button"
                  class="btn-close"
                  data-bs-dismiss="modal"
                  aria-label="Close"></button>
        </div>
        <!-- TODO: <form action="#" method="post"> -->
        <form action="#">
          <div class="modal-body">Are you sure you want to delete this post? This action cannot be undone.</div>
          <div class="modal-footer">
            <button type="button"
                    class="btn btn-outline-secondary"
                    data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-outline-danger">Delete</button>
          </div>
        </form>
      </div>
    </div>
  </div>
{% endblock content %}

```

### snippets.txt

مقاطع معالجات الأخطاء كمادة مساعدة؛ لا ينفذ Python هذا الملف.

```powershell
notepad .\snippets.txt
```

```text
## StarletteHTTPException Handler
@app.exception_handler(StarletteHTTPException)
def general_http_exception_handler(request: Request, exception: StarletteHTTPException):
    message = (
        exception.detail
        if exception.detail
        else "An error occurred. Please check your request and try again."
    )

    if request.url.path.startswith("/api"):
        return JSONResponse(
            status_code=exception.status_code,
            content={"detail": message},
        )
    return templates.TemplateResponse(
        request,
        "error.html",
        {
            "status_code": exception.status_code,
            "title": exception.status_code,
            "message": message,
        },
        status_code=exception.status_code,
    )


### RequestValidationError Handler
@app.exception_handler(RequestValidationError)
def validation_exception_handler(request: Request, exception: RequestValidationError):
    if request.url.path.startswith("/api"):
        return JSONResponse(
            status_code=status.HTTP_422_UNPROCESSABLE_CONTENT,
            content={"detail": exception.errors()},
        )
    return templates.TemplateResponse(
        request,
        "error.html",
        {
            "status_code": status.HTTP_422_UNPROCESSABLE_CONTENT,
            "title": status.HTTP_422_UNPROCESSABLE_CONTENT,
            "message": "Invalid request. Please check your input and try again.",
        },
        status_code=status.HTTP_422_UNPROCESSABLE_CONTENT,
    )

```

### templates/error.html

عرض رمز الخطأ ورسالته داخل layout.html.

```powershell
notepad .\templates\error.html
```

```html
{% extends "layout.html" %}
{% block content %}
  <article class="content-section py-3 px-4 mb-4">
    <h1>
      <a class="article-title" href="#">Oops... {{ status_code }} Error</a>
    </h1>
    <p class="article-content">{{ message }}</p>
  </article>
{% endblock content %}

```

### templates/home.html

ربط عنوان كل منشور بصفحته عبر url_for واسم الدالة post_page.

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
            <a class="article-title"
               href="{{ url_for("post_page", post_id=post.id) }}">{{ post.title }}</a>
          </h2>
          <p class="article-content">{{ post.content }}</p>
        </div>
      </div>
    </article>
  {% endfor %}
{% endblock content %}

```

### templates/post.html

عرض منشور واحد مع نافذة تأكيد حذف تمهيدية؛ لا يوجد تنفيذ للحذف.

```powershell
notepad .\templates\post.html
```

```html
{% extends "layout.html" %}
{% block content %}
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
        <h2 class="article-title">{{ post.title }}</h2>
        <p class="article-content">{{ post.content }}</p>
        <!-- Only show edit/delete options if the user is the author -->
        <!-- TODO: Add user authentication and authorization -->
        <div class="post-actions mt-3 pt-3 border-top">
          <a class="btn btn-outline-secondary me-1" href="#">Edit Post</a>
          <button type="button"
                  class="btn btn-outline-danger"
                  data-bs-toggle="modal"
                  data-bs-target="#deleteModal">Delete Post</button>
        </div>
        <!-- TODO: Add user authentication and authorization -->
      </div>
    </div>
  </article>

  <!-- Delete Confirmation Modal -->
  <div class="modal fade"
       id="deleteModal"
       tabindex="-1"
       aria-labelledby="deleteModalLabel"
       aria-hidden="true">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="deleteModalLabel">Delete Post?</h5>
          <button type="button"
                  class="btn-close"
                  data-bs-dismiss="modal"
                  aria-label="Close"></button>
        </div>
        <!-- TODO: <form action="#" method="post"> -->
        <form action="#">
          <div class="modal-body">Are you sure you want to delete this post? This action cannot be undone.</div>
          <div class="modal-footer">
            <button type="button"
                    class="btn btn-outline-secondary"
                    data-bs-dismiss="modal">Cancel</button>
            <button type="submit" class="btn btn-outline-danger">Delete</button>
          </div>
        </form>
      </div>
    </div>
  </div>
{% endblock content %}

```

## 3. مزامنة البيئة وتشغيل المشروع

```powershell
uv sync --locked
.\.venv\Scripts\python.exe -m uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

الأمر الأول يطابق البيئة مع uv.lock الموجود دون تحديثه. لم نضف مكتبات.
الثاني يشغّل Uvicorn باستخدام Python من بيئة المشروع؛ تفعيل البيئة اختيارياً وليس ضرورياً هنا.
main:app يعني تحميل ملف main.py واستخدام كائن app. reload يعيد التحميل عند تعديل الملفات أثناء التطوير.
نفّذ التشغيل من جذر المشروع لأن static وtemplates مساران نسبيان في كود المدرّس.

## 4. التحقق يدوياً

افتح طرفية PowerShell ثانية واترك الخادم في الأولى:

```powershell
Invoke-RestMethod http://127.0.0.1:8000/api/posts
Invoke-RestMethod http://127.0.0.1:8000/api/posts/1
curl.exe -i http://127.0.0.1:8000/posts/1
curl.exe -i http://127.0.0.1:8000/posts/999
curl.exe -i http://127.0.0.1:8000/api/posts/999
curl.exe -i http://127.0.0.1:8000/posts/abc
curl.exe -i http://127.0.0.1:8000/api/posts/abc
```

| المسار | النتيجة |
|---|---|
| `/` أو `/posts` | 200، قائمة المنشورات وروابط صفحاتها |
| `/posts/1` أو `/posts/2` | 200، HTML لمنشور واحد |
| `/api/posts/1` أو `/api/posts/2` | 200، JSON لمنشور واحد |
| `/posts/999` | 404، صفحة خطأ HTML |
| `/api/posts/999` | 404، JSON ورسالة Post not found |
| `/posts/abc` | 422، صفحة خطأ HTML |
| `/api/posts/abc` | 422، JSON يوضح خطأ تحويل المعرّف إلى عدد صحيح |
| `/docs` | Swagger UI ويعرض مسارات API فقط |

افتح المتصفح على http://127.0.0.1:8000 واضغط عنوان المنشور، ثم جرّب صفحة الأخطاء.
أوقف الخادم بـ Ctrl+C. للتشغيل لاحقاً أعد أمر Set-Location وأمر uvicorn فقط.

## 5. كيف ينفّذ التطبيق الطلب؟

1. يحمل Uvicorn main.py؛ ينشئ FastAPI التطبيق ويجهز static وJinja2 ويسجل المسارات ومعالجات الأخطاء.
2. عند GET /posts، ينفذ home ويرسل قائمة posts إلى home.html، الذي يرث layout.html.
3. يولّد url_for("post_page", post_id=post.id) رابط كل منشور باستخدام اسم مساره؛ لا ينفّذ post_page أثناء بناء الرابط.
4. عند النقر على /posts/1 يستخرج FastAPI القيمة من المسار ويتحقق من أنها int قبل استدعاء post_page.
5. تبحث الدالة في القائمة، وتُمرر المنشور إلى post.html. [:50] يختصر عنوان علامة التبويب إلى 50 محرفاً؛ لا يغير العنوان الأصلي.
6. إذا كان المعرّف عدداً لكنه غير موجود مثل 999، ترفع الدالة HTTPException برمز 404.
7. يلتقط general_http_exception_handler الاستثناء؛ يعيد JSON إن بدأ المسار بـ /api، وإلا يعرض error.html. رمز HTTP يبقى 404.
8. إذا كانت القيمة abc، يحدث RequestValidationError قبل دخول الدالة؛ يعيده المعالج الخاص برمز 422، HTML للصفحات وJSON للـ API.
9. /api/posts/1 يطبق البحث نفسه، لكن يرجع dict فيحوّله FastAPI إلى JSON دون استعمال Jinja2.

الفرق عن Type hints العادية: Python لا يفرض int تلقائياً، لكن FastAPI يقرأ هذا التعريف للتحقق من مدخلات HTTP.
معالج StarletteHTTPException يلتقط أيضاً أخطاء HTTP الصادرة من طبقة Starlette مثل المسارات غير الموجودة.
include_in_schema=False يخفي صفحات HTML من وثائق OpenAPI فقط، ولا يمنع الوصول إليها.
معيار HTML مقابل JSON هنا هو بادئة المسار، وليس ترويسة Accept، كما هو في الدرس.
معالجات الأخطاء المضافة تخص HTTP والتحقق من الطلب؛ ليست معالجاً عاماً لكل استثناء Python.

## 6. الملفات التي بقيت من المحاضرات السابقة

- templates/layout.html: الإطار المشترك والقائمة الجانبية والثيم وBootstrap.
- static/: CSS والصور والأيقونات والملفات الثابتة كما كانت.
- pyproject.toml: تعريف المشروع واعتمادياته.
- uv.lock: إصدارات الاعتماديات المقفلة، لم يتغير.
- .python-version: بقي 3.13 بما يناسب Python المثبت على الجهاز.
- home_finished.html وlayout_finished.html: نسخ تعليمية قديمة احتُفظ بها، ولا يستدعيها التطبيق.
- README_LESSON_01_AR.md وREADME_LESSON_02_AR.md وLESSON_02_MANUAL_AR.md: شروحات المراحل السابقة.

## 7. ما الذي نفّذه المساعد فعلياً؟

تم جلب المرجع للتحقق منه بالأمر التالي من مجلد عمل المحادثة، ثم مقارنة ملفات سطح المكتب بمرجع المحاضرة الثانية:

```powershell
git clone --depth 1 https://github.com/CoreyMSchafer/FastAPI-03-Path-Parameters.git work/fastapi-lesson-03-source
git -C work/fastapi-lesson-03-source rev-parse HEAD
```

ثم أنشأ المساعد سكربت Apply-FastAPI-Lesson-03.ps1 يحتوي النص الكامل للملفات السبعة، وأوامر النسخ الاحتياطي والحفظ ومزامنة البيئة، وشغّله محلياً. خطوات Notepad أعلاه هي البديل اليدوي لنفس الكتابة، وليست ادعاء بأن الملفات كُتبت فعلياً عبر Notepad.
لا تحتاج إلى تنزيل مستودع المدرّس أو نسخ مشروع جديد لتنفيذ الخطوات اليدوية في هذا الدليل.
أضيف هذا الدليل وملفات التوثيق بعد تحديث الكود. اختبارات التحقق وأدوات توليد الدليل خارج مشروع الدرس.

المصدر: https://github.com/CoreyMSchafer/FastAPI-03-Path-Parameters

Commit: `d97136e799bf07259abf83dec11f5416ccfedfe7`.
