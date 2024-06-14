#include <SFML/Graphics.hpp>
#include <iostream>
#include <vector>

using namespace sf;

int main() {
    // Create the window
    RenderWindow window(VideoMode(640, 640), "Space Shooter");
    window.setFramerateLimit(60);

    // Menu variables
    Font font;
    if (!font.loadFromFile("C:\\Users\\omerh\\OneDrive\\Desktop\\Glow Hockey\\neon.otf")) {
        // Error loading font
        return -1;
    }

    Text title("Space Shooter", font, 50);
    title.setPosition(200, 200);

    Text startGame("Start Game", font, 30);
    startGame.setPosition(260, 300);

    Text exitGame("Exit", font, 30);
    exitGame.setPosition(310, 350);

    bool inMenu = true;
    // Create a sprite and set its texture for the rocket
    Texture rocketTexture;
    if (!rocketTexture.loadFromFile("C:\\Users\\omerh\\OneDrive\\Desktop\\sherry\\rocket.png")) {
        // Error loading texture
        return -1;
    }
    Sprite sprite(rocketTexture);


    // Background
    Texture backgroundTexture;
    if (!backgroundTexture.loadFromFile("C:\\Users\\omerh\\OneDrive\\Desktop\\sherry\\background.png")) {
        // Error loading texture
        return -1;
    }
    Sprite backgroundS(backgroundTexture);

    // Set the initial position of the sprite
    sprite.setPosition(320, 510);

    // Set the scale of the sprite to make it smaller
    sprite.setScale(0.1f, 0.1f);
    

    // Set the speed of the sprite movement
    float playerSpeed = 200.0f;

    // Create a clock to measure elapsed time
    Clock clock;

    // Shooting variables
    bool isShooting = false;
    CircleShape bullet(5);  // Yellow circle representing the bullet
    bullet.setFillColor(Color::Yellow);
    float bulletSpeed = 400.0f;

    // Enemy variables
    const int numEnemies = 50;
    std::vector<Sprite> enemies(numEnemies);
    std::vector<Clock> respawnClocks(numEnemies);

    // Load the enemy texture
    Texture enemyTexture;
    if (!enemyTexture.loadFromFile("C:\\Users\\omerh\\OneDrive\\Desktop\\sherry\\enemy.png")) {
        // Error loading texture
        return -1;
    }

    // Initialize enemies' positions and textures
    for (int i = 0; i < numEnemies; ++i) {
        enemies[i].setTexture(enemyTexture);
        enemies[i].setPosition(100 + i * 100, 50);
        enemies[i].setScale(0.1f, 0.1f);
    }

    float enemySpeed = 100.0f;
    Time respawnTime = seconds(5.0f);

    // Flags to indicate whether enemies should disappear
    std::vector<bool> disappearEnemies(numEnemies, false);

    // Score variables
    int score = 0;

    // Main menu loop
    while (window.isOpen() && inMenu) {
        Event event;
        while (window.pollEvent(event)) {
            if (event.type == Event::Closed) {
                window.close();
            }
        }

        window.clear();
        Vector2f mousePosition = window.mapPixelToCoords(Mouse::getPosition(window));

        // Check if the mouse is over the "Start Game" text
        if (startGame.getGlobalBounds().contains(mousePosition)) {
            startGame.setFillColor(Color::Red);

            if (Mouse::isButtonPressed(Mouse::Left)) {
                // Start Game selected
                inMenu = false; // Exit the menu loop and start the game
            }
        }
        else {
            startGame.setFillColor(Color::White);
        }

        // Check if the mouse is over the "Exit" text
        if (exitGame.getGlobalBounds().contains(mousePosition)) {
            exitGame.setFillColor(Color::Red);

            if (Mouse::isButtonPressed(Mouse::Left)) {
                // Exit selected
                window.close();
            }
        }
        else {
            exitGame.setFillColor(Color::White);
        }

        window.clear();

        // Draw menu items
        window.draw(title);
        window.draw(startGame);
        window.draw(exitGame);

        window.display();
    }

    // Main game loop
    while (window.isOpen()) {
        Event event;
        while (window.pollEvent(event)) {
            if (event.type == Event::Closed) {
                window.close();
            }
            else if (event.type == Event::KeyPressed) {
                if (event.key.code == Keyboard::Space) {
                    // Spacebar pressed, initiate shooting
                    isShooting = true;

                    // Set the bullet position to the rocket position
                    bullet.setPosition(sprite.getPosition().x + sprite.getGlobalBounds().width / 2, sprite.getPosition().y);
                }
            }
        }

        Time elapsedTime = clock.restart();

        // Move the player horizontally
        if (Keyboard::isKeyPressed(Keyboard::Left)) {
            sprite.move(-playerSpeed * elapsedTime.asSeconds(), 0);
        }

        if (Keyboard::isKeyPressed(Keyboard::Right)) {
            sprite.move(playerSpeed * elapsedTime.asSeconds(), 0);
        }

        // Shooting logic
        if (isShooting) {
            bullet.move(0, -bulletSpeed * elapsedTime.asSeconds());

            // Check for collision with enemies
            for (int i = 0; i < numEnemies; ++i) {
                if (bullet.getGlobalBounds().intersects(enemies[i].getGlobalBounds()) && !disappearEnemies[i]) {
                    disappearEnemies[i] = true;
                    respawnClocks[i].restart();  // Reset the respawn clock
                    isShooting = false;
                    score += 5;
                }
            }

            if (bullet.getPosition().y < 0) {
                // Bullet is off-screen, reset
                isShooting = false;
            }
        }

        // Check for collision with player's rocket
        for (int i = 0; i < numEnemies; ++i) {
            if (sprite.getGlobalBounds().intersects(enemies[i].getGlobalBounds())) {
                // Display "Game Over" message
                sf::Font font;
                if (!font.loadFromFile("C:\\Users\\omerh\\OneDrive\\Desktop\\Glow Hockey\\neon.otf")) {
                    // Error loading font
                    return -1;
                }

                sf::Text gameOverMessage("Game Over! Score: " + std::to_string(score), font, 50);
                gameOverMessage.setPosition(150, 250);
                window.draw(gameOverMessage);
                window.display();

                // Wait for a moment before closing the window
                sf::sleep(sf::seconds(2));
                window.close();
            }
        }

        // Move enemies vertically (from top to bottom) or respawn if disappeared
        for (int i = 0; i < numEnemies; ++i) {
            if (!disappearEnemies[i]) {
                enemies[i].move(0, enemySpeed * elapsedTime.asSeconds());
            }
            else if (respawnClocks[i].getElapsedTime() > respawnTime) {
                enemies[i].setPosition(100 + i * 100, 50);
                disappearEnemies[i] = false;
            }
        }

        window.clear();
        // Draw background
        window.draw(backgroundS);
        // Draw the player sprite
        window.draw(sprite);

        // Draw the bullet if shooting
        if (isShooting) {
            window.draw(bullet);
        }

        // Draw enemies
        for (int i = 0; i < numEnemies; ++i) {
            if (!disappearEnemies[i]) {
                window.draw(enemies[i]);
            }
        }

        window.display();
    }
    return 0;
}