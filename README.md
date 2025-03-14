# profile2025

FECHA=`date +%d%m%y%H%M`; export FECHA
LOG=`pwd`; export LOG

$ORACLE_HOME/bin/sqlplus -S / as sysdba << EOF! > ${LOG}/PROFILE_${ORACLE_SID}_${FECHA}.log

spool Auditoria.txt
set echo on
set linesize 300
set pagesize 300
col PROFILE for a30
col version form a40
col description form a85
col action_date form a20




prompt ---database---

SELECT sys_context('USERENV','DB_NAME') AS Database_Name FROM dual;


select distinct profile from dba_profiles ;



prompt ---Create profile APPUSER

SET SERVEROUTPUT ON;
DECLARE
    ddl_qry     VARCHAR2 (1000);
BEGIN
    ddl_qry := 'CREATE PROFILE APPUSER
    LIMIT
         COMPOSITE_LIMIT UNLIMITED
         SESSIONS_PER_USER UNLIMITED
         CPU_PER_SESSION UNLIMITED
         CPU_PER_CALL UNLIMITED
         LOGICAL_READS_PER_SESSION UNLIMITED
         LOGICAL_READS_PER_CALL UNLIMITED
         IDLE_TIME UNLIMITED
         CONNECT_TIME UNLIMITED
         PRIVATE_SGA UNLIMITED
         FAILED_LOGIN_ATTEMPTS UNLIMITED
         PASSWORD_LIFE_TIME UNLIMITED
         PASSWORD_REUSE_TIME UNLIMITED
         PASSWORD_REUSE_MAX UNLIMITED
         PASSWORD_VERIFY_FUNCTION NULL
         PASSWORD_LOCK_TIME UNLIMITED
         PASSWORD_GRACE_TIME UNLIMITED';
    EXECUTE IMMEDIATE ddl_qry;
EXCEPTION
              WHEN OTHERS THEN
                 null;
end;
/



prompt ---Create profile DBAUSERS


SET SERVEROUTPUT ON;
DECLARE
    ddl_qry     VARCHAR2 (1000);
BEGIN
    ddl_qry := 'CREATE PROFILE DBAUSERS
    LIMIT
         PASSWORD_ROLLOVER_TIME		DEFAULT
         PASSWORD_REUSE_TIME		120
         FAILED_LOGIN_ATTEMPTS		3
         CPU_PER_SESSION		UNLIMITED
         CONNECT_TIME		UNLIMITED
         PASSWORD_LIFE_TIME		90
         PASSWORD_GRACE_TIME		5
         IDLE_TIME		UNLIMITED
         PASSWORD_LOCK_TIME		5
         CPU_PER_CALL		UNLIMITED
         INACTIVE_ACCOUNT_TIME		60
         PASSWORD_VERIFY_FUNCTION		ADMUSERS_VERIFYFUNCTION
         LOGICAL_READS_PER_SESSION		UNLIMITED
         SESSIONS_PER_USER		25
         PASSWORD_REUSE_MAX		20
         LOGICAL_READS_PER_CALL		UNLIMITED
         COMPOSITE_LIMIT		UNLIMITED
         PRIVATE_SGA		UNLIMITED';
    EXECUTE IMMEDIATE ddl_qry;
EXCEPTION
              WHEN OTHERS THEN
                 null;
end;
/

prompt ---Create profile USR_CONSULTA


SET SERVEROUTPUT ON;
DECLARE
    ddl_qry     VARCHAR2 (1000);
BEGIN
    ddl_qry := 'CREATE PROFILE USR_CONSULTA
    LIMIT
         PASSWORD_ROLLOVER_TIME		DEFAULT
         PASSWORD_REUSE_TIME		120
         FAILED_LOGIN_ATTEMPTS		3
         CPU_PER_SESSION		UNLIMITED
         CONNECT_TIME		UNLIMITED
         PASSWORD_LIFE_TIME		UNLIMITED
         PASSWORD_GRACE_TIME		5
         IDLE_TIME		UNLIMITED
         PASSWORD_LOCK_TIME		5
         CPU_PER_CALL		UNLIMITED
         INACTIVE_ACCOUNT_TIME		90
         PASSWORD_VERIFY_FUNCTION		ADMUSERS_VERIFYFUNCTION
         LOGICAL_READS_PER_SESSION		UNLIMITED
         SESSIONS_PER_USER		10
         PASSWORD_REUSE_MAX		20
         LOGICAL_READS_PER_CALL		UNLIMITED
         COMPOSITE_LIMIT		UNLIMITED
         PRIVATE_SGA		UNLIMITED';
    EXECUTE IMMEDIATE ddl_qry;
EXCEPTION
              WHEN OTHERS THEN
                 null;
end;
/


prompt ---creacion de profile APPUSER y asignacion de usuarios.



set linesize 300
col username for a20
SELECT username, profile
FROM dba_users
WHERE username NOT LIKE '%USR%'
AND username NOT LIKE '%CC%'
AND username NOT IN ('RMANADMIN', 'INF_MONTR', 'DBA_SOPRT','DBA_SOPN2','DBA_SOPCC') AND  ORACLE_MAINTAINED  != 'Y';



CREATE OR REPLACE PROCEDURE SYS.PROFILE_USERS_APPUSER
IS
    TYPE user_profile_rec IS RECORD (
        username VARCHAR2(128),
        profile  VARCHAR2(128) 
    );
    v_cursor sys_refcursor := NULL;
    v_rec    user_profile_rec;
BEGIN
    OPEN v_cursor FOR
        SELECT username, profile
     FROM dba_users
     WHERE username NOT LIKE '%USR%'
     AND username NOT LIKE '%CC%'
     AND username NOT IN ('RMANADMIN', 'INF_MONTR', 'DBA_SOPRT','DBA_SOPN2','DBA_SOPCC') AND  ORACLE_MAINTAINED  != 'Y';
    LOOP
        FETCH v_cursor INTO v_rec;
        EXIT WHEN v_cursor%NOTFOUND;
        EXECUTE IMMEDIATE 'ALTER USER ' || v_rec.username || ' PROFILE APPUSER';
    END LOOP;
    CLOSE v_cursor;
END;
/


exec SYS.PROFILE_USERS_APPUSER;

prompt ---listado de usuarios aplicativos.

set linesize 300
col username for a20
SELECT username, profile
FROM dba_users
WHERE username NOT LIKE '%USR%'
AND username NOT LIKE '%CC%'
AND username NOT IN ('RMANADMIN', 'INF_MONTR', 'DBA_SOPRT','DBA_SOPN2','DBA_SOPCC') AND  ORACLE_MAINTAINED  != 'Y';




