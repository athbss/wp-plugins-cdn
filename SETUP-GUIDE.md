# WordPress Plugins Distribution System - Setup Guide

מדריך התקנה והפעלה של מערכת הפצת עדכונים לפלאגיני WordPress דרך GitHub Actions ו-Railway.

## סקירה כללית

המערכת מורכבת משלושה חלקים:
1. **GitHub Actions** - בונה ZIP ויוצר GitHub Release עם כל tag
2. **GitHub Releases** - משמש כ-CDN לקבצי הZIP (חינם ומהיר)
3. **Railway CDN Server** - שרת סטטי שמספק plugin-info.json מעודכן

## ארכיטקטורה

```
פלאגין WordPress
    ↓ (בודק עדכונים)
GitHub Raw: plugin-info.json
    ↓ (מכיל URL להורדה)
GitHub Releases: plugin.zip
```

## שלב 1: הגדרת Railway CDN

### 1.1 חיבור Repository ל-Railway

1. היכנס ל-Railway Dashboard: https://railway.com/project/f22150f3-b9a2-4926-9b79-2497008fa501
2. לחץ על השרת `cdn-server`
3. לחץ על **Settings** → **Source**
4. בחר **Connect Repo** ובחר: `athbss/wp-plugins-cdn`
5. השרת יפרוס אוטומטית

### 1.2 הוספת דומיין מותאם (אופציונלי)

1. בלשונית **Settings** → **Networking**
2. לחץ **Generate Domain** (אם עדיין לא קיים)
3. או הוסף **Custom Domain** משלך

**URL מותאם:** https://updates.amiteam.io

## שלב 2: הגדרת GitHub Secrets

כל repository של פלאגין (at-agency-sites-manager-plugin, wordpress-ai-assistant) צריך להגדיר:

### בRepository של כל פלאגין:

1. **Settings** → **Secrets and variables** → **Actions**
2. לחץ **New repository secret**

הוסף:
- **`RAILWAY_CDN_REPO`** = `athbss/wp-plugins-cdn`
- **`RAILWAY_CDN_TOKEN`** = Personal Access Token עם הרשאות repo

### יצירת Personal Access Token:

1. GitHub → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. **Generate new token (classic)**
3. תן לו הרשאות: `repo` (כל ההרשאות)
4. העתק את הtoken ושמור בsecrets

## שלב 3: תהליך פרסום גרסה חדשה

### 3.1 עדכון גרסה

ערוך את קובץ הראשי של הפלאגין (`plugin-name.php`):

```php
/**
 * Version: 1.0.1  // ← שנה את הגרסה
 */
```

### 3.2 Commit & Push

```bash
git add .
git commit -m "bump: version 1.0.1"
git push origin main
```

### 3.3 יצירת Tag

```bash
git tag -a v1.0.1 -m "Version 1.0.1"
git push origin v1.0.1
```

### 3.4 תהליך אוטומטי

GitHub Actions יריץ אוטומטית:
1. ✅ בניית קובץ ZIP
2. ✅ יצירת GitHub Release
3. ✅ העלאת plugin-info.json
4. ✅ דחיפה לRailway CDN Repository
5. ✅ Railway יעדכן את השרת אוטומטית

## שלב 4: בדיקת מערכת העדכונים

### בדיקה ידנית:

```bash
# בדוק את plugin-info.json
curl https://raw.githubusercontent.com/amit-trabelsi-digital/at-agency-sites-manager-wp-plugin/main/plugin-info.json

# בדוק את קובץ הZIP
curl -I https://github.com/amit-trabelsi-digital/at-agency-sites-manager-wp-plugin/releases/download/v0.12.0/at-agency-sites-manager.zip
```

### בדיקה ב-WordPress:

1. התקן את הפלאגין באתר WordPress
2. לך ל-**Dashboard** → **Updates**
3. צריך להופיע עדכון אם יש גרסה חדשה

## מבנה קבצים

```
wp-plugins-cdn/                    # Railway CDN Repository
├── server.js                      # Express server
├── package.json
├── public/                        # קבצים סטטיים
│   ├── at-agency-sites-manager/
│   │   └── plugin-info.json
│   └── wordpress-ai-assistant/
│       └── plugin-info.json
└── README.md

at-agency-sites-manager-plugin/    # Plugin Repository
├── .github/
│   └── workflows/
│       └── release-to-railway.yml # GitHub Actions
├── plugin-info.json               # מעודכן אוטומטית
└── at-agency-sites-manager.php    # Update URI מוגדר

wordpress-ai-assistant/             # Plugin Repository
├── .github/
│   └── workflows/
│       └── release-to-railway.yml # GitHub Actions
├── plugin-info.json               # מעודכן אוטומטית
└── wordpress-ai-assistant.php     # Update URI מוגדר
```

## API Endpoints

Railway CDN מספק:

- `GET /plugins` - רשימת כל הפלאגינים
- `GET /:plugin/info.json` - plugin-info.json
- `GET /health` - בדיקת תקינות

## Troubleshooting

### העדכון לא מופיע ב-WordPress

1. בדוק ש-Update URI בפלאגין נכון
2. וודא ש-plugin-info.json מעודכן ב-GitHub
3. מחק cache ב-WordPress:
   ```php
   delete_site_transient('update_plugins');
   ```

### GitHub Actions נכשל

1. בדוק GitHub Secrets (RAILWAY_CDN_REPO, RAILWAY_CDN_TOKEN)
2. בדוק שיש הרשאות write ל-Railway CDN repo
3. בדוק logs ב-Actions tab

### Railway לא מתעדכן

1. בדוק ש-Repository מחובר ב-Railway Dashboard
2. בדוק Deployment logs ב-Railway
3. וודא ש-public/ תיקייה קיימת ב-repo

## קישורים חשובים

- **Railway Project:** https://railway.com/project/f22150f3-b9a2-4926-9b79-2497008fa501
- **Railway CDN URL:** https://cdn-server-production.up.railway.app
- **GitHub CDN Repo:** https://github.com/athbss/wp-plugins-cdn
- **AT Agency Plugin:** https://github.com/amit-trabelsi-digital/at-agency-sites-manager-wp-plugin
- **AI Assistant Plugin:** https://github.com/athbss/wp-ai-bro

## תזרים עבודה מלא

```
1. עדכן גרסה בקובץ PHP
2. git commit & push
3. git tag v1.0.1 & push
   ↓
4. GitHub Actions מופעל
   ↓
5. בונה ZIP + יוצר Release
   ↓
6. מעדכן plugin-info.json
   ↓
7. דוחף ל-Railway CDN repo
   ↓
8. Railway פורס אוטומטית
   ↓
9. WordPress בודק עדכונים
   ↓
10. משתמש רואה עדכון זמין!
```

## תחזוקה

- GitHub Releases מאחסנים את קבצי הZIP - **חינם**
- Railway Free Tier: 500 שעות/חודש (מספיק לשרת CDN)
- כל העדכונים אוטומטיים דרך Git tags

---

**נוצר:** פברואר 2026
**Author:** Amit Trabelsi
