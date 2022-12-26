DECLARE
	v_sql        VARCHAR2(4000);
	v_tabla      VARCHAR2(50);
	v_tam        VARCHAR2(50);
	v_nulo       VARCHAR2(10);
	v_comma      VARCHAR2(1);
	i            NUMBER;
	v_num_cols   NUMBER;
	v_len        NUMBER;
	v_existe_pri NUMBER;
	v_existe_uni NUMBER;
	v_existe_for NUMBER;
BEGIN
	v_tabla := upper('tb_ad_nombres_listas');
	i       := 0;

	BEGIN
		SELECT COUNT(*)
			INTO v_num_cols
			FROM all_tab_cols tc
		 WHERE tc.table_name = v_tabla;
	
	EXCEPTION
		WHEN OTHERS THEN
			NULL;
	END;

	BEGIN
		SELECT MAX(length(tc.column_name))
			INTO v_len
			FROM all_tab_cols tc
		 WHERE tc.table_name = v_tabla;
	
	EXCEPTION
		WHEN OTHERS THEN
			NULL;
	END;

	dbms_output.put_line('Número de columnas: ' || v_num_cols || chr(13));

	v_sql := 'create table ' || lower(v_tabla) || ' (';
	dbms_output.put_line(v_sql);

	FOR x IN (SELECT tc.column_name
									,tc.data_type
									,tc.data_length
									,tc.data_precision
									,tc.data_scale
									,tc.nullable
							FROM all_tab_cols tc
						 WHERE tc.owner = USER
							 AND tc.table_name = v_tabla
						 ORDER BY tc.table_name
										 ,tc.internal_column_id) LOOP
		IF i = v_num_cols - 1 THEN
			v_comma := '';
		ELSE
			v_comma := ',';
		END IF;
		i := i + 1;
		IF x.data_type = 'NUMBER' THEN
			IF x.data_precision IS NOT NULL THEN
				IF x.data_scale <> 0 THEN
					v_tam := '(' || x.data_precision || ',' || x.data_scale || ')';
				ELSE
					v_tam := '(' || x.data_precision || ')';
				END IF;
			ELSE
				v_tam := '';
			END IF;
		ELSIF x.data_type = 'DATE'
					OR x.data_type = 'BLOB'
					OR x.data_type = 'CLOB' THEN
			v_tam := '';
		ELSE
			v_tam := '(' || x.data_length || ')';
		END IF;
		IF x.nullable = 'Y' THEN
			v_nulo := '';
		ELSE
			v_nulo := ' not null';
		END IF;
		v_sql := chr(9) || lower(x.column_name) ||
						 lpad(' '
								 ,(v_len + 2) - length(x.column_name)) ||
						 lower(x.data_type) || v_tam || v_nulo || v_comma;
		dbms_output.put_line(v_sql);
	END LOOP;
	v_sql := ');';
	dbms_output.put_line(v_sql);

	BEGIN
		SELECT 1
			INTO v_existe_pri
			FROM all_constraints c
		 WHERE c.owner = USER
			 AND c.constraint_type = 'P'
			 AND c.table_name = v_tabla;
	EXCEPTION
		WHEN OTHERS THEN
			NULL;
	END;

	BEGIN
		SELECT 1
			INTO v_existe_uni
			FROM all_constraints c
		 WHERE c.owner = USER
			 AND c.constraint_type = 'U'
			 AND c.table_name = v_tabla;
	EXCEPTION
		WHEN OTHERS THEN
			NULL;
	END;

	BEGIN
		SELECT 1
			INTO v_existe_for
			FROM all_constraints c
		 WHERE c.owner = USER
			 AND c.constraint_type = 'R'
			 AND c.table_name = v_tabla;
	EXCEPTION
		WHEN OTHERS THEN
			NULL;
	END;

	IF v_existe_pri = 1 THEN
		FOR x IN (SELECT c.constraint_name
										,listagg(cc.column_name
														,',') within GROUP(ORDER BY cc.position) column_name
								FROM all_constraints c
							 INNER JOIN all_cons_columns cc
									ON c.constraint_name = cc.constraint_name
							 WHERE c.owner = USER
								 AND c.constraint_type = 'P'
								 AND c.table_name = v_tabla
							 GROUP BY c.constraint_name) LOOP
			v_sql := 'alter table ' || lower(v_tabla) || ' ' || 'add constraint ' ||
							 x.constraint_name || ' primary key (' ||
							 lower(x.column_name) || ');';
			dbms_output.put_line(v_sql);
		END LOOP;
	END IF;

	IF v_existe_uni = 1 THEN
		FOR x IN (SELECT c.constraint_name
										,listagg(cc.column_name
														,',') within GROUP(ORDER BY cc.position) column_name
								FROM all_constraints c
							 INNER JOIN all_cons_columns cc
									ON c.constraint_name = cc.constraint_name
							 WHERE c.owner = USER
								 AND c.constraint_type = 'U'
								 AND c.table_name = v_tabla
							 GROUP BY c.constraint_name) LOOP
			v_sql := 'alter table ' || lower(v_tabla) || ' ' || 'add constraint ' ||
							 x.constraint_name || ' unique (' || lower(x.column_name) || ');';
			dbms_output.put_line(v_sql);
		END LOOP;
	END IF;

	IF v_existe_for = 1 THEN
		FOR x IN (SELECT c.constraint_name
										,listagg(cc.column_name
														,',') within GROUP(ORDER BY cc.position) column_name
										,(SELECT c1.table_name
												FROM all_constraints c1
											 WHERE c1.constraint_name = c.r_constraint_name) ref_table_name
										,(SELECT cc1.column_name
												FROM all_cons_columns cc1
											 INNER JOIN all_constraints c2
													ON cc1.constraint_name = c2.constraint_name
											 WHERE c2.constraint_name = c.r_constraint_name) ref_column_name
								FROM all_constraints c
							 INNER JOIN all_cons_columns cc
									ON c.constraint_name = cc.constraint_name
							 WHERE c.owner = USER
								 AND c.constraint_type = 'R'
								 AND c.table_name = v_tabla
							 GROUP BY c.constraint_name
											 ,c.r_constraint_name
											 ,cc.position
											 ,c.table_name
							 ORDER BY cc.position) LOOP
			v_sql := 'alter table ' || lower(v_tabla) || ' ' || 'add constraint ' ||
							 x.constraint_name || ' foreign key (' ||
							 lower(x.column_name) || ') references ' ||
							 lower(x.ref_table_name) || '(' || lower(x.ref_column_name) || ');';
			dbms_output.put_line(v_sql);
		END LOOP;
	END IF;

END;
/
