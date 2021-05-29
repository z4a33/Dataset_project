CREATE TABLE Serwisanci(
Id_Serwisanta SERIAL,
Imie VARCHAR(30),
Nazwisko VARCHAR(50),
Miasto VARCHAR(30),
Adres TEXT
);

CREATE TABLE Opiekunowie(
Id_Wlasciciela SERIAL,
Imie VARCHAR(30),
Nazwisko VARCHAR(50),
Nr_Pozwolenia VARCHAR(7),
Pesel VARCHAR(11)
);

CREATE TABLE Trenerzy(
Id_Strzelnicy INTEGER,
Nr_Licencji VARCHAR(10),
Imie VARCHAR(30),
Nazwisko VARCHAR(50),
Pesel VARCHAR(11),
Klasa VARCHAR(3),
Nr_Telefonu VARCHAR(9)
);

CREATE TABLE Kierownicy(
Id_Kierownika SERIAL,
Imie VARCHAR(30),
Nazwisko VARCHAR(50),
Pesel VARCHAR(11),
Nr_Telefonu VARCHAR(9)
);

CREATE TABLE Typ_Broni(
Typ_Broni VARCHAR(30),
Id_Typu_Broni SERIAL
);

CREATE TABLE Profesja(
Id_Typu_Broni INTEGER,
Id_Serwisanta INTEGER
);

CREATE TABLE Serwis(
Nr_Zlecenia_Serwisu SERIAL,
Nr_Seryjny VARCHAR(14),
Id_Serwisanta INTEGER,
Data_Serwisu DATE,
Notatka_Z_Naprawy TEXT
);

CREATE TABLE Rodzaje_Broni(
Id_Typu_Broni INTEGER,
Id_Rodzaju SERIAL,
Model VARCHAR(20),
KALIBER FLOAT,
Producent VARCHAR(30),
Pochodzenie VARCHAR(30),
Rok_Wynalezienia VARCHAR(4)
);

CREATE TABLE Amunicja(
Id_Typu_Broni INTEGER,
Id_Ammo SERIAL,
Nazwa VARCHAR(30),
Kaliber FLOAT,
Producent VARCHAR(30),
Data_Produkcji DATE
);

CREATE TABLE Magazyn_Ammo(
Id_Ammo INTEGER,
Id_Magazynu INTEGER,
Ilosc INTEGER
);

CREATE TABLE Magazyn(
Id_Kierownika INTEGER,
Id_Magazynu SERIAL,
Miasto VARCHAR(30),
Adres TEXT,
Pojemnosc_Ammo INTEGER,
Pojemnosc_Broni INTEGER
);

CREATE TABLE Egzemplarz_Broni(
Id_Rodzaju INTEGER,
Nr_Seryjny VARCHAR(14),
Id_Magazynu INTEGER,
Id_Wlasciciela INTEGER,
Rok_Produkcji VARCHAR(4),
Stan VARCHAR(15)
);

CREATE TABLE Strzelnica(
Id_Magazynu INTEGER,
Id_Strzelnicy SERIAL,
Miasto VARCHAR(30),
Adres TEXT
);

