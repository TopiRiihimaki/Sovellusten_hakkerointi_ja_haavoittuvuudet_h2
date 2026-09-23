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



  # c) Solve dirfuzt-1 

  Tein pitkälti, mitä materiaalin esimerkki tapauksessa (https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/) Eli asenssin ffuf, latasin sanakirjaston komennolla.
'''
wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/common.txt
'''
Josta sain tuon common.txt tiedoston:

<img width="692" height="68" alt="image" src="https://github.com/user-attachments/assets/2d4d8e20-3a8f-4a4d-bcb7-dedbb277ff25" />

  
Latasin tuon dirfuzt-1. Tiedosto ei tullut execute muodossa niin annoin tiedostolle ne:
  '''
  chmod +x dirfuzt-1
  '''
  Jonka jälkeen ajoin sen
  '''
  ./dirfuzt-1
  '''
  Se loi http-portaalin joka näytti tältä:
  <img width="616" height="139" alt="image" src="https://github.com/user-attachments/assets/fafca494-f5e9-43d1-a2af-f9a71bcf0b96" />

  ### ffuf

Ajoin ohjeiden mukaisesti tuon ffuf ohjelman sanakirjastolla komennolla:
'''
ffuf -w common.txt -u http://127.0.0.2:8000/FUZZ
'''
Josta sain suuren määrän tuloksia. Huomasin kuitenkin, että kaikissa oli sama size, joka oli 154

<img width="850" height="82" alt="image" src="https://github.com/user-attachments/assets/1f05ce11-1104-41e4-a404-e1d33a7c5f29" />

Päätin kokeilla, että jätetään kaikki joiden koko on 154 niin pois laittamalla komennon perään vielä -fs 154, josta sain:

<img width="861" height="106" alt="image" src="https://github.com/user-attachments/assets/e46c0d78-d057-46e0-91a3-22d51f9eaecd" />

Katoin mitä tuolla .git löyty ja löysin tämän:

<img width="478" height="183" alt="image" src="https://github.com/user-attachments/assets/1f811752-6208-4bf2-981d-32091918177c" />

# d) Break into 020-your-eyes-only

Katsoin ffuf avulla, että löytyykö joitakin URL päätteitä. Löysin /admin-console/. Menin sinne, mutta se vaatii sisäänkirjautumisen. Kokeilin hakea laajennettua ffuf hakua jossa oli 30k hakua, mutta mitään uutta ei tullut. Kokeilin sitten rekistöröityä ihan normi käyttäjänä ja sen jälkeen laittaa tuon /admin-console/ suoraan URL-kenttään. Pääsin sitten tänne:

<img width="586" height="232" alt="image" src="https://github.com/user-attachments/assets/f341db04-675f-4448-852c-64ea07ee5ec6" />

# e) Fix the 020-your-eyes-only vulnerability
Koodissa oli näin
'''
class AdminShowAllView(UserPassesTestMixin, TemplateView):
    template_name = "hats/admin-show-all.html"

    def test_func(self):
        return self.request.user.is_authenticated
'''
Tämä mahdollisti, että sivulle pääsi suoraan normikäyttäjällä.

Laittamalla tuon def test_func(self): loppuun vielä and self.request.user.is_staff, niin nyt sinne ei enään pääse, ilman että koodi katsoo onko käyttäjä osa henkilökuntaa. 

Nyt se näyttää tältä:
'''
class AdminShowAllView(UserPassesTestMixin, TemplateView):
    template_name = "hats/admin-show-all.html"

    def test_func(self):
        return self.request.user.is_authenticated and self.request.user.is_staff
'''





