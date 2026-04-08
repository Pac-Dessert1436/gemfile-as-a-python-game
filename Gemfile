import pygame
import random
import math

pygame.init()
pygame.display.set_caption("Gemfile")
SCREEN_WIDTH, SCREEN_HEIGHT = 800, 600
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
clock = pygame.time.Clock()
font = pygame.font.Font(None, 36)
pygame.mixer.init()
pygame.mixer.music.load("main_theme.mp3")
pygame.mixer.music.play(-1)


class Particle:
    def __init__(self, x, y, color):
        angle = random.uniform(0, 2 * math.pi)
        speed = random.uniform(2, 6)
        self.x = x
        self.y = y
        self.vx = math.cos(angle) * speed
        self.vy = math.sin(angle) * speed
        self.life = 30
        self.color = color
    
    def update(self):
        self.x += self.vx
        self.y += self.vy
        self.vy += 0.2  
        self.life -= 1
    
    def draw(self, surface):
        alpha = self.life / 30
        size = int(4 * alpha)
        if size > 0:
            pygame.draw.circle(surface, self.color, (int(self.x), int(self.y)), size)
    
    def is_dead(self):
        return self.life <= 0


class Ruby:
    def __init__(self):
        self.x = random.randint(50, SCREEN_WIDTH - 50)
        self.y = -30
        self.size = random.randint(15, 25)
        self.rotation = 0
    
    def update(self, fall_speed):
        self.y += fall_speed
        self.rotation += 2
    
    def draw(self, surface):
        crown = [
            (self.x, self.y - self.size),
            (self.x + self.size * 0.7, self.y - self.size * 0.3),
            (self.x + self.size * 0.3, self.y),
            (self.x - self.size * 0.3, self.y),
            (self.x - self.size * 0.7, self.y - self.size * 0.3),
        ]
        pavilion = [
            (self.x + self.size * 0.3, self.y),
            (self.x + self.size * 0.7, self.y - self.size * 0.3),
            (self.x, self.y + self.size * 0.8),
            (self.x - self.size * 0.7, self.y - self.size * 0.3),
            (self.x - self.size * 0.3, self.y),
        ]
        pygame.draw.polygon(surface, (220, 20, 60), crown)
        pygame.draw.polygon(surface, (180, 0, 30), pavilion)
        highlight = [
            (self.x, self.y - self.size),
            (self.x + self.size * 0.35, self.y - self.size * 0.5),
            (self.x, self.y - self.size * 0.2),
        ]
        pygame.draw.polygon(surface, (255, 100, 120), highlight)
        pygame.draw.polygon(surface, (139, 0, 0), crown, 2)
        pygame.draw.polygon(surface, (139, 0, 0), pavilion, 2)
        pygame.draw.line(surface, (100, 0, 0), (self.x, self.y - self.size), 
                         (self.x, self.y + self.size * 0.8), 1)
        pygame.draw.line(surface, (100, 0, 0), (self.x - self.size * 0.7, self.y - self.size * 0.3),
                         (self.x + self.size * 0.7, self.y - self.size * 0.3), 1)
    
    def check_collision(self, basket_x, basket_y, basket_width, basket_height):
        basket_left = basket_x - basket_width // 2
        basket_right = basket_x + basket_width // 2
        basket_top = basket_y - basket_height // 2
        return (basket_left < self.x < basket_right and 
                basket_top < self.y < basket_y + basket_height // 2)


class Basket:
    def __init__(self):
        self.x = SCREEN_WIDTH // 2
        self.y = SCREEN_HEIGHT - 80
        self.width = 100
        self.height = 40
        self.target_x = self.x
    
    def update(self, mouse_x):
        self.target_x = mouse_x
        self.target_x = max(self.width // 2, 
            min(SCREEN_WIDTH - self.width // 2, self.target_x))      
        self.x += (self.target_x - self.x) * 0.15
    
    def draw(self, surface):
        left = self.x - self.width // 2
        right = self.x + self.width // 2
        top = self.y - self.height // 2
        bottom = self.y + self.height // 2
        basket_shape = [
            (right + 10, top),
            (left - 10, top),
            (left, bottom),
            (right, bottom)
        ]
        pygame.draw.polygon(surface, (139, 69, 19), basket_shape)
        pygame.draw.polygon(surface, (101, 67, 33), basket_shape, 3)
        handle_points = []
        for i in range(20):
            angle = math.pi + (math.pi * i / 19)
            hx = self.x + ((self.width - 30) // 2 + 10) * math.cos(angle)
            hy = top + 15 * math.sin(angle)
            handle_points.append((hx, hy))
        if len(handle_points) >= 2:
            pygame.draw.lines(surface, (101, 67, 33), False, handle_points, 4)
        for i in range(1, 4):
            y_line = top + (self.height * i // 4)
            pygame.draw.line(surface, (160, 82, 45), (left + 5, y_line), (right - 5, y_line), 2)


class Game:
    def __init__(self):
        self.score = 0
        self.lives = 3
        self.game_over = False
        self.rubies = []
        self.particles = []
        self.basket = Basket()
        self.ruby_spawn_timer = 0
        self.ruby_spawn_delay = 60
        self.ruby_fall_speed = 4
        self.score_threshold = 100
    
    def reset(self):
        self.__init__()
    
    def spawn_ruby(self):
        self.ruby_spawn_timer += 1
        if self.ruby_spawn_timer >= self.ruby_spawn_delay:
            self.ruby_spawn_timer = 0
            self.rubies.append(Ruby())
    
    def update_difficulty(self):
        if self.score > 0 and self.score >= self.score_threshold:
            self.ruby_fall_speed = min(10, 4 + self.score // 80)
            self.ruby_spawn_delay = max(25, 60 - self.score // 15)
            self.score_threshold += 100
    
    def update(self):
        if self.game_over:
            return
        mouse_x, _ = pygame.mouse.get_pos()
        self.basket.update(mouse_x)
        self.spawn_ruby()
        for ruby in self.rubies[:]:
            ruby.update(self.ruby_fall_speed)
            if ruby.check_collision(
                self.basket.x, self.basket.y, self.basket.width, self.basket.height
            ):
                self.score += ruby.size
                for _ in range(8):
                    self.particles.append(Particle(ruby.x, ruby.y, (255, 50, 80)))
                self.rubies.remove(ruby)
            elif ruby.y > SCREEN_HEIGHT:
                self.lives -= 1
                self.rubies.remove(ruby)
                if self.lives <= 0:
                    self.game_over = True
        for particle in self.particles[:]:
            particle.update()
            if particle.is_dead():
                self.particles.remove(particle)
        self.update_difficulty()
    
    def draw_background(self, surface):
        surface.fill((25, 25, 40))
        X_OFFSET, Y_OFFSET = 125, 75  
        for i in range(50):
            x = (i * X_OFFSET) % SCREEN_WIDTH
            y = (i * Y_OFFSET) % SCREEN_HEIGHT
            brightness = (i % 3) + 1
            pygame.draw.circle(surface, (200, 200, 255), (x, y), brightness)
    
    def draw_ui(self, surface):
        score_text = font.render(f"Score: {self.score}", True, (255, 255, 255))
        surface.blit(score_text, (10, 10))
        lives_text = font.render(f"Lives: {self.lives}", True, (255, 100, 100))
        surface.blit(lives_text, (10, 50))
        hint_text = font.render("Move mouse to control basket", True, (150, 150, 150))
        surface.blit(hint_text, (SCREEN_WIDTH // 2 - 160, SCREEN_HEIGHT - 30))
    
    def draw_game_over(self, surface):
        game_over_text = font.render("- GAME OVER -", True, (255, 50, 50))
        final_score_text = font.render(f"Final Score: {self.score}", True, (255, 255, 255))
        restart_text = font.render("Press SPACE to Restart", True, (100, 255, 100))
        
        surface.blit(game_over_text, (SCREEN_WIDTH // 2 - 100, SCREEN_HEIGHT // 2 - 60))
        surface.blit(final_score_text, (SCREEN_WIDTH // 2 - 110, SCREEN_HEIGHT // 2))
        surface.blit(restart_text, (SCREEN_WIDTH // 2 - 150, SCREEN_HEIGHT // 2 + 60))
    
    def draw(self, surface):
        self.draw_background(surface) 
        if not self.game_over:
            for ruby in self.rubies:
                ruby.draw(surface)
            self.basket.draw(surface)
            for particle in self.particles:
                particle.draw(surface)
            self.draw_ui(surface)
        else:
            self.draw_game_over(surface)


game = Game()
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE and game.game_over:
                game.reset()
            elif event.key == pygame.K_ESCAPE:
                running = False
    
    game.update()
    game.draw(screen)
    
    pygame.display.flip()
    clock.tick(60)

pygame.quit()