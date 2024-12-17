# ORACLE-TRIGGER
/* 
VERIFIER si la commission est supérieure au salaire
et d’afficher un message d’erreur
à chaque insertion || modification table pilote
.*/
Create or replace trigger VerifComm
before insert or update
on Pilote
for each row 
begin
	if :new.comm>:new.sal
		then
			raise_application_error(-20002, 'Impossible la commission est supérieure au salaire.');
	end if;
END ;
/


/* 
VERIFIER que la somme de toutes les commissions des employés ne dépassent pas 10000.
A chaque insertion || modification table employé,
*/
Create or replace trigger checkComm
after update of comm or insert
on Emp
Declare commNow number;
begin
	select SUM(comm) into commNow from Emp;
	if commNow>10000
		then
			raise_application_error(-20003, 'Impossible, la somme des commissions est trop élevée.');
	end if;
END ;
/

/* 
AJOUTER champ revenu mensuel total, revenu annuel total table Pilote.
ceux-ci doivent être calculés automatiquement.
après chaque insertion || modification,
*/
Alter table Pilote add RevMois number(8,2);
Alter table Pilote add RevAnnee number(8,2);

Create or replace trigger getSomme
before insert or update on Pilote
for each row
begin
	:new.RevMois:=:new.Sal+nvl(:new.Comm,0);
	:new.RevAnnee:=:new.RevMois*12;
END ;
/

/* Trigger d'audit.
RECUPERER dans une table intermédiaire ce qui a été modifié 
et ce qui a été supprimé, en ajoutant la date et
l'utilisateur ayant effectué les actions.
Après chaque modification || suppression sur la table employé,
*/
Create table AuditEmp
as select * from Emp where 2=0;

Create or replace trigger audEmp
before delete or update on Emp
for each row
begin
	insert into AuditEmp() values(select :old.* from Emp, user(), sysdate());
END ;
/

/* Créer une table action avec
id, nom , date et le nom d'utilisateur

 CREER un trigger qui à chaque mise à jour de la table vol (insert, update ou delete) de savoir quelle action
a été réalisée, par qui et quand.
*/
Create table ActionTable
(idA int,
nomA varchar(20),
utilA varchar(60),
dateA date,
primary key(idA));

Create or replace trigger UpVolAction
before insert or update or delete on Vol
for each row
Declare action varchar(20);
MaxId int;
begin
	if inserting
		then
			action:='insert';
	elsif updating
		then
			action:='update';
	else
		action:='delete';
	end if;
	select nvl(max(idA),0) into MaxId from ActionTable; 
	insert into ActionTable values(MaxId+1, action, user(), sysdate());
	DBMS_OUTPUT.put_line(user() || ' a ' || action || ' sur la table Vol le ' || sysdate());
END ;
/

/*AJOUTER le champ 
nbVols table Pilote.
Mettre à jour le champ NbVols. 

Créer un trigger qui permet 
si c'est une insertion d'incrémenter le nombre de vols, 
si c'est une modification || une suppression de décrémenter le nombre de vols
table Pilote
.*/ 
Alter table pilote
add NbVols int not null;

update Pilote P set NbVols=(select count(Pilote) from Affectation A where P.NoPilote=A.Pilote group by Pilote);

Create or replace trigger getNbVols
after insert or update or delete on Affectation
for each row
begin
	if inserting
		then
			update Pilote set NbVols=nvl(NbVols+1,1) where NoPilote=:new.Pilote;
	elsif updating('Pilote')
		then
			update Pilote set NbVols=nvl(NbVols-1,1) where NoPilote=:old.Pilote;
			update Pilote set NbVols=nvl(NbVols+1,1) where NoPilote=:new.Pilote;
	else
		update Pilote set NbVols=NbVols-1 where NoPilote=:old.Pilote;
	end if;
END ;
/

/* Réaliser une prcédure qui permet après la saisie du numéro Pilote de vérifier sa commission et son salaire.
Si Comm>Sal -> Comm=Sal/2
Si Comm est null ou égale à 0 -> Comm=Sal/4
Si Comm et Sal sont null -> Afficher une message d'erreur
Ajouter l'option qui, si le pilote n'existe pas, affiche un message d'erreur.*/
