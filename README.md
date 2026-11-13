# AI-Powered Resume Parser & Multi-Channel 2FA Onboarding Pipeline

An internship-ready FastAPI backend implementing the three-day task PDF. **No frontend UI is included**; Swagger `/docs` is the interactive testing interface. Resume extraction uses deterministic regex heuristics, not an LLM.

## Features
- PDF and DOCX parsing with field mapping to JSON
- Staged registration, Argon2id password hashing, async PostgreSQL
- Email and WhatsApp 6-digit OTP, browser-authenticator TOTP, 5-minute expiry, 5 failed-attempt limit, 60-second resend cooldown
- JWT bearer token on successful verification
- Editable protected profile and student dashboard endpoint
- Console-mode OTP delivery for local development; optional SMTP and Twilio WhatsApp integration

## Windows 10 / VS Code setup (Python 3.11)
1. Install PostgreSQL; create database `resume_pipeline` using pgAdmin.
2. Open this folder in VS Code terminal (PowerShell).
3. `py -3.11 -m venv .venv`
4. `.\.venv\Scripts\Activate.ps1` (if blocked, use `.\.venv\Scripts\python.exe` for commands)
5. `python -m pip install -r requirements.txt`
6. Copy `.env.example` to `.env`, set your PostgreSQL password, random JWT_SECRET and OTP_PEPPER. **Never commit `.env`.**
7. `uvicorn app.main:app --reload`
8. Visit `http://127.0.0.1:8000/docs`.

## Swagger testing sequence
1. POST `/api/v1/resume/parse`: upload a PDF or DOCX; copy parsed fields. Review and correct any parsing errors.
2. POST `/api/v1/auth/register`: use corrected values, password (10+ characters); optionally provide `resume_filename` and `resume_metadata` to persist the uploaded metadata. **This is a demo-only trust boundary: clients supply the metadata; production should bind uploads to a server-generated ID.**
3. POST `/api/v1/auth/send-2fa`: `{"email":"student@example.com","channel":"email"}`. In console mode, read the code in the VS Code terminal. For WhatsApp, use `channel=whatsapp` and a real phone field; for TOTP, use `channel=totp` and add the returned provisioning URI to an authenticator app.
4. POST `/api/v1/auth/verify-2fa`: supply the matching channel and code.
5. Click **Authorize** in Swagger, paste `access_token` (Bearer scheme).
6. GET `/api/v1/profile`, PATCH `/api/v1/profile`, GET `/dashboard/student`.

## Tests
`python -m pytest -q`

## Security and deployment notes
- OTP console mode prints secrets to server logs and is **for development only**.
- This project demonstrates verification of one selected factor after registration. It is **not** a full password-login-plus-second-factor flow; the PDF doesn't specify a login endpoint. Add password login, account recovery, stronger rate limiting and persistent TOTP replay prevention before production use.
- A TOTP provisioning URI is sensitive: show it only in an authenticated or short-lived enrollment flow in production. The demo endpoint identifies the user by email only and must not be exposed publicly as-is.
- No file storage; resumes are parsed in memory. `user_resumes` records metadata when supplied at registration.
- `create_all()` is development schema initialization, not a versioned migration. For production use Alembic migrations, HTTPS, a reverse proxy, secrets manager, audit logging and monitoring.
- For production, never use the fallback development secrets. Set strong unique secrets in `.env` or deployment secrets.
- SMTP and Twilio require provider credentials and network access; they are not guaranteed working until configured.
- The parser cannot OCR scanned PDFs and its name/experience extraction is heuristic.

## Endpoints
`POST /api/v1/resume/parse`, `POST /api/v1/auth/register`, `POST /api/v1/auth/send-2fa`, `POST /api/v1/auth/verify-2fa`, `GET /api/v1/profile`, `PATCH /api/v1/profile`, `GET /dashboard/student`.
