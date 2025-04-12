```
#include <raylib.h>
#include <stdbool.h>
#include <stdio.h>

#define screenWidth  1100
#define screenHeight 900
#define gameTitle    "UWU"

typedef enum { 
	LOGO       = 0, 
	MENU       = 1, 
	GAMEPLAY   = 2, 
	END        = 3
} GameScreen;

typedef struct {
	Rectangle Body;
	Vector2   RecordPosition;
	Color     BodyColor;
	bool      WallTouch;
	bool      KeyHold;
	bool      KeyAreaEnter;
} Player ;

void movePlayer(Player *ThisPlayer, Rectangle *ThisWalls, int wallCount, Rectangle door, bool doorNotOpen) {	
	ThisPlayer->WallTouch = false;

	if (IsKeyDown(KEY_A)) {
		ThisPlayer->Body.x -= 200 * GetFrameTime();
	} else if (IsKeyDown(KEY_D)) {
		ThisPlayer->Body.x += 200 * GetFrameTime();
	}

	if (IsKeyDown(KEY_W)) {
		ThisPlayer->Body.y -= 200 * GetFrameTime();
	} else if (IsKeyDown(KEY_S)) {
		ThisPlayer->Body.y += 200 * GetFrameTime();
	}

	for (int i = 0; i < wallCount; i++) {
		if (CheckCollisionRecs(ThisPlayer->Body, ThisWalls[i])) {
			ThisPlayer->WallTouch = true;
			ThisPlayer->Body.x = ThisPlayer->RecordPosition.x;
			ThisPlayer->Body.y = ThisPlayer->RecordPosition.y;
			break;
		}
	}

	if (CheckCollisionRecs(ThisPlayer->Body, door) && doorNotOpen) {
		if (!ThisPlayer->WallTouch && !doorNotOpen) { 
			ThisPlayer->WallTouch = true;
			ThisPlayer->Body.x = ThisPlayer->RecordPosition.x;
			ThisPlayer->Body.y = ThisPlayer->RecordPosition.y;
		}
	}

	if (!ThisPlayer->WallTouch) {
		ThisPlayer->RecordPosition.x = ThisPlayer->Body.x;
		ThisPlayer->RecordPosition.y = ThisPlayer->Body.y;
	}
}

typedef struct {
	Rectangle obj;
} Objet;
 
void PlayerEnterKeyAreaCheck(Player *ThisPlayer, Objet *Key, Rectangle KeyArea, Rectangle *doorArea, bool *DoorNotOpen) {
	if (CheckCollisionRecs(ThisPlayer->Body, Key->obj)) {
		if (IsKeyPressed(KEY_E) && !ThisPlayer->KeyHold) {
			ThisPlayer->KeyHold = true;
			printf("helloooo \n");
		}
	}

	if (CheckCollisionRecs(ThisPlayer->Body, KeyArea)) {
		if (IsKeyPressed(KEY_E) && ThisPlayer->KeyHold) {
			ThisPlayer->KeyHold = false;
			*DoorNotOpen = false;
			doorArea->y -= 50;
			printf("kello \n");
		}
	}

	if (ThisPlayer->KeyHold) {
		Key->obj.x = ThisPlayer->Body.x;
		Key->obj.y = ThisPlayer->Body.y;
	}
}

void NextLevel(Player *ThisPlayer, Rectangle doorArea, bool DoorNotOpen, int *LevelCount) {
	if (CheckCollisionRecs(ThisPlayer->Body, doorArea)) {
		if (!DoorNotOpen) {
			*LevelCount += 1;
		}
	}
}

int main(void) {
    InitWindow(screenWidth, screenHeight, gameTitle);
	SetConfigFlags(FLAG_VSYNC_HINT | FLAG_MSAA_4X_HINT | FLAG_WINDOW_HIGHDPI); 
	
	InitAudioDevice();

	GameScreen CurrentScreen = LOGO;
	float      LogoTimer = 3.0f;

	char LogoTimerDebug[32];
	sprintf(LogoTimerDebug, "Timer: %f", LogoTimer);


	Texture2D BG = LoadTexture("./bg.png");
	BG.height = screenHeight;
	BG.width  = screenWidth;


	Music     BG_Music = LoadMusicStream("././radio_music_0.ogg");
	PlayMusicStream(BG_Music);	
	
	Player ThisPlayer = {
		.WallTouch = false,
		.RecordPosition = {(float)screenWidth/2-25, (float)screenHeight-200},
		.Body           = {(float)screenWidth/2-25, (float)screenHeight-200, 50.0f, 50.0f},
		.KeyAreaEnter   = false,
		.BodyColor      = PURPLE,
		.KeyHold        = false,
	};

	

	Rectangle Level0_Walls[] = {
		// left
		{0,   0, 50, screenHeight},
		// right
		{screenWidth-50, 0, 50, screenHeight},
		// top
		{50,  0,                50,             50},
		{200, 0,               150,             50},
		{350, 0,               screenWidth-350, 50},
		// bottom
		{ 50, screenHeight-50, screenWidth-100, 50}
	};
	int Level0_Walls_count = sizeof(Level0_Walls) / sizeof(Level0_Walls[0]);
	Rectangle Level0_door  = (Rectangle){50, 0, 150, 50};
	Rectangle Level0_Area  = (Rectangle){screenWidth-100, 200,              50, 50};
	
	Objet Level0_key = {
		.obj = {screenWidth-300, screenHeight-300, 50, 50}
	};
	bool Level0_door_notOpen = true;




	int LevelCount = 0;
    SetTargetFPS(60);
    while (!WindowShouldClose()) {
		// Update Game
		switch (CurrentScreen) {
			case LOGO:
				if (LogoTimer > 0) {
					LogoTimer -= GetFrameTime();
					sprintf(LogoTimerDebug, "Timer: %f", LogoTimer);
				} else {
					CurrentScreen = MENU;
					LogoTimer = 3.0f;
				}

				break;
			case MENU:
				if (IsKeyPressed(KEY_ENTER)) {
					CurrentScreen = GAMEPLAY;
				}

				break;
			case GAMEPLAY:
			UpdateMusicStream(BG_Music);
				switch (LevelCount) {
					case 0:
						movePlayer(&ThisPlayer, Level0_Walls, Level0_Walls_count, Level0_door, Level0_door_notOpen);
						PlayerEnterKeyAreaCheck(&ThisPlayer, &Level0_key, Level0_Area, &Level0_door, &Level0_door_notOpen);
						
						NextLevel(&ThisPlayer, Level0_door, Level0_door_notOpen, &LevelCount);
						break;
					case 1:
						break;
				}
				break;
			case END:


				break;
			default:

				break;
		}


		BeginDrawing();
            ClearBackground(BLACK);
			// Draw Things

			switch (CurrentScreen) {
				case LOGO:
					DrawText("Logo Screen", 10, 10, 30, PURPLE);
					DrawText(LogoTimerDebug, 10, 40, 30, PURPLE);

					break;
				case MENU:
					DrawText("Menu Screen -> Press Enter", 10, 10, 30, PURPLE);

					break;
				case GAMEPLAY:
					DrawTexture(BG, 0, 0, WHITE);
					switch (LevelCount) {
						case 0:							
							for (int i = 0; i < Level0_Walls_count; i++) {
								DrawRectangle(Level0_Walls[i].x, Level0_Walls[i].y, Level0_Walls[i].width, Level0_Walls[i].height, DARKPURPLE);
							};

							if (Level0_door_notOpen) {
								DrawRectangle(Level0_door.x, Level0_door.y, Level0_door.width, Level0_door.height, GREEN);
							}
								
							DrawRectangle(Level0_Area.x, Level0_Area.y, Level0_Area.width, Level0_Area.height, RED);
							DrawRectangle(Level0_key.obj.x,  Level0_key.obj.y,  Level0_key.obj.width,  Level0_key.obj.height,  BLUE);
							
							DrawRectangle(ThisPlayer.Body.x, ThisPlayer.Body.y, ThisPlayer.Body.width, ThisPlayer.Body.height, ThisPlayer.BodyColor);
							
							break;
						case 1:
							DrawText("level 1", screenWidth/2, screenHeight/2, 60, PURPLE);
					}
					DrawText("Gameplay Screen", 50, 50, 30, PURPLE);
					break;
				case END:
					DrawText("Ending Screen", 10, 10, 30, PURPLE);

					break;
				default:

					break;
			}

        EndDrawing();

	}


	UnloadMusicStream(BG_Music);
	UnloadTexture(BG);


	CloseWindow();
    return 0;
}

```
