# Guia: Unió d’un Zorin OS al domini Active Directory foodlogistic.test


**Objectiu** : connectar un Zorin al domini, que un usuari del domini puga iniciar sessió, tindre la seua carpeta personal i accedir a carpetes compartides.

---

## Captura – Creació de la Unitat Organitzativa (OU) “linux”

![Creació OU linux](/tascaUD9/imgUD9/c1.png)

**Què veiem?**  
Des del servidor Windows amb Active Directory, s’ha creat una OU anomenada **linux** dins del domini `foodlogistic.test`. La casella “Protegit contra eliminació accidental” està marcada.

**Per què?**  
Per tindre ordenats els equips i usuaris relacionats amb Linux. Així no barregem amb els objectes per defecte.

---

## Captura – Text estrany (possible error d’OCR)

![Text il·legible](/tascaUD9/imgUD9/c2.png)

**Què veiem?**  
Apareix un text repetitiu sobre FreeOCR. Segurament és una captura errònia o un placeholder. En la pràctica real, aquesta imatge no conté informació rellevant. Simplement la documentem per mantindre l’ordre.

**Conclusió:**  
No afecta el procediment.

---

## Captura – Llista de números llarga

![Números de l’1 al 1017](/tascaUD9/imgUD9/c3.png)

**Què veiem?**  
Una seqüència numèrica de l’1 fins a 1017. Possiblement una captura mal feta o d’una altra eina.

**Importància:**  
Cap. Simplement apareix a la teva llista d’imatges. La documentem per complir amb l’ordre.

---

## Captura – Creació d’un usuari al Active Directory

![Nou usuari al AD](/tascaUD9/imgUD9/c4.png)

**Què veiem?**  
Ventana per crear un usuari dins de `foodlogistic.test/linux`. La contrasenya està marcada com “Password never expires”. No s’obliga a canviar-la.

**Per què?**  
Per tindre un usuari de domini (per exemple `usuari@foodlogistic.test`) que després farà login al Zorin. Al marcar que la contrasenya no expira, evitem problemes de caducitat durant les proves.

---

## Captura – Vista de l’AD Users and Computers

![Estructura del AD](/tascaUD9/imgUD9/c5.png)

**Què veiem?**  
L’arbre del domini `foodlogistic.test` amb la carpeta `linux` creada. A dins hi haurà els usuaris i ordinadors Linux.

**Observació:**  
També apareixen les OU per defecte: `Builtin`, `Computers`, `Domain Controllers`, `Users`, etc.

---

## Captura – Configuració manual de la IP i DNS al Zorin

![Configuració IPv4 manual](/tascaUD9/imgUD9/c6.png)

**Què veiem?**  
S’està configurant la connexió cablejada amb IPv4 **Manual**. S’ha posat el DNS `10.0.2.4`, que és l’adreça del controlador de domini.

**Per què?**  
Perquè el client Linux puga resoldre el nom `foodlogistic.test` i trobar el servidor Kerberos. Sense un DNS correcte, no pots unir-te al domini.

---

## Captura – Instal·lació dels paquets necessaris

![Instal·lació sssd, realmd, adcli](/tascaUD9/imgUD9/c7.png)

**Què veiem?**  
S’executa `sudo apt install sssd-ad sssd-tools realmd adcli -y`. Tots eixos paquets són per a la integració amb Active Directory.

**Explicació:**  
- `realmd`: detecta i unix el domini.  
- `sssd`: autentica i guarda en caché els usuaris.  
- `adcli`: fa la unió tècnica al Directori Actiu.

---

## Captura – Text estrany `text[[0,0,997,996]]`

![Captura c8](/tascaUD9/imgUD9/c8.png)

**Què veiem?**  
Una línia de text que sembla metadades d’una imatge. No té significat pràctic. La incloem per respectar l’ordre.

---

## Captura – Data i hora correcta (vista del sistema)

![Data correcta](/tascaUD9/imgUD9/c9.png)

**Què veiem?**  
El rellotge del sistema mostra `3:48:04 PM Tuesday, May 5, 2026`. És una data coherent.

**Per què importa?**  
Kerberos necessita que l’hora del client estiga sincronitzada amb el DC (màxim 5 minuts de diferència). Abans d’unir-se al domini, hauràs d’ajustar l’hora si no és correcta.

---

## Captura – Comanda `date` amb data incorrecta abans de sincronitzar

![Data incorrecta](/tascaUD9/imgUD9/c10.png)

**Què veiem?**  
`mar 95 may 2926 15:46:26 CEST`. És una data impossible (95 de maig de 2926). Açò passava abans de corregir l’hora.

**Què has de fer?**  
Instal·lar `chrony` o `ntp` i sincronitzar amb el DC. Exemple:  
`sudo timedatectl set-ntp true` o bé `sudo ntpdate 10.0.2.4`.

---

## Captura – Descobrir el domini amb `realm discover`

![Realm discover](/tascaUD9/imgUD9/c11.png)

**Què veiem?**  
`realm discover foodlogistic.test` mostra la informació del domini: tipus Kerberos, programari Active Directory, i els paquets necessaris (ja instal·lats). Surt `configured: no` perquè encara no ens hem unit.

**Per què fer-ho?**  
Per comprovar que el client veu correctament el domini abans d’unir-se.

---

## Captura – Unió al domini amb `realm join`

![Realm join](/tascaUD9/imgUD9/c12.png)

**Què veiem?**  
`sudo realm join foodlogistic.test` i després demana la contrasenya de l’usuari `Administrator`. No apareix cap error.

**Resultat:**  
L’ordinador Zorin (amb nom `pc-linux-gnu`) queda registrat al Directori Actiu. El fitxer `/etc/sssd/sssd.conf` es configura automàticament.

---

## Captura – Propietats de l’ordinador al AD

![Propietats pc-linux-gnu](/tascaUD9/imgUD9/c13.png)

**Què veiem?**  
Des del servidor Windows, es veu l’objecte `pc-linux-gnu` dins de la carpeta `Computers` (o on s’haja creat). Es mostren les pestanyes Generals, Sistema operatiu, etc.

**Conclusió:**  
La unió ha sigut exitosa perquè el controlador de domini reconeix l’equip.

---

## Captura – Activar creació automàtica de la carpeta personal

![pam-auth-update mkhomedir](/tascaUD9/imgUD9/c14.png)

**Què veiem?**  
`sudo pam-auth-update --enable mkhomdeir` (hi ha una errada: hauria de ser `mkhomedir` però s’accepta). Activa el mòdul PAM que crea el directori `/home/usuari@domini` al primer inici de sessió.

**Per què?**  
Sense això, l’usuari del domini pot autenticar-se però no tindria on guardar els seus fitxers personals.

---

## Captura – Prompt de l’usuari del domini

![Prompt usuari@foodlogistic.test](/tascaUD9/imgUD9/c15.png)

**Què veiem?**  
El terminal mostra `usuari@foodlogistic.test@zorin:~$`. Açò indica que l’usuari actual pertany al domini `foodlogistic.test`.

---

## Captura – Comanda `id` per veure identificadors del domini

![Comanda id](/tascaUD9/imgUD9/c16.png)

**Què veiem?**  
`uid=825401113(usuari@foodlogistic.test) gid=825400513(domain users@foodlogistic.test)`. Els números alts són típics d’usuaris d’Active Directory.

**Comprovació:**  
L’usuari està autenticat pel domini i pertany al grup `domain users`.

---

## Captura – Editar fitxer sudoers per als administradors del domini

![nano /etc/sudoers.d/domainadmins](/tascaUD9/imgUD9/c17.png)

**Què veiem?**  
S’obre l’editor `nano` per crear un fitxer dins de `/etc/sudoers.d/` anomenat `domainadmins`.

**Per què?**  
Per donar permisos `sudo` a usuaris o grups del domini sense modificar el fitxer `sudoers` principal.

---

## Captura – Contingut del fitxer sudoers

![Regles sudo](/tascaUD9/imgUD9/c18.png)

**Què veiem?**  
Dues línies:  
1. `administrator@foodlogistic.test    ALL=(ALL)   ALL`  
2. `%linuxadmins@foodlogistic.test    ALL=(ALL)   ALL`

**Explicació:**  
La primera permet a l’usuari `administrator` fer qualsevol cosa amb `sudo`. La segona permet a **tots els membres** del grup `linuxadmins` del domini executar ordres com a root. El símbol `%` indica grup.

---

## Captura – Prova de `sudo su`

![sudo su correcte](/tascaUD9/imgUD9/c19.png)

**Què veiem?**  
L’usuari `usuari@foodlogistic.test` executa `sudo su`, li demana la seua contrasenya (la del domini) i després esdevé `root@zorin:/#`. El prompt canvia a `root`.

**Comprovació:**  
La regla sudo funciona. Si no tinguera permisos, donaria error.

---

## Captura – Instal·lar suport per a SMB (carpetes compartides)

![Instal·lació python3-smbc](/tascaUD9/imgUD9/c20.png)

**Què veiem?**  
`sudo apt update && sudo apt install python3-smbc`. Aquest paquet permet accedir a recursos SMB/CIFS (comparticions de Windows).

**Per què?**  
Per poder muntar o navegar a carpetes de grup compartides al servidor.

---

## Captura – Accés a un recurs SMB

![smb://110.0.2.41](/tascaUD9/imgUD9/c21.png)

**Què veiem?**  
Des del navegador d’arxius de Zorin s’accedeix a `smb://110.0.2.41`. Apareix una carpeta anomenada `public`.

**Observació:**  
Eixa IP és probablement la del servidor que té les carpetes compartides del domini (pot ser el mateix controlador de domini o un servidor de fitxers).

---

## Captura – Contingut de la carpeta compartida

![Carpeta public](/tascaUD9/imgUD9/c22.png)

**Què veiem?**  
Dins del recurs `public` es mostra l’estructura de carpetes típica d’un usuari (Descàrregues, Documents, etc.). Açò demostra que l’usuari del domini pot llegir i escriure (depèn dels permisos) en carpetes de grup.

