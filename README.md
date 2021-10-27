# docker-image-kakuttaja

Tässä repossa on travis-build joka kakuttaa muiden OPH:n buildien tarvitsemia yleisiä docker-imageita AWS ECR:ään.
Tällä tavalla eri buildeissä voidaan välttää Docker Hub:in pull rate limit -ongelmat joita voi tulla jos imageita haetaan
suoraan Docker Hubista. Jos kakuttaja törmää pull rate limittiin, niin se ei yleensä haittaa, koska melko tuore image on 
kuitenkin siitä huolimatta kakutettuna edellisestä onnistuneesta kakuttaja-buildistä.

Kakutettavat imaget ovat sellaisia mitä tyypillisesti tarvitaan eri palveluiden buildeissä testien suorittamiseen,
alkuvaiheessa näitä ovat postgres ja redis. Molemmista haetaan ECR:ään sellaiset versiot, jotka vastaavat
pilviympäristöissä käytettäviä versioita vastaavista komponenteista.

Postgres-imageita myös kustomoidaan kakutuksen yhteydessä sen verran, että niihin lisätään `fi_FI`-locale. Tämä korjaa 
virheen joka tuli geneerisellä postgres-imagella vastaan atarun migraatioissa:

    ERROR: collation "fi_FI" for encoding "UTF8" does not exist

## Kakutettujen imageiden käyttö

Tässä tuotettujen imageiden hakeminen ECR:stä vaatii saman autentikaation kuin mm. baseimagen hakeminen, 
`ci-tools`-gitrepon työkaluilla. 

Postgres-imagen tunniste on esim. versiolle 12:

    190073735177.dkr.ecr.eu-west-1.amazonaws.com/utility/postgres:12

ja vastaavasti löytyy tällä hetkellä myös major-versio 11. Postgresista on tarkoitus pitää jatkossa tarjolla kaikki 
tarvittavat major-versiot ja minor-versiot päivittyvät automaattisesti niin että esim. tagillä `12` saa aina melko
tuoreen minor-version.

Redis 5.0 -image löytyy tunnisteella:

    190073735177.dkr.ecr.eu-west-1.amazonaws.com/utility/redis:5.0

Tässä vaiheessa versio 5.0 vastaa pilviympäristöjemme redis-komponentteja ja jatkossa voidaan kakuttaa muitakin 
tarvittavia versioita.

Muita imageita lisätään tänne vähitellen ja kaikkea ei ehkä dokumentoida README:hen. Ks. Dockerfile:ssa tarkempi listaus.

## Kakkujen päivitys

Postgres- ja redis-imageiden hakeminen ECR-repoihin tapahtuu tämän git-repon travis-buildissä, joka ajetaan 
automaattisesti aina viikon aluksi maanantaiaamuna. ECR:ään päivittyy tällöin uusin saatavilla oleva esim. 
postgres-image tagillä `12` jne. Ja hakemalla postgres-imagen tagillä `12` ECR:stä saa siis normaalisti korkeintaan
viikon verran kakutusviiveen takia vanhentuneen imagen.

Travis-buildin voi myös triggeröidä manuaalisesti muinakin aikoina kakkujen päivittämiseksi.
