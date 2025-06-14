# dog_jump_game
import pygame
import sys
import math
import random

pygame.init()

# Screen setup
WIDTH, HEIGHT = 800, 400
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("🐶 Dog Jump Game with Pixel Bird")
clock = pygame.time.Clock()
FPS = 60

# Colors
SKY_BLUE = (135, 206, 250)
YELLOW = (255, 223, 0)
GROUND_COLOR = (120, 120, 120)
BROWN = (139, 69, 15 )
DARK_BROWN = (100, 50, 20)
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
PINK = (255, 192, 203)
GREEN = (124, 252, 0)
YELLOW_BODY = (255, 220, 0)
ORANGE_BEAK = (255, 140, 0)
BROWN_OUTLINE = (139, 69, 19)

# Ground setup
GROUND_HEIGHT = 40
ground_y = HEIGHT - GROUND_HEIGHT

# Game variables
dog_x = 100
dog_y = ground_y - 60
dog_y_velocity = 0
is_jumping = False
gravity = 1
frame = 0
score = 0
font = pygame.font.SysFont(None, 48)

# Obstacles
obstacles = []
obstacle_timer = pygame.USEREVENT + 1
pygame.time.set_timer(obstacle_timer, 1500)

# Bird setup (pixel bird positions)
pixel_size = 5
bird_start_x = 600
bird_y = 100
bird_speed = 2
bird_spacing = 300

def draw_dog(surface, x, y, frame):
    pygame.draw.ellipse(surface, BROWN, (x, y + 30, 80, 40))  # Body
    pygame.draw.circle(surface, BROWN, (x + 90, y + 40), 25)  # Head
    pygame.draw.circle(surface, DARK_BROWN, (x + 75, y + 25), 10)
    pygame.draw.circle(surface, DARK_BROWN, (x + 105, y + 25), 10)
    pygame.draw.circle(surface, WHITE, (x + 83, y + 35), 5)
    pygame.draw.circle(surface, WHITE, (x + 97, y + 35), 5)
    pygame.draw.circle(surface, BLACK, (x + 83, y + 35), 2)
    pygame.draw.circle(surface, BLACK, (x + 97, y + 35), 2)
    pygame.draw.circle(surface, BLACK, (x + 90, y + 47), 4)  # Nose
    pygame.draw.rect(surface, PINK, (x + 88, y + 52, 5, 8))   # Tongue

    leg_offset = int(5 * math.sin(frame * 0.3)) if y >= ground_y - 60 else 0
    pygame.draw.rect(surface, BROWN, (x + 10, y + 65 + leg_offset, 10, 15))
    pygame.draw.rect(surface, BROWN, (x + 30, y + 65 - leg_offset, 10, 15))
    pygame.draw.rect(surface, BROWN, (x + 60, y + 65 + leg_offset, 10, 15))
    pygame.draw.rect(surface, BROWN, (x + 80, y + 65 - leg_offset, 10, 15))

    tail_end_x = x - 10 + int(5 * math.cos(frame * 0.3))
    tail_end_y = y + 40 + int(5 * math.sin(frame * 0.3))
    pygame.draw.line(surface, DARK_BROWN, (x - 10, y + 40), (tail_end_x, tail_end_y), 5)

def draw_reversed_bird(surface, x, y):
    bird_pixels_reversed = [
        (0, 4, YELLOW_BODY), (0, 3, YELLOW_BODY),
        (1, 5, YELLOW_BODY), (1, 4, YELLOW_BODY), (1, 3, YELLOW_BODY), (1, 2, YELLOW_BODY),
        (2, 6, YELLOW_BODY), (2, 5, YELLOW_BODY), (2, 4, YELLOW_BODY), (2, 3, YELLOW_BODY), (2, 2, YELLOW_BODY),
        (3, 6, YELLOW_BODY), (3, 5, YELLOW_BODY), (3, 4, YELLOW_BODY), (3, 3, YELLOW_BODY), (3, 2, YELLOW_BODY),
        (4, 5, YELLOW_BODY), (4, 4, YELLOW_BODY), (4, 3, YELLOW_BODY),
        (2, 1, ORANGE_BEAK), (2, 0, ORANGE_BEAK),
        (3, 1, ORANGE_BEAK), (3, 0, ORANGE_BEAK),
        (1, 2, WHITE), (2, 2, WHITE),
        (1, 1, WHITE), (2, 1, WHITE),
        (2, 1, BLACK),
        (0, 5, BROWN_OUTLINE), (0, 2, BROWN_OUTLINE),
        (1, 6, BROWN_OUTLINE), (1, 1, BROWN_OUTLINE),
        (4, 6, BROWN_OUTLINE), (4, 2, BROWN_OUTLINE),
        (5, 5, BROWN_OUTLINE), (5, 4, BROWN_OUTLINE), (5, 3, BROWN_OUTLINE),
        (2, -1, BROWN_OUTLINE), (3, -1, BROWN_OUTLINE),
        (1, 0, BROWN_OUTLINE), (2, 0, BROWN_OUTLINE), (3, 0, BROWN_OUTLINE),
        (4, 1, BROWN_OUTLINE),
        (0, 1, BROWN_OUTLINE),
        (3, 2, BROWN_OUTLINE), (3, 1, BROWN_OUTLINE),
    ]
    for row_offset, col_offset, color in bird_pixels_reversed:
        pygame.draw.rect(surface, color,
            (x + col_offset * pixel_size,
             y + row_offset * pixel_size,
             pixel_size, pixel_size))

# Bird positions (2 birds)
bird_positions = [bird_start_x, bird_start_x + bird_spacing]

# Game loop
running = True
while running:
    screen.fill(SKY_BLUE)
    pygame.draw.circle(screen, YELLOW, (WIDTH - 100, 80), 40)
    pygame.draw.rect(screen, GROUND_COLOR, (0, ground_y, WIDTH, GROUND_HEIGHT))

    # Events
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == obstacle_timer:
            rect = pygame.Rect(WIDTH, ground_y - 50, 50, 50)
            obstacles.append({"rect": rect, "scored": False})

    keys = pygame.key.get_pressed()
    if keys[pygame.K_SPACE] and not is_jumping:
        dog_y_velocity = -18
        is_jumping = True

    # Physics
    dog_y += dog_y_velocity
    dog_y_velocity += gravity
    if dog_y >= ground_y - 60:
        dog_y = ground_y - 60
        is_jumping = False

    # Draw dog
    draw_dog(screen, dog_x, dog_y, frame)
    dog_rect = pygame.Rect(dog_x, dog_y, 100, 80)  # Rough hitbox

    # Move and draw birds
    for i in range(len(bird_positions)):
        bird_positions[i] -= bird_speed
        if bird_positions[i] < -50:
            bird_positions[i] = WIDTH + random.randint(100, 200)
        draw_reversed_bird(screen, bird_positions[i], bird_y + (i * 20))

    # Draw obstacles
    for obs in obstacles[:]:
        obs["rect"].x -= 7
        pygame.draw.rect(screen, (255, 0, 0), obs["rect"])
        if dog_rect.colliderect(obs["rect"]):
            print(f"Game Over! Final Score: {score}")
            pygame.quit()
            sys.exit()
        if not obs["scored"] and obs["rect"].right < dog_x:
            score += 1
            obs["scored"] = True
        if obs["rect"].right < 0:
            obstacles.remove(obs)

    # Score
    score_text = font.render(f"Score: {score}", True, BLACK)
    screen.blit(score_text, (10, 10))

    pygame.display.update()
    clock.tick(FPS)
    frame += 1

pygame.quit()
