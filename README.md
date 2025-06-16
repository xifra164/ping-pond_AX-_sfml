#include <windows.h>
#include <SFML/Audio.hpp>
#include <SFML/Graphics.hpp>
#include <iostream>
using namespace sf;

int main()
{
  /*opzetten van game componenten*/

    // maken van een scherm
    RenderWindow scherm(VideoMode(1280, 800), "naam van onze game");

    //maken van de paddles
    RectangleShape paddle1(vector2f(25,120));
    RectangleShape paddle2(vector2f(25,120));
    paddle1.setFillColor(Color::Red);
    paddle2.setFillColor(Color::Red);
    paddle1.setPosition(10, 340);
    paddle1.setPosition(605, 340);

    //maken van een ping-pong bal
    CircleShape bal(20);
    ball.setFillColor(Color::Yellow);
    ball.setPosition(640, 400);
    float balsnelheidx = -0.3f;
    float balsnelheidy = 0.3f;

//game loop
     while (scherm.isOpen())
     {
        //process events
        Event event;
        while (scherm.pollEvent(event))
        {
            if (event.type == Event::Closed)
            scherm.close();
        }



        //clear scherm
        scherm.clear();

        /*tekeningen maken van bal,paddles,achtergrond en scores*/
        scherm.draw(paddle1);
        scherm.draw(paddle2);
        scherm.draw(bal);

        //display wat er is getekend
        scherm.display();
     }
    return 0;
}
