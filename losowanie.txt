import psycopg2
import names
from random import randint, sample, choice
import string

# Dodaj prawdziwe pesele
# losowanie danych
kody = []
rand_numers = []
adresy =      []
miasta =      ['BIAŁYSTOK','BIELSKO-BIALA','GDYNIA','LEGNICA','BYDGOSZCZ','LUBLIN','CZĘSTOCHOWA','CHORZÓW','TARNÓW',
               'ZIELONA GÓRA','GRUDZIĄDZ','PŁOCK','JASTRZĘBIE - ZDRÓJ','JAWORZNO','KIELCE','KOSZALIN','ZABRZE','SUWAŁKI',
               'RYBNIK','WAŁBRZYCH','NOWY SĄCZ','KALISZ','SIEDLCE','KONIN','RUDA ŚLĄSKA','DĄBROWA GÓRNICZA','ELBLĄG',
               'SOSNOWIEC','RADOM','PIOTRKÓW TRYB.','WŁOCŁAWEK','MYSŁOWICE','BYTOM']
typ_b =       ["karabin","strzelba","pistolet","rewolwer","reczyn karabin maszynowy","pneumatyczna","snajperka"]
klasy =       ["I", "II", "III", "IV", "M"]
producenci_ammo = [names.get_last_name()+"company" for i in range(10)]
producenci_broni = [names.get_last_name()+"company" for j in range(10)]
pochodzenie = ['USA',"Niemcy","Rosja","Francja","Polska","Chiny"]
kaliber = [(40+i*5)/10 for i in range(6)]
stan = ["dostepny","niedostepny"]

def rand_numer(n):
    r = int()
    while True:
        range_start = 10**(n-1)
        range_end = (10**n)-1
        r = randint(range_start, range_end)
        if r not in rand_numers: break
    rand_numers.append(r)
    return r
def kod(l,d):
    while True:
        t = str()
        for i in sample(string.ascii_letters,l):
            t += i.upper()
        t += "-"+"{}".format(rand_numer(d))
        if t not in kody: return t

def adres():
    a = str()
    while True:
        a = names.get_last_name() + "a {}".format(rand_numer(2))
        if a not in adresy: break
    adresy.append(a)
    return a

def data_ammo():
    m = str(randint(1,12))
    d = str(randint(1,27))
    if len(m) == 1:
        m = "0" + m
    if len(d) == 1:
        d = "0" + d
    return str(randint(2010,2020))+"-"+m+"-"+d


#Polaczenie sie z baza danych
conn = psycopg2.connect(database="strzelnica", user="postgres", password="emil123",port=5432)

curr = conn.cursor()

# KIEROWNICY---Nie zalezny
for i in range(7):
    curr.execute("INSERT INTO Kierownicy (Imie, Nazwisko ,Pesel ,Nr_Telefonu) VALUES ('{}','{}','{}','{}')"
             .format(names.get_first_name(),names.get_last_name(),rand_numer(11),rand_numer(9)))
conn.commit()

#MAGAZYN-- zalezny od Kirownikow
curr.execute("SELECT Id_Kierownika FROM Kierownicy;")
id_kier = curr.fetchall()
for i in range(7):
    curr.execute("INSERT INTO Magazyn (Id_Kierownika, Miasto, Adres, Pojemnosc_Ammo, Pojemnosc_Broni)"
                 " VALUES ({},'{}','{}',{},{})"
                 .format(id_kier[i][0], choice(miasta), adres(), randint(1000,10000),randint(50,500)))
conn.commit()
del id_kier

#Strzelnica -- Zalezny od Magazyn
curr.execute("SELECT Id_Magazynu, Miasto FROM Magazyn;")
mag = curr.fetchall()
for i in range(10):
    curr.execute("INSERT INTO Strzelnica (Id_Magazynu, Miasto, Adres)"
                 "VALUES ({},'{}','{}')"
                 .format(mag[i%7][0],mag[i%7][1] , adres()))
conn.commit()# Różne miasta
del mag

# Trenerzy -- Zalezny od Strzelnica
curr.execute("SELECT Id_Strzelnicy FROM Strzelnica;")
strz = curr.fetchall()
for i in range(22):
    curr.execute("INSERT INTO Trenerzy "
                 "VALUES ({},'{}','{}','{}','{}','{}','{}')"
                 .format(strz[i%10][0], kod(3,6), names.get_first_name(), names.get_last_name(), rand_numer(11),
                         choice(klasy), rand_numer(9)))
conn.commit()
del strz

# Typ_Broni -- Nie zalezny
for i in range(7):
    curr.execute("INSERT INTO Typ_Broni (Typ_Broni)"
                 "VALUES ('{}')"
                 .format(typ_b[i]))
conn.commit()

#Serwisanci -- Nie zalezny
for i in range(15):
    curr.execute("INSERT INTO Serwisanci (Imie, Nazwisko, Miasto, Adres)"
                 "VALUES ('{}','{}','{}','{}')"
                 .format(names.get_first_name(),names.get_last_name(),choice(miasta),adres()))

conn.commit()

#Profesja -- Zalezny od Typ_Broni, Serwisanci
curr.execute("SELECT Id_Typu_Broni FROM Typ_Broni;")
id_typu = curr.fetchall()
curr.execute("SELECT Id_Serwisanta FROM Serwisanci;")
serw = curr.fetchall()
for t in id_typu:
    for s in sample(serw,len(serw)-3):
        curr.execute("INSERT INTO Profesja "
                     "VALUES ({},{})"
                     .format(t[0],s[0]))
conn.commit()
del id_typu, serw

#Rodzaje_Broni -- Zalezny od Typ_Broni
curr.execute("SELECT Id_Typu_Broni FROM Typ_Broni;")
typ = curr.fetchall()
for i in range(22):
    curr.execute("INSERT INTO Rodzaje_Broni (Id_Typu_Broni, Model, Kaliber, Producent, Pochodzenie, Rok_Wynalezienia)"
                 "VALUES ({},'{}',{},'{}','{}','{}')"
                 .format(choice(typ)[0],kod(2,3),choice(kaliber),choice(producenci_broni),choice(pochodzenie),randint(1970,2000)))
conn.commit()
del typ

# Opiekunowie -- Nie zalezny
for i in range(40):
    curr.execute("INSERT INTO Opiekunowie (Imie, Nazwisko, Nr_Pozwolenia, Pesel)"
                 "VALUES ('{}','{}','{}','{}')"
                 .format(names.get_first_name(),names.get_last_name(),kod(2,4),rand_numer(11)))
conn.commit()

# Ezlemplarze -- Zalezny od Rodzaje_Broni, Magazyn, Opiekunowie
curr.execute("SELECT Id_Rodzaju, Rok_Wynalezienia FROM Rodzaje_Broni;")
rodz = curr.fetchall()
curr.execute("SELECT Id_Magazynu, pojemnosc_broni FROM Magazyn;")
mag = curr.fetchall()
curr.execute("SELECT Id_Wlasciciela FROM Opiekunowie;")
wlas = curr.fetchall()
ilosc_w_mag = {}
il = 0
for i in range(400):
    if mag[i%7][0] not in ilosc_w_mag.keys(): ilosc_w_mag[mag[i%7][0]] = 0
    il = mag[i%7][1] - ilosc_w_mag[mag[i%7][0]]
    if il == 0 : continue
    ilosc_w_mag[mag[i%7][0]] +=1
    r = choice(rodz)
    curr.execute("INSERT INTO Egzemplarz_Broni "
                 "VALUES ({},'{}',{},{},'{}','{}')"
                 .format(r[0], kod(3,10), mag[i%7][0], choice(wlas)[0],randint(int(r[1]),2020),'dostepny'))# nr seryjny AAA-nr(10)
conn.commit()
del mag, wlas, rodz , r, ilosc_w_mag, il

# Amunicja -- Zalezny od Opiekunowie
curr.execute("SELECT Id_Typu_Broni FROM Typ_Broni;")
typ = curr.fetchall()
for i in range(44):
    curr.execute("INSERT INTO Amunicja (Id_Typu_Broni, Nazwa, Kaliber, Producent, Data_Produkcji) "
                 "VALUES({},'{}',{},'{}','{}')".format(choice(typ)[0],kod(2,3),choice(kaliber),choice(producenci_ammo),data_ammo()))
conn.commit()
del typ

# Magazyn_Ammo -- Zalezny od Magazyn, Amunicja
curr.execute("SELECT Id_Magazynu, pojemnosc_ammo  FROM Magazyn;")
mag = curr.fetchall()
curr.execute("SELECT ID_Ammo FROM Amunicja;")
id_ammo = curr.fetchall()
ilosc_w_mag = {} 
r = 0
for i in range(50):
    if mag[i%7][0] not in ilosc_w_mag.keys(): ilosc_w_mag[mag[i%7][0]] = 0
    r = mag[i%7][1] - ilosc_w_mag[mag[i%7][0]]
    if r < 70 : continue
    il = randint(50,r)
    ilosc_w_mag[mag[i%7][0]] += il
    curr.execute("INSERT INTO Magazyn_Ammo "
                 "VALUES({},{},{})".format(id_ammo[i%44][0],mag[i%7][0],il))
del mag, id_ammo, ilosc_w_mag, il
conn.commit()

# serwis
curr.execute("SELECT nr_seryjny FROM egzemplarz_broni;")
nr_ser = curr.fetchall()
curr.execute("SELECT id_serwisanta FROM serwisanci;")
serw = curr.fetchall()
for i in range(30):
    curr.execute("INSERT INTO serwis (nr_seryjny,id_serwisanta,data_serwisu, notatka_z_naprawy)"
                        " VALUES('{}',{},'{}','{}')".format(choice(nr_ser)[0], choice(serw)[0],data_ammo(),"bron oddana z serwisu"))
conn.commit()
del nr_ser,serw                        