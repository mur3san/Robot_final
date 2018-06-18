
/*
 * 1.verioficarea codul / debugging
 * 2.consolidare hardware*
 * 3. sensor ultrasonici oprire la intalnirea unui obiect  mobil
 * 4.evitare obstacol fix, recalcura traseu din pozitia curenta
 * (poate)cresterea razei pentru cititoru de carduri/ inlocuirea lui.
 * 5. Cautarea Rfidului prin miscari circulare crescatoare.
*/
#include <SPI.h>
#include <MFRC522.h>
#include <NewPing.h>
#define Catchup 600 // delay value
#define SIZE 8
#define SPEED 180  // viteza roomba
#define TURNSPEED 150 
#define MAX_DISTANCE 30
#define Travel_distance 100
int counter=0;    // counter pentru detectarea tipului de obiect mobil/fix
int counter1=0;   // counter pentru ScoutRfid
int pozitie=1; //Nord=1 Sud=2 Est=3 Vest=4
int Array[SIZE][SIZE];
int x=0,y=0;
int k=0,k1=0,k2=0,k3=0;
int solution[SIZE][SIZE];
constexpr uint8_t RST_PIN = 5;     // Configurable *schimba pt Mega
constexpr uint8_t SS_PIN = 53;     // Configurable 
MFRC522 mfrc522(SS_PIN, RST_PIN);   // Create MFRC522 instance.
MFRC522::MIFARE_Key key;
NewPing Usenzor1(10, 9, MAX_DISTANCE);  
NewPing Usenzor2(35, 34, MAX_DISTANCE);
void setup() {
 Serial3.begin(57600);// robot serial
 Serial1.begin(9600); //Bluetooth serial
 Serial.begin(9600);
 SPI.begin();
 mfrc522.PCD_Init();
 mfrc522.PCD_SetAntennaGain(mfrc522.RxGain_max);
 for (byte i = 0; i < 6; i++) {
        key.keyByte[i] = 0xFF;
    }
 delay(3000);
 startSafe();
 RfidRead();
}

void loop() {
  Go();
  
}
void startSafe(){
  Serial3.write((byte)128); // START
  Serial3.write((byte)132); // SAFE MODE
  delay(2000);
  
}
void RfidRead(){
  Serial.println("Apropie Card:");
  if ( ! mfrc522.PICC_IsNewCardPresent())
        return;
  if ( ! mfrc522.PICC_ReadCardSerial())
        return;
       counter1=0;
        // Show some details of the PICC (that is: the tag/card)
    Serial.print(F("Card UID:"));
    dump_byte_array(mfrc522.uid.uidByte, mfrc522.uid.size);
    Serial.println();
    Serial.print(F("PICC type: "));
    MFRC522::PICC_Type piccType = mfrc522.PICC_GetType(mfrc522.uid.sak);
    Serial.println(mfrc522.PICC_GetTypeName(piccType));
        // Check for compatibility
 if (    piccType != MFRC522::PICC_TYPE_MIFARE_MINI
        &&  piccType != MFRC522::PICC_TYPE_MIFARE_1K
        &&  piccType != MFRC522::PICC_TYPE_MIFARE_4K) {
        Serial.println(F("This sample only works with MIFARE Classic cards."));
        return;
    }
     byte sector3        = 3;
     byte sector2        = 2;
     byte blockAddr      = 8;
     byte blockAddr5     = 9;
     byte blockAddr6     = 10;
     byte trailerBlock   = 11;
     byte blockAddr7     = 12;
     byte blockAddr8     = 13;
    MFRC522::StatusCode status;
    byte buffer[18];
    byte size = sizeof(buffer);
    Serial.println(F("Current data in sector2 2:"));
    mfrc522.PICC_DumpMifareClassicSectorToSerial(&(mfrc522.uid), &key, sector3 );
    Serial.println();
    status = (MFRC522::StatusCode) mfrc522.MIFARE_Read(blockAddr8, buffer, &size);
    if (status != MFRC522::STATUS_OK) {
        Serial.print(F("MIFARE_Read() failed: "));
        Serial.println(mfrc522.GetStatusCodeName(status));
    }
    // x=buffer[0];
    // y=buffer[1];
    status = (MFRC522::StatusCode) mfrc522.MIFARE_Read(blockAddr7, buffer, &size);
    if (status != MFRC522::STATUS_OK) {
        Serial.print(F("MIFARE_Read() failed: "));
        Serial.println(mfrc522.GetStatusCodeName(status));
    }
    for(int i=0 ; i<2 ;i++){
    for(int j=0;j<8;j++){
     Array[i][j]=buffer[k];
     k++;  
  }}
 
    // Show the whole sector2 as it currently is
     Serial.println(F("Current data in sector2 1:"));
    mfrc522.PICC_DumpMifareClassicSectorToSerial(&(mfrc522.uid), &key, sector2);
    Serial.println();
  
     // Read data from the block
    
  status = (MFRC522::StatusCode) mfrc522.MIFARE_Read(blockAddr6, buffer, &size);
    if (status != MFRC522::STATUS_OK) {
        Serial.print(F("MIFARE_Read() failed: "));
        Serial.println(mfrc522.GetStatusCodeName(status));
    }
    for(int i=2 ; i<4 ;i++){
    for(int j=0;j<8;j++){
     Array[i][j]=buffer[k1];
     k1++;  
  }}
  status = (MFRC522::StatusCode) mfrc522.MIFARE_Read(blockAddr5, buffer, &size);
    if (status != MFRC522::STATUS_OK) {
        Serial.print(F("MIFARE_Read() failed: "));
        Serial.println(mfrc522.GetStatusCodeName(status));
    }
    for(int i=4 ; i<6 ;i++){
    for(int j=0;j<8;j++){
     Array[i][j]=buffer[k2];
     k2++;  
  }}
  status = (MFRC522::StatusCode) mfrc522.MIFARE_Read(blockAddr, buffer, &size);
    if (status != MFRC522::STATUS_OK) {
        Serial.print(F("MIFARE_Read() failed: "));
        Serial.println(mfrc522.GetStatusCodeName(status));
    }
    for(int i=6 ; i<8 ;i++){
    for(int j=0;j<8;j++){
     Array[i][j]=buffer[k3];
     k3++;  
  }}
  x=y=0;
  k=k1=k2=k3=0;
  counter1=0;
  for(int i=0 ; i<8 ;i++){
    Serial.println();  
    for(int j=0;j<8;j++){
     Serial.print(Array[i][j]);
     Serial.print(" ");     
  }}
  Serial.println();
  // Halt PICC
  mfrc522.PICC_HaltA();
  // Stop encryption on PCD
  mfrc522.PCD_StopCrypto1();
  sol();
}
void dump_byte_array(byte *buffer, byte bufferSize) {
    for (byte i = 0; i < bufferSize; i++) {
        Serial.print(buffer[i] < 0x10 ? " 0" : " ");
        Serial.print(buffer[i], HEX);
    }
}
boolean CheckNorth(){
  int nextNumber = solution[x+1][y];
  if(nextNumber==1 && x+1 < SIZE || nextNumber==2 && x+1 < SIZE ){
    return true;
  } else {
    Serial.println("North false");
    return false;
  }
}
boolean CheckSouth(){
  int nextNumber = solution[x-1][y];
  if(nextNumber==1 && x-1 >= 0 || nextNumber==2 && x-1 >= 0 ){
    Serial.println("South true");
    return true;
  } else {
    Serial.println("South false");
    return false;
  }
}
boolean CheckWest(){
  int nextNumber = solution[x][y+1];  
  if(nextNumber == 1 && y+1 < SIZE || nextNumber == 2 && y+1 < SIZE ){
    Serial.println(" West true");
    return true;
  } else {
    Serial.println(" West false");
    return false;
  }
}
boolean CheckEast(){
  int nextNumber = solution[x][y-1];
  if(nextNumber == 1 && y-1 >= 0 || nextNumber == 2 && y-1 >= 0){
    Serial.println("East true");
 /* Serial.print("X=");
    Serial.println(x);
    Serial.print("Y=");
    Serial.println(y);  */
    return true;
  } else {
    Serial.println("East false");
    return false;
  }
}
boolean Checkfinish(){
  int nextNumber = Array[x][y];
  if(nextNumber==Array[SIZE-1][SIZE-1]){
    Serial.println("finish true");
    return true;
  } else {
    Serial.println("finish false");
    return false;
  }
}
void TurnRight(int speed, int angle) {

  angle = -angle;

  Serial3.write((byte)137);
  Serial3.write((byte)(speed >> 8 & 0xFF));
  Serial3.write((byte)(speed & 0xFF));
  Serial3.write((byte)0xFF);
  Serial3.write((byte)0xFF);

  Serial3.write((byte)157);
  Serial3.write((byte)(angle >> 8 & 0xFF));
  Serial3.write((byte)(angle & 0xFF));
  delay(100);
  Serial3.write((byte)137);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
}
void TurnLeft(int speed, int angle) {

  Serial3.write((byte)137);
  Serial3.write((byte)(speed >> 8 & 0xFF));
  Serial3.write((byte)(speed & 0xFF));
  Serial3.write((byte)0);
  Serial3.write((byte)1);
  
  Serial3.write((byte)157);
  Serial3.write((byte)(angle >> 8 & 0xFF));
  Serial3.write((byte)(angle & 0xFF));
  delay(100);
  Serial3.write((byte)137);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
}
bool MoveForward(int speed, int distance) {  //merge
  if(Usenzori()==true){
    return false;
  }else
  Serial3.write((byte)137);
  Serial3.write((byte)(speed >> 8 & 0xFF));
  Serial3.write((byte)(speed & 0xFF));
  Serial3.write((byte)0x80);
  Serial3.write((byte)0);

  Serial3.write((byte)156);
  Serial3.write((byte)(distance >> 8 & 0xFF));
  Serial3.write((byte)(distance & 0xFF));
  delay(100);
  Serial3.write((byte)137);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
  return true;
}
void Stop() {
  Serial3.write((byte)137);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
}
void MoveBackward(int speed, int distance) {   //merge
 
  speed = -speed;

  Serial3.write((byte)137);
  Serial3.write((byte)(speed >> 8 & 0xFF));
  Serial3.write((byte)(speed & 0xFF));
  Serial3.write((byte)0x80);
  Serial3.write((byte)0);

  distance = -distance;

  Serial3.write((byte)156);
  Serial3.write((byte)(distance >> 8 & 0xFF));
  Serial3.write((byte)(distance & 0xFF));
  delay(100);
  Serial3.write((byte)137);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
  Serial3.write((byte)0);
}
void song(){
  Serial3.write((byte)140);
  Serial3.write((byte)1);
  Serial3.write((byte)3);
  Serial3.write((byte)80);
  Serial3.write((byte)16);
  Serial3.write((byte)60);
  Serial3.write((byte)16);
  Serial3.write((byte)80);
  Serial3.write((byte)16);
  delay(100);
  Serial3.write((byte)141);
  Serial3.write((byte)1);
}
 void printsolution()
{
    int i,j;
    for(i=0;i<SIZE;i++)
    {
        for(j=0;j<SIZE;j++)
        {
            Serial.print(solution[i][j]);
        }
        Serial.println();
    }
}
int solvemaze(int r, int c)
{
    if((r==SIZE-1) && (c==SIZE-1))
    {
        solution[r][c] = 1;
        return 1;
    }
    if(r>=0 && c>=0 && r<SIZE && c<SIZE && solution[r][c] == 0 && Array[r][c] == 0)
    {
        //if safe to visit then visit the cell
        solution[r][c] = 1;
        //going down
        if(solvemaze(r+1, c))
            return 1;
        //going right
        if(solvemaze(r, c+1))
            return 1;
        //going up
        if(solvemaze(r-1, c))
            return 1;
        //going left
        if(solvemaze(r, c-1))
            return 1;
        //backtracking
        solution[r][c] = 0;
        return 0;
    }
    return 0;

}
boolean sol(){
  int i1,j1;
    for(i1=0; i1<SIZE; i1++)
    {
        for(j1=0; j1<SIZE; j1++)
        {
            solution[i1][j1] = 0;
        }
    }
 
    if (solvemaze(x,y)){
      Serial.print("Xmaze=");
      Serial.print(x);
      Serial.println();
      Serial.print("Ymaze=");
      Serial.print(y);
      Serial.println();
        printsolution();
        return true;
    }
    else
        Serial.print("No solution\n");
        RfidRead();
        return false;
}
boolean Usenzori(){
    delay(50);
    if(Usenzor1.ping_cm()!= 0 || Usenzor2.ping_cm()!=0){
      Serial.println("Obiect detected");
      if(counter==5){
        Serial.println("Changing Path");
      switch(pozitie){
         case 1:
         Array[x+1][y]=1;
         break;
         case 2:
         Array[x-1][y]=1;
         break;
         case 3:
         Array[x][y-1]=1;
         break;
         case 4:
         Array[x][y+1]=1;
         break;
      }
      counter=0;
      } else
      counter++;
      return true;
    } else
    counter=0;
    Serial.println("Path Clear");
    return false;
  }
  void Go(){
    while(counter1>5){
    ScoutRFID();
    Serial.print("Go (while):counter1=");
    Serial.print(counter1);
    Serial.println();
}
    if(Checkfinish()==true){
    if(counter1<16)counter1++;
    Serial.print("CheckFinish: counter1=");
    Serial.print(counter1);
    Serial.println();
  }
    if(sol()==true && Checkfinish()==false ){
      AngleCorrection();
      bool hasMovedForward = false;
      if(CheckNorth()==true){
       switch(pozitie){
         case 1:
          hasMovedForward = MoveForward(SPEED,Travel_distance);
          delay(Catchup);
         break;
        case 2:
          TurnLeft(TURNSPEED,180);
          pozitie=1;
          delay(Catchup);
          hasMovedForward =  MoveForward(SPEED,Travel_distance);
          delay(Catchup);
         break;
        case 3:
          TurnLeft(TURNSPEED,90);
          pozitie=1;
          delay(Catchup);
          hasMovedForward = MoveForward(SPEED,Travel_distance);
          delay(Catchup);
         break;
        case 4:
          TurnRight(TURNSPEED,90);
          pozitie=1;
          delay(Catchup);
          hasMovedForward =  MoveForward(SPEED,Travel_distance);
          delay(Catchup);
         break; 
  }     
  if(hasMovedForward == false){
  return;
  } else
      //Array[x][y]=3;
      solution[x][y]=3;
      delay(100);
      if(x+1<SIZE)x++;
     }
    if(CheckSouth()==true && CheckNorth()==false ){
       switch(pozitie){
         case 1:
         TurnLeft(TURNSPEED,180);
         delay(Catchup);
         pozitie=2;
         hasMovedForward = MoveForward(SPEED,Travel_distance);
         delay(Catchup);
         break;
        case 2:
         hasMovedForward = MoveForward(SPEED,Travel_distance);
          delay(Catchup);
         break;
        case 3:
          TurnRight(TURNSPEED,90);
          pozitie=2;
          delay(Catchup);
         hasMovedForward = MoveForward(SPEED,Travel_distance);
          delay(Catchup);
         break;
        case 4:
          TurnLeft(TURNSPEED,90);
          pozitie=2;
          delay(Catchup);
         hasMovedForward = MoveForward(SPEED,Travel_distance);
          delay(Catchup);
         break; 
  } 
  if(hasMovedForward == false){
  return;  
  } else 
      //Array[x][y]=3;
      solution[x][y]=3;
      delay(100);
      if(x>0)x--;
     }
     if(CheckWest()==true && CheckNorth()==false){
     switch(pozitie){
      case 1:
      TurnLeft(TURNSPEED,90);
      pozitie=4;
      delay(Catchup);
      hasMovedForward = MoveForward(SPEED,Travel_distance);
      delay(Catchup);
      break;
      case 2:
      TurnRight(TURNSPEED,90);
      pozitie=4;
      delay(Catchup);
      hasMovedForward = MoveForward(SPEED,Travel_distance);
      delay(Catchup);
      break;
      case 3:
      TurnLeft(TURNSPEED,180);
      pozitie=4;
      delay(Catchup);
      hasMovedForward = MoveForward(SPEED,Travel_distance);
      delay(Catchup);
      break;
      case 4:
      hasMovedForward = MoveForward(SPEED,100);
      delay(Catchup);
      break;
    }
    if(hasMovedForward == false){
  return;
    }else
    //Array[x][y]=3;
    solution[x][y]=3;
      delay(100);
      if(y<SIZE)y++;  
     }
     if(CheckEast()==true && CheckNorth()==false){
     switch(pozitie){
    case 1:
    TurnRight(TURNSPEED,90);
    pozitie=3;
    delay(Catchup);
    hasMovedForward = MoveForward(SPEED,Travel_distance);
    delay(Catchup);
    break;
    case 2:
    TurnLeft(TURNSPEED,90);
    pozitie=3;
    delay(Catchup);
    hasMovedForward = MoveForward(SPEED,Travel_distance);
    delay(Catchup);
    break;
    case 3:
    hasMovedForward = MoveForward(SPEED,Travel_distance);
    delay(Catchup);
    break;
    case 4:
    TurnRight(TURNSPEED,180);
    pozitie=3;
    delay(Catchup);
    hasMovedForward = MoveForward(SPEED,Travel_distance);
    delay(Catchup);
    break;
  }
  if(hasMovedForward == false){
  return;
  }else
      //Array[x][y]=3;
      solution[x][y]=3;
      delay(100);
      if(y>0)y--;
     }
}else
 RfidRead();
 }
  
int rotate90degree(){
  for(int r=1 ; r<10 ; r++){
    TurnLeft(60,10);
    Serial.println("rotate90degrees");
    delay(500);
    if(mfrc522.PICC_IsNewCardPresent()){
      Serial.print("rotate90degree: r=");
    Serial.print(r);
    Serial.println();
      return r;
    }
  }
}
void ScoutRFID(){
  int p=20;
  if(counter1<15){
  while(! mfrc522.PICC_IsNewCardPresent()){
    MoveForward(80,p);
    rotate90degree();
    counter1++;
    p+=5;
    Serial.print("ScoutRFID: counter1=");
    Serial.print(counter1);
    Serial.println();
    if(mfrc522.PICC_IsNewCardPresent()){
       break;
    } 
  }
  counter1=0;
  RfidRead();
  Serial.print("counter1=");
    Serial.print(counter1);
    Serial.println();
 // AngleCorrection(); // trebe mutat altundeva 
  }else Serial.println("giveup"); 
}
void AngleCorrection(){
  switch(rotate90degree()){
    case 1:
    TurnLeft(60,80);
    ModificaPozitie();
    break;
    case 2:
    TurnLeft(60,70);
    ModificaPozitie();
    break;
    case 3:
    TurnLeft(60,60);
    ModificaPozitie();
    break;
    case 4:
    TurnLeft(60,50);
    ModificaPozitie();
    break;
    case 5:
    TurnLeft(60,40);
    ModificaPozitie();
    break;
    case 6:
    TurnLeft(60,30);
    ModificaPozitie();
    break;
    case 7:
    TurnLeft(60,20);
    ModificaPozitie();
    break;
    case 8:
    TurnLeft(60,10);
    ModificaPozitie();
    break;
    case 9:
    ModificaPozitie();
    break;
  }
}
void ModificaPozitie(){
  switch(pozitie){
    case 1:     // Nord=1 Sud=2 Est =3 Vest=4
    pozitie=4;
    break;
    case 2:
    pozitie=3;
    break;
    case 3:
    pozitie=1;
    break;
    case 4:
    pozitie=2;
    break;  
  }  
}
