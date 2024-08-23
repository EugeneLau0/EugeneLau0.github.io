如下SQL所示，先将有问题的表id找出来，id跟entryid的关系是一对多，下面SQL解决的是同一个id下按照entryid进行seq从1到N的修改。

```sql
do
$BODY$
declare
	fenteryid_cursor refcursor;
	v_fid bigint;
	v_fentryid bigint;
	v_seq int;
fid_cursor cursor for
	-- 找出有问题的id：最大的seq不等于总的分录数时
	select id from (
		select tio.fid as id, max(fseq) as maxSeq, count(fentryid) as entryCount
		from t_target_table tio
		group by tio.fid ) temp
	where maxSeq != entryCount ;

begin 
	-- 开启id游标
	open fid_cursor;
	loop 
		fetch fid_cursor into v_fid;
	exit when not found;
		-- 开启分录游标
		open fenteryid_cursor for
			select fentryid from t_target_table where fid = v_fid order by fentryid asc, fmaterialid ;
			v_seq := 1;
		loop 
			fetch fenteryid_cursor into v_fentryid;
		if found then 
			-- 将seq+1
			update t_target_table set fseq = v_seq where fid = v_fid and fentryid = v_fentryid;
 			v_seq := v_seq + 1;
		else exit;
		end if;
		end loop;
		close fenteryid_cursor ;
	end loop;
	close fid_cursor;
end;

$BODY$ language plpgsql;

```
