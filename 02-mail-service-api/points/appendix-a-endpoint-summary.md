# Appendix A: Full Endpoint Summary

| Method | Path | Description | Auth |
|---|---|---|---|
| **Folders** | | | |
| `GET` | `/v1/folders` | List folders + counters | USER |
| `POST` | `/v1/folders` | Create custom folder | USER |
| `PUT` | `/v1/folders/{id}` | Rename folder | USER |
| `DELETE` | `/v1/folders/{id}` | Delete custom folder | USER |
| **Mails** | | | |
| `GET` | `/v1/mails` | List mails in folder | USER |
| `GET` | `/v1/mails/{id}` | Get mail detail | USER |
| `POST` | `/v1/mails` | Create draft / send | USER |
| `PUT` | `/v1/mails/{id}` | Update draft | USER |
| `POST` | `/v1/mails/{id}/send` | Send draft | USER |
| `POST` | `/v1/mails/{id}/reply` | Reply / disposition | USER |
| `POST` | `/v1/mails/{id}/move` | Move to folder | USER |
| `POST` | `/v1/mails/{id}/mark-read` | Mark as read → Read Items | USER |
| `DELETE` | `/v1/mails/{id}` | Delete (trash/permanent) | USER |
| `POST` | `/v1/mails/{id}/restore` | Restore from trash | USER |
| `DELETE` | `/v1/mails/trash` | Empty trash | USER |
| `GET` | `/v1/mails/{id}/read-status` | Recipient read status | USER |
| `GET` | `/v1/mails/{id}/thread` | Full thread chain | USER |
| **Recipients** | | | |
| `GET` | `/v1/mails/{id}/recipients` | List recipients | USER |
| `POST` | `/v1/mails/{id}/recipients` | Add recipient | USER |
| `PATCH` | `/v1/mails/{id}/recipients/{rid}` | Update notif flags | USER |
| `DELETE` | `/v1/mails/{id}/recipients/{rid}` | Remove recipient | USER |
| **Attachments** | | | |
| `GET` | `/v1/mails/{id}/attachments` | List mail attachments | USER |
| `POST` | `/v1/mails/{id}/attachments` | Upload attachment | USER |
| `GET` | `/v1/archives/{id}/attachments` | List archive attachments | USER |
| `POST` | `/v1/archives/{id}/attachments` | Upload archive attachment | USER |
| `GET` | `/v1/attachments/{id}/download` | Download file | USER |
| `DELETE` | `/v1/attachments/{id}` | Delete attachment | USER |
| **Archives** | | | |
| `GET` | `/v1/archives` | List archives (admin) | MAIL_ADMIN |
| `GET` | `/v1/archives/search` | Search archives (user) | USER |
| `POST` | `/v1/archives` | Create archive | MAIL_ADMIN |
| `PUT` | `/v1/archives/{id}` | Update archive | MAIL_ADMIN |
| `POST` | `/v1/archives/{id}/finalize` | Finalize archive | MAIL_ADMIN |
| `DELETE` | `/v1/archives/{id}` | Delete archive | MAIL_ADMIN |
| `GET` | `/v1/archives/{id}/access` | List access control | MAIL_ADMIN |
| `PUT` | `/v1/archives/{id}/access` | Set access control | MAIL_ADMIN |
| **Lookups** | | | |
| `GET` | `/v1/mail-types` | List mail types | USER |
| `GET` | `/v1/mail-types/tree` | Mail types as tree | USER |
| `POST` | `/v1/mail-types` | Create type | MAIL_ADMIN |
| `PUT` | `/v1/mail-types/{id}` | Update type | MAIL_ADMIN |
| `DELETE` | `/v1/mail-types/{id}` | Delete type | MAIL_ADMIN |
| `GET` | `/v1/mail-categories` | List categories | USER |
| `POST` | `/v1/mail-categories` | Create category | MAIL_ADMIN |
| `PUT` | `/v1/mail-categories/{id}` | Update category | MAIL_ADMIN |
| `DELETE` | `/v1/mail-categories/{id}` | Delete category | MAIL_ADMIN |
| **Search** | | | |
| `GET` | `/v1/mails/search` | Search across all folders | USER |
| `GET` | `/v1/mails/find` | Find mails for referencing | USER |
| **Reports** | | | |
| `GET` | `/v1/reports/mails` | Mail report | MAIL_ADMIN |
| `GET` | `/v1/reports/mails/export` | Export to Excel | MAIL_ADMIN |
| `GET` | `/v1/reports/statistics/by-category` | Stats by category | MAIL_ADMIN |
| `GET` | `/v1/reports/statistics/by-organization` | Stats by org | MAIL_ADMIN |
| `GET` | `/v1/reports/statistics/response-time` | Response time stats | MAIL_ADMIN |
| **Admin** | | | |
| `GET` | `/v1/admin/notification-config` | Get notif config | SUPER_ADMIN |
| `PUT` | `/v1/admin/notification-config` | Update notif config | SUPER_ADMIN |

**Total: 42 endpoints**

<!-- nav -->
---
**Navigation:** [< Prev](../endpoints/5.12-notification-config.md) | Next >
<!-- /nav -->
