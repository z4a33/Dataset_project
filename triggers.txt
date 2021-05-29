CREATE OR REPLACE FUNCTION dobry_rok()
RETURNS TRIGGER AS $$ 
DECLARE
rok_wyn INT;
BEGIN
	SELECT TO_NUMBER(rok_wynalezienia,'9999') INTO rok_wyn FROM rodzaje_broni
	WHERE id_rodzaju = NEW.id_rodzaju;
	IF rok_wyn > TO_NUMBER(NEW.rok_produkcji,'9999') THEN
	RAISE EXCEPTION 'zla data produkcji';
	END IF;
	RETURN NEW;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER sprawdz_rok_prod BEFORE  INSERT ON egzemplarz_broni
FOR EACH ROW EXECUTE PROCEDURE dobry_rok();

CREATE OR REPLACE FUNCTION miejsce_w_mag_ammo()
RETURNS TRIGGER AS $$ 
DECLARE
miejsce INT;
ile_jest INT;
BEGIN
SELECT pojemnosc_ammo INTO miejsce FROM magazyn
WHERE id_magazynu = NEW.id_magazynu;
SELECT SUM(ilosc) INTO ile_jest FROM magazyn_ammo 
WHERE id_magazynu = NEW.id_magazynu;
IF ile_jest+NEW.ilosc > miejsce THEN
RAISE EXCEPTION 'za malo miejsca w magazynie';
END IF;
RETURN NEW;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER pojemnosc_ammo_trigger BEFORE  INSERT ON magazyn_ammo
FOR EACH ROW EXECUTE PROCEDURE miejsce_w_mag_ammo();



CREATE OR REPLACE FUNCTION miejsce_w_mag_bron()
RETURNS TRIGGER AS $$ 
DECLARE
miejsce INT;
ile_jest INT;
BEGIN
SELECT pojemnosc_broni INTO miejsce FROM magazyn
WHERE id_magazynu = NEW.id_magazynu;
SELECT COUNT(*) INTO ile_jest FROM egzemplarz_broni
WHERE id_magazynu = NEW.id_magazynu;
IF ile_jest+1 > miejsce THEN
RAISE EXCEPTION 'za malo miejsca w magazynie';
END IF;
RETURN NEW;
END;
$$ LANGUAGE 'plpgsql';

CREATE TRIGGER pojemnosc_broni_trigger BEFORE  INSERT ON egzemplarz_broni
FOR EACH ROW EXECUTE PROCEDURE miejsce_w_mag_bron();

