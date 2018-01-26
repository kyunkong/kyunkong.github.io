---
title: "DB2 tablespace status"
date: 2018-01-16 17:25:42
---
### DB2 tablespace status
```
0x0                 Normal                                   
0x1                 Quiesced: SHARE                          
0x2                 Quiesced: UPDATE                        
0x4                 Quiesced: EXCLUSIVE                     
0x8                 Load pending                             
0x10               Delete pending                           
0x20               Backup pending                           
0x40               Roll forward in progress                 
0x80               Roll forward pending                     
0x100             Restore pending                          
0x100             Recovery pending (not used)              
0x200             Disable pending                          
0x400             Reorg in progress                        
0x800             Backup in progress                       
0x1000           Storage must be defined                  
0x2000           Restore in progress                     
0x4000           Offline and not accessible               
0x8000           Drop pending                             
0x2000000     Storage may be defined                  
0x4000000     StorDef is in 'final' state              
0x8000000     StorDef was changed prior to rollforward
0x10000000   DMS rebalancer is active                 
0x20000000   TBS deletion in progress                 
0x40000000   TBS creation in progress                 
0x8                 For service use only
```

###how to find in DB2
```
[db2v10i@hadr02 ~]$ db2tbst 0x8000
State = Drop pending
```


