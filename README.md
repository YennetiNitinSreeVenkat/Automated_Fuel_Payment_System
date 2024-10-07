# Automated_Fuel_Payment_System
Embedded Systems Project
Designed and developed an Automated Fuel Payment System using RFID technology to streamline and simplify the payment process at fuel stations. This system automates transactions, reduces waiting times, and enhances user convenience by integrating secure digital payment methods, ensuring efficient and hassle-free refueling experiences.


//Hello, This is the code to use a 4x4 keypad matrix with and Arduino and show the result on an LCD screen
//You should wire you keypad from 8to1 (keypad pins) to 9to2 Arduino digital pins
//SurtrTech
#include <SPI.h>
#include <MFRC522.h>
#include<Keypad.h> //The keypad and LCD i2c libraries
#include<Wire.h>
#include<LCD.h>
#include<LiquidCrystal_I2C.h>

#define RST_PIN         A3          // Configurable, see typical pin layout above
#define SS_PIN          10         // Configurable, see typical pin layout above
#define I2C_ADDR 0x27 //defining the LCD pins
#define BACKLIGHT_PIN 3
#define En_pin 2
#define Rw_pin 1
#define Rs_pin 0
#define D4_pin 4
#define D5_pin 5
#define D6_pin 6
#define D7_pin 7



byte read_card[4];
MFRC522::MIFARE_Key key ;
byte card_in[16] ;
byte read_val[18];
int card_out=0 ;
int val;
int balance ;
int lcd_out ; 
int state ;
byte status;
int k=0 ;
char lcd_in[10];



MFRC522 card(SS_PIN, RST_PIN);  // Create MFRC522 instance


LiquidCrystal_I2C lcd(I2C_ADDR,En_pin,Rw_pin,Rs_pin,D4_pin,D5_pin,D6_pin,D7_pin);

 

const byte numRows= 4; //number of rows on the keypad
const byte numCols= 4; //number of columns on the keypad
                      //keymap defines the key pressed according to the row and columns just as appears on the keypad

char keymap[numRows][numCols]=
{
{'1', '2', '3'},
{'4', '5', '6'},
{'7', '8', '9'},
{'*', '0', '#'}
};


byte rowPins[numRows] = {8,9,7,6}; //Rows 0 to 3 //if you modify your pins you should modify this too
byte colPins[numCols]= {5,4,3,2}; //Columns 0 to 3

//initializes an instance of the Keypad class
Keypad myKeypad= Keypad(makeKeymap(keymap), rowPins, colPins, numRows, numCols);

void card_write()
{
   status = card.PCD_Authenticate(MFRC522::PICC_CMD_MF_AUTH_KEY_A, 3, &key, &(card.uid));
    if(status!=MFRC522::STATUS_OK)
    {
      Serial.print("auth :");
      Serial.println(card.GetStatusCodeName(status));
    return;
    }
    status=card.MIFARE_Write(2, card_in, 16);
    if(status!=MFRC522::STATUS_OK)
    {
     Serial.println("write :");
     Serial.println(card.GetStatusCodeName(status));
     return ;
     
    }
    Serial.println("writing done");
}

void card_read()
{
  card_out=0;
  status = card.PCD_Authenticate(MFRC522::PICC_CMD_MF_AUTH_KEY_A, 3, &key, &(card.uid));
    if(status!=MFRC522::STATUS_OK)
    {
      Serial.print("auth :");
      Serial.println(card.GetStatusCodeName(status));
    return;
    }
    byte size=18;
    status=card.MIFARE_Read(2, read_val, &size);
    if(status!=MFRC522::STATUS_OK)
    {
     Serial.println("read :");
     Serial.println(card.GetStatusCodeName(status));
    
     return ;
     
    }
     for(int i = 0 ; i<16;i++)
    {
      card_out += read_val[i];
    }
    Serial.println("reading done");
}

void convert (int val )
{
    for(int i=0 ;val>=0;i++)
  {
    if(val<255)
      card_in[i] = val ; 
    else
      card_in[i] = 255 ;
      val -= 255;  
  }
}

void wlcm()
{
  state = 0 ;
  lcd.clear();
  lcd.setCursor(2,0);
  lcd.print("Select Method");
  lcd.setCursor(0,1);
  lcd.print("(*)Amt");
  lcd.setCursor(10, 1);
  lcd.print("Lit(#)");
 }

 void amt()
 {
    state = 1;
    lcd.clear();    
    lcd.setCursor(1, 0);
    lcd.print("Enter Amount:");
    lcd.setCursor(1, 1);
    lcd.print("Rs.");
    lcd.blink();
    delay(1000);

 }

 void lit()
 {
    state = 3;
    lcd.clear();    
    lcd.setCursor(1, 0);
    lcd.print("Enter Litre:");
    lcd.setCursor(12, 1);
    lcd.print("L");
    lcd.setCursor(1,1);
    lcd.blink();
    delay(1000);
 }

 int final(char y[10],int j)
 {
  lcd_out=0;
  Serial.print("y ; ");
  for(int h= 0;h<=j;h++)
  Serial.print(y[h]);
  lcd.clear();
  lcd.noBlink();
  lcd.print("Processing");
  lcd.blink();
  lcd_out = atoi(y);
  if(state==3)
  lcd_out = 100*atoi(y);
  Serial.println(lcd_out);
   
 }
  


void setup()
{
     Serial.begin(9600);
     lcd.begin (16,2);
     lcd.setBacklightPin(BACKLIGHT_PIN,POSITIVE);
     lcd.setBacklight(HIGH);
     lcd.home();
     wlcm();
     SPI.begin();
    card.PCD_Init();
    Serial.println("init done , card lao");
    for (byte i = 0; i < 6; i++) 
    {
      key.keyByte[i] = 0xFF;
    }
     
}

void loop()
{
  if(state==0)
  {
    char inp = myKeypad.getKey();
    if(inp!=NO_KEY)
    {
      if(inp=='*')
        amt();
      if(inp=='#')
        lit();
    Serial.println(inp);
    }
  }

  if(state==1||state==3)
  {
    char inp = myKeypad.getKey();
    if(inp!=NO_KEY)
    {
      if(inp=='#')
      {
        Serial.print("lcd_in ; ");
        for(int h=0; h<k;h++)
        Serial.print(lcd_in[h]);
        Serial.println(k);
        final(lcd_in,k-1);
        lcd.noBlink();
        state=2;
        goto l;
      }
      Serial.println(inp);
      lcd.write(inp);
      lcd_in[k] = inp;
      k++;
    }
  }
  
  l: 
  if(state==2)
  {
    card.PCD_Init();
    if(!card.PICC_IsNewCardPresent())
  {return;}
  if(!card.PICC_ReadCardSerial())
  return;
  card_read();
  balance = card_out ;
  if(balance<lcd_out)
  {
    lcd.clear();
    lcd.print("Insuff. Bal.");
    delay(5000);
    wlcm();
    goto p;
  }
  convert(balance-lcd_out) ;
  Serial.print("balance = ");Serial.println(balance);
  Serial.print("lcd_out = ");Serial.println(lcd_out);
  for(int p = 0 ; p<16;p++)
  {
    Serial.print(card_in[p]);Serial.print(" ");
  }
  card_write();
  lcd.clear();
  lcd.print("Done !");
  delay(5000);
  wlcm();
  p: card.PICC_HaltA();
      k=0;
    for(int h= 0 ; h<10 ; h++)
    {
      lcd_in[h] = '0';
    }
  }
 }
