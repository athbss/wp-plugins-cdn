# WordPress Plugins Distribution Server

שרת הפצה מרכזי לפלאגיני WordPress של AT.

## מבנה תיקיות

```
public/
├── at-agency-sites-manager/
│   ├── at-agency-sites-manager.zip
│   └── plugin-info.json
└── wordpress-ai-assistant/
    ├── wordpress-ai-assistant.zip
    └── plugin-info.json
```

## API Endpoints

- `GET /plugins` - רשימת כל הפלאגינים
- `GET /:plugin/info.json` - מידע על פלאגין
- `GET /:plugin/download` - הורדת קובץ ZIP
- `GET /health` - בדיקת תקינות

## שימוש

הפלאגינים בוורדפרס צריכים להצביע ל:
```
Update URI: https://your-domain.com/plugin-name/info.json
```
