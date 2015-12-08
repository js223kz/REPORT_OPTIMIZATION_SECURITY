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
för resterande 80-90%.[1, s. 1], [3]. Det finns flera sätt att minska på antalet Http-förfrågningar.
Nedan listar jag några som rör just den här applikationen.</p>

<h3>Ta bort oanvända resurser</h3>
Jag har uppmärksammat ett par filer som aldrig används i applikationen. Kanske behövs de i ett senare utvecklingsskede,
men fram tills dess att den implementationen sker bör de tas bort.
T ex: b.jpeg, clock.png, delete.png, favicon.png är samma som logo.png och används inte, Dump.html

Vidare försöker browsern ladda en sida som skickar tillbaka en 404:a : materialize.js






[1] Steve Souders, The importance of web performance i <italic>High Performance Web Sites: Essential Knowledge for Frontend Engineers</italic>,
Redaktör, Andy Oram. Sebastopol, California: O'Reilly Media, 2007, 1-5.

[2] Steve Souders, HTTP overview i <italic>High Performance Web Sites: Essential Knowledge for Frontend Engineers</italic>,
Redaktör, Andy Oram. Sebastopol, California: O'Reilly Media, 2007, 6-9.

[3] Steve Souders,Rule 1: make fewer http requests i <italic>High Performance Web Sites: Essential Knowledge for Frontend Engineers</italic>,
Redaktör, Andy Oram. Sebastopol, California: O'Reilly Media, 2007, 10.