---
link: https://www.antoinefi.net/index.php/2024/11/15/laboratoire-infra-linux-ha-7/
excerpt: Du premier au cinquième opus, on est parti d’un serveur LAMP qu’on a
  fait évoluer en une plateforme à haute disponibilité (si ce n’est MySQL, j’ai
  pas oublié). Lors du précédent on a commencé à mesurer l’activité de nos
  machines avec Munin. C’est chouette mais si au moindre problème t’es
  désemparé, savoir faire tout ça te serviras pas à grand chose.
slurped: 2024-12-19T09:18
title: "Laboratoire infra Linux HA #7 – Maintenances basiques MySQL, DRBD et
  diverses – La fin des K7 📼"
---

Du premier au cinquième opus, on est parti d’un serveur LAMP qu’on a fait évoluer en une plateforme à haute disponibilité (si ce n’est `MySQL`, j’ai pas oublié). Lors du précédent on a commencé à mesurer l’activité de nos machines avec `Munin`. C’est chouette mais si au moindre problème t’es désemparé, savoir faire tout ça te serviras pas à grand chose.

On va imaginer que tu n’es pas le seul à administrer la plateforme (co-administration avec l’utilisateur final, collègues distraits ou farceur) et que des erreurs, même simples peuvent arriver. Dont je peux même être l’auteur par méconnaissance ou étourderie.

## Problème de réplication MySQL

Je tenais particulièrement à documenter la correction d’un problème de réplication `MySQL` pour ne pas l’oublier.

Situation : tout va bien dans ta vie, les mojitos sont toujours réussis et bien dosés. Seulement, lorsque tu te connectes à `MySQL` de `lab07-db-01`, le constat est accablant : la réplication est cassée !

lab07-db-01

```
root@lab07-db-01:~# mysql
[...]
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000021 |      357 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0,00 sec)
```

lab07-db-02

```
root@lab07-db-02:~# mysql
[...]
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: lab07-db-01
                  Master_User: roy_batty
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000021
          Read_Master_Log_Pos: 357
               Relay_Log_File: lab07-db-02-relay-bin.000014
                Relay_Log_Pos: 373
        Relay_Master_Log_File: mysql-bin.000021
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
[...] 
                   Last_Errno: 1007
                   Last_Error: Coordinator stopped because there were error(s) in the worker(s). The most recent failure being: Worker 1 failed executing transaction 'ANONYMOUS' at source log mysql-bin.000021, end_log_pos 357. See error log and/or performance_schema.replication_applier_status_by_worker table for more details about this failure or others, if any.
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 157
[...]
               Last_SQL_Errno: 1007
               Last_SQL_Error: Coordinator stopped because there were error(s) in the worker(s). The most recent failure being: Worker 1 failed executing transaction 'ANONYMOUS' at source log mysql-bin.000021, end_log_pos 357. See error log and/or performance_schema.replication_applier_status_by_worker table for more details about this failure or others, if any.
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: 00917e2f-6022-11ef-9584-08002719b90d
             Master_Info_File: mysql.slave_master_info
[...]
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 240926 21:42:35
[...]
1 row in set, 1 warning (0,00 sec)
```

Bon c’est quoi ce bordel ? La réplication n’arrive pas à passer l’instruction à la position 357 du log binaire `mysql-bin.000021`. On peut utiliser `mysqlbinlog` pour lire le log binaire et aller voir ce que MySQL tente de faire à la position 357.

lab07-db-01

```
root@lab07-db-01:~# mysqlbinlog /var/log/mysql/mysql-bin.000021
# The proper term is pseudo_replica_mode, but we use this compatibility alias
# to make the statement usable on server versions 8.0.24 and older.
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
[...]
# at 234
#240926 21:42:35 server id 1  end_log_pos 357 CRC32 0x29128d5e 	Query	thread_id=11	exec_time=0	error_code=0	Xid = 21
SET TIMESTAMP=1727379755/*!*/;
SET @@session.pseudo_thread_id=11/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1168113696/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8mb4 *//*!*/;
SET @@session.character_set_client=255,@@session.collation_connection=255,@@session.collation_server=255/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
/*!80011 SET @@session.default_collation_for_utf8mb4=255*//*!*/;
/*!80016 SET @@session.default_table_encryption=0*//*!*/;
create database toutcasse
/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

Merde les log binaires sont plus bavards que moi ? Je pensais pas que c’était possible. Même en enlevant une partie des lignes, c’est guère lisible.

On entre dans le concret à la ligne 9, faut que tu remarques `end_log_pos 357` et à la ligne 21 la fameuse instruction qui plante `create database toutcasse`.

> Ok le log binaire c’est vraiment pas confortable à lire, le journal d’erreur de MySQL peut très bien nous en dire autant. Mais t’as déjà compris au travers des autres articles du blog que le but c’est de voir plusieurs méthodes.

lab07-db-02

```
root@lab07-db-02:~# grep ERROR /var/log/mysql/error.log | tail -1
2024-09-26T19:42:35.615219Z 11 [ERROR] [MY-010584] [Repl] Replica SQL for channel '': Worker 1 failed executing transaction 'ANONYMOUS' at source log mysql-bin.000021, end_log_pos 357; Error 'Can't create database 'toutcasse'; database exists' on query. Default database: 'toutcasse'. Query: 'create database toutcasse', Error_code: MY-001007
```

Donc ma réplication MySQL est cassée rien que parce qu’il n’arrive pas à créer une nouvelle base sur le slave ? Pas d’bol ! On va regarder ce qui se passe dessus du coup.

lab07-db-02

```
root@lab07-db-02:~# mysql
[...]
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| toutcasse          |
| wordpress          |
+--------------------+
6 rows in set (0,00 sec)
```

Elle existe déjà ?! Mais pourtant il m’a dit qu’il y arrivait pas ! Aaahhh mais si… c’est normal qu’il puisse pas exécuter l’instruction si la base existe déjà. Je vérifie le contenu des bases de chaque côté pour être sûr de pas perdre d’info.

lab07-db-02

```
mysql> use toutcasse;
Database changed
mysql> show tables;
Empty set (0,00 sec)
```

Ok c’est ISO, les deux bases sont identiques sur les deux serveurs DB. On va indiquer au _slave_ de passer l’instruction de la position 357 et puis c’est tout. On redémarrera le _slave_ après.

lab07-db-02

```
root@lab07-db-02:~# mysql
[...]
mysql> SET GLOBAL sql_slave_skip_counter = 357;
Query OK, 0 rows affected, 1 warning (0,00 sec)

mysql> START SLAVE;
Query OK, 0 rows affected, 1 warning (0,02 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: lab07-db-01
                  Master_User: roy_batty
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000021
          Read_Master_Log_Pos: 357
               Relay_Log_File: lab07-db-02-relay-bin.000014
                Relay_Log_Pos: 573
        Relay_Master_Log_File: mysql-bin.000021
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
[...]
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 355
          Exec_Master_Log_Pos: 357
[...]
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
[...]
1 row in set, 1 warning (0,01 sec)
```

Réplication réparée ! Si je choppe celui qui a créé une base sur le slave, je le goum !  
Dans tous les cas, je n’avais pas encore passé les DB en haute dispo juste pour te montrer ça et c’est fait.

## DRBD ne va pas bien

### Waiting for connection

~~A force d’éteindre/allumer mes VM ça arrive que `DRBD` ne sache plus où il habite~~ ( <– C’est totalement faux). Mon nœud primaire se croit secondaire.

lab07-nfs-01

```
root@lab07-nfs-01:~# cat /proc/drbd
version: 8.4.11 (api:1/proto:86-101)
srcversion: 19D914EA50F713FCCE48607 
 0: cs:WFConnection ro:Secondary/Unknown ds:UpToDate/DUnknown C r-----
    ns:0 nr:0 dw:0 dr:0 al:0 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:0
```

Et mon nœud secondaire veut plus m’parler.

lab07-nfs-02

```
root@lab07-nfs-02:~# cat /proc/drbd
version: 8.4.11 (api:1/proto:86-101)
srcversion: 19D914EA50F713FCCE48607
```

`lab07-nfs-01` nous indique _WFConnection_ soit, _waiting for connection_. Qu’à cela ne tienne.

lab07-nfs-02

```
root@lab07-nfs-02:~# drbdadm connect r0
r0: Failure: (158) Unknown resource
additional info from kernel:
unknown resource
Command 'drbdsetup-84 connect r0 ipv4:192.168.50.28:7788 ipv4:192.168.50.27:7788' terminated with exit code 10
root@lab07-nfs-02:~# cat /proc/drbd
version: 8.4.11 (api:1/proto:86-101)
srcversion: 19D914EA50F713FCCE48607 
root@lab07-nfs-02:~# drbdadm up r0
root@lab07-nfs-02:~# cat /proc/drbd
version: 8.4.11 (api:1/proto:86-101)
srcversion: 19D914EA50F713FCCE48607 
 0: cs:Connected ro:Secondary/Primary ds:UpToDate/UpToDate C r-----
    ns:0 nr:0 dw:0 dr:0 al:0 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:0
```

Ça paraissait évident qu’il fallait connecter la ressource, bah non en fait. Donc on va juste la monter et tout fonctionne de nouveau.

![](https://www.antoinefi.net/wp-content/uploads/2024/10/misconf.jpg)

lab07-nfs-02

```
root@lab07-nfs-02:~# systemctl status drbd
○ drbd.service - DRBD -- please disable. Unless you are NOT using a cluster manager.
     Loaded: loaded (/lib/systemd/system/drbd.service; disabled; preset: enabled)
     Active: inactive (dead)
```

Ah donc à un moment j’ai désactivé le lancement du service avec la machine ? Oué je suis un peu Cruchot. On le réactive et on reboot pour vérifier qu’il fonctionnera bien dès le démarrage pour les prochaines fois.

lab07-nfs-02

```
root@lab07-nfs-02:~# systemctl enable drbd
Synchronizing state of drbd.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable drbd
Created symlink /etc/systemd/system/multi-user.target.wants/drbd.service → /lib/systemd/system/drbd.service.
root@lab07-nfs-02:~# reboot
[...]
root@lab07-nfs-02:~# systemctl status drbd
● drbd.service - DRBD -- please disable. Unless you are NOT using a cluster manager.
     Loaded: loaded (/lib/systemd/system/drbd.service; enabled; preset: enabled)
     Active: active (exited) since Tue 2024-10-01 21:18:57 CEST; 15s ago
    Process: 563 ExecStart=/lib/drbd/scripts/drbd start (code=exited, status=0/SUCCESS)
   Main PID: 563 (code=exited, status=0/SUCCESS)
        CPU: 34ms

oct. 01 21:18:56 lab07-nfs-02 drbd[563]: Starting DRBD resources:/lib/drbd/scripts/drbd: ligne 148: /var/lib/linstor/loop_device_mapping: Aucun fichier ou dossier de ce type
oct. 01 21:18:56 lab07-nfs-02 drbd[609]: [
oct. 01 21:18:56 lab07-nfs-02 drbd[609]:      create res: r0
oct. 01 21:18:56 lab07-nfs-02 drbd[609]:    prepare disk: r0
oct. 01 21:18:56 lab07-nfs-02 drbd[609]:     adjust disk: r0
oct. 01 21:18:56 lab07-nfs-02 drbd[609]:      adjust net: r0
oct. 01 21:18:56 lab07-nfs-02 drbd[609]: ]
oct. 01 21:18:57 lab07-nfs-02 drbd[664]: WARN: stdin/stdout is not a TTY; using /dev/console
oct. 01 21:18:57 lab07-nfs-02 drbd[563]: .
oct. 01 21:18:57 lab07-nfs-02 systemd[1]: Finished drbd.service - DRBD -- please disable. Unless you are NOT using a cluster manager..
root@lab07-nfs-02:~# cat /proc/drbd
version: 8.4.11 (api:1/proto:86-101)
srcversion: 19D914EA50F713FCCE48607 
 0: cs:Connected ro:Secondary/Primary ds:UpToDate/UpToDate C r-----
    ns:0 nr:0 dw:0 dr:0 al:0 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:0
```

### Split brain

Là j’ai pas l’exemple sous les yeux mais on peut se retrouver avec un _split brain_ où les deux serveurs sont secondaires. Il va falloir faire marcher un peu plus les neurones.

Détermine quel nœud était primaire en dernier (donc qui a les données à jour) à l’aide de `dmesg`, de `journalctl` ou `/var/log/messages` si tu as un `syslog`.

lab07-nfs-02

```
root@lab07-nfs-02:~# dmesg -T
[...]
[ven. sept. 27 20:57:45 2024] drbd r0: conn( StandAlone -> Unconnected ) 
[ven. sept. 27 20:57:45 2024] drbd r0: Starting receiver thread (from drbd_w_r0 [2265])
[ven. sept. 27 20:57:45 2024] drbd r0: receiver (re)started
[ven. sept. 27 20:57:45 2024] drbd r0: conn( Unconnected -> WFConnection ) 
[ven. sept. 27 20:57:46 2024] drbd r0: Handshake successful: Agreed network protocol version 101
[ven. sept. 27 20:57:46 2024] drbd r0: Feature flags enabled on protocol level: 0xf TRIM THIN_RESYNC WRITE_SAME WRITE_ZEROES.
[ven. sept. 27 20:57:46 2024] drbd r0: conn( WFConnection -> WFReportParams ) 
[ven. sept. 27 20:57:46 2024] drbd r0: Starting ack_recv thread (from drbd_r_r0 [2267])
[ven. sept. 27 20:57:46 2024] block drbd0: drbd_sync_handshake:
[ven. sept. 27 20:57:46 2024] block drbd0: self 2E3194215BBCBB5E:0000000000000000:FE33684D305530AE:FE32684D305530AF bits:0 flags:0
[ven. sept. 27 20:57:46 2024] block drbd0: peer 2E3194215BBCBB5E:0000000000000000:FE33684D305530AF:FE32684D305530AF bits:0 flags:0
[ven. sept. 27 20:57:46 2024] block drbd0: uuid_compare()=0 by rule 40
[ven. sept. 27 20:57:46 2024] block drbd0: peer( Unknown -> Secondary ) conn( WFReportParams -> Connected ) pdsk( DUnknown -> UpToDate ) 
[ven. sept. 27 20:57:46 2024] block drbd0: peer( Secondary -> Primary )
```

Si le pair `lab07-nfs-01` était le dernier serveur primaire (`peer( Secondary -> Primary )`), on va indiquer au dernier nœud secondaire qu’il est nœud secondaire et de se défausser de ses données.

lab07-nfs-02

```
root@lab07-nfs-02:~# drbdadm secondary r0
root@lab07-nfs-02:~# drbdadm -- --discard-my-data connect r0
```

## HAProxy est trop rapide pour le backend

Dans l’article de configuration d’`HAProxy`, j’avais initialement configuré des tiemout beaucoup trop courts et ça a résulté en de multiples erreurs HTTP 504, _gateway timeout_. Mais pas dans la navigation sur le Worpdress… Non non non ! Uniquement dans l’administration du site une fois authentifié.

Donc en cas de 504, vérifie les timeouts que tu as configuré dans **tes** LB.

## Administration de la plateforme

![](https://www.antoinefi.net/wp-content/uploads/2024/09/naviguer_vers_le_lab-1.png)

D’une manière générale, il faudra regarder les logs (un `tail -f` sur les fichiers de log par exemple) de chaque étage en gardant à l’esprit l’ordre dans lequel la navigation d’un visiteur va traverser notre plateforme.

- Du côté LB, uniquement sur le porteur de la VIP.
- Les deux serveurs FRONT.
- Le serveur primaire DRBD.
- Le master MySQL.

Tu sais quoi ? On va enfin pouvoir faire de la haute dispo aussi pour notre MySQL !  
Si si, je te jure !

![fin de la cassette](https://blog.antoinefi.net/wp-content/uploads/2024/05/oa0qcp6gow0-1024x683.jpg)

Fin de la bande