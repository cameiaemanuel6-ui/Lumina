import pygame
import random
import math

pygame.init()

WIDTH, HEIGHT = 1280, 720
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Zombie Survival Prototype")

clock = pygame.time.Clock()

WORLD_W = 4000
WORLD_H = 4000

WHITE = (255,255,255)
RED = (220,50,50)
GREEN = (50,220,50)
BLACK = (20,20,20)
GRAY = (60,60,60)
YELLOW = (255,220,0)

font = pygame.font.SysFont(None, 36)

player_pos = [WORLD_W//2, WORLD_H//2]
player_speed = 6
player_hp = 100

bullets = []
zombies = []

kills = 0

camera_x = 0
camera_y = 0

spawn_timer = 0

class Zombie:
    def __init__(self):
        side = random.randint(0,3)

        if side == 0:
            self.x = random.randint(0,WORLD_W)
            self.y = 0
        elif side == 1:
            self.x = random.randint(0,WORLD_W)
            self.y = WORLD_H
        elif side == 2:
            self.x = 0
            self.y = random.randint(0,WORLD_H)
        else:
            self.x = WORLD_W
            self.y = random.randint(0,WORLD_H)

        self.hp = 30
        self.speed = random.uniform(1.5,2.5)

    def update(self):
        global player_hp

        dx = player_pos[0] - self.x
        dy = player_pos[1] - self.y

        dist = math.hypot(dx,dy)

        if dist > 0:
            self.x += dx/dist*self.speed
            self.y += dy/dist*self.speed

        if dist < 25:
            player_hp -= 0.15

running = True

while running:

    dt = clock.tick(60)

    spawn_timer += dt

    if spawn_timer > 1000:
        spawn_timer = 0
        zombies.append(Zombie())

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        if event.type == pygame.MOUSEBUTTONDOWN:

            mx,my = pygame.mouse.get_pos()

            wx = mx + camera_x
            wy = my + camera_y

            dx = wx-player_pos[0]
            dy = wy-player_pos[1]

            dist = math.hypot(dx,dy)

            if dist > 0:
                bullets.append([
                    player_pos[0],
                    player_pos[1],
                    dx/dist*12,
                    dy/dist*12
                ])

    keys = pygame.key.get_pressed()

    if keys[pygame.K_w]:
        player_pos[1] -= player_speed

    if keys[pygame.K_s]:
        player_pos[1] += player_speed

    if keys[pygame.K_a]:
        player_pos[0] -= player_speed

    if keys[pygame.K_d]:
        player_pos[0] += player_speed

    player_pos[0] = max(0,min(WORLD_W,player_pos[0]))
    player_pos[1] = max(0,min(WORLD_H,player_pos[1]))

    for bullet in bullets[:]:
        bullet[0] += bullet[2]
        bullet[1] += bullet[3]

        if (
            bullet[0] < 0 or
            bullet[1] < 0 or
            bullet[0] > WORLD_W or
            bullet[1] > WORLD_H
        ):
            bullets.remove(bullet)

    for zombie in zombies[:]:

        zombie.update()

        for bullet in bullets[:]:

            d = math.hypot(
                zombie.x-bullet[0],
                zombie.y-bullet[1]
            )

            if d < 20:

                zombie.hp -= 15

                if bullet in bullets:
                    bullets.remove(bullet)

                if zombie.hp <= 0:
                    zombies.remove(zombie)
                    kills += 1

                break

    camera_x = player_pos[0] - WIDTH//2
    camera_y = player_pos[1] - HEIGHT//2

    screen.fill(GRAY)

    pygame.draw.rect(
        screen,
        (40,120,40),
        (-camera_x,-camera_y,WORLD_W,WORLD_H)
    )

    for bullet in bullets:
        pygame.draw.circle(
            screen,
            YELLOW,
            (
                int(bullet[0]-camera_x),
                int(bullet[1]-camera_y)
            ),
            4
        )

    for zombie in zombies:
        pygame.draw.circle(
            screen,
            RED,
            (
                int(zombie.x-camera_x),
                int(zombie.y-camera_y)
            ),
            18
        )

    pygame.draw.circle(
        screen,
        GREEN,
        (
            int(player_pos[0]-camera_x),
            int(player_pos[1]-camera_y)
        ),
        20
    )

    hp_text = font.render(
        f"HP: {int(player_hp)}",
        True,
        WHITE
    )

    kill_text = font.render(
        f"Kills: {kills}",
        True,
        WHITE
    )

    screen.blit(hp_text,(20,20))
    screen.blit(kill_text,(20,60))

    if player_hp <= 0:

        over = font.render(
            "GAME OVER",
            True,
            RED
        )

        screen.blit(
            over,
            (
                WIDTH//2-100,
                HEIGHT//2
            )
        )

    pygame.display.flip()

pygame.quit()
