例如：user1表：
	CREATE TABLE `user1` (
	  `user_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键id',
	  `user_name` varchar(100) DEFAULT NULL COMMENT '用户名',
	  `user_age` tinyint(3) DEFAULT NULL COMMENT '用户年龄',
	  `user_password` varchar(100) DEFAULT NULL COMMENT '用户密码',
	  `user_sex` tinyint(1) DEFAULT NULL COMMENT '性别 1-男，0-女',
	  `user_province` varchar(32) DEFAULT NULL COMMENT '用户所在省',
	  `user_city` varchar(32) DEFAULT NULL COMMENT '用户所在城市',
	  `user_area` varchar(32) DEFAULT NULL COMMENT '用户所在区',
	  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
	  `modified_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
	  PRIMARY KEY (`user_id`),
	  KEY `idx_user_name` (`user_name`),
	  KEY `idx_user_age` (`user_age`),
	  KEY `idx_province_city_area` (`user_province`,`user_city`,`user_area`),
	  KEY `idx_sex_province_city_area` (`user_sex`,`user_province`,`user_city`,`user_area`),
	  KEY `idx_sex` (`user_sex`)
	) ENGINE=InnoDB AUTO_INCREMENT=223793 DEFAULT CHARSET=utf8;
		
测试sql：
	explain select * from user1 where user_province = '山西省' and user_city like '%原%'  and create_time = '2020-08-18 15:45:08';
	
表中数据摘要：
	+---------+-----------+----------+----------------------------------+----------+---------------+-----------+-----------+---------------------+---------------------+
	| user_id | user_name | user_age | user_password                    | user_sex | user_province | user_city | user_area | create_time         | modified_time       |
	+---------+-----------+----------+----------------------------------+----------+---------------+-----------+-----------+---------------------+---------------------+
	|   13708 | 席此郎    |       68 | 20bba5cce730401d96f054843de430f0 |        1 | 陕西省        | 阳江市    | 伊春区    | 2020-08-18 15:45:08 | 2020-08-18 15:45:08 |
	|   13709 | 华敦似    |       88 | 26e17d356a6c4276bf93ba6e07a40b31 |        1 | 辽宁省        | 东莞市    | 五营区    | 2020-08-18 15:45:08 | 2020-08-18 15:45:08 |
	|   27679 | 韦直竞    |       69 | fffa223160e24e93bc67c993b5885389 |        1 | 山西省        | 太原市    | 上甘岭区  | 2020-08-18 15:45:08 | 2020-08-18 15:45:08 |
	|   38374 | 盛组师    |        0 | ab94a53a6a834d6f82aaabcd4bed1a56 |        1 | 山西省        | 太原市    | 上甘岭区  | 2020-08-18 15:45:08 | 2020-08-18 15:45:08 |
	|   67703 | 洪杳荐    |       46 | 824fea10883c4e0db700fd66fd80d575 |        1 | 山西省        | 抚州市    | 乌马河区  | 2020-08-18 15:45:18 | 2020-08-18 15:45:18 |
	|   69102 | 杭雅席    |       49 | d65fdda9fd04459c8c6c725c76bdaeee |        0 | 山西省        | 松原市    | 美溪区    | 2020-08-18 15:45:18 | 2020-08-18 15:45:18 |
	|   75910 | 韦侵气    |       75 | 6329691ee6b141a9a42cf85aa74726e3 |        1 | 山西省        | 安顺市    | 上甘岭区  | 2020-08-18 15:45:28 | 2020-08-18 15:45:28 |
	|   92244 | 胡家赞    |       44 | 2eddaea405dd46df800031f117edbbb3 |        1 | 山西省        | 固原市    | 伊春区    | 2020-08-18 15:45:28 | 2020-08-18 15:45:28 |
	|  121410 | 项师土    |       30 | 341306b97c2f42bdbb934574c4de1b3a |        0 | 山西省        | 天水市    | 金山屯区  | 2020-08-18 15:45:44 | 2020-08-18 15:45:44 |
	+---------+-----------+----------+----------------------------------+----------+---------------+-----------+-----------+---------------------+---------------------+
	
MySQL5.6中：
	mysql> explain select * from user1 where user_province = '山西省' and user_city like '%原%'  and create_time = '2020-08-18 15:45:08';
	+----+-------------+-------+------------+------+------------------------+------------------------+---------+-------+-------+-----------------------+
	| id | select_type | table | partitions | type | possible_keys          | key                    | key_len | ref   | rows  | Extra                 |
	+----+-------------+-------+------------+------+------------------------+------------------------+---------+-------+-------+-----------------------+
	|  1 | SIMPLE      | user1 | NULL       | ref  | idx_province_city_area | idx_province_city_area | 99      | const | 17572 | Using index condition |
	+----+-------------+-------+------------+------+------------------------+------------------------+---------+-------+-------+-----------------------+
	1 row in set, 1 warning (0.00 sec)

MySQL5.5中：
	mysql> explain select * from user1 where user_province = '山西省' and user_city like '%原%' and create_time = '2020-08-18 15:45:08';
	+----+-------------+-------+------------+------+------------------------+------------------------+---------+-------+-------+-----------------------+
	| id | select_type | table | partitions | type | possible_keys          | key                    | key_len | ref   | rows  | Extra                 |
	+----+-------------+-------+------------+------+------------------------+------------------------+---------+-------+-------+-----------------------+
	|  1 | SIMPLE      | user1 | NULL       | ref  | idx_province_city_area | idx_province_city_area | 99      | const | 16400 | Using where           |
	+----+-------------+-------+------------+------+------------------------+------------------------+---------+-------+-------+-----------------------+
	1 row in set, 1 warning (0.00 sec)
	
解释：
	没有ICP：MySQL可以使用索引扫描省份 user_province = '山西省'。第二部分（user_city like '%原%'）不能用于限制必须扫描的行数，因为他不能走索引。
	因此，此查询必须为所有具有的"山西省"检索完整的表行。即把所有"山西省"的数据推送到MySQL服务器，然后MySQL服务器过滤出 user_city like '%原%'的数据.（Using where）

		(
			1、在索引 idx_province_city_area 中(非聚簇索引)，找到所有山西省的主键 user_id： 27679、38374、67703、69102、75910、92244、121410。
			2、循环遍历主键 user_id 去主表中，查出对应的完整数据行。(非聚簇索引查询数据的过程)。每获取到的一行完整数据，推送到MySQL服务器，
				让服务器层过滤 user_city 和 其他条件，决定保留行还是放弃行。
			3、重复2直到 一开始找到的所有山西省的数据过滤完。
		)


	有ICP：首先根据索引查到所有 user_province = '山西省'，然后虽然 user_city 不走索引，但是索引中包含了该值，完全可以过滤出包含 “原”的数据。
			此过程，过滤掉了没有"原"的数据。推送到服务器的数据行更少，更符合条件。(Using index condition)
			
		(
			1、在索引 idx_province_city_area 中(非聚簇索引)，找到所有山西省的主键 user_id：27679、38374、67703、69102、75910、92244、121410。
			2、在索引中过滤，去掉 user_city 中，不满足 like '%原%' 的数据 主键 user_id，此时，主键 user_id 剩下 27679、38374、69102、92244
			3、循环遍历满足条件的主键id 去主表中，查出对应的完整数据行。每获取到一行完整数据，就推送到MySQL服务器，服务器再过滤其他条件。
			4、重复3
		)
