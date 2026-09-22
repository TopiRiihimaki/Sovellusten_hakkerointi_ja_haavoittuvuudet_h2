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

  # a) Break into 010-staff-only

  Ensin katsoin, mitä käy kun laitan 123 ja odotetusti sain tämän:
  <img width="1234" height="327" alt="image" src="https://github.com/user-attachments/assets/4ab211b6-dc0c-4a13-801e-624dc4486547" />


  Eli sain Somedude. Olin hiukan hukassa joten katsoin ensinmmäisen vihjeen, jossa kiinnitin näihin erityisesti huomiota:
  - You can bypass those with Firefox F12, Inspector. Use Ctrl-Shift-C element picker, then modify your local DOM Tree.

  Aloin miettiä, että "niin miltäs se näyttää tuolla HTML puolella?" ja menin katsomaan.

  Menin katsomaan inspectorillä ja löysin tuon kentän sieltä:
  
  <img width="436" height="70" alt="image" src="https://github.com/user-attachments/assets/914b248a-9b96-47c8-9c11-b35d552472b3" />

  Minun kokeiluissani, kun koitin suoraan laittaa jotain muuta kuin numeroita, niin kenttä valitti, että "Please enter a number". Ajattelin, että mitä jos tuon HTML kohdan jossa lukee:
  '''
  input type ="number"
  '''
  Niin jos muuttaisin sen vaikka "any".
  
  <img width="358" height="50" alt="image" src="https://github.com/user-attachments/assets/d39e7ecd-06dc-40d0-ab68-a549c4cc61f7" />


  Tein näin, jonka jälkeen laitoin:
  '''
  123'OR 1+1=2 --
  '''
  ja katsos kummaa, sainkin "foo"

<img width="682" height="241" alt="image" src="https://github.com/user-attachments/assets/20571e2d-7442-4a9b-a50c-9678c25e3149" />

Minun syötteeni voi jopa nähdä HTML:ssä

<img width="418" height="53" alt="image" src="https://github.com/user-attachments/assets/855b5b8c-3396-4087-923c-dd88ba5f90c7" />

(Tuossa se on kerennyt jo palaamaan taas input type = "number", mutta se ei meitä enään kiinnosta)

## Ennen b) osaa
Huomasin kun olin menossa tekemään tuota b) osaa, niin siellä oli vielä salasanoja jäljellä. 

Halusin selvittää, miten pääsisin noihin käsiksi ja niin, että minä en tietäisi miten koodin logiikka toimisi. 

Lähdin siis miettimään, että mahdollisesti tuo minun nykyinen SQl injektio hakee vain ensinmmäisen kohdan, koska se antaa aina True. 

Voisiko olla mahdollista, että jos sanon sille, että "älä ota ensinmmäistä kohtaa, vaan otakkin se toinen". Kokeilin tälläistä SQL komentoa seuraavaksi:
'''
123' OR 1+1=2 LIMIT 1 OFFSET 1 --
'''
Kun tuon laittaa, niin saa "Somedude" joten aattelin korvata tuon OFFSET 1:sen 2:lla ja sain tämmöisen:
<img width="967" height="157" alt="image" src="https://github.com/user-attachments/assets/97f494a7-76ad-43a1-af0e-fe07fad60ca3" />

Koska tämä haavottuvaisuus toimii, niin jos haluaisin nähdä mahdollisimman monta salasanaa, niin laittaisin aina uuden OFFSET numeron (tai realistisesti tekisin varmaan jonkin ohjelman joka tekisi niin).

## b)

### SQL-injektion korjaus

Sovelluksessa käyttäjän antama PIN liitettiin suoraan SQL-kyselyyn:

```python
sql = "SELECT password FROM pins WHERE pin='"+pin+"';"
```

Tämä mahdollisti SQL-injektion, koska käyttäjän syöte pystyi muuttamaan kyselyn rakennetta.

Korjasin ongelman muuttamalla kyselyä:

```python
sql = text("SELECT password FROM pins WHERE pin = :pin")
res = db.session.execute(sql, {"pin": pin})
```

Nyt käyttäjän syöte käsitellään datana eikä SQL-koodina, joten se ei voi muuttaa kyselyn rakennetta.

Testasin korjauksen uudelleen samalla injektiolla, jolla haavoittuvuutta aiemmin hyödynnettiin. Korjauksen jälkeen injektio ei enää toiminut.



  


