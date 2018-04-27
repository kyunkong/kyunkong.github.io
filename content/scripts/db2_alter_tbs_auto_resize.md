---
title: "Modify DB2 tablespace to auto resize"
date: 2018-01-16 16:37:24
---

```
-- file name: Alter_Auto_Resize_TBS.ksh
-- Platform: AIX platform:
db2 list db directory|grep -p Indirect|grep "Database alias"|cut -d "=" -f 2| while read cur_db_name; do
   echo "-- DB Name: $cur_db_name --"
   db2pd -db $cur_db_name -tablespaces | grep -p "Autoresize" |grep "No No "|cut -d " " -f 1 |while read cur_addr; do
      cur_line=`db2pd -db $cur_db_name -tablespaces | grep -p "Tablespace Configuration" |grep $cur_addr`
      flg_dms=`echo $cur_line |grep "DMS "|grep -v grep |wc -l`
      if [[ $flg_dms -eq "1" ]]; then
         trg_tbnm=`echo $cur_line|cut -d " " -f 15`
         echo "alter tablespace $trg_tbnm autoresize yes"
      fi
   done
done
```

```
-- Platform: linux platform:
db2 list db directory|grep -B 5 Indirect|grep "Database alias"|cut -d "=" -f 2| while read cur_db_name; do
   echo "-- DB Name: $cur_db_name --"
   cur_sections=`db2pd -db $cur_db_name -tablespaces | egrep -n -e "Tablespace .*:|Containers:"`
   cur_tbsp_cfg=`echo $cur_sections|cut -d ":" -f 1`
   cur_tbsp_sts=`echo $cur_sections|cut -d ":" -f 3`
   cur_tbsp_ars=`echo $cur_sections|cut -d ":" -f 5`
   cur_contnr=`echo $cur_sections|cut -d ":" -f 7`
   ((cur_ts_ars_lns=$cur_contnr - $cur_tbsp_ars))
   ((cur_ts_cfg_lns=$cur_tbsp_sts - cur_tbsp_cfg))
   db2pd -db $cur_db_name -tablespaces | head -n $cur_contnr | tail -n $cur_ts_ars_lns |grep "No No "|cut -d " " -f 1 |while read cur_addr; do
      cur_line=`db2pd -db $cur_db_name -tablespaces | head -n $cur_tbsp_sts | tail -n $cur_ts_cfg_lns |grep $cur_addr`
      flg_dms=`echo $cur_line |grep "DMS "|grep -v grep |wc -l`
      if [[ $flg_dms -eq "1" ]]; then
         trg_tbnm=`echo $cur_line|cut -d " " -f 15`
         echo "alter tablespace $trg_tbnm autoresize yes"
      fi
   done
done
```



