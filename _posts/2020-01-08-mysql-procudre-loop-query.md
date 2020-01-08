---
title: MySQL使用存储过程进行循环查询
tags: 技术 数据库 MySQL 存储过程
--- 

最近年底PMO在度量项目的时候，需要查询某个项目ID对应的计数。但是一个格子会填几个ID

类似这样 表： project_issue

|  id  | project |
| ---- | ------- |
|   1  |  a,b    |
|   2  |  b,c    |
|   3  |  c,d,e  |

当统计某一个项目有多少条记录的时候，count(1) where project like ''， 但是PMO希望得到一个项目分组，a=1， b=2, c=2…… 这样的结果，这又不能单纯group by project。所以最后还是借用了临时表和存储过程，实现了循环查询。

<!--more-->

完整过程如下:

创建一个临时表，或者正式表也可以，用来存储待遍历的项目ID，和输出结果。

    DROP TABLE IF EXISTS project_list;
    CREATE TEMPORARY TABLE project_list (
        project_name VARCHAR(50) NOT NULL,
        project_count INT default 0
    );
    INSERT INTO project_list value ("a",0);
    INSERT INTO project_list value ("b",0);

使用存储过程，遍历project里面的project_name，然后执行统计查询：

    DROP PROCEDURE IF EXISTS proj_statis_procudre;
    DELIMITER $$ -- 修改换号符号为 $$ 符号，避免分号时被执行
    CREATE PROCEDURE proj_statis_procudre()
    BEGIN
        DECLARE done INT DEFAULT false; -- 声明一个结束标记变量
        DECLARE proj_name VARCHAR(50); -- 项目名称，临时变量
        DECLARE proj_cursor CURSOR FOR SELECT project_name FROM project_list; -- 项目名称表的游标
        DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE; -- 结束时设置变量done=True

        OPEN proj_cursor; -- 开启游标
        proj_loop:LOOP -- 使用循环
            FETCH proj_cursor INTO proj_name; -- 从游标获取一个项目名称
            IF NOT done THEN
                UPDATE project_list SET project_count = (SELECT count(1) FROM project_issue WHERE project REGEXP proj_name) where project_name=proj_name; -- 执行查询，并更新结果到临时表
            ELSE
                LEAVE proj_loop;
            END IF;
        END LOOP;

        CLOSE proj_cursor; -- 关闭游标
        select * from project_list; -- 输出临时表的最终结果
    END;
    $$

    DELIMITER ; -- 将分隔符重置为分号;

执行存储过程， 更新项目统计表

    call proj_statis_procudre();