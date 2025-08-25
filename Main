#include <Adafruit_GFX.h>
#include <Adafruit_ILI9341.h>
#include <BfButton.h>

// TFT Display pins
#define TFT_CS 10
#define TFT_RST 9
#define TFT_DC 8
#define TFT_MOSI 11
#define TFT_SCK 13

// Button pins
#define BTN_RIGHT 2
#define BTN_DOWN 3
#define BTN_UP 4
#define BTN_LEFT 12
#define BTN_CONFIRM A4

// Rotary Encoder pins
#define ROT_SW 5
#define ROT_DT 6
#define ROT_CLK 7

// Joystick pins
#define JOY_VRX A0
#define JOY_VRY A1
#define JOY_SW A3

// Other pins
#define BUZZER A2
#define DEBUG_LED A5

// Display object
Adafruit_ILI9341 tft = Adafruit_ILI9341(TFT_CS, TFT_DC, TFT_MOSI, TFT_SCK, TFT_RST);

// Button objects
BfButton btnRight(BfButton::STANDALONE_DIGITAL, BTN_RIGHT, true, LOW);
BfButton btnDown(BfButton::STANDALONE_DIGITAL, BTN_DOWN, true, LOW);
BfButton btnUp(BfButton::STANDALONE_DIGITAL, BTN_UP, true, LOW);
BfButton btnLeft(BfButton::STANDALONE_DIGITAL, BTN_LEFT, true, LOW);
BfButton btnConfirm(BfButton::STANDALONE_DIGITAL, BTN_CONFIRM, true, LOW);
BfButton rotSW(BfButton::STANDALONE_DIGITAL, ROT_SW, true, LOW);

// Game states
enum GameState {
  MAIN_MENU,
  FLAPPY_BIRD,
  DINO_GAME,
  REACTION_TEST,
  PONG,
  SNAKE,
  BREAKOUT,
  MEMORY_GAME,
  MAZE_ESCAPE,
  GAME_2048,
  TETRIS,
  ASTEROIDS,
  RACING,
  PACMAN,
  GALAGA,
  DICE_ROLLER,  // New
  TIC_TAC_TOE   // New
};

GameState currentState = MAIN_MENU;
int menuSelection = 0;
int lastMenuSelection = -1;
const int numGames = 16;  // Changed from 14 to 16
bool needsRedraw = true;

// Game names and descriptions
const char* gameNames[] = {
  "Flappy Bird", "Dino Game", "Reaction Test", "Pong",
  "Snake", "Breakout", "Memory Game", "Maze Escape", 
  "2048", "Tetris", "Asteroids", "Racing", "Pac-Man",
  "Galaga", "Dice Roller", "Tic Tac Toe"  // Added new games
};

const char* gameDescs[] = {
  "Tap to fly through pipes",
  "Jump over obstacles",
  "Test your reflexes",
  "Classic paddle game",
  "Eat food, grow longer",
  "Break all the bricks",
  "Remember the pattern",
  "Find the exit",
  "Merge tiles to 2048",
  "Stack falling blocks",
  "Shoot the asteroids",
  "Dodge the obstacles",
  "Collect dots, avoid ghosts",
  "Shoot alien invaders"
};

// Common game variables
bool gameOver = false;
int score = 0;
unsigned long lastButtonPress = 0;

// Snake game variables
#define SNAKE_SIZE 4
#define GRID_SIZE 26
struct { int x, y; } snake[50];
int snakeLength = 3;
struct { int x, y; } food;
int dx = 1, dy = 0;

// Flappy Bird variables (OPTIMIZED)
float birdY = 120;
float birdVel = 0;
int pipeX[3] = {320, 480, 640};
int pipeGap[3] = {100, 120, 110};
unsigned long lastFlappyUpdate = 0;
bool jumpPressed = false;

// Dino game variables
int dinoY = 150;
int dinoVel = 0;
bool jumping = false;
int obstacleX[2] = {320, 500};
int obstacleSpeed = 10;

// Pong variables
int paddleY = 100;
int aiY = 100;
int ballX = 160, ballY = 120;
int ballDX = 4, ballDY = 3;

// Breakout variables
int breakPaddleX = 140;
int breakBallX = 160, breakBallY = 200;
int breakBallDX = 2, breakBallDY = -2;
uint8_t bricks[8][4];

// Memory game variables
int sequence[20];
int seqLength = 1;
int playerIndex = 0;
bool showing = true;
unsigned long showTime = 0;
int showIndex = 0;

// Maze variables (FIXED)
#define MAZE_W 32
#define MAZE_H 24
uint8_t currentMaze[MAZE_H][MAZE_W];
int playerX = 1, playerY = 1;
bool mazeInitialized = false;

// 2048 variables
int grid2048[4][4];
bool moved2048 = false;

// Tetris variables
uint8_t tetrisGrid[20][10];
int pieceX = 4, pieceY = 0;
int currentPiece = 0;
int rotation = 0;
unsigned long dropTime = 0;
const uint8_t tetrisPieces[7][4][4] = {
  // I piece
  {{0,0,0,0},{1,1,1,1},{0,0,0,0},{0,0,0,0}},
  // O piece
  {{0,0,0,0},{0,1,1,0},{0,1,1,0},{0,0,0,0}},
  // T piece
  {{0,0,0,0},{0,1,0,0},{1,1,1,0},{0,0,0,0}},
  // S piece
  {{0,0,0,0},{0,1,1,0},{1,1,0,0},{0,0,0,0}},
  // Z piece
  {{0,0,0,0},{1,1,0,0},{0,1,1,0},{0,0,0,0}},
  // J piece
  {{0,0,0,0},{1,0,0,0},{1,1,1,0},{0,0,0,0}},
  // L piece
  {{0,0,0,0},{0,0,1,0},{1,1,1,0},{0,0,0,0}}
};

// Asteroids variables
float shipX = 160, shipY = 120;
float shipAngle = 0;
struct { float x, y, vx, vy, angle; int size; bool active; } asteroids[12];
struct { float x, y, vx, vy; bool active; } bullets[8];
bool thrustPressed = false;

// Racing variables
int carX = 160;
int roadOffset = 0;
int obstacles[3] = {320, 420, 520};

// Pac-Man variables (COMPLETELY REWRITTEN)
#define PAC_SCALE 8  // Smaller scale for better performance
#define PAC_MAZE_W 28
#define PAC_MAZE_H 31
float pacX = 14, pacY = 23;  // Using float for smooth movement
float pacSpeed = 0.25;  // Movement speed
int pacDir = 0;  // 0=right, 1=down, 2=left, 3=up
int nextDir = -1;  // Buffered direction
struct Ghost {
  float x, y;
  int dir;
  int mode;  // 0=chase, 1=scatter, 2=frightened, 3=eaten
  int targetX, targetY;
  bool inHouse;
};
Ghost ghosts[4];
uint8_t pacMaze[PAC_MAZE_H][PAC_MAZE_W];
int dotsLeft = 0;
bool powerMode = false;
unsigned long powerModeEnd = 0;
unsigned long modeTimer = 0;
int currentMode = 0;  // 0=scatter, 1=chase
unsigned long lastPacUpdate = 0;
float pacMouthAngle = 0;
bool pacMouthOpening = true;

// Classic Pac-Man maze (simplified)
const uint8_t classicMaze[PAC_MAZE_H][PAC_MAZE_W] PROGMEM = {
  {1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,0,1},
  {1,3,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,3,1},
  {1,0,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,0,1},
  {1,0,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,0,1},
  {1,0,0,0,0,0,0,1,1,0,0,0,0,1,1,0,0,0,0,1,1,0,0,0,0,0,0,1},
  {1,1,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,1,1},
  {1,1,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,1,1},
  {1,1,1,1,1,1,0,1,1,0,0,0,0,0,0,0,0,0,0,1,1,0,1,1,1,1,1,1},
  {1,1,1,1,1,1,0,1,1,0,1,1,1,2,2,1,1,1,0,1,1,0,1,1,1,1,1,1},
  {1,1,1,1,1,1,0,1,1,0,1,2,2,2,2,2,2,1,0,1,1,0,1,1,1,1,1,1},
  {0,0,0,0,0,0,0,0,0,0,1,2,2,2,2,2,2,1,0,0,0,0,0,0,0,0,0,0},
  {1,1,1,1,1,1,0,1,1,0,1,2,2,2,2,2,2,1,0,1,1,0,1,1,1,1,1,1},
  {1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1},
  {1,1,1,1,1,1,0,1,1,0,0,0,0,0,0,0,0,0,0,1,1,0,1,1,1,1,1,1},
  {1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1},
  {1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,0,1},
  {1,0,1,1,1,1,0,1,1,1,1,1,0,1,1,0,1,1,1,1,1,0,1,1,1,1,0,1},
  {1,3,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,3,1},
  {1,1,1,0,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,0,1,1,1},
  {1,1,1,0,1,1,0,1,1,0,1,1,1,1,1,1,1,1,0,1,1,0,1,1,0,1,1,1},
  {1,0,0,0,0,0,0,1,1,0,0,0,0,1,1,0,0,0,0,1,1,0,0,0,0,0,0,1},
  {1,0,1,1,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,1,1,0,1},
  {1,0,1,1,1,1,1,1,1,1,1,1,0,1,1,0,1,1,1,1,1,1,1,1,1,1,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1}
};

// Galaga variables (OPTIMIZED)
int playerShipX = 160;
struct PlayerBullet {
  int x, y;
  bool active;
  int lastY;  // Track last position for efficient clearing
};
PlayerBullet playerBullets[4];  // Reduced for better performance

struct Enemy {
  int x, y, type;
  bool active;
  int lastX, lastY;  // Track last position
};
Enemy enemies[15];  // Reduced enemy count for performance

struct EnemyBullet {
  int x, y;
  bool active;
  int lastY;
};
EnemyBullet enemyBullets[6];  // Reduced for performance

int waveNumber = 1;
unsigned long lastGalagaUpdate = 0;
unsigned long lastEnemyMove = 0;
int enemyMoveDirection = 1;

// 2. Add these helper functions BEFORE initGalaga() (around line 1655):

// Add these forward declarations somewhere before initGalaga() 
// (typically after your variable declarations but before any function definitions)

// Forward declarations for Galaga functions
void drawEnemy(int index);
void clearEnemy(int index);
void updateGalagaPlayer();
void updatePlayerBullets();
void updateEnemies();
void updateEnemyBullets();
void drawGalagaScore();

void setup() {
  Serial.begin(115200);
  
  // Initialize display
  tft.begin();
  tft.setRotation(1);
  tft.fillScreen(ILI9341_BLACK);
  
  // Initialize buttons
  pinMode(BTN_RIGHT, INPUT_PULLUP);
  pinMode(BTN_DOWN, INPUT_PULLUP);
  pinMode(BTN_UP, INPUT_PULLUP);
  pinMode(BTN_LEFT, INPUT_PULLUP);
  pinMode(BTN_CONFIRM, INPUT_PULLUP);
  pinMode(ROT_SW, INPUT_PULLUP);
  pinMode(ROT_CLK, INPUT_PULLUP);
  pinMode(ROT_DT, INPUT_PULLUP);
  
  btnRight.onPress(buttonPressed);
  btnDown.onPress(buttonPressed);
  btnUp.onPress(buttonPressed);
  btnLeft.onPress(buttonPressed);
  btnConfirm.onPress(buttonPressed);
  rotSW.onPress(buttonPressed);
  
  // Initialize other pins
  pinMode(BUZZER, OUTPUT);
  pinMode(DEBUG_LED, OUTPUT);
  
  // Quick startup sound
  tone(BUZZER, 1000, 100);
  delay(150);
  tone(BUZZER, 1500, 100);
  
  // Show splash screen
  showSplash();
  delay(1000);
  
  showMainMenu();
}

void loop() {
  // Read buttons
  btnRight.read();
  btnDown.read();
  btnUp.read();
  btnLeft.read();
  btnConfirm.read();
  rotSW.read();
  
  // Read rotary encoder
  static int lastCLKState = HIGH;
  int clkState = digitalRead(ROT_CLK);
  if (clkState != lastCLKState && clkState == LOW) {
    if (digitalRead(ROT_DT) != clkState) {
      menuSelection = (menuSelection + 1) % numGames;
    } else {
      menuSelection = (menuSelection - 1 + numGames) % numGames;
    }
    if (currentState == MAIN_MENU) needsRedraw = true;
    tone(BUZZER, 1000, 20);
  }
  lastCLKState = clkState;
  
  // Game loop
  switch (currentState) {
    case MAIN_MENU:
      if (needsRedraw || menuSelection != lastMenuSelection) {
        updateMainMenu();
        lastMenuSelection = menuSelection;
        needsRedraw = false;
      }
      break;
      
    case FLAPPY_BIRD:
      updateFlappyBird();
      break;
      
    case DINO_GAME:
      updateDinoGame();
      delay(8);
      break;
      
    case SNAKE:
      updateSnake();
      delay(100);
      break;
      
    case PONG:
      updatePong();
      delay(15);
      break;
      
    case REACTION_TEST:
      updateReactionTest();
      delay(10);
      break;
      
    case BREAKOUT:
      updateBreakout();
      delay(20);
      break;
      
    case MEMORY_GAME:
      updateMemoryGame();
      break;
      
    case MAZE_ESCAPE:
      updateMaze();
      delay(50);
      break;
      
    case GAME_2048:
      update2048();
      delay(50);
      break;
      
    case TETRIS:
      updateTetris();
      break;
      
    case ASTEROIDS:
      updateAsteroids();
      delay(20);
      break;
      
    case RACING:
      updateRacing();
      delay(30);
      break;
      
    case PACMAN:
      updatePacMan();
      break;
      
    case GALAGA:
      updateGalaga();
      break;
      
    case DICE_ROLLER:  // New
      updateDiceRoller();
      delay(10);
      break;
      
    case TIC_TAC_TOE:  // New
      updateTicTacToe();
      delay(10);
      break;
  }
}

void buttonPressed(BfButton *btn, BfButton::press_pattern_t pattern) {
  if (currentState == MAIN_MENU) {
    if (btn == &btnUp) {
      menuSelection = (menuSelection - 1 + numGames) % numGames;
      needsRedraw = true;
      tone(BUZZER, 1000, 20);
    } else if (btn == &btnDown) {
      menuSelection = (menuSelection + 1) % numGames;
      needsRedraw = true;
      tone(BUZZER, 1000, 20);
    } else if (btn == &btnConfirm || btn == &rotSW) {
      tone(BUZZER, 1500, 50);
      startGame(menuSelection);
    }
  } else {
    handleGameInput(btn);
  }
}

// Update your startGame function to handle rotation for Galaga:
void startGame(int index) {
  gameOver = false;
  score = 0;
  
  switch (index) {
    case 0: currentState = FLAPPY_BIRD; initFlappyBird(); break;
    case 1: currentState = DINO_GAME; initDinoGame(); break;
    case 2: currentState = REACTION_TEST; initReactionTest(); break;
    case 3: currentState = PONG; initPong(); break;
    case 4: currentState = SNAKE; initSnake(); break;
    case 5: currentState = BREAKOUT; initBreakout(); break;
    case 6: currentState = MEMORY_GAME; initMemoryGame(); break;
    case 7: currentState = MAZE_ESCAPE; initMaze(); break;
    case 8: currentState = GAME_2048; init2048(); break;
    case 9: currentState = TETRIS; initTetris(); break;
    case 10: currentState = ASTEROIDS; initAsteroids(); break;
    case 11: currentState = RACING; initRacing(); break;
    case 12: currentState = PACMAN; initPacMan(); break;
    case 13: 
      currentState = GALAGA; 
      tft.setRotation(0);  // Switch to portrait mode for Galaga
      initGalaga(); 
      break;
    case 14: currentState = DICE_ROLLER; initDiceRoller(); break;
    case 15: currentState = TIC_TAC_TOE; initTicTacToe(); break;
  }
}

// Update handleGameInput to restore landscape when returning to menu:
void handleGameInput(BfButton *btn) {
  if (btn == &btnLeft) {
    // If we're in Galaga, restore landscape rotation
    if (currentState == GALAGA) {
      tft.setRotation(1);  // Back to landscape
    }
    currentState = MAIN_MENU;
    needsRedraw = true;
    lastMenuSelection = -1;
    return;
  }
  
  if (gameOver && (btn == &btnConfirm || btn == &rotSW)) {
    startGame(getCurrentGameIndex());
    return;
  }
}

int getCurrentGameIndex() {
  switch (currentState) {
    case FLAPPY_BIRD: return 0;
    case DINO_GAME: return 1;
    case REACTION_TEST: return 2;
    case PONG: return 3;
    case SNAKE: return 4;
    case BREAKOUT: return 5;
    case MEMORY_GAME: return 6;
    case MAZE_ESCAPE: return 7;
    case GAME_2048: return 8;
    case TETRIS: return 9;
    case ASTEROIDS: return 10;
    case RACING: return 11;
    case PACMAN: return 12;
    case GALAGA: return 13;
    case DICE_ROLLER: return 14;   // New
    case TIC_TAC_TOE: return 15;   // New
    default: return 0;
  }
}

// UI Functions
void showSplash() {
  tft.fillScreen(ILI9341_BLACK);
  tft.setTextColor(ILI9341_CYAN);
  tft.setTextSize(3);
  tft.setCursor(80, 80);
  tft.print("RETRO");
  tft.setCursor(60, 120);
  tft.print("ARCADE");
  
  // Simple loading bar
  tft.drawRect(60, 180, 200, 10, ILI9341_WHITE);
  for (int i = 0; i < 200; i += 20) {
    tft.fillRect(60 + i, 180, 20, 10, ILI9341_GREEN);
    delay(50);
  }
}

void showMainMenu() {
  tft.fillScreen(ILI9341_BLACK);
  
  tft.setTextColor(ILI9341_CYAN);
  tft.setTextSize(2);
  tft.setCursor(120, 10);
  tft.print("GAMES");
  
  // Show 6 games at a time
  int startIdx = (menuSelection / 6) * 6;
  tft.setTextSize(1);
  
  for (int i = 0; i < 6 && startIdx + i < numGames; i++) {
    int idx = startIdx + i;
    int y = 40 + i * 25;
    
    if (idx == menuSelection) {
      tft.fillRect(10, y - 2, 300, 20, ILI9341_BLUE);
      tft.setTextColor(ILI9341_YELLOW);
    } else {
      tft.setTextColor(ILI9341_WHITE);
    }
    
    tft.setCursor(15, y);
    tft.print("> ");
    tft.print(gameNames[idx]);
  }
  
  // Show description for selected game
  tft.fillRect(10, 190, 300, 20, ILI9341_BLACK);
  tft.setTextColor(ILI9341_GREEN);
  tft.setCursor(10, 195);
  tft.print(gameDescs[menuSelection]);
  
  // Instructions
  tft.setTextColor(ILI9341_WHITE);
  tft.setCursor(10, 215);
  tft.print("UP/DN: Select  A4/RotSW: Play");
  
  // Page indicator
  tft.setCursor(280, 220);
  tft.print(menuSelection + 1);
  tft.print("/");
  tft.print(numGames);
}

void updateMainMenu() {
  // Only update what changed
  static int lastStartIdx = -1;
  int startIdx = (menuSelection / 6) * 6;
  
  if (startIdx != lastStartIdx) {
    // Full redraw needed
    showMainMenu();
    lastStartIdx = startIdx;
  } else {
    // Just update selection
    tft.setTextSize(1);
    
    for (int i = 0; i < 6 && startIdx + i < numGames; i++) {
      int idx = startIdx + i;
      int y = 40 + i * 25;
      
      if (idx == menuSelection) {
        tft.fillRect(10, y - 2, 300, 20, ILI9341_BLUE);
        tft.setTextColor(ILI9341_YELLOW);
      } else {
        tft.fillRect(10, y - 2, 300, 20, ILI9341_BLACK);
        tft.setTextColor(ILI9341_WHITE);
      }
      
      tft.setCursor(15, y);
      tft.print("> ");
      tft.print(gameNames[idx]);
    }
    
    // Update description
    tft.fillRect(10, 190, 300, 20, ILI9341_BLACK);
    tft.setTextColor(ILI9341_GREEN);
    tft.setCursor(10, 195);
    tft.print(gameDescs[menuSelection]);
    
    // Update page indicator
    tft.fillRect(280, 220, 40, 10, ILI9341_BLACK);
    tft.setTextColor(ILI9341_WHITE);
    tft.setCursor(280, 220);
    tft.print(menuSelection + 1);
    tft.print("/");
    tft.print(numGames);
  }
}

void showGameOver() {
  tft.fillRect(80, 80, 160, 80, ILI9341_BLACK);
  tft.drawRect(80, 80, 160, 80, ILI9341_RED);
  
  tft.setTextColor(ILI9341_RED);
  tft.setTextSize(2);
  tft.setCursor(100, 90);
  tft.print("GAME OVER");
  
  tft.setTextColor(ILI9341_WHITE);
  tft.setTextSize(1);
  tft.setCursor(120, 115);
  tft.print("Score: ");
  tft.print(score);
  
  tft.setCursor(95, 140);
  tft.print("A4=Retry L=Menu");
  
  tone(BUZZER, 200, 500);
}

void drawScore() {
  tft.fillRect(5, 5, 80, 15, ILI9341_BLACK);
  tft.setTextColor(ILI9341_WHITE);
  tft.setTextSize(1);
  tft.setCursor(5, 5);
  tft.print("Score:");
  tft.print(score);
}

// FLAPPY BIRD (OPTIMIZED - NO DELAYS)
// FLAPPY BIRD (OPTIMIZED - FASTER PIPES, NO JUMP DELAY)
void initFlappyBird() {
  tft.fillScreen(ILI9341_CYAN);
  tft.fillRect(0, 220, 320, 20, ILI9341_GREEN);
  
  birdY = 120;
  birdVel = 0;
  pipeX[0] = 320;
  pipeX[1] = 420;  // Closer spacing for more challenge
  pipeX[2] = 520;
  pipeGap[0] = random(60, 100);  // Smaller gaps
  pipeGap[1] = random(60, 100);
  pipeGap[2] = random(60, 100);
  lastFlappyUpdate = millis();
  jumpPressed = false;
  
  drawScore();
}

void updateFlappyBird() {
  if (gameOver) return;
  
  unsigned long currentTime = millis();
  if (currentTime - lastFlappyUpdate < 16) return;  // 60 FPS
  lastFlappyUpdate = currentTime;
  
  // Clear bird
  tft.fillRect(50, birdY, 16, 16, ILI9341_CYAN);
  
  // Instant jump response - no delay
  bool currentJump = (digitalRead(BTN_UP) == LOW || 
                     digitalRead(BTN_CONFIRM) == LOW || 
                     digitalRead(ROT_SW) == LOW || 
                     analogRead(JOY_VRY) > 600);
  
  // Jump immediately when button pressed
  if (currentJump) {
    birdVel = -6;  // Stronger jump
    tone(BUZZER, 800, 30);
  }
  
  // Faster gravity for more challenging gameplay
  birdVel += 1.5;  // Increased gravity
  if (birdVel > 15) birdVel = 15;  // Increased max fall speed
  birdY += birdVel;
  
  // Draw bird
  tft.fillRect(50, birdY, 16, 16, ILI9341_YELLOW);
  
  // Update pipes with faster speed
  for (int i = 0; i < 3; i++) {
    // Clear pipe
    if (pipeX[i] < 320 && pipeX[i] > -30) {
      tft.fillRect(pipeX[i], 0, 30, 220, ILI9341_CYAN);
    }
    
    // Move pipe faster
    pipeX[i] -= 8;  // Doubled speed from 4 to 8
    if (pipeX[i] < -30) {
      pipeX[i] = 420 + i * 100;  // Tighter spacing when respawning
      pipeGap[i] = random(50, 90);  // Even smaller gaps as game progresses
      score++;
      drawScore();
    }
    
    // Draw pipe
    if (pipeX[i] < 320 && pipeX[i] > -30) {
      tft.fillRect(pipeX[i], 0, 30, pipeGap[i], ILI9341_GREEN);
      tft.fillRect(pipeX[i], pipeGap[i] + 50, 30, 220 - pipeGap[i] - 50, ILI9341_GREEN);
    }
  }
  
  // Collision
  if (birdY < 0 || birdY > 204) {
    gameOver = true;
    showGameOver();
  }
  
  for (int i = 0; i < 3; i++) {
    if (pipeX[i] < 66 && pipeX[i] > 20 && 
        (birdY < pipeGap[i] || birdY > pipeGap[i] + 34)) {
      gameOver = true;
      showGameOver();
    }
  }
}
/// PAC-MAN (FIXED VERSION - WITH DOTS, 2 GHOSTS, CENTERED MOVEMENT)
void initPacMan() {
  tft.fillScreen(ILI9341_BLACK);
  
  // Copy maze from PROGMEM and count dots
  dotsLeft = 0;
  for (int y = 0; y < PAC_MAZE_H; y++) {
    for (int x = 0; x < PAC_MAZE_W; x++) {
      pacMaze[y][x] = pgm_read_byte(&classicMaze[y][x]);
      if (pacMaze[y][x] == 0) dotsLeft++;
      else if (pacMaze[y][x] == 3) {
        pacMaze[y][x] = 0;  // Power pellet is a dot
        dotsLeft++;
      }
    }
  }
  
  // Draw initial maze with dots
  drawFullMaze();
  
  // Initialize Pac-Man - start at exact grid center
  pacX = 14.0;  // Exact center of grid
  pacY = 23.0;  // Exact center of grid
  pacDir = 0;   // Start facing right
  nextDir = -1;
  pacSpeed = 1.0;  // Move exactly one grid space at a time
  pacMouthAngle = 0;
  pacMouthOpening = true;
  
  // Initialize only 2 ghosts
  ghosts[0] = {14, 11, 2, 1, 14, 11, false};  // Blinky (red)
  ghosts[1] = {14, 14, 3, 1, 14, 14, false};  // Pinky (pink)
  // Set other ghosts to invalid positions (off-screen)
  ghosts[2] = {-10, -10, 0, 0, -10, -10, false};
  ghosts[3] = {-10, -10, 0, 0, -10, -10, false};
  
  // Mode timer
  modeTimer = millis() + 7000;  // 7 seconds scatter mode
  currentMode = 0;
  powerMode = false;
  
  lastPacUpdate = millis();
  drawScore();
}

void drawFullMaze() {
  for (int y = 0; y < PAC_MAZE_H; y++) {
    for (int x = 0; x < PAC_MAZE_W; x++) {
      int px = x * PAC_SCALE + 4;
      int py = y * PAC_SCALE;
      
      if (pacMaze[y][x] == 1) {
        // Wall - draw as lines for better performance
        tft.drawFastHLine(px, py, PAC_SCALE, ILI9341_BLUE);
        tft.drawFastHLine(px, py + PAC_SCALE - 1, PAC_SCALE, ILI9341_BLUE);
        tft.drawFastVLine(px, py, PAC_SCALE, ILI9341_BLUE);
        tft.drawFastVLine(px + PAC_SCALE - 1, py, PAC_SCALE, ILI9341_BLUE);
      } else if (pacMaze[y][x] == 0) {
        // Check if power pellet position
        if ((x == 1 && y == 3) || (x == 26 && y == 3) || 
            (x == 1 && y == 23) || (x == 26 && y == 23)) {
          // Power pellet
          tft.fillCircle(px + PAC_SCALE/2, py + PAC_SCALE/2, 3, ILI9341_WHITE);
        } else {
          // Regular dot
          tft.fillCircle(px + PAC_SCALE/2, py + PAC_SCALE/2, 1, ILI9341_WHITE);
        }
      }
    }
  }
}

// Fixed Pac-Man update with grid-locked movement
void updatePacMan() {
  if (gameOver) return;
  
  unsigned long currentTime = millis();
  if (currentTime - lastPacUpdate < 150) return;  // Slower, more controlled movement
  lastPacUpdate = currentTime;
  
  // Input handling with direction buffering
  int joyX = analogRead(JOY_VRX);
  int joyY = analogRead(JOY_VRY);
  
  if (digitalRead(BTN_UP) == LOW || joyY > 600) nextDir = 3;
  else if (digitalRead(BTN_DOWN) == LOW || joyY < 400) nextDir = 1;
  else if (digitalRead(BTN_LEFT) == LOW || joyX > 600) nextDir = 2;
  else if (digitalRead(BTN_RIGHT) == LOW || joyX < 400) nextDir = 0;
  
  // Only change direction when at exact grid position
  if (isAtGridCenter(pacX, pacY)) {
    if (nextDir != -1 && canMoveGrid((int)pacX, (int)pacY, nextDir)) {
      pacDir = nextDir;
      nextDir = -1;
    }
  }
  
  // Move Pac-Man only if we can move in current direction
  float oldX = pacX;
  float oldY = pacY;
  
  if (canMoveGrid((int)pacX, (int)pacY, pacDir)) {
    switch (pacDir) {
      case 0: pacX += 1.0; break;  // Move exactly one grid space
      case 1: pacY += 1.0; break;
      case 2: pacX -= 1.0; break;
      case 3: pacY -= 1.0; break;
    }
    
    // Tunnel wrap - snap to exact positions
    if (pacX < 0) pacX = PAC_MAZE_W - 1;
    if (pacX >= PAC_MAZE_W) pacX = 0;
  }
  
  // Clear old position if moved
  if (oldX != pacX || oldY != pacY) {
    int oldPx = oldX * PAC_SCALE + 4;
    int oldPy = oldY * PAC_SCALE;
    tft.fillRect(oldPx + 1, oldPy + 1, PAC_SCALE - 2, PAC_SCALE - 2, ILI9341_BLACK);
  }
  
  // Check for dot eating when at exact grid position
  int gridX = (int)round(pacX);
  int gridY = (int)round(pacY);
  
  if (gridX >= 0 && gridX < PAC_MAZE_W && gridY >= 0 && gridY < PAC_MAZE_H) {
    if (pacMaze[gridY][gridX] == 0) {
      pacMaze[gridY][gridX] = 2;  // Mark as empty
      dotsLeft--;
      score += 10;
      drawScore();
      tone(BUZZER, 500, 30);
      
      // Check if power pellet
      bool isPowerPellet = (gridX == 1 && gridY == 3) || (gridX == 26 && gridY == 3) || 
                          (gridX == 1 && gridY == 23) || (gridX == 26 && gridY == 23);
      
      if (isPowerPellet) {
        powerMode = true;
        powerModeEnd = millis() + 6000;
        score += 40;
        drawScore();
        
        // Set first 2 ghosts to frightened
        for (int i = 0; i < 2; i++) {
          if (ghosts[i].x >= 0 && ghosts[i].y >= 0) {
            ghosts[i].mode = 2;
          }
        }
        
        // Power pellet sound
        tone(BUZZER, 800, 100);
        delay(50);
        tone(BUZZER, 1000, 100);
      }
      
      // Check win
      if (dotsLeft == 0) {
        gameOver = true;
        tft.setTextColor(ILI9341_GREEN);
        tft.setTextSize(2);
        tft.setCursor(100, 100);
        tft.print("YOU WIN!");
        tone(BUZZER, 1500, 500);
        return;
      }
    }
  }
  
  // Draw Pac-Man at exact grid center
  int px = pacX * PAC_SCALE + 4;
  int py = pacY * PAC_SCALE;
  
  // Animate mouth
  if (pacMouthOpening) {
    pacMouthAngle += 15;
    if (pacMouthAngle >= 45) pacMouthOpening = false;
  } else {
    pacMouthAngle -= 15;
    if (pacMouthAngle <= 0) pacMouthOpening = true;
  }
  
  // Draw Pac-Man body
  tft.fillCircle(px + PAC_SCALE/2, py + PAC_SCALE/2, PAC_SCALE/2 - 1, ILI9341_YELLOW);
  
  // Draw mouth
  if (pacMouthAngle > 0) {
    switch (pacDir) {
      case 0: // Right
        tft.fillTriangle(px + PAC_SCALE/2, py + PAC_SCALE/2, 
                        px + PAC_SCALE - 1, py + 1, 
                        px + PAC_SCALE - 1, py + PAC_SCALE - 2, ILI9341_BLACK);
        break;
      case 1: // Down
        tft.fillTriangle(px + PAC_SCALE/2, py + PAC_SCALE/2, 
                        px + 1, py + PAC_SCALE - 1, 
                        px + PAC_SCALE - 2, py + PAC_SCALE - 1, ILI9341_BLACK);
        break;
      case 2: // Left
        tft.fillTriangle(px + PAC_SCALE/2, py + PAC_SCALE/2, 
                        px + 1, py + 1, 
                        px + 1, py + PAC_SCALE - 2, ILI9341_BLACK);
        break;
      case 3: // Up
        tft.fillTriangle(px + PAC_SCALE/2, py + PAC_SCALE/2, 
                        px + 1, py + 1, 
                        px + PAC_SCALE - 2, py + 1, ILI9341_BLACK);
        break;
    }
  }
  
  // Update only 2 ghosts
  updateGhosts();
  
  // Check ghost collision (only first 2 ghosts)
  for (int i = 0; i < 2; i++) {
    // Only check ghosts that are on-screen (not at -10, -10)
    if (ghosts[i].x >= 0 && ghosts[i].y >= 0) {
      float dist = abs(pacX - ghosts[i].x) + abs(pacY - ghosts[i].y);
      if (dist < 1.5) {
        gameOver = true;
        showGameOver();
        return;
      }
    }
  }
  
  // End power mode
  if (powerMode && millis() > powerModeEnd) {
    powerMode = false;
    for (int i = 0; i < 2; i++) {
      if (ghosts[i].x >= 0 && ghosts[i].y >= 0 && ghosts[i].mode == 2) {
        ghosts[i].mode = currentMode;
      }
    }
  }
}

// Helper function to check if Pac-Man is at exact grid center
bool isAtGridCenter(float x, float y) {
  return (abs(x - round(x)) < 0.1 && abs(y - round(y)) < 0.1);
}

// Grid-based movement check
bool canMoveGrid(int x, int y, int dir) {
  int newX = x;
  int newY = y;
  
  switch (dir) {
    case 0: newX++; break;  // Right
    case 1: newY++; break;  // Down
    case 2: newX--; break;  // Left
    case 3: newY--; break;  // Up
  }
  
  // Handle tunnel wrap
  if (newX < 0) newX = PAC_MAZE_W - 1;
  if (newX >= PAC_MAZE_W) newX = 0;
  
  // Check bounds
  if (newY < 0 || newY >= PAC_MAZE_H) {
    return false;
  }
  
  // Check if target cell is not a wall
  return (pacMaze[newY][newX] != 1);
}

// Updated ghost function for only 2 ghosts
void updateGhosts() {
  // Only update first 2 ghosts
  for (int i = 0; i < 2; i++) {
    // Skip ghosts that are off-screen
    if (ghosts[i].x < 0 || ghosts[i].y < 0) continue;
    
    // Store old position for clearing
    float oldGhostX = ghosts[i].x;
    float oldGhostY = ghosts[i].y;
    int oldPx = oldGhostX * PAC_SCALE + 4;
    int oldPy = oldGhostY * PAC_SCALE;
    
    // Simple AI: move towards Pac-Man
    float dx = pacX - ghosts[i].x;
    float dy = pacY - ghosts[i].y;
    
    float speed = 0.5;  // Slower ghost movement
    
    // Move towards Pac-Man
    if (abs(dx) > abs(dy)) {
      if (dx > 0 && canMoveGrid((int)ghosts[i].x, (int)ghosts[i].y, 0)) {
        ghosts[i].x += speed;
      } else if (dx < 0 && canMoveGrid((int)ghosts[i].x, (int)ghosts[i].y, 2)) {
        ghosts[i].x -= speed;
      } else if (dy > 0 && canMoveGrid((int)ghosts[i].x, (int)ghosts[i].y, 1)) {
        ghosts[i].y += speed;
      } else if (dy < 0 && canMoveGrid((int)ghosts[i].x, (int)ghosts[i].y, 3)) {
        ghosts[i].y -= speed;
      }
    } else {
      if (dy > 0 && canMoveGrid((int)ghosts[i].x, (int)ghosts[i].y, 1)) {
        ghosts[i].y += speed;
      } else if (dy < 0 && canMoveGrid((int)ghosts[i].x, (int)ghosts[i].y, 3)) {
        ghosts[i].y -= speed;
      } else if (dx > 0 && canMoveGrid((int)ghosts[i].x, (int)ghosts[i].y, 0)) {
        ghosts[i].x += speed;
      } else if (dx < 0 && canMoveGrid((int)ghosts[i].x, (int)ghosts[i].y, 2)) {
        ghosts[i].x -= speed;
      }
    }
    
    // Clear old position
    tft.fillRect(oldPx, oldPy, PAC_SCALE, PAC_SCALE, ILI9341_BLACK);
    
    // Redraw any walls that might have been cleared
    int oldGridX = round(oldGhostX);
    int oldGridY = round(oldGhostY);
    
    if (oldGridX >= 0 && oldGridX < PAC_MAZE_W && oldGridY >= 0 && oldGridY < PAC_MAZE_H) {
      if (pacMaze[oldGridY][oldGridX] == 1) {
        // Redraw wall
        tft.drawFastHLine(oldPx, oldPy, PAC_SCALE, ILI9341_BLUE);
        tft.drawFastHLine(oldPx, oldPy + PAC_SCALE - 1, PAC_SCALE, ILI9341_BLUE);
        tft.drawFastVLine(oldPx, oldPy, PAC_SCALE, ILI9341_BLUE);
        tft.drawFastVLine(oldPx + PAC_SCALE - 1, oldPy, PAC_SCALE, ILI9341_BLUE);
      } else if (pacMaze[oldGridY][oldGridX] == 0) {
        // Redraw dot if it's still there
        bool isPowerPellet = (oldGridX == 1 && oldGridY == 3) || (oldGridX == 26 && oldGridY == 3) || 
                            (oldGridX == 1 && oldGridY == 23) || (oldGridX == 26 && oldGridY == 23);
        if (isPowerPellet) {
          tft.fillCircle(oldPx + PAC_SCALE/2, oldPy + PAC_SCALE/2, 3, ILI9341_WHITE);
        } else {
          tft.fillCircle(oldPx + PAC_SCALE/2, oldPy + PAC_SCALE/2, 1, ILI9341_WHITE);
        }
      }
    }
    
    // Draw ghost
    int px = ghosts[i].x * PAC_SCALE + 4;
    int py = ghosts[i].y * PAC_SCALE;
    
    uint16_t color = (i == 0) ? ILI9341_RED : ILI9341_MAGENTA;
    
    // Draw ghost body
    tft.fillRect(px + 1, py + 1, PAC_SCALE - 2, PAC_SCALE - 2, color);
    
    // Draw eyes
    tft.fillRect(px + 2, py + 2, 2, 2, ILI9341_WHITE);
    tft.fillRect(px + 4, py + 2, 2, 2, ILI9341_WHITE);
  }
}
// DICE ROLLER GAME
// Add these variables at the top with other game variables
int diceValues[2] = {1, 1};
bool rolling = false;
unsigned long rollStartTime = 0;
int rollDuration = 0;
unsigned long lastDiceUpdate = 0;

void initDiceRoller() {
  tft.fillScreen(ILI9341_BLACK);
  
  // Title
  tft.setTextColor(ILI9341_CYAN);
  tft.setTextSize(2);
  tft.setCursor(80, 20);
  tft.print("DICE ROLLER");
  
  // Instructions
  tft.setTextColor(ILI9341_WHITE);
  tft.setTextSize(1);
  tft.setCursor(50, 60);
  tft.print("Press any button to roll!");
  tft.setCursor(90, 200);
  tft.print("L=Menu");
  
  // Draw initial dice
  drawDice(80, 100, diceValues[0]);
  drawDice(180, 100, diceValues[1]);
  
  // Show total
  updateDiceTotal();
}

void updateDiceRoller() {
  // Check for menu exit
  if (digitalRead(BTN_LEFT) == LOW) {
    currentState = MAIN_MENU;
    needsRedraw = true;
    lastMenuSelection = -1;
    return;
  }
  
  // Check for roll input
  if (!rolling && (digitalRead(BTN_UP) == LOW || digitalRead(BTN_DOWN) == LOW ||
      digitalRead(BTN_RIGHT) == LOW || digitalRead(BTN_CONFIRM) == LOW || 
      digitalRead(ROT_SW) == LOW || analogRead(JOY_VRX) < 400 || analogRead(JOY_VRX) > 600 ||
      analogRead(JOY_VRY) < 400 || analogRead(JOY_VRY) > 600)) {
    
    rolling = true;
    rollStartTime = millis();
    rollDuration = random(1000, 2000);  // Roll for 1-2 seconds
    tone(BUZZER, 800, 50);
  }
  
  // Animate rolling
  if (rolling) {
    unsigned long currentTime = millis();
    
    // Update dice animation every 50ms
    if (currentTime - lastDiceUpdate > 50) {
      // Clear old dice
      tft.fillRect(80, 100, 50, 50, ILI9341_BLACK);
      tft.fillRect(180, 100, 50, 50, ILI9341_BLACK);
      
      // Generate random values for animation
      int tempDice1 = random(1, 7);
      int tempDice2 = random(1, 7);
      
      // Draw animated dice
      drawDice(80, 100, tempDice1);
      drawDice(180, 100, tempDice2);
      
      lastDiceUpdate = currentTime;
      
      // Sound effect
      tone(BUZZER, 500 + random(500), 20);
    }
    
    // Check if rolling is done
    if (currentTime - rollStartTime > rollDuration) {
      rolling = false;
      
      // Final values
      diceValues[0] = random(1, 7);
      diceValues[1] = random(1, 7);
      
      // Clear and draw final dice
      tft.fillRect(80, 100, 50, 50, ILI9341_BLACK);
      tft.fillRect(180, 100, 50, 50, ILI9341_BLACK);
      drawDice(80, 100, diceValues[0]);
      drawDice(180, 100, diceValues[1]);
      
      // Update total
      updateDiceTotal();
      
      // Final sound
      tone(BUZZER, 1200, 200);
      
      // Special effects for certain rolls
      int total = diceValues[0] + diceValues[1];
      if (total == 12) {
        // Boxcars!
        tft.setTextColor(ILI9341_YELLOW);
        tft.setTextSize(2);
        tft.setCursor(90, 170);
        tft.print("BOXCARS!");
        tone(BUZZER, 1500, 100);
        delay(100);
        tone(BUZZER, 1500, 100);
      } else if (total == 2) {
        // Snake eyes!
        tft.setTextColor(ILI9341_RED);
        tft.setTextSize(2);
        tft.setCursor(70, 170);
        tft.print("SNAKE EYES!");
        tone(BUZZER, 300, 300);
      } else if (diceValues[0] == diceValues[1]) {
        // Doubles!
        tft.setTextColor(ILI9341_GREEN);
        tft.setTextSize(2);
        tft.setCursor(90, 170);
        tft.print("DOUBLES!");
        tone(BUZZER, 1000, 150);
      }
    }
  }
}

void drawDice(int x, int y, int value) {
  // Draw dice outline
  tft.drawRect(x, y, 50, 50, ILI9341_WHITE);
  tft.fillRect(x + 1, y + 1, 48, 48, ILI9341_BLACK);
  
  // Draw dots based on value
  int dotSize = 4;
  uint16_t dotColor = ILI9341_WHITE;
  
  switch (value) {
    case 1:
      // Center dot
      tft.fillCircle(x + 25, y + 25, dotSize, dotColor);
      break;
    case 2:
      // Top left, bottom right
      tft.fillCircle(x + 15, y + 15, dotSize, dotColor);
      tft.fillCircle(x + 35, y + 35, dotSize, dotColor);
      break;
    case 3:
      // Diagonal line
      tft.fillCircle(x + 15, y + 15, dotSize, dotColor);
      tft.fillCircle(x + 25, y + 25, dotSize, dotColor);
      tft.fillCircle(x + 35, y + 35, dotSize, dotColor);
      break;
    case 4:
      // Four corners
      tft.fillCircle(x + 15, y + 15, dotSize, dotColor);
      tft.fillCircle(x + 35, y + 15, dotSize, dotColor);
      tft.fillCircle(x + 15, y + 35, dotSize, dotColor);
      tft.fillCircle(x + 35, y + 35, dotSize, dotColor);
      break;
    case 5:
      // Four corners plus center
      tft.fillCircle(x + 15, y + 15, dotSize, dotColor);
      tft.fillCircle(x + 35, y + 15, dotSize, dotColor);
      tft.fillCircle(x + 25, y + 25, dotSize, dotColor);
      tft.fillCircle(x + 15, y + 35, dotSize, dotColor);
      tft.fillCircle(x + 35, y + 35, dotSize, dotColor);
      break;
    case 6:
      // Two columns
      tft.fillCircle(x + 15, y + 12, dotSize, dotColor);
      tft.fillCircle(x + 15, y + 25, dotSize, dotColor);
      tft.fillCircle(x + 15, y + 38, dotSize, dotColor);
      tft.fillCircle(x + 35, y + 12, dotSize, dotColor);
      tft.fillCircle(x + 35, y + 25, dotSize, dotColor);
      tft.fillCircle(x + 35, y + 38, dotSize, dotColor);
      break;
  }
}

void updateDiceTotal() {
  // Clear previous total
  tft.fillRect(130, 160, 60, 20, ILI9341_BLACK);
  
  // Show new total
  tft.setTextColor(ILI9341_WHITE);
  tft.setTextSize(2);
  tft.setCursor(130, 160);
  tft.print("= ");
  tft.print(diceValues[0] + diceValues[1]);
}

// TIC TAC TOE GAME
// Add these variables at the top with other game variables
int tttGrid[3][3];  // 0=empty, 1=X, 2=O
int currentPlayer = 1;  // 1=X, 2=O
int cursorX = 1, cursorY = 1;
bool tttGameOver = false;
int winner = 0;  // 0=none, 1=X, 2=O, 3=tie

void initTicTacToe() {
  tft.fillScreen(ILI9341_BLACK);
  
  // Clear grid
  for (int y = 0; y < 3; y++) {
    for (int x = 0; x < 3; x++) {
      tttGrid[y][x] = 0;
    }
  }
  
  currentPlayer = 1;
  cursorX = 1;
  cursorY = 1;
  tttGameOver = false;
  winner = 0;
  
  // Title
  tft.setTextColor(ILI9341_CYAN);
  tft.setTextSize(2);
  tft.setCursor(80, 10);
  tft.print("TIC TAC TOE");
  
  // Draw grid
  drawTTTGrid();
  
  // Show current player
  updateTTTPlayer();
  
  // Instructions
  tft.setTextColor(ILI9341_WHITE);
  tft.setTextSize(1);
  tft.setCursor(50, 210);
  tft.print("Move:Arrows  Place:A4  L:Menu");
}

void updateTicTacToe() {
  if (tttGameOver) {
    // Check for new game
    if (digitalRead(BTN_CONFIRM) == LOW || digitalRead(ROT_SW) == LOW) {
      initTicTacToe();
      return;
    }
  }
  
  // Check for menu exit
  if (digitalRead(BTN_LEFT) == LOW) {
    currentState = MAIN_MENU;
    needsRedraw = true;
    lastMenuSelection = -1;
    return;
  }
  
  if (!tttGameOver) {
    static unsigned long lastMove = 0;
    if (millis() - lastMove > 150) {
      // Clear cursor
      drawTTTCell(cursorX, cursorY, false);
      
      // Move cursor
      int joyX = analogRead(JOY_VRX);
      int joyY = analogRead(JOY_VRY);
      
      if (digitalRead(BTN_UP) == LOW || joyY > 600) {
        cursorY = (cursorY - 1 + 3) % 3;
        lastMove = millis();
      }
      if (digitalRead(BTN_DOWN) == LOW || joyY < 400) {
        cursorY = (cursorY + 1) % 3;
        lastMove = millis();
      }
      if (digitalRead(BTN_LEFT) == LOW || joyX > 600) {
        cursorX = (cursorX - 1 + 3) % 3;
        lastMove = millis();
      }
      if (digitalRead(BTN_RIGHT) == LOW || joyX < 400) {
        cursorX = (cursorX + 1) % 3;
        lastMove = millis();
      }
      
      // Draw cursor
      drawTTTCell(cursorX, cursorY, true);
      
      // Place piece
      if (digitalRead(BTN_CONFIRM) == LOW || digitalRead(ROT_SW) == LOW) {
        if (tttGrid[cursorY][cursorX] == 0) {
          tttGrid[cursorY][cursorX] = currentPlayer;
          drawTTTCell(cursorX, cursorY, false);
          tone(BUZZER, 800, 50);
          
          // Check for winner
          winner = checkTTTWinner();
          if (winner != 0) {
            tttGameOver = true;
            showTTTResult();
          } else {
            // Switch player
            currentPlayer = (currentPlayer == 1) ? 2 : 1;
            updateTTTPlayer();
          }
          
          lastMove = millis();
        } else {
          // Invalid move
          tone(BUZZER, 200, 100);
        }
      }
    }
  }
}

void drawTTTGrid() {
  // Draw grid lines
  int gridX = 80;
  int gridY = 50;
  int cellSize = 50;
  
  // Vertical lines
  tft.drawLine(gridX + cellSize, gridY, gridX + cellSize, gridY + cellSize * 3, ILI9341_WHITE);
  tft.drawLine(gridX + cellSize * 2, gridY, gridX + cellSize * 2, gridY + cellSize * 3, ILI9341_WHITE);
  
  // Horizontal lines
  tft.drawLine(gridX, gridY + cellSize, gridX + cellSize * 3, gridY + cellSize, ILI9341_WHITE);
  tft.drawLine(gridX, gridY + cellSize * 2, gridX + cellSize * 3, gridY + cellSize * 2, ILI9341_WHITE);
  
  // Draw all cells
  for (int y = 0; y < 3; y++) {
    for (int x = 0; x < 3; x++) {
      drawTTTCell(x, y, false);
    }
  }
}

void drawTTTCell(int x, int y, bool highlight) {
  int gridX = 80;
  int gridY = 50;
  int cellSize = 50;
  
  int cellX = gridX + x * cellSize;
  int cellY = gridY + y * cellSize;
  
  // Clear cell
  tft.fillRect(cellX + 2, cellY + 2, cellSize - 4, cellSize - 4, ILI9341_BLACK);
  
  // Draw highlight if cursor
  if (highlight) {
    tft.drawRect(cellX + 5, cellY + 5, cellSize - 10, cellSize - 10, ILI9341_YELLOW);
  }
  
  // Draw X or O
  if (tttGrid[y][x] == 1) {
    // Draw X
    tft.drawLine(cellX + 10, cellY + 10, cellX + cellSize - 10, cellY + cellSize - 10, ILI9341_RED);
    tft.drawLine(cellX + cellSize - 10, cellY + 10, cellX + 10, cellY + cellSize - 10, ILI9341_RED);
    // Thicker X
    tft.drawLine(cellX + 11, cellY + 10, cellX + cellSize - 9, cellY + cellSize - 10, ILI9341_RED);
    tft.drawLine(cellX + cellSize - 11, cellY + 10, cellX + 9, cellY + cellSize - 10, ILI9341_RED);
  } else if (tttGrid[y][x] == 2) {
    // Draw O
    tft.drawCircle(cellX + cellSize/2, cellY + cellSize/2, cellSize/2 - 10, ILI9341_BLUE);
    // Thicker O
    tft.drawCircle(cellX + cellSize/2, cellY + cellSize/2, cellSize/2 - 11, ILI9341_BLUE);
  }
}

void updateTTTPlayer() {
  // Clear previous player indicator
  tft.fillRect(50, 35, 220, 15, ILI9341_BLACK);
  
  // Show current player
  tft.setTextColor(currentPlayer == 1 ? ILI9341_RED : ILI9341_BLUE);
  tft.setTextSize(1);
  tft.setCursor(120, 35);
  tft.print("Player ");
  tft.print(currentPlayer == 1 ? "X" : "O");
  tft.print("'s turn");
}

int checkTTTWinner() {
  // Check rows
  for (int y = 0; y < 3; y++) {
    if (tttGrid[y][0] != 0 && tttGrid[y][0] == tttGrid[y][1] && tttGrid[y][1] == tttGrid[y][2]) {
      return tttGrid[y][0];
    }
  }
  
  // Check columns
  for (int x = 0; x < 3; x++) {
    if (tttGrid[0][x] != 0 && tttGrid[0][x] == tttGrid[1][x] && tttGrid[1][x] == tttGrid[2][x]) {
      return tttGrid[0][x];
    }
  }
  
  // Check diagonals
  if (tttGrid[0][0] != 0 && tttGrid[0][0] == tttGrid[1][1] && tttGrid[1][1] == tttGrid[2][2]) {
    return tttGrid[0][0];
  }
  if (tttGrid[0][2] != 0 && tttGrid[0][2] == tttGrid[1][1] && tttGrid[1][1] == tttGrid[2][0]) {
    return tttGrid[0][2];
  }
  
  // Check for tie
  bool tie = true;
  for (int y = 0; y < 3; y++) {
    for (int x = 0; x < 3; x++) {
      if (tttGrid[y][x] == 0) {
        tie = false;
        break;
      }
    }
  }
  
  if (tie) return 3;  // Tie
  return 0;  // No winner yet
}

void showTTTResult() {
  // Draw winning line if applicable
  if (winner == 1 || winner == 2) {
    int gridX = 80;
    int gridY = 50;
    int cellSize = 50;
    
    // Check which line won
    // Rows
    for (int y = 0; y < 3; y++) {
      if (tttGrid[y][0] == winner && tttGrid[y][1] == winner && tttGrid[y][2] == winner) {
        int lineY = gridY + y * cellSize + cellSize/2;
        tft.drawLine(gridX + 10, lineY, gridX + cellSize * 3 - 10, lineY, ILI9341_GREEN);
        tft.drawLine(gridX + 10, lineY + 1, gridX + cellSize * 3 - 10, lineY + 1, ILI9341_GREEN);
      }
    }
    
    // Columns
    for (int x = 0; x < 3; x++) {
      if (tttGrid[0][x] == winner && tttGrid[1][x] == winner && tttGrid[2][x] == winner) {
        int lineX = gridX + x * cellSize + cellSize/2;
        tft.drawLine(lineX, gridY + 10, lineX, gridY + cellSize * 3 - 10, ILI9341_GREEN);
        tft.drawLine(lineX + 1, gridY + 10, lineX + 1, gridY + cellSize * 3 - 10, ILI9341_GREEN);
      }
    }
    
    // Diagonals
    if (tttGrid[0][0] == winner && tttGrid[1][1] == winner && tttGrid[2][2] == winner) {
      tft.drawLine(gridX + 10, gridY + 10, gridX + cellSize * 3 - 10, gridY + cellSize * 3 - 10, ILI9341_GREEN);
      tft.drawLine(gridX + 11, gridY + 10, gridX + cellSize * 3 - 9, gridY + cellSize * 3 - 10, ILI9341_GREEN);
    }
    if (tttGrid[0][2] == winner && tttGrid[1][1] == winner && tttGrid[2][0] == winner) {
      tft.drawLine(gridX + cellSize * 3 - 10, gridY + 10, gridX + 10, gridY + cellSize * 3 - 10, ILI9341_GREEN);
      tft.drawLine(gridX + cellSize * 3 - 11, gridY + 10, gridX + 9, gridY + cellSize * 3 - 10, ILI9341_GREEN);
    }
  }
  
  // Show result message
  tft.fillRect(70, 205, 180, 30, ILI9341_BLACK);
  tft.setTextSize(2);
  tft.setCursor(90, 210);
  
  if (winner == 3) {
    tft.setTextColor(ILI9341_YELLOW);
    tft.print("IT'S A TIE!");
    tone(BUZZER, 500, 300);
  } else {
    tft.setTextColor(winner == 1 ? ILI9341_RED : ILI9341_BLUE);
    tft.print("PLAYER ");
    tft.print(winner == 1 ? "X" : "O");
    tft.print(" WINS!");
    
    // Victory sound
    tone(BUZZER, 1000, 100);
    delay(150);
    tone(BUZZER, 1200, 100);
    delay(150);
    tone(BUZZER, 1500, 200);
  }
  
  // Show replay prompt
  tft.setTextColor(ILI9341_WHITE);
  tft.setTextSize(1);
  tft.setCursor(80, 230);
  tft.print("A4:New Game  L:Menu");
}

// GALAGA (OPTIMIZED)
// Galaga Portrait Mode - Complete Implementation
// Screen is now 240x320 (portrait) instead of 320x240 (landscape)

void initGalaga() {
  tft.fillScreen(ILI9341_BLACK);
  
  playerShipX = 120;  // Center of 240 width
  
  // Clear bullets
  for (int i = 0; i < 4; i++) {
    playerBullets[i].active = false;
    playerBullets[i].lastY = -1;
  }
  
  for (int i = 0; i < 6; i++) {
    enemyBullets[i].active = false;
    enemyBullets[i].lastY = -1;
  }
  
  // Initialize enemies in portrait layout
  int enemyIndex = 0;
  for (int row = 0; row < 3; row++) {
    for (int col = 0; col < 5; col++) {
      if (enemyIndex < 15) {
        enemies[enemyIndex].active = true;
        enemies[enemyIndex].x = 30 + col * 40;  // Adjusted for 240 width
        enemies[enemyIndex].y = 40 + row * 35;  // More vertical space
        enemies[enemyIndex].type = (row == 0) ? 1 : 0;
        enemies[enemyIndex].lastX = enemies[enemyIndex].x;
        enemies[enemyIndex].lastY = enemies[enemyIndex].y;
        enemyIndex++;
      }
    }
  }
  
  waveNumber = 1;
  lastGalagaUpdate = millis();
  lastEnemyMove = millis();
  enemyMoveDirection = 1;
  
  // Draw initial enemies
  for (int i = 0; i < 15; i++) {
    if (enemies[i].active) {
      drawEnemy(i);
    }
  }
  
  // Draw UI elements
  drawGalagaScore();
  
  // Draw side borders for portrait mode
  tft.drawFastVLine(0, 0, 320, ILI9341_BLUE);
  tft.drawFastVLine(239, 0, 320, ILI9341_BLUE);
}

void updateGalaga() {
  if (gameOver) return;
  
  unsigned long currentTime = millis();
  
  // Faster updates in portrait mode
  if (currentTime - lastGalagaUpdate > 25) {  // ~40 FPS
    updateGalagaPlayer();
    updatePlayerBullets();
    updateEnemyBullets();
    lastGalagaUpdate = currentTime;
  }
  
  // Update enemies less frequently
  if (currentTime - lastEnemyMove > 400) {  // Slightly faster
    updateEnemies();
    lastEnemyMove = currentTime;
  }
}

void updateGalagaPlayer() {
  // Clear old ship position only if moved
  static int lastShipX = -1;
  
  // Player input - FIXED JOYSTICK CONTROLS
  int joyX = analogRead(JOY_VRX);
  int joyY = analogRead(JOY_VRY);
  int newShipX = playerShipX;
  
  // Joystick X-axis control (with deadzone)
  if (joyX < 400) {  // Joystick pushed left
    newShipX = max(15, playerShipX - 6);
  } else if (joyX > 600) {  // Joystick pushed right
    newShipX = min(225, playerShipX + 6);  // Adjusted for 240 width
  }
  
  // Button controls (keep as backup)
  if (digitalRead(BTN_LEFT) == LOW) {
    newShipX = max(15, playerShipX - 6);
  }
  if (digitalRead(BTN_RIGHT) == LOW) {
    newShipX = min(225, playerShipX + 6);
  }
  
  // Only redraw if moved
  if (newShipX != playerShipX) {
    // Clear old position
    if (lastShipX != -1) {
      tft.fillRect(playerShipX - 8, 290, 16, 20, ILI9341_BLACK);
    }
    
    playerShipX = newShipX;
    
    // Draw new position at bottom of portrait screen
    tft.fillTriangle(playerShipX, 290, playerShipX - 8, 310, playerShipX + 8, 310, ILI9341_GREEN);
    tft.fillRect(playerShipX - 2, 285, 4, 6, ILI9341_GREEN);
    
    lastShipX = playerShipX;
  } else if (lastShipX == -1) {
    // Initial draw
    tft.fillTriangle(playerShipX, 290, playerShipX - 8, 310, playerShipX + 8, 310, ILI9341_GREEN);
    tft.fillRect(playerShipX - 2, 285, 4, 6, ILI9341_GREEN);
    lastShipX = playerShipX;
  }
  
  // Fire controls - Joystick Y-axis UP or buttons
  if (joyY > 600 ||  // Joystick pushed up
      digitalRead(BTN_UP) == LOW || 
      digitalRead(BTN_CONFIRM) == LOW || 
      digitalRead(ROT_SW) == LOW ||
      digitalRead(JOY_SW) == LOW) {  // Added joystick button
    
    static unsigned long lastFire = 0;
    if (millis() - lastFire > 150) {  // Faster fire rate
      for (int i = 0; i < 4; i++) {
        if (!playerBullets[i].active) {
          playerBullets[i].x = playerShipX;
          playerBullets[i].y = 285;
          playerBullets[i].active = true;
          playerBullets[i].lastY = 285;
          lastFire = millis();
          tone(BUZZER, 1200, 20);
          break;
        }
      }
    }
  }
}
void updatePlayerBullets() {
  for (int i = 0; i < 4; i++) {
    if (playerBullets[i].active) {
      // Clear old position
      if (playerBullets[i].lastY != -1 && playerBullets[i].lastY < 320) {
        tft.fillRect(playerBullets[i].x - 1, playerBullets[i].lastY, 2, 10, ILI9341_BLACK);
      }
      
      playerBullets[i].lastY = playerBullets[i].y;
      playerBullets[i].y -= 20;  // Even faster bullets in portrait
      
      if (playerBullets[i].y < 0) {
        playerBullets[i].active = false;
        playerBullets[i].lastY = -1;
      } else {
        // Draw new position
        tft.fillRect(playerBullets[i].x - 1, playerBullets[i].y, 2, 10, ILI9341_YELLOW);
        
        // Check collision with enemies
        for (int j = 0; j < 15; j++) {
          if (enemies[j].active) {
            if (abs(playerBullets[i].x - enemies[j].x) < 12 && 
                abs(playerBullets[i].y - enemies[j].y) < 12) {
              // Clear bullet
              tft.fillRect(playerBullets[i].x - 1, playerBullets[i].y, 2, 10, ILI9341_BLACK);
              playerBullets[i].active = false;
              playerBullets[i].lastY = -1;
              
              // Clear enemy
              clearEnemy(j);
              enemies[j].active = false;
              
              score += (enemies[j].type == 1) ? 50 : 20;
              drawGalagaScore();
              tone(BUZZER, 800, 50);
              
              // Simple explosion
              tft.fillCircle(enemies[j].x, enemies[j].y, 8, ILI9341_ORANGE);
              
              break;
            }
          }
        }
      }
    }
  }
}

void updateEnemies() {
  bool moveDown = false;
  bool anyEnemies = false;
  
  // Check boundaries (adjusted for portrait)
  for (int i = 0; i < 15; i++) {
    if (enemies[i].active) {
      anyEnemies = true;
      if ((enemies[i].x <= 15 && enemyMoveDirection == -1) || 
          (enemies[i].x >= 225 && enemyMoveDirection == 1)) {
        moveDown = true;
        break;
      }
    }
  }
  
  // Move enemies
  if (moveDown) {
    enemyMoveDirection *= -1;
    for (int i = 0; i < 15; i++) {
      if (enemies[i].active) {
        clearEnemy(i);
        enemies[i].lastX = enemies[i].x;
        enemies[i].lastY = enemies[i].y;
        enemies[i].y += 25;  // Move down more in portrait
        
        if (enemies[i].y > 260) {  // Adjusted for portrait height
          gameOver = true;
          showGameOver();
          return;
        }
        
        drawEnemy(i);
      }
    }
  } else {
    for (int i = 0; i < 15; i++) {
      if (enemies[i].active) {
        clearEnemy(i);
        enemies[i].lastX = enemies[i].x;
        enemies[i].lastY = enemies[i].y;
        enemies[i].x += enemyMoveDirection * 15;  // Slower horizontal movement
        drawEnemy(i);
        
        // Random enemy fire
        if (random(100) < 3) {  // Less frequent
          for (int j = 0; j < 6; j++) {
            if (!enemyBullets[j].active) {
              enemyBullets[j].x = enemies[i].x;
              enemyBullets[j].y = enemies[i].y + 10;
              enemyBullets[j].active = true;
              enemyBullets[j].lastY = enemies[i].y + 10;
              break;
            }
          }
        }
      }
    }
  }
  
  // Check if wave complete
  if (!anyEnemies) {
    waveNumber++;
    score += 100;
    drawGalagaScore();
    tone(BUZZER, 1500, 300);
    
    // Reinitialize enemies
    int enemyIndex = 0;
    for (int row = 0; row < 3; row++) {
      for (int col = 0; col < 5; col++) {
        if (enemyIndex < 15) {
          enemies[enemyIndex].active = true;
          enemies[enemyIndex].x = 30 + col * 40;
          enemies[enemyIndex].y = 40 + row * 35;
          enemies[enemyIndex].type = (row == 0) ? 1 : 0;
          enemies[enemyIndex].lastX = enemies[enemyIndex].x;
          enemies[enemyIndex].lastY = enemies[enemyIndex].y;
          drawEnemy(enemyIndex);
          enemyIndex++;
        }
      }
    }
  }
}

void updateEnemyBullets() {
  for (int i = 0; i < 6; i++) {
    if (enemyBullets[i].active) {
      // Clear old position
      if (enemyBullets[i].lastY != -1 && enemyBullets[i].lastY < 320) {
        tft.fillRect(enemyBullets[i].x - 1, enemyBullets[i].lastY, 2, 8, ILI9341_BLACK);
      }
      
      enemyBullets[i].lastY = enemyBullets[i].y;
      enemyBullets[i].y += 10;  // Faster in portrait
      
      if (enemyBullets[i].y > 320) {
        enemyBullets[i].active = false;
        enemyBullets[i].lastY = -1;
      } else {
        // Draw new position
        tft.fillRect(enemyBullets[i].x - 1, enemyBullets[i].y, 2, 8, ILI9341_RED);
        
        // Check collision with player
        if (abs(enemyBullets[i].x - playerShipX) < 12 && enemyBullets[i].y > 280) {
          gameOver = true;
          // Player explosion
          for (int j = 0; j < 5; j++) {
            tft.fillCircle(playerShipX, 300, j * 8, ILI9341_RED);
            delay(40);
          }
          showGameOver();
          return;
        }
      }
    }
  }
}
// Add these functions AFTER your updateEnemyBullets() function
// These were missing from your code and causing the compilation error

void drawEnemy(int index) {
  if (!enemies[index].active) return;
  
  uint16_t color = (enemies[index].type == 1) ? ILI9341_MAGENTA : ILI9341_CYAN;
  
  // Draw enemy as a simple invader shape
  int x = enemies[index].x;
  int y = enemies[index].y;
  
  // Body
  tft.fillRect(x - 6, y - 4, 12, 8, color);
  
  // Eyes
  tft.fillRect(x - 4, y - 2, 2, 2, ILI9341_BLACK);
  tft.fillRect(x + 2, y - 2, 2, 2, ILI9341_BLACK);
  
  // Antennae
  tft.drawLine(x - 3, y - 4, x - 3, y - 6, color);
  tft.drawLine(x + 3, y - 4, x + 3, y - 6, color);
}

void clearEnemy(int index) {
  if (enemies[index].lastX != -1 && enemies[index].lastY != -1) {
    tft.fillRect(enemies[index].lastX - 6, enemies[index].lastY - 6, 12, 12, ILI9341_BLACK);
  }
}

// Also add the drawGalagaScore function if it's missing:
void drawGalagaScore() {
  tft.fillRect(5, 5, 100, 15, ILI9341_BLACK);
  tft.setTextColor(ILI9341_WHITE);
  tft.setTextSize(1);
  tft.setCursor(5, 5);
  tft.print("Score:");
  tft.print(score);
  
  // If using portrait mode, also show wave number
  if (currentState == GALAGA) {
    tft.setCursor(180, 5);
    tft.print("Wave:");
    tft.print(waveNumber);
  }
}
// Special score display for Galaga




// Fixed canMove function for Pac-Man
bool canMove(float x, float y, int dir) {
  // Calculate the position we're trying to move to
  float newX = x;
  float newY = y;
  
  // Move by pacSpeed (0.25)
  switch (dir) {
    case 0: newX += pacSpeed; break;  // Right
    case 1: newY += pacSpeed; break;  // Down
    case 2: newX -= pacSpeed; break;  // Left
    case 3: newY -= pacSpeed; break;  // Up
  }
  
  // Check tunnel wrap first
  if (newX < 0 || newX >= PAC_MAZE_W) {
    return true;  // Allow tunnel movement
  }
  
  // Check bounds
  if (newY < 0 || newY >= PAC_MAZE_H) {
    return false;
  }
  
  // Check the grid position we'd be moving into
  int checkX = (int)round(newX);
  int checkY = (int)round(newY);
  
  // Make sure grid coordinates are in bounds
  if (checkX < 0 || checkX >= PAC_MAZE_W || checkY < 0 || checkY >= PAC_MAZE_H) {
    return false;
  }
  
  // Check if the target cell is a wall (1 = wall, 0 = dot, 2 = empty)
  if (pacMaze[checkY][checkX] == 1) {
    return false;
  }
  
  return true;
}
// TRUE MAZE GENERATION - Creates proper maze structure
// Add these structures (replace the old ones)
struct MazeStack {
  int x, y;
};

MazeStack mazeStack[500];
int stackTop = -1;

void pushStack(int x, int y) {
  if (stackTop < 499) {
    stackTop++;
    mazeStack[stackTop].x = x;
    mazeStack[stackTop].y = y;
  }
}

bool popStack(int &x, int &y) {
  if (stackTop >= 0) {
    x = mazeStack[stackTop].x;
    y = mazeStack[stackTop].y;
    stackTop--;
    return true;
  }
  return false;
}

void initMaze() {
  tft.fillScreen(ILI9341_BLACK);
  
  // Show generating message
  tft.setTextColor(ILI9341_CYAN);
  tft.setTextSize(2);
  tft.setCursor(50, 100);
  tft.print("Generating");
  tft.setCursor(70, 120);
  tft.print("True Maze...");
  
  // Generate proper maze
  generateProperMaze();
  
  // Clear screen and draw maze
  tft.fillScreen(ILI9341_BLACK);
  
  playerX = 1;
  playerY = 1;
  
  drawMaze();
  
  // Draw entrance (green border)
  tft.drawRect(0, 0, 12, 12, ILI9341_GREEN);
  tft.drawRect(1, 1, 10, 10, ILI9341_GREEN);
  
  // Draw exit (red with animation)
  int exitX = (MAZE_W-2) * 10;
  int exitY = (MAZE_H-2) * 10;
  tft.fillRect(exitX, exitY, 10, 10, ILI9341_RED);
  tft.drawRect(exitX-1, exitY-1, 12, 12, ILI9341_YELLOW);
  tft.drawRect(exitX-2, exitY-2, 14, 14, ILI9341_RED);
  
  // Draw player
  tft.fillRect(playerX * 10 + 2, playerY * 10 + 2, 6, 6, ILI9341_YELLOW);
  tft.drawRect(playerX * 10 + 1, playerY * 10 + 1, 8, 8, ILI9341_WHITE);
  
  drawScore();
}

void generateProperMaze() {
  // Step 1: Fill everything with walls
  for (int y = 0; y < MAZE_H; y++) {
    for (int x = 0; x < MAZE_W; x++) {
      currentMaze[y][x] = 1;  // All walls
    }
  }
  
  // Step 2: Create the maze structure using recursive backtracking
  // This creates a PERFECT maze (exactly one path between any two points)
  bool visited[MAZE_H][MAZE_W];
  for (int y = 0; y < MAZE_H; y++) {
    for (int x = 0; x < MAZE_W; x++) {
      visited[y][x] = false;
    }
  }
  
  // Start from entrance
  stackTop = -1;
  pushStack(1, 1);
  currentMaze[1][1] = 0;  // Clear entrance
  visited[1][1] = true;
  
  // Generate maze using iterative backtracking
  while (stackTop >= 0) {
    int currentX, currentY;
    popStack(currentX, currentY);
    
    // Find all unvisited neighbors (2 cells away to ensure walls between)
    int neighbors[4][2];
    int neighborCount = 0;
    
    // Check 4 directions: up, right, down, left
    int dirs[4][2] = {{0, -2}, {2, 0}, {0, 2}, {-2, 0}};
    
    for (int i = 0; i < 4; i++) {
      int newX = currentX + dirs[i][0];
      int newY = currentY + dirs[i][1];
      
      // Check if neighbor is within bounds and unvisited
      if (newX > 0 && newX < MAZE_W-1 && 
          newY > 0 && newY < MAZE_H-1 && 
          !visited[newY][newX]) {
        neighbors[neighborCount][0] = newX;
        neighbors[neighborCount][1] = newY;
        neighborCount++;
      }
    }
    
    if (neighborCount > 0) {
      // Put current cell back on stack (we'll return to it)
      pushStack(currentX, currentY);
      
      // Choose random unvisited neighbor
      int chosen = random(neighborCount);
      int nextX = neighbors[chosen][0];
      int nextY = neighbors[chosen][1];
      
      // Remove wall between current and chosen neighbor
      int wallX = currentX + (nextX - currentX) / 2;
      int wallY = currentY + (nextY - currentY) / 2;
      
      // Clear the path
      currentMaze[nextY][nextX] = 0;      // Clear destination
      currentMaze[wallY][wallX] = 0;      // Clear wall between
      visited[nextY][nextX] = true;
      
      // Add chosen neighbor to stack
      pushStack(nextX, nextY);
    }
  }
  
  // Step 3: Ensure exit is reachable
  currentMaze[MAZE_H-2][MAZE_W-2] = 0;
  
  // Step 4: Add strategic complexity while maintaining maze properties
  addMazeComplexity();
  
  // Step 5: Add some dead ends for extra challenge
  addStrategicDeadEnds();
  
  // Step 6: Verify maze is solvable
  if (!isPathToExit()) {
    createEmergencyPath();
  }
}

void addMazeComplexity() {
  // Add a FEW strategic openings to create some loops and alternate paths
  // But not too many - we want to keep the maze-like structure
  
  int maxOpenings = (MAZE_W * MAZE_H) / 25;  // Much fewer openings
  int openingsAdded = 0;
  
  for (int attempts = 0; attempts < maxOpenings * 3 && openingsAdded < maxOpenings; attempts++) {
    int x = 2 + random(MAZE_W - 4);
    int y = 2 + random(MAZE_H - 4);
    
    // Only consider walls
    if (currentMaze[y][x] == 1) {
      // Count adjacent paths
      int pathCount = 0;
      int pathDirs = 0;
      
      if (currentMaze[y-1][x] == 0) { pathCount++; pathDirs |= 1; }  // Up
      if (currentMaze[y+1][x] == 0) { pathCount++; pathDirs |= 2; }  // Down  
      if (currentMaze[y][x-1] == 0) { pathCount++; pathDirs |= 4; }  // Left
      if (currentMaze[y][x+1] == 0) { pathCount++; pathDirs |= 8; }  // Right
      
      // Only open walls that connect exactly 2 paths from different directions
      // This creates strategic loops without destroying the maze structure
      if (pathCount == 2) {
        // Make sure the paths are from different directions (not just straight line)
        if ((pathDirs == 5) || (pathDirs == 10) ||  // Opposite directions
            (pathDirs == 3) || (pathDirs == 6) || (pathDirs == 9) || (pathDirs == 12)) {  // Adjacent directions
          
          // Only 30% chance to actually open it (keep it sparse)
          if (random(100) < 30) {
            currentMaze[y][x] = 0;
            openingsAdded++;
          }
        }
      }
    }
  }
}

void addStrategicDeadEnds() {
  // Add some extra dead-end branches to make the maze more challenging
  // These are short 1-3 cell extensions from existing paths
  
  int deadEndsAdded = 0;
  int maxDeadEnds = 8 + random(5);  // 8-12 additional dead ends
  
  for (int attempts = 0; attempts < maxDeadEnds * 4 && deadEndsAdded < maxDeadEnds; attempts++) {
    // Find a path cell that could have a dead end attached
    int x = 1 + random(MAZE_W - 2);
    int y = 1 + random(MAZE_H - 2);
    
    if (currentMaze[y][x] == 0) {  // Found a path
      // Try to add a dead end in a random direction
      int dirs[4][2] = {{0, -1}, {1, 0}, {0, 1}, {-1, 0}};
      int dir = random(4);
      
      int deadEndLength = 1 + random(3);  // 1-3 cells long
      int currentX = x;
      int currentY = y;
      bool canCreate = true;
      
      // Check if we can create this dead end
      for (int i = 0; i < deadEndLength && canCreate; i++) {
        currentX += dirs[dir][0];
        currentY += dirs[dir][1];
        
        // Make sure we stay in bounds and don't hit existing paths
        if (currentX <= 0 || currentX >= MAZE_W-1 || 
            currentY <= 0 || currentY >= MAZE_H-1 || 
            currentMaze[currentY][currentX] == 0) {
          canCreate = false;
        }
      }
      
      // Create the dead end if possible
      if (canCreate) {
        currentX = x;
        currentY = y;
        
        for (int i = 0; i < deadEndLength; i++) {
          currentX += dirs[dir][0];
          currentY += dirs[dir][1];
          currentMaze[currentY][currentX] = 0;
        }
        
        deadEndsAdded++;
      }
    }
  }
}

bool isPathToExit() {
  // Use flood fill to check if exit is reachable
  bool visited[MAZE_H][MAZE_W];
  for (int y = 0; y < MAZE_H; y++) {
    for (int x = 0; x < MAZE_W; x++) {
      visited[y][x] = false;
    }
  }
  
  stackTop = -1;
  pushStack(1, 1);  // Start position
  visited[1][1] = true;
  
  while (stackTop >= 0) {
    int x, y;
    popStack(x, y);
    
    // Check if we reached the exit
    if (x == MAZE_W-2 && y == MAZE_H-2) {
      return true;
    }
    
    // Check 4 directions
    int dirs[4][2] = {{0, -1}, {1, 0}, {0, 1}, {-1, 0}};
    for (int i = 0; i < 4; i++) {
      int newX = x + dirs[i][0];
      int newY = y + dirs[i][1];
      
      if (newX >= 0 && newX < MAZE_W && newY >= 0 && newY < MAZE_H &&
          !visited[newY][newX] && currentMaze[newY][newX] == 0) {
        visited[newY][newX] = true;
        pushStack(newX, newY);
      }
    }
  }
  
  return false;  // No path found
}

void createEmergencyPath() {
  // Create a simple path to exit if none exists
  // This should rarely be needed with proper maze generation
  
  int x = 1, y = 1;
  int targetX = MAZE_W - 2;
  int targetY = MAZE_H - 2;
  
  // Create a basic path (this is emergency only)
  while (x != targetX || y != targetY) {
    currentMaze[y][x] = 0;
    
    if (x < targetX && random(100) < 70) {
      x++;
    } else if (x > targetX && random(100) < 70) {
      x--;
    } else if (y < targetY) {
      y++;
    } else if (y > targetY) {
      y--;
    }
  }
  
  currentMaze[targetY][targetX] = 0;
}

void drawMaze() {
  // Draw the maze with enhanced visuals
  
  // Draw outer boundary
  tft.drawRect(0, 0, MAZE_W * 10, MAZE_H * 10, ILI9341_CYAN);
  tft.drawRect(1, 1, MAZE_W * 10 - 2, MAZE_H * 10 - 2, ILI9341_BLUE);
  
  // Draw walls efficiently
  for (int y = 0; y < MAZE_H; y++) {
    for (int x = 0; x < MAZE_W; x++) {
      if (currentMaze[y][x] == 1) {  // Wall
        // Draw wall with slight texture
        uint16_t wallColor = ILI9341_WHITE;
        
        // Add some visual variation to walls
        if ((x + y) % 3 == 0) {
          wallColor = 0xDEDB;  // Slightly gray
        } else if ((x + y) % 5 == 0) {
          wallColor = 0xE71C;  // Light gray
        }
        
        tft.fillRect(x * 10, y * 10, 10, 10, wallColor);
        
        // Add corner details for visual appeal
        if (x > 0 && y > 0 && 
            currentMaze[y-1][x] == 1 && currentMaze[y][x-1] == 1) {
          tft.drawPixel(x * 10, y * 10, ILI9341_CYAN);
        }
      } else {
        // Draw path (empty space) 
        tft.fillRect(x * 10, y * 10, 10, 10, ILI9341_BLACK);
        
        // Add subtle path indicators
        if ((x + y) % 8 == 0) {
          tft.drawPixel(x * 10 + 5, y * 10 + 5, 0x2104);  // Very dark gray
        }
      }
    }
  }
}
void updateMaze() {
  if (gameOver) return;
  
  int newX = playerX;
  int newY = playerY;
  
  // Input
  int joyX = analogRead(JOY_VRX);
  int joyY = analogRead(JOY_VRY);
  
  if (digitalRead(BTN_UP) == LOW || joyY > 600) newY--;
  else if (digitalRead(BTN_DOWN) == LOW || joyY < 400) newY++;
  else if (digitalRead(BTN_LEFT) == LOW || joyX > 600) newX--;
  else if (digitalRead(BTN_RIGHT) == LOW || joyX < 400) newX++;
  
  // Check valid move
  if (newX >= 0 && newX < MAZE_W && newY >= 0 && newY < MAZE_H && !currentMaze[newY][newX]) {
    // Clear old position
    tft.fillRect(playerX * 10 + 2, playerY * 10 + 2, 6, 6, ILI9341_BLACK);
    
    playerX = newX;
    playerY = newY;
    score++;
    drawScore();
    
    // Draw new position
    tft.fillRect(playerX * 10 + 2, playerY * 10 + 2, 6, 6, ILI9341_YELLOW);
    
    // Check win
    if (playerX == MAZE_W-2 && playerY == MAZE_H-2) {
      gameOver = true;
      tft.setTextColor(ILI9341_GREEN);
      tft.setTextSize(2);
      tft.setCursor(100, 100);
      tft.print("YOU WIN!");
      tft.setTextSize(1);
      tft.setCursor(95, 140);
      tft.print("A4=New Maze L=Menu");
      tone(BUZZER, 1500, 500);
      
      // Generate new maze for next game
      mazeInitialized = false;
    }
  }
}

// DINO GAME
void initDinoGame() {
  tft.fillScreen(ILI9341_WHITE);
  tft.drawLine(0, 200, 320, 200, ILI9341_BLACK);
  
  dinoY = 180;
  dinoVel = 0;
  jumping = false;
  obstacleX[0] = 320;
  obstacleX[1] = 520;
  obstacleSpeed = 10;
  
  drawScore();
}

void updateDinoGame() {
  if (gameOver) return;
  
  // Clear dino
  tft.fillRect(50, dinoY, 20, 20, ILI9341_WHITE);
  
  // Jump
  if (!jumping && (digitalRead(BTN_UP) == LOW || digitalRead(BTN_CONFIRM) == LOW || 
      digitalRead(ROT_SW) == LOW || analogRead(JOY_VRY) > 600)) {
    jumping = true;
    dinoVel = -25;
    tone(BUZZER, 600, 30);
  }
  
  // Physics
  if (jumping) {
    dinoVel += 4;
    dinoY += dinoVel;
    if (dinoY >= 180) {
      dinoY = 180;
      jumping = false;
    }
  }
  
  // Draw dino
  tft.fillRect(50, dinoY, 20, 20, ILI9341_GREEN);
  
  // Update obstacles
  for (int i = 0; i < 2; i++) {
    // Clear obstacle
    tft.fillRect(obstacleX[i], 160, 20, 40, ILI9341_WHITE);
    
    // Move
    obstacleX[i] -= obstacleSpeed;
    if (obstacleX[i] < -20) {
      obstacleX[i] = 320 + random(100, 200);
      score++;
      drawScore();
      
      // Increase speed
      if (score % 3 == 0 && obstacleSpeed < 30) {
        obstacleSpeed++;
      }
    }
    
    // Draw obstacle
    tft.fillRect(obstacleX[i], 160, 20, 40, ILI9341_RED);
    
    // Collision
    if (obstacleX[i] > 30 && obstacleX[i] < 70 && dinoY > 160) {
      gameOver = true;
      showGameOver();
    }
  }
}

// Keep all other existing game functions (Snake, Pong, Breakout, etc.)
// [Previous implementations remain the same]

// SNAKE
void initSnake() {
  tft.fillScreen(ILI9341_BLACK);
  tft.drawRect(0, 0, 312, 240, ILI9341_WHITE);
  
  snakeLength = 3;
  snake[0] = {10, 10};
  snake[1] = {9, 10};
  snake[2] = {8, 10};
  dx = 1; dy = 0;
  
  food.x = random(GRID_SIZE);
  food.y = random(GRID_SIZE);
  
  for (int i = 0; i < snakeLength; i++) {
    tft.fillRect(snake[i].x * 12, snake[i].y * 12, 10, 10, ILI9341_GREEN);
  }
  tft.fillRect(food.x * 12, food.y * 12, 10, 10, ILI9341_RED);
  
  drawScore();
}

// Fixed Snake update function
void updateSnake() {
  if (gameOver) return;
  
  // Input
  int joyX = analogRead(JOY_VRX);
  int joyY = analogRead(JOY_VRY);
  
  if ((digitalRead(BTN_UP) == LOW || joyY > 600) && dy != 1) { dx = 0; dy = -1; }
  else if ((digitalRead(BTN_DOWN) == LOW || joyY < 400) && dy != -1) { dx = 0; dy = 1; }
  else if ((digitalRead(BTN_LEFT) == LOW || joyX > 600) && dx != 1) { dx = -1; dy = 0; }
  else if ((digitalRead(BTN_RIGHT) == LOW || joyX < 400) && dx != -1) { dx = 1; dy = 0; }
  
  // Move head
  int newX = snake[0].x + dx;
  int newY = snake[0].y + dy;
  
  // Check walls
  if (newX < 0 || newX >= GRID_SIZE || newY < 0 || newY >= GRID_SIZE) {
    gameOver = true;
    showGameOver();
    return;
  }
  
  // Check self collision
  for (int i = 0; i < snakeLength; i++) {
    if (snake[i].x == newX && snake[i].y == newY) {
      gameOver = true;
      showGameOver();
      return;
    }
  }
  
  // Check food
  bool ate = (newX == food.x && newY == food.y);
  
  if (!ate) {
    tft.fillRect(snake[snakeLength-1].x * 12, snake[snakeLength-1].y * 12, 10, 10, ILI9341_BLACK);
  }
  
  // Move body
  for (int i = snakeLength - 1; i > 0; i--) {
    snake[i] = snake[i-1];
  }
  snake[0].x = newX;
  snake[0].y = newY;
  
  // Draw head
  tft.fillRect(snake[0].x * 12, snake[0].y * 12, 10, 10, ILI9341_YELLOW);
  
  if (ate) {
    snakeLength++;
    score += 10;
    drawScore();
    tone(BUZZER, 1200, 100);
    
    // FIXED: Ensure food doesn't spawn on snake
    bool validPosition = false;
    int attempts = 0;
    while (!validPosition && attempts < 100) {
      food.x = random(GRID_SIZE);
      food.y = random(GRID_SIZE);
      
      // Check if food position overlaps with snake
      validPosition = true;
      for (int i = 0; i < snakeLength; i++) {
        if (snake[i].x == food.x && snake[i].y == food.y) {
          validPosition = false;
          break;
        }
      }
      attempts++;
    }
    
    // Draw food with a border to make it more visible
    tft.fillRect(food.x * 12, food.y * 12, 10, 10, ILI9341_RED);
    tft.drawRect(food.x * 12, food.y * 12, 10, 10, ILI9341_WHITE);
  }
  
  // Redraw the body with green color (in case head overwrote it)
  for (int i = 1; i < snakeLength; i++) {
    tft.fillRect(snake[i].x * 12, snake[i].y * 12, 10, 10, ILI9341_GREEN);
  }
}
// PONG
void initPong() {
  tft.fillScreen(ILI9341_BLACK);
  
  for (int y = 0; y < 240; y += 20) {
    tft.drawLine(160, y, 160, y + 10, ILI9341_WHITE);
  }
  
  paddleY = 100;
  aiY = 100;
  ballX = 160;
  ballY = 120;
  ballDX = 5;
  ballDY = 4;
  
  drawScore();
}

void updatePong() {
  if (gameOver) return;
  
  // Clear
  tft.fillRect(10, paddleY, 5, 40, ILI9341_BLACK);
  tft.fillRect(305, aiY, 5, 40, ILI9341_BLACK);
  tft.fillRect(ballX - 5, ballY - 5, 10, 10, ILI9341_BLACK);
  
  // Input
  int joyY = analogRead(JOY_VRY);
  if (digitalRead(BTN_UP) == LOW || joyY > 600) paddleY = max(0, paddleY - 6);
  if (digitalRead(BTN_DOWN) == LOW || joyY < 400) paddleY = min(200, paddleY + 6);
  
  // AI
  if (aiY + 20 < ballY) aiY += 4;
  if (aiY + 20 > ballY) aiY -= 4;
  aiY = constrain(aiY, 0, 200);
  
  // Ball physics
  ballX += ballDX;
  ballY += ballDY;
  
  // Wall collision
  if (ballY <= 5 || ballY >= 235) {
    ballDY = -ballDY;
    tone(BUZZER, 500, 20);
  }
  
  // Paddle collision
  if (ballX <= 15 && ballY >= paddleY && ballY <= paddleY + 40) {
    ballDX = abs(ballDX);
    tone(BUZZER, 800, 50);
  }
  if (ballX >= 300 && ballY >= aiY && ballY <= aiY + 40) {
    ballDX = -abs(ballDX);
    tone(BUZZER, 800, 50);
  }
  
  // Scoring
  if (ballX < 0) {
    ballX = 160;
    ballY = 120;
    ballDX = 5;
  }
  if (ballX > 320) {
    score++;
    drawScore();
    ballX = 160;
    ballY = 120;
    ballDX = -5;
  }
  
  // Draw
  tft.fillRect(10, paddleY, 5, 40, ILI9341_WHITE);
  tft.fillRect(305, aiY, 5, 40, ILI9341_WHITE);
  tft.fillRect(ballX - 5, ballY - 5, 10, 10, ILI9341_WHITE);
}

// BREAKOUT
void initBreakout() {
  tft.fillScreen(ILI9341_BLACK);
  
  // Initialize bricks
  for (int x = 0; x < 8; x++) {
    for (int y = 0; y < 4; y++) {
      bricks[x][y] = 1;
      uint16_t color = ILI9341_RED;
      if (y == 1) color = ILI9341_YELLOW;
      else if (y == 2) color = ILI9341_GREEN;
      else if (y == 3) color = ILI9341_BLUE;
      tft.fillRect(x * 40, y * 15 + 30, 38, 13, color);
    }
  }
  
  breakPaddleX = 140;
  breakBallX = 160;
  breakBallY = 200;
  breakBallDX = 2;
  breakBallDY = -2;
  
  drawScore();
}

void updateBreakout() {
  if (gameOver) return;
  
  // Clear paddle and ball
  tft.fillRect(breakPaddleX, 220, 40, 5, ILI9341_BLACK);
  tft.fillRect(breakBallX - 3, breakBallY - 3, 6, 6, ILI9341_BLACK);
  
  // Fixed joystick controls
  int joyX = analogRead(JOY_VRX);
  if (digitalRead(BTN_LEFT) == LOW || joyX > 600) breakPaddleX = max(0, breakPaddleX - 6);
  if (digitalRead(BTN_RIGHT) == LOW || joyX < 400) breakPaddleX = min(280, breakPaddleX + 6);
  
  // Ball physics
  breakBallX += breakBallDX;
  breakBallY += breakBallDY;
  
  // Wall collisions
  if (breakBallX <= 3 || breakBallX >= 317) breakBallDX = -breakBallDX;
  if (breakBallY <= 3) breakBallDY = -breakBallDY;
  
  // Paddle collision
  if (breakBallY >= 217 && breakBallY <= 220 && 
      breakBallX >= breakPaddleX && breakBallX <= breakPaddleX + 40) {
    breakBallDY = -abs(breakBallDY);
    tone(BUZZER, 1000, 50);
  }
  
  // Brick collisions
  int brickX = breakBallX / 40;
  int brickY = (breakBallY - 30) / 15;
  if (brickX >= 0 && brickX < 8 && brickY >= 0 && brickY < 4) {
    if (bricks[brickX][brickY]) {
      bricks[brickX][brickY] = 0;
      tft.fillRect(brickX * 40, brickY * 15 + 30, 38, 13, ILI9341_BLACK);
      breakBallDY = -breakBallDY;
      score += 10;
      drawScore();
      tone(BUZZER, 1200, 100);
      
      // Check if all bricks broken
      bool allBroken = true;
      for (int x = 0; x < 8; x++) {
        for (int y = 0; y < 4; y++) {
          if (bricks[x][y]) {
            allBroken = false;
            break;
          }
        }
      }
      if (allBroken) {
        gameOver = true;
        tft.setTextColor(ILI9341_GREEN);
        tft.setTextSize(2);
        tft.setCursor(100, 100);
        tft.print("YOU WIN!");
        tone(BUZZER, 1500, 500);
      }
    }
  }
  
  // Game over
  if (breakBallY > 240) {
    gameOver = true;
    showGameOver();
  }
  
  // Draw paddle and ball
  tft.fillRect(breakPaddleX, 220, 40, 5, ILI9341_WHITE);
  tft.fillRect(breakBallX - 3, breakBallY - 3, 6, 6, ILI9341_WHITE);
}

// MEMORY GAME
void initMemoryGame() {
  tft.fillScreen(ILI9341_BLACK);
  
  // Draw buttons in diamond layout
  tft.fillRect(120, 40, 80, 80, ILI9341_GREEN);   // UP
  tft.fillRect(200, 120, 80, 80, ILI9341_RED);    // RIGHT
  tft.fillRect(120, 120, 80, 80, ILI9341_YELLOW); // DOWN
  tft.fillRect(40, 120, 80, 80, ILI9341_BLUE);    // LEFT
  
  seqLength = 1;
  playerIndex = 0;
  showing = true;
  showTime = millis() + 1000;
  showIndex = 0;
  
  sequence[0] = random(4);
  
  tft.setTextColor(ILI9341_WHITE);
  tft.setTextSize(1);
  tft.setCursor(130, 10);
  tft.print("WATCH!");
}

void updateMemoryGame() {
  if (gameOver) return;
  
  if (showing) {
    if (millis() > showTime) {
      if (showIndex > 0) {
        drawMemoryButton(sequence[showIndex - 1], false);
      }
      
      if (showIndex < seqLength) {
        drawMemoryButton(sequence[showIndex], true);
        showTime = millis() + 600;
        showIndex++;
      } else {
        drawMemoryButton(sequence[showIndex - 1], false);
        showing = false;
        playerIndex = 0;
        
        tft.fillRect(130, 10, 60, 10, ILI9341_BLACK);
        tft.setCursor(130, 10);
        tft.print("REPEAT!");
      }
    }
  } else {
    // Player input
    int button = -1;
    if (digitalRead(BTN_UP) == LOW) button = 0;
    else if (digitalRead(BTN_RIGHT) == LOW) button = 1;
    else if (digitalRead(BTN_DOWN) == LOW) button = 2;
    else if (digitalRead(BTN_LEFT) == LOW) button = 3;
    
    if (button >= 0) {
      drawMemoryButton(button, true);
      delay(200);
      drawMemoryButton(button, false);
      
      if (button == sequence[playerIndex]) {
        playerIndex++;
        if (playerIndex >= seqLength) {
          score = seqLength;
          seqLength++;
          sequence[seqLength - 1] = random(4);
          
          tft.fillRect(130, 10, 60, 10, ILI9341_BLACK);
          tft.setCursor(130, 10);
          tft.print("CORRECT!");
          tone(BUZZER, 1500, 200);
          
          delay(1000);
          showing = true;
          showTime = millis() + 500;
          showIndex = 0;
          
          tft.fillRect(130, 10, 60, 10, ILI9341_BLACK);
          tft.setCursor(130, 10);
          tft.print("WATCH!");
        }
      } else {
        gameOver = true;
        tone(BUZZER, 200, 500);
        showGameOver();
      }
      
      delay(200);
    }
  }
}

void drawMemoryButton(int button, bool highlight) {
  uint16_t color;
  int x, y;
  
  switch (button) {
    case 0: // UP
      color = highlight ? ILI9341_WHITE : ILI9341_GREEN;
      x = 120; y = 40;
      if (highlight) tone(BUZZER, 523, 200);
      break;
    case 1: // RIGHT
      color = highlight ? ILI9341_WHITE : ILI9341_RED;
      x = 200; y = 120;
      if (highlight) tone(BUZZER, 659, 200);
      break;
    case 2: // DOWN
      color = highlight ? ILI9341_WHITE : ILI9341_YELLOW;
      x = 120; y = 120;
      if (highlight) tone(BUZZER, 784, 200);
      break;
    case 3: // LEFT
      color = highlight ? ILI9341_WHITE : ILI9341_BLUE;
      x = 40; y = 120;
      if (highlight) tone(BUZZER, 1047, 200);
      break;
  }
  
  tft.fillRect(x, y, 80, 80, color);
}

// 2048 GAME
void init2048() {
  tft.fillScreen(ILI9341_BLACK);
  
  // Initialize grid
  for (int y = 0; y < 4; y++) {
    for (int x = 0; x < 4; x++) {
      grid2048[y][x] = 0;
    }
  }
  
  // Add two starting tiles
  addNewTile();
  addNewTile();
  
  draw2048Grid();
  drawScore();
}

void update2048() {
  if (gameOver) return;
  
  int oldGrid[4][4];
  memcpy(oldGrid, grid2048, sizeof(grid2048));
  
  // Joystick controls
  int joyX = analogRead(JOY_VRX);
  int joyY = analogRead(JOY_VRY);
  
  bool moved = false;
  
  if (digitalRead(BTN_LEFT) == LOW || joyX > 600) {
    moved = moveLeft2048();
  } else if (digitalRead(BTN_RIGHT) == LOW || joyX < 400) {
    moved = moveRight2048();
  } else if (digitalRead(BTN_UP) == LOW || joyY > 600) {
    moved = moveUp2048();
  } else if (digitalRead(BTN_DOWN) == LOW || joyY < 400) {
    moved = moveDown2048();
  }
  
  if (moved) {
    addNewTile();
    draw2048Grid();
    drawScore();
    
    // Check for 2048
    for (int y = 0; y < 4; y++) {
      for (int x = 0; x < 4; x++) {
        if (grid2048[y][x] == 2048) {
          gameOver = true;
          tft.setTextColor(ILI9341_GREEN);
          tft.setTextSize(2);
          tft.setCursor(100, 100);
          tft.print("YOU WIN!");
          tone(BUZZER, 1500, 500);
          return;
        }
      }
    }
    
    // Check game over
    if (!canMove2048()) {
      gameOver = true;
      showGameOver();
    }
    
    delay(150); // Prevent too fast moves
  }
}

void addNewTile() {
  int empty[16][2];
  int emptyCount = 0;
  
  for (int y = 0; y < 4; y++) {
    for (int x = 0; x < 4; x++) {
      if (grid2048[y][x] == 0) {
        empty[emptyCount][0] = x;
        empty[emptyCount][1] = y;
        emptyCount++;
      }
    }
  }
  
  if (emptyCount > 0) {
    int idx = random(emptyCount);
    grid2048[empty[idx][1]][empty[idx][0]] = (random(10) < 9) ? 2 : 4;
  }
}

void draw2048Grid() {
  for (int y = 0; y < 4; y++) {
    for (int x = 0; x < 4; x++) {
      int px = 10 + x * 75;
      int py = 30 + y * 50;
      
      tft.fillRect(px, py, 70, 45, ILI9341_BLACK);
      tft.drawRect(px, py, 70, 45, ILI9341_WHITE);
      
      if (grid2048[y][x] > 0) {
        uint16_t color = ILI9341_WHITE;
        if (grid2048[y][x] == 2) color = ILI9341_LIGHTGREY;
        else if (grid2048[y][x] == 4) color = ILI9341_DARKGREY;
        else if (grid2048[y][x] == 8) color = ILI9341_ORANGE;
        else if (grid2048[y][x] == 16) color = ILI9341_RED;
        else if (grid2048[y][x] == 32) color = ILI9341_MAGENTA;
        else if (grid2048[y][x] == 64) color = ILI9341_PINK;
        else if (grid2048[y][x] >= 128) color = ILI9341_YELLOW;
        
        tft.setTextColor(color);
        tft.setTextSize(2);
        tft.setCursor(px + 20, py + 15);
        tft.print(grid2048[y][x]);
      }
    }
  }
}

bool moveLeft2048() {
  bool moved = false;
  for (int y = 0; y < 4; y++) {
    int writePos = 0;
    for (int x = 0; x < 4; x++) {
      if (grid2048[y][x] != 0) {
        if (writePos != x) {
          grid2048[y][writePos] = grid2048[y][x];
          grid2048[y][x] = 0;
          moved = true;
        }
        writePos++;
      }
    }
    
    for (int x = 0; x < 3; x++) {
      if (grid2048[y][x] != 0 && grid2048[y][x] == grid2048[y][x + 1]) {
        grid2048[y][x] *= 2;
        score += grid2048[y][x];
        grid2048[y][x + 1] = 0;
        moved = true;
      }
    }
    
    writePos = 0;
    for (int x = 0; x < 4; x++) {
      if (grid2048[y][x] != 0) {
        if (writePos != x) {
          grid2048[y][writePos] = grid2048[y][x];
          grid2048[y][x] = 0;
        }
        writePos++;
      }
    }
  }
  return moved;
}

bool moveRight2048() {
  bool moved = false;
  for (int y = 0; y < 4; y++) {
    int writePos = 3;
    for (int x = 3; x >= 0; x--) {
      if (grid2048[y][x] != 0) {
        if (writePos != x) {
          grid2048[y][writePos] = grid2048[y][x];
          grid2048[y][x] = 0;
          moved = true;
        }
        writePos--;
      }
    }
    
    for (int x = 3; x > 0; x--) {
      if (grid2048[y][x] != 0 && grid2048[y][x] == grid2048[y][x - 1]) {
        grid2048[y][x] *= 2;
        score += grid2048[y][x];
        grid2048[y][x - 1] = 0;
        moved = true;
      }
    }
    
    writePos = 3;
    for (int x = 3; x >= 0; x--) {
      if (grid2048[y][x] != 0) {
        if (writePos != x) {
          grid2048[y][writePos] = grid2048[y][x];
          grid2048[y][x] = 0;
        }
        writePos--;
      }
    }
  }
  return moved;
}

bool moveUp2048() {
  bool moved = false;
  for (int x = 0; x < 4; x++) {
    int writePos = 0;
    for (int y = 0; y < 4; y++) {
      if (grid2048[y][x] != 0) {
        if (writePos != y) {
          grid2048[writePos][x] = grid2048[y][x];
          grid2048[y][x] = 0;
          moved = true;
        }
        writePos++;
      }
    }
    
    for (int y = 0; y < 3; y++) {
      if (grid2048[y][x] != 0 && grid2048[y][x] == grid2048[y + 1][x]) {
        grid2048[y][x] *= 2;
        score += grid2048[y][x];
        grid2048[y + 1][x] = 0;
        moved = true;
      }
    }
    
    writePos = 0;
    for (int y = 0; y < 4; y++) {
      if (grid2048[y][x] != 0) {
        if (writePos != y) {
          grid2048[writePos][x] = grid2048[y][x];
          grid2048[y][x] = 0;
        }
        writePos++;
      }
    }
  }
  return moved;
}

bool moveDown2048() {
  bool moved = false;
  for (int x = 0; x < 4; x++) {
    int writePos = 3;
    for (int y = 3; y >= 0; y--) {
      if (grid2048[y][x] != 0) {
        if (writePos != y) {
          grid2048[writePos][x] = grid2048[y][x];
          grid2048[y][x] = 0;
          moved = true;
        }
        writePos--;
      }
    }
    
    for (int y = 3; y > 0; y--) {
      if (grid2048[y][x] != 0 && grid2048[y][x] == grid2048[y - 1][x]) {
        grid2048[y][x] *= 2;
        score += grid2048[y][x];
        grid2048[y - 1][x] = 0;
        moved = true;
      }
    }
    
    writePos = 3;
    for (int y = 3; y >= 0; y--) {
      if (grid2048[y][x] != 0) {
        if (writePos != y) {
          grid2048[writePos][x] = grid2048[y][x];
          grid2048[y][x] = 0;
        }
        writePos--;
      }
    }
  }
  return moved;
}

bool canMove2048() {
  // Check for empty spaces
  for (int y = 0; y < 4; y++) {
    for (int x = 0; x < 4; x++) {
      if (grid2048[y][x] == 0) return true;
    }
  }
  
  // Check for possible merges
  for (int y = 0; y < 4; y++) {
    for (int x = 0; x < 3; x++) {
      if (grid2048[y][x] == grid2048[y][x + 1]) return true;
    }
  }
  
  for (int x = 0; x < 4; x++) {
    for (int y = 0; y < 3; y++) {
      if (grid2048[y][x] == grid2048[y + 1][x]) return true;
    }
  }
  
  return false;
}

// TETRIS
void initTetris() {
  tft.fillScreen(ILI9341_BLACK);
  
  // Clear grid
  for (int y = 0; y < 20; y++) {
    for (int x = 0; x < 10; x++) {
      tetrisGrid[y][x] = 0;
    }
  }
  
  // Draw borders
  tft.drawRect(99, 19, 122, 202, ILI9341_WHITE);
  
  spawnNewPiece();
  dropTime = millis();
  
  drawScore();
}

void updateTetris() {
  if (gameOver) return;
  
  // Input handling
  static unsigned long lastMove = 0;
  if (millis() - lastMove > 100) {
    int joyX = analogRead(JOY_VRX);
    
    if (digitalRead(BTN_LEFT) == LOW || joyX > 600) {
      if (canPlacePiece(pieceX - 1, pieceY, rotation)) {
        clearPiece();
        pieceX--;
        drawPiece();
        lastMove = millis();
      }
    }
    if (digitalRead(BTN_RIGHT) == LOW || joyX < 400) {
      if (canPlacePiece(pieceX + 1, pieceY, rotation)) {
        clearPiece();
        pieceX++;
        drawPiece();
        lastMove = millis();
      }
    }
    if (digitalRead(BTN_UP) == LOW || digitalRead(BTN_CONFIRM) == LOW) {
      int newRot = (rotation + 1) % 4;
      if (canPlacePiece(pieceX, pieceY, newRot)) {
        clearPiece();
        rotation = newRot;
        drawPiece();
        lastMove = millis();
      }
    }
  }
  
  // Faster drop
  bool fastDrop = digitalRead(BTN_DOWN) == LOW || analogRead(JOY_VRY) < 400;
  int dropDelay = fastDrop ? 50 : 300;
  
  if (millis() - dropTime > dropDelay) {
    if (canPlacePiece(pieceX, pieceY + 1, rotation)) {
      clearPiece();
      pieceY++;
      drawPiece();
    } else {
      lockPiece();
      clearLines();
      spawnNewPiece();
      
      if (!canPlacePiece(pieceX, pieceY, rotation)) {
        gameOver = true;
        showGameOver();
      }
    }
    dropTime = millis();
  }
}

void spawnNewPiece() {
  pieceX = 3;
  pieceY = 0;
  currentPiece = random(7);
  rotation = 0;
}

bool canPlacePiece(int x, int y, int rot) {
  for (int py = 0; py < 4; py++) {
    for (int px = 0; px < 4; px++) {
      if (getPieceBlock(currentPiece, rot, px, py)) {
        int boardX = x + px;
        int boardY = y + py;
        
        if (boardX < 0 || boardX >= 10 || boardY >= 20) return false;
        if (boardY >= 0 && tetrisGrid[boardY][boardX]) return false;
      }
    }
  }
  return true;
}

bool getPieceBlock(int piece, int rot, int x, int y) {
  switch (rot) {
    case 0: return tetrisPieces[piece][y][x];
    case 1: return tetrisPieces[piece][3-x][y];
    case 2: return tetrisPieces[piece][3-y][3-x];
    case 3: return tetrisPieces[piece][x][3-y];
  }
  return false;
}

void clearPiece() {
  for (int py = 0; py < 4; py++) {
    for (int px = 0; px < 4; px++) {
      if (getPieceBlock(currentPiece, rotation, px, py)) {
        int screenX = 100 + (pieceX + px) * 12;
        int screenY = 20 + (pieceY + py) * 10;
        if (pieceY + py >= 0) {
          tft.fillRect(screenX, screenY, 11, 9, ILI9341_BLACK);
        }
      }
    }
  }
}

void drawPiece() {
  uint16_t color = ILI9341_CYAN;
  if (currentPiece == 1) color = ILI9341_YELLOW;
  else if (currentPiece == 2) color = ILI9341_MAGENTA;
  else if (currentPiece == 3) color = ILI9341_GREEN;
  else if (currentPiece == 4) color = ILI9341_RED;
  else if (currentPiece == 5) color = ILI9341_BLUE;
  else if (currentPiece == 6) color = ILI9341_ORANGE;
  
  for (int py = 0; py < 4; py++) {
    for (int px = 0; px < 4; px++) {
      if (getPieceBlock(currentPiece, rotation, px, py)) {
        int screenX = 100 + (pieceX + px) * 12;
        int screenY = 20 + (pieceY + py) * 10;
        if (pieceY + py >= 0) {
          tft.fillRect(screenX, screenY, 11, 9, color);
        }
      }
    }
  }
}

void lockPiece() {
  for (int py = 0; py < 4; py++) {
    for (int px = 0; px < 4; px++) {
      if (getPieceBlock(currentPiece, rotation, px, py)) {
        int boardX = pieceX + px;
        int boardY = pieceY + py;
        if (boardY >= 0) {
          tetrisGrid[boardY][boardX] = currentPiece + 1;
        }
      }
    }
  }
  tone(BUZZER, 500, 50);
}

void clearLines() {
  int linesCleared = 0;
  
  for (int y = 19; y >= 0; y--) {
    bool full = true;
    for (int x = 0; x < 10; x++) {
      if (!tetrisGrid[y][x]) {
        full = false;
        break;
      }
    }
    
    if (full) {
      // Move lines down
      for (int moveY = y; moveY > 0; moveY--) {
        for (int x = 0; x < 10; x++) {
          tetrisGrid[moveY][x] = tetrisGrid[moveY - 1][x];
        }
      }
      
      // Clear top line
      for (int x = 0; x < 10; x++) {
        tetrisGrid[0][x] = 0;
      }
      
      linesCleared++;
      y++; // Check same line again
    }
  }
  
  if (linesCleared > 0) {
    // Redraw entire grid
    tft.fillRect(100, 20, 120, 200, ILI9341_BLACK);
    for (int y = 0; y < 20; y++) {
      for (int x = 0; x < 10; x++) {
        if (tetrisGrid[y][x]) {
          uint16_t color = ILI9341_CYAN;
          int piece = tetrisGrid[y][x] - 1;
          if (piece == 1) color = ILI9341_YELLOW;
          else if (piece == 2) color = ILI9341_MAGENTA;
          else if (piece == 3) color = ILI9341_GREEN;
          else if (piece == 4) color = ILI9341_RED;
          else if (piece == 5) color = ILI9341_BLUE;
          else if (piece == 6) color = ILI9341_ORANGE;
          
          tft.fillRect(100 + x * 12, 20 + y * 10, 11, 9, color);
        }
      }
    }
    
    score += linesCleared * 100;
    drawScore();
    tone(BUZZER, 1500, 200);
  }
}

// RACING (ENHANCED - Progressive difficulty)
void initRacing() {
  tft.fillScreen(ILI9341_BLACK);
  
  // Draw track borders
  tft.fillRect(40, 0, 20, 240, ILI9341_WHITE);
  tft.fillRect(260, 0, 20, 240, ILI9341_WHITE);
  
  carX = 160;
  roadOffset = 0;
  
  for (int i = 0; i < 3; i++) {
    obstacles[i] = -100 - i * 150;
  }
  
  drawScore();
}

void updateRacing() {
  if (gameOver) return;
  
  // Clear car
  tft.fillRect(carX - 10, 200, 20, 30, ILI9341_BLACK);
  
  // Input
  int joyX = analogRead(JOY_VRX);
  if (digitalRead(BTN_LEFT) == LOW || joyX > 600) carX = max(70, carX - 5);
  if (digitalRead(BTN_RIGHT) == LOW || joyX < 400) carX = min(250, carX + 5);
  
  // Road markings
  roadOffset = (roadOffset + 8) % 60;
  for (int y = roadOffset - 60; y < 240; y += 60) {
    tft.fillRect(158, y, 4, 30, ILI9341_WHITE);
  }
  
  // Obstacles with progressive difficulty
  int baseSpeed = 6;
  int speedIncrease = score / 10; // Increase speed every 10 points
  int currentSpeed = baseSpeed + speedIncrease;
  
  for (int i = 0; i < 3; i++) {
    // Clear old
    if (obstacles[i] > -30 && obstacles[i] < 270) {
      tft.fillRect(100 + i * 60, obstacles[i], 20, 30, ILI9341_BLACK);
    }
    
    obstacles[i] += currentSpeed;
    
    if (obstacles[i] > 240) {
      obstacles[i] = -100;
      score++;
      drawScore();
    }
    
    // Draw
    if (obstacles[i] > -30 && obstacles[i] < 240) {
      int obsX = 100 + i * 60;
      tft.fillRect(obsX, obstacles[i], 20, 30, ILI9341_RED);
      
      // Collision
      if (abs(carX - obsX - 10) < 20 && obstacles[i] > 170 && obstacles[i] < 230) {
        gameOver = true;
        showGameOver();
      }
    }
  }
  
  // Draw car
  tft.fillRect(carX - 10, 200, 20, 30, ILI9341_GREEN);
}

// REACTION TEST
void initReactionTest() {
  tft.fillScreen(ILI9341_BLACK);
  tft.setTextColor(ILI9341_WHITE);
  tft.setTextSize(2);
  tft.setCursor(80, 80);
  tft.print("REACTION");
  tft.setCursor(100, 110);
  tft.print("TEST");
  tft.setTextSize(1);
  tft.setCursor(70, 150);
  tft.print("Wait for GREEN...");
  tft.setCursor(60, 170);
  tft.print("Then press any button!");
  tft.setCursor(90, 200);
  tft.print("L=Menu anytime");
  
  // Draw red stoplight box
  tft.fillRect(140, 100, 40, 40, ILI9341_RED);
}

void updateReactionTest() {
  static unsigned long startTime = 0;
  static bool waiting = false;
  static bool started = false;
  
  // Check for menu exit at any time
  if (digitalRead(BTN_LEFT) == LOW) {
    currentState = MAIN_MENU;
    needsRedraw = true;
    lastMenuSelection = -1;
    started = false;
    waiting = false;
    return;
  }
  
  if (!started) {
    startTime = millis() + random(2000, 5000);
    started = true;
  }
  
  if (!waiting && millis() > startTime) {
    // Change red box to green
    tft.fillRect(140, 100, 40, 40, ILI9341_GREEN);
    tft.setTextColor(ILI9341_WHITE);
    tft.setTextSize(2);
    tft.setCursor(120, 150);
    tft.print("PRESS!");
    waiting = true;
    startTime = millis();
  }
  
  if (waiting) {
    if (digitalRead(BTN_UP) == LOW || digitalRead(BTN_DOWN) == LOW ||
        digitalRead(BTN_RIGHT) == LOW || digitalRead(BTN_CONFIRM) == LOW || 
        digitalRead(ROT_SW) == LOW) {
      
      int reactionTime = millis() - startTime;
      tft.fillScreen(ILI9341_BLACK);
      tft.setTextColor(ILI9341_YELLOW);
      tft.setTextSize(2);
      tft.setCursor(100, 80);
      tft.print(reactionTime);
      tft.print(" ms");
      
      tft.setTextSize(1);
      tft.setTextColor(ILI9341_WHITE);
      tft.setCursor(90, 150);
      tft.print("A4: Again  L: Menu");
      
      gameOver = true;
      started = false;
      waiting = false;
      tone(BUZZER, 1500, 200);
    }
  }
}
// ASTEROIDS GAME IMPLEMENTATION
// Add this code to your Arduino sketch after the other game implementations

void initAsteroids() {
  tft.fillScreen(ILI9341_BLACK);
  
  shipX = 160;
  shipY = 120;
  shipAngle = 0;
  
  // Initialize more asteroids
  for (int i = 0; i < 12; i++) {
    asteroids[i].active = (i < 8);
    asteroids[i].x = random(320);
    asteroids[i].y = random(240);
    // Make sure they don't spawn too close to ship
    while (abs(asteroids[i].x - shipX) < 50 && abs(asteroids[i].y - shipY) < 50) {
      asteroids[i].x = random(320);
      asteroids[i].y = random(240);
    }
    asteroids[i].vx = random(-3, 4);
    asteroids[i].vy = random(-3, 4);
    asteroids[i].angle = random(360) * PI / 180;
    asteroids[i].size = 20;
  }
  
  // Clear bullets
  for (int i = 0; i < 8; i++) {
    bullets[i].active = false;
  }
  
  drawScore();
}

void updateAsteroids() {
  if (gameOver) return;
  
  // Clear ship
  drawShip(ILI9341_BLACK);
  
  // Ship controls - ship stays centered, joystick aims
  int joyX = analogRead(JOY_VRX);
  int joyY = analogRead(JOY_VRY);
  
  // Calculate aim angle from joystick position
  float joyCenterX = 512; // Center of joystick
  float joyCenterY = 512;
  float joyRange = 512; // Range of joystick movement
  
  if (abs(joyX - joyCenterX) > 50 || abs(joyY - joyCenterY) > 50) {
    // Calculate angle from joystick position
    float dx = (joyX - joyCenterX) / joyRange;
    float dy = (joyY - joyCenterY) / joyRange;
    shipAngle = atan2(dy, dx);
  }
  
  // Ship stays in center - no thrust movement
  thrustPressed = false;
  
  // Fire with UP, CONFIRM, or ROT_SW button
  if (digitalRead(BTN_UP) == LOW || digitalRead(BTN_CONFIRM) == LOW || digitalRead(ROT_SW) == LOW) {
    static unsigned long lastFire = 0;
    if (millis() - lastFire > 150) {
      for (int i = 0; i < 8; i++) {
        if (!bullets[i].active) {
          bullets[i].x = shipX;
          bullets[i].y = shipY;
          bullets[i].vx = cos(shipAngle) * 10;
          bullets[i].vy = sin(shipAngle) * 10;
          bullets[i].active = true;
          lastFire = millis();
          tone(BUZZER, 1000, 50);
          break;
        }
      }
    }
  }
  
  // Draw ship
  drawShip(ILI9341_WHITE);
  
  // Update bullets
  for (int i = 0; i < 8; i++) {
    if (bullets[i].active) {
      // Clear old bullet
      tft.fillCircle(bullets[i].x, bullets[i].y, 2, ILI9341_BLACK);
      
      // Move bullet
      bullets[i].x += bullets[i].vx;
      bullets[i].y += bullets[i].vy;
      
      // Wrap bullets
      if (bullets[i].x < 0) bullets[i].x = 320;
      if (bullets[i].x > 320) bullets[i].x = 0;
      if (bullets[i].y < 0) bullets[i].y = 240;
      if (bullets[i].y > 240) bullets[i].y = 0;
      
      // Draw bullet
      tft.fillCircle(bullets[i].x, bullets[i].y, 2, ILI9341_YELLOW);
    }
  }
  
  // Update asteroids
  bool anyActive = false;
  for (int i = 0; i < 12; i++) {
    if (asteroids[i].active) {
      anyActive = true;
      
      // Clear old asteroid
      tft.drawCircle(asteroids[i].x, asteroids[i].y, asteroids[i].size, ILI9341_BLACK);
      
      // Move asteroid faster
      asteroids[i].x += asteroids[i].vx;
      asteroids[i].y += asteroids[i].vy;
      
      // Wrap
      if (asteroids[i].x < -30) asteroids[i].x = 350;
      if (asteroids[i].x > 350) asteroids[i].x = -30;
      if (asteroids[i].y < -30) asteroids[i].y = 270;
      if (asteroids[i].y > 270) asteroids[i].y = -30;
      
      // Draw asteroid
      tft.drawCircle(asteroids[i].x, asteroids[i].y, asteroids[i].size, ILI9341_WHITE);
      
      // Check ship collision
      float shipDist = sqrt(pow(shipX - asteroids[i].x, 2) + pow(shipY - asteroids[i].y, 2));
      if (shipDist < asteroids[i].size + 8) {
        gameOver = true;
        // Explosion effect
        for (int j = 0; j < 5; j++) {
          tft.drawCircle(shipX, shipY, j * 8, ILI9341_RED);
          delay(50);
        }
        showGameOver();
        return;
      }
      
      // Check bullet collision
      for (int j = 0; j < 8; j++) {
        if (bullets[j].active) {
          float dist = sqrt(pow(bullets[j].x - asteroids[i].x, 2) + pow(bullets[j].y - asteroids[i].y, 2));
          if (dist < asteroids[i].size) {
            bullets[j].active = false;
            
            // Explosion effect
            for (int k = 0; k < 3; k++) {
              tft.drawCircle(asteroids[i].x, asteroids[i].y, k * 5, ILI9341_ORANGE);
              delay(30);
            }
            tft.fillCircle(asteroids[i].x, asteroids[i].y, asteroids[i].size, ILI9341_BLACK);
            
            asteroids[i].active = false;
            score += 10;
            drawScore();
            tone(BUZZER, 500, 100);
            
            // Spawn smaller asteroids
            if (asteroids[i].size > 12) {
              for (int k = 0; k < 12; k++) {
                if (!asteroids[k].active) {
                  asteroids[k].active = true;
                  asteroids[k].x = asteroids[i].x + random(-10, 11);
                  asteroids[k].y = asteroids[i].y + random(-10, 11);
                  asteroids[k].vx = random(-4, 5);
                  asteroids[k].vy = random(-4, 5);
                  asteroids[k].angle = random(360) * PI / 180;
                  asteroids[k].size = 10;
                  break;
                }
              }
            }
          }
        }
      }
    }
  }
  
  // Win condition
  if (!anyActive) {
    gameOver = true;
    tft.setTextColor(ILI9341_GREEN);
    tft.setTextSize(2);
    tft.setCursor(100, 100);
    tft.print("YOU WIN!");
    tone(BUZZER, 1500, 500);
  }
}

void drawShip(uint16_t color) {
  // Draw triangle ship
  int tipX = shipX + cos(shipAngle) * 12;
  int tipY = shipY + sin(shipAngle) * 12;
  int baseX1 = shipX + cos(shipAngle + 2.5) * 8;
  int baseY1 = shipY + sin(shipAngle + 2.5) * 8;
  int baseX2 = shipX + cos(shipAngle - 2.5) * 8;
  int baseY2 = shipY + sin(shipAngle - 2.5) * 8;
  
  tft.drawLine(tipX, tipY, baseX1, baseY1, color);
  tft.drawLine(tipX, tipY, baseX2, baseY2, color);
  tft.drawLine(baseX1, baseY1, baseX2, baseY2, color);
  
  // Thrust flame
  if (thrustPressed && color == ILI9341_WHITE) {
    int flameX = shipX + cos(shipAngle + PI) * 15;
    int flameY = shipY + sin(shipAngle + PI) * 15;
    tft.drawLine(shipX, shipY, flameX, flameY, ILI9341_RED);
  }
}
