
CREATE VIEW Magazyny AS
SELECT s.Id_Strzelnicy, 
m.Adres AS Adres_Magazynu, 
m.Miasto, k.Imie AS Imie_Kierownika, 
k.Nazwisko AS Nazwisko_Kierownika, 
k.Nr_Telefonu
FROM Strzelnica s
	JOIN Magazyn m
		ON s.Id_Magazynu = m.Id_Magazynu
	JOIN Kierownicy k
		ON m.Id_Kierownika = k.Id_Kierownika;

CREATE VIEW Dostepne_Bronie AS
SELECT o.Nazwisko AS Wlasciciel,
o.Nr_Pozwolenia, rb.Model, 
eb.Nr_Seryjny, eb.Stan, 
k.Nr_Telefonu AS Tel_Kierownika_Magazynu
FROM Egzemplarz_Broni eb
	Join Opiekunowie o
		ON eb.Id_Wlasciciela = o.Id_Wlasciciela
	JOIN Rodzaje_Broni rb
		ON eb.Id_Rodzaju = rb.Id_Rodzaju
	JOIN Magazyn m
		ON eb.Id_Magazynu = m.Id_Magazynu
	JOIN Kierownicy k
		ON m.Id_Kierownika = k.Id_Kierownika;

CREATE VIEW Ile_Broni AS
SELECT m.Id_Magazynu,
m.Miasto,
COUNT(eb.Nr_Seryjny) AS Aktualna_Ilosc_Broni, 
m.Pojemnosc_Broni AS Maksymalna_Ilosc_Broni
FROM Magazyn m
	JOIN Egzemplarz_Broni eb
		ON m.Id_Magazynu = eb.Id_Magazynu
GROUP BY m.Id_Magazynu;

CREATE VIEW Ile_Ammo AS
SELECT m.Id_Magazynu,
m.Miasto,
SUM(ma.Ilosc) AS Aktualna_Ilosc_Ammo, 
m.Pojemnosc_Ammo AS Maksymalna_Ilosc_Ammo
FROM Magazyn m
	JOIN Magazyn_Ammo ma
		ON m.Id_Magazynu = ma.Id_Magazynu
GROUP BY m.Id_Magazynu;

CREATE VIEW Ilosc_W_Mag AS
SELECT ia.Id_Magazynu,
ia.Aktualna_Ilosc_Ammo,
ia.Maksymalna_Ilosc_Ammo,
ib.Aktualna_Ilosc_Broni,
ib.Maksymalna_Ilosc_Broni
FROM Ile_Ammo ia
	JOIN Ile_Broni ib
		ON ia.Id_Magazynu = ib.Id_Magazynu;

CREATE VIEW Klasy_Trenerow AS
SELECT t.Klasa, 
s.Miasto, 
COUNT(*) AS Ilosc_Trenerow
FROM Trenerzy t
	JOIN Strzelnica s
		ON t.Id_Strzelnicy = s.Id_Strzelnicy
GROUP BY t.Klasa, s.Miasto;

CREATE VIEW Licznosc_Modeli AS
SELECT rb.Model,
COUNT(*) AS Ilosc
FROM Rodzaje_Broni rb
	JOIN Egzemplarz_Broni eb
		ON rb.Id_Rodzaju = eb.Id_Rodzaju
GROUP BY rb.Model;

CREATE VIEW Ile_Serwisowana AS
SELECT eb.Nr_Seryjny,
COUNT(*) AS Ile_Razy_Serwisowana
FROM Egzemplarz_Broni eb
	JOIN Serwis s
		ON eb.Nr_Seryjny = s.Nr_Seryjny
GROUP BY eb.Nr_Seryjny;

CREATE VIEW Ostatni_Serwis AS
SELECT DISTINCT s.Nr_Seryjny,
s.Nr_Zlecenia_Serwisu,
s.Data_Serwisu,
sp.Imie,
sp.Nazwisko
FROM Serwis s
	JOIN Serwisanci sp
		ON s.Id_Serwisanta = sp.Id_Serwisanta
ORDER BY s.Data_Serwisu DESC;


CREATE VIEW Ostatni_Serwis_Plus AS
SELECT o.Nr_Seryjny,
o.Nr_Zlecenia_Serwisu,
o.Data_Serwisu,
o.Imie,
o.Nazwisko,
i.Ile_Razy_Serwisowana
FROM Ostatni_Serwis o
	LEFT JOIN Ile_Serwisowana i
		ON o.Nr_Seryjny = i.Nr_Seryjny;

CREATE VIEW Ladny_Widok AS
SELECT s.Imie,
s.Nazwisko,
s.Miasto,
tb.Typ_Broni
FROM Profesja p
    RIGHT JOIN Serwisanci s
        ON p.Id_Serwisanta = s.Id_Serwisanta
    LEFT JOIN Typ_Broni tb
        ON p.Id_Typu_Broni = tb.Id_Typu_Broni;








