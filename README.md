_SECTION_BEGIN("MarketType");
MarketType=ParamList("MarketType","FX,FW20,akcje");
_SECTION_END();
_SECTION_BEGIN("VarMACD");
//OptimizerSetEngine("cmae"); // you can also use "spso" or "trib" 
//Averages
function geoavg(Array, Periods){ //!
sume = 1; 
for(i = 1; i<=Periods; i++){
		x = Ref(Close, -1 * i);
		sume *= x;
	}
geo = (sume) ^ (1/Periods);
return geo;
}
function quadavg(Array, Periods){  //!
sume = 1;
for(i = 1; i<=Periods; i++){
		x = Ref(Close, -1 * i);
		x=x^2;
		sume += x;
	}
quad = ((sume)/Periods) ^ (1/2);
return quad;
}
function poweravg(Array, Periods){ //!
sume = 1;
Power = Param("Power",3,2,10,1);
for(i = 1; i<=Periods; i++){
		x = ref(Close, -1 * i);
		x=x^Power;
		sume += x;
	}
powerA = ((sume)/Periods) ^ (1/Power);
return powerA;}
function haravg(Array,Periods){  //!
sume = 1;
for(i = 1; i<=Periods; i++){
		x = Ref(Close, -1 * i);
		x= 1/(x);
		sume += x;
	}
har = Periods/sume ;
return har;
}
function logavg(Array, Periods){
 x = (Ref(Close,-Periods) - Ref(Close,-1))/ln(Ref(Close,-Periods))/ln(Ref(Close,-1));
 return x;
}
function FRAMA(Array, Periods){ //!
Price = (H+L)/2;
N3 = ( HHV( High, Periods ) - LLV( Low, Periods ) ) / Periods;

HH = HHV( High, Periods / 2 ) ;
LL = LLV( Low, Periods / 2 );

N1 = ( HH - LL ) / ( Periods / 2 );

HH = HHV( Ref( High, - Periods/2 ), Periods/2 );
LL = LLV( Ref( Low, - Periods/2 ), Periods/ 2 );

N2 = ( HH - LL ) / ( Periods / 2 );

Dimen = IIf( N1 > 0 AND N2 > 0 AND N3 > 0, ( log( N1+N2) - log( N3 ) )/log( 2 ), Null );

alpha = exp( -4.6 * (Dimen -1 ) );
alpha = Min( Max( alpha, 0.01 ), 1 ); // bound to 0.01...1 range

Fram = AMA( Price, alpha );
if ( (Periods & 1) != 0 )Periods ++;
return Fram;}
function eVWMA(Array, Periods){
  result[0] = array[0];
  for (i = 1; i < BarCount; i++) 
  {
    result[i] = ((Periods - Volume[i]) * result[i - 1] + Volume[i] * array[i]) / Periods;
  }
  return result;
}
function T3(Array, Periods){
  s = 0.84;
  e1 = EMA(Array, Periods);
  e2 = EMA(e1, Periods);
  e3 = EMA(e2, Periods);
  e4 = EMA(e3, Periods);
  e5 = EMA(e4, Periods);
  e6 = EMA(e5, Periods);
  c1 = -s^3;
  c2 = 3*s^2+3*s^3;
  c3 = -6*s^2 - 3*s - 3*s^3;
  c4 = 1 + 3*s + s^3 + 3*s^2;
  return c1 * e6 + c2 * e5 + c3 * e4 + c4*e3;
}
function SMMA(Array, Periods){ //!
  fsum = 0;
  for (i = 0; i < periods; i++)
  {
    result[i] = Null;
    fsum += Array[i];
  } 
  result[periods] = fsum / Periods;

  for (i = periods + 1; i < BarCount; i++) 
  {
    result[i] = (result[i - 1] * periods - result[i - 1] + array[i]) / periods;
  }
  return result;
}
function SWMA(Array, Periods){ 
  SetBarsRequired(1000, 0);
    
  PI = 4 * atan(1);
  k = PI / (periods + 1);

  // denominator
  den = 0;
  for (i = 1; i <= periods; i++) den += sin(i * k);
  
  for (i = 0; i < periods - 1; i++) result[i] = Null;
  for (i = periods - 1; i < BarCount; i++) 
  { 
    nom = 0;
    for (j = 1; j <= periods; j++) nom += sin(j * k) * array[i - periods + j];
    
    result[i] = nom / den;
  }

  return result;}
function TMA(Array, Periods){
  if ((Periods % 2) > 0)
  { 
    // odd
    Coef1 = (Periods + 1) /2;
    Coef2 = Coef1;
  }
  else
  {
    // even
    Coef1 = Periods / 2 + 1;
    Coef2 = Periods / 2;
  }
  
  return MA(MA(Array, Coef1), Coef2);
}
function VWMA(Array, Periods){ 
  return MA(Volume * Array, Periods) / MA(Volume, Periods);
}
function ADMA(Array, Periods){
Periods=Param("Y",8,2,100,1);
Array=Param("Z",1,1,10,1);
//Period=Param("Period",1,2,100,0.1);
//EOM=Param("EOM",1,1,100,1);
//IIf(EaseOfMovement(EOM)>0,Range=round(Period*Momentum(C,EOM)/10),IIf(EaseofMovement(EOM)==0,Range=Ref(EaseOfMovement(EOM),-1),Range=round((-1)*Period*Momentum(C,EOM)/10)));
//IIf(Range==0,Range=2,Range=Range);
x=WMA(AccDist(),Array);
return x;}

//MACD
//P = ParamField("Field");
P = ( Ref(High,0) + Ref(Low,0) + Ref(Close,0) * 4 ) / 6;
MAType = ParamList("MA Type", "Exponential,Wilders,Weighted,logarithmic,Hull,T3,SWMA,TMA,VWMA,TSF,LinearRegression");
r1 = Param( "Fast avg", 12 );
r2 = Param( "Slow avg", 26 );
r3 = Param( "Signal avg", 9 );
r1 =  Optimize("Fast avg",r1,3,45,1);
r2 =  Optimize("Slow avg",r2,3,60,1);
r3 =  Optimize("Signal avg",r3,3,25,1);

if (MAType == "Simple")					{ a = MA(P, r1); b = MA(P, r2); }
if (MAType == "Exponential")			{ a = EMA(P, r1); b = EMA(P, r2); }
if (MAType == "Double Exponential")		{ a = DEMA(P, r1); b = DEMA(P, r2); }
if (MAType == "Triple Exponential")		{ a = TEMA(P, r1); b = TEMA(P, r2); }
if (MAType == "Wilders")				{ a = Wilders(P, r1); b = Wilders(P, r2);}
if (MAType == "Weighted")				{ a = WMA(P, r1); b = WMA(P, r2); }
if (MAType == "Quadratic")              { a = quadavg(P,r1); b = quadavg(P,r2);}
if (MAType == "Geometric")              { a = geoavg(P,r1); b = geoavg(P,r2);}
if (MAType == "Power")                  { a = poweravg(P,r1); b = poweravg(P,r2);}
if (MAType == "Harmonic")               { a = haravg(P,r1); b = haravg(P,r2);}
if (MAType == "logarithmic")            { a = logavg(P,r1); b = logavg(P,r2);}
if (MAType == "FRAMA")                  { a = FRAMA(P,r1); b = FRAMA(P,r2);}
if (MAType == "Hull")                  	{ a = HMA(P,r1); b = HMA(P,r2);}
if (MAType == "eVWMA")                  { a = eVWMA(P,r1); b = eVWMA(P,r2);}
if (MAType == "T3")                  	{ a = T3(P,r1); b = T3(P,r2);}
if (MAType == "SMMA")                  	{ a = SMMA(P,r1); b = SMMA(P,r2);}
if (MAType == "SWMA")                  	{ a = SWMA(P,r1); b = SWMA(P,r2);}
if (MAType == "TMA")                  	{ a = TMA(P,r1); b = TMA(P,r2);}
if (MAType == "VWMA")                  	{ a = VWMA(P,r1); b = VWMA(P,r2);}
if (MAType == "AMA")                  	{ a = AMA(P,r1); b = AMA(P,r2);}
if (MAType == "TSF")                  	{ a = TSF(P,r1); b = TSF(P,r2);}
if (MAType == "LinearRegression")       { a = LinearReg(P,r1); b = TSF(P,r2);}
if (MAType == "ADMA")       			{ a = ADMA(P,r1); b = ADMA(P,r2);}
varMACD = a - b;
if (MAType == "Simple")					{ VarSignal = MA(VarMACD, r3); }
if (MAType == "Exponential")			{ VarSignal = EMA(VarMACD, r3); }
if (MAType == "Double Exponential")		{ VarSignal = DEMA(VarMACD, r3); }
if (MAType == "Triple Exponential")		{ VarSignal = TEMA(VarMACD, r3); }
if (MAType == "Wilders")				{ VarSignal = Wilders(VarMACD, r3); }
if (MAType == "Weighted")				{ VarSignal = WMA(VarMACD, r3); }
if (MAType == "Geometric") 				{ VarSignal = geoavg(VarMACD, r3); }
if (MAType == "Quadratic") 				{ VarSignal = quadavg(VarMACD, r3); }
if (MAType == "Power") 					{ VarSignal = poweravg(VarMACD, r3); }
if (MAType == "Harmonic") 				{ VarSignal = haravg(VarMACD, r3); }
if (MAType == "logarithmic") 			{ VarSignal = logavg(VarMACD, r3); }
if (MAType == "FRAMA") 					{ VarSignal = FRAMA(VarMACD, r3); }
if (MAType == "Hull") 					{ VarSignal = HMA(VarMACD, r3); }
if (MAType == "eVWMA") 					{ VarSignal = eVWMA(VarMACD, r3); }
if (MAType == "T3") 					{ VarSignal = T3(VarMACD, r3); }
if (MAType == "SMMA") 					{ VarSignal = SMMA(VarMACD, r3); }
if (MAType == "SWMA") 					{ VarSignal = SWMA(VarMACD, r3); }
if (MAType == "TMA") 					{ VarSignal = TMA(VarMACD, r3); }
if (MAType == "VWMA") 					{ VarSignal = VWMA(VarMACD, r3); }
if (MAType == "AMA") 					{ VarSignal = AMA(VarMACD, r3); }
if (MAType == "TSF") 					{ VarSignal = TSF(VarMACD, r3); }
if (MAType == "LinearRegression") 		{ VarSignal = LinearReg(VarMACD, r3); }
if (MAType == "ADMA") 					{ VarSignal = AccDist(); }


//Plot( ml = varMACD, StrFormat(_SECTION_NAME()+"(%g,%g)", r1, r2), ParamColor("MACD color", colorRed ), ParamStyle("MACD style") );
//Plot( sl = VarSignal, "Signal" + _PARAM_VALUES(), ParamColor("Signal color", colorBlue ), ParamStyle("Signal style") );
ml = varMACD;
sl = VarSignal;

PositionSize = MarginDeposit = 1;
if(MarketType=="FW20"){
startTime = 085100; endTime = 152900;
tn = TimeNum();
startTime = 085100; 
endTime = 152900;  
timeOK = tn >= startTime AND tn <= endTime;

 Buy = Cross(ml , sl) AND timeOK;
 Sell = Cross( sl, ml) OR TimeNum() > 163900;
 Short = Cross( sl, ml) AND timeOK;
 Cover = Cross(ml , sl) OR TimeNum() > 163900;}
 if(MarketType =="FX"){
 if(r1<r2){
 Buy = Cross(ml , sl)AND Ref(Close,0) > Ref(Open,0);
 Sell = Cross( sl, ml)AND Ref(Close,0) < Ref(Open,0);
 Short = Cross( sl, ml)AND Ref(Close,0) < Ref(Open,0);
 Cover = Cross(ml , sl)AND Ref(Close,0) > Ref(Open,0); 
 }else{
 Buy = Cross(sl , ml)AND Ref(Close,0) > Ref(Open,0);
 Sell = Cross( ml, sl)AND Ref(Close,0) < Ref(Open,0);
 Short = Cross( ml, sl)AND Ref(Close,0) < Ref(Open,0);
 Cover = Cross(sl , ml)AND Ref(Close,0) > Ref(Open,0); }
 }
  
AlertIf( Buy, "SOUND C:\\Windows\\Media\\notify.wav", "Buy alert", 1 );
AlertIf( Short, "SOUND C:\\Windows\\Media\\notify.wav", "Short alert", 3 );
  
Ribbon=ParamList("Ribbon","On,Off");

if(r1 < r2){
if(Ribbon=="On"){
ribbonHistogram = IIf( ml-sl > 0, colorGreen, IIf( ml-sl < 0, colorRed, colorLightGrey) );
Plot(1, "", ribbonHistogram, styleOwnScale | styleArea | styleNoLabel | styleNoTitle, -0.5, 100);
GraphXSpace = 5;}
PlotShapes(Buy*shapeUpArrow, colorLime,layer=0,yposition=Low);
 PlotShapes(Short*shapeDownArrow, colorOrange,layer=0,yposition=High);
}else{
if(Ribbon=="On"){
ribbonHistogram = IIf( ml-sl > 0, colorRed, IIf( ml-sl < 0, colorGreen, colorLightGrey) );
Plot(1, "", ribbonHistogram, styleOwnScale | styleArea | styleNoLabel | styleNoTitle, -0.5, 100);
GraphXSpace = 5;}
PlotShapes(Buy*shapeUpArrow, colorLime,layer=0,yposition=Low);
 PlotShapes(Short*shapeDownArrow, colorOrange,layer=0,yposition=High);}

_SECTION_END();

_SECTION_BEGIN("PivotPoints");

GraphXSpace = 5 ;
SetChartOptions(0,chartShowArrows|chartShowDates);

Plot(C,"Close",colorBlack, styleCandle);
ppl = ParamToggle("Plot Pivot Levels","Off|On",1);

numbars = LastValue(Cum(Status("barvisible")));
fraction= IIf(StrRight(Name(),3) == "", 3.2, 3.2);
hts = -33.5;

PivotPeriod = ParamList("PivotPeriod","inDaily,inWeekly");
if( PivotPeriod == "inDaily"){
Hi = TimeFrameGetPrice("H", inDaily, -1);		
Lo = TimeFrameGetPrice("L", inDaily, -1);		
C1= TimeFrameGetPrice("C", inDaily, -1);}
if( PivotPeriod == "inWeekly"){
Hi = TimeFrameGetPrice("H", inWeekly, -1);		
Lo = TimeFrameGetPrice("L", inWeekly, -1);		
C1= TimeFrameGetPrice("C", inWeekly, -1);}

PivotType = ParamList("PivotPoints","Fibonacci,Camarilla");
if( PivotType == "Fibonacci" ){
rg = (Hi - Lo);
PP = (Hi + Lo + C1)/3;  bpI = LastValue (PP,1);
r1 = PP + (rg * 2.618); r1I = LastValue (r1,1);
r2 = PP + (rg * 1.618); r2I = LastValue (r2,1);
r3 = PP + (rg); 		r3I = LastValue (r3,1);
r4 = PP + (rg * 0.786); r4I = LastValue (r4,1);
r5 = PP + (rg * 0.618); r5I = LastValue (r5,1);
r6 = PP + (rg * 0.5);   r6I = LastValue (r6,1);
r7 = PP + (rg * 0.382); r7I = LastValue (r7,1);
r8 = PP + (rg * 0.236); r8I = LastValue (r8,1);
s1 = PP - (rg * 2.618); s1I = LastValue (s1,1);
s2 = PP - (rg * 1.618); s2I = LastValue (s2,1);
s3 = PP - (rg); 		s3I = LastValue (s3,1);
s4 = PP - (rg * 0.786); s4I = LastValue (s4,1);
s5 = PP - (rg * 0.618); s5I = LastValue (s5,1);
s6 = PP - (rg * 0.5);   s6I = LastValue (s6,1);
s7 = PP - (rg * 0.382); s7I = LastValue (s7,1);
s8 = PP - (rg * 0.236); s8I = LastValue (s8,1);
L1 = Lo - 1; 			L1I = LastValue (L1,1);
H1 = Hi;				H1I = LastValue (H1,1);

if(MarketType=="FW20"){
pp = round(pp);
s1 = round(s1);
s2 = round(s2);
s3 = round(s3);
s4 = round(s4);
s5 = round(s5);
s6 = round(s6);
s7 = round(s7);
s8 = round(s8);
r1 = round(r1);
r2 = round(r2);
r3 = round(r3);
r4 = round(r4);
r5 = round(r5);
r6 = round(r6);
r7 = round(r7);
r8 = round(r8);}

if(ppl==1) {
Plot(PP,"",colorYellow,styleLine|styleDots|styleNoRescale);
Plot(s1,"",colorWhite,styleLine|styleNoRescale);
Plot(s2,"",colorWhite,styleLine|styleNoRescale);
Plot(s3,"",colorLightBlue,styleLine|styleNoRescale);
Plot(s4,"",colorLightBlue,styleLine|styleNoRescale);
Plot(r1,"",colorWhite,styleLine|styleNoRescale);
Plot(r2,"",colorWhite,styleLine|styleNoRescale);
Plot(r3,"",colorLightBlue,styleLine|styleNoRescale);
Plot(r4,"",colorLightBlue,styleLine|styleNoRescale);
Plot(r5,"",colorGreen,styleLine|styleNoRescale);
Plot(r6,"",colorRed,styleLine|styleNoRescale);
Plot(r7,"",colorRed,styleLine|styleNoRescale);
Plot(r8,"",colorRed,styleLine|styleNoRescale);
Plot(s5,"",colorRed,styleLine|styleNoRescale);
Plot(s6,"",colorGreen,styleLine|styleNoRescale);
Plot(s7,"",colorGreen,styleLine|styleNoRescale);
Plot(s8,"",colorGreen,styleLine|styleNoRescale);
Plot(L1,"",ColorRGB(70,70,70),styleLine | styleNoRescale | styleDashed);
Plot(H1,"",ColorRGB(70,70,70),styleLine | styleNoRescale | styleDashed);}}

if( PivotType == "Camarilla" ) {
rg = (Hi - Lo);
H6 = (Hi / Lo) * C1; 		H6I = LastValue (H6,1);
H4 = 0.55*rg+  C1; 			H4I = LastValue (H4,1);
H3 = 0.275*rg + C1; 		H3I = LastValue (H3,1);
H2 = 0.183 * rg + C1; 		H2I = LastValue (H2,1);
H1 = 0.0916*rg + C1;  		H1I = LastValue (H1,1);
L1 = C1 - (0.0916 * rg);	L1I = LastValue (L1,1);
L2 = C1 - (0.183*rg); 		L2I = LastValue (L2,1);
L3 = C1 - 0.275*rg; 		L3I = LastValue (L3,1);
L4 = C1 - 0.55*rg; 			L4I = LastValue (L4,1);
L6 = C1 - (H6 - C1); 		L6I = LastValue (L6,1);
H5 = H4 + (H6 - H4)/2; 		H5I = LastValue (H5,1);
L5 = L6 + (L4 - L6) / 2; 	L5I = LastValue (L5,1);
L11 = Lo - 1; 				L11I = LastValue (L11,1);
H11 = Hi;					H11I = LastValue (H11,1);

if(MarketType=="FW20"){
H6 = round(H6);
H5 = round(H5);
H4 = round(H4);
H3 = round(H3);
H2 = round(H2);
H1 = round(H1);
L1 = round(L1);
L2 = round(L2);
L3 = round(L3);
L4 = round(L4);
L5 = round(L5);
L6 = round(L6);}

if(ppl==1) {
Plot(H6,"",colorWhite,styleLine|styleNoRescale);
Plot(L6,"",colorWhite,styleLine|styleNoRescale);
Plot(H5,"",colorPlum,styleLine|styleNoRescale);
Plot(L5,"",colorPlum,styleLine|styleNoRescale);
Plot(H4,"",colorGreen,styleLine|styleNoRescale);
Plot(L4,"",colorRed,styleLine|styleNoRescale);
Plot(H3,"",colorRed,styleLine|styleNoRescale);
Plot(L3,"",colorGreen,styleLine|styleNoRescale);
Plot(H2,"",colorRed,styleLine|styleNoRescale);
Plot(L2,"",colorGreen,styleLine|styleNoRescale);
Plot(H1,"",colorRed,styleLine|styleNoRescale);
Plot(L1,"",colorGreen,styleLine|styleNoRescale);
Plot(L11,"",ColorRGB(70,70,70),styleLine | styleNoRescale | styleDashed);
Plot(H11,"",ColorRGB(70,70,70),styleLine | styleNoRescale | styleDashed);}}
_SECTION_END();
_SECTION_BEGIN("ATR");
periods = Param( "Periods", 12, 1, 200, 1 );
Plot( ATR(periods), _DEFAULT_NAME(), ParamColor( "Color", colorCycle ),ParamStyle("Style", styleNoDraw | styleNoRescale | styleOwnScale ) );
_SECTION_END();
_SECTION_BEGIN("Volume");
//Plot( Volume, _DEFAULT_NAME(), ParamColor("Color", ColorRGB(60,60,60)), ParamStyle( "Style", styleHistogram | styleOwnScale | styleThick, maskHistogram  ) );
if(MarketType=="FW20" OR MarketType=="akcje"){
Plot(Volume, _DEFAULT_NAME(),
IIf(Ref(Volume,0)-Ref(Volume,-1)*1.1>0,ParamColor("ColorUP",ColorRGB(0,60,0)),IIf(Ref(Volume,0)-Ref(Volume,-1)*1.1<0,ParamColor("ColorDOWN",ColorRGB(60,0,0)),ParamColor("Color",ColorRGB(60,60,60)))),
ParamStyle("Style", styleHistogram | styleOwnScale | styleThick, maskHistogram ));}
_SECTION_END();
