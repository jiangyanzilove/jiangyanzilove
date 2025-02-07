import pygame
import random

# 初始化Pygame
pygame.init()

# 游戏窗口设置
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("生态守护者 - Environmental Defender")

# 颜色定义
WHITE = (255, 255, 255)
GREEN = (0, 128, 0)
BROWN = (139, 69, 19)
BLUE = (0, 0, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)

# 游戏角色类
class Player(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = pygame.Surface((30, 40))
        self.image.fill(BLUE)
        self.rect = self.image.get_rect(center=(WIDTH//2, HEIGHT//2))
        self.speed = 5

    def update(self, keys):
        if keys[pygame.K_a]:
            self.rect.x -= self.speed
        if keys[pygame.K_d]:
            self.rect.x += self.speed
        if keys[pygame.K_w]:
            self.rect.y -= self.speed
        if keys[pygame.K_s]:
            self.rect.y += self.speed

# 污染物类
class Pollutant(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = pygame.Surface((20, 20))
        self.image.fill(RED)
        self.rect = self.image.get_rect(
            center=(random.randint(0, WIDTH), random.randint(0, HEIGHT))
        )

# 树木类
class Tree(pygame.sprite.Sprite):
    def __init__(self, pos):
        super().__init__()
        self.image = pygame.Surface((15, 30))
        self.image.fill(GREEN)
        self.rect = self.image.get_rect(center=pos)

# 游戏主类
class Game:
    def __init__(self):
        self.player = Player()
        self.pollutants = pygame.sprite.Group()
        self.trees = pygame.sprite.Group()
        self.score = 0
        self.clock = pygame.time.Clock()
        self.running = True
        
        # 初始化污染物
        for _ in range(10):
            self.pollutants.add(Pollutant())

    def run(self):
        while self.running:
            self.clock.tick(60)
            self.handle_events()
            self.update()
            self.draw()
            
    def handle_events(self):
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                self.running = False
                
            # 种植树木
            if event.type == pygame.MOUSEBUTTONDOWN:
                if self.score >= 10:  # 每种植一棵树需要10分
                    pos = pygame.mouse.get_pos()
                    self.trees.add(Tree(pos))
                    self.score -= 10

    def update(self):
        keys = pygame.key.get_pressed()
        self.player.update(keys)
        
        # 收集污染物
        collected = pygame.sprite.spritecollide(self.player, self.pollutants, True)
        self.score += len(collected) * 5
        
        # 自动生成新污染物
        if len(self.pollutants) < 10:
            self.pollutants.add(Pollutant())

    def draw(self):
        screen.fill(WHITE)
        
        # 绘制游戏元素
        screen.blit(self.player.image, self.player.rect)
        self.pollutants.draw(screen)
        self.trees.draw(screen)
        
        # 显示分数
        font = pygame.font.Font(None, 36)
        text = font.render(f"Score: {self.score}  树木: {len(self.trees)}", True, BLACK)
        screen.blit(text, (10, 10))
        
        # 显示提示文字
        help_text = font.render("A/W/S/D移动 鼠标点击种植树木（需10分）", True, BLACK)
        screen.blit(help_text, (10, HEIGHT-40))
        
        pygame.display.flip()

if __name__ == "__main__":
    game = Game()
    game.run()
    pygame.quit()
