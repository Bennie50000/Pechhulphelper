# Ons Café — Discord Bot

Discord bot met **moderatie**, **tickets** (dropdown + formulier per type), **verjaardagen**,
een **speeltijd-tracking**, **uitgebreide server logs** en **aanpasbare
berichten (embeds)**.

Gebouwd met [discord.js](https://discord.js.org/) v14 + TypeScript, gebruikt **slash commands**
en een lokale SQLite database (`bot.db`, via `better-sqlite3`) — geen externe database nodig.

## 📁 Structuur

```
discord-bot-ts/
├── package.json
├── tsconfig.json
├── .env.example
├── src/                  # TypeScript broncode
│   ├── bot.ts              # Startpunt: login, command-registratie, interactie-routering
│   ├── config.ts           # Laadt instellingen uit .env
│   ├── database.ts         # SQLite database + tabellen
│   ├── permissions.ts      # Staff-rol check (los van Discord's Administrator-permissie)
│   ├── types.ts            # Gedeelde types (BotModule, SlashCommand)
│   └── modules/
│       ├── general.ts        # /help /ping /serverinfo /userinfo /avatar
│       ├── moderation.ts     # /kick /ban /unban /mute /unmute /purge /setlogchannel
│       ├── tickets.ts         # /ticketsetup /addtickettype /ticketpanel + dropdown/formulier/knoppen
│       ├── birthdays.ts       # /setbirthday /birthdaylist + dagelijkse felicitaties
│       ├── playtime.ts         # Speeltijd per game, /playtime
│       ├── serverlogs.ts       # Berichten/leden/rollen/kanalen, /setserverlogchannel
│       ├── embedbuilder.ts     # /createembed + rol-gated "Bewerken"-knop
│       ├── staff.ts            # /addstaffrole /removestaffrole /liststaffroles
│       ├── quickreplies.ts     # /addbutton /buttonpanel — klik-knop stuurt kant-en-klaar bericht
│       ├── scheduling.ts        # /actiepaneel /plangesprek — gesprek plannen + inactiviteitsbericht
│       └── games.ts             # /telspel /galgje /raadgetal /dobbelsteen — minigames
└── dist/                 # Al-gecompileerde JavaScript-versie (kant-en-klaar, zie hieronder)
```

**Belangrijk:** deze map bevat ook een kant-en-klaar gecompileerde `dist/`-map. Dat is met
opzet zo — sommige hostingpanelen (zoals mcnode) hebben te weinig geheugen om TypeScript
zelf te compileren. Je hoeft dus **niet** zelf `npm run build` te draaien als je gewoon
online wilt gaan; alleen als je zelf iets in `src/` aanpast, moet je opnieuw bouwen.

## 🚀 Lokaal starten (VS Code / je eigen pc)

```bash
npm install
cp .env.example .env      # daarna DISCORD_TOKEN + CLIENT_ID invullen in .env
npm run build              # niet nodig als je de meegeleverde dist/ gebruikt
npm start
```

Tijdens ontwikkelen kun je ook direct zonder losse bouwstap draaien:
```bash
npm run dev
```

## 🎛️ Installatie op een paneel (mcnode, Pterodactyl e.d.)

Dit soort panelen draait meestal alleen `npm install` gevolgd door `node <bestand>` — ze
compileren TypeScript niet zelf, en hebben vaak weinig geheugen (bv. 128 MB), waardoor
`tsc` tijdens het compileren kan crashen ("Killed" / geheugenfout). Daarom:

1. **Verwijder eerst alles** wat er al in de hoofdmap van je server staat (om oude/verkeerd
   geplaatste bestanden kwijt te raken)
2. Upload dit hele project, **inclusief de meegeleverde `dist/`-map** (die staat er al
   voorgecompileerd bij, je hoeft niets zelf te bouwen)
3. Zorg dat de mapstructuur op de server er zo uitziet:
   ```
   /home/container/src/...
   /home/container/dist/bot.js       ← al kant-en-klaar
   /home/container/dist/...
   /home/container/package.json
   /home/container/tsconfig.json
   /home/container/.env               ← met je echte token
   ```
4. Zet in de **Startup**-instellingen het veld voor het startbestand (heet vaak **"Bot js
   file"**) op exact:
   ```
   dist/bot.js
   ```
   (niet `/dist/bot.js`, niet `home/container/dist/bot.js` — alleen `dist/bot.js`, de server
   plakt de rest van het pad er zelf voor)
5. **Reinstall Server**, daarna **Start**

### Bot aanmaken op Discord

1. Ga naar https://discord.com/developers/applications
2. Klik **New Application**, geef een naam
3. Kopieer op de **General Information** pagina de **Application ID** (dit is je `CLIENT_ID`)
4. Ga naar **Bot** → **Reset Token** → kopieer de token (dit is je `DISCORD_TOKEN`)
5. Zet onder **Privileged Gateway Intents** de volgende AAN:
   - `SERVER MEMBERS INTENT`
   - `MESSAGE CONTENT INTENT`
   - `PRESENCE INTENT` (nodig voor `/playtime`)
6. Ga naar **OAuth2 → URL Generator**, selecteer scope `bot` en `applications.commands`,
   en bij permissions minimaal: `Manage Roles`, `Manage Channels`, `Kick Members`,
   `Ban Members`, `Moderate Members`, `Manage Messages`, `Send Messages`, `Read Message History`,
   `View Audit Log` (nodig om bij server logs te tonen wíé een actie heeft uitgevoerd).
7. Open de gegenereerde URL en nodig de bot uit op je server.

## 🔧 Eerste configuratie in je server

**Belangrijkste eerste stap:** configuratiecommands (`/setlogchannel`, `/ticketsetup`, enz.)
vereisen geen Discord "Administrator"-permissie, maar een eigen **staff-rol systeem**. Twee
manieren om dat in te stellen:

**Optie A — rechtstreeks in `.env` (geen Discord Administrator nodig):**
```
STAFF_ROLE_IDS=123456789012345678
```
Komma-gescheiden voor meerdere rollen. Na een wijziging moet je de bot herstarten.

**Optie B — via Discord zelf (met een account dat wél Administrator heeft):**
```
/addstaffrole role:@JouwStaffRol
```
(of `role_id:<ID>` als je de rol liever via een ruw ID wilt opgeven)

| Command | Wat het doet |
|---|---|
| `/setlogchannel #kanaal` | Moderatielogs (kicks/bans/mutes) |
| `/setserverlogchannel #kanaal` | Uitgebreide server logs — bijna alles wat Discord's eigen Audit Log ook toont (zie hieronder) |
| `/ticketsetup` | Logkanaal (+ optioneel archiefcategorie) voor tickets |
| `/addtickettype` | Ticket-type toevoegen aan de dropdown (herhalen per type) |
| `/ticketpanel` | Ticket-paneel (dropdown) plaatsen |
| `/setbirthdaychannel #kanaal` | Kanaal voor automatische verjaardagsfelicitaties |
| `/setembededitrole` | Rol die met de "Bewerken"-knop berichten mag aanpassen |

## 📋 Uitgebreide server logs (net als Discord's Audit Log)

`/setserverlogchannel` dekt zo goed als alles wat Discord's eigen Audit Log ook bijhoudt:

- **Berichten**: bewerkt, verwijderd, bulk-verwijderd
- **Leden**: binnenkomen, vertrekken/gekickt, verbannen, ban opgeheven, rollen gewijzigd,
  bijnaam gewijzigd, timeout gegeven/opgeheven
- **Kanalen**: aangemaakt, verwijderd, naam/topic aangepast
- **Rollen**: aangemaakt, verwijderd, naam/kleur/permissies aangepast
- **Emoji's**: toegevoegd, verwijderd
- **Uitnodigingen**: aangemaakt, verwijderd/verlopen
- **Webhooks**: gewijzigd
- **Threads**: aangemaakt, verwijderd
- **Voicekanalen**: gejoined, verlaten, gewisseld
- **Serverinstellingen**: naam/icoon aangepast
- **AutoMod**: regel aangemaakt/verwijderd, actie uitgevoerd

Waar mogelijk toont de bot ook **wie** de actie heeft uitgevoerd (bv. wie een kanaal
verwijderde) — dat haalt de bot op uit Discord's eigen audit-logboek, dus geef de bot bij
het uitnodigen ook de permissie **"Audit-logboek weergeven"** (`View Audit Log`), anders
staat er "Onbekend" in plaats van een naam.

## 🎫 Tickets: archiveren + transcript

Bij `/ticketsetup` kun je optioneel een `archive_category` meegeven. Als je dat doet:
- Bij het sluiten van een ticket (knop of `/closeticket`) wordt een **transcript** (`.txt`
  bestand met alle berichten) automatisch meegestuurd naar het logkanaal.
- Het ticketkanaal wordt daarna **niet verwijderd**, maar verplaatst naar de archiefcategorie,
  met alle toegang ingetrokken behalve voor staff.
- In het gearchiveerde kanaal verschijnt een **"Verwijder ticket"**-knop (alleen voor staff).

Geen `archive_category` ingesteld? Dan wordt het kanaal na sluiten na 5 seconden verwijderd
(transcript wordt nog steeds gelogd).

## ✏️ Aanpasbare berichten (embed-editor)

`/createembed` plaatst een bericht met titel, beschrijving, kleur, afbeelding en footer,
met een **"Bewerken"**-knop eronder. Kleur geef je op als hex-code, bv. `color:#FF5733` of
gewoon `color:FF5733` (zonder `/` en `#` mag ook). Zonder opgegeven kleur gebruikt de bot
het standaard Discord-blauw.

Alleen leden met de rol die je instelt via `/setembededitrole` (of staff/Administrator)
kunnen op de "Bewerken"-knop klikken — dat opent een formulier waarmee ze titel,
beschrijving, afbeelding-URL, footertekst en footer-afbeelding-URL kunnen aanpassen,
waarna het bericht direct wordt bijgewerkt. **De kleur zelf wijzig je niet via dat
formulier** (Discord staat maximaal 5 velden per formulier toe, en die zitten al vol) —
de kleur blijft dus staan zoals bij het aanmaken, tenzij je het bericht verwijdert en met
`/createembed` opnieuw plaatst met een andere kleur.

## 🎮 Minigames

Vier minigames, elk maximaal 1 actief potje per kanaal tegelijk. **Telspel, galgje en
raadgetal draaien altijd in één vast, van tevoren ingesteld kanaal** — ongeacht in welk
kanaal je het start-command typt, het spel begint altijd daar:

```
/setspelkanaal spel:Telspel kanaal:#telspel
/setspelkanaal spel:Galgje kanaal:#galgje
/setspelkanaal spel:Raad het getal kanaal:#raad-het-getal
```
(1x instellen per spel, staff-only. Daarna hoeft dat nooit meer.)

**🔢 Telspel** (`/telspel start` / `/telspel stop`, staff-only) — leden tellen om beurten
omhoog in het ingestelde kanaal (1, 2, 3, ...). Dezelfde persoon mag niet 2x achter elkaar
tellen, en bij een fout getal begint de telling weer bij 1. De bot houdt het hoogste
record bij. Optioneel krijgt degene die het verpest automatisch een "straf"-rol:
```
KAN_NIET_TELLEN_ROLE_ID=de_rol_id
```
(De bot-rol moet in Serverinstellingen → Rollen boven die rol staan, anders wordt het
stilletjes overgeslagen en gaat het spel gewoon door.)

**🎪 Galgje** (`/galgje start [woord]` / `/galgje stop`, **iedereen mag dit gebruiken**) —
klassiek hangman in het ingestelde kanaal. Zonder eigen `woord` kiest de bot een
willekeurig Nederlands woord. Typ gewoon een losse letter in het kanaal om te raden; na 6
foute letters is het potje verloren.

**🔮 Getal raden** (`/raadgetal start [min] [max]` / `/raadgetal stop`, **iedereen mag dit
gebruiken**) — de bot denkt aan een willekeurig getal tussen `min` en `max` (standaard
1-100) in het ingestelde kanaal, typ je gok en de bot reageert met 📈 (hoger) of 📉 (lager)
tot iemand het goed heeft.

**🎲 Dobbelsteen** (`/dobbelsteen [zijden] [aantal]`, **iedereen mag dit gebruiken**) —
gooi direct met 1 of meerdere dobbelstenen (standaard 1x 6-zijdig), overal bruikbaar, geen
kanaal-instelling nodig.

**💬 Quote** (`/quote`, iedereen) — toont een willekeurige inspiratie-/motivatiequote uit
een ingebouwde lijst van ~25 bekende quotes (Engels + een paar Nederlandse). Geen internet
of externe API nodig, werkt altijd offline/lokaal.

**🪙 Kop of munt** (`/kopofmunt`, iedereen) — simpele muntworp, 50/50 kans op kop of munt.

## 📅 Gesprekken plannen + inactiviteitsbericht (`/actiepaneel`)

`/actiepaneel` plaatst één paneel met meerdere actie-knoppen naast elkaar:

**📅 Plan gesprek** — bij een klik:
1. Je krijgt één keuzemenu (alleen voor jou zichtbaar): 1-3 begeleiders kiezen. De
   kandidaat wordt **niet** los gekozen — de bot tagt automatisch de hele
   sollicitanten-rol, ingesteld via `.env`:
   ```
   SOLLICITANT_ROLE_ID=de_rol_id
   ```
2. Klik op **"Volgende: dag/tijd invullen →"**
3. Er verschijnt een formulier voor dag, tijd, type sessie en verwachte duur
4. Na verzenden post de bot automatisch het complete uitnodigingsbericht, mét
   **✅**/**❌** reacties erbij voor de RSVP — **altijd in hetzelfde vaste kanaal als de
   Basistraining reminder** (`BASISTRAINING_CHANNEL_ID`), ongeacht in welk kanaal je op de
   knop drukt of het command typt. Er is dus geen apart kanaal voor "Plan gesprek" nodig —
   die gebruikt gewoon dezelfde instelling als de reminder-knop hieronder. Onderaan het
   bericht wordt automatisch het wachtkamer-voicekanaal gelinkt, in te stellen via `.env`:
   ```
   WACHTKAMER_CHANNEL_ID=het_voicekanaal_id
   ```

**🚫 Afgelast** — opent een formuliertje met 2 velden (tijd + reden, allebei vooringevuld
met een voorbeeld). Na verzenden stuurt de bot een nette afgelast-melding naar hetzelfde
kanaal, met de sollicitanten-rol getagd, een link naar het mededelingen-kanaal, en
onderaan **jouw eigen naam** als afzender (wie op de knop drukte). Instellen via `.env`:
```
MEDEDELINGEN_CHANNEL_ID=het_kanaal_id
```

**⚠️ Inactiviteit** — stuurt direct (geen tussenstappen) dit vaste bericht:
```
<rol> Je hebt je uren deze week niet gehaald.
Dat betekent dat je deze week extra hard bezig moet gaan met uren halen.
⚠️Anders ben je ontslagen!⚠️
```
Dit gaat **altijd** naar hetzelfde kanaal en tagt **altijd** dezelfde rol, ongeacht in welk
kanaal je op de knop drukt — dat stel je in via `.env`:
```
INACTIVITY_CHANNEL_ID=het_kanaal_id
INACTIVITY_ROLE_ID=de_rol_id
```
Zonder deze twee ingevuld geeft de knop een duidelijke foutmelding in plaats van te crashen.

**🎓 Basistraining reminder** — opent eerst een kort formuliertje met 1 veld ("Over hoeveel
minuten beginnen de gesprekken?", standaard ingevuld met "10"). Na verzenden stuurt de bot:
```
<rol> **REMINDER**
De gesprekken beginnen over **X minuten**!
Join alvast: <link naar het kanaal> zodat we zo direct kunnen starten.
❗ **Belangrijk:**
Je dient tijdens het gesprek ook aanwezig te zijn **in-game op het Pechhulp HB**.
```
Rol, kanaal én de link in het bericht blijven **altijd hetzelfde** — alleen de minuten vul
je zelf in bij elke keer klikken. Instellen via `.env`:
```
BASISTRAINING_ROLE_ID=de_rol_id
BASISTRAINING_CHANNEL_ID=het_kanaal_id
```

Losse `/plangesprek` (alles in één command, zonder knoppen/paneel) staat er ook nog steeds
naast, voor als je snel iets wilt plannen zonder door de stappen te klikken. Ook hier wordt
automatisch de `SOLLICITANT_ROLE_ID`-rol getagd, en het bericht gaat ook hier altijd naar
`BASISTRAINING_CHANNEL_ID`, ongeacht in welk kanaal je het command typt:
```
/plangesprek dag:Vrijdag tijd:19:00 begeleider1:@Bennie begeleider2:@Bjorn
```
Optioneel aan te passen: `type` (standaard "Gesprek / Basistraining") en `duur` (standaard
"ongeveer 1 uur"), en tot 3 begeleiders.

## 🔘 Snelle-antwoord-knoppen (voor als je lui bent 😉)

Zet een paneel met knoppen in een kanaal — bij een klik stuurt de bot direct een
kant-en-klaar bericht in dat kanaal. Handig voor veelgestelde vragen, regels, of
standaardantwoorden waar je zelf niet elke keer opnieuw voor wilt typen.

```
/addbutton label:"Regels" message:"Lees de regels in #regels voordat je begint!" emoji:📜
/addbutton label:"Skins" message:"De skinlijst staat in #outfits." emoji:👕 style:Groen
/buttonpanel title:"Veelgestelde vragen"
```

- `/addbutton` — voeg een knop toe (label + het bericht dat verstuurd wordt bij een klik,
  optioneel een emoji en kleur)
- `/removebutton` — een knop verwijderen
- `/listbuttons` — overzicht van alle ingestelde knoppen
- `/buttonpanel [title]` — plaatst het paneel (alle knoppen tot nu toe) in het huidige kanaal

Na het toevoegen/verwijderen van knoppen moet je `/buttonpanel` opnieuw plaatsen zodat het
paneel klopt. Maximaal 25 knoppen per paneel (Discord's limiet: 5 knoppen per rij, 5 rijen).

## 🎮 Speeltijd (`/playtime`)

De bot houdt via Discord-activiteiten (rich presence) bij hoe lang leden per game spelen,
en `/playtime [lid]` toont de top 3 als kaartjes. Vereist `PRESENCE INTENT` (zie hierboven).

## 👑 Owner-only: (bijna) alle permissies aan een rol geven

`/grantadmin <rol>` zet **alle permissies aan, behalve "Server Beheren"** (Manage Server).
Let op: de losse "Administrator"-vlag wordt hierbij bewust **niet** aangezet, want die
overrulet alle andere checks en zou "Server Beheren" alsnog meegeven. Je krijgt dus een
rol met vrijwel alle rechten losjes aangevinkt (kicken, bannen, kanalen/rollen beheren,
berichten beheren, enz.), maar zonder Server Beheren. Dit command is met opzet
**alleen bruikbaar door jou** — iedereen anders die het probeert krijgt een generieke
"geen rechten"-melding te zien (het command zelf blijft wel zichtbaar in de `/`-lijst van
Discord, dat kan een bot technisch niet per persoon verbergen; wél kun je in
**Serverinstellingen → Apps → [botnaam] → grantadmin** zelf een rol/persoon instellen die
het command mag zien, als je dat nóg strikter wil).

Instellen:
```
OWNER_ID=jouw_discord_user_id
```
Je eigen Discord user-ID vind je via **Instellingen → Geavanceerd → Ontwikkelaarsmodus**
aanzetten, dan rechtermuisklik op je eigen naam → **Copy User ID**.

**Let op:** de rol van de bot zelf moet in **Serverinstellingen → Rollen** boven de rol
staan die je Administrator-rechten wil geven — dat is een harde Discord-beperking (een bot
kan nooit rechten geven aan een rol die hoger staat dan zijn eigen rol). Het command geeft
een duidelijke foutmelding als dat niet het geval is.

## ⚠️ Let op

- **Deel je `.env` bestand nooit** en zet het niet op GitHub.
- `better-sqlite3` gebruikt meestal een prebuilt binary bij `npm install` (geen zware
  compilatie nodig), maar `tsc` (TypeScript compileren) kan op hostingpanelen met weinig
  RAM wél problemen geven — vandaar dat deze map de gecompileerde `dist/` al meelevert.
- Voor grotere servers kun je later overstappen op PostgreSQL, maar SQLite is ruim
  voldoende voor de meeste servers.
