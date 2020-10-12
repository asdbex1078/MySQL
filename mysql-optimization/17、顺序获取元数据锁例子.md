例如，RENAME TABLE是一个DDL语句，它按照名称顺序获取锁:

此重命名表语句将tbla重命名为其他名称，并将tblc重命名为tbla:

	RENAME TABLE tbla TO tbld, tblc TO tbla;
	该语句按顺序获取tbla、tblc和tbld上的元数据锁(因为tbld按照名称顺序，在tblc之后)

	RENAME TABLE tbla TO tblb, tblc TO tbla;
	语句按顺序获取tbla、tblb和tblc上的元数据锁(因为按照名称顺序，tblb在tblc之前)
	
这两条语句都按照这个顺序获得了tbla和tblc上的锁，但是不同之处在于其余表名是在tblc之前还是之后，导致获取锁的顺序不同。