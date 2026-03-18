# t_rex_game.py
import pygame
import sys
import random

#initialize pygame
pygame.init()

#constants
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 200
GROUND_Y = 150
T_REX_WIDTH = 40
T_REX_HEIGHT = 40
OBSTACLE_WIDTH = 20
OBSTACLE_MIN_HEIGHT = 20
OBSTACLE_MAX_HEIGHT = 60
GRAVITY = 1
JUMP_STRENGTH = -15
INITIAL_SPEED = 4
SPEED_INCREMENT = 0.003

#colors
BLACK = (0, 0, 0)
PURPLE = (128, 0, 128)
LIGHT_PINK = (255, 182, 193)

# Set up the display
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("T-Rex Game")
clock = pygame.time.Clock()

# T-Rex properties
class TRex:
    def __init__(self):
        self.x = 50
        self.y = GROUND_Y - T_REX_HEIGHT
        self.width = T_REX_WIDTH
        self.height = T_REX_HEIGHT
        self.velocity_y = 0
        self.on_ground = True

    def jump(self):
        if self.on_ground:
            self.velocity_y = JUMP_STRENGTH
            self.on_ground = False

    def update(self):
        self.velocity_y += GRAVITY
        self.y += self.velocity_y

        if self.y >= GROUND_Y - T_REX_HEIGHT:
            self.y = GROUND_Y - T_REX_HEIGHT
            self.velocity_y = 0
            self.on_ground = True

    def draw(self):
        pygame.draw.rect(screen, PURPLE, (self.x, self.y, self.width, self.height))

# Obstacle properties
class Obstacle:
    def __init__(self, x, height):
        self.x = x
        self.y = GROUND_Y - height
        self.width = OBSTACLE_WIDTH
        self.height = height

    def update(self, speed):
        self.x -= speed

    def draw(self):
        pygame.draw.rect(screen, LIGHT_PINK, (self.x, self.y, self.width, self.height))

    def off_screen(self):
        return self.x + self.width < 0
    
def main():
    t_rex =TRex()
    obstacles = []
    speed = INITIAL_SPEED
    time_elapsed = 0
    font = pygame.font.SysFont("comicsansms", 30)
    game_over = False

    next_spawn_x = SCREEN_WIDTH - random.randint(200, 400)  # initial spawn position for the first obstacle

    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    if game_over:
                        # Reset the game
                        t_rex = TRex()
                        obstacles = []
                        speed = INITIAL_SPEED
                        speed = min(speed, 10)  # cap the speed increase
                        time_elapsed = 0
                        game_over = False
                        next_spawn_x = SCREEN_WIDTH - random.randint(200, 400)
                    else:
                        t_rex.jump()

        if not game_over:
            #update time
            time_elapsed += 1/60  # assuming 60 FPS

            #update speed
            speed += SPEED_INCREMENT

            #update t-rex
            t_rex.update()

            #generate new obstacles
            if not obstacles or obstacles[-1].x < next_spawn_x:
                obstacle_height = random.randint(OBSTACLE_MIN_HEIGHT, OBSTACLE_MAX_HEIGHT)
                obstacles.append(Obstacle(SCREEN_WIDTH, obstacle_height))
                next_spawn_x = SCREEN_WIDTH - random.randint(200, 400)

            #update obstacles
            for obstacle in obstacles[:]:
                obstacle.update(speed)
                if obstacle.off_screen():
                    obstacles.remove(obstacle)

            #check for collisions
            t_rex_rect = pygame.Rect(t_rex.x, t_rex.y, t_rex.width, t_rex.height)
            for obstacle in obstacles:
                obstacle_rect = pygame.Rect(obstacle.x, obstacle.y, obstacle.width, obstacle.height)
                if t_rex_rect.colliderect(obstacle_rect):
                    game_over = True
                    break

        #draw everything
        screen.fill(BLACK)

        #draw ground
        pygame.draw.line(screen, LIGHT_PINK, (0, GROUND_Y), (SCREEN_WIDTH, GROUND_Y), 2)

        #draw t-rex
        t_rex.draw()

        #draw obstacles
        for obstacle in obstacles:
            obstacle.draw()
        #draw time
        time_text = font.render(f"Time: {time_elapsed:.1f}s", True, LIGHT_PINK)
        screen.blit(time_text, (10, 10))

        if game_over:
            game_over_text = font.render("Game Over! Press SPACE to Restart", True, LIGHT_PINK)
            screen.blit(game_over_text, (SCREEN_WIDTH // 2 - 150, SCREEN_HEIGHT // 2))

        pygame.display.flip()
        clock.tick(60)

if __name__ == "__main__":
    main()
