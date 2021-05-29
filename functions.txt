

CREATE OR REPLACE FUNCTION nowy_serwisant
(imiea VARCHAR(30), nazwiskoa VARCHAR(50),miastoa VARCHAR(30),
adresa TEXT,profesjaa TEXT[] )
RETURNS TEXT AS $$
DECLARE
id_serw INT;
id_typu INT;
i TEXT;
BEGIN
	INSERT INTO serwisanci (Imie, Nazwisko, Miasto, Adres)
	VALUES (imiea, nazwiskoa, miastoa, adresa);
	SELECT id_serwisanta INTO id_serw FROM serwisanci 
	WHERE serwisanci.imie = imiea AND serwisanci.nazwisko = nazwiskoa
					AND serwisanci.adres = adresa;
	IF ( NOT FOUND) THEN
		RAISE EXCEPTION 'nie udalo sie dodaj serwisanta';
	END IF;
	FOREACH i IN ARRAY profesjaa
	LOOP
	SELECT id_typu_broni INTO id_typu FROM typ_broni WHERE typ_broni = i;
	INSERT INTO profesja VALUES( id_typu, id_serw);
	END LOOP;
	RETURN 'Dodano ' || imiea;
END;
$$ LANGUAGE 'plpgsql';


CREATE OR REPLACE FUNCTION przenies_bron(mag INT, bron TEXT[])
RETURNS TEXT AS $$
DECLARE
i TEXT;
BEGIN
	IF mag NOT IN (SELECT id_magazynu FROM magazyn) THEN
		RAISE EXCEPTION 'nie ma takiego magazynu';
	END IF;
	FOREACH i IN ARRAY bron
	LOOP
		IF i NOT IN (SELECT nr_seryjny FROM egzemplarz_broni) THEN
			RAISE EXCEPTION 'nie ma takiego egzemplarza';
		END IF;
		UPDATE egzemplarz_broni  SET id_magazynu = mag
		WHERE nr_seryjny = i;
	END LOOP;
	RETURN 'Bronie zosstaly przeniesione';
END;
$$ LANGUAGE 'plpgsql';

CREATE OR REPLACE FUNCTION przenies_wszystkie_bronie(magA INT, magB INT)
RETURNS TEXT AS $$
DECLARE
i TEXT;
pojemnosc INT;
ilosc_broni INT; 
BEGIN
	IF magA NOT IN (SELECT id_magazynu FROM magazyn) THEN
		RAISE EXCEPTION 'Magazyn z ktorego chcesz przeniesc bronie nie istnieje';
	END IF;
	IF magB NOT IN (SELECT id_magazynu FROM magazyn) THEN
		RAISE EXCEPTION 'Magazyn do ktorego chcesz przeniesc bronie nie istnieje';
	END IF;
	SELECT pojemnosc_broni INTO pojemnosc FROM magazyn
	WHERE id_magazynu = magB;
	SELECT COUNT(nr_seryjny) INTO ilosc_broni FROM  egzemplarz_broni 
	WHERE id_magazynu = magA OR id_magazynu = magB;
	IF pojemnosc < ilosc_broni THEN
	RAISE EXCEPTION 'za duzo broni, za maly magazyn';
	END IF;	
	UPDATE egzemplarz_broni SET id_magazynu = magB
	WHERE id_magazynu = magA;
	RETURN 'Bronie zostaly przeniesione';
END;
$$ LANGUAGE 'plpgsql';

CREATE OR REPLACE FUNCTION przenies_ammo(magA INT ,magB INT, ammo INT, ile INT)
RETURNS TEXT AS $$
DECLARE
ilosc_w_B INT;
ilosc_w_A INT;
BEGIN
	IF ammo NOT IN (SELECT id_ammo FROM magazyn_ammo) THEN 
	RAISE EXCEPTION 'nie ma takiej amunicji';
	END IF;
	
	IF magA NOT IN (SELECT id_magazynu FROM  magazyn) THEN
	RAISE EXCEPTION 'nie ma takiego magazynu';
	END IF;
	
	IF magB NOT IN (SELECT id_magazynu FROM  magazyn) THEN
	RAISE EXCEPTION 'nie ma takiego magazynu';
	END IF;
	
	IF ile > (SELECT ilosc FROM magazyn_ammo 
					WHERE id_ammo = ammo AND id_magazynu = magA) THEN
		RAISE EXCEPTION 'Za malo amunicji w magazynie';
	END IF;
	
		IF (SELECT SUM(ilosc) FROM magazyn_ammo 
		WHERE id_magazynu = magB) + ile > (SELECT pojemnosc_ammo FROM magazyn WHERE id_magazynu = magB) THEN
			RAISE EXCEPTION 'Za maly magazyn';
	END IF;
	
	SELECT ilosc INTO ilosc_w_B FROM magazyn_ammo
	WHERE id_ammo = ammo AND id_magazynu = magB;
	
	SELECT ilosc INTO ilosc_w_A FROM magazyn_ammo
	WHERE id_ammo = ammo AND id_magazynu = magA;
	
	IF ammo IN (SELECT id_ammo FROM magazyn_ammo WHERE id_magazynu = magB) THEN

		UPDATE magazyn_ammo SET ilosc = ilosc_w_B +ile WHERE id_ammo = ammo AND id_magazynu = magB;
		
	ELSE 
			INSERT INTO magazyn_ammo VALUES(ammo,magB,ile);
	END IF;
	UPDATE magazyn_ammo SET ilosc =ilosc_w_A-ile  WHERE id_ammo = ammo AND id_magazynu = magA;
RETURN 'udalo sie przenies amunicje';
END;
$$ LANGUAGE 'plpgsql';

CREATE OR REPLACE FUNCTION przenies_ammo_wszystko(magA INT ,magB INT)
RETURNS TEXT AS $$
DECLARE
ile_A INT;
ile_B INT;
pojemnosc_B INT;
BEGIN
	IF magA NOT IN (SELECT id_magazynu FROM magazyn) THEN 
	RAISE EXCEPTION 'Nie ma magazynu z ktorego chcesz porzeniesc ammo';
	ELSIF magB NOT IN (SELECT id_magazynu FROM magazyn) THEN
	RAISE EXCEPTION 'Nie ma magazynu do ktorego chcesz przeniesc';
	END IF;
	SELECT SUM(ilosc) INTO ile_A FROM magazyn_ammo 
	WHERE id_magazynu = magA; 
	SELECT SUM(ilosc) INTO ile_B FROM magazyn_ammo 
	WHERE id_magazynu = magB; 
	SELECT pojemnosc_ammo INTO pojemnosc_B FROM magazyn WHERE id_magazynu = magB;
	IF pojemnosc_B < ile_A + ile_B THEN
	RAISE EXCEPTION 'Za mala pojemnosc magazynu';
	END IF;
	UPDATE magazyn_ammo SET id_magazynu = magB WHERE id_magazynu = magA;
	RETURN 'udalo przeniesc sie amunicje';
END;
$$ LANGUAGE 'plpgsql';

CREATE OR REPLACE FUNCTION bron_do_serwisu(nr_ser VARCHAR(14) ,serw INT)
RETURNS TEXT AS $$
BEGIN
IF nr_ser NOT IN (SELECT nr_seryjny FROM egzemplarz_broni) THEN 
RAISE EXCEPTION 'nie ma takiej broni';
ELSIF serw NOT IN (SELECT id_serwisanta FROM serwisanci) THEN
RAISE EXCEPTION 'Nie ma takiego serwisanta';
ELSIF (SELECT stan FROM egzemplarz_broni WHERE nr_seryjny = nr_ser) = 'niedostepny' THEN
RAISE EXCEPTION 'bron jest juz w serwisie';
END IF;
INSERT INTO serwis (nr_seryjny, id_serwisanta ,data_serwisu,notatka_z_naprawy)
VALUES (nr_ser, serw, CURRENT_DATE,'w trakcie serwisu');
UPDATE egzemplarz_broni SET stan = 'niedostepny' WHERE nr_ser = nr_seryjny;
RETURN 'bron o numerze zostala przekazan do serwiusu ' || nr_ser ;
END; 
$$ LANGUAGE 'plpgsql';

CREATE OR REPLACE FUNCTION bron_z_serwisu(nr_zl INT ,notka TEXT DEFAULT 'serwis zakonczony')
RETURNS TEXT AS $$
DECLARE
nr_ser VARCHAR(14);
BEGIN
IF nr_zl NOT IN (SELECT nr_zlecenia_serwisu FROM serwis) THEN
RAISE EXCEPTION 'nie ma takiego numeru zlecenia';
END IF;
SELECT nr_seryjny INTO nr_ser FROM serwis WHERE nr_zlecenia_serwisu = nr_zl;
UPDATE serwis SET data_serwisu = CURRENT_DATE, notatka_z_naprawy = notka
WHERE nr_zlecenia_serwisu = nr_zl;
UPDATE egzemplarz_broni SET stan = 'dostepny' WHERE nr_seryjny = nr_ser;
RETURN 'bron o numerze zostala zwrocona z serwisu' || nr_ser;
END; 
$$ LANGUAGE 'plpgsql';

