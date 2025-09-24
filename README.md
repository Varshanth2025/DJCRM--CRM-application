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
- Django-Tailwind (for Tailwind CSS assets)

### Setup

1. Clone the repository.

--> Clone the repository using the command below :

```bash
git clone https://github.com/Varshanth2025/DJCRM--CRM-application

```

2. Create a new file named `.env` inside the `djcrm` folder.
3. Copy all variables from [`djcrm/.template.env`](djcrm/.template.env) and assign your own values.
4. Install Python dependencies:
   ```sh
   pip install -r requirements.txt
   ```
5. Build static assets:
   ```sh
   npm run build
   ```
6. Run migrations:
   ```sh
   python manage.py migrate
   ```
7. Create a superuser:
   ```sh
   python manage.py createsuperuser
   ```
8. Start the development server:
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
python manage.py create_leads leads.csv
```

### App Preview :

<table width="100%"> 
  <tr>
    <td width="50%">
      <p align="center">Home and login page</p>
      <img src="https://github.com/Varshanth2025/DJCRM--CRM-application/blob/main/static/images/home_login.png">
    </td>
    <td width="50%">
      <p align="center">List of leads</p>
      <img src="https://github.com/Varshanth2025/DJCRM--CRM-application/blob/main/static/images/leads.png">
    </td>
  </tr>
  <tr>
    <td colspan="2" align="center">
      <p align="center">Create new lead</p>
      <img src="https://github.com/Varshanth2025/DJCRM--CRM-application/blob/main/static/images/create_new_lead.png">
    </td>
  </tr>
</table>
