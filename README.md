# smash
import pygame
import sys

# Constants
WIDTH, HEIGHT = 800, 600
PLAYER_WIDTH, PLAYER_HEIGHT = 50, 80
GRAVITY = 0.8
JUMP_STRENGTH = -16
PLAYER_SPEED = 7
ATTACK_RANGE = 70
ATTACK_COOLDOWN = 25
MAX_HEALTH = 100

# Colors
WHITE = (255, 255, 255)
RED = (200, 50, 50)
BLUE = (50, 50, 200)
BLACK = (0, 0, 0)
GREEN = (0, 180, 0)

pygame.init()
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Python Smash Bros!")
clock = pygame.time.Clock()
font = pygame.font.SysFont("Arial", 28)

class Player:
    def __init__(self, x, y, color, left_keys, right_keys):
        self.rect = pygame.Rect(x, y, PLAYER_WIDTH, PLAYER_HEIGHT)
        self.color = color
        self.vel_y = 0
        self.on_ground = False
        self.health = MAX_HEALTH
        self.facing = 1 if x < WIDTH // 2 else -1
        self.attack_cooldown = 0
        self.left_keys = left_keys
        self.right_keys = right_keys

    def move(self, keys):
        # Horizontal
        dx = 0
        if keys[self.left_keys['left']]:
            dx -= PLAYER_SPEED
            self.facing = -1
        if keys[self.left_keys['right']]:
            dx += PLAYER_SPEED
            self.facing = 1

        self.rect.x += dx

        # Stay in bounds
        self.rect.x = max(0, min(WIDTH - PLAYER_WIDTH, self.rect.x))

        # Jump
        if keys[self.left_keys['jump']] and self.on_ground:
            self.vel_y = JUMP_STRENGTH
            self.on_ground = False

    def apply_gravity(self):
        self.vel_y += GRAVITY
        self.rect.y += int(self.vel_y)
        # Ground collision
        if self.rect.y >= HEIGHT - PLAYER_HEIGHT - 40:
            self.rect.y = HEIGHT - PLAYER_HEIGHT - 40
            self.vel_y = 0
            self.on_ground = True

    def attack(self, other):
        if self.attack_cooldown == 0:
            attack_rect = pygame.Rect(
                self.rect.centerx + self.facing * ATTACK_RANGE // 2,
                self.rect.y + 10,
                ATTACK_RANGE,
                PLAYER_HEIGHT - 20
            )
            if attack_rect.colliderect(other.rect):
                other.health -= 10
                other.vel_y = -10
                other.rect.x += self.facing * 40
            self.attack_cooldown = ATTACK_COOLDOWN

    def update(self):
        if self.attack_cooldown > 0:
            self.attack_cooldown -= 1

    def draw(self, surf):
        pygame.draw.rect(surf, self.color, self.rect)
        # Draw a face direction line
        pygame.draw.line(
            surf, BLACK, self.rect.center,
            (self.rect.centerx + self.facing * 30, self.rect.centery), 3
        )

def draw_health_bar(player, x, y):
    bar_width = 200
    health_ratio = max(0, player.health) / MAX_HEALTH
    pygame.draw.rect(screen, BLACK, (x, y, bar_width, 20))
    pygame.draw.rect(screen, GREEN, (x, y, bar_width * health_ratio, 20))

def main():
    # Controls: WASD for Player 1, Arrow keys for Player 2
    p1 = Player(150, HEIGHT - 200, RED, 
                {'left': pygame.K_a, 'right': pygame.K_d, 'jump': pygame.K_w, 'attack': pygame.K_SPACE},
                {})
    p2 = Player(WIDTH - 200, HEIGHT - 200, BLUE, 
                {'left': pygame.K_LEFT, 'right': pygame.K_RIGHT, 'jump': pygame.K_UP, 'attack': pygame.K_RETURN},
                {})

    running = True
    winner = None

    while running:
        clock.tick(60)
        screen.fill(WHITE)
        pygame.draw.rect(screen, (70, 180, 70), (0, HEIGHT-40, WIDTH, 40)) # ground

        keys = pygame.key.get_pressed()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

            # Attacks
            if event.type == pygame.KEYDOWN:
                if event.key == p1.left_keys['attack']:
                    p1.attack(p2)
                if event.key == p2.left_keys['attack']:
                    p2.attack(p1)

        # Player logic
        if not winner:
            p1.move(keys)
            p2.move(keys)
            p1.apply_gravity()
            p2.apply_gravity()
            p1.update()
            p2.update()

            if p1.health <= 0:
                winner = "Player 2 Wins!"
            if p2.health <= 0:
                winner = "Player 1 Wins!"

        p1.draw(screen)
        p2.draw(screen)
        draw_health_bar(p1, 50, 30)
        draw_health_bar(p2, WIDTH - 250, 30)

        screen.blit(font.render("Player 1", True, BLACK), (50, 5))
        screen.blit(font.render("Player 2", True, BLACK), (WIDTH - 250, 5))

        if winner:
            msg = font.render(winner + " Press R to restart.", True, BLACK)
            screen.blit(msg, (WIDTH//2 - msg.get_width()//2, HEIGHT//2))

            if keys[pygame.K_r]:
                main()
                return

        pygame.display.flip()

    pygame.quit()
    sys.exit()

if __name__ == "__main__":
    main()
