# 7. Async Processing & Events

## 7.1 Event-Driven Architecture

Mail Service mempublish event ke RabbitMQ untuk processing async:

| Event | Trigger | Consumer Action |
|---|---|---|
| `mail.sent` | Surat berhasil dikirim | Notification Service: kirim email ke recipients yang flag email=1 |
| `mail.archive.finalized` | Arsip difinalisasi | Notification Service: kirim internal mail ke user dengan akses |
| `mail.archive.access.updated` | Access control diupdate | Notification Service: kirim internal mail ke user baru |

## 7.2 Event Schema

```json
{
  "eventType": "mail.sent",
  "timestamp": "2026-03-12T06:41:52Z",
  "payload": {
    "mailId": 180002,
    "mailNumber": "001/I/HRD-485.1/III/2026",
    "subject": "Undangan Rapat",
    "createdByName": "Budi Santoso",
    "createdByTitle": "Kabag SDM",
    "recipients": [
      {
        "userId": 20,
        "empName": "Andi Pratama",
        "empEmail": "andi@company.com",
        "emailNotif": true,
        "smsNotif": false
      }
    ]
  }
}
```

## 7.3 Notification Service (Downstream)

Notification Service (separate microservice) bertanggung jawab:
- Consume events dari RabbitMQ
- Render email body dari template (`msg_template`)
- Kirim via SMTP berdasarkan `smtp_mail_config`
- Log ke `smtp_mail_log`
- Handle retry & dead letter queue

> **Fase transisi**: Notification bisa tetap di Mail Service sebagai `@Async` method, kemudian diekstrak ke service terpisah.

<!-- nav -->
---
**Navigation:** [< Prev](06-hr-service-integration.md) | [Next >](08-migration-backward-compatibility.md)
<!-- /nav -->
