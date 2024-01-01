// Define the ports numbers
const byte ON = 2, START=3, EXTRA=4, FULL=5;
const byte HEATER = 8, LOCK=9, DRAIN=10, PUMP=12;
const byte MOTOR=11;
// Define the state
byte state = 0;
// Define two timers
unsigned long timer1_interval=0, timer2_interval=0;
unsigned long timer1_prev = 0, timer2_prev = 0;
unsigned long timer1_current=0, timer2_current=0;
int startingTime=0, rTime, preState = 3, dryTimer;
bool timer1_On = false, timer2_On = false, spin=true, interrupted = false, interruptedOFF = false;
bool oneTime = true, interruptT = false;
String st ="";
  
//INTERRUPT FOR OFF
void machineTurnedOFF() { // ISR
    Serial.println(preState);

   if(state != 0 && preState !=0 && preState !=6 && preState !=7 && preState !=5){
    preState = state;
  	Serial.println("STOPPING MOTOR");
  	interruptedOFF = true;
   	interrupted = false;

     state = 5;}

}

//INTERRUPT FOR STOP
void machineSTOP() { // ISR
   Serial.println(preState);
   if(state != 0 && preState !=0 && preState !=6 && preState !=7 && preState !=5){
     Serial.println("STOPPING MOTOR");
  	 interrupted = true;
     interruptT = true;
    preState = state;

     state = 5;}


}

bool timer1(unsigned long amount) {
  if(timer1_On==false){
    timer1_On=true;
    timer1_interval = amount;
    timer1_prev = millis();
  }
  else{
    timer1_current = millis();
    if(timer1_current - timer1_prev > timer1_interval) {
      timer1_On = false;
      return true;     
    }
    
  }
  return false;
}

bool timer2(unsigned long amount) {
  if(timer2_On==false){
    timer2_On=true;
    timer2_interval = amount;
    timer2_prev = millis();
  }
  else{
    timer2_current = millis();
    if(timer2_current - timer2_prev > timer2_interval) {
      timer2_On = false;
      return true;     
    }
    
  }
  return false;
}

// REMAINING TIME
int remainingTime(bool mode,int start){
      if (!mode){
        return 200 - (millis()-start);
      }
      else{
        return 150 - (millis()-start);
      }


}

void setup() {
  	Serial.begin(9600);
  	// Setup the input pins
    pinMode(ON, INPUT_PULLUP);
    pinMode(START, INPUT_PULLUP);
    pinMode(EXTRA, INPUT_PULLUP);
    pinMode(FULL, INPUT_PULLUP);
  	// Setup the output pins
    pinMode(HEATER, OUTPUT);
    pinMode(LOCK, OUTPUT);
    pinMode(DRAIN, OUTPUT);
    pinMode(PUMP, OUTPUT);
  	pinMode(MOTOR, OUTPUT);

}

void loop() {
  //WHICH STATE WE ARE IN
  Serial.print("State ");
  Serial.print(state);
  Serial.println(": "+st);

  
  switch(state)
  {
	//OFF
    case 0:
    {
      st = "Off";
      detachInterrupt(digitalPinToInterrupt(START));
      detachInterrupt(digitalPinToInterrupt(ON));
      
      digitalWrite(HEATER, 0);
      digitalWrite(LOCK, 0);
      digitalWrite(DRAIN, 0);
      analogWrite(MOTOR, 0);
      digitalWrite(PUMP, 0);
      interruptT = false;
      interrupted = false; 
      interruptedOFF = false;


      if(digitalRead(ON)==0 && digitalRead(START)==0){
        preState = 0;
        state=1;
        Serial.println("State became 1");

      }
      break;
    }
    
    //PUMPING
    case 1:
    {


      st = "Pumping water";
      oneTime = true;
      dryTimer = 150;


      digitalWrite(LOCK, 1);
      digitalWrite(PUMP, 1);
      digitalWrite(DRAIN, 1);
      if(timer1(150)==true){
        preState = 1;
      	state=2;
        Serial.println("State became 2");
      }
      attachInterrupt(digitalPinToInterrupt(START), machineSTOP, RISING); // INT0 is pin2
	  attachInterrupt(digitalPinToInterrupt(ON), machineTurnedOFF, RISING); // INT0 is pin2
      break;    
    }
    
    //HEATING
    case 2: 
    {
      st = "Heating water";
      digitalWrite(HEATER, 1);
      digitalWrite(PUMP, 0);

      if (analogRead(A0) >= 229){
        state = 3;
        preState = 2;

      }
      break;
      
    }
    //WASHING
    case 3: 
    {
      st = "Washing";
      digitalWrite(HEATER, 0);
      
      // Turn motor on and off every 10 seconds
      if(oneTime){
        startingTime = millis();
       	oneTime = false;
      }

      

      if (!interruptT){
      	rTime = remainingTime(digitalRead(EXTRA),startingTime);

      }
      else{
        rTime -=10;
      }
      Serial.print("remaining time: ");
      Serial.println(rTime);
      Serial.println("");
       
        
      if(timer1(100) == true) {
        if(spin){
          analogWrite(MOTOR, 100);
          spin =!spin;
        }
        else{
          analogWrite(MOTOR,0);
          spin =!spin;
        }
      }
      if (timer2(rTime) == true || rTime <=0) {
        preState = 3;

        rTime =0;
        interruptT = false;
        state = 4;}
      break;
    }
    //DRYING
    case 4: 
    {
      st = "Drying";
      analogWrite(MOTOR, 255);
      dryTimer-=10;
      Serial.print("Time remaining: ");
      Serial.println(dryTimer);
      Serial.println("");
      
      if (dryTimer <= 0){
        dryTimer = 350;
        preState = 4;
        state = 5;
      }
      break;
    }
    //DRAINING
    case 5: 
    {
      //our interrupt case so every output will be reformatted
      st= "Draining";
      digitalWrite(HEATER, 0);
      digitalWrite(LOCK, 1);
      digitalWrite(DRAIN, 0);
      analogWrite(MOTOR, 0);
      digitalWrite(PUMP,0);
      Serial.print("Stop: ");
      Serial.println(interrupted);
      Serial.print("OFF: ");
      Serial.println(interruptedOFF);
      Serial.println(" ");

      if (interrupted && digitalRead(FULL) == 0){
        interrupted = false;
        preState = 5;

        state = 7;}
      else if (interruptedOFF && !digitalRead(FULL)){
      	interruptedOFF = false;
        preState = 5;
        state = 0;
      }
      else if (digitalRead(FULL) == 0){
        preState = 5;
        state = 6;}
      break;
    }
    //FINISH
    case 6:
    {
      st = "Finished";
      //finished tag allows user to switch off to properly restart
      digitalWrite(LOCK, 0);



      if(digitalRead(FULL) && (digitalRead(START) || digitalRead(ON)))
      {
        preState = 6;
        state = 0;
      }
      break;
    }
    //STOPPED
    case 7:
    {
      st = "Stopped";
      digitalWrite(LOCK,0);
      if(!digitalRead(START)){
        preState = 7;
        state =1;}
      break;
    }
    default:
    	break;
  }
}
