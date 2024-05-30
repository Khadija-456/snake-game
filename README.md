#include <iostream>
#include <SFML/Graphics.hpp>
#include <SFML/system.hpp>
#include <ctime>
//#include"../SFML/Images/"
using namespace sf;
using namespace std;





const int resolutionX = 960;
const int resolutionY = 960;
const int boxPixelsX = 32;
const int boxPixelsY = 32;
const int gameRows = resolutionX / boxPixelsX; // Total rows on grid
const int gameColumns = resolutionY / boxPixelsY; // Total columns on grid

// Initializing GameGrid.
int gameGrid[gameRows][gameColumns] = {};



void drawSnake(RenderWindow& window, Sprite& Snake_Head_Sprite, Sprite& Snake_Body_Sprite, int** Snake);
void UpdatePosition(int** Snake, Clock& Snake_Clock, int Fruit[]);
void ChangeDirection(int** Snake, Clock& Head_clock);
void drawFruit(RenderWindow& window, Sprite& Fruit_Sprite, int fruit[]);
void increaseLength(int** Snake);
void GenerateFruit(int Fruit[]);


//Drawing the background
void createBack(RenderWindow& window) {
	//Drawing the background
	Image map_image;
	map_image.loadFromFile("../SFML/Images/snake.jpg");
	Texture map;
	map.loadFromImage(map_image);
	Sprite s_map;
	s_map.setTexture(map);
	s_map.setScale(1.75, 1.98);
	s_map.setPosition(0, 0);
	window.draw(s_map);
}

//Drawing the map

int X_Axis = 0;
int Y_Axis = 1;
int Direction = 2;
int is_head = 3;

int exists = 3;

int UP = 0;
int RIGHT = 1;
int DOWN = 2;
int LEFT = 3;

int Snake_length = 2;


int main()
{

	srand(time(0));


	Texture Snake_Head_Texture;
	Clock Snake_Head_Clock;
	Sprite Snake_Head_Sprite;
	Snake_Head_Texture.loadFromFile("../SFML/Images/Head.png");
	Snake_Head_Sprite.setTexture(Snake_Head_Texture);
	Snake_Head_Sprite.setTextureRect(sf::IntRect(0, 0, boxPixelsX, boxPixelsY));



	Texture Snake_Body_Texture;
	Clock Snake_Body_Clock;
	Sprite Snake_Body_Sprite;
	Snake_Body_Texture.loadFromFile("../SFML/Images/Body.png");
	Snake_Body_Sprite.setTexture(Snake_Body_Texture);
	Snake_Body_Sprite.setTextureRect(sf::IntRect(0, 0, boxPixelsX, boxPixelsY));


	int** Snake = new int*[Snake_length];
	for (int i = 0; i < 10; i++)
	{
		Snake[i] = new int[4];
	}

	Snake[0][X_Axis] = (15 * 32);
	Snake[0][Y_Axis] = (15 * 32);
	Snake[0][Direction] = RIGHT;
	Snake[0][is_head] = true;

	gameGrid[15 - 1][15 - 1] = 2;

	for (int i = 1; i < Snake_length; i++)
	{
		Snake[i][X_Axis] = (Snake[i - 1][X_Axis] - 32);
		Snake[i][Y_Axis] = Snake[0][Y_Axis];
		Snake[i][Direction] = RIGHT;
		Snake[i][is_head] = false;

	}



	Texture Fruit_Texture;
	Clock Fruit_Clock;
	Sprite Fruit_Sprite;
	Fruit_Texture.loadFromFile("../SFML/Images/pea.png");
	Fruit_Sprite.setTexture(Fruit_Texture);
	Fruit_Sprite.setTextureRect(sf::IntRect(0, 0, 28, 28));


	int Fruit[3];
	Fruit[X_Axis] = (rand() % 30) * 32;
	Fruit[Y_Axis] = (rand() % 30) * 32;
	Fruit[exists] = true;

	gameGrid[(Fruit[Y_Axis]) / 32][Fruit[X_Axis] / 32] = 5;

	//Create a window, n*n
	RenderWindow window(VideoMode(960, 960), "Plants Vs Zombies");
	//Game icon
	Image icon;
	if (!icon.loadFromFile("../SFML/Images/icon.png"))
	{
		return 1;
	}
	window.setIcon(32, 32, icon.getPixelsPtr());

	


	Clock clock;

	while (window.isOpen())
	{


		for (int i = 0; i < 30; i++)
		{
			for (int j = 0; j < 30; j++)
			{

				cout << gameGrid[i][j] << " ";

			}
			cout << endl;
		}
		cout << endl << endl;




		Event event;
		while (window.pollEvent(event))
		{
			if ((event.type == Event::Closed) || sf::Keyboard::isKeyPressed(sf::Keyboard::Escape))
				window.close();
		}

		//Create a background
		createBack(window);
		
		//Draw Snake
		drawSnake(window, Snake_Head_Sprite, Snake_Body_Sprite, Snake);
		drawFruit(window, Fruit_Sprite, Fruit);
		ChangeDirection(Snake, Snake_Body_Clock);
		UpdatePosition(Snake, Snake_Head_Clock, Fruit);
		
		


		//window.setSize(sf::Vector2u(960, 960));
		window.display();
	}
	return 0;
}


void drawSnake(RenderWindow& window, Sprite& Snake_Head_Sprite, Sprite& Snake_Body_Sprite, int** Snake)
{
	
	
	
	Snake_Head_Sprite.setPosition(Snake[0][X_Axis], Snake[0][Y_Axis]);
	window.draw(Snake_Head_Sprite);

	for (int i = 1; i < Snake_length; i++)
	{
		Snake_Body_Sprite.setPosition(Snake[i][X_Axis], Snake[i][Y_Axis]);
		window.draw(Snake_Body_Sprite);
	}

}

void drawFruit(RenderWindow& window, Sprite& Fruit_Sprite, int fruit[])
{
	if(fruit[exists])
	{
		Fruit_Sprite.setPosition(fruit[X_Axis], fruit[Y_Axis]);
		window.draw(Fruit_Sprite);
	}
}


void UpdatePosition(int** Snake, Clock& Snake_Clock, int Fruit[])
{
	if (Snake_Clock.getElapsedTime().asMilliseconds() < 100)
		return;

	Snake_Clock.restart();

		for (int i = Snake_length - 1; i > 0; i--)
		{

			Snake[i][X_Axis] = Snake[i - 1][X_Axis];
			Snake[i][Y_Axis] = Snake[i - 1][Y_Axis];
			Snake[i][Direction] = Snake[i - 1][Direction];


		}
	
		gameGrid[(Snake[0][Y_Axis]) / 32 - 1][(Snake[0][X_Axis]) / 32 - 1] = 0;
		

		if (gameGrid[(Snake[0][Y_Axis]) / 32][(Snake[0][X_Axis]) / 32] == 5)
		{
			Fruit[exists] = false;
			increaseLength(Snake);
			GenerateFruit(Fruit);
		}

		if (Snake[0][Direction] == RIGHT)
		{
			Snake[0][X_Axis] += 32;
		}

		else if (Snake[0][Direction] == LEFT)
		{
			Snake[0][X_Axis] -= 32;
        }

		else if (Snake[0][Direction] == UP)
		{
			Snake[0][Y_Axis] -= 32;
		}

		else if (Snake[0][Direction] == DOWN)
		{
			Snake[0][Y_Axis] += 32;
		}


		gameGrid[(Snake[0][Y_Axis]) / 32 - 1][(Snake[0][X_Axis]) / 32 - 1] = 2;


}




void ChangeDirection(int** Snake, Clock& Head_clock)
{
	if (Head_clock.getElapsedTime().asMilliseconds() < 30)
		return;
	Head_clock.restart();

	if (sf::Keyboard::isKeyPressed(sf::Keyboard::Left) && Snake[0][Direction] != RIGHT && Snake[0][X_Axis] > 0) {

		Snake[0][Direction] = LEFT;

	}
	if (sf::Keyboard::isKeyPressed(sf::Keyboard::Right) && Snake[0][Direction] != LEFT && Snake[0][X_Axis] < 928) {
		Snake[0][Direction] = RIGHT;
	}
	if (sf::Keyboard::isKeyPressed(sf::Keyboard::Down) && Snake[0][Direction] != UP && Snake[0][Y_Axis] < 928) {
		Snake[0][Direction] = DOWN;
	}
	if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up) && Snake[0][Direction] != DOWN && Snake[0][Y_Axis] > 0 ) {
		Snake[0][Direction] = UP;
	}

}



void increaseLength(int** Snake)
{

	int** temp = new int* [Snake_length + 1];
	for (int i = 0; i <= Snake_length; i++)
	{
		temp[i] = new int[4];
	}

	for (int i = 0; i < Snake_length; i++)
	{
		for (int j = 0; j < 4; j++)
		{
			temp[i][j] = Snake[i][j];
		}
	}

	if (Snake[Snake_length - 1][Direction] == UP)
	{
		temp[Snake_length][X_Axis] = Snake[Snake_length - 1][X_Axis];
		temp[Snake_length][Y_Axis] = Snake[Snake_length - 1][Y_Axis] + 32;
		temp[Snake_length][Direction] = Snake[ Snake_length - 1 ][Direction];
		temp[Snake_length][is_head] = Snake[ Snake_length - 1 ][is_head];
	}

	else if (Snake[Snake_length - 1][Direction] == DOWN)
	{
		temp[Snake_length][X_Axis] = Snake[Snake_length - 1][X_Axis];
		temp[Snake_length][Y_Axis] = Snake[Snake_length - 1][Y_Axis] - 32;
		temp[Snake_length][Direction] = Snake[Snake_length - 1][Direction];
		temp[Snake_length][is_head] = Snake[Snake_length - 1][is_head];
	}

	else if (Snake[Snake_length - 1][Direction] == RIGHT)
	{
		temp[Snake_length][X_Axis] = Snake[Snake_length - 1][X_Axis] - 32;
		temp[Snake_length][Y_Axis] = Snake[Snake_length - 1][Y_Axis];
		temp[Snake_length][Direction] = Snake[Snake_length - 1][Direction];
		temp[Snake_length][is_head] = Snake[Snake_length - 1][is_head];
	}

	else if (Snake[Snake_length - 1][Direction] == LEFT)
	{
		temp[Snake_length][X_Axis] = Snake[Snake_length - 1][X_Axis] + 32;
		temp[Snake_length][Y_Axis] = Snake[Snake_length - 1][Y_Axis];
		temp[Snake_length][Direction] = Snake[Snake_length - 1][Direction];
		temp[Snake_length][is_head] = Snake[Snake_length - 1][is_head];
	}

	//for (int i = 0; i < Snake_length; i++)
	//{
	//	delete[] Snake[i];
	//}
	//delete Snake;
	
	Snake_length++;    //Updating the snake length 

	Snake = temp;
	

}


void GenerateFruit(int Fruit[])
{

	Fruit[X_Axis] = (rand() % 30) * 32;
	Fruit[Y_Axis] = (rand() % 30) * 32;
	Fruit[exists] = true;
	gameGrid[(Fruit[Y_Axis]) / 32][Fruit[X_Axis] / 32] = 5;


}
