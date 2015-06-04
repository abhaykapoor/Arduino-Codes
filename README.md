# Arduino-Codes
Arduino Code for Date and Time on an LCD
#include <Time.h>  
// include liquid crystal libary
#include <LiquidCrystal.h>
// set pins for liquid crystal libary
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);
// time sync to PC is HEADER followed by unix time_t as ten ascii digits
#define TIME_MSG_LEN  11  // Header tag for serial time sync message
#define TIME_HEADER  'T'  // ASCII bell character requests a time sync message 
#define TIME_REQUEST  7    // run setup script
void setup()  
{
// set seiral port
  Serial.begin(9600);
  //set function to call when sync required
  setSyncProvider( requestSync);  
    // set up the LCD's number of rows and columns: 
  lcd.begin(16, 2);
  lcd.setCursor(0, 0);
    lcd.print("Waiting For Time");
    lcd.setCursor(0, 1);
   lcd.print("From Computer");
}
// end of setup
//start time update when time is grabbed
void loop(){    
  // check if time is uploaded from computer
  if(Serial.available() ) 
  {
    // proscess the time from computer
    lcd.clear();
    processSyncMessage();
  }
  if(timeStatus()!= timeNotSet)   
  {
    // turn on pin 13 light if time is set
    digitalWrite(13,timeStatus() == timeSet); // on if synced, off if needs refresh  
    // set lcd to 0,0 to write time
   lcd.setCursor(0, 0);
    digitalClockDisplay();
   // set lcd to 0,0 to write date  
    lcd.setCursor(0, 1);
    digitaldateDisplay();  
    
  }
  delay(1000);
}


void digitalClockDisplay(){


  // display time on line 0
  //print time:
  lcd.print( "TIME: ");
  //print hour
  lcd.print(hour());
  // print :
  lcd.print(":");
  //print minute
  lcd.print(minute());
  // print :
  lcd.print(":");
  //print seconds
  lcd.print(second());
  
}
void digitaldateDisplay(){
  // display date on line 1
  //print date:
  lcd.print("Date:");
  //print 
  lcd.print(" ");
  //print day
  lcd.print(day());
//print / 
  lcd.print("/");
   //print month
  lcd.print(month());
 //print / 
  lcd.print("/");
  //print year
  lcd.print(year()); 
  lcd.println(); 
}

void printDigits(int digits){
  // utility function for digital clock display: prints preceding colon and leading 0
  Serial.print(":");
  if(digits < 10)
    Serial.print('0');
    Serial.print(digits);
}

void processSyncMessage() {
  // if time sync available from serial port, update time and return true
  while(Serial.available() >=  TIME_MSG_LEN ){  // time message consists of a header and ten ascii digits
    char c = Serial.read() ; 
    Serial.print(c);  
    if( c == TIME_HEADER ) {       
      time_t pctime=0;
      for(int i=0; i < TIME_MSG_LEN -1; i++)
      {   
        c = Serial.read();          
        if( c >= '0' && c <= '9'){   
          pctime = (10 * pctime) + (c - '0') ; // convert digits to a number    
      }
      }   
      setTime(pctime);   // Sync Arduino clock to the time received on the serial port
    }  
  }
}

time_t requestSync()
{
  Serial.print(TIME_REQUEST);  
  return 0; // the time will be sent later in response to serial mesg
}
