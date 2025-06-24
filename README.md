 #include <windows.h> 
#include <SFML/Audio.hpp> 
#include <SFML/Graphics.hpp>
 using namespace sf;

 int main() {
 //opzetten van game componenten 
 // maken van een scherm
 const int FPS = 60;
 RenderWindow window(VideoMode({ 640, 480}), "Ping pong!");


  //maken van de paddles
    RectangleShape paddle1(vector2f(25,120));
    RectangleShape paddle2(vector2f(25,120));
    paddle1.setFillColor(Color::Red);
    paddle2.setFillColor(Color::Red);
    paddle1.setPosition(10, 340);
    paddle2.setPosition(605, 340);

   //maken van een ping-pong bal
    CircleShape bal(20);
    ball.setFillColor(Color::Yellow);
    ball.setPosition(315,235 );
    float balsnelheidx = -0.3f;
    float balsnelheidy = 0.3f;

  //game loop 
 
while(window.isOpen())
{
	while (const std::optional event = window.pollEvent())
	{
		if (event->is<::Event::Closed>())
			window.close();
	}

//clear scherm
 window.clear();
 
 /*tekeningen maken van bal,paddles,achtergrond en scores*/
        window.draw(paddle1);
        window.draw(paddle2);
        window.draw(bal);

  //display wat er is getekend
   window.display();
    
  }
    return 0;
}
