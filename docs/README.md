# Töölogi

**Meeskonnaliikmed ja rollid:** Christina Vahi

### Projekti seadistus
##### Valitud serverivariant:
- Alustasin variant B-st. Render.com 
- Proovisin ka varianti C. Zone VPS (Ubuntu 22.04) 

##### Põhjendus: 
- **Variant B:** See tundus alguses kõige lihtsam viis saada toimiv CI/CD pipeline ilma SSH seadistuseta.
- **Variant C:** Tahtsin päris serveri kogemust.
  
##### Serveri info:
- **Variant B:** Render.com (Static Site), automaatne HTTPS.
- **Variant C:**
  - **IP-aadress:** 185.31.243.1
  - **Veebiserver:** Nginx
  - **Domeen:** pixelchaos.me
  
##### SSH ja GitHub Secrets staatus:
- **SSH võtmepaar:** Loodud ed25519 võtmepaar. Avalik võti lisatud serveri deploy kasutaja authorized_keys faili. Parooliga sisselogimine on turvalisuse huvides piiratud.
- **GitHub Secrets:** Seadistatud Repository Secrets:
  - SSH_HOST: Serveri IP
  - SSH_USERNAME: deploy
  - SSH_PRIVATE_KEY: privaatne võti turvaliseks ühenduseks.
   
### Tehtud tegevused
1. Loodud projekti struktuur kaustadega `public/` ja `docs/`.
##### Variant B:
2. Ühendatud GitHubi repo Render.com teenusega.
3. Seadistatud automaatne deploy: iga "push" main harusse uuendab lehte.

##### Variant C:
2. Lõin Zone-s uue VPSi.
3. Genereerisin ed25519 võtmepaari.
4. Lõin spetsiaalse kasutaja nimega deploy.
5. Avalik võti lisatud serveri deploy kasutaja authorized_keys faili.
6. Lisasin deploy kasutaja sudo gruppi ja andsin talle täieliku kontrolli kausta /var/www/pixelchaos üle (chown käsk), et ta saaks sinna faile kopeerida.
7. Paigaldasin ja seadistasin Nginxi. Lõin konfiguratsioonifaili, mis ütleb serverile: "Kui keegi küsib pixelchaos.me, siis näita talle faile, mis asuvad kaustas /var/www/pixelchaos/public." Lülitasin konfiguratsiooni sisse ja testisin, et Nginx ei teeks vigu.
8. Kustutasin Namecheap'is vanad GitHub Pages ja Render IP-aadressid, mis suunasid domeeni valesse kohta.
9. Lisasin uued A-kirjed, mis viitavad uuele Zone serveri IP-le.
10. Salvestasin IP ja privaatse võtme GitHub Secretsi alla.
11. Lisasin .github/workflows/deploy.yml faili, mis käivitub iga kord, kui teha git push. See kopeerib automaatselt värske koodi GitHubist otse serverisse.
12. Tegin testmuudatuse kohalikus index.html failis ja see jõudis edukalt serverisse.
13. Kasutasin Certbot, et saada Let's Encrypti poolt tasuta ja ametlik SSL-sertifikaat.

### Probleemid ja lahendused
- Renderi puhul esimesel katsel git push main harusse ei uuendanud lehte. Tegin Manual Deploy. Teisel katsel kõik juba toimis.
- GitHubist ei suutnud ühendada Zone-i serveriga, kuna ei olnud lisanud terviklikku privaatvõtit GitHub Secretsisse. Sain teada, et tuleb lisada kõik, kaasa arvatud see osa, kus on -----BEGIN OPENSSH PRIVATE KEY----- ja -----END OPENSSH PRIVATE KEY----- 
- DNS-i viivitus: domeen näitas vana IP-aadressi, mistõttu Certbot ebaõnnestus. Olin jätnud mingi A-kirje kustutamata. Kasutasin dig käsku DNS-i leviku jälgimiseks ja ka Google DNS Flush teenust.

### Deploy testimise tulemus:
##### Variant B:
* Veebileht on edukalt üleval Renderi pakutud domeenil https://pixelchaos.onrender.com/
#### Variant C:
* Veebileht on edukalt üleval domeenil https://pixelchaos.me
* GitHub Actions (workflow) on edukalt seadistatud. Iga git push origin main käivitab automaatse deploimise.
* Testmuudatus kohalikus index.html failis jõudis pärast push-i serverisse vähem kui 20 sekundiga.
* Turvalisus: Veebileht on kättesaadav üle HTTPS-i ja suunab automaatselt ümber turvalisele ühendusele.