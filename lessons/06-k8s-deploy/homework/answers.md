## שאלה 1
בתחילת השיעור בנינו את ה-images עם `-t shoplist-backend:local`. למה לא השתמשנו בשמות ש-Compose נתן (`docker-backend`)? ומה היה קורה בלי `imagePullPolicy: Never`?

## תשובה 1
Compose נותן שמות לפי שם התיקייה ולא אנחנו אלו שבוחרים את השמות,
בקבצי הyaml צריך שם קבוע וידוע מראש.

בלי Never, Kubernetes ינסה להוריד אתה-images מהאינטרנט, ושם אין image כזה,
לכן הpod לא יצליח וייכשל בשגיאת הורדה.


## שאלה 2
פענחו את `POSTGRES_PASSWORD` מה-Secret. איזו פקודה הרצתן? מה זה אומר על האבטחה של Secret?

## תשובה 2
כבר הרצנו קודם את הפקודה הזו, בבדיקת הפיענוח. זו הייתה:
[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("c2hvcHBhc3M="))
והיא החזירה shoppass.

base64 הוא קידוד ולא הצפנה. כל מי שיכולה לקרוא את ה‑Secret מפענחת אותו בפקודה אחת. ההגנה האמיתית היא הרשאות: מי מורשה לקרוא אותו (RBAC).

## שאלה 3
מחקתן את ה-pod של postgres והמוצרים נשארו. איזה קובץ אחראי לזה, ואיזו פקודה **כן** הייתה מוחקת אותם?

## תשובה 3
הקובץ שאחראי לזה הוא postgres-pvc.yml. ה‑PVC הוא "ארונית" שנשארת גם כשה‑pod נמחק, ו‑pod חדש מתחבר לאותה ארונית.
הפקודה שכן הייתה מוחקת את המוצרים: kubectl delete pvc postgres-pvc -n shoplist. זה כמו docker compose down -v.

## שאלה 4
ב-Compose היה `depends_on`. ב-Kubernetes אין. מה קורה אם ה-backend עולה לפני postgres? מה דואג שהוא לא יקבל בקשות לפני שהוא מוכן?

## תשובה 4

אם ה‑backend עולה לפני postgres, הוא לא מצליח להתחבר, והבדיקה שלו נכשלת. Kubernetes פשוט ממשיך לנסות שוב ושוב, עד ש‑postgres מוכן והכול מסתדר לבד.
מה ששומר עליו הוא ה‑readinessProbe. כל עוד הבדיקה ל‑/health נכשלת, ה‑pod במצב 0/1, ה‑Service לא מכניס אותו ל‑endpoints, ולכן לא שולח אליו בקשות.

## שאלה 5
ד8: מה ראיתן בדפדפן? האם ה-pods של backend נפלו? מה הראה `get endpoints`, ולמה?

בדפדפן: הנורית של backend הייתה אדומה, המוצרים לא הוצגו ("Backend unavailable"), וגם הסטטיסטיקה לא עבדה.
ה‑pods: לא נפלו. שניהם המשיכו להיות 1/1 Running.
get endpoints: הראה <none>, כלומר אין אף כתובת. ה‑Service חיפש pods עם המדבקה app: backnd, ועל ה‑pods הייתה המדבקה app: backend, ולכן הוא לא מצא אף pod לשלוח אליו בקשות.

## שאלה 6
שיניתן ערך ב-`app-config` והרצתן `apply`, אבל ה-backend עדיין רואה את הערך הישן. למה, ומה הפתרון?

למה: משתני סביבה נקראים רק פעם אחת, ברגע שה‑pod עולה. שינוי ב‑ConfigMap לא משנה pod שכבר רץ.
הפתרון: להפעיל מחדש את ה‑pods, כדי שיעלו חדשים ויקראו את הערך החדש:
kubectl rollout restart deployment/backend -n shoplist

## שאלה 7
כמה פעמים כתבתן `shoplist` ב-12 הקבצים? ואם מחר נרצה להריץ עותק שני של ShopList ב-namespace בשם `shoplist-test`, כמה קבצים נצטרך לשנות?

## תשובה 7
20
השורה namespace: shoplist מופיעה בכל קובץ, ולכן כדי לעבור ל‑shoplist-test צריך לשנות את כל 13 הקבצים (12 ועוד app-config.yml).
זה מסורבל ומועד לטעויות, וזו בדיוק הבעיה ש‑Helm פותר בשיעור הבא: כותבים את הערך פעם אחת במקום אחד.