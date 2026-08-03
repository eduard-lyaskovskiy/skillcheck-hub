Главная функциональность

# 1. Multi-tenant система

Пользователь принадлежит к конкретной организации.

Например:

Tenant A
Tenant B
у каждого свои пользователи
свои тесты
свои кандидаты
свои отчеты

Главная цель: чтобы данные одного клиента нельзя было получить из другого клиента.

Это очень хороший материал для собеседования, потому что можно рассказывать про:

tenant isolation
role-based access control
authorization guards
database filtering by tenantId
security risks

# 2. Auth & Roles

Сделай нормальную авторизацию:

Роли:

SUPER_ADMIN
TENANT_ADMIN
REVIEWER
CANDIDATE

Функции:

login/password
JWT access token
refresh token
guards в NestJS
decorators типа @CurrentUser()
role guard
tenant guard

Пример:

```
@Get(':id/report')
@Roles(UserRole.TENANT_ADMIN, UserRole.REVIEWER)
async getCandidateReport(@Param('id') id: string) {}
```

Что тренируешь:

JWT
guards
decorators
RBAC
security
NestJS architecture

# 3. Assessment Builder

Админ компании может создать тест.

Сущности:

Assessment
Section
Question
QuestionOption
Candidate
CandidateSession
Answer

Типы вопросов:

single choice
multiple choice
text answer
scale from 1 to 5
yes/no

Функции:

создать assessment
добавить вопросы
опубликовать assessment
отправить invite кандидату
candidate проходит тест по unique code/link

Это хорошо связано с твоим опытом screening platform.

# 4. Candidate Session

Кандидат получает ссылку:

https://app.local/test/abc123

Он проходит тест.

Backend должен:

создать session
сохранить start time
сохранить ответы
сохранить finish time
посчитать basic score
создать report generation job

Статусы:

CREATED
IN_PROGRESS
COMPLETED
REPORT_GENERATING
REPORT_READY
FAILED

# 5. Report Generation

После завершения теста backend не должен сразу генерировать отчет в request-response flow.

Лучше сделать через очередь:

Candidate finishes test
Backend creates job in Redis/BullMQ
Worker generates report
Report is uploaded to MinIO/S3
Report status becomes READY
Admin can download PDF/JSON report

Что тренируешь:

background jobs
Redis
async processing
retries
error handling
file storage
report downloading

# 6. Python Lambda часть

Чтобы честно практиковать Python из CV, добавь отдельный Python Lambda-like service.

Например:

Вариант A — Python report analyzer

Node.js генерирует raw report data и отправляет его в Python function.

Python function:

считает дополнительные метрики
проверяет suspicious answers
считает consistency score
возвращает результат

Пример логики без AI:

If candidate answered opposite questions inconsistently,
increase risk score.

Например:

Question A:

I have never lied at work.

Question B:

Sometimes I lie to avoid problems.

Можно руками сделать простую scoring logic.

Вариант B — Python file processor

Python Lambda принимает JSON/CSV report, валидирует его и возвращает summary.

Как запускать

Для начала не надо сразу AWS.

Можно сделать:

локально как отдельный Python service
потом завернуть в Docker
потом optional: задеплоить в AWS Lambda
или использовать LocalStack

Так ты сможешь честно говорить:

I used Python for Lambda-style report post-processing. My main stack is Node.js/TypeScript, but I can work with Python for serverless tasks.

# 7. MinIO / S3 Storage

Используй MinIO локально как S3-compatible storage.

Храни:

generated reports
uploaded candidate files
exported CSV
maybe screenshots/mock files

API:

POST /files/upload
GET /reports/:id/download
GET /reports/:id/signed-url

Что тренируешь:

S3-compatible storage
file upload
signed URLs
secure file access
MinIO from CV

# 8. Socket.io live status

Сделай простой realtime dashboard.

Когда кандидат проходит тест, reviewer видит:

Candidate John Smith started test
Candidate John Smith answered question 5/20
Candidate John Smith completed test
Report is generating
Report is ready

Через Socket.io:

tenant:{tenantId}:sessions

Что тренируешь:

WebSocket / Socket.io
auth for socket connections
rooms
tenant isolation
realtime events

# 9. Optional WebRTC module

Не обязательно делать полноценное видео. Это может съесть много времени.

Но можно сделать маленький модуль:

Candidate session room
Reviewer can see connection status
Socket.io signaling mock
WebRTC peer connection demo

Минимальная цель:

создать room
connect two browser tabs
exchange offer/answer через Socket.io
показать video stream from webcam locally

Это даст тебе возможность освежить WebRTC, но не утонуть в проекте.

# 10. Admin UI

Frontend можно сделать простым, чтобы не тратить много времени.

Стек:

Next.js
React Hook Form
basic UI components
Tailwind или обычный CSS

Страницы:

```
/login
/dashboard
/assessments
/assessments/:id
/candidates
/candidates/:id
/reports
/test/:token
```

Но основной фокус всё равно backend.

Рекомендуемый стек

Я бы выбрал такой:

Backend
NestJS
TypeScript
Prisma or TypeORM
PostgreSQL or MySQL
Redis + BullMQ
Socket.io
Swagger
JWT
Storage
MinIO locally
S3-compatible SDK
Python
Python FastAPI service first
Then optional AWS Lambda style
Frontend
Next.js
React
Basic UI only
DevOps
Docker
docker-compose
GitHub Actions
ESLint
Prettier
Jest

# multi-tenancy

I implemented tenant isolation by attaching every user, assessment, candidate session and report to a tenantId. I also added authorization guards to make sure users can only access resources from their own tenant.

# background jobs

Report generation is handled asynchronously. When a candidate completes a test, the API creates a job in Redis using BullMQ. A separate worker processes the job, generates a report, uploads it to MinIO, and updates the report status.

# Python

I used Python for Lambda-style report post-processing. The Node.js backend sends raw test data to a Python processor, which calculates additional metrics and returns a structured result for the final report.

# Socket.io

I used Socket.io to send live candidate session updates to reviewers. Each tenant has isolated rooms, so users only receive events related to their organization.

# MinIO/S3

I used MinIO locally as an S3-compatible storage solution for generated reports. The backend stores file keys in the database and provides protected download endpoints.
