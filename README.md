#include <windows.h> 
#include <SFML/Audio.hpp> 
#include <SFML/Graphics.hpp>
using namespace sf;

int main() {
	//opzetten van game componenten 
	// maken van een scherm
	const int FPS = 60;
	RenderWindow window(VideoMode({ 640, 480 }), "Ping pong!");
        float windowWidth = 640;
        float windowHeight = 480;


	//maken van de paddles
	RectangleShape paddle1(Vector2f(25, 120));
	RectangleShape paddle2(Vector2f(25, 120));
	paddle1.setFillColor(Color::Red);
	paddle2.setFillColor(Color::Red);
	paddle1.setPosition(Vector2f(10, 340));
	paddle2.setPosition(Vector2f(605, 340));
	float paddle1snelheid = 5.0f;
	float paddle2snelheid = 5.0f;
	
	float cpu = 0.0f;

	//maken van een ping-pong bal
	CircleShape bal(20);
	bal.setFillColor(Color::Yellow);
	bal.setPosition(Vector2f(315, 235));
	float balsnelheidx = -0.3f;
	float balsnelheidy = 0.3f;

	//game loop 

	while (window.isOpen())
	{
		while (const std::optional event = window.pollEvent())
		{
			if (event->is<::Event::Closed>())
				window.close();
		}


		// --- Ball Movement ---
		bal.move(sf::Vector2f(balsnelheidx, balsnelheidy));

		// --- Ball Collision with Walls ---
		// Top wall collision
		if (bal.getPosition().y <= 0) {
			balsnelheidy *= -1; // Reverse vertical direction
			bal.setPosition(sf::Vector2f(bal.getPosition().x, 0));
		}

		// Bottom wall collision
		//rechterkant
		if (bal.getPosition().y + (bal.getRadius() * 2) >= window.getSize().y) {
			balsnelheidy *= -1; // Reverse vertical direction
			bal.setPosition(sf::Vector2f(bal.getPosition().x, window.getSize().y - (bal.getRadius() * 2)));
		}


		// --- Bal Botsing met ZIJ-MUREN
		// Als de bal de linkerkant passeert 
		if (bal.getPosition().x <= 0) {
			balsnelheidx *= -1;
			bal.setPosition(sf::Vector2f(window.getSize().x / 2 - bal.getRadius(), window.getSize().y / 2 - bal.getRadius())); // Reset bal naar midden


		}

		if (bal.getPosition().x + (bal.getRadius() * 2) >= window.getSize().x) {
			balsnelheidx *= -1; // Keer de horizontale richting om bij reset (zodat hij naar speler 2 beweegt)
			bal.setPosition(sf::Vector2f(window.getSize().x / 2 - bal.getRadius(), window.getSize().y / 2 - bal.getRadius())); // Reset bal naar midden


		}
		//paddle 1 beweging//
		if (sf::Keyboard::isKeyPressed(sf::Keyboard::Key::W)) {
			if (paddle1.getPosition().y > 0)
				paddle1.move((sf::Vector2f(0, -paddle1snelheid))); // Paddle 2 beweegt naar boven
		}
		if (sf::Keyboard::isKeyPressed(sf::Keyboard::Key::S)) {
			if (paddle1.getPosition().y + paddle1.getSize().y < windowHeight)
				paddle1.move((sf::Vector2f(0, paddle1snelheid))); // Paddle 1 beweegt naar beneden	
		}

		//Paddle 2 (Up/Down)
		if (sf::Keyboard::isKeyPressed(sf::Keyboard::Key::Up)) {
			if (paddle2.getPosition().y > 0)
				paddle2.move((sf::Vector2f(0, -paddle2snelheid)));
		}
		if (sf::Keyboard::isKeyPressed(sf::Keyboard::Key::Down)) {
			if (paddle2.getPosition().y + paddle2.getSize().y < windowHeight)
				paddle2.move((sf::Vector2f(0, paddle2snelheid)));
		}
		paddle2.move((sf::Vector2f(cpu, balsnelheidy)));

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
