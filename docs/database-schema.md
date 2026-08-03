tenants

- id
- name
- slug
- createdAt

users

- id
- tenantId
- email
- passwordHash
- role
- createdAt

assessments

- id
- tenantId
- title
- description
- status
- createdById
- createdAt

questions

- id
- assessmentId
- text
- type
- order

candidate_sessions

- id
- tenantId
- assessmentId
- candidateId
- token
- status
- startedAt
- completedAt

answers

- id
- sessionId
- questionId
- value

reports

- id
- sessionId
- status
- score
- fileKey
- createdAt
