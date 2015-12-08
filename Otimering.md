<h1>Optimering</h1>

<p>Det finns en del utrymme för optimering av applikationens prestanda.
Jag har i första hand tittat på hur applikationen kan optimeras frontend
och det med relativt enkla åtgärder. Säkerhetsapekten är såklart den viktigaste,
men för användarupplevelsen rekommenderar jag att man genomför så mycket som
möjligt av nedanstående åtgärder. Primärt rör det sig om laddningstider[1]-[2].
Då jag inte har någon uppfattning om antalet användare av applikationen
kan jag inte uttala mig om miskade kostnader i utnyttjandet av bandbredd,
men vid ett stort antal användare blir det en trevlig bonus.</p>


<h2>Minska antalet Http-förfrågningar</h2>
<p>Ju fler förfrågningar som skickas från browsern till webbservern ju längre tid ta sidan att laddas.
Faktum är att enligt undersökningar står HTML-dokumentet endast av 10-20% av laddningstiden för slutanvändaren.
Alla andra komponenter så som bilder, css-filer, skriptfiler , som det refereras till i HTML-dokumentet står
för resterande 80-90%.[1, s.1], [3]. Längre laddningstider innebär sämre användarupplevelse.
Det finns flera sätt att minska på antalet Http-förfrågningar. Nedan listar jag några som rör just den här applikationen.</p>

<h3>Ta bort oanvända resurser</h3>
En bild som laddas, men aldrig syns, b.jpeg, skapar onödig laddningstid.
Browsern försöker ladda, men servern skickar tillbaka en 404:a, materialize.js, skapar onödig laddningstid.
Filer som inte laddas, men som heller inte används, clock.png, delete.png, favicon.png, dump.html.

Rensa bort outnyttjade resurser även om de inte förlänger laddningstiderna.

<h3>Cachea komponenter [4]</h3>
Första gången en användare besöker er sida laddas alla komponenter in och laddningstiden blir längre. Hur lång den
sedan blir vid efterföljande besök på samma sida beror på huruvida komponenterna cachaes eller inte. Dynamiska HTML-sidor
cachar man inte, men bilder, css-filer och skriptfiler går alldeles utmärkt att cacha om de inte ändras stup i kvarten.
Gör en utvärdering av komponenterna som laddas, har de inte ett närliggande "last modified datum" är de säkert kandidater för
cachning [4, s.22-27].

Som utvecklare kan man säkerställa att komponter sparas i browserns cache genom att lägga till en Future Expires Header.
Future Expires Header använder ett specifikt datum för att hålla koll på när en komponent måste laddas om. Det innebär
att när det datumet löper ut måste servern tillhandahålla ett nytt datum [4, s.22-23].
Ett smidigare sätt är då att använda Cache-control och max age, som bestämmer hur länge en komponent får cachaeas i sekunder.
De browsers som inte stödjer HTTP/1.1 kan inte använda Cache-Control, men då det är väldigt få bör det inte ställa till
några problem [4, s.22-24].

Använder man inte future expires kommer browsern ändå att spara resursen i sin cache och sedan när resursen efterfrågas
skicka en sk Conditional GET till servern. Om komponenten inte har ändrats skickar servern tillbaka ett svar till browsern
att avnända den komponent den har i sin cache. Med andra ord en onödig tripp till servern som förlänger sidans laddningstid, [2], [4, s.28].

I denna applikationen är alla komponenters future expire satt till -1, vilket säger till browsern att inte cacha något.
Det är en sanning med modifikation att browsern inte cachar något. det gör den, men varje gång en resurs efterfrågas
skickar den en Conditional GET till servern och frågar om resursen


Slutsats: eftersom applikationens användare kommer att vara återkommande besökare är cachning en viktigt åtgärd för
användarupplevelsen och jag rekommenderar verkligen att det implementeras [4, s.24-25]. Något att tänka på när man låter
browsers och prxies cachea komponenter är hur man sköter versionshantering och uppdatering av filer. Läs mer om Revving
Filenames [4, s.27].




[1] Steve Souders, The importance of web performance i <italic>High Performance Web Sites: Essential Knowledge for Frontend Engineers</italic>,
Redaktör, Andy Oram. Sebastopol, California: O'Reilly Media, 2007, 1-5.

[2] Steve Souders, HTTP overview i <italic>High Performance Web Sites: Essential Knowledge for Frontend Engineers</italic>,
Redaktör, Andy Oram. Sebastopol, California: O'Reilly Media, 2007, 6-9.

[3] Steve Souders, Rule 1: make fewer http requests i <italic>High Performance Web Sites: Essential Knowledge for Frontend Engineers</italic>,
Redaktör, Andy Oram. Sebastopol, California: O'Reilly Media, 2007, 10.

[4] Steve Souders, Rule 3: add an expires header i <italic>High Performance Web Sites: Essential Knowledge for Frontend Engineers</italic>,
Redaktör, Andy Oram. Sebastopol, California: O'Reilly Media, 2007, 22-28.

