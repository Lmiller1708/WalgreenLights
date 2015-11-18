// code to bit bang the protocol for the Brite Star Christmas lights
// Modified from WalgreenChristmasLights.ino 
// https://github.com/jbrown123/WalgreenLights/blob/master/WalgreenChristmasLights.ino

class BriteStarLights{
private:
  int  pin;  // the pin to write to
  int  numLights;

  volatile uint8_t *pinPort;
  int pinMask;

  void SendPacket(word val)
  {
    register volatile uint8_t *port = pinPort;
    register int mask = pinMask;

    /////////////////////////////////////////////
    //  **********   WARNING  **********
    // This code runs with INTERRUPTS DISABLED
    ////////////////////////////////////////////
    uint8_t oldSREG = SREG;  // get interrupt register state
    cli();    // disable interrupts

    //START BITS
    // 3 pulses of H 6.25us + L 6.67us
    // I measured 4 pluses of H 6.16us and L 6.56us
    // But 6 for each works just fine
    for (int i = 0; i < 3; i++)
    {
      *port |= mask;  // HIGH
      delayMicroseconds(6.16);
      *port &= ~mask;  // LOW
      delayMicroseconds(6.56);
    }
    //This 2us delay is needed
    delayMicroseconds(2);
    
    double  dlow = 4;  // LOW Delay
    double  dhigh = 1;  // HIGH Delay

    // Bits 1-4 Red
    // Bits 5-8 Blue
    // Bits 9-12 Green
    for (int i = 0; i < 12; i++)
    {
      *port |= mask;  // HIGH
      delayMicroseconds((val & 1) ? dlow : dhigh);
      *port &= ~mask;  // LOW
      delayMicroseconds(dlow);
      val >>= 1;  //Right Shift     
     }
       
    
    *port &= ~mask;  // LOW

    SREG = oldSREG;  // restore interrupts

    // hold for at least one full packet time
    // this allows the downstream neighbor to forward on
    // his state to the next guy
    // could be 60us if we wanted to get really tight
    // 100us seems like plenty
    // default system waits 1.13ms between packets
    // Seems to work best at 1.13ms
    delayMicroseconds(1130);
  }


  void InitString(void)
  {
    // these delay values were determined by monitoring a set of lights
    digitalWrite(pin, HIGH);
    delay(150);
    digitalWrite(pin, LOW);
    delay(225);

    for (int i = 0; i < numLights; i++)
    {
      SendPacket(0);
    }
  }

  // set the value on one color of lights only
  // all others at 'other' (default 0 = off)
  // offsets: 0=White, 1=Orange, 2=Blue, 3=Green, 4=Red
  void OneColor(int offset, word val, word other = 0)
  {
    for (int i = 0; i < numLights; i++)
    {
     if (i % 5 == offset)
        SendPacket(val);
     else
       SendPacket(other);
    }
  }

public:
  BriteStarLights(int _pin, int _numLights)
  {
    pin = _pin;
    numLights = _numLights;

    pinPort = (pin < 8) ? &PORTD : &PORTB;
    pinMask = (pin < 8) ? 1 << pin : 1 << (pin - 8);

    pinMode(pin, OUTPUT);
    digitalWrite(pin, LOW);

    InitString();
  }

  ~BriteStarLights(void)
  {
    digitalWrite(pin, LOW);
    pinMode(pin, INPUT);
  }

  // valPtr points to an array of char
  // each char holds the value (0-15) for
  // the bulbs in the string
  void SendValue(word *valPtr)
  {
    for (int i = 0; i < numLights; i++)
    {
      SendPacket(*valPtr++);
    }
  }

  void ColorWheel(word val)
  {
    for (word t = 4095; t > 0; t-=1)
    {    
      for (int i = 0; i < numLights; i++)
        {
          SendPacket(t);    
          //val <<= 1;
        }
      delay(50);
    }
  }

 word GetRGBColor (word red, word green, word blue)
 {
   red <<=8;
   blue <<=4; 
    
   word val;
    
   val = red;
   val |= blue;
   val |= green;
   
   return val;
 }

  
  void SetColorToOne(int LEDNum, word LastState, word red, word green, word blue)
  {
    
   word val = GetRGBColor(red,green,blue);
    
    for (int i = 0; i < numLights; i++)
    {
       if (i == LEDNum)
         SendPacket(val);
       else
         SendPacket(LastState) ;
    }
  }
  
  word SetColorToAll(word red, word green, word blue)
  {
    
   word val = GetRGBColor(red,green,blue);
    
    for (int i = 0; i < numLights; i++)
    {
         SendPacket(val);
    }
    return val;
  }
  
   void SwipeAllToColor(word laststate, int wtime, word red, word green, word blue)
  {
    
   word val = GetRGBColor(red,green,blue);
   
    if (laststate == 0){
       for (int i = 0; i < numLights + 1; i++)
       {
         for (int p = 0; p < i; p++)
         {
           SendPacket(val);
         }
         delay(wtime);
        }
    }
    else{
        for (int i = 0; i < numLights + 1; i++)
          {
           SendPacket(val);
           for (int p = numLights - i; p > 0; p--)
           {
             SendPacket(laststate);
           }
           delay(wtime);
          }
    }
  }
  
   
  // set the value on the red lights only
  // Pass in Brightness
  void RedOnly(word val)
  {
    val <<= 8;  //Left Shift
    for (int i = 0; i < numLights; i++)
    {
        SendPacket(val);
    }
  }
  void BlueOnly(word val)
  {
    val <<= 4;  //Left Shift
    for (int i = 0; i < numLights; i++)
    {
        SendPacket(val);
    }
  }
  void GreenOnly(word val)
  {
    val <<= 0;  //Left Shift
    for (int i = 0; i < numLights; i++)
    {
        SendPacket(val);
    }
  }
  void WhiteOnly(word val)
  {
    val = GetRGBColor(val,val,val);
    
    for (int i = 0; i < numLights; i++)
    {
        SendPacket(val);
    }
  }

  void AllOn(int Power)
  {
    SetColorToAll(Power,Power,Power);
  }

  void AllOff(void)
  {
    SetColorToAll(0,0,0);
  }
};

class BriteStarLights *lights;//, *lights2;

void setup()
{
  lights = new BriteStarLights(13, 10);
  //lights2 = new BriteStarLights(12, 10);
}

void loop()
{
  
  word LastState;
  
  // ramp up / down
  for (int v = 0; v < 15; v++)
  {
   LastState = lights->SetColorToAll(v,0,0); delay(100);
  }
  
  delay(500);
  
  //Flash Color
  for (int v = 1; v >= -1; v--)
  {
    lights->SetColorToAll(15,0,0); delay(100);
    lights->SetColorToAll(0,15,0); delay(100);
    lights->SetColorToAll(0,0,15); delay(100);
    lights->SetColorToAll(15,15,15); delay(100);
  }


  //Swipe
  for (int v = 1; v >= -1; v--)
  {
    lights->SwipeAllToColor(0,100,0,0,15); delay(250);
    LastState = lights->GetRGBColor(0,0,15);
    lights->SwipeAllToColor(LastState,100,0,15,0); delay(250);
  }
  
  delay(500);

  //Sparkle
  for (int x = 1; x >= -1; x--)
  {
    for (int v = 10; v >= -1; v--)
    {
      lights->SetColorToOne(v,LastState,15,0,0); delay(100);
      lights->SetColorToOne(v,LastState,0,15,0); delay(100);
      lights->SetColorToOne(v,LastState,0,0,15); delay(100);
      lights->SetColorToOne(v,LastState,15,15,15); delay(100);
    }
  }
  
  lights->SwipeAllToColor(0,100,15,0,0); delay(250);

  delay(500);
  
  
  // ramp up / down
  for (int v = 15; v > 0; v--)
  {
   LastState = lights->SetColorToAll(v,0,0); delay(100);
  }
  
  delay(500);  

}
