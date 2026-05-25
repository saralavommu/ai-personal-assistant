# WhatsApp Webhook — Complete Overview

---

## 1. What is This System?

This is a **WhatsApp automation bot** built for a school management platform called **Schoolify**.

When someone sends a WhatsApp message to the school's number, this PHP script receives it, figures out **who** sent it, and **routes** them to the right handler automatically.

```
Parent / Teacher / Counsellor
        sends WhatsApp message
                ↓
         Your PHP Webhook
                ↓
     Identify → Route → Reply
```

---

## 2. How WhatsApp Delivers Messages to Your Server

Meta (WhatsApp) uses a system called a **Webhook**.

```
User sends message on WhatsApp
        ↓
Meta's servers receive it
        ↓
Meta does POST request → your server URL
        ↓
Your PHP file runs and processes it
        ↓
Your server sends reply back via Meta API
```

Your server URL (webhook) is registered inside your Meta Developer App.

---

## 3. The Two Critical Tokens

There are **two separate tokens** in this system. They are completely different.

| Variable | Name | Purpose | Where to Get |
|---|---|---|---|
| `$token` | WhatsApp Access Token | Send messages back to users via Meta API | Meta Developer Console → Your App → WhatsApp → API Setup |
| `$gptkey` | OpenAI API Key | Convert voice notes to text (Whisper) | platform.openai.com → API Keys |

### `$token` (WhatsApp Token)
```php
$token = 'EAAxxxxxxxxxxxxxxxxxx';
```
- Starts with **EAA**
- Used in every WhatsApp reply your server sends
- Two types:
  - **Temporary** → expires every 24 hours (testing only)
  - **Permanent** → never expires (use for production via System User)

### `$gptkey` (OpenAI Key)
```php
$gptkey = 'sk-proj-xxxxxxxxxxxxxxxxx';
```
- Starts with **sk-**
- Used **only** when a user sends a voice note
- Sends the audio to OpenAI Whisper → gets back text
- Your current key starts with `sk-None-` which means it is **already revoked** and needs replacement

---

## 4. The Phone Number ID

```php
$phoneNumberId = '1234567890123456';
```

- This is NOT a phone number you can call
- It is Meta's internal ID for your WhatsApp Business Number
- Used in every API call to tell Meta **which number is sending the reply**
- Found on the same page as `$token` — Meta Developer Console → WhatsApp → API Setup

---

## 5. How a Reply is Sent

Every time your bot replies to a user, it does this:

```
POST https://graph.facebook.com/v18.0/{$phoneNumberId}/messages
Header: Authorization: Bearer {$token}
Body:   {
          "messaging_product": "whatsapp",
          "to": "919876543210",
          "type": "text",
          "text": { "body": "Hello!" }
        }
```

If `$token` or `$phoneNumberId` is missing or wrong → **no reply is sent, silently fails.**

---

## 6. Database Connection

```php
$servername = "localhost";
$username   = "u512935380_schoolify_db";
$password   = "Schoolify_db@123$";
$db         = "u512935380_schoolify_db";
```

- Uses **PDO** (PHP Data Objects) to connect to MySQL
- Timezone is set to **Asia/Kolkata (IST)** for accurate timestamps
- If DB connection fails → script logs the error and exits

---

## 7. Message Types Handled

The webhook can receive and handle multiple types of WhatsApp messages:

| Type | What it is | What happens |
|---|---|---|
| `text` | Normal typed message | Read directly |
| `interactive` | Button tap reply | Read the button ID |
| `audio` / `voice` | Voice note | Downloaded → sent to OpenAI Whisper → converted to text |
| `document` | PDF or file | Downloaded and saved to server |
| `image` | Photo | Downloaded and saved to server |

---

## 8. Who is Allowed to Use the Bot?

The script checks the sender's phone number against the database.

```
Incoming number ($from)
        ↓
Check employees table (contact_number column)
        ↓ found?             ↓ not found?
   Employee/Counsellor    Check student table
                          (mother_mobile / father_mobile)
                                ↓ found?        ↓ not found?
                              Parent           BLOCKED → exit silently
```

Three allowed user types:

| User Type | Who | Identified By |
|---|---|---|
| `employee` | Teacher or Staff | `employees.contact_number` |
| `counsellor` | Counsellor (role ID 192) | Same as employee + `employee_role = 192` |
| `parent` | Mother or Father | `student.mother_mobile` or `student.father_mobile` |

If the number is not found → **no reply, script exits quietly.**

---

## 9. Voice Note Flow (Step by Step)

```
1. User sends voice note on WhatsApp
2. Meta sends webhook with audio ID
3. Your server fetches the audio file URL using $token
4. Downloads the .ogg audio file
5. Sends it to OpenAI Whisper API using $gptkey
6. Gets back the transcribed text
7. Sends confirmation back to user: "I heard: [text]"
8. Processes the text as if user had typed it
```

---

## 10. Routing Logic

After identifying the user, the message is routed to the right handler file:

```
Counsellor  → demo-counsellor-enquiry-link.php

Employee/Teacher
    ├── Button: hw_update / hw_add / hw_cancel  → demo-teacher-homework.php
    ├── Button: homework_option                 → demo-teacher-homework.php
    ├── Button: attendance_option               → demo-teacher-greeting.php
    ├── Message contains homework keywords      → demo-teacher-homework.php
    ├── Message contains attendance keywords    → demo-teacher-greeting.php
    ├── Message contains collection keywords    → demo-daily-collection.php
    └── Default (no match)                      → demo-teacher-greeting.php

Parent      → demo-parent-greeting.php
```

### Homework Keywords Detected
`homework, hw, assignment, class, subject, math, science, english, exercise, chapter, complete, read, write, solve, learn, study, prepare, submit, due, from, to`

### Attendance Keywords Detected
`attendance, absent, present, roll, student, all present, all absent, mark, hazari, hajiri` (also Telugu and Hindi words)

---

## 11. File/Media Handling

When a teacher sends a document or image:

```
Receive media ID from webhook
        ↓
Check if it's a homework attachment (by keywords)
        ↓ yes                    ↓ no
Skip download now          Send "Processing File..." to user
Pass to homework handler   Download file from Meta using $token
                           Save to /uploads/homeworks/ folder
                           Store public URL in $attachment_path
```

---

## 12. Global Variables Passed to Handler Files

All data is passed via `$GLOBALS` so the included handler files can use them:

```php
$GLOBALS['user_type']    // 'employee', 'counsellor', or 'parent'
$GLOBALS['user_id']      // Database ID of the user
$GLOBALS['user_name']    // Display name
$GLOBALS['from']         // Sender's WhatsApp number
$GLOBALS['msg']          // The message text (or transcribed voice text)
$GLOBALS['token']        // WhatsApp API token (for sending replies)
$GLOBALS['phoneNumberId']// WhatsApp Phone Number ID
$GLOBALS['schoolify']    // Active DB connection
$GLOBALS['gptkey']       // OpenAI key
```

---

## 13. Logging

Every action is logged to `demo-log.txt` with a timestamp:

```
[2025-05-25 10:30:00] ========== SCRIPT STARTED ==========
[2025-05-25 10:30:00] Received TEXT message from 919876543210: hello
[2025-05-25 10:30:00] AUTHORIZED: Employee Ravi Kumar (ID: 45) with number 919876543210
[2025-05-25 10:30:00] Routing to attendance handler (employee)
```

Useful for debugging when messages aren't being processed correctly.

---

## 14. Security Issues to Fix

| Issue | Risk | Fix |
|---|---|---|
| `$token` not defined in file | All replies silently fail | Add to config section |
| `$gptkey` hardcoded and exposed | API key visible in code | Move to `.env` file |
| DB password hardcoded | Database credentials exposed | Move to `.env` file |
| `sk-None-` OpenAI key | Voice notes won't work | Generate new key at platform.openai.com |

### Recommended Fix — Use a `.env` file:
```
# .env file (never commit to git)
WHATSAPP_TOKEN=EAAxxxxxxxxx
WHATSAPP_PHONE_ID=1234567890
OPENAI_KEY=sk-proj-xxxxxxxxx
DB_PASSWORD=your_password
```

---

## 15. Where to Find Your Tokens

| Token | Location |
|---|---|
| `$token` + `$phoneNumberId` | developers.facebook.com → Your App → WhatsApp → API Setup |
| Permanent `$token` | Meta Business Settings → System Users → Generate Token |
| New `$gptkey` | platform.openai.com → API Keys → Create new secret key |

---

## Summary — How Everything Connects

```
WhatsApp User
     │
     │  sends message
     ▼
Meta Servers
     │
     │  POST webhook
     ▼
demo.php (this file)
     │
     ├── DB lookup → Who is this user?
     │
     ├── Voice? → OpenAI Whisper ($gptkey) → text
     │
     ├── Media? → Download using $token → save file
     │
     ├── Route to correct handler file
     │
     └── Handler sends reply
              │
              │  POST to Meta API
              │  using $token + $phoneNumberId
              ▼
         WhatsApp User receives reply
```

---

*Document covers: webhook flow, token types, user authentication, message routing, media handling, voice processing, logging, and security recommendations.*
