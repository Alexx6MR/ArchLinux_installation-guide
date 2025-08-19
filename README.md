# Gruppuppgift

I denna uppgift ska ni installera Arch Linux i en virtuell maskin. Utför alla steg manuellt med Linux-kommandon, utan grafisk guide.

## Syfte

- Få en djupare förståelse för Linux som skrivbordsmiljö.
- Få fördjupad kunskap om krypterade hårddiskar och partitioner i Linux.
- Få förståelse för Linux uppstartsprocess och hur parametrar kan läggas till Linux-kärnan.
- Få erfarenhet att på egen hand eller i grupp läsa och följa dokumentation samt lösa problem.

Uppgiften består av en obligatorisk grunddel och tre valfria bonusdelar som ger bonuspoäng.

Varje grupp väljer själv om, vilka och hur många av de valfria delarna som ska genomföras. Om alla valfria delar genomförs i tid ger de totalt 5 bonuspoäng till tentan och eventuella omtentor.

## Redovisning

Ert arbete ska resultera i två leveranser:

1. **Muntlig redovisning:** En färdig installation som visas för utbildaren. Alla i gruppen måste kunna förklara samtliga moment.

2. **Skriftlig dokumentation:** Ett dokument med detaljerade instruktioner för ert tillvägagångssätt.

### Skriftligt i `arch_installation.md`

- Använd filformatet [Markdown][md]. Visual Studio Code erbjuder ett användarvänligt stöd för Markdown, inklusive en praktisk förhandsgranskning.
- Alla använda kommandon ska dokumenteras och förklaras. Det går bra att gruppera relaterade kommandon och skriva en gemensam förklaring för dem när det är lämpligt.
- Använd [code blocks eller inline code][mdf] för alla kommandon och deras utmatning.
- Använd screenshots vid behov, exempelvis för grafiska gränssnitt som Xorg, Wayland, GDM, Gnome, i3, Sway och eventuellt UI för backupprogrammet. Placera alla screenshots i katalogen `img/` och länka in dem i rapporten så att de syns korrekt i förhandsgranskningen.

### Muntligt för utbildaren

Det är bra att dela upp arbetet, men vid redovisningen måste **alla** i gruppen förstå **alla** steg i detalj.

## Uppgiftsdelar

Gå igenom hela uppgiften tillsammans i gruppen och prata om hur ni tolkar varje moment. Se till att alla förstår vad som ska göras. Ta upp eventuella frågor med utbildaren innan ni börjar.

### Grunddel (Obligatorisk)

#### Virtuell maskin

- Två diskar på 30 GB vardera: en används som systemdisk och den andra för ren lagring.
- 8 GB RAM.
- Samma CPU-typ som värddatorn.
- Start i UEFI-läge.

#### Övergripande om installation

- Följ [Installation guide][aig] och installera manuellt med Linux-kommandon. Använd **inte** någon grafisk guide eller automatiserad installationsmetod.
- Systemet ska installeras som ett [UEFI][uefi]-system.

#### Systemdisk

- Använd GPT-partitionering.
- Skapa `/boot` och `/boot/efi` eller kombinera dem.
- Skapa en partition som är krypterad med dm-crypt/LUKS på resten av disken.
- Skapa en ext4-partition för `/` (root) som tar upp hela utrymmet i den krypterade behållaren.
- Använd swapfil, inte swappartition.

#### Uppstart med full hårddiskkryptering

- Använd GRUB som boot loader.
- Följ instruktioner för dm-crypt/LUKS i sidospåret "system encryption".
- Konfigurera systemet så att datorn frågar efter lösenord vid uppstart för att låsa upp den krypterade `/` (root)-partitionen.

##### Lagringsdisk

- Skapa en stor partition som täcker hela disken och kryptera den med **dm-crypt/LUKS**.
- Skapa sedan en ext4-partition som tar upp hela utrymmet i den krypterade behållaren.
- Skapa en nyckelfil `/storagekey` för att låsa upp partitionen. Nyckelfilen ska:
  - Ägas av root (`root:root`).
  - Ha rättigheterna `600` (endast root kan läsa och skriva).
- Konfigurera systemet så att den krypterade lagringsdisken låses upp automatiskt vid start med hjälp av en nyckelfil och monteras som `/storage`. För detta behöver både `/etc/crypttab` och `/etc/fstab` användas.

#### Efter grundinstallationen

1. Installera `sudo`, skapa en användare med sudo-rättigheter och inaktivera root-inloggning med lösenord.
2. Konfigurera nätverket till att använda DHCP.
   - **Tips: Använd systemd-networkd:**

      ```bash
      sudo nano /etc/systemd/network/20-wired.network
      ```

      Innehåll:

      ```ini
      [Match]
      Name=enp*

      [Network]
      DHCP=yes
      ```

      Aktivera tjänsterna:

      ```bash
      sudo systemctl enable --now systemd-networkd systemd-resolved
      sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
      ```

      *Förklaring:* `systemd-resolved` hanterar DNS-inställningar. Länken till `/etc/resolv.conf` måste skapas **efter** att tjänsten har startats för att DNS ska fungera.

3. Installera microcode för Intel/AMD (även om det inte påverkar i VM).

4. Installera `locate`.

#### Färdig desktop

Det sista steget är att installera en grafisk skrivbordsmiljö som **GNOME** – en användarvänlig och modern skrivbordsmiljö.
**GNOME** kan köras ovanpå **Xorg** eller **Wayland**. Båda är displaytekniker som hanterar grafiska fönster i Linux.

För inloggningsskärmen använder vi GNOME Display Manager (GDM). Den låter dig välja skrivbordsmiljö (t.ex. “GNOME” eller “GNOME on Xorg”)

##### Xorg eller Wayland

Undersök för- och nackdelar innan ni väljer mellan Xorg och Wayland och genomför relevant underrubrik.

##### Xorg

- Installera Xorg – en traditionell displayserver som länge varit standard i Linux.

- Installera GNOME.

- Installera GDM.

##### Wayland

(Ingen separat installation krävs för Wayland – stöd ingår i GNOME och GDM).

- Installera GNOME.

- Installera GDM.

### Bonusdelar (Valfria)

#### Lyxprompt (1 poäng)

- Installera kommandoskalet **Zsh** som ett alternativ till Bash och konfigurera det med **Oh My Zsh** med ett valfritt tema.

- Installera och konfigurera **Starship prompt** för Bash som standardprompt.

#### Upplev en tiling-fönsterhanterare: i3 eller Sway (2 poäng)

En sak som skiljer arbetsmiljön i Linux från andra operativsystem är att den är mycket modulär. Det gör det möjligt att använda kraftfulla, men mindre kommersiellt vanliga, upplägg.

Ett exempel är tiling-fönsterhanterare, som automatiskt placerar fönster och låter dig styra det mesta via tangentbordet.

##### Xorg eller Wayland

Ert tidigare val mellan Xorg och Wayland påverkar vilken underrubrik ni följer nedan. Behåll GNOME installerat så att det är valbart via GDM vid inloggning.

###### Xorg

- Installera fönsterhanteraren [i3][i3] och ersätt det enkla standardstatusfältet i3status med det externa programmet [Polybar][plyb].

###### Wayland

- Installera fönsterhanteraren [Sway][sway] och ersätt det enkla standardstatusfältet i3status med det externa programmet [Waybar][wayb].

###### Flexibilitet

- Ni kan även välja andra alternativ, om ni är beredda att arbeta ännu mer självständigt, så länge följande uppfylls:

- Installera en valfri tiling-fönsterhanterare och byt ut dess standardstatusfält mot ett externt program.

- Använd valfritt externt program för statusfältet tillsammans med er fönsterhanterare.

##### Redovisa kunskap i att använda er tiling-fönsterhanterare

Alla i gruppen måste som minst lära sig att endast använda tangentbordet för att utföra följande:

- Starta ett nytt terminalfönster.
- Flytta fokus från ett fönster till ett annat.
- Stäng fönstret i fokus.

#### Schemalagd backup mellan Arch och en annan dator  (2 poäng)

Skapa en periodiskt körande backup från uppgiftens dator till en annan dator via nätverket.

Backupen ska:

- Använda ett dedikerat backupprogram, inte enbart filkopiering (exempel på inte godkända verktyg: `rsync`, `rclone`, `scp`).

- Köras automatiskt vid bestämda intervall (t.ex. med systemd timer eller cron).

- Kunna återställa tidigare versioner av filer (versionering/snapshots).

- Vara möjligt att använda i en professionell miljö (stöd för kryptering, deduplicering eller fjärrlagring är plus).

Exempel på godkända verktyg: Restic, BorgBackup (gärna med Vorta), Duplicati, Déjà Dup.

Andra verktyg får användas om de uppfyller kriterierna.

##### Redovisa motivering och körning av er backuplösning

- Val av program och motivering.

- Hur backupen schemaläggs.

- Hur man gör en återställning från backupen.

## Tips

- Ta snapshot innan större steg – det är nästan alltid det enklaste sättet att backa.

- Håll din egen virtuella maskin uppdaterad så att du kan följa med i arbetet och testa själv.

- Dokumentera direkt och löpande under arbetets gång för att slippa återskapa steg i efterhand.

- Välj arbetssätt efter situationen: jobba vid en dator när det är bäst, dela upp när det är bäst.

- Om omstarten misslyckas och ingen GRUB-meny visas – och ni saknar snapshot: starta från Arch-installationsmediet, lås upp LUKS, montera och fortsätt via `arch-chroot` i stället för att börja om.

- Använd [Arch Wiki][aw] och den officiella dokumentationen för er tiling-fönsterhanterare och eventuell backuplösning.

## Filer att lämna in

| Fil                    | Beskrivning                                                     |
| ---------------------- | --------------------------------------------------------------- |
| `arch_installation.md` | Dokumentet ni skapar                                            |
| `img/*`                | Bildfiler i formaten PNG och JPEG som länkas in från dokumentet |

Använd endast små bokstäver i filnamn. Inga andra filer ska lämnas in och är ignorerade i `.gitignore`.

## Inlämning och redovisning

Läs dokumentet [Studentguide till GitHub Classroom][sgc] för instruktioner om hur ni lämnar in via Git.

När inlämningen är gjord i Git ska gruppen även göra en muntlig redovisning vid den tid som anges av utbildaren.

[aig]: https://wiki.archlinux.org/title/Installation_guide
[aw]: https://wiki.archlinux.org
[i3]: https://i3wm.org/
[md]: https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax
[mdf]: https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#quoting-code
[plyb]: https://github.com/polybar/polybar
[sgc]: https://github.com/nackc8/kursmaterial/blob/main/shared/studentguide-till-github-classroom.md
[sway]: https://swaywm.org/
[uefi]: https://en.wikipedia.org/wiki/UEFI
[wayb]: https://github.com/Alexays/Waybar
