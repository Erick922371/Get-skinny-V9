import pygame
import sys
import random

pygame.init()

# Constants
WIDTH, HEIGHT = 800, 600
FPS = 60
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GREEN = (50, 200, 50)
RED = (200, 50, 50)
BLUE = (50, 50, 255)
GRAY = (200, 200, 200)

# Screen
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Healthy Diet Adventure")
clock = pygame.time.Clock()

font = pygame.font.SysFont("Arial", 28)
big_font = pygame.font.SysFont("Arial", 40)

# Load sprites (Use 8-bit style images here)
player_sprites = [
    pygame.image.load("avatar1.png"),
    pygame.image.load("avatar2.png")
]

enemy_img = pygame.image.load("junk_food.png")  # Replace with 8-bit junk food sprite
goal_img = pygame.image.load("healthy_meal.png")  # Replace with 8-bit healthy meal sprite
bg_color = (92, 148, 252)  # Light blue 8-bit sky background

# Player class
class Player(pygame.sprite.Sprite):
    def __init__(self, avatar_img):
        super().__init__()
        self.image = avatar_img
        self.rect = self.image.get_rect()
        self.rect.x = 50
        self.rect.y = HEIGHT - 100
        self.speed_y = 0
        self.jump_power = -15
        self.on_ground = True

    def update(self):
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            self.rect.x -= 5
        if keys[pygame.K_RIGHT]:
            self.rect.x += 5
        if keys[pygame.K_SPACE] and self.on_ground:
            self.speed_y = self.jump_power
            self.on_ground = False

        self.speed_y += 1  # Gravity
        self.rect.y += self.speed_y

        if self.rect.bottom >= HEIGHT - 50:
            self.rect.bottom = HEIGHT - 50
            self.speed_y = 0
            self.on_ground = True

# Enemy class
class Enemy(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = enemy_img
        self.rect = self.image.get_rect()
        self.rect.x = x
        self.rect.y = y
        self.move_direction = random.choice([-1, 1])
    
    def update(self):
        self.rect.x += self.move_direction * 2
        if self.rect.left < 0 or self.rect.right > WIDTH:
            self.move_direction *= -1

# Goal
class Goal(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = goal_img
        self.rect = self.image.get_rect()
        self.rect.x = WIDTH - 100
        self.rect.y = HEIGHT - 130

# Survey screen
def survey_screen():
    inputs = []
    questions = ["What's your name?", "What's your favorite fruit?", "Do you like veggies? (yes/no)"]
    
    for q in questions:
        answer = text_input(q)
        inputs.append(answer)

    return inputs

# Text input
def text_input(prompt):
    input_text = ''
    while True:
        screen.fill(WHITE)
        render_text = font.render(prompt, True, BLACK)
        screen.blit(render_text, (50, 150))

        user_input = font.render(input_text, True, BLUE)
        pygame.draw.rect(screen, GRAY, (50, 200, 700, 40))
        screen.blit(user_input, (60, 210))

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_RETURN:
                    return input_text
                elif event.key == pygame.K_BACKSPACE:
                    input_text = input_text[:-1]
                else:
                    input_text += event.unicode

        pygame.display.flip()
        clock.tick(FPS)

# Avatar selection screen
def avatar_select():
    selected = 0
    while True:
        screen.fill(WHITE)
        screen.blit(big_font.render("Choose Your Avatar", True, BLACK), (250, 50))

        for i, avatar in enumerate(player_sprites):
            x = 150 + i * 300
            border_color = GREEN if i == selected else GRAY
            pygame.draw.rect(screen, border_color, (x - 10, 190, 120, 120), 5)
            screen.blit(avatar, (x, 200))

        instructions = font.render("Use LEFT/RIGHT arrows and press ENTER to select", True, BLUE)
        screen.blit(instructions, (100, 400))

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_RIGHT:
                    selected = (selected + 1) % len(player_sprites)
                elif event.key == pygame.K_LEFT:
                    selected = (selected - 1) % len(player_sprites)
                elif event.key == pygame.K_RETURN:
                    return player_sprites[selected]

        pygame.display.flip()
        clock.tick(FPS)

# Game over screen
def show_message(text, color):
    screen.fill(WHITE)
    msg = big_font.render(text, True, color)
    screen.blit(msg, (WIDTH // 2 - msg.get_width() // 2, HEIGHT // 2 - 20))
    pygame.display.flip()
    pygame.time.delay(3000)

# Game loop
def game_loop(player_img):
    player = Player(player_img)
    player_group = pygame.sprite.Group(player)

    enemies = pygame.sprite.Group()
    for i in range(5):
        x = random.randint(200, WIDTH - 200)
        y = HEIGHT - 100
        enemies.add(Enemy(x, y))

    goal = Goal()
    goal_group = pygame.sprite.Group(goal)

    running = True
    while running:
        clock.tick(FPS)
        screen.fill(bg_color)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

        player_group.update()
        enemies.update()

        player_group.draw(screen)
        enemies.draw(screen)
        goal_group.draw(screen)

        # Check collisions
        if pygame.sprite.spritecollide(player, enemies, False):
            show_message("Oh no! Junk food got you!", RED)
            return False

        if pygame.sprite.spritecollide(player, goal_group, False):
            show_message("You reached your healthy meal!", GREEN)
            return True

        pygame.display.flip()

# Main
def main():
    survey_results = survey_screen()
    selected_avatar = avatar_select()
    result = game_loop(selected_avatar)
    if result:
        print("Game Complete: Healthy meal reached!")
    else:
        print("Game Over: Try again!")

if __name__ == "__main__":
    main()
