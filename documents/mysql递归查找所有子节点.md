	CREATE FUNCTION `getChildLst`(userCode VARCHAR(20)) RETURNS varchar(200) CHARSET utf8
	    READS SQL DATA
	BEGIN
	
	  DECLARE tempCode VARCHAR(200);
	  DECLARE tempChildCodes VARCHAR(200);
	
	  SET tempCode = '';
	  SET tempChildCodes =cast(userCode as CHAR);
	
	  WHILE tempChildCodes is not null DO
	
	    SET tempCode = concat(tempCode,',',tempChildCodes);
	
			select GROUP_CONCAT(user_code) into tempChildCodes from t_opp_user_relation where FIND_IN_SET(parent_code,tempChildCodes) > 0;
	
	  END WHILE;
	
	  RETURN tempCode;
	
	END



	select * from t_opp_user_relation where find_in_set(parent_code, getChildLst('7'));




