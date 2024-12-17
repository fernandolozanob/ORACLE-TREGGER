# ORACLE-TRIGGER

**VERIFIER** si la commission est supérieure au salaire <br/>
Afficher un message d’erreur <br/>
à chaque insertion || modification table pilote <br/>

**Create or replace trigger VerifComm** <br/>
**before insert or update** <br/>
**on Pilote**<br/>
for each row <br/>
**begin** <br/>
	if :new.comm>:new.sal <br/>
		then <br/>
			raise_application_error(-20002, 'Impossible la commission est supérieure au salaire.'); <br/>
	end if;<br/>
**END;**<br/>

<br/><br/>


**VERIFIER** que la somme de toutes les commissions des employés ne dépassent pas 10000. <br/>
A chaque insertion || modification table employé <br/>

**Create or replace trigger checkComm**<br/>
**after update of comm or insert** <br/>
**on Emp** <br/>
Declare commNow number; <br/>
**begin** <br/>
	select SUM(comm) into commNow from Emp; <br/>
	if commNow>10000 <br/>
		then <br/>
			raise_application_error(-20003, 'Impossible, la somme des commissions est trop élevée.'); <br/>
	end if; <br/>
**END;** <br/>

<br/><br/>

**AJOUTER* champ revenu mensuel total, revenu annuel total table Pilote.<br/>
ceux-ci doivent être calculés automatiquement.<br/>
après chaque insertion || modification <br/>

Alter table Pilote add RevMois number(8,2); <br/>
Alter table Pilote add RevAnnee number(8,2); <br/>

**Create or replace trigger getSomme** <br/>
**before insert or update on Pilote** <br/>
for each row <br/>
**begin** <br/>
	:new.RevMois:=:new.Sal+nvl(:new.Comm,0); <br/>
	:new.RevAnnee:=:new.RevMois*12; <br/>
**END;** <br/>
<br/><br/>

**Trigger d'audit.**</br>
**RECUPERER** dans une table intermédiaire ce qui a été modifié <br/>
et ce qui a été supprimé, en ajoutant la date et <br/>
l'utilisateur ayant effectué les actions. <br/>
Après chaque modification || suppression sur la table employé, <br/>

**Create table AuditEmp** <br/>
**as select * from Emp where 2=0;** <br/>

**Create or replace trigger audEmp** <br/>
**before delete or update on Emp** <br/>
for each row <br/>
**begin** <br/>
	insert into AuditEmp() values(select :old.* from Emp, user(), sysdate()); <br/>
**END;** <br/>
<br/><br/>

 **CREER** une table action avec <br/>
id, nom , date et le nom d'utilisateur <br/>

 **CREER** un trigger qui à chaque mise à jour de la table vol (insert, update ou delete) de savoir quelle action </br>
a été réalisée, par qui et quand. </br>

**Create table ActionTable** </br>
(idA int, </br>
nomA varchar(20), </br>
utilA varchar(60), </br>
dateA date, </br>
primary key(idA)); </br>

**Create or replace trigger UpVolAction** </br>
**before insert or update or delete on Vol** </br>
for each row </br>
Declare action varchar(20); </br>
MaxId int; </br>
**begin** </br>
	if inserting </br>
		then </br>
			action:='insert'; </br>
	elsif updating </br>
		then </br>
			action:='update'; </br>
	else </br>
		action:='delete'; </br>
	end if; </br>
	select nvl(max(idA),0) into MaxId from ActionTable; </br>
	insert into ActionTable values(MaxId+1, action, user(), sysdate()); </br>
	DBMS_OUTPUT.put_line(user() || ' a ' || action || ' sur la table Vol le ' || sysdate()); </br>
**END;** </br>
</br></br>

**AJOUTER** le champ </br>
nbVols table Pilote. </br>
Mettre à jour le champ NbVols.  </br>

**Créer** un trigger qui permet  </br>
si c'est une insertion d'incrémenter le nombre de vols, </br>
si c'est une modification || une suppression de décrémenter le nombre de vols</br>
table Pilote</br>

Alter table pilote </br>
add NbVols int not null; </br>

update Pilote P set NbVols=(select count(Pilote) from Affectation A where P.NoPilote=A.Pilote group by Pilote);</br>

**Create or replace trigger getNbVols** </br>
**after insert or update or delete on Affectation** </br>
for each row </br>
**begin** </br>
	if inserting </br>
		then </br>
			update Pilote set NbVols=nvl(NbVols+1,1) where NoPilote=:new.Pilote; </br>
	elsif updating('Pilote') </br>
		then</br>
			update Pilote set NbVols=nvl(NbVols-1,1) where NoPilote=:old.Pilote;</br>
			update Pilote set NbVols=nvl(NbVols+1,1) where NoPilote=:new.Pilote;</br>
	else</br>
		update Pilote set NbVols=NbVols-1 where NoPilote=:old.Pilote;</br>
	end if;</br>
**END;**</br>


**Réaliser* une prcédure qui permet après la saisie du numéro Pilote de vérifier sa commission et son salaire.</br>
Si Comm>Sal -> Comm=Sal/2 </br>
Si Comm est null ou égale à 0 -> Comm=Sal/4 </br>
Si Comm et Sal sont null -> Afficher une message d'erreur </br>
Ajouter l'option qui, si le pilote n'existe pas, affiche un message d'erreur.</br>
