Calvin D
Kilian Georget

# MSPR_JAVA : RECYCL

### Générer la base FRANCE

Ouvrir SQL+ et saisir les lignes suivantes : 
- @ C:\load\france\cre_recycl_france.sql
- @ C:\load\france\utils.plsql


Ouvrir un cmd et saisir les lignes suivantes : 
```
sqlldr userid=rfrance/rfrance   control=c:\load\paris\entreprise.ctl   log=c:\load\log\entrepriseParis.log
sqlldr userid=rfrance/rfrance   control=c:\load\lille\entreprise.ctl   log=c:\load\log\entrepriseLille.log
sqlldr userid=rfrance/rfrance   control=c:\load\paris\centre.ctl   log=c:\load\log\centreParis.log
sqlldr userid=rfrance/rfrance   control=c:\load\lille\centre.ctl   log=c:\load\log\centreLille.log
sqlldr userid=rfrance/rfrance   control=c:\load\paris\typedechet.ctl   log=c:\load\log\typedechetParis.log
sqlldr userid=rfrance/rfrance   control=c:\load\paris\camion.ctl   log=c:\load\log\camionParis.log
sqlldr userid=rfrance/rfrance   control=c:\load\lille\camion.ctl   log=c:\load\log\camionLille.log
sqlldr userid=rfrance/rfrance   control=c:\load\lille\fonction.ctl   log=c:\load\log\fonctionLille.log
sqlldr userid=rfrance/rfrance   control=c:\load\france\site.ctl   log=c:\load\log\siteLogFrance.log
sqlldr userid=rfrance/rfrance   control=c:\load\paris\employe.ctl   log=c:\load\log\employeParis.log
sqlldr userid=rfrance/rfrance   control=c:\load\lille\employe.ctl   log=c:\load\log\employeLille.log
sqlldr userid=rfrance/rfrance   control=c:\load\paris\tournee.ctl   log=c:\load\log\tourneeParis.log
sqlldr userid=rfrance/rfrance   control=c:\load\lille\tournee.ctl   log=c:\load\log\tourneeLille.log
sqlldr userid=rfrance/rfrance   control=c:\load\paris\demande.ctl   log=c:\load\log\demandeParis.log
sqlldr userid=rfrance/rfrance   control=c:\load\lille\demande.ctl   log=c:\load\log\demandeLille.log
sqlldr userid=rfrance/rfrance   control=c:\load\paris\detaildemande.ctl   log=c:\load\log\detaildemandeParis.log
sqlldr userid=rfrance/rfrance   control=c:\load\lille\detaildemande.ctl   log=c:\load\log\detaildemandeLille.log
sqlldr userid=rfrance/rfrance   control=c:\load\paris\detaildepot.ctl   log=c:\load\log\detaildepotParis.log
sqlldr userid=rfrance/rfrance   control=c:\load\lille\detaildepot.ctl   log=c:\load\log\detaildepotLille.log
```

Les fichiers de logs sont disponibles dans c:\load\log\

### Sauvegarder la base

Ouvrir une cmd puis saisir : 
```
rman target u1/p1[rfrance]
```
Si le catalogue n'existe pas, il faut le créer une première fois à l'aide des commandes : 
```
CONNECT CATALOG rfrance@xe
register database;
```
Enfin, pour créer une sauvegarde de la base de données, saisir les commandes suivantes :
```
shutdown immediate;
startup mount; 
backup database;
recover database;
alter database open resetlogs;
```
La sauvegarde est disponible à l'adresse suivante dans le cas d'une installation par défaut avec aaaa_mm_jj correspondant à une date:
```
C:\oraclexe\app\oracle\fast_recovery_area\XE\BACKUPSET\aaaa_mm_jj
```

### Restaurer la base

Ouvrir une cmd puis saisir : 
```
rman target u1/p1[rfrance]
```

Pour restaurer la base à partir de la dernière sauvegarde, exécuter les commandes suivantes :
```
shutdown immediate;
startup mount; 
RESTORE DATABASE;
recover database;
alter database open resetlogs;
```

### Quelques requêtes
<<?>> correspond à une valeur à remplacer
##### Chercher les demandes qui ont été faites après une date donnée. (Params : date)
```
SELECT * FROM DEMANDE WHERE DATEDEMANDE > ?;
```
##### Pour une demande donnée, afficher la raison sociale de l’entreprise, la tournée correspondante et la quantité à récupérer pour chaque type de déchet. (Params : demande)
```
SELECT entreprise.RaisonSociale, tournee.NoTournee, detaildemande.QuantiteEnlevee FROM demande LEFT OUTER JOIN detaildemande ON demande.Nodemande = detaildemande.Nodemande LEFT OUTER JOIN entreprise ON demande.Siret = entreprise.Siret LEFT OUTER JOIN tournee ON demande.NoTournee = tournee.NoTournee WHERE demande.NoDemande = ?; 
```
##### Afficher la quantité totale récupérée par type de déchet pour un mois/année donné. (Params : année, mois)
```
SELECT T.NomTypeDechet,  SUM(dd.quantiteenlevee) FROM TYPEDECHET T, DETAILDEMANDE DD, DEMANDE D WHERE T.NoTypeDechet = DD.NoTypeDechet AND DD.NoDemande = D.NoDemande AND EXTRACT(year from DateEnlevement)= ? AND EXTRACT(month FROM DateEnlevement) = ? GROUP BY NomTypeDechet; 
```
##### Afficher les employés ayant réalisé moins de n tournées. Triez le résultat sur le nombre de tournées. (Params : n)
```
select noemploye, count(*) as tournee from tournee GROUP BY noemploye HAVING count(*) < ? ORDER BY count(*);
```
##### Affichez les informations de l’entreprise qui a réalisé plus de demandes que l’entreprise Formalys (ou une autre entreprise dont vous fournissez le nom).  (Params : nom)
```
SELECT e.raisonsociale, count(*) as total FROM demande d INNER JOIN entreprise e ON d.siret = e.siret  GROUP BY e.raisonsociale HAVING count(*) > (SELECT count(*) FROM demande d INNER JOIN entreprise e ON d.siret = e.siret where e.raisonsociale = 'Formalys');
```
##### Affichez les informations des demandes qui ne sont pas encore inscrites dans une tournée.
```
SELECT * FROM demande WHERE notournee IS NULL;
```

### Quelques commandes 

##### Obtenir le nom de la base de données
```
SELECT * FROM GLOBAL_NAME;
```
##### Obtenir la version du SGBD utilisé
```
SELECT * FROM v$version;
SELECT * FROM v$version WHERE banner LIKE 'Oracle%';
```
##### Obtenir les tablespaces
```
SELECT TABLESPACE_NAME, STATUS, CONTENTS FROM USER_TABLESPACES;
```
##### Obtenir les tables de la base de données
```
SELECT table_name, owner AS schema_name FROM all_tables ORDER BY owner;
```
##### Obtenir les index (remplacer table_name par le nom de la table souhaité)
```
SELECT index_name FROM all_indexes WHERE table_name = 'table_name';
```
##### Obtenir les vues
```
SELECT VIEW_NAME FROM USER_VIEWS;
```
##### Obtenir les procédures, fonctions et triggers
```
SELECT DISTINCT(name), type FROM ALL_SOURCE WHERE OWNER = 'RFRANCE' ORDER BY NAME;
```
##### Emplacement de la dernière sauvegarde 
```
C:\oraclexe\app\oracle\fast_recovery_area\XE\BACKUPSET\aaaa_mm_jj
```
