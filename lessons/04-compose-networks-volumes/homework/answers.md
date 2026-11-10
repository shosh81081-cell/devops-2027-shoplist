# שיעורי בית 4: תשובות

## 1. ל-backend אין ports. איך stats מגיע אליו?
*תשובה: compose יצר רשת שבה כל שירות מוכר לפי השם שלו, stats נחוץ רק כדי להגיע מהמחשב עצמו,ports פונה ישירות לhttp://backend:5000


## 2. down ואז up: המוצרים נשארו? ומה עם down -v?
*תשובה: down -v מוחק ממש את המוצרים.

## 3. השורה ../sql/init.sql:... איזה סוג volume זה? מה יקרה אם נשנה את init.sql אחרי שהמסד נוצר?
*תשובה: סוג: זה bind mount. יודעים לפי זה שבצד שמאל יש נתיב לקובץ במחשב (../sql/init.sql), ולא שם של volume (כמו postgres-data).

שינוי אחרי שהמסד נוצר: לא יקרה כלום. postgres מריץ את הקבצים שבתיקייה docker-entrypoint-initdb.d רק בפעם הראשונה, כשה-volume ריק. כדי שהשינוי ייכנס, צריך למחוק את הנתונים (docker compose down -v) ואז up -d.

## 4. שם הרשת ושם ה-volume, ומאיפה הגיעו השמות
רשת: docker_default
Volume: docker_postgres-data

מאיפה השמות: Compose מוסיף לכל שם קידומת של שם הפרויקט, ושם הפרויקט הוא כברירת מחדל שם התיקייה שבה נמצא קובץ ה-compose, כלומר docker.

הרשת נקראת default כי לא הגדרנו רשת בעצמנו, ו-Compose יצר אחת אוטומטית.
ה-volume נקרא postgres-data כי זה השם שנתנו לו בקובץ.

## 5. Dockerfile.multistage: מה יש ב-image הסופי ומה אין? למה אין CMD? למה stats לא צריך multi-stage?
מה יש ומה אין: ב-image הסופי יש רק Nginx, את קבצי האתר המוכנים מתיקיית dist ואת nginx.conf. אין בו Node, את build.js או את קבצי המקור. כל אלה נשארו בשלב הבנייה ("המטבח").

למה אין CMD: ה-image מבוסס על nginx, שכבר מגיע עם CMD משלו שמפעיל את Nginx, ואנחנו יורשים אותו.

למה stats לא צריך multi-stage: ל-stats אין שלב בנייה. אין חבילות להתקין ואין קבצים לעבד, ו-Node נדרש גם בזמן הריצה עצמה. לכן אין כלים מיותרים שאפשר להשאיר מאחור.

## 6. שלושת הגדלים, ולמה multi כמעט כמו single
הגדלים:

single: 92.7MB
multi: 92.7MB
kitchen: 244MB

למה multi כמעט כמו single: שניהם מבוססים על אותו image של nginx:1.30-alpine, ושניהם מכילים רק את קבצי האתר הקטנים. ה-multi לוקח מהמטבח רק את תיקיית dist, ומשאיר מאחור את Node וכלי הבנייה. לעומת זאת, kitchen גדול בהרבה כי הוא מכיל את כל Node.

## 7. למה /api/stats מגיע ל-stats? ומה הלוכסן בסוף עושה ל-/api/products?
: Nginx בוחר את ה-location עם ההתאמה הארוכה ביותר. /api/stats ארוך ומדויק יותר מ-/api/, ולכן הוא גובר.

מה הלוכסן בסוף עושה: כשיש לוכסן בסוף http://backend:5000/, Nginx מוריד את החלק /api/ מהכתובת לפני שהוא מעביר אותה. כך /api/products מגיע ל-backend בתור /products, וזו בדיוק הכתובת שה-backend מכיר. בלי הלוכסן, הבקשה הייתה מגיעה בתור /api/products, וה-backend לא היה מוצא אותה.