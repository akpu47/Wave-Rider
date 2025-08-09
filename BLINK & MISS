// Pins for 5 rows and 5 columns (added pin 22)
int rows[5] = {13, 12, 14, 27, 26}; // Row 4 = paddle
int cols[5] = {25, 33, 32, 23, 22}; // Added column pin 22

// Buttons
int btnLeft = 4;
int btnRight = 15;
int btnReset = 18; // Reset button on pin D18

// Buzzer pin
int buzzerPin = 5; // Choose an available digital pin for buzzer

// Game variables
int ballRow = 0;
int ballCol;
int paddleCol = 1;  // Start paddle in column 1
int lastBallCol = -1;

// Control inversion variables
bool invertControls = false;
unsigned long lastToggleTime = 0;
const unsigned long toggleInterval = 10000; // 10 seconds

// Dance mode variables
bool danceMode = false;
unsigned long lastDanceTrigger = 0;
const unsigned long danceInterval = 25000; // every 25 seconds
const unsigned long danceDuration = 3000;  // dance lasts 3 seconds
unsigned long danceStartTime = 0;

// Lose condition variables
bool lost = false;
unsigned long lostStartTime = 0;   // track when lost started
unsigned long gameStartTime = 0;
const unsigned long loseTimeLimit = 120000; // 2 minutes

// Super Mario buzzer melody globals
const int marioNotes[] = {659, 784, 988}; // E5, G5, B5
const int marioDurations[] = {150, 150, 300};
const int marioNotesCount = 3;
unsigned long marioNoteStartTime = 0;
int marioCurrentNote = 0;
bool marioPlaying = false;

void setup() {
  Serial.begin(115200);
  randomSeed(analogRead(0));

  for (int i = 0; i < 5; i++) pinMode(rows[i], OUTPUT);
  for (int i = 0; i < 5; i++) pinMode(cols[i], OUTPUT);  // 5 columns now

  pinMode(btnLeft, INPUT_PULLUP);
  pinMode(btnRight, INPUT_PULLUP);
  pinMode(btnReset, INPUT_PULLUP);  // Reset button input

  pinMode(buzzerPin, OUTPUT);
  digitalWrite(buzzerPin, LOW);

  allOff();
  spawnBall();
  lastDanceTrigger = millis();
  gameStartTime = millis();
}

void loop() {
  // Check reset button
  if (digitalRead(btnReset) == LOW) { // pressed (active LOW)
    resetGame();
  }

  if (lost) {
    unsigned long now = millis();

    // Blink all LEDs every 300ms
    if ((now / 300) % 2 == 0) {
      allOn();
    } else {
      allOff();
    }

    // Play Mario tune for 2 seconds after lost
    if (now - lostStartTime < 2000) {
      playSuperMarioSoundNonBlocking(now);
    } else {
      noTone(buzzerPin);
      marioPlaying = false; // stop playing melody after 2 sec
    }
    return; // skip rest of game while lost
  }

  unsigned long currentMillis = millis();

  if (currentMillis - gameStartTime > loseTimeLimit) {
    Serial.println("You survived 2 minutes! You win!");
    gameStartTime = currentMillis;
  }

  if (currentMillis - lastToggleTime > toggleInterval) {
    invertControls = !invertControls;
    lastToggleTime = currentMillis;
    Serial.print("Controls inverted? ");
    Serial.println(invertControls ? "YES" : "NO");
  }

  if (!danceMode && currentMillis - lastDanceTrigger > danceInterval) {
    danceMode = true;
    danceStartTime = currentMillis;
    lastDanceTrigger = currentMillis;
    Serial.println("Dance mode started! 💃🕺");
  }

  if (danceMode) {
    randomDance();

    if (currentMillis - danceStartTime > danceDuration) {
      danceMode = false;
      allOff();
      Serial.println("Dance mode ended, back to catching!");
    }
    return;
  }

  handlePaddleMovement();
  displayGame();

  static unsigned long lastFallTime = 0;
  if (currentMillis - lastFallTime > 200) {
    lastFallTime = currentMillis;
    updateBall();
  }
}

// Reset game function
void resetGame() {
  Serial.println("Game reset!");
  lost = false;
  ballRow = 0;
  paddleCol = 1;
  lastBallCol = -1;
  invertControls = false;
  danceMode = false;
  allOff();
  spawnBall();
  gameStartTime = millis();
  lastDanceTrigger = millis();
  lastToggleTime = millis();
  digitalWrite(buzzerPin, LOW);
  marioPlaying = false;
}

// ------------------- GAME LOGIC -------------------

void spawnBall() {
  do {
    ballCol = random(0, 5);  // 5 columns now
  } while (ballCol == lastBallCol);
  lastBallCol = ballCol;
  ballRow = 0;
}

void updateBall() {
  if (ballRow < 4) {
    ballRow++;
  } else {
    if (ballCol == paddleCol) {
      Serial.println("Caught!");
    } else {
      Serial.println("Missed!");
      lost = true;
      lostStartTime = millis();  // Start lost mode timer
      Serial.println("You lost! Game over.");
    }
    spawnBall();
  }
}

void handlePaddleMovement() {
  if (!invertControls) {
    if (digitalRead(btnLeft) == LOW && paddleCol > 0) {
      paddleCol--;
      delay(150);
    }
    if (digitalRead(btnRight) == LOW && paddleCol < 4) {  // max col 4 now
      paddleCol++;
      delay(150);
    }
  } else {
    if (digitalRead(btnLeft) == LOW && paddleCol < 4) {
      paddleCol++;
      delay(150);
    }
    if (digitalRead(btnRight) == LOW && paddleCol > 0) {
      paddleCol--;
      delay(150);
    }
  }
}

// ------------------- DISPLAY -------------------

void displayGame() {
  for (int r = 0; r < 5; r++) {
    allOff();
    digitalWrite(rows[r], LOW);
    for (int c = 0; c < 5; c++) {  // loop through 5 columns
      if (r == ballRow && c == ballCol) digitalWrite(cols[c], HIGH);
      if (r == 4 && c == paddleCol) digitalWrite(cols[c], HIGH);
    }
    delay(2);
  }
}

void allOff() {
  for (int i = 0; i < 5; i++) digitalWrite(rows[i], HIGH);
  for (int i = 0; i < 5; i++) digitalWrite(cols[i], LOW);
}

void allOn() {
  for (int i = 0; i < 5; i++) digitalWrite(rows[i], LOW);
  for (int i = 0; i < 5; i++) digitalWrite(cols[i], HIGH);
}

// ------------------- RANDOM DANCE -------------------

void randomDance() {
  allOff();
  int r = random(0, 5);
  int c = random(0, 5);  // 5 columns
  digitalWrite(rows[r], LOW);
  digitalWrite(cols[c], HIGH);
  delay(200);
}

// ------------------- NON-BLOCKING SUPER MARIO BUZZER SOUND -------------------

void playSuperMarioSoundNonBlocking(unsigned long currentMillis) {
  if (!marioPlaying) {
    // start first note
    marioCurrentNote = 0;
    marioNoteStartTime = currentMillis;
    tone(buzzerPin, marioNotes[marioCurrentNote]);
    marioPlaying = true;
  }

  // Check if current note duration elapsed
  if (currentMillis - marioNoteStartTime >= marioDurations[marioCurrentNote]) {
    noTone(buzzerPin);
    marioCurrentNote++;
    if (marioCurrentNote >= marioNotesCount) {
      marioCurrentNote = 0; // Loop back to first note
    }
    marioNoteStartTime = currentMillis;
    tone(buzzerPin, marioNotes[marioCurrentNote]);
  }
}
