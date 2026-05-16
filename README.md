#include <iostream>
#include <Windows.h>
#include <random>
#include <vector>

using namespace std;

void gotoxy(int x, int y)
{
    COORD pos = { x, y };
    HANDLE output = GetStdHandle(STD_OUTPUT_HANDLE);
    SetConsoleCursorPosition(output, pos);
}

void hideCursor()
{
    HANDLE consoleHandle = GetStdHandle(STD_OUTPUT_HANDLE);
    CONSOLE_CURSOR_INFO info;
    info.dwSize = 100;
    info.bVisible = FALSE;
    SetConsoleCursorInfo(consoleHandle, &info);
}

const int WIDTH = 20;
const int HEIGHT = 15;

enum Direction { UP, DOWN, LEFT, RIGHT };

int main()
{
    hideCursor();
    
    vector<int> snakeX = { WIDTH / 2 };
    vector<int> snakeY = { HEIGHT / 2 };
    
    Direction dir = UP;
    bool gameRunning = true;
    bool win = false;
    
    random_device rd;
    mt19937 gen(rd());
    uniform_int_distribution<> distW(0, WIDTH - 1);
    uniform_int_distribution<> distH(0, HEIGHT - 1);
    
    int foodX = distW(gen);
    int foodY = distH(gen);
    
    float timing = clock();

    while(gameRunning)
    {
        if (GetKeyState('W') & 0x8000 && dir != DOWN) dir = UP;
        if (GetKeyState('S') & 0x8000 && dir != UP) dir = DOWN;
        if (GetKeyState('A') & 0x8000 && dir != RIGHT) dir = LEFT;
        if (GetKeyState('D') & 0x8000 && dir != LEFT) dir = RIGHT;

        if ((clock() - timing) / CLOCKS_PER_SEC >= 0.15)
        {
            timing = clock();

            if (snakeX[0] == foodX && snakeY[0] == foodY)
            {
                snakeX.push_back(0);
                snakeY.push_back(0);
                
                // Проверка на победу
                if (snakeX.size() == WIDTH * HEIGHT)
                {
                    win = true;
                    gameRunning = false;
                    break;
                }
                
                foodX = distW(gen);
                foodY = distH(gen);
            }

            for (int i = snakeX.size() - 1; i > 0; --i)
            {
                snakeX[i] = snakeX[i - 1];
                snakeY[i] = snakeY[i - 1];
            }

            switch(dir)
            {
                case UP:    snakeY[0]--; break;
                case DOWN:  snakeY[0]++; break;
                case LEFT:  snakeX[0]--; break;
                case RIGHT: snakeX[0]++; break;
            }

            if (snakeX[0] < 0 || snakeX[0] >= WIDTH || 
                snakeY[0] < 0 || snakeY[0] >= HEIGHT)
            {
                gameRunning = false;
            }

            for (int i = 1; i < snakeX.size(); i++)
            {
                if (snakeX[0] == snakeX[i] && snakeY[0] == snakeY[i])
                {
                    gameRunning = false;
                }
            }

            gotoxy(0, 0);
            
            cout << "+";
            for (int i = 0; i < WIDTH; i++) cout << "-";
            cout << "+\n";
            
            for (int y = 0; y < HEIGHT; y++)
            {
                cout << "|";
                for (int x = 0; x < WIDTH; x++)
                {
                    if (x == foodX && y == foodY)
                        cout << "*";
                    else
                    {
                        bool isSnake = false;
                        for (int i = 0; i < snakeX.size(); i++)
                        {
                            if (snakeX[i] == x && snakeY[i] == y)
                            {
                                cout << (i == 0 ? "O" : "o");
                                isSnake = true;
                                break;
                            }
                        }
                        if (!isSnake) cout << " ";
                    }
                }
                cout << "|\n";
            }
            
            cout << "+";
            for (int i = 0; i < WIDTH; i++) cout << "-";
            cout << "+\n";
            
            cout << "Score: " << snakeX.size() - 1;
        }
    }

    gotoxy(0, HEIGHT + 3);
    if (win)
    {
        cout << "YOU WIN! Score: " << snakeX.size() - 1;
    }
    else
    {
        cout << "GAME OVER! Score: " << snakeX.size() - 1;
    }
    cout << "\n";

    return 0;
}
