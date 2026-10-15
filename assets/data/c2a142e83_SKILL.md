---
name: meta-business-agent-training
description: Configure, train, test, and safely deploy Meta Business Agents for WhatsApp with UI skills, tools, and verified handoffs.
---

# אימון והפעלת Meta Business Agent ב-WhatsApp

**גרסה:** 1.1, 23/09/2026. סקיל כללי, ללא מזהים ופרטי גישה של עסק מסוים.

## עקרונות
Meta Business Agent הוא הסוכן של Meta שפועל בתוך WhatsApp. אין לבלבל בינו לבין chatbot באפליקציה חיצונית שמשיב מאותו מספר. כל שלב דורש הרשאה תקינה על ה-WABA והמספר. אין להטמיע טוקנים בקוד, בהודעות או בתיעוד. אל תכריז שסוכן פעיל על סמך סימולטור בלבד.

## 1. גילוי ותיעוד
גלו את `WABA_ID`, `PHONE_NUMBER_ID`, סטטוס המספר, ה-app webhook וה-owner של השיחה. מזהה `entity_id` של תצורת הסוכן הוא `PHONE_NUMBER_ID`. בצעו GET להגדרות, skills, UI skills, connectors ו-allowlist לפני שינויים, שמרו גיבוי לסקילים הקיימים. השתמשו בכותרת `Authorization: Bearer <authorized token>` ו-`X-API-Version: 2.0.0` אלא אם צוין אחרת.

## 2. Agent Skills
`GET/POST https://api.facebook.com/<PHONE_NUMBER_ID>/agent_config/skills`; עדכון קיים ב-`PUT /<skill_id>` עם השדות לשינוי. מגדירים לכל skill `title`, `description` (מתי משתמשים בו), `skill` (ההוראה עצמה), `status=active` ו-`channel=whatsapp` אם נדרש.

כתבו skills ממוקדים: פתיחה וזיהוי צורך, שיחת מכירה קצרה, איסוף פרטי ליד בהסכמה, ניהול מסמכים, מעבר לאדם וגבולות מקצועיים. שאלו שאלה אחת בתור, אל תחזרו לתפריט לאחר בחירה, אל תבטיחו ריבית/אישור/מועד שלא אומת. לעולם אל תגידו שפנייה נמסרה, שנפתח תיק, או שמישהו יחזור לפני שכלי אמיתי החזיר הצלחה מתאימה.

## 3. כפתורים ושפה חופשית
`GET/POST https://api.facebook.com/<PHONE_NUMBER_ID>/agent-ui-skills`. הגדרת UI skill לדוגמה:
`{"title":"short-replies-and-expert","component_type":"interactive_reply_buttons","status":"enabled","instruction":"כששואלים שאלת מיון, שלח הודעה קצרה עם עד 3 כפתורי תשובה: שתי תשובות רלוונטיות וכפתור לדבר עם מומחה. אפשר תמיד לענות גם בטקסט חופשי. אל תטען למסירה בלי אישור מכלי אמיתי."}`

WhatsApp מאפשר עד שלושה כפתורי תשובה בהודעה, לא כפתור קבוע במסך השיחה. גם הסוכן אינו מחויב לבחור UI skill בכל תור, לכן מאמתים הופעה בפועל בשיחת ניסוי חיה. אפשר להשתמש גם ברשימה (`interactive_list`) וב-WhatsApp Flow אם נדרש טופס ארוך יותר.

## 4. Connector לביצוע פעולות
`GET/POST https://api.facebook.com/<PHONE_NUMBER_ID>/agent_connectors` ולאחר יצירה `GET/POST /<connector_id>/tools`. חברו endpoint מאובטח ב-HTTPS עם API_KEY מוגבל, ללא טוקן שמופיע בטקסט לקוח. הפעולות חייבות להיות צרות, למשל `submit_consented_lead`: שליחת פרטי קשר, צורך, מסמכים קיימים/חסרים והסכמה למערכת CRM, פתיחת/עדכון תיק, והתראה למנהלים המורשים. קשרו את מזהה משתמש WhatsApp באמצעות מאקרו `WHATSAPP_PHONE_NUMBER` ואת מזהה השיחה באמצעות `WHATSAPP_CONVERSATION_ID` ולא רק מטקסט שהמודל מנחש. הגדירו `request_definition` עם body.params בסכמות מפורשות. בדקו את הכלי ישירות, ואז ב-`agent_test`, ואז בשיחה חיה. ענו למודל רק עם מה שבאמת הצליח. בצעו מניעת כפילויות ורישום ניסיון מסירה פר-נמען. אל תשלחו מסמך רגיש בהודעת טקסט כשנדרש מסלול העלאה מאובטח.

## 5. בדיקה מבודדת
`POST https://api.facebook.com/<PHONE_NUMBER_ID>/agent_test` עם `{"user_msg":"..."}`; השתמשו ב-`conversation_id` שמוחזר להמשך רב-תורי. בדקו לפחות: ליד חדש, בקשת מומחה, מענה חופשי במקום כפתור, מסמכים חסרים, שיחה שאינה מתקדמת, פעולה שנכשלה ותחום מחוץ לשירות. תגובה טובה בסימולטור אינה מוכיחה מסירה או התנהגות WhatsApp חיה. בפרט בדקו שהסוכן לא ממציא handoff כשאין כלי מחובר.

## 6. Rollout מבוקר
`GET https://api.facebook.com/<PHONE_NUMBER_ID>/agent_config/settings`. הוסיפו רק מספרי בדיקה מזוהים ב-`/agent_config/allowlist` אחרי GET כדי למנוע כפילויות. הפעילו `rollout.enabled=true` עם `ai_audience=ALLOWLISTED_ONLY`. אל תפתחו `EVERYONE` לפני בדיקת אמצעי חיוב אם נדרש, מענה חי, חסימת בוטים כפולים ו-handoff מאומת מקצה לקצה.

## 7. הודעות יזומות ושליטת שיחה
להודעה יזומה מחוץ לחלון שירות של 24 שעות השתמשו בתבנית WhatsApp במצב APPROVED, דרך Cloud API. הודעה שנשלחת מאפליקציית Cloud API יכולה להעביר לה שליטה ב-thread ולהשתיק את Meta Agent. כשהאפליקציה היא בעלת השליטה, שחררו את השיחה באמצעות `POST https://api.facebook.com/business/whatsapp/phone_numbers/<PHONE_NUMBER_ID>/thread_control`, עם `X-API-Version: 1.0.0` וגוף `{"messaging_product":"whatsapp","action":"release","to":"<customer_phone>"}`. `pass` לסוכן אינו תחליף שנבדק כאן. אם מוחזרת `Caller is not the thread owner`, אל תפרשו זאת כהוכחה שהכול תקין: בדקו מי באמת ענה בשיחה.

## 8. אימות חי וסגירה
אחרי הודעה חיה ממספר בדיקה, קראו `GET https://api.facebook.com/<PHONE_NUMBER_ID>/insights/conversations` עם טווח תאריכים ו-`GET /insights/conversations/turns?user_phone_number=<digits>`. ודאו `ai_threads>0` ו-turns המתאימים, והשוו ל-webhook logs ולהודעות שיוצאות מהקוד הישן. כשאין turns אל תאשימו היסטוריית לקוח בלי ראיה: בדקו thread control, app override, routing ותשובות כפולות. אימות ליד מחייב תיק אמיתי ותוצאה של כל מסירה נדרשת, לא רק תשובת טקסט יפה. אין להציג `Graph 200` כאישור מסירה לטלפון: זה רק אישור קבלת בקשת שליחה.

## מקורות רשמיים
https://developers.facebook.com/documentation/meta-business-agent/llms.txt
https://developers.facebook.com/documentation/meta-business-agent/usage-guides/writing-ui-skills.md
https://developers.facebook.com/documentation/meta-business-agent/reference/configure/connectors.md
https://developers.facebook.com/documentation/meta-business-agent/reference/configure/connector-tools.md
