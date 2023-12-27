1.) Primary serverin postgresql.conf dosyasında şu değişiklikler yapılır:

listen_addresses = '*'

archive_mode = on

max_wal_senders = 5 

max_wal_size = 10GB    

wal_level = replica

hot_standby = on   

archive_command = 'rsync -a %p /opt/pg_archives/%f' 

2.)  Primary serverin pg_hba.conf dosyasında şu değişiklikler yapılır:
	 host    replication  all   secondary_serverin_ip/32   trust
ardından systemctl restart postgresql-16.service denerek servis yeniden başlatılır.

3.)Secondary serverda postgre servisini durdurunuz .Secondary serverda şu işlem yapılır ( eğer database clusterını başlattıysanız ls den sonrasını yapmanız yeterli):
[root@pg ~]# su postgres

bash-4.2$ /usr/pgsql-16/bin/initdb -D /var/lib/pgsql/16/data/

bash-4.2$ cd /var/lib/pgsql/16/data/

bash-4.2$ ls

base pg_commit_ts  pg_hba.conf    pg_logical    pg_notify pg_serial     pg_stat    pg_subtrans pg_twophase  pg_wal   postgresql.auto.conf global pg_dynshmem   pg_ident.conf  pg_multixact  pg_replslot pg_snapshots  pg_stat_tmp  pg_tblspc PG_VERSION   pg_xact  postgresql.conf

bash-4.2$ rm -rf *

4.) Directory yi boşaltınca

bash-4.2$ /usr/pgsql-16/bin/pg_basebackup -D /var/lib/pgsql/16/data/ -h primary_server_ip -p 5432 -Xs -R -P

-D = data directory

-h  = IP address of primary server

-p = Port on which primary instance is running

-Xs = WAL method - stream

-P = Progress information

--slot=SLOTNAME  #optional

-R = Write configuration parameters for replication

#primary_conninfo will automatically be defined by the pg_basebackup command.  Verify the same in postgresql.auto.conf

5.) Secondary serverda postgresql.conf a şunları yazın:
restore_command = 'rsync -a  postgres@192.168.57.101:/opt/pg
_archives/%f %p' 

recovery_target_timeline = 'latest'

6.) Secondary serveri tekrar başlatın.

NOT: Replikasyona başlamadan ilk serverınızda /opt/pg_archives dizini oluşturun ve sahiplik yetkisini postgres e verin
NOT: Kontrol için gereken sorguların görselleri projede yer almaktadır. pg_stat_replication primary'de, pg_stat_wal_receiver ise secondary de sorulmalıdır. 
