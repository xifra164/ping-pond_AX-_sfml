#include <windows.h>
#include <SFML/Audio.hpp>
#include <SFML/Graphics.hpp> 
#include <iostream>
#include<string>
#include<cstdlib>
#include<optional>
#include<cmath>
using namespace std;
using namespace sf; 

enum GameState {
	MENU,
	PLAYING,
	GAME_OVER,

	
};

// Globale variabelen voor game-componenten (nodig voor resetGame functie)
// Let op: het gebruik van globale pointers is functioneel voor dit voorbeeld,
// maar in grotere projecten is het beter om een klasse te maken.
RenderWindow* globalWindowPtr = nullptr;
RectangleShape* globalPaddle1Ptr = nullptr;
RectangleShape* globalPaddle2Ptr = nullptr;
CircleShape* globalBallPtr = nullptr;

float* globalBallSpeedXPtr = nullptr;
float* globalBallSpeedYPtr = nullptr;
int* globalPlayer1ScorePtr = nullptr;
int* globalPlayer2ScorePtr = nullptr;
bool* globalGameOverPtr = nullptr; // Deze is nog niet volledig geïmplementeerd in de PLAYING state
int* globalWinnerPtr = nullptr;

const int* globalBallRadiusPtr = nullptr;
float* globalWindowWidthPtr = nullptr;
float* globalWindowHeightPtr = nullptr;

// Functie om de game te resetten naar de beginstand
// Functie om de bal en paddles te resetten voor een nieuwe ronde
void resetRoundElements() {
	if (globalWindowPtr && globalPaddle1Ptr && globalPaddle2Ptr && globalBallPtr &&
		globalBallSpeedXPtr && globalBallSpeedYPtr &&
		globalBallRadiusPtr && globalWindowWidthPtr && globalWindowHeightPtr) {

  		// Reset paddle posities naar het midden van de zijkant
		globalPaddle1Ptr->setPosition(Vector2f(10, *globalWindowHeightPtr / 2 - globalPaddle1Ptr->getSize().y / 2));
		globalPaddle2Ptr->setPosition(Vector2f(*globalWindowWidthPtr - 25 - 10, *globalWindowHeightPtr / 2 - globalPaddle2Ptr->getSize().y / 2));

		// Reset bal positie naar het midden van het scherm
		globalBallPtr->setPosition(Vector2f(*globalWindowWidthPtr / 2 - *globalBallRadiusPtr, *globalWindowHeightPtr / 2 - *globalBallRadiusPtr));

		// Willekeurige startrichting voor de bal
		*globalBallSpeedXPtr = (rand() % 2 == 0) ? -3.0f : 3.0f; // Start links of rechts
		*globalBallSpeedYPtr = (rand() % 7) - 3.0f; // Willekeurige hoek tussen -3 en +3
		if (*globalBallSpeedYPtr == 0) *globalBallSpeedYPtr = 1.0f; // Voorkom 0 verticale snelheid
	}
}

// Functie om een volledig nieuw spel te starten (inclusief scores resetten)
void startNewGame() {
	if (globalPlayer1ScorePtr && globalPlayer2ScorePtr && globalGameOverPtr && globalWinnerPtr) {
		*globalPlayer1ScorePtr = 0;
		*globalPlayer2ScorePtr = 0;
		*globalGameOverPtr = false; 
		*globalWinnerPtr = 0;    
	}
	resetRoundElements(); // Roep de functie aan die alleen de elementen reset
}




int main() {
	//opzetten van game componenten 
	// 
	//maken van een scherm 
	srand(static_cast<unsigned int>(time(0)));
	const int FPS = 60;
	RenderWindow window(VideoMode({ 640, 480 }), "Ping pong!");
	float windowHeight = 480;
	float windowWidth = 640;
	window.setFramerateLimit(FPS);

	//maken van de paddles
	RectangleShape paddle1(Vector2f(25, 120));
	RectangleShape paddle2(Vector2f(25, 120));
	paddle1.setFillColor(Color::Magenta);
	paddle2.setFillColor(Color::Magenta);
	paddle1.setPosition(Vector2f(10, 340));
	paddle2.setPosition(Vector2f(605, 340));
	const float paddle_speed = 5.0f;



	// NIEUW: Omtrek voor de paddles
	paddle1.setOutlineThickness(2);
	paddle1.setOutlineColor(Color::White);
	paddle2.setOutlineThickness(2);
	paddle2.setOutlineColor(Color::White);


	// For the dashed line in the middle - NEW
	float centerX = windowWidth / 2.0f;
	const float dashLength = 10.0f;
	const float gapLength = 5.0f;
	const float totalSegmentLength = dashLength + gapLength;


	// NIEUW: Teken de gestippelde cirkel in het midden
	float centerY = windowHeight / 2.0f;
	int numCircleSegments = 60; // Aantal{


//maken van een ping-pong bal
CircleShape bal(20);
bal.setFillColor(Color::Blue);
// NIEUW: Omtrek voor de bal
bal.setOutlineThickness(2);
bal.setOutlineColor(Color::Magenta);
bal.setPosition(Vector2f(315, 235));
float balsnelheidx = -3.0f;
float balsnelheidy = 3.0f;
int ballRadius = 20;







// Initialize scores and winning condition
int player1_score = 0;
int player2_score = 0;
const int winning_score = 10;
bool game_over = false;
int winner = 0; // 0 = no winner, 1 = player1, 2 = player2 

// Koppel globale pointers aan lokale variabelen
globalWindowPtr = &window;
globalPaddle1Ptr = &paddle1;
globalPaddle2Ptr = &paddle2;
globalBallPtr = &bal;
globalBallSpeedXPtr = &balsnelheidx;
globalBallSpeedYPtr = &balsnelheidy;
globalPlayer1ScorePtr = &player1_score;
globalPlayer2ScorePtr = &player2_score;
globalGameOverPtr = &game_over;
globalWinnerPtr = &winner;
globalBallRadiusPtr = &ballRadius;
globalWindowWidthPtr = &windowWidth;
globalWindowHeightPtr = &windowHeight;


// Initialiseer game state
GameState currentState = MENU; // Start in het menu

// Create font and text for messages/
Font font;
if (!font.openFromFile("arial.ttf")) { // Handle font loading error retur

	return EXIT_FAILURE;
}

// Tekst voor startscherm en scores
sf::Text messageText(font);
messageText.setCharacterSize(60);
messageText.setFillColor(Color::Blue);
messageText.setStyle(Text::Bold);
// NIEUW: Tekst omtrek voor de titel
messageText.setOutlineThickness(3);
messageText.setOutlineColor(Color::Magenta); // Witte omtrek


sf::Text instructionsText(font);
instructionsText.setCharacterSize(30);
instructionsText.setFillColor(Color::Magenta);

sf::Text player1ScoreText(font);
player1ScoreText.setFont(font);
player1ScoreText.setCharacterSize(40);
player1ScoreText.setFillColor(Color::Blue);
player1ScoreText.setPosition(sf::Vector2f(20, 20)); // Positie linksboven
player1ScoreText.setOutlineThickness(2);
player1ScoreText.setOutlineColor(Color::White);


sf::Text player2ScoreText(font);
player2ScoreText.setFont(font);
player2ScoreText.setCharacterSize(40);
player2ScoreText.setFillColor(Color::Blue);
player2ScoreText.setOutlineThickness(2);
player2ScoreText.setOutlineColor(Color::White);


// NIEUW: Klok voor pulserende tekst in het menu
sf::Clock menuClock;


// Tekst voor win/lose berichten (eenmalig initialiseren)
sf::Text winLoseMessageText(font);
winLoseMessageText.setCharacterSize(60);
winLoseMessageText.setStyle(Text::Bold);

sf::Text restartPromptText(font);
restartPromptText.setCharacterSize(30);
restartPromptText.setFillColor(Color::White);



RectangleShape dash(Vector2f(2.0f, dashLength));
dash.setFillColor(Color::Magenta);

FloatRect ballBounds;
FloatRect paddle1Bounds;
FloatRect paddle2Bounds;
RectangleShape overlay(Vector2f(windowWidth, windowHeight)); // Initialiseer overlay hier
overlay.setFillColor(Color(0, 0, 0, 180)); // Semi-transparant zwart



// Tekstobject voor de scoreweergave in de PLAYING-state (eenmalig initialiseren)
sf::Text scoreDisplay(font);
scoreDisplay.setCharacterSize(40);
scoreDisplay.setFillColor(Color::Yellow);


// NIEUW: Variabelen voor de gestippelde cirkel
const float dashedCircleRadius = 100.0f; // Straal van de gestippelde cirkel
const float circleDashLength = 5.0f; // Lengte van elk streepje van de cirkel
const float circleDashThickness = 2.0f; // Dikte van elk streepje van de cirkel
RectangleShape circleDash(Vector2f(circleDashLength, circleDashThickness));
circleDash.setFillColor(Color::Magenta);
circleDash.setOrigin(sf::Vector2f(circleDash.getSize().x / 2.0f, circleDash.getSize().y / 2.0f)); // Origin voor rotatie

//game loop while 
while (window.isOpen())
{

	while (const std::optional<Event> event = window.pollEvent())
	{
		if (event->is<::Event::Closed>()) {
			window.close();
		}


		// Input handling voor state changes - DIT BLOK MOET HIER BINNEN ZIJN

		if (event->is<::Event::KeyPressed>()) {// Let op: event->type en event->key.code
			if (currentState == MENU) {
				if (Keyboard::isKeyPressed(Keyboard::Key::Space)) {
					currentState = PLAYING;
				}
				else if (Keyboard::isKeyPressed(Keyboard::Key::Escape)) {
					window.close();
				}

			}
			else if (currentState == GAME_OVER) {
				if (Keyboard::isKeyPressed(Keyboard::Key::Space)) { // Correct: event.key.code
					startNewGame(); // Start een nieuw spel (reset scores, etc.)
					currentState = PLAYING; // Verander de staat naar 'spelen'
					std::cout << "DEBUG: Overgang naar PLAYING staat vanuit GAME_OVER." << std::endl; // Debug output
				}
				else if (Keyboard::isKeyPressed(Keyboard::Key::Escape)) { // Correct: event.key.code
					window.close(); // Sluit het venster
				}
				// Andere state inputs (bijv. voor pauze) zouden hier komen
			}
		}
	}


        window.clear(sf::Color::Black);

switch (currentState) {
case MENU:


	// Teken de titel van de game
	messageText.setString("Ping Pong!");
	messageText.setFillColor(Color::Blue);
	messageText.setCharacterSize(80);

	// Centreer de tekst
	messageText.setPosition(sf::Vector2f(windowWidth / 2.0f - (messageText.getCharacterSize() * 4.0f) / 2.0f, // Geschatte breedte
		windowHeight / 2.0f - (messageText.getCharacterSize() / 2.0f) - 50.0f)); // Geschatte hoogte
	window.draw(messageText);

	// Teken de instructies voor de speler
	instructionsText.setString("Druk op SPATIE om te starten\nDruk op ESC om af te sluiten");
	instructionsText.setFillColor(Color::Magenta);
	// ALTERNATIEVE MANIER: Positioneer de tekst handmatig.
	instructionsText.setPosition(sf::Vector2f(windowWidth / 2.0f - (instructionsText.getCharacterSize() * 12.0f) / 2.0f, // Geschatte breedte
		windowHeight / 2.0f + (instructionsText.getCharacterSize() / 2.0f) + 50.0f)); // Geschatte hoogte
	window.draw(instructionsText);
	break;


case PLAYING: {

	// --- Ball Movement ---
	bal.move(sf::Vector2f(balsnelheidx, balsnelheidy));
	// --- Ball Collision with Walls --- 

	// // Top wall collision 
	if (bal.getPosition().y <= 0) {
		balsnelheidy *= -1;// Reverse vertical direction 
		bal.setPosition(sf::Vector2f(bal.getPosition().x, 0));
	}

	// Bottom wall collision //rechterkant
	if (bal.getPosition().y + (bal.getRadius() * 2) >= window.getSize().y) {
		balsnelheidy *= -1; // Reverse vertical direction
		bal.setPosition(sf::Vector2f(bal.getPosition().x, window.getSize().y - (bal.getRadius() * 2)));
	}


	// --- Ball Collision with Side Walls (Scoring) ---
			// CORRECTE SCORING LOGICA: Bal gaat voorbij de randen van het scherm
	if (bal.getPosition().x + (bal.getRadius() * 2) < 0) { // Bal gaat links van het scherm: Speler 2 scoort
		player2_score++;
		std::cout << "DEBUG: Speler 2 scoorde! Score: " << player2_score << std::endl; // Debug output
		if (player2_score >= winning_score) {
			game_over = true;
			winner = 2;
			currentState = GAME_OVER; // Verander naar GAME_OVER staat
			std::cout << "DEBUG: Game Over! Speler 2 wint. Overgang naar GAME_OVER staat." << std::endl; // Debug output
		}
		else {

			resetRoundElements();// Reset bal en paddles na score (of game over)
		}

	}

	if (bal.getPosition().x > window.getSize().x) { // Bal gaat rechts van het scherm: Speler 1 scoort
		player1_score++;
		std::cout << "DEBUG: Speler 1 scoorde! Score: " << player1_score << std::endl; // Debug output
		if (player1_score >= winning_score) {
			game_over = true;
			winner = 1;
			currentState = GAME_OVER; // Verander naar GAME_OVER staat
			std::cout << "DEBUG: Game Over! Speler 1 wint. Overgang naar GAME_OVER staat." << std::endl; // Debug output
		}
		else {

			resetRoundElements(); // Reset alleen bal en paddles voor de volgende ronde
		}
	}



	//paddle 1 beweging//
	 // --- Paddle 1 Beweging (W/S) ---
	// -- - Paddle 1 Beweging(W / S) -- -
		// Controleert of de 'W' toets is ingedrukt (beweegt paddle omhoog)
	if (sf::Keyboard::isKeyPressed(sf::Keyboard::Key::W)) {
		paddle1.move(sf::Vector2f(0, -paddle_speed));
	}
	if (sf::Keyboard::isKeyPressed(sf::Keyboard::Key::S)) {
		paddle1.move(sf::Vector2f(0, paddle_speed));

	}

	//Paddle 2 (Up/Down) 
	if (sf::Keyboard::isKeyPressed(sf::Keyboard::Key::Up)) {
		paddle2.move(sf::Vector2f(0, -paddle_speed));

	}

	if (sf::Keyboard::isKeyPressed(sf::Keyboard::Key::Down)) {
		paddle2.move(sf::Vector2f(0, paddle_speed));

	}


	//Voorkomen dat de paddles uit het scherm gaan
   //paddle 1 grenzen
	if (paddle1.getPosition().y < 0) {  //Bovenkant
		paddle1.setPosition(sf::Vector2f(paddle1.getPosition().x, 0));
	}

	if (paddle1.getPosition().y + paddle1.getSize().y > window.getSize().y) {  //Bovenkant
		paddle1.setPosition(sf::Vector2f(paddle1.getPosition().x, window.getSize().y - paddle1.getSize().y));
	}

	// paddle 2 grenzen
	if (paddle2.getPosition().y < 0) {  //Bovenkant
		paddle2.setPosition(sf::Vector2f(paddle2.getPosition().x, 0));
	}

	if (paddle2.getPosition().y + paddle2.getSize().y > window.getSize().y) {  //Bovenkant
		paddle2.setPosition(sf::Vector2f(paddle2.getPosition().x, window.getSize().y - paddle2.getSize().y));
	}





	// Check collision with all paddles
	FloatRect ballBounds = bal.getGlobalBounds();
	FloatRect paddle1Bounds = paddle1.getGlobalBounds();
	FloatRect paddle2Bounds = paddle2.getGlobalBounds();

	//Botsing met paddle 1
	if (ballBounds.findIntersection(paddle1Bounds)) { // Only bounce if moving toward paddle1
		if (balsnelheidx < 0) {
			balsnelheidx *= -1; // Adjust ball position to prevent sticking
			bal.setPosition(sf::Vector2f(paddle1.getPosition().x + paddle1.getSize().x + 1, bal.getPosition().y)); // Add vertical direction based on where ball hits paddle 
			float hitPosition = (bal.getPosition().y + bal.getRadius() - paddle1.getPosition().y) / paddle1.getSize().y;
			balsnelheidy = 8 * (hitPosition - 0.5f); // -4 to +4 range 
		}
	}
	//Botsing met paddle 2
	if (ballBounds.findIntersection(paddle2Bounds)) { // Only bounce if moving toward paddle2
		if (balsnelheidx > 0) {
			balsnelheidx *= -1; // Adjust ball position to prevent sticking
			bal.setPosition(Vector2f(paddle2.getPosition().x - paddle2.getSize().x - 1, bal.getPosition().y)); // Add vertical direction based on where ball hits paddle 
			float hitPosition = (bal.getPosition().y + ballRadius - paddle2.getPosition().y) / paddle2.getSize().y;
			balsnelheidy = 8 * (hitPosition - 0.5f); // -4 to +4 range 

		}
	}

         // // tekeningen maken van bal, paddles, achtergrond en scores / 
window.draw(paddle1);
window.draw(paddle2);
window.draw(bal);


// --- NEW: Draw the dashed line in the middle ---
// Teken de gestippelde lijn in het midden
	// 'dash' is nu gedeclareerd buiten de switch, dus dit is OK

float centerX = windowWidth / 2.0f; // Calculate the center X position

for (float y = 0; y < windowHeight; y += totalSegmentLength) {
	dash.setPosition(sf::Vector2f(centerX - dash.getSize().x / 2.0f, y)); // Center the dash horizontally
	window.draw(dash);
}

// NIEUW: Teken de gestippelde cirkel in het midden
float centerY = windowHeight / 2.0f;


for (int i = 0; i < numCircleSegments; ++i) {
	if (i % 2 == 0) { // Teken elk ander segment voor het gestippelde effect
		float angleRad = i * angleStep * (3.1415926535f / 180.0f); // Converteer naar radialen

		float dashX = centerX + dashedCircleRadius * std::cos(angleRad);
		float dashY = centerY + dashedCircleRadius * std::sin(angleRad);

		circleDash.setPosition(sf::Vector2(dashX, dashY));
		// Roteer het streepje zodat het naar buiten wijst vanaf het midden van de cirkel
		// +90 om de oriëntatie aan te passen
		window.draw(circleDash);
	}
}

// Draw scores



// Update en teken scores aan weerszijden van de middenlijn
	// Score speler 1 (linksboven)
player1ScoreText.setString(std::to_string(player1_score));
player1ScoreText.setPosition(sf::Vector2f(10, 20)); // Aangepast: X-coördinaat naar 10
window.draw(player1ScoreText);



// Score speler 2 (rechtsboven)
player2ScoreText.setString(std::to_string(player2_score));
// Positioneer rechtsboven. We gebruiken een geschatte breedte van 50 pixels voor de tekst
// om het consistent te houden zonder getGlobalBounds().width.
player2ScoreText.setPosition(sf::Vector2f(windowWidth - 50 - 20, 20)); // windowW++idth - geschatte_breedte - padding
window.draw(player2ScoreText);
break; // Einde van PLAYING case

				case
				GAME_OVER:
					std::cout << "DEBUG: Entering GAME_OVER state. Winner: " << winner << std::endl; // Debug output

					// Gradiënt achtergrond (gelijk aan MENU)
					RectangleShape topGradientGameOver(Vector2f(windowWidth, windowHeight / 2));
					topGradientGameOver.setFillColor(Color(0, 0, 50)); // Donkerblauw
					window.draw(topGradientGameOver);
					RectangleShape bottomGradientGameOver(Vector2f(windowWidth, windowHeight / 2));
					bottomGradientGameOver.setPosition(sf::Vector2f(0, windowHeight / 2));
					bottomGradientGameOver.setFillColor(Color(0, 0, 100)); // Iets lichter blauw
					window.draw(bottomGradientGameOver);

					// "Game Over!" titel (gelijk aan MENU's "Ping Pong!")
					messageText.setString("GAME OVER!");
					messageText.setFillColor(Color::Red); // Kleur voor Game Over
					messageText.setCharacterSize(80);
					messageText.setOutlineThickness(3); // Behoud omtrek
					messageText.setOutlineColor(Color::White); // Behoud omtrek kleur

					// Positionering: iets hoger dan het midden
					messageText.setPosition(sf::Vector2f(windowWidth / 2.0f - (messageText.getCharacterSize() * 4.0f) / 2.0f,
						windowHeight / 2.0f - (messageText.getCharacterSize() / 2.0f) - 50.0f));
					window.draw(messageText);

					// Winnaar bericht
					if (winner == 1) {
						winLoseMessageText.setString("Speler 1 Wint!\nGefeliciteerd!");
						winLoseMessageText.setFillColor(Color::Green);
					}
					else {
						winLoseMessageText.setString("Speler 2 Wint!\nMeer geluk volgende keer!");
						winLoseMessageText.setFillColor(Color::Red);
					}
					winLoseMessageText.setCharacterSize(40); // Iets kleiner dan de titel
					// Positionering: onder "GAME OVER!"
					winLoseMessageText.setPosition(sf::Vector2f(windowWidth / 2.0f -                                                                            winLoseMessageText.getCharacterSize() * 4.0f / 2.0f,
						windowHeight / 2.0f - (winLoseMessageText.getCharacterSize() / 2.0f) - 50.0f));
					window.draw(winLoseMessageText);
					std::cout << "DEBUG: Win/Verlies bericht getekend." << std::endl; // Debug output
					break; // Einde van GAME_OVER case

}
}



		//display wat er is getekend 
		window.display();




	}
	         return 0;


}
	










