# IT Quiz — quiz.infortel.pl

Practice platform for IT job interviews. Bilingual (Polish / English).

## What is this?

A free, open-source quiz app for developers preparing for technical interviews. Multiple choice questions and hands-on code challenges covering Python, Django, SQL, REST API, Linux, and Git.

**Live:** [quiz.infortel.pl](https://quiz.infortel.pl)

## Features

- **Multiple choice quizzes** — instant feedback with explanations
- **Code challenges** — write Python/SQL code in the browser, validated server-side
- **XP system** — earn points, level up (Beginner → Junior → Mid → Senior → Expert)
- **Bilingual** — every question in Polish and English, switch anytime
- **Magic link auth** — enter email, get a login link, no passwords
- **Progress tracking** — see your score per category, find weak spots
- **Leaderboard** — compare with other users
- **Suggest questions** — submit ideas for new questions and categories

## Tech Stack

| Component | Technology |
|-----------|------------|
| Backend | Django 5.1, Gunicorn |
| Frontend | Django Templates + HTMX + Tailwind CSS |
| Code editor | CodeMirror |
| Database | PostgreSQL |
| Server | Ubuntu (VPS), nginx, Supervisor |
| SSL | Let's Encrypt |

No JavaScript frameworks. No build step. HTMX handles all interactivity.

## Categories

| Category | Topics |
|----------|--------|
| **Python** | Syntax, types, OOP, exceptions, comprehensions, decorators |
| **Django** | Models, views, ORM, middleware, forms, signals |
| **SQL / MSSQL** | SELECT, JOIN, GROUP BY, subqueries, window functions, T-SQL |
| **REST API** | HTTP methods, status codes, authentication, CORS, versioning |
| **Linux** | Commands, permissions, processes, systemd, networking |
| **Git** | Commands, branching, merge vs rebase, conflicts |

More categories can be added — see [Adding Questions](#adding-questions) below.

## Project Structure

```
quizapp/
├── manage.py
├── requirements.txt
├── .env.example              # Environment variables template
│
├── quizapp/                  # Django project config
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
│
├── accounts/                 # Authentication (magic link)
│   ├── models.py             # QuizUser — email + UUID token
│   ├── views.py              # Login, magic link, logout
│   ├── middleware.py          # Attach quiz_user to request
│   └── templates/accounts/   # Login page, email template
│
├── quiz/                     # Core quiz logic
│   ├── models.py             # Category, Question, Answer, CodeChallenge,
│   │                         # QuizAttempt, UserAnswer, CodeSubmission, Suggestion
│   ├── views.py              # Quiz flow, progress, suggestions
│   ├── admin.py              # Django admin for managing questions
│   ├── runner.py             # Sandboxed code execution (Python/SQL)
│   ├── templatetags/
│   │   └── quiz_tags.py      # Bilingual template tag {% bi obj "field" %}
│   ├── management/commands/
│   │   └── load_questions.py # Seed initial questions from JSON
│   └── templates/quiz/       # All quiz templates (HTMX partials)
│
├── templates/
│   └── base.html             # Base template (Tailwind + HTMX + CodeMirror)
│
├── static/                   # CSS, JS, images
├── locale/                   # i18n translations (PL/EN)
│   ├── pl/LC_MESSAGES/
│   └── en/LC_MESSAGES/
└── fixtures/                 # Initial question data (JSON)
```

## XP System

| Action | XP |
|--------|-----|
| Quiz correct (easy) | 5 |
| Quiz correct (medium) | 10 |
| Quiz correct (hard) | 20 |
| Code challenge (easy) | 15 |
| Code challenge (medium) | 30 |
| Code challenge (hard) | 50 |
| Accepted suggestion | 25 |

**Levels:** Beginner (0) → Junior (100) → Mid (500) → Senior (1500) → Expert (5000)

XP is earned once per question — repeating doesn't give more points.

## Adding Questions

### Via Django Admin

1. Go to `https://quiz.infortel.pl/admin/`
2. Log in with superuser credentials
3. Add categories, questions with answers, or code challenges
4. Each question needs both `_pl` and `_en` fields filled

### Via JSON fixtures

Add questions to `fixtures/questions.json`:

```json
{
  "category": "python",
  "difficulty": "medium",
  "text_pl": "Co zwróci `len([1, [2, 3], 4])`?",
  "text_en": "What will `len([1, [2, 3], 4])` return?",
  "explanation_pl": "Lista ma 3 elementy: 1, [2,3] i 4. Zagnieżdżona lista liczy się jako jeden element.",
  "explanation_en": "The list has 3 elements: 1, [2,3] and 4. The nested list counts as one element.",
  "answers": [
    {"text_pl": "3", "text_en": "3", "is_correct": true},
    {"text_pl": "4", "text_en": "4", "is_correct": false},
    {"text_pl": "5", "text_en": "5", "is_correct": false},
    {"text_pl": "Błąd", "text_en": "Error", "is_correct": false}
  ]
}
```

Then run:
```bash
python manage.py load_questions
```

### Via suggestion form

Users can submit question ideas through the app. Admins review and approve them in the admin panel.

## Local Development

```bash
# Clone
git clone https://github.com/zacharm-ewsoft/it-quiz.git
cd it-quiz

# Setup
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Configure
cp .env.example .env
# Edit .env — set SECRET_KEY, DATABASE_URL (or use sqlite for dev)

# Run
python manage.py migrate
python manage.py load_questions
python manage.py createsuperuser
python manage.py runserver
```

## Deployment

The app runs on a VPS alongside other services:

- **Gunicorn** on port 8001 (managed by Supervisor)
- **nginx** reverse proxy with SSL (Let's Encrypt)
- **PostgreSQL** database

Deploy:
```bash
git push origin main
# On VPS:
cd /home/claude-worker/quizapp
git pull
python manage.py migrate
python manage.py collectstatic --noinput
supervisorctl restart quizapp
```

## Contributing

Contributions welcome! You can:
- Submit question suggestions through the app
- Open issues on GitHub
- Send pull requests with new questions or features

## License

MIT

---

Built with Django, HTMX, and AI-assisted development.
