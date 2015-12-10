<h1>Säkerhet</h1>

Jag började med att titta på säkerheten eftersom jag anser det vara viktigast. När jag har gått igenom applikationen
inser jag att den fortfarande är i utvecklingsstadiet varpå jag tycker det är bättre att se mina kommentarer om åtgärder
som en checklista snarare kritik. Åtgärderna är listade i ordning efter hur vanliga eventuella attacker är och hur stora
skadorna kan bli.

<h5>Sql Injection [1]</h5>
Applikationen är i nuläget vidöppen för sk Sql injections. Vilket betyder att elaka användare skjuter in skadlig kod
i applikationen gärna via formulär av olika slag. Databaser är särskilt utsatta för denna typen av attacker eftersom de
ofta innehåller värdefull data. Injections kan resultera i att konfidentiell data läcker ut, att hela databaser
förstörs eller till och med en total "hostile takeover" av värddatorn.

<b>Nuläge:</b>
Applikationen har ingen som helst validering av indata från användare, vare sig på klient eller på server. Inte heller
finns det något skydd i kommunikationen mellan applikation och databas vid loginfunktionen.

<b>Åtgärder:</b>
Validera all inkommande data. Strippa t ex  bort < och > så att en attacker inte kan skjuta in
t ex exekverbar javascriptkod. Finns det speciella tecken som behövs, gör en lista över de tillåtna tecknen, [2].
Ge användaren feedback på felaktiga inmatningar så skapas samtidigt en bättre användarupplevelse.

Använd prepared statements med variabel bindning, sk parameterized queries, i kommunikation med databasen. I parameterized queries
definerar man först all Sql-kod, sen skickar man in parametrarna till frågan. Det betyder att databasen kan skilja mellan kod och
data oavsett vad användaren har skickat in. Prepared statements säkerställer att en attacker inte kan ändra avsikten med databasfrågan
och bör alltid användas, [2]-[3].

Databasen SQLite 3 som används i det här fallet stödjer inte lagrade procedurer annars skulle jag rekommenderat det.
Skriver lite om det ändå ifall ni skulle ändra valet av databas. Skillanden mellan lagrade procedurer
och prepared statements är att lagrade procedurer skapas och lagras i databasen. Som utevecklare kan man begränsa applikationens
rättigheter till att enbart få använda lagrade procedurer i kommunikationen mot databasen. I övrigt har lagrade procedurer samma funktion som
parametiserade frågor och man kan som utvecklare välja enbart ett av sätten. Jag skulle personligen implementerat båda då
det kanske inte alltid är möjligt för databasadministratören att säkerställa att applikationens kod implementeras korrekt.
Men som sagt inte ett alternativ i detta fall så kör på prepared statements [2]-[3].


<h3>Referenser</h3>

[1] "Top 10, 2013" OWASP, juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A1-Injection. [Hämtad: 11 december, 2015].

[2] "Sql injection prevention cheat sheet" OWASP, november 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet. [Hämtad: 11 december, 2015].

[3] "Query parameterization cheat sheet" OWASP, november 2014 [Online] Tillgänglig: https://www.owasp.org/index.php/Query_Parameterization_Cheat_Sheet. [Hämtad: 11 december, 2015].


