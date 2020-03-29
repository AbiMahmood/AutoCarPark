#include <Servo.h>
#include <Wire.h> 
#include <LiquidCrystal_I2C.h>


Servo myservo; /*~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~naming the motor to myservo */
LiquidCrystal_I2C lcd(0x3F, 2, 1, 0, 4, 5, 6, 7, 3, POSITIVE); /*LCD address */

 
const int place[]={13,12,11,10,9,8}; /*~~~~~~~~~~~~~~~~~~~~~~~~~ Parking space IR sensors */
byte val[6];
const int in  =7; /*~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ IR input pin for Car In */
const int out =6; /*~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ IR input pin for Car OUT */
int count=0;
int valin=0;
int valout=0;
int pos=0; 
int y=0;




/***************************************************** Void Setup Starts here ********************************************************/
void setup() {
  Serial.begin(9600);
  for(int i=0;i<5;i++)
     {pinMode(place[i], INPUT);}  
     
  pinMode(in, INPUT);
  pinMode(out, INPUT);

/**************************** Servo Calibration Starts here ****************************/
 
 myservo.attach(5);
 myservo.write(0);
  
  for (pos = 0; pos <= 80; pos += 4) { 
    myservo.write(pos);              
    delay(30);                       
  }
  delay(1000);
  for (pos = 80; pos >= 0; pos -= 4) { 
    myservo.write(pos);             
    delay(30);                       
  }
   myservo.detach();

/**************************** Servo Calibration Ends here ****************************/
  
  lcd.begin(16, 2);
  lcd.setCursor(0, 0);
  lcd.print("----WELCOME-----");
  delay(1000)
  lcd.clear();
  lcd.setCursor(0, 0);
  count=0; 
  
}
/***************************************************** Void Setup Ends here ********************************************************/







/*********************************************************************   Void Loop Starts Here ******************************************************************/
void loop() {
  
  lcd.setCursor(0, 0);
  lcd.print("P-L:"); //PL = Parking Left
  lcd.setCursor(5, 0);
  
  for( y=0;y<6;y++) /*~~~~~~~~~~~~~~~~~~~~~ It displays the available parking space */
     {      
      val[y]=digitalRead(place[y]);         
           if(val[y]==1)
               {lcd.print(y+1);
                lcd.print(","); }            
     }
       
  lcd.print("       ");        
  valin =digitalRead(in);
  valout=digitalRead(out);


  if(count>=6) {  /*~~~~~~~~~~~~~~~~~~~~~~~ Displays Full when there is no Space */
        count=6;
        myservo.write(0);
        delay(1000);
        myservo.detach();
        lcd.setCursor(12, 1);  
        lcd.print("FULL");
      }





/*-----------------------------------------------------------  For In Going Car to the ParkingLot  ------------------------------------------------------------*/   
/*-------------------------------------------------------------------------------------------------------------------------------------------------------------*/  
  if(valout==LOW){  
    if(count<6){ myservo.attach(5); }  /*~~~~~~~~~~~~~ If ParkingLot isn't FULL yet, ServoMotor Starts. 
                                                       When ParkingLot is FULL, ServoMotor won't Start */
        
    for (pos = 0; pos <= 80; pos += 2){   /*~~~~~~~~~~ Barrier opens if a car is infront of the Ingoing sensor*/
               myservo.write(pos);
               delay(30); }     
                   
    while(valout==LOW){
         valout=digitalRead(out); }
        
        count++;  /* ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ Counter increased as one car is parked */
        
    if (count>6){ count = 6; }          // 
    else if (count<6){ }                //
        
    delay(1000);     
    
    for (pos = 80; pos >= 0; pos -= 2) { /* ~~~~~~~~~~ Barrier closes Down after a car has passes through the Ingoing sesnor*/
                myservo.write(pos);
                delay(30);}      
                
       myservo.detach();}  /*~~~~~~~~~~~~~~~~~~~~~~~~ Servo motor turns off after a car is passed through the ingoing sensor. 
                                                      Turning off the motor for better Power Efficiency */
      
/*-------------------------------------------------------------------------------------------------------------------------------------------------------------*/   




/*----------------------------------------------------------  For Out Going Car from the ParkingLot  ----------------------------------------------------------*/
/*-------------------------------------------------------------------------------------------------------------------------------------------------------------*/      
  if(valin ==LOW){ 
  if(count>0){myservo.attach(5);}    /*~~~~~~~~~~~~~~ If ParkingLot isn't EMPTY yet, ServoMotor Starts. 
                                                      When ParkingLot is EMPTY, ServoMotor won't Start */  
  
   for (pos = 0; pos <= 80; pos += 2) { /*~~~~~~~~~~~ Barrier will be open if a car is infront or the Outgoing sensor*/
             myservo.write(pos);
             delay(30);}  
  
   while(valin ==LOW){
             valin=digitalRead(in);}    
         
         count--; /*~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Counter decreased as one car is left */
        
    if(count<=0){count=0;}
        
    delay(1000);      
    
    for (pos = 80; pos >= 0; pos -= 2) {  /* ~~~~~~~~~ Barrier closes Down after a car has passes through the Outgoing sesnor*/
              myservo.write(pos);
              delay(30);}       
               
       myservo.detach(); }   /*~~~~~~~~~~~~~~~~~~~~~~ Servo motor turns off after a car is passed through the outgoing sensor. 
                                                      Turning off the motor for better Power Efficiency */
       
/*-------------------------------------------------------------------------------------------------------------------------------------------------------------*/ 




        
  lcd.setCursor(0, 1);
  lcd.print("FILLED:"); /*~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Shows how many cars are filled in ParkingLot */
  lcd.print(count);  
  
  if(count>=6){
      lcd.setCursor(12, 1);
      lcd.print("FULL");}   /*~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ When ParkingLot is FULL, Print Full*/
      
  else {                    /*~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ When ParkingLot is not FULL, Print blank*/
      lcd.setCursor(12, 1);
      lcd.print("    ");}
}

/*********************************************************************   Void Loop Ends Here *********************************************************************/
