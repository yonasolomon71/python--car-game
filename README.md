# python
the first python
import pygame
import random
import sys
import time

# Initialize Pygame
pygame.init()

# Game constants
SCREEN_WIDTH = 400
SCREEN_HEIGHT = 600
ROAD_WIDTH = 300
LANE_WIDTH = ROAD_WIDTH // 3
CAR_WIDTH = 40
CAR_HEIGHT = 70
OBSTACLE_WIDTH = 40
OBSTACLE_HEIGHT = 40
SPEED = 5
FPS = 60

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
GRAY = (100, 100, 100)

# Set up the display
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Car Racing Game")
clock = pygame.time.Clock()

# Font setup
font = pygame.font.SysFont(None, 36)
small_font = pygame.font.SysFont(None, 24)


class Car:
    def __init__(self):
        self.width = CAR_WIDTH
        self.height = CAR_HEIGHT
        self.x = SCREEN_WIDTH // 2 - self.width // 2
        self.y = SCREEN_HEIGHT - self.height - 20
        self.speed = 8
        self.color = RED

    def draw(self):
        # Car body
        pygame.draw.rect(screen, self.color, (self.x, self.y, self.width, self.height))

        # Car details (windows)
        pygame.draw.rect(screen, BLACK, (self.x + 5, self.y + 5, self.width - 10, self.height // 3))

        # Wheels
        pygame.draw.rect(screen, BLACK, (self.x - 5, self.y + 10, 5, 20))
        pygame.draw.rect(screen, BLACK, (self.x - 5, self.y + self.height - 30, 5, 20))
        pygame.draw.rect(screen, BLACK, (self.x + self.width, self.y + 10, 5, 20))
        pygame.draw.rect(screen, BLACK, (self.x + self.width, self.y + self.height - 30, 5, 20))

    def move_left(self):
        if self.x > (SCREEN_WIDTH - ROAD_WIDTH) // 2:
            self.x -= self.speed

    def move_right(self):
        if self.x < SCREEN_WIDTH - (SCREEN_WIDTH - ROAD_WIDTH) // 2 - self.width:
            self.x += self.speed

    def get_rect(self):
        return pygame.Rect(self.x, self.y, self.width, self.height)


class Obstacle:
    def __init__(self):
        self.width = OBSTACLE_WIDTH
        self.height = OBSTACLE_HEIGHT
        self.lane = random.randint(0, 2)
        lane_center = (SCREEN_WIDTH - ROAD_WIDTH) // 2 + LANE_WIDTH * self.lane + LANE_WIDTH // 2
        self.x = lane_center - self.width // 2
        self.y = -self.height
        self.speed = random.uniform(5, 8)
        self.color = (random.randint(0, 255), random.randint(0, 255), random.randint(0, 255))

    def draw(self):
        pygame.draw.rect(screen, self.color, (self.x, self.y, self.width, self.height))

    def update(self):
        self.y += self.speed

    def get_rect(self):
        return pygame.Rect(self.x, self.y, self.width, self.height)

    def is_off_screen(self):
        return self.y > SCREEN_HEIGHT


def draw_road():
    # Road background
    road_x = (SCREEN_WIDTH - ROAD_WIDTH) // 2
    pygame.draw.rect(screen, GRAY, (road_x, 0, ROAD_WIDTH, SCREEN_HEIGHT))

    # Lane markings
    lane1_x = road_x + LANE_WIDTH
    lane2_x = road_x + 2 * LANE_WIDTH

    # Draw dashed lines for lanes
    dash_length = 30
    gap_length = 20
    num_dashes = SCREEN_HEIGHT // (dash_length + gap_length) + 1

    for i in range(num_dashes):
        y_pos = i * (dash_length + gap_length) - (pygame.time.get_ticks() // 30) % (dash_length + gap_length)
        pygame.draw.rect(screen, WHITE, (lane1_x - 2, y_pos, 4, dash_length))
        pygame.draw.rect(screen, WHITE, (lane2_x - 2, y_pos, 4, dash_length))

    # Road edges
    pygame.draw.rect(screen, WHITE, (road_x, 0, 5, SCREEN_HEIGHT))
    pygame.draw.rect(screen, WHITE, (road_x + ROAD_WIDTH - 5, 0, 5, SCREEN_HEIGHT))


def show_score(score):
    score_text = font.render(f"Score: {score}", True, WHITE)
    screen.blit(score_text, (10, 10))


def show_game_over(score):
    overlay = pygame.Surface((SCREEN_WIDTH, SCREEN_HEIGHT))
    overlay.set_alpha(180)
    overlay.fill(BLACK)
    screen.blit(overlay, (0, 0))

    game_over_text = font.render("GAME OVER", True, RED)
    score_text = font.render(f"Score: {score}", True, WHITE)
    restart_text = small_font.render("Press SPACE to play again", True, WHITE)
    quit_text = small_font.render("Press ESC to quit", True, WHITE)

    screen.blit(game_over_text, (SCREEN_WIDTH // 2 - game_over_text.get_width() // 2, SCREEN_HEIGHT // 2 - 60))
    screen.blit(score_text, (SCREEN_WIDTH // 2 - score_text.get_width() // 2, SCREEN_HEIGHT // 2))
    screen.blit(restart_text, (SCREEN_WIDTH // 2 - restart_text.get_width() // 2, SCREEN_HEIGHT // 2 + 50))
    screen.blit(quit_text, (SCREEN_WIDTH // 2 - quit_text.get_width() // 2, SCREEN_HEIGHT // 2 + 80))


def show_start_screen():
    screen.fill(BLACK)
    title_text = font.render("CAR RACING GAME", True, WHITE)
    start_text = small_font.render("Press SPACE to start", True, WHITE)
    controls_text = small_font.render("Use LEFT and RIGHT arrows to move", True, WHITE)

    screen.blit(title_text, (SCREEN_WIDTH // 2 - title_text.get_width() // 2, SCREEN_HEIGHT // 2 - 60))
    screen.blit(start_text, (SCREEN_WIDTH // 2 - start_text.get_width() // 2, SCREEN_HEIGHT // 2))
    screen.blit(controls_text, (SCREEN_WIDTH // 2 - controls_text.get_width() // 2, SCREEN_HEIGHT // 2 + 50))


def game_loop():
    car = Car()
    obstacles = []
    score = 0
    obstacle_timer = 0
    obstacle_frequency = 1500  # milliseconds
    last_time = pygame.time.get_ticks()
    game_over = False

    while True:
        current_time = pygame.time.get_ticks()
        dt = current_time - last_time
        last_time = current_time

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:
                    pygame.quit()
                    sys.exit()
                if game_over and event.key == pygame.K_SPACE:
                    return  # Restart the game

        if game_over:
            show_game_over(score)
            pygame.display.update()
            continue

        # Handle continuous key presses
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            car.move_left()
        if keys[pygame.K_RIGHT]:
            car.move_right()

        # Update score
        score += dt * 0.01

        # Generate obstacles
        obstacle_timer += dt
        if obstacle_timer >= obstacle_frequency:
            obstacles.append(Obstacle())
            obstacle_timer = 0
            # Increase difficulty over time
            obstacle_frequency = max(500, obstacle_frequency - 10)

        # Update obstacles
        for obstacle in obstacles[:]:
            obstacle.update()
            if obstacle.is_off_screen():
                obstacles.remove(obstacle)
            elif car.get_rect().colliderect(obstacle.get_rect()):
                game_over = True

        # Draw everything
        screen.fill(BLACK)
        draw_road()
        car.draw()
        for obstacle in obstacles:
            obstacle.draw()
        show_score(int(score))

        pygame.display.update()
        clock.tick(FPS)


def main():
    while True:
        show_start_screen()
        pygame.display.update()

        # Wait for space to start
        waiting = True
        while waiting:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_SPACE:
                        waiting = False
                    if event.key == pygame.K_ESCAPE:
                        pygame.quit()
                        sys.exit()
            clock.tick(FPS)

        # Start the game
        game_loop()


if __name__ == "__main__":
    main()
