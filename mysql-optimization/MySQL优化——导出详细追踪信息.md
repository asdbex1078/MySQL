1. 使用命令行窗口

2. 开启追踪参数，只对当前会话有效，对其他会话无效。(可能会遇到权限问题)
   	SET OPTIMIZER_TRACE="enabled=on",END_MARKERS_IN_JSON=on; 
   	SET OPTIMIZER_TRACE_MAX_MEM_SIZE=1000000; 
   	
3. 执行SQL语句 （支持 SELECT/INSERT/REPLACE/UPDATE/DELETE/EXPLAIN/DECLARE/CASE/IF/RETURN）

4. 查看追踪信息
   	SELECT * FROM INFORMATION_SCHEMA.OPTIMIZER_TRACE;
   	
5. 输出追踪信息到文件
   	SELECT TRACE INTO DUMPFILE "/var/lib/mysql-files//test.trace" FROM INFORMATION_SCHEMA.OPTIMIZER_TRACE;
   	(注意：此处需要是 / 并且，输出的目录需要留心)

	①、可能会遇到如下问题：
		mysql> SELECT TRACE INTO DUMPFILE "C:/ProgramData/MySQL/MySQL Server 5.7/Uploads/test.trace" FROM INFORMATION_SCHEMA.OPTIMIZER_TRACE;
		ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
	   解决方案：
			首先查看 secure_file_priv 值
			mysql> show variables like '%secure%';
			+--------------------------+------------------------------------------------+
			| Variable_name            | Value                                          |
			+--------------------------+------------------------------------------------+
			| require_secure_transport | OFF                                            |
			| secure_auth              | ON                                             |
			| secure_file_priv         | C:\ProgramData\MySQL\MySQL Server 5.7\Uploads\ |
			+--------------------------+------------------------------------------------+
			
			要么，导出文件到 secure_file_priv 下；
			要么，修改 secure_file_priv 该值，并重启MySQL服务。
