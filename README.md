# ADVS
options validvarname=upcase nofmterr missing=' ' msglevel=i;
proc sort data=sdtm.vs out=vs; by usubjid; run;

data vs1(keep=studyid domain usubjid vsseq vstestcd vstest vscat vsorres aval
param parmacd vsorresu vsstresc vsstresu vsstat vsblfl visitnum visit visitdy epoch vsdtc adt ady avisit avisitn);
set vs (drop=domain);
domain="ADVS";
param=cats(vsteset, '(',vsorresu,')');
paramcd=vstest;
aval=vsstresn;
avalc=vsstresc;
ablfl=vsblfl;
adt=vsdtc;
ady=vsdy;
if ady<=1 then avisit="baseline";
else if ady>=2 and ady<=61 then avisit="Month1";
else if ady>=62 and ady<=136 then avisit="Month3";
else if ady>=137 and ady<=211 then avisit="Month6";

if avisit="Baseline" then avisitn=0;
else if avisit="Month 1" then avisitn=1;
else if avisit="Month 3" then avisitn=3;
else if avisit="Month 6" then avisitn=6;
run;

Proc sort data=vs1 out=base (keep=usubjid paramcd aval ablfl avisit avisitn ady); 
where ablfl="Y" and aval ne .;
by usubjid paramcd;
run;

proc sort data=vs1 out=pbase (Keep=usubjid paramcd avala ablfl avisit avisitn);
where ablfl ne 'Y' and aval ne. and ady >= 1;
by usubjid paramcd;
run;

data base1 (rename=(aval=base));
set base pbase;
by usubjid;
run;

Proc sort data=base; by usubjid; run;
Proc sort data=pbase; by usubjid; run;

data vs4 (keep=usubjid paramcd aval avisit avisitn );
set base pbase;
by usubjid;
run;

Proc sort data=vs1; by usubjid paramcd; run;
proc sort data=base; by usubjid paramcd; run;

data vs4;
merge vs1 base (keep=usubjid base paramcd);
by usubjid paramcd;
run;

data vs5;
set vs4;
if aval ne. and base ne . then
chg=aval-base;
else chg=.;
if nmiss(aval,base)=0 then
pchg=((aval-base)/base)*100;
else pchg=.;
run;

/*compress ('parameter',,'kw'); to remove extra spaces*/


if paramcd='sysbp' and ((aval>180 or CHG>40) or (Aval<90 or CHG
else if paramcd='DIABP' and ((AVAL>105 or CHG>30) or (AVAL<50
else if parmacd='HR' and ((AVAL>105 or CHG>30) or (AVAL<50 or CHG
else if paramcd='Temp' and (Aval>38 or CHG>-=1) then pcsfl='Y';
else if paramcd='weight' and ((CHG>7% of Basline value) or (CHG<=
run;

data base;
set vs1;
if vsblfl="Y" and aval ne. Then base=aval;
run;

