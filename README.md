# game
import pygame
import random
import time

class GameSprite(pygame.sprite.Sprite):
    def __init__(self, player_x, player_y, player_speed, width, height):
        super().__init__()
        self.image = pygame.Surface((width, height))
        self.image.fill((255, 255, 255))  # Цвет ракетки - белый
        self.speed = player_speed
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y

    def reset(self):
        window.blit(self.image, (self.rect.x, self.rect.y))

class Player(GameSprite):
    def update_r(self):
        keys = pygame.key.get_pressed()
        if keys[pygame.K_UP] and self.rect.y > 5:
            self.rect.y -= self.speed
        if keys[pygame.K_DOWN] and self.rect.y < win_height - 80:
            self.rect.y += self.speed
    def update_l(self):
        keys = pygame.key.get_pressed()
        if keys[pygame.K_w] and self.rect.y > 5:
            self.rect.y -= self.speed
        if keys[pygame.K_s] and self.rect.y < win_height - 80:
            self.rect.y += self.speed

class Ball(GameSprite):
    def __init__(self, player_x, player_y, player_speed, width, height):
        super().__init__(player_x, player_y, player_speed, width, height)
        self.image = pygame.transform.scale(pygame.image.load('ball.png'), (width, height))
        self.base_speed = player_speed
        self.base_width = width
        self.base_height = height
        self.bonus_active = False
        self.bonus_end_time = 0

    def apply_bonus(self, bonus_type):
        self.bonus_active = True
        self.bonus_end_time = time.time() + 3  # бонус длится 3 секунды

        if bonus_type == "speed":
            self.speed = self.base_speed * 2
        elif bonus_type == "duplicate":
            global extra_balls
            extra_ball = Ball(self.rect.x, self.rect.y, self.base_speed, self.base_width, self.base_height)
            extra_balls.append(extra_ball)
        elif bonus_type == "shrink":
            self.image = pygame.transform.scale(pygame.image.load('ball.png'), (self.base_width // 2, self.base_height // 2))
            self.rect = self.image.get_rect(center=(self.rect.x, self.rect.y))

    def update(self):
        if self.bonus_active and time.time() > self.bonus_end_time:
            self.bonus_active = False
            self.speed = self.base_speed
            self.image = pygame.transform.scale(pygame.image.load('ball.png'), (self.base_width, self.base_height))
            self.rect = self.image.get_rect(center=(self.rect.x, self.rect.y))

class Bonus(GameSprite):
    def __init__(self, player_x, player_y, width, height, bonus_type):
        super().__init__(player_x, player_y, 0, width, height)
        self.image.fill((random.randint(0, 255), random.randint(0, 255), random.randint(0, 255)))
        self.bonus_type = bonus_type

pygame.init()
font = pygame.font.Font(None, 35)

back = (200, 255, 255)
win_width = 600
win_height = 500
window = pygame.display.set_mode((win_width, win_height))
window.fill(back)

game = True
finish = False
clock = pygame.time.Clock()
FPS = 60

racket1 = Player(30, 200, 4, 50, 150)
racket2 = Player(520, 200, 4, 50, 150)
ball = Ball(200, 200, 4, 50, 50)
extra_balls = []

score1 = 0
score2 = 0

speed_x = 3
speed_y = 3

bonus = None
bonus_timer = 0
bonus_interval = 7

while game:
    for e in pygame.event.get():
        if e.type == pygame.QUIT:
            game = False
  
    if not finish:
        window.fill(back)
        racket1.update_l()
        racket2.update_r()

        ball.rect.x += speed_x
        ball.rect.y += speed_y

        if pygame.sprite.collide_rect(racket1, ball) or pygame.sprite.collide_rect(racket2, ball):
            speed_x *= -1
      
        if ball.rect.y > win_height-50 or ball.rect.y < 0:
            speed_y *= -1

        if ball.rect.x < 0:
            score2 += 1
            ball.rect.x, ball.rect.y = win_width // 2, win_height // 2
            speed_x *= -1

        if ball.rect.x > win_width:
            score1 += 1
            ball.rect.x, ball.rect.y = win_width // 2, win_height // 2
            speed_x *= -1

        current_time = pygame.time.get_ticks() // 1000
        if current_time - bonus_timer > bonus_interval:
            bonus_x = random.randint(50, win_width - 50)
            bonus_y = random.randint(50, win_height - 50)
            bonus_type = random.choice(["speed", "duplicate", "shrink"])
            bonus = Bonus(bonus_x, bonus_y, 30, 30, bonus_type)
            bonus_timer = current_time

        if bonus and pygame.sprite.collide_rect(ball, bonus):
            ball.apply_bonus(bonus.bonus_type)
            bonus = None

        ball.update()
        racket1.reset()
        racket2.reset()
        ball.reset()

        for extra_ball in extra_balls:
            extra_ball.rect.x += speed_x
            extra_ball.rect.y += speed_y
            if pygame.sprite.collide_rect(racket1, extra_ball) or pygame.sprite.collide_rect(racket2, extra_ball):
                speed_x *= -1
            if extra_ball.rect.y > win_height-50 or extra_ball.rect.y < 0:
                speed_y *= -1
            if extra_ball.rect.x < 0 or extra_ball.rect.x > win_width:
                extra_balls.remove(extra_ball)
            extra_ball.reset()

        if bonus:
            bonus.reset()

        player1_text = font.render(f'Игрок 1: {score1}', True, (0, 0, 0))
        player2_text = font.render(f'Игрок 2: {score2}', True, (0, 0, 0))
        window.blit(player1_text, (10, 10))
        window.blit(player2_text, (win_width - player2_text.get_width() - 10, 10))

    pygame.display.update()
    clock.tick(FPS)
