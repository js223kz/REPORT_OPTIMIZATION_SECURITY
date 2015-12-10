<h1>Säkerhet</h1>

Jag började med att titta på säkerheten eftersom jag anser det vara viktigast. När jag har gått igenom applikationen
inser jag att den fortfarande är i utvecklingsstadiet varpå jag tycker det är bättre att se mina kommentarer om åtgärder
som en checklista snarare kritik.

<h5>Injection [1]</h5>
Applikationen är i nuläget delvis för sk injections. Vilket betyder att elaka användare skjuter in skadlig kod
i applikationen. Databaser är särskilt utsatta för denna typen av attacker eftersom de ofta innehåller värdefull data.
Injections kan resultera i att konfidentiell data läcker ut, att hela databaser förstörs eller till och med en total
"hostile takeover" av värddatorn. Båda indata och utdata måste valideras så att skadlig kod inte lyckas ta sig in i applikationen
för att sedan köras ut till intet ont anande användare via DOM-dokumentet.

<b>Nuläge:</b>
När en användare loggar in finns inget extra skydd i kommunikationen mellan applikationen och databasen.

<b>Åtgärder:</b>
Använd prepared statements med variabel bindning, sk parameterized queries, i kommunikation med databasen. I parameterized queries
definerar man först all Sql-kod, sen skickar man in parametrarna till frågan. Det betyder att databasen kan skilja mellan kod och
data oavsett vad användaren har skickat in. Prepared statements säkerställer att en attacker inte kan ändra avsikten med databasfrågan
och bör alltid användas, [2]-[3]. Det är redan gjort när meddelanden läggs till databasen så det är bara att kopiera det exemplet.

Databasen SQLite 3 som används i det här fallet stödjer inte lagrade procedurer annars skulle jag rekommenderat det.
Skriver lite om det ändå ifall ni skulle ändra valet av databas. Skillanden mellan lagrade procedurer
och prepared statements är att lagrade procedurer skapas och lagras i databasen. Som utevecklare kan man begränsa applikationens
rättigheter till att enbart få använda lagrade procedurer i kommunikationen mot databasen. I övrigt har lagrade procedurer samma funktion som
parametiserade frågor och man kan som utvecklare välja enbart ett av sätten. Jag skulle personligen implementerat båda då
det kanske inte alltid är möjligt för databasadministratören att säkerställa att applikationens kod implementeras korrekt.
Men som sagt inte ett alternativ i detta fall så kör på prepared statements [2]-[3].


<h5>Användaruppgifter [4]</h5>
Med tanke på hur vanligt det är med Sql-injections är det av yttersta vikt att lagra användaruppgifter på ett säkert sätt.
Tänk på vad som kan hända om en attacker får tag på hela tabeller med användarnamn och lösenord och lösenorden har lagrats
i klartext.

<b>Nuläge:</b>
Lösenorden lagras i databasen i klartext. Vad jag kan se sker ingen verifiering av användaren inmatat lösenord med
ett saltat+hashat lösenord från databasen.

<b>Åtgärder:</b>
Lösenord bör saltas och hashas innan de sparas i databasen. När en användare sedan loggar in verifierar man det inskrivna
lösenordet mot det saltade+hashade som finns i databasen. Ett saltat+hashat lösenord kan aldrig återställas och om två olika
användare har samma lösenord blir saltning+hashning aldrig samma. Visst, man förhindrar inte att en attacker kan komma åt
tabellen med uppgifter, men man gör det i stort sett omöjligt för dem att knäcka lösenorden. Man bör också se till att varje
enskild användare har ett eget unikt salt [5].

<h5>Sessionshantering[4]</h5>
Det är självklart att sätta sessioner för att behålla information genom hela applikationen när en inloggare har loggat in.
Annars skulle en användare vara tvungen att logga in vid varje ny förfrågan och det är ohållbart. Något att tänka på när man
sätter sessioner är dock att det är guld för en attacker att få tag på en legitim session för att sedan utge sig för att
vara den inloggade legitime användaren och göra allt som den kan, sk session hijacking [6].

<b>Nuläge:</b>
Som det är nu används användarid som ett av sessionsvärdena och användarens roll i klartext som värde i den andra.
Kommunikation mellan klient och server går via en okrypterad lina.
Applikationen utnyttjar inte möjligheten att sätta sessionsflaggor som Httponly och secure.

<b>Åtgärder:</b>
Byt sessionsvärden något mindre känsligt och inte så lätt att gissa sig till. Rollen som admin är typiskt farlig då den
är lätt att gissa sig till värdet "admin" och såklart värdefull att komma åt, [6].

Sätt sessionsflaggorna Httponly till true och secure till true, [6]. Se filen config/express.js

Använd er också av SSL, krypterad kommunikation mellan klient och server, det bästa skyddet på session hijacking, [6].

<h5>Sessionshantering vid utloggning, [4]</h5>
Ha man satt sessioner vid t ex inloggning måste man vara noga med att de förstörs när en användare loggar ut, [6].
När en legitim användare loggat ut och lämnar datorn är det fullt möjligt för vem som helst att besöka de sidor den
legitime användaren just besökt på samma dator om man inte ser till att sessionerna förstörs.

<b>Nuläge:</b>
Det går fortfarande nå localhost:8080/messages även efter att ha loggat ut. Sessionen userId sätts till 0 och sessionen
role hanteras inte alls.

<b>Åtgärder:</b>
Sätt båda sessionerna till null vid utloggning.

<h5>Validering av utdata, [7]</h5>
Säg nu att någon elaking har lyckats få in skript på er server. Validerar man inte den data som puttas in i html-taggar
innan de renderas ut till en snäll användares browser kan ett sådant skript få tag i cookies, session tokens,
eller annan känslig data som finns i den snälle användarens browser. Detta kallas en XSS-attack, Cross Site Scripting.

<b>Nuläge:</b>
Ingen utdata valideras innan den renderas ut till användarens browser.

<b>Åtgärder:</b>
Använd en escape-metod som finns att tillgå i alla programmeringsspråk. Farliga tecken förvandlas till ofarliga strängar, [8].

<h5>Potentiella säkerhetsläckor, [9]</h5>
Filer med känslig information bör skyddas från åtkomst utifrån. Debugmeddelanden/konsollmeddelanden som kan avslöja
information om applikationens uppbyggnad bör aldrig följa med ut i deployad applikation. Felmeddelanden bör inte ge mer
information en ett för användaren hjälpsamt meddelande. Alla sådan här läckor är väldigt hjälpsam information för en elak
användare som försöker komma underfund med hur din applikation är uppbyggd och hur de ska hitta hål i säkerheten.

<b>Nuläge:</b>
Jag kan inte se att databasen, sessionsfilerna skyddas från åtkomst. Debugmeddelanden finns lite varstans i applikationen.
Filerna

<b>Åtgärder:</b>
Rensa bort debugmeddelanden innan deployment. Se över felmeddelande så att de inte bara skriver ut felet som automatgenereras.
Skydda vägen till databasen genom att använda config-filen, [9]-[10]. Ställ in vilka filer som inte ska vara möjliga att komma åt.
Använd .htaccess, [11].



<h3>Referenser</h3>

[1] "Top 10 2013-A1-Injection" OWASP, juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A1-Injection. [Hämtad: 11 december, 2015].

[2] "Sql injection prevention cheat sheet" OWASP, november 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet. [Hämtad: 11 december, 2015].

[3] "Query parameterization cheat sheet" OWASP, november 2014 [Online] Tillgänglig: https://www.owasp.org/index.php/Query_Parameterization_Cheat_Sheet. [Hämtad: 11 december, 2015].

[4] "Top 10 2013-A2-Broken Authentication and Session Management" OWASP, juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A2-Broken_Authentication_and_Session_Management [Hämtad: 11 december, 2015].

[5] "Password storage cheat sheet" OWASP, november 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet [Hämtad: 11 december, 2015].

[6] "Session management cheat sheet" OWASP, oktober 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/Session_Management_Cheat_Sheet [Hämtad: 11 december, 2015].

[7] "Top 10 2013-A3-Cross-Site Scripting (XSS)" OWASP, februari 2014 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A3-Cross-Site_Scripting_%28XSS%29 [Hämtad: 11 december, 2015].

[8] "XSS prevention cheat sheet" OWASP, december 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet [Hämtad: 11 december, 2015].

[9] "Top 10 2013-A5-Security Misconfiguration" OWASP, juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A5-Security_Misconfiguration [Hämtad: 11 december, 2015].

[8] "XSS prevention cheat sheet" OWASP, december 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet [Hämtad: 11 december, 2015].

[9] "Configuration" OWASP, maj 2009 [Online] Tillgänglig: https://www.owasp.org/index.php/Configuration [Hämtad: 11 december, 2015].

[10] "Error Handling" OWASP, juni 2009 [Online] Tillgänglig: https://www.owasp.org/index.php/Error_Handling [Hämtad: 11 december, 2015].

[11] Egen kunskap, vet inte var jag lärde mig det.

