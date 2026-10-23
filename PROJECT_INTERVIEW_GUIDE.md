# DJCRM — Infosys SP/DSE Project Interview Guide

> **Repository:** `Varshanth2025/DJCRM--CRM-application`  
> Based on inspected code in `djcrm/`, `leads/`, `agents/`, `manage.py`, `requirements.txt`, and the repository README. README-only claims are marked for verification.

## 1. Project purpose and architecture

DJCRM is a Django CRM application for managing leads, agents, categories, and follow-ups. It supports two application roles: organisation users and agents. Organisation users can manage leads and agents; agents see leads assigned to them.

### Architecture

- **Framework:** Django 3.1 using the MVT pattern.
- **Project configuration:** `djcrm/settings.py`, `djcrm/urls.py`, `djcrm/wsgi.py`, `djcrm/asgi.py`.
- **Lead domain:** `leads/models.py`, `leads/views.py`, `leads/forms.py`, `leads/urls.py`.
- **Agent domain:** `agents/views.py`, `agents/forms.py`, `agents/mixins.py`, `agents/urls.py`.
- **Database:** PostgreSQL configured through environment variables and Django ORM.
- **Frontend:** Django templates with Tailwind/django-crispy-forms.
- **Imports:** `leads/management/commands/create_leads.py` imports leads from CSV.
- **Static/media:** WhiteNoise for static assets; uploaded files use Django media storage.

### End-to-end execution flow

1. `manage.py` sets `DJANGO_SETTINGS_MODULE` to `djcrm.settings` and delegates commands to Django.
2. Django loads environment configuration, installed apps, middleware, PostgreSQL settings, the custom user model, static/media settings, and authentication redirects from `djcrm/settings.py`.
3. `djcrm/urls.py` routes requests to authentication views, the dashboard, `leads.urls`, and `agents.urls`.
4. A user signs up through `SignupView`. The custom `User` model is created, and the `post_user_created_signal` in `leads/models.py` creates a related `UserProfile`.
5. Authenticated users access the dashboard through `DashboardView`. Its context calculates total leads, leads added in the last 30 days, and recently converted leads.
6. Organisation users can create a lead through `LeadCreateView`. `form_valid()` attaches `request.user.userprofile` as the organisation, saves the lead, sends an email, and adds a success message.
7. `LeadListView` filters data by role: organisation users see their organisation’s assigned leads; agents see only leads assigned to the logged-in agent. Organisation users also receive unassigned leads in the context.
8. Organisation users can create agents through `AgentCreateView`. The view creates a user, sets role flags, generates a random password, creates an `Agent` record linked to the organisation, and sends an invitation email.
9. Leads can be assigned to agents through `AssignAgentView`; the form restricts selectable agents to the current organisation.
10. Categories and follow-ups are managed through class-based views. `LeadCategoryUpdateView` records `converted_date` when a lead first changes to the `Converted` category.
11. The CSV command receives a filename and organisation email, resolves the organisation’s `UserProfile`, and creates `Lead` rows for each CSV record.

## 2. Important files, functions, and talking points

| File | Important code | Interview talking point |
|---|---|---|
| `leads/models.py` | `User`, `UserProfile`, `Lead`, `FollowUp`, `Agent`, `Category` | Domain model and relationships. |
| `leads/models.py` | `post_user_created_signal()` | Automatically creates a profile for each user. |
| `leads/views.py` | `DashboardView.get_context_data()` | Aggregate CRM statistics and date filtering. |
| `leads/views.py` | `LeadListView.get_queryset()` | Organisation/agent data isolation. |
| `leads/views.py` | `LeadCreateView.form_valid()` | Assigns organisation server-side and triggers email. |
| `leads/views.py` | `LeadCategoryUpdateView.form_valid()` | Tracks conversion time on a state transition. |
| `leads/views.py` | `AssignAgentView.form_valid()` | Assigns a lead to an organisation-scoped agent. |
| `leads/forms.py` | `LeadModelForm`, `AssignAgentForm` | Form binding, model validation, and request-scoped querysets. |
| `agents/views.py` | `AgentListView`, `AgentCreateView`, CRUD views | Organisation-only agent management. |
| `agents/mixins.py` | `OrganisorAndLoginRequiredMixin.dispatch()` | Centralized authentication and role authorization. |
| `djcrm/settings.py` | `AUTH_USER_MODEL`, PostgreSQL, middleware | Security/configuration and custom user model. |
| `djcrm/urls.py` | Project-level routes | URL composition and built-in auth flows. |
| `leads/management/commands/create_leads.py` | `Command.handle()` | Bulk CSV import through Django’s command system. |

**Personal contribution warning:** The repository does not prove which features you personally implemented. Verify your exact contribution before the interview.

## 3. Technologies and design decisions

- **Django MVT:** Provides routing, ORM, forms, authentication, templates, and class-based views in one framework.
- **PostgreSQL:** Production-oriented relational storage for users, organisations, leads, agents, categories, and follow-ups.
- **Django ORM:** Encapsulates SQL and supports relation traversal such as `lead__organisation` and `lead__agent__user`.
- **Custom `User`:** Extends `AbstractUser` with `is_organisor` and `is_agent` flags. `AUTH_USER_MODEL = 'leads.User'` is configured before migrations.
- **Class-based views:** Reuse Django’s CRUD behavior while overriding `get_queryset`, `form_valid`, and `get_context_data` for business rules.
- **Django forms:** `ModelForm` derives fields and validation from models; `AssignAgentForm` sets its queryset dynamically from the current request.
- **Tailwind/crispy forms:** Used for styling and form rendering; verify the exact template usage before claiming a specific component strategy.
- **WhiteNoise:** Serves compressed static files in deployment scenarios.
- **Gunicorn:** Included for production WSGI serving.
- **django-environ:** Reads database, secret, email, and debug settings from environment variables.

### Key design decisions

1. **Organisation ownership:** `Lead`, `Agent`, and `Category` reference `UserProfile`, allowing tenant-like filtering by organisation.
2. **Role flags:** The application uses boolean flags instead of a separate role table. This is simple but can become difficult to extend for many roles.
3. **Server-side ownership assignment:** Lead creation ignores any client-supplied organisation and sets it from the authenticated user’s profile.
4. **Scoped form choices:** `AssignAgentForm` starts with `Agent.objects.none()` and fills the queryset for the current organisation in `__init__`, preventing cross-organisation agent selection in that form.
5. **State transition timestamp:** `converted_date` is set only when the category changes into `Converted`, preserving when conversion occurred.
6. **Signals:** `post_user_created_signal` creates `UserProfile` automatically, reducing repeated profile-creation code.

## 4. Database, APIs, algorithms, error handling, and debugging

### Database model

- `User`: Custom Django auth user with `is_organisor` and `is_agent`.
- `UserProfile`: One-to-one extension of `User`; represents an organisation owner/profile.
- `Lead`: Contact data, organisation, optional agent, optional category, description, dates, email, phone, and profile picture.
- `Agent`: One-to-one link to a user plus organisation ownership.
- `Category`: Organisation-specific lead status/category.
- `FollowUp`: Lead-specific notes and optional uploaded file.

Deletion behavior is important: deleting an organisation profile can cascade to related leads, agents, categories, and follow-ups depending on relation paths; verify migration behavior before describing it in detail.

### URL/API surface

This is primarily a server-rendered HTML application, not a REST API.

- `/`: landing page.
- `/dashboard/`: organisation dashboard.
- `/leads/`: lead list and CRUD routes.
- `/leads/json/`: `LeadJsonView`, returning all leads’ first name, last name, and age as JSON.
- `/agents/`: agent list and CRUD routes.
- `/signup/`, `/login/`, `/logout/`, and password-reset routes.
- `/admin/`: Django admin.
- Management command: `python manage.py create_leads <file_name> <organisor_email>`.

### Algorithms and queries

- **Role-based filtering:** `LeadListView.get_queryset()` chooses organisation-wide assigned leads for organisation users and agent-specific assigned leads for agents.
- **Dashboard counts:** Uses database `.count()` with date filters; recently added means `date_added__gte` 30 days ago.
- **Conversion tracking:** Compares the prior category with the new category and sets `converted_date` only on the first transition to `Converted`.
- **CSV import:** `csv.DictReader` iterates rows and creates one `Lead` per row.
- **Data isolation:** Querysets traverse relationships such as `organisation=user.userprofile` and `lead__agent__user=user`.

### Error handling and security

Implemented protections include:

- `LoginRequiredMixin` for authenticated views.
- `OrganisorAndLoginRequiredMixin` for organisation-only views.
- Django CSRF middleware.
- Django form validation.
- `secure` deployment settings when `DEBUG` is false, including secure cookies, HSTS, SSL redirect, X-Frame-Options, and content-type sniffing protection.
- Environment-based secrets and database/email configuration.

Visible weaknesses include:

- Some legacy function-based views such as `lead_detail`, `lead_update`, and `lead_delete` use `Lead.objects.get(id=pk)` without role/ownership filtering.
- `lead_list()` returns `Lead.objects.all()` and is not the protected class-based list route; verify whether it is reachable or legacy.
- `LeadJsonView` returns `Lead.objects.all()` without authentication or organisation filtering.
- `DashboardView` calls `Category.objects.get(name="Converted")` globally; it can raise `DoesNotExist` or `MultipleObjectsReturned`, and it does not scope the category to the organisation.
- `AgentCreateView` assigns a random password and emails a generic invitation but does not show a secure password-setup flow.
- `settings.py` uses `ALLOWED_HOSTS = ['*']`, including in production.
- `djcrm/.template.env` contains `SECRET_KEY=1234`, which must never be used as a production secret.
- Email uses the console backend during debug, so emails are printed rather than delivered.

### Debugging checklist

1. Run `python manage.py check` and inspect settings/environment variables.
2. Confirm `AUTH_USER_MODEL` migrations are applied before creating users.
3. Check PostgreSQL connection values in the `.env` file.
4. For missing leads, inspect organisation ownership and the role branch in `LeadListView.get_queryset()`.
5. For cross-tenant access, test every detail/update/delete/JSON route with two organisations.
6. For dashboard failures, verify a category named `Converted` exists and is correctly scoped.
7. For agent assignment failures, inspect the dynamic queryset in `AssignAgentForm.__init__()`.
8. For email issues, distinguish console backend behavior in debug from SMTP configuration when `DEBUG=False`.
9. For CSV import issues, check required column names: `first_name`, `last_name`, `age`, and `email`.
10. Use Django query logging or `assertNumQueries` in tests to identify N+1 queries and unnecessary database access.

## 5. Fifteen likely Infosys SP/DSE interview questions

1. **Explain DJCRM in one minute.**
   - **Sample answer:** DJCRM is a Django CRM for managing leads, agents, categories, and follow-ups. It uses a custom user model with organisation and agent roles, PostgreSQL through the Django ORM, class-based views, model forms, and organisation-scoped querysets to control access. It also has dashboard counts, email notifications, and CSV lead import.
   - **Follow-up:** What is the most important authorization rule?

2. **Why use a custom Django user model?**
   - **Answer:** `leads.User` extends `AbstractUser` with `is_organisor` and `is_agent`. This keeps Django authentication while adding application-specific role flags. `AUTH_USER_MODEL` points to it in settings.
   - **Follow-up:** What must be done before the first migration when using a custom user model?

3. **Explain organisation versus agent access.**
   - **Answer:** Organisation users filter by `user.userprofile`; agents use `user.agent.organisation` and then filter `agent__user=user`. This means an organisation sees its leads, while an agent sees only assigned leads.
   - **Follow-up:** Where must this rule be enforced besides the list page?

4. **How is a lead created?**
   - **Answer:** `LeadCreateView` binds `LeadModelForm`, and `form_valid()` calls `save(commit=False)`, assigns the organisation from the authenticated user’s profile, saves the lead, sends an email, and displays a success message.
   - **Follow-up:** Why should organisation ownership be assigned on the server?

5. **What is the purpose of `save(commit=False)`?**
   - **Answer:** It creates a model instance without immediately saving it, allowing server-controlled fields such as `organisation` to be set before calling `save()`.
   - **Follow-up:** What could happen if you trusted a form field for organisation?

6. **How does the dashboard calculate statistics?**
   - **Answer:** `DashboardView.get_context_data()` counts organisation-owned leads, filters recent leads with `date_added__gte` for the last 30 days, and counts converted leads whose `converted_date` is within that period.
   - **Follow-up:** What indexing or query optimization would you add?

7. **How is conversion time tracked?**
   - **Answer:** `LeadCategoryUpdateView.form_valid()` loads the previous lead, compares the submitted category with the `Converted` category, and sets `converted_date` only if the lead was not previously converted.
   - **Follow-up:** What should happen if a converted lead is moved back and converted again?

8. **Why use class-based views?**
   - **Answer:** Django generic views provide reusable CRUD behavior. The project customizes only the important points—`get_queryset`, `form_valid`, `get_success_url`, and context generation—so repeated code is reduced.
   - **Follow-up:** When would a function-based view be clearer?

9. **How does `AssignAgentForm` prevent invalid agent selection?**
   - **Answer:** It starts with `Agent.objects.none()` and, in `__init__`, replaces the field queryset with agents whose organisation matches `request.user.userprofile`. The form therefore presents only organisation-owned agents.
   - **Follow-up:** Is form filtering alone sufficient security?

10. **Explain the data model relationships.**
    - **Answer:** A user has a one-to-one `UserProfile`; an organisation profile owns many leads, agents, and categories; a lead can have one agent and one category; a lead has many follow-ups. Nullable foreign keys allow unassigned leads and uncategorized leads.
    - **Follow-up:** Why use `SET_NULL` for agent/category?

11. **How does the CSV import work?**
    - **Answer:** `create_leads.py` receives a filename and organisation email, resolves the `UserProfile`, uses `DictReader`, and creates a `Lead` per row using the required columns. It is run through Django’s management command framework.
    - **Follow-up:** How would you make imports atomic and report invalid rows?

12. **What security protections are present?**
    - **Answer:** Login-required mixins, an organisation authorization mixin, CSRF middleware, Django form validation, environment-based configuration, and production security settings are present. However, some legacy routes lack ownership filtering and need review.
    - **Follow-up:** Why is authentication not the same as authorization?

13. **Identify a concrete bug or security risk.**
    - **Answer:** `LeadJsonView` returns `Lead.objects.all()` without authentication or organisation filtering, so it may expose all leads. The legacy function-based detail/update/delete views also use unrestricted `Lead.objects.get`. I would remove or protect those routes and centralize scoped querysets.
    - **Follow-up:** How would you test for cross-organisation data leakage?

14. **What is the role of the post-save signal?**
    - **Answer:** `post_user_created_signal` creates a `UserProfile` whenever a `User` is created. This ensures the profile required by organisation ownership logic exists automatically.
    - **Follow-up:** What are the disadvantages of signals?

15. **How would you improve the project for production?**
    - **Answer:** Scope every route by organisation, remove unsafe legacy views, fix the JSON endpoint, scope dashboard categories, use secure invitation/password setup, set explicit allowed hosts, add tests and indexes, validate CSV rows, use transactions for imports, add audit logging, and replace development email/static settings with production configuration.
    - **Follow-up:** Which improvement would you implement first?

## 6. Bugs, limitations, and improvements

### Directly visible issues

- **Potential data exposure:** `LeadJsonView` serializes `Lead.objects.all()` without checking authentication or organisation.
- **Unsafe legacy CRUD functions:** `lead_detail`, `lead_update`, and `lead_delete` fetch by primary key without role or organisation scoping.
- **Unprotected legacy list:** `lead_list()` returns all leads and is less safe than `LeadListView`.
- **Dashboard category lookup:** `Category.objects.get(name="Converted")` is global, not organisation-scoped, and can fail if the category is missing or duplicated.
- **Agent invitation security:** `AgentCreateView` sets a random password and sends a generic email; there is no visible password reset or one-time activation token flow.
- **Duplicate role flags:** `is_organisor` defaults to `True`, while agent creation explicitly sets it to `False`; role invariants are not enforced at the database level.
- **CSV validation:** `create_leads.py` assumes the file exists and has required columns; it does not use a transaction, validate types, report row errors, or avoid duplicates.
- **Missing tests:** `agents/tests.py` contains only the default placeholder. The inspected `leads/tests.py` path was not available, so verify the current test coverage.
- **Hardening concern:** `ALLOWED_HOSTS = ['*']` is overly permissive, even though the code adds other production security flags.
- **Template secret:** `.template.env` contains `SECRET_KEY=1234`; it must be replaced with a generated secret and never reused.
- **Email configuration:** Console email is used by default; production SMTP behavior depends on environment variables.
- **Potential query inefficiency:** Related objects are frequently traversed in views without visible `select_related`/`prefetch_related`; verify query counts and add optimization where needed.
- **Outdated dependency versions:** The project pins older Django and package versions; verify compatibility and security status before deployment.
- **Repository hygiene:** `.DS_Store` is committed and should normally be ignored.

### Recommended improvements

1. Remove or protect legacy function-based views and enforce organisation scoping in every read/write endpoint.
2. Secure `LeadJsonView` or replace it with an authenticated, organisation-filtered API response.
3. Scope `Converted` category lookups by organisation and handle missing categories safely.
4. Implement secure agent invitations with a one-time activation/password-reset workflow.
5. Make CSV imports transactional, validate headers/types, collect row-level errors, and support duplicate detection.
6. Add database constraints or a clearer role model for organisation/agent invariants.
7. Add tests for authentication, role authorization, cross-organisation isolation, lead CRUD, assignment, conversion timestamps, follow-ups, and CSV import.
8. Use `select_related`/`prefetch_related` and database indexes for organisation, agent, category, and date filters.
9. Replace wildcard hosts and sample secrets with secure deployment configuration.
10. Add audit logs for lead changes, assignments, category transitions, and follow-ups.

## Verify before the interview

- Your exact personal contribution and which CRUD/security features you implemented.
- Whether all routes shown in the repository are reachable in the current UI, especially legacy function-based routes.
- Whether PostgreSQL, SMTP, Tailwind build, Gunicorn, and production deployment were actually configured.
- Current test coverage; the inspected agent test file is only a placeholder.
- Whether the README’s feature list reflects the current code, especially password reset, role behavior, and all CRUD operations.
