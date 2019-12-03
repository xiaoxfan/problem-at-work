#### mysql主从同步-数据不一致

从库数据与主库不一致

重新做主从，完全同步

1. 进入master，锁表，防止数据写入

   ```js
   mysql> flush tables with read lock
   ```

2. 备份master数据

   ```js
   mysqldump -uroot -p -P3306 -hlocalhost > myslq.bak.sql
   ```

3. 查看master状态 在master执行

   ```js
   mysql> show master status\G;
   *************************** 1. row ***************************
                File: master-bin-log.000005
            Position: 82279
        Binlog_Do_DB: 
    Binlog_Ignore_DB: 
   Executed_Gtid_Set: 
   1 row in set (0.00 sec)
   
   ERROR: 
   No query specified
   ```

4. 将master备份文件拷贝到slave

   ```js
   scp mysql.bak.sql root@192.168.1.206:/tmp/
   ```

5. 停止从库状态

   ```js
   mysql> stop slave;
   ```

6. slave导入数据

   ```js
   mysql> source /tmp/mysql.bak.sql
   ```

7. 设置slave同步，同步点就是第3步中的File和Position

   ```js
   change master to 
   master_host='192.168.1.1', 
   master_user='root', 
   master_password='123456',
   master_log_file='master-bin-log.000005', 
   master_log_pos=82279;
   ```

8. 重新开始slave同步

   ```js
   mysql> start slave;
   ```

9. 查看slave同步状态

   **Slave_IO_Running: Yes**
   **Slave_SQL_Running: Yes**

   ```js
   mysql> show slave status\G;
   *************************** 1. row ***************************
                  Slave_IO_State: Waiting for master to send event
                     Master_Host: 192.168.1.1
                     Master_User: root
                     Master_Port: 3326
                   Connect_Retry: 60
                 Master_Log_File: master-bin-log.000005
             Read_Master_Log_Pos: 82279
                  Relay_Log_File: wmbt-prod-2-relay-bin.000002
                   Relay_Log_Pos: 739
           Relay_Master_Log_File: master-bin-log.000005
                Slave_IO_Running: Yes
               Slave_SQL_Running: Yes
                 Replicate_Do_DB: 
             Replicate_Ignore_DB: 
              Replicate_Do_Table: 
          Replicate_Ignore_Table: 
         Replicate_Wild_Do_Table: 
     Replicate_Wild_Ignore_Table: 
                      Last_Errno: 0
                      Last_Error: 
                    Skip_Counter: 0
             Exec_Master_Log_Pos: 82279
                 Relay_Log_Space: 952
                 Until_Condition: None
                  Until_Log_File: 
                   Until_Log_Pos: 0
              Master_SSL_Allowed: No
              Master_SSL_CA_File: 
              Master_SSL_CA_Path: 
                 Master_SSL_Cert: 
               Master_SSL_Cipher: 
                  Master_SSL_Key: 
           Seconds_Behind_Master: 0
   Master_SSL_Verify_Server_Cert: No
                   Last_IO_Errno: 0
                   Last_IO_Error: 
                  Last_SQL_Errno: 0
                  Last_SQL_Error: 
     Replicate_Ignore_Server_Ids: 
                Master_Server_Id: 1
                     Master_UUID: 9f7fb2dc-1231-11e9-8fae-00163e0495f4
                Master_Info_File: /var/lib/mysql/master.info
                       SQL_Delay: 0
             SQL_Remaining_Delay: NULL
         Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
              Master_Retry_Count: 86400
                     Master_Bind: 
         Last_IO_Error_Timestamp: 
        Last_SQL_Error_Timestamp: 
                  Master_SSL_Crl: 
              Master_SSL_Crlpath: 
              Retrieved_Gtid_Set: 
               Executed_Gtid_Set: 
                   Auto_Position: 0
            Replicate_Rewrite_DB: 
                    Channel_Name: 
              Master_TLS_Version: 
   1 row in set (0.00 sec)
   
   ERROR: 
   No query specified
   ```