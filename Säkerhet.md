<h5>Rapport skriven av Johanna Szepanski js223kz</h5>

<h1>Säkerhetshål i applikationen https://github.com/thajo/messy_labbage</h1>
Nedan har jag listat de typer av attacker applikationen i nuläget är mottaglig för och åtgärder ni kan implementera
för att motverka respektive attack.

<h5>SQL injections, [1]</h5>
Attacker genom SQL injection utförs genom att någon med onda avsikter skjuter in skadlig kod i databaser, med syftet
att förstöra, komma åt konfidentiella uppgifter eller att i ett senare skede få ut den koden till snälla användares browsers.
Den skadliga koden skickas med anrop till en applikationen och hanteras på samma sätt som data från ett vanligt formulär eller
inputfält. Jag kan inte se att det sker någon validering av indata från användaren. T ex har jag kunnat skriva in "<script></script>",
i textfältet för meddelande och det har utan förändring lagrats i databasen.


<b>Åtgärder:</b>
Validera all indata. Strippa text från farliga tecken såsom < och >. Det finns escapemetoder i de flesta programmeringsspråk
som hanterar detta. Valideringen bör ske både på klient och server.

Använd prepared statements med variabelbindning, sk parameterized queries, i all kommunikation med databasen. I parameterized queries
definerar man först all Sql-kod, sen skickar man in parametrarna till frågan. Det betyder att databasen kan skilja mellan kod och
data oavsett vad användaren har skickat in. Prepared statements säkerställer att en attacker inte kan ändra avsikten med databasfrågan
och bör alltid användas, [2]-[3]. Det är redan gjort när meddelanden läggs till databasen så det är bara att kopiera det exemplet och
använda i login.js.

Databasen SQLite 3 som används i det här fallet stödjer inte lagrade procedurer annars skulle jag rekommenderat det.
Skriver lite om det ändå ifall ni skulle ändra valet av databas. Skillanden mellan lagrade procedurer
och prepared statements är att lagrade procedurer skapas och lagras i databasen. Som utvecklare kan man begränsa applikationens
rättigheter till att enbart få använda lagrade procedurer i kommunikationen mot databasen. I övrigt har lagrade procedurer samma funktion som
parametiserade frågor och man kan som utvecklare välja enbart ett av sätten. Jag skulle personligen implementerat båda då
det kanske inte alltid är möjligt för databasadministratören att säkerställa att applikationens kod implementeras korrekt.
Men som sagt inte ett alternativ i detta fall så kör på prepared statements [2]-[3].


<h5>Stöld av användaruppgifter [4]</h5>
Användaruppgifter sparas i klartext i databasen. Julafton för en elak användare som får tag på alla databasuppgifter genom
en SQL injection (se ovan). De behöver inte ens anstränga sig för att knäcka någon lösenordskryptering. Det är också stor
risk för att de ska lyckas med en bruce force attack. De låter ett helt automatiserat skript ligga och testa loginuppgifter
mot er applikation med hjälp av tabeller med 100000-tals kombinationer.

<b>Åtgärder:</b>
Lösenord ska aldrig sparas i klartext i en databas. Bästa lösningen är att salta + hasha lösenordet på servern innan
det sparas i databasen. När en användare sedan loggar in verifierar man det inskrivna lösenordet mot det saltade+hashade
som finns i databasen. Ett saltat+hashat lösenord kan aldrig återställas och om två olika användare har samma lösenord blir
saltning+hashning aldrig samma. Visst, man förhindrar inte att en attacker kan komma åt tabellen med uppgifter,
men man gör det i stort sett omöjligt för dem att knäcka lösenorden. Man bör också se till att varje
enskild användare har ett eget unikt salt. I kombination med åtgärderna mot SQL injection är databasen bra skyddad, [5].

Logga misslyckade inloggningsförsök för att blockera ip-adresser som är onormalt ofta förekommande, [5].


<h5>Session hijacking[4]</h5>
Tänk att du loggar in en applikation. Ett sessionsid sätts för att hålla koll på att du är inloggad när du navigerar
runt i applikationen och utför vissa operationer. Tänk att som elak användare få tag på detta sessionsid. Jag kan bli du.
Jag kan använda mig av det och vara inloggad i applikationen och utföra allt du kan göra. Detta kallas session hijacking, [6].

<b>Åtgärder:</b>
Använd SSL för att skicka all data krypterad mellan klient och server för att undvika att sessionen plockas upp genom sniffing.</br>
Sätt användarens ip-adress i en session och kontrollera att det är anrop från samma ip-adress som vill utföra kritiska aktioner.</br>
Vad jag kan se sätts det två sessioner. Båda med sessionsvärden som är lätta att gissa sig till, användarid och roll. Byt till
något mer oförutsägbart och mindre känsligt.</br>
Sätt sessionsflaggorna Httponly till true och secure till true, se filen config/express.js.</br>
Se till att förstör sessionerna helt vid utloggning. Som det är nu kan jag fortfarande komma åt localhost:8080/messages efter
utloggning. Tänk om jag använder någon annans dator och loggar ut, men egentligen inte är utloggad, [6].


<h5>XSS-attacker, [7]</h5>
Att skjuta in skadlig kod med avsikt att den ska exekveras i en intet ont anande användares browser, kallas en XSS-attack, Cross Site Scripting.
Säg att en elak användare lyckas skicka in en länk som vid otillräcklig validering av utdata renderas ut på sidan i den snälle användarens
browser. Vad händer om denna snälla användare klickar på den länken? Ja, det bestämmer den som lyckats med attacken. Stjäla uppgifter som sessionsid
är väldigt populärt.

<b>Åtgärder:</b>
Som det är nu kan jag skicka in skadlig kod som lagras i databasen, exemplet med "<script></script>", men den renderas inte ut i listan med meddelanden.
Det betyder att det finns någon form av validering av utdata. Om jag inte har helt fel för mig ligger valideringen som ett reguljärt uttryck i jquery.js.
Jag skulle dock ha gjort denna validering redan på servern för att det aldrig någonsin ska skickas skadlig kod från servern till
klienten. Använd inbyggda escapemetoder som finns i programmeringsspråket. Reguljära uttryck är knepiga att få till rätt och gör
iallafall mig lätt nervös, [8].


<h5>Potentiella säkerhetsläckor, [9]</h5>
Filer med känslig information bör skyddas från åtkomst utifrån. Debugmeddelanden/konsollmeddelanden som kan avslöja
information om applikationens uppbyggnad bör aldrig följa med ut i deployad applikation. Felmeddelanden bör inte ge mer
information en ett för användaren hjälpsamt meddelande. Alla sådan här läckor är väldigt hjälpsam information för en elak
användare som försöker komma underfund med hur din applikation är uppbyggd och hur de ska hitta hål i säkerheten.

<b>Åtgärder, [9]-[10]:</b>
Rensa bort debugmeddelanden innan deployment.</br>
Se över felmeddelande så att de inte bara skriver ut felet som automatgenereras.
Skydda vägen till databasen genom att använda config-filen. </br>
Ställ också in vilka filer som inte ska vara möjliga att komma åt genom att skydda känsliga mappar.</br>
Innehållet i filerna siteViews/javascript/Message.js och siteViews/javascript/MessageBoard.js borde kapslas in bättre och inte ge
ifrån sig så mycket information.</br>


<h5>Auktorisering av särskilda funktioner, [12]</h5>
För att säkerställa att obehöriga personer inte kan utföra vissa handlingar i en applikation måste användaren auktoriseras
och autentiseras på servern innan dessa funktioner kan köras. Det handlar helt enkelt om att begränsa tillgången till vissa url:er
beroende av nivå av behörighet i applikationen. Som det är nu kan jag komma åt och läsa alla meddelanden från databasen
genom att helt enkelt ange url:en localhost:8080/message/data. Eftersom jag kan komma åt och läsa data om alla meddelanden
kom jag snabbt fram till att jag kan radera ett meddelande genom att ange url:en localhost:8080/message/delete/?id=44
Detta kan jag göra utan att vara inloggad.

<b>Åtgärder:</b>
Kontrollera behörighet och autentisera användaren på servern före applikationen listar alla meddelanden eller raderar ett meddelande, [13].

<h5>Inför unika tokens vid alla POST-anrop, [14]</h5>
En elak användare kan skapa falska Http-anrop och tvinga en legitim, inloggad användare att utföra anropet, genom t ex en XSS-attack.
Attackern kan skapa POST-anrop, men inte få information genom GET-anrop, en sk Cross Site Request Forgery CSRF. Om attackern lyckas
innebär detta att hen kan skicka meddelanden och radera meddelanden i er applikation.
Jag ser att det finns en tanke på att skydda applikationen mot denna typ av attack när jag kollar i filen
siteViews/javascript/MessageBoard.js, men det är inte implementerat.

<b>Åtgärder:</b>
Synchronized Token Pattern. Lägg ett hidden field i alla formulär. Generera ett unikt, slumpmässigt token som värde till det fältet. Kontrollera på servern
att det existerar ett sådan token och att det är giltigt innan applikationen utför handlingen. Att bara kontrollera att användaren är
inloggad hjälper inte eftersom attackern automatiskt är inloggad via den legitime användaren vid den här typen av attacker. Det går också att använda CAPTCHA, men jag tycker det
är lite omständligt för användaren i den här typen av applikation. Kontrollera också origin header vid anropet, [15].


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

[12] "Top 10 2013-A7-Missing Function Level Access Control" OWASP, juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A7-Missing_Function_Level_Access_Control [Hämtad: 11 december, 2015].

[13] "Top 10 2007-Failure to Restrict URL Access" OWASP, april 2010 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2007-Failure_to_Restrict_URL_Access [Hämtad: 11 december, 2015].

[14] "Top 10 2013-A8-Cross-Site Request Forgery (CSRF)" OWASP, september 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A8-Cross-Site_Request_Forgery_%28CSRF%29 [Hämtad: 11 december, 2015].

[15] "Cross-site request forgery (CSRF) prevention cheat sheet" OWASP, december 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/Cross-Site_Request_Forgery_%28CSRF%29_Prevention_Cheat_Sheet [Hämtad: 11 december, 2015].



