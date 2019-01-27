# utl-last-assay-date-prior-to-exposure-date
    Last assay date prior to exposure date                                                                                             
                                                                                                                                       
    This is a common topic, good to have mutilple solutions (sql may ne the slowest?)                                                  
                                                                                                                                       
       Three Solutions                                                                                                                 
                                                                                                                                       
             1. Proc Sql                                                                                                               
             2. HASH Paul Dorfman                                                                                                      
                sashole@bellsouth.net                                                                                                  
             3. DOW loop                                                                                                               
                                                                                                                                       
    github                                                                                                                             
    https://tinyurl.com/ycy5gxvx                                                                                                       
    https://github.com/rogerjdeangelis/utl-last-assay-date-prior-to-exposure-date-clinical                                             
                                                                                                                                       
    SAS Forum                                                                                                                          
    https://tinyurl.com/ycth2bm4                                                                                                       
    https://communities.sas.com/t5/SAS-Programming/Flag-Last-Assay-Date-Prior-to-Exposure-Date/m-p/530358                              
                                                                                                                                       
    Novinosrin Profile                                                                                                                 
    https://communities.sas.com/t5/user/viewprofilepage/user-id/138205                                                                 
                                                                                                                                       
    INPUT                                                                                                                              
    =====                                                                                                                              
                                                                                                                                       
    data have ;                                                                                                                        
       input                                                                                                                           
             @1 seq 1.                                                                                                                 
             @3 subject 3.                                                                                                             
             @7 assay_name $24.                                                                                                        
            @32 assay_date  ymddttm16.                                                                                                 
            @49 exposure_date ymddttm16.                                                                                               
       ;                                                                                                                               
    format                                                                                                                             
          assay_date exposure_date datetime18.;                                                                                        
    cards4;                                                                                                                            
    1 001 Alanine Aminotransferase 2017-07-05T12:01 2017-07-19T09:55                                                                   
    2 001 Alanine Aminotransferase 2017-07-19T09:22 2017-07-19T09:55                                                                   
    3 001 Albumin                  2017-08-15T09:38 2017-07-19T09:55                                                                   
    4 001 Albumin                  2017-08-30T09:09 2017-07-19T09:55                                                                   
    5 001 Albumin                  2017-07-15T08:10 2017-07-19T09:55                                                                   
    ;;;;                                                                                                                               
    run;quit;                                                                                                                          
                                                                                                                                       
                                                              |                                                                        
     SUBJECT  ASSAY_NAME  ASSAY_DATE      EXPOSURE_DATE   | DELTA_SECS    WANT                                                         
                                                          |                                                                            
        1     Alanine  05JUL17:12:01:00  19JUL17:09:55:00 |    1202040     N                                                           
        1     Alanine  19JUL17:09:22:00  19JUL17:09:55:00 |       1980     Y  asy closest before expos                                 
                                                                                                                                       
        1     Albumin  15AUG17:09:38:00  19JUL17:09:55:00 | 1.7977E308     N                                                           
        1     Albumin  30AUG17:09:09:00  19JUL17:09:55:00 | 1.7977E308     N                                                           
        1     Albumin  15JUL17:08:10:00  19JUL17:09:55:00 |     351900     Y  asy closest before expos                                 
                                                                                                                                       
                                                                                                                                       
    EXAMPLE OUTPUT                                                                                                                     
    --------------                                                                                                                     
                                                                                                                                       
    WORK.WANT total obs=5                                                                                                              
                                                                                                                                       
      SEQ    SUBJECT    ASSAY_NAME                     ASSAY_DATE        EXPOSURE_DATE      DELTA_SECS    WANT                         
                                                                                                                                       
       1        1       Alanine Aminotransferase    05JUL17:12:01:00    19JUL17:09:55:00       1202040     N                           
       2        1       Alanine Aminotransferase    19JUL17:09:22:00    19JUL17:09:55:00          1980     Y                           
       3        1       Albumin                     15AUG17:09:38:00    19JUL17:09:55:00    1.7977E308     N                           
       4        1       Albumin                     30AUG17:09:09:00    19JUL17:09:55:00    1.7977E308     N                           
       5        1       Albumin                     15JUL17:08:10:00    19JUL17:09:55:00        351900     Y                           
                                                                                                                                       
    PROCESS                                                                                                                            
    =======                                                                                                                            
                                                                                                                                       
    1. Proc Sql                                                                                                                        
    -----------                                                                                                                        
                                                                                                                                       
    proc sql;                                                                                                                          
      create                                                                                                                           
         table want as                                                                                                                 
      select                                                                                                                           
         *                                                                                                                             
        ,case                                                                                                                          
           when (exposure_date-assay_date<0) then constant('bigint')                                                                   
           else exposure_date-assay_date                                                                                               
         end as delta_secs                                                                                                             
        ,case                                                                                                                          
           when (min(calculated delta_secs)=calculated delta_secs) then 'Y'                                                            
           else 'N'                                                                                                                    
         end as want                                                                                                                   
      from                                                                                                                             
         have                                                                                                                          
      group                                                                                                                            
         by  subject                                                                                                                   
            ,assay_name                                                                                                                
      order                                                                                                                            
         by seq                                                                                                                        
    ;quit;                                                                                                                             
                                                                                                                                       
                                                                                                                                       
    2. HASH Paul Dorfman                                                                                                               
    --------------------                                                                                                               
                                                                                                                                       
    data want (drop = _:) ;                                                                                                            
      if _n_ = 1 then do ;                                                                                                             
        dcl hash h () ;                                                                                                                
        h.definekey ("subject", "assay_name") ;                                                                                        
        h.definedata ("_min") ;                                                                                                        
        h.definedone () ;                                                                                                              
        do until (z) ;                                                                                                                 
          set have end = z ;                                                                                                           
          if exposure_date < assay_date then continue ;                                                                                
          if h.find() ne 0 then _min = exposure_date - assay_date ;                                                                    
          else _min = min (_min, exposure_date - assay_date) ;                                                                         
          h.replace() ;                                                                                                                
        end ;                                                                                                                          
      end ;                                                                                                                            
      set have ;                                                                                                                       
      delta_secs = ifN (exposure_date => assay_date, exposure_date - assay_date, constant ("bigint")) ;                                
      h.find() ;                                                                                                                       
      want = char ("NY", (delta_secs = _min) + 1) ;                                                                                    
    run ;                                                                                                                              
                                                                                                                                       
                                                                                                                                       
    3. DOW loop                                                                                                                        
    -----------                                                                                                                        
    data want;                                                                                                                         
      retain mindif %sysfunc(constant(BIG));                                                                                           
      do until (last.assay_name);                                                                                                      
        set have;                                                                                                                      
        by subject assay_name;                                                                                                         
        dif = exposure_date-assay_date;                                                                                                
        if dif<0 then dif=constant('big');                                                                                             
        mindif=min(dif,mindif);                                                                                                        
      end;                                                                                                                             
      do until (last.assay_name);                                                                                                      
        set have;                                                                                                                      
        by subject assay_name;                                                                                                         
        if exposure_date-assay_date = mindif then flg='Y';                                                                             
        else flg='N';                                                                                                                  
        output;                                                                                                                        
      end;                                                                                                                             
      mindif = constant('big');                                                                                                        
    run;quit;                                                                                                                          
                                                                                                                                       
