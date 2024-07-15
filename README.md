# monitor_temp_files

# function 구문
DROP function monitor_temp_files();
CREATE OR REPLACE FUNCTION monitor_temp_files()
RETURNS TABLE (
    pid int,
    datname text,
    usename text,
    query text,
    temp_files int,
    temp_Mbytes bigint,
    io_timings double precision
) AS $$
BEGIN
    RETURN QUERY
    SELECT
        a.pid,
        a.datname::text,
        a.usename::text,
        a.query::text,
        d.temp_files::int,
        d.temp_bytes/1024 AS temp_Mbytes,
        d.blk_read_time + d.blk_write_time AS io_timings
    FROM
        pg_stat_activity a
    JOIN
        pg_stat_database d ON a.datid = d.datid
    WHERE
        d.temp_files > 0
        ORDER BY temp_Mbytes desc;
END;
$$ LANGUAGE plpgsql;

# 사용방법
select * from monitor_temp_files();

#데이터 베이스 전체에서 사용하는 tempfile 갯수와 사이즈
SELECT temp_files AS "Temporary files count",
temp_bytes/1024 AS "Size of temporary files(MB)"
FROM   pg_stat_database db WHERE datname = 'DB_NAME';

#임시 파일 경로
/data/master/gpseg-1/base/pgsql_tmp

# 해당 function 모니터링을 통하여 분석 할 내용
 1) 특정쿼리가 임시파일을 많이 사용하는지 아니면 특정 경우에 동시에 여러 트랜젝션이 많이 유입되어 처리하는데 임시파일을 대량으로 사용는지 확인 필요
 2) work_mem size 와 리소스 그룹에서 설정한 Concurrency 갯수가 적절한 값인지 확인 필요


