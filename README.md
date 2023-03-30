# Ansible-MySQL-GTID-Reseed
Ansible MySQL 8 GTID Reseed Playbook


## Overview

This is a slightly different take on an older Ansible playbook I wrote to reseed a MySQL 5.6 replica.  
The playbook has a few debug statements so that you can see what is going on.

## Requirements

- ansible [core 2.14.0]
  python version = 3.11.0 (main, Nov 21 2022, 19:57:48) [Clang 14.0.0 (clang-1400.0.29.202)]
  jinja version = 3.1.2
  libyaml = True

- ansible-playbook [core 2.14.0]
  python version = 3.11.0 (main, Nov 21 2022, 19:57:48) [Clang 14.0.0 (clang-1400.0.29.202)]
  jinja version = 3.1.2
  libyaml = True

- mysql  Ver 8.0.32-24
- xtrabackup version 8.0.32-25 based on MySQL server 8.0.32 Linux (x86_64) (revision id: 14f007fb)

## Usage
```bash
(data-sync) ansible-playbook -i inventory reseed.yml -e "source=primary target=replica"

PLAY [Check params] *********************************************************************************************

TASK [assert] ***************************************************************************************************
ok: [127.0.0.1] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [assert] ****************************************************************************************************
ok: [127.0.0.1] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [assert] ****************************************************************************************************
ok: [127.0.0.1] => {
    "changed": false,
    "msg": "All assertions passed"
}

PLAY [Ensure Source looks OK] ************************************************************************************

PLAY [Ensure Target looks OK] ************************************************************************************

PLAY [primary] ***************************************************************************************************

TASK [Install percona-toolkit] ***********************************************************************************
ok: [primary]

TASK [Stop MySQL on the target] **********************************************************************************
changed: [primary -> replica(10.5.0.153)]

TASK [Delete data in data01 logs01 | target] *********************************************************************
changed: [primary -> replica(10.5.0.153)]

TASK [wait for ibdata01 to be deleted | target] ******************************************************************
ok: [primary -> replica(10.5.0.153)]

TASK [Clean any failed screen sessions on the replica | target] **************************************************
fatal: [primary -> replica(10.5.0.153)]: FAILED! => {"changed": true, "cmd": "screen -ls | grep migrate | cut -d. -f1 | awk '{print $1}' | xargs kill 2>>/dev/null", "delta": "0:00:00.019011", "end": "2023-03-30 04:40:10.791402", "msg": "non-zero return code", "rc": 123, "start": "2023-03-30 04:40:10.772391", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring

TASK [Clean any failed screen sessions on the primary | source] ***************************************************
fatal: [primary]: FAILED! => {"changed": true, "cmd": "screen -ls | grep migrate2 | cut -d. -f1 | awk '{print $1}' | xargs kill 2>>/dev/null", "delta": "0:00:00.015764", "end": "2023-03-30 04:41:19.997435", "msg": "non-zero return code", "rc": 123, "start": "2023-03-30 04:41:19.981671", "stderr": "", "stderr_lines": [], "stdout": "", "stdout_lines": []}
...ignoring

TASK [start listener on replica | target] ***************************************************************************
ASYNC OK on primary: jid=485522636369.28608
changed: [primary -> replica(10.8.0.153)]

TASK [ansible.builtin.debug] ****************************************************************************************
ok: [primary] => {
    "migrate_nc1": {
        "ansible_job_id": "485522636369.28608",
        "changed": true,
        "cmd": "screen -S migrate -md bash -c 'nc -l 4000 | tee >(sha1sum > target_checksum) > mysql-$(date +%F).xbstream'",
        "delta": "0:00:00.006944",
        "end": "2023-03-30 04:40:11.873600",
        "failed": false,
        "failed_when_result": false,
        "finished": 1,
        "msg": "",
        "rc": 0,
        "results_file": "/root/.ansible_async/485522636369.28608",
        "start": "2023-03-30 04:40:11.866656",
        "started": 1,
        "stderr": "",
        "stderr_lines": [],
        "stdout": "",
        "stdout_lines": []
    }
}

TASK [start transfer on primary] ************************************************************************************
changed: [primary]

TASK [ansible.builtin.debug] ****************************************************************************************
ok: [primary] => {
    "migrate_nc2": {
        "changed": true,
        "cmd": "screen -S migrate2 -md bash -c 'xtrabackup --backup --compress --compress-threads=1  --stream=xbstream --parallel=1 --datadir=/var/lib/mysql --target-dir=./ | tee >( sha1sum > source_checksum ) | nc -N \"replica\" 4000'",
        "delta": "0:00:00.006197",
        "end": "2023-03-30 04:41:22.715820",
        "failed": false,
        "failed_when_result": false,
        "msg": "",
        "rc": 0,
        "start": "2023-03-30 04:41:22.709623",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "",
        "stdout_lines": []
    }
}

TASK [Get pid of screen session on the primary] *********************************************************************
changed: [primary]

TASK [ansible.builtin.debug] *****************************************************************************************
ok: [primary] => {
    "p_screen": {
        "changed": true,
        "cmd": [
            "pgrep",
            "screen"
        ],
        "delta": "0:00:00.010556",
        "end": "2023-03-30 04:41:23.212209",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2023-03-30 04:41:23.201653",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "25624",
        "stdout_lines": [
            "25624"
        ]
    }
}

TASK [Wait for screen session on primary to complete] ******************************************************************
ok: [primary]

PLAY [replica] **********************************************************************************************************

TASK [Install percona-toolkit] ******************************************************************************************
ok: [replica]

TASK [Get the sha1sum of the source file] *******************************************************************************
changed: [replica -> primary(10.5.0.152)]

TASK [ansible.builtin.debug] *********************************************************************************************
ok: [replica] => {
    "source_checksum": {
        "changed": true,
        "cmd": "cat source_checksum | awk '{print $1}'",
        "delta": "0:00:00.023957",
        "end": "2023-03-30 04:41:29.661767",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2023-03-30 04:41:29.637810",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "8136de005d2ee0a46a82493dac21d9ae3333d9df",
        "stdout_lines": [
            "8136de005d2ee0a46a82493dac21d9ae3333d9df"
        ]
    }
}

TASK [Get the sha1sum of the target file] *********************************************************************************
changed: [replica]

TASK [ansible.builtin.debug] **********************************************************************************************
ok: [replica] => {
    "target_checksum": {
        "changed": true,
        "cmd": "cat target_checksum | awk '{print $1}'",
        "delta": "0:00:00.008021",
        "end": "2023-03-30 04:40:21.292793",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2023-03-30 04:40:21.284772",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "8136de005d2ee0a46a82493dac21d9ae3333d9df",
        "stdout_lines": [
            "8136de005d2ee0a46a82493dac21d9ae3333d9df"
        ]
    }
}

TASK [Check if the checksums match] ****************************************************************************************
ok: [replica] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [extract the xbstream files | target] *********************************************************************************
changed: [replica]

TASK [ansible.builtin.debug] ***********************************************************************************************
ok: [replica] => {
    "xbstream": {
        "changed": true,
        "cmd": "xbstream -xv < mysql-$(date +%F).xbstream -C /var/lib/mysql/",
        "delta": "0:00:00.181684",
        "end": "2023-03-30 04:40:21.979772",
        "failed": false,
        "failed_when_result": false,
        "msg": "",
        "rc": 0,
        "start": "2023-03-30 04:40:21.798088",
        "stderr": "ibdata1.qp\nsys/sys_config.ibd.qp\nbook/source_words.ibd.qp\nbook/million_words.ibd.qp\nmysql.ibd.qp\nundo_002.qp\nundo_001.qp\nperformance_schema/binary_log_trans_198.sdi.qp\nperformance_schema/user_defined_fun_197.sdi.qp\nperformance_schema/socket_instances_156.sdi.qp\nperformance_schema/events_stages_su_126.sdi.qp\nperformance_schema/persisted_variab_196.sdi.qp\nperformance_schema/users_153.sdi.qp\nperformance_schema/events_stages_su_125.sdi.qp\nperformance_schema/replication_grou_172.sdi.qp\nperformance_schema/events_transacti_147.sdi.qp\nperformance_schema/replication_grou_178.sdi.qp\nperformance_schema/events_waits_sum_96.sdi.qp\nperformance_schema/replication_conn_173.sdi.qp\nperformance_schema/table_io_waits_s_117.sdi.qp\nperformance_schema/keyring_componen_202.sdi.qp\nperformance_schema/events_errors_su_150.sdi.qp\nperformance_schema/mutex_instances_106.sdi.qp\nperformance_schema/processlist_109.sdi.qp\nperformance_schema/session_variable_194.sdi.qp\nperformance_schema/metadata_locks_168.sdi.qp\nperformance_schema/events_stages_cu_120.sdi.qp\nperformance_schema/events_waits_cur_93.sdi.qp\nperformance_schema/setup_consumers_112.sdi.qp\nperformance_schema/file_summary_by__103.sdi.qp\nperformance_schema/objects_summary__107.sdi.qp\nperformance_schema/events_statement_134.sdi.qp\nperformance_schema/socket_summary_b_158.sdi.qp\nperformance_schema/events_stages_su_123.sdi.qp\nperformance_schema/status_by_accoun_186.sdi.qp\nperformance_schema/session_connect__159.sdi.qp\nperformance_schema/events_transacti_140.sdi.qp\nperformance_schema/events_statement_133.sdi.qp\nperformance_schema/status_by_thread_188.sdi.qp\nperformance_schema/events_statement_130.sdi.qp\nperformance_schema/memory_summary_g_162.sdi.qp\nperformance_schema/events_statement_136.sdi.qp\nperformance_schema/variables_info_195.sdi.qp\nperformance_schema/events_statement_128.sdi.qp\nperformance_schema/replication_appl_180.sdi.qp\nperformance_schema/replication_appl_176.sdi.qp\nperformance_schema/events_transacti_143.sdi.qp\nperformance_schema/events_stages_su_127.sdi.qp\nperformance_schema/events_statement_129.sdi.qp\nperformance_schema/events_errors_su_148.sdi.qp\nperformance_schema/events_transacti_146.sdi.qp\nperformance_schema/replication_appl_177.sdi.qp\nperformance_schema/malloc_stats_201.sdi.qp\nperformance_schema/keyring_keys_161.sdi.qp\nperformance_schema/session_account__160.sdi.qp\nperformance_schema/file_summary_by__104.sdi.qp\nperformance_schema/data_locks_169.sdi.qp\nperformance_schema/error_log_92.sdi.qp\nperformance_schema/session_status_191.sdi.qp\nperformance_schema/events_waits_sum_101.sdi.qp\nperformance_schema/events_statement_132.sdi.qp\nperformance_schema/log_status_183.sdi.qp\nperformance_schema/replication_appl_174.sdi.qp\nperformance_schema/events_transacti_145.sdi.qp\nperformance_schema/replication_asyn_181.sdi.qp\nperformance_schema/memory_summary_b_165.sdi.qp\nperformance_schema/file_instances_102.sdi.qp\nperformance_schema/malloc_stats_tot_200.sdi.qp\nperformance_schema/replication_asyn_182.sdi.qp\nperformance_schema/host_cache_105.sdi.qp\nperformance_schema/data_lock_waits_170.sdi.qp\nperformance_schema/events_stages_hi_122.sdi.qp\nperformance_schema/tls_channel_stat_199.sdi.qp\nperformance_schema/memory_summary_b_166.sdi.qp\nperformance_schema/table_handles_167.sdi.qp\nperformance_schema/replication_appl_179.sdi.qp\nperformance_schema/setup_actors_111.sdi.qp\nperformance_schema/events_transacti_142.sdi.qp\nperformance_schema/prepared_stateme_184.sdi.qp\nperformance_schema/table_io_waits_s_116.sdi.qp\nperformance_schema/setup_threads_115.sdi.qp\nperformance_schema/events_stages_su_124.sdi.qp\nperformance_schema/events_statement_135.sdi.qp\nperformance_schema/hosts_155.sdi.qp\nperformance_schema/events_errors_su_149.sdi.qp\nperformance_schema/events_statement_137.sdi.qp\nperformance_schema/global_variables_193.sdi.qp\nperformance_schema/replication_conn_171.sdi.qp\nperformance_schema/events_transacti_144.sdi.qp\nperformance_schema/cond_instances_91.sdi.qp\nperformance_schema/memory_summary_b_164.sdi.qp\nperformance_schema/events_waits_his_94.sdi.qp\nperformance_schema/events_errors_su_152.sdi.qp\nperformance_schema/events_waits_sum_98.sdi.qp\nperformance_schema/status_by_host_187.sdi.qp\nperformance_schema/events_statement_139.sdi.qp\nperformance_schema/socket_summary_b_157.sdi.qp\nperformance_schema/table_lock_waits_118.sdi.qp\nperformance_schema/variables_by_thr_192.sdi.qp\nperformance_schema/events_waits_sum_99.sdi.qp\nperformance_schema/events_statement_131.sdi.qp\nperformance_schema/accounts_154.sdi.qp\nperformance_schema/setup_objects_114.sdi.qp\nperformance_schema/global_status_190.sdi.qp\nperformance_schema/events_waits_sum_100.sdi.qp\nperformance_schema/events_errors_su_151.sdi.qp\nperformance_schema/rwlock_instances_110.sdi.qp\nperformance_schema/performance_time_108.sdi.qp\nperformance_schema/events_waits_sum_97.sdi.qp\nperformance_schema/user_variables_b_185.sdi.qp\nperformance_schema/events_stages_hi_121.sdi.qp\nperformance_schema/status_by_user_189.sdi.qp\nperformance_schema/events_waits_his_95.sdi.qp\nperformance_schema/events_statement_138.sdi.qp\nperformance_schema/events_transacti_141.sdi.qp\nperformance_schema/replication_appl_175.sdi.qp\nperformance_schema/memory_summary_b_163.sdi.qp\nperformance_schema/setup_instrument_113.sdi.qp\nperformance_schema/threads_119.sdi.qp\nmysql/general_log.CSV.qp\nmysql/general_log.CSM.qp\nmysql/slow_log.CSV.qp\nmysql/general_log_224.sdi.qp\nmysql/slow_log.CSM.qp\nmysql/slow_log_225.sdi.qp\nmysql-bin.000007.qp\nmysql-bin.index.qp\nxtrabackup_binlog_info.qp\nxtrabackup_logfile.qp\nxtrabackup_checkpoints\nib_buffer_pool.qp\nbackup-my.cnf.qp\nxtrabackup_info.qp\nxtrabackup_tablespaces.qp",
        "stderr_lines": [
            "ibdata1.qp",
            "sys/sys_config.ibd.qp",
            "book/source_words.ibd.qp",
            "book/million_words.ibd.qp",
            "mysql.ibd.qp",
            "undo_002.qp",
            "undo_001.qp",
            "performance_schema/binary_log_trans_198.sdi.qp",
            "performance_schema/user_defined_fun_197.sdi.qp",
            "performance_schema/socket_instances_156.sdi.qp",
            "performance_schema/events_stages_su_126.sdi.qp",
            "performance_schema/persisted_variab_196.sdi.qp",
            "performance_schema/users_153.sdi.qp",
            "performance_schema/events_stages_su_125.sdi.qp",
            "performance_schema/replication_grou_172.sdi.qp",
            "performance_schema/events_transacti_147.sdi.qp",
            "performance_schema/replication_grou_178.sdi.qp",
            "performance_schema/events_waits_sum_96.sdi.qp",
            "performance_schema/replication_conn_173.sdi.qp",
            "performance_schema/table_io_waits_s_117.sdi.qp",
            "performance_schema/keyring_componen_202.sdi.qp",
            "performance_schema/events_errors_su_150.sdi.qp",
            "performance_schema/mutex_instances_106.sdi.qp",
            "performance_schema/processlist_109.sdi.qp",
            "performance_schema/session_variable_194.sdi.qp",
            "performance_schema/metadata_locks_168.sdi.qp",
            "performance_schema/events_stages_cu_120.sdi.qp",
            "performance_schema/events_waits_cur_93.sdi.qp",
            "performance_schema/setup_consumers_112.sdi.qp",
            "performance_schema/file_summary_by__103.sdi.qp",
            "performance_schema/objects_summary__107.sdi.qp",
            "performance_schema/events_statement_134.sdi.qp",
            "performance_schema/socket_summary_b_158.sdi.qp",
            "performance_schema/events_stages_su_123.sdi.qp",
            "performance_schema/status_by_accoun_186.sdi.qp",
            "performance_schema/session_connect__159.sdi.qp",
            "performance_schema/events_transacti_140.sdi.qp",
            "performance_schema/events_statement_133.sdi.qp",
            "performance_schema/status_by_thread_188.sdi.qp",
            "performance_schema/events_statement_130.sdi.qp",
            "performance_schema/memory_summary_g_162.sdi.qp",
            "performance_schema/events_statement_136.sdi.qp",
            "performance_schema/variables_info_195.sdi.qp",
            "performance_schema/events_statement_128.sdi.qp",
            "performance_schema/replication_appl_180.sdi.qp",
            "performance_schema/replication_appl_176.sdi.qp",
            "performance_schema/events_transacti_143.sdi.qp",
            "performance_schema/events_stages_su_127.sdi.qp",
            "performance_schema/events_statement_129.sdi.qp",
            "performance_schema/events_errors_su_148.sdi.qp",
            "performance_schema/events_transacti_146.sdi.qp",
            "performance_schema/replication_appl_177.sdi.qp",
            "performance_schema/malloc_stats_201.sdi.qp",
            "performance_schema/keyring_keys_161.sdi.qp",
            "performance_schema/session_account__160.sdi.qp",
            "performance_schema/file_summary_by__104.sdi.qp",
            "performance_schema/data_locks_169.sdi.qp",
            "performance_schema/error_log_92.sdi.qp",
            "performance_schema/session_status_191.sdi.qp",
            "performance_schema/events_waits_sum_101.sdi.qp",
            "performance_schema/events_statement_132.sdi.qp",
            "performance_schema/log_status_183.sdi.qp",
            "performance_schema/replication_appl_174.sdi.qp",
            "performance_schema/events_transacti_145.sdi.qp",
            "performance_schema/replication_asyn_181.sdi.qp",
            "performance_schema/memory_summary_b_165.sdi.qp",
            "performance_schema/file_instances_102.sdi.qp",
            "performance_schema/malloc_stats_tot_200.sdi.qp",
            "performance_schema/replication_asyn_182.sdi.qp",
            "performance_schema/host_cache_105.sdi.qp",
            "performance_schema/data_lock_waits_170.sdi.qp",
            "performance_schema/events_stages_hi_122.sdi.qp",
            "performance_schema/tls_channel_stat_199.sdi.qp",
            "performance_schema/memory_summary_b_166.sdi.qp",
            "performance_schema/table_handles_167.sdi.qp",
            "performance_schema/replication_appl_179.sdi.qp",
            "performance_schema/setup_actors_111.sdi.qp",
            "performance_schema/events_transacti_142.sdi.qp",
            "performance_schema/prepared_stateme_184.sdi.qp",
            "performance_schema/table_io_waits_s_116.sdi.qp",
            "performance_schema/setup_threads_115.sdi.qp",
            "performance_schema/events_stages_su_124.sdi.qp",
            "performance_schema/events_statement_135.sdi.qp",
            "performance_schema/hosts_155.sdi.qp",
            "performance_schema/events_errors_su_149.sdi.qp",
            "performance_schema/events_statement_137.sdi.qp",
            "performance_schema/global_variables_193.sdi.qp",
            "performance_schema/replication_conn_171.sdi.qp",
            "performance_schema/events_transacti_144.sdi.qp",
            "performance_schema/cond_instances_91.sdi.qp",
            "performance_schema/memory_summary_b_164.sdi.qp",
            "performance_schema/events_waits_his_94.sdi.qp",
            "performance_schema/events_errors_su_152.sdi.qp",
            "performance_schema/events_waits_sum_98.sdi.qp",
            "performance_schema/status_by_host_187.sdi.qp",
            "performance_schema/events_statement_139.sdi.qp",
            "performance_schema/socket_summary_b_157.sdi.qp",
            "performance_schema/table_lock_waits_118.sdi.qp",
            "performance_schema/variables_by_thr_192.sdi.qp",
            "performance_schema/events_waits_sum_99.sdi.qp",
            "performance_schema/events_statement_131.sdi.qp",
            "performance_schema/accounts_154.sdi.qp",
            "performance_schema/setup_objects_114.sdi.qp",
            "performance_schema/global_status_190.sdi.qp",
            "performance_schema/events_waits_sum_100.sdi.qp",
            "performance_schema/events_errors_su_151.sdi.qp",
            "performance_schema/rwlock_instances_110.sdi.qp",
            "performance_schema/performance_time_108.sdi.qp",
            "performance_schema/events_waits_sum_97.sdi.qp",
            "performance_schema/user_variables_b_185.sdi.qp",
            "performance_schema/events_stages_hi_121.sdi.qp",
            "performance_schema/status_by_user_189.sdi.qp",
            "performance_schema/events_waits_his_95.sdi.qp",
            "performance_schema/events_statement_138.sdi.qp",
            "performance_schema/events_transacti_141.sdi.qp",
            "performance_schema/replication_appl_175.sdi.qp",
            "performance_schema/memory_summary_b_163.sdi.qp",
            "performance_schema/setup_instrument_113.sdi.qp",
            "performance_schema/threads_119.sdi.qp",
            "mysql/general_log.CSV.qp",
            "mysql/general_log.CSM.qp",
            "mysql/slow_log.CSV.qp",
            "mysql/general_log_224.sdi.qp",
            "mysql/slow_log.CSM.qp",
            "mysql/slow_log_225.sdi.qp",
            "mysql-bin.000007.qp",
            "mysql-bin.index.qp",
            "xtrabackup_binlog_info.qp",
            "xtrabackup_logfile.qp",
            "xtrabackup_checkpoints",
            "ib_buffer_pool.qp",
            "backup-my.cnf.qp",
            "xtrabackup_info.qp",
            "xtrabackup_tablespaces.qp"
        ],
        "stdout": "",
        "stdout_lines": []
    }
}

TASK [Extract qpress files | target] ******************************************************************************
changed: [replica]

TASK [Prepare the logs | target] **********************************************************************************
changed: [replica]

TASK [Change owner ship to mysql of directories under /db] ********************************************************
changed: [replica]

TASK [change owner ship to mysql of files under /db] ************************************************************
changed: [replica]

TASK [Remove auto.cnf from /var/lib/mysql dir on target] ********************************************************
ok: [replica]

TASK [Start MySQL on the target] ********************************************************************************
changed: [replica]

TASK [Wait for MySQL before proceeding on target] ***************************************************************
ok: [replica]

TASK [Cat xtrabackup_binlog file] *******************************************************************************
changed: [replica]

TASK [ansible.builtin.debug] ************************************************************************************
ok: [replica] => {
    "xtra1_file.stdout": "1d1fff5a-c9bc-11ed-9c19-02a36d996b94:1-4,c4709bcc-c9bb-11ed-8d19-02a36d996b94:1-33"
}

TASK [Purge gtids on the target] *****************************************************************************
changed: [replica]

TASK [Stop Replication] **************************************************************************************
changed: [replica]

TASK [Change primary to start replication on target] *********************************************************
changed: [replica]

TASK [Debug | Repl_stat3 | Replication Status] ***************************************************************
ok: [replica] => {
    "repl_stat3": {
        "changed": true,
        "failed": false,
        "failed_when_result": false,
        "queries": [
            "CHANGE MASTER TO MASTER_HOST='primary',MASTER_USER='repl',MASTER_PASSWORD='********',MASTER_AUTO_POSITION=1"
        ]
    }
}

TASK [Start Replication on target] ******************************************************************************
changed: [replica]

TASK [Check replication] ****************************************************************************************
ok: [replica]

TASK [Debug | Repl_stat2 | Replication status] ******************************************************************
ok: [replica] => {
    "repl_stat2": {
        "Auto_Position": 1,
        "Channel_Name": "",
        "Connect_Retry": 60,
        "Exec_Source_Log_Pos": 237,
        "Executed_Gtid_Set": "1d1fff5a-c9bc-11ed-9c19-02a36d996b94:1-4,\nc4709bcc-c9bb-11ed-8d19-02a36d996b94:1-33",
        "Get_Source_public_key": 0,
        "Is_Replica": true,
        "Last_Errno": 0,
        "Last_Error": "",
        "Last_IO_Errno": 0,
        "Last_IO_Error": "",
        "Last_IO_Error_Timestamp": "",
        "Last_SQL_Errno": 0,
        "Last_SQL_Error": "",
        "Last_SQL_Error_Timestamp": "",
        "Network_Namespace": "",
        "Read_Source_Log_Pos": 237,
        "Relay_Log_File": "mysql-relay-bin.000002",
        "Relay_Log_Pos": 373,
        "Relay_Log_Space": 583,
        "Relay_Source_Log_File": "mysql-bin.000007",
        "Replica_IO_Running": "Yes",
        "Replica_IO_State": "Waiting for source to send event",
        "Replica_SQL_Running": "Yes",
        "Replica_SQL_Running_State": "Replica has read all relay log; waiting for more updates",
        "Replicate_Do_DB": "",
        "Replicate_Do_Table": "",
        "Replicate_Ignore_DB": "",
        "Replicate_Ignore_Server_Ids": "",
        "Replicate_Ignore_Table": "",
        "Replicate_Rewrite_DB": "",
        "Replicate_Wild_Do_Table": "",
        "Replicate_Wild_Ignore_Table": "",
        "Retrieved_Gtid_Set": "",
        "SQL_Delay": 0,
        "SQL_Remaining_Delay": null,
        "Seconds_Behind_Source": 0,
        "Skip_Counter": 0,
        "Source_Bind": "",
        "Source_Host": "primary",
        "Source_Info_File": "mysql.slave_master_info",
        "Source_Log_File": "mysql-bin.000007",
        "Source_Port": 3306,
        "Source_Retry_Count": 86400,
        "Source_SSL_Allowed": "No",
        "Source_SSL_CA_File": "",
        "Source_SSL_CA_Path": "",
        "Source_SSL_Cert": "",
        "Source_SSL_Cipher": "",
        "Source_SSL_Crl": "",
        "Source_SSL_Crlpath": "",
        "Source_SSL_Key": "",
        "Source_SSL_Verify_Server_Cert": "No",
        "Source_Server_Id": 80152,
        "Source_TLS_Version": "",
        "Source_UUID": "c4709bcc-c9bb-11ed-8d19-02a36d996b94",
        "Source_User": "repl",
        "Source_public_key_path": "",
        "Until_Condition": "None",
        "Until_Log_File": "",
        "Until_Log_Pos": 0,
        "changed": false,
        "failed": false,
        "queries": []
    }
}

TASK [Debug | Repl_stat2 | Source_Host] ***************************************************************************
ok: [replica] => {
    "(repl_stat2.Source_Host,repl_stat2.Replica_IO_Running,repl_stat2.Replica_SQL_Running,repl_stat2.Executed_Gtid_Set)": "('primary', 'Yes', 'Yes', '1d1fff5a-c9bc-11ed-9c19-02a36d996b94:1-4,\\nc4709bcc-c9bb-11ed-8d19-02a36d996b94:1-33')"
}

TASK [include_tasks] **********************************************************************************************
included: /data-sync/cleanup.yml for replica

TASK [Target | Clean up the backup directory on Target] ***********************************************************
changed: [replica]

TASK [Source | Clean up the backup directory on the Primary] ******************************************************
changed: [replica -> primary(10.5.0.152)]

PLAY RECAP ********************************************************************************************************
127.0.0.1                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
primary                    : ok=13   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=2
replica                    : ok=28   changed=15   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


```