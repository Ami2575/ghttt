import pygame
import random
import sys

# Initialize pygame
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Waterpret Avontuur")

# Colors
BLUE = (64, 164, 223)  # Water color
WHITE = (255, 255, 255)
RED = (255, 0, 0)
YELLOW = (255, 255, 0)
GREEN = (0, 255, 0)

# Player (the boy)
class Jongen:
    def __init__(self):
        self.x = WIDTH // 2
        self.y = HEIGHT // 2
        self.width = 40
        self.height = 60
        self.speed = 5
        self.air = 100
        self.score = 0
    
    def draw(self):
        pygame.draw.rect(screen, (0, 120, 255), (self.x, self.y, self.width, self.height))
        # Simple face
        pygame.draw.circle(screen, (255, 206, 160), (self.x + self.width//2, self.y + 15), 10)
    
    def move(self, keys):
        if keys[pygame.K_LEFT] and self.x > 0:
            self.x -= self.speed
        if keys[pygame.K_RIGHT] and self.x < WIDTH - self.width:
            self.x += self.speed
        if keys[pygame.K_UP] and self.y > 0:
            self.y -= self.speed
        if keys[pygame.K_DOWN] and self.y < HEIGHT - self.height:
            self.y += self.speed
        
        # Air decreases over time
        self.air -= 0.1
        if self.air < 0:
            self.air = 0

# Collectible items
class Voorwerp:
    def __init__(self):
        self.reset()
        self.type = random.choice(["eend", "bal", "frisbee"])
        
        if self.type == "eend":
            self.color = YELLOW
            self.value = 5
            self.size = 20
        elif self.type == "bal":
            self.color = RED
            self.value = 10
            self.size = 15
        else:  # frisbee
            self.color = GREEN
            self.value = 15
            self.size = 25
    
    def reset(self):
        self.x = random.randint(0, WIDTH - 30)
        self.y = random.randint(0, HEIGHT - 30)
    
    def draw(self):
        if self.type == "eend":
            pygame.draw.ellipse(screen, self.color, (self.x, self.y, 30, 20))
        else:
            pygame.draw.circle(screen, self.color, (self.x, self.y), self.size)

# Obstacles
class Obstakel:
    def __init__(self):
        self.x = random.randint(0, WIDTH - 40)
        self.y = random.randint(0, HEIGHT - 40)
        self.width = 40
        self.height = 40
        self.type = random.choice(["sprinkler", "bij"])
    
    def draw(self):
        if self.type == "sprinkler":
            pygame.draw.rect(screen, (70, 70, 70), (self.x, self.y, self.width, self.height))
        else:  # bij
            pygame.draw.ellipse(screen, (255, 255, 0), (self.x, self.y, 30, 20))
            pygame.draw.ellipse(screen, (0, 0, 0), (self.x, self.y, 30, 20), 2)

# Game setup
jongen = Jongen()
voorwerpen = [Voorwerp() for _ in range(5)]
obstakels = [Obstakel() for _ in range(3)]
clock = pygame.time.Clock()
font = pygame.font.SysFont(None, 36)

# Game loop
running = True
game_over = False

while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN and game_over:
            if event.key == pygame.K_RETURN:
                # Reset game
                jongen = Jongen()
                voorwerpen = [Voorwerp() for _ in range(5)]
                obstakels = [Obstakel() for _ in range(3)]
                game_over = False
    
    if not game_over:
        keys = pygame.key.get_pressed()
        jongen.move(keys)
        
        # Check for collisions with objects
        for v in voorwerpen:
            if (jongen.x < v.x + 30 and jongen.x + jongen.width > v.x and
                jongen.y < v.y + 30 and jongen.y + jongen.height > v.y):
                jongen.score += v.value
                jongen.air = min(100, jongen.air + 10)  # Get air from bubbles
                v.reset()
        
        # Check for collisions with obstacles
        for o in obstakels:
            if (jongen.x < o.x + o.width and jongen.x + jongen.width > o.x and
                jongen.y < o.y + o.height and jongen.y + jongen.height > o.y):
                if o.type == "sprinkler":
                    jongen.air -= 2
                else:  # bij
                    jongen.score = max(0, jongen.score - 5)
        
        # Check if out of air
        if jongen.air <= 0:
            game_over = True
    
    # Drawing
    screen.fill(BLUE)
    
    # Draw water surface
    pygame.draw.rect(screen, (100, 200, 255), (0, 0, WIDTH, 20))
    
    # Draw objects
    for v in voorwerpen:
        v.draw()
    
    # Draw obstacles
    for o in obstakels:
        o.draw()
    
    # Draw player
    jongen.draw()
    
    # Draw UI
    lucht_text = font.render(f"Lucht: {int(jongen.air)}", True, WHITE)
    score_text = font.render(f"Score: {jongen.score}", True, WHITE)
    screen.blit(lucht_text, (10, 10))
    screen.blit(score_text, (10, 50))
    
    if game_over:
        over_text = font.render("GAME OVER - Druk op ENTER om opnieuw te spelen", True, WHITE)
        screen.blit(over_text, (WIDTH//2 - 250, HEIGHT//2))
    
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
sys.exit()
