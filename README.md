# DJCRM

A simple CRM built with Django, featuring lead and agent management, categories, follow-ups, and a modern Tailwind CSS UI.

## Features

- User authentication (signup, login, password reset)
- Organisor and agent roles
- Lead CRUD (create, read, update, delete)
- Agent CRUD
- Lead categories and follow-ups
- Dashboard with lead statistics
- Responsive UI with Tailwind CSS

## Getting Started

### Prerequisites

- Python 3.7+
- PostgreSQL
- Node.js (for Tailwind CSS assets)

### Setup

1. Clone the repository.
2. Create a new file named `.env` inside the `djcrm` folder.
3. Copy all variables from [`djcrm/.template.env`](djcrm/.template.env) and assign your own values.
4. Install Python dependencies:
   ```sh
   pip install -r requirements.txt
   ```
5. Install Node dependencies for Tailwind CSS:
   ```sh
   cd theme/static_src
   npm install
   ```
6. Build static assets:
   ```sh
   npm run build
   ```
7. Run migrations:
   ```sh
   python manage.py migrate
   ```
8. Create a superuser:
   ```sh
   python manage.py createsuperuser
   ```
9. Start the development server:
   ```sh
   python manage.py runserver
   ```

### Running Tests

```sh
python manage.py test
```

### Import Leads from CSV

You can import leads using the management command:

```sh
python manage.py create_leads leads.csv user@example.com
```
