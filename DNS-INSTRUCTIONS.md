# shimi.land root: מעבר מבייס44 לגיטהאב Pages
## בספייסשיפ (DNS של shimi.land)
1. מחק: A @ -> 216.24.57.1
2. הוסף 4 רשומות A על @ (root):
   185.199.108.153
   185.199.109.153
   185.199.110.153
   185.199.111.153
3. (אופציונלי) CNAME www -> sunraz.github.io
4. אל תיגע ברשומות aidoor / studio / studio2 / הקדמה — הן כבר נכונות.
## אחרי ההחלפה (דקות עד שעה)
- GitHub: Settings -> Pages -> Enforce HTTPS (יופיע כשהתעודה תונפק)
- אימות: curl -I https://shimi.land -> 200, <title> "רשת שימי — shimi.land"
