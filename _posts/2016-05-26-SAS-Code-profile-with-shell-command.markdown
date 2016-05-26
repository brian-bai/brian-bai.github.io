---
layout: post
title:  "SAS code profile with shell command"
date:   2016-05-26 01:13:05 +0000

categories: linux shell
---
- How many lines of code have you written?
- What procedure have you used?

Such kind of questions are often asked in an interview for technical position.
Recently, our project is looking forward to hire a new resource.
When I asked this question to the candidates, I was also wondering how big my current project is.

So I decided to have a look at the current projet code base.I have done the similar analysis before for other projects using Java or Python. For I am working on my linux laptop recently, I used shell command this time.


## Analysis step

{% highlight shell %} 
#total lines
find ./ -type f -name *.sas | xargs cat | wc -l
>107689
#empty lines
find ./ -type f -name *.sas | xargs grep '^\s*$' | wc -l
>15960

#source code
find ./ -type f -name *.sas | xargs /home/brian/github/dev_tips/remove_sas_comment.sh | grep -v '^\s*$' | wc -l
>66113

#file number
find ./ -type f -name *.sas | wc -l
>518

#list top 3 files by linenumber
find ./ -type f -name *.sas | xargs wc -l | sort -g | tail -4
>1499 ./ETL/UTIL/J001_CUT_OFF_VALIDATION_RECOVERY.sas
>2693 ./MACRO/STAGING/UTIL_STAG_POSITION.sas
>3885 ./ETL/UTIL/J003_ORACLE_TO_METADATA.sas
>107689 total

#calculate the avg line number
expr $((66113/518))
>127

#how many proc used
find ./ -type f -name *.sas | xargs remove_sas_comment.sh | grep -v '^\s*$' | grep -i '^\s*proc\s' | wc -l
>2800

#list the proc
find ./ -type f -name *.sas | xargs remove_sas_comment.sh | grep -v '^\s*$' | grep -i '^\s*proc\s' | awk '{print $2;}' |sort > procs.txt

#how many data step used
find ./ -type f -name *.sas | xargs remove_sas_comment.sh | grep -v '^\s*$' | grep -i '^\s*data\s' |wc -l
>2054

#unique proc
tr -cs "[:alpha:]" "\n" < procs.txt | tr "[:lower:]" "[:upper:]" | uniq -c | sort -bnr
>
   1258 SQL
    439 SORT
    247 DATASETS
    186 PRINTTO
    107 TRANSPOSE
    100 RISK
     71 APPEND
     67 CONTENTS
     55 COMPILE
     43 FORMAT
     23 REPORT
     23 GPLOT
     22 METADATA
     18 SORTT
     17 OLAP
     14 SUMMARY
     14 OLAPOPERATE
     13 PRINT
     13 EXPORT
     12 COPY
     11 METALIB
     11 EXPAND
      8 TABULATE
      5 MEANS
      5 GCHART
      5 DELETE
      3 FREQ
      2 OPTIONS
      2 IOMOPERATE
      2 INFOMAPS
      2 IML
      1 IMPORT
      1 CORR


#unique proc number
tr -cs "[:alpha:]" "\n" < procs.txt | tr "[:lower:]" "[:upper:]" | uniq | wc -l
>33
{% endhighlight %}

## Result
The top 10 procedure is below:

{% highlight shell %} 

1258 SQL
 439 SORT
 247 DATASETS
 186 PRINTTO
 107 TRANSPOSE
 100 RISK
  71 APPEND
  67 CONTENTS
  55 COMPILE
  43 FORMAT
{% endhighlight %}

From this we know the *SQL* and *SORT* and *TRANSPOSE* proc are used most often for data wrangling in this SAS project. Proc *RISK* is here for this project is for Risk anlalysis.
