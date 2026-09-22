# X)

## OWASP: OWASP Top 10:
-  Tämä ei toiminut minulla, sain 404 virheilmoituksen, kun koitin painaa linkkiä joka oli materiaaleissa

## Karvinen 2023: (https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/)
- URL-päätteitä voidaan löytää brute forcettamalla.
- Tällaisessa tiedustelussa kannattaa tarkistaa tulokset, sillä vastaan voi tulla paljon false positiveja (esim. HTTP-vastaus 200), vaikka kyseinen URL ei oikeasti sisältäisi mitään kiinnostavaa.
- Näitä voi etsiä käsin, mutta kannattaa käyttää työkalua ja listaa, jotka käyvät yleisimmät URL-päätteet automaattisesti läpi.

## PortSwigger: (https://portswigger.net/web-security/access-control)
- On olemassa vertikaalisia- ja horisonttaalisia käyttöoikeuskontrolleja (vertikaalisessa puhutaan esim. pääsyä sivuston ominaisuuksiin ja horisonttaalisessa puhutaan esim. käyttäjän pääsystä tietoihin/resursseihin).
- On myös kontekstiin riippuvia käyttöoikeuskontrolleja. Nämä estävät esim. että kun on tehnyt sivulla ostoksen, ei voi enään ostoksen jälkeen muokata ostosta.
- Yksi yleinen horisontaalisen käyttöoikeuden haavoittuvuus on IDOR, jossa käyttäjä voi muuttaa pyynnössä olevaa tunnistetta ja päästä käsiksi toisen käyttäjän resurssiin.
- Käyttöoikeudet kannattaa tarkistaa jokaisessa pyynnössä, eikä olettaa, että käyttäjä on päässyt tiettyyn vaiheeseen vain asianmukaista reittiä pitkin.
- Käyttöoikeuksia ei pitäisi toteuttaa pelkästään piilottamalla URL:ia tai toimintoja käyttöliittymästä, koska käyttäjä voi silti yrittää käyttää niitä suoraan.
