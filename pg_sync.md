# PG主从配置

## 主库

1.创建角色psql

	create role replica login replication encrypted password 'replica';
				<用户名> 		<固定>						   <密码>

2.更改pg_hba.conf

	host    all             all             主库IP/32      trust
	host    all             all             从库IP/32      trust
	host    replication     replica         从库IP/32      md5

3.更改postgresql.conf

	# 监听端口
	listen_addresses = '*'

	# 监听端口
	port = 8432  

	# 连接数
	max_connections = 2000

	# 归档模式
	archive_mode = on  

	# 归档命令
	archive_command = 'cp %p /home/records/pgdata/pg_archive/%f'

	# 决定多少信息写入WAL，此处为replica模式
	wal_level = replica 

	# 最大流复制连接
	max_wal_senders = 40

	# 
	wal_keep_segments = 64s

	# 流复制超时时间
	wal_sender_timeout = 60

# 从库

1.切换用户

	su postgres

2.删除数据库目录

	rm -rf /home/records/pgdata/* 

3.复制主库

	pg_basebackup -h 192.168.1.230 -p 8432 -U replica -P -X stream -D /home/records/pgdata/	

	输入replica的密码
	replica

4.添加recovery.conf配置

	standby_mode = on
	primary_conninfo = 'host=主库IP port=主库端口 user=replica password=replica'
	recovery_target_timeline = 'latest'
