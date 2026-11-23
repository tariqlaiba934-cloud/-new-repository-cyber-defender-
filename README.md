"""
CYBER DEFENDER
Controls:  LEFT / RIGHT (ya A / D) = move,  SPACE = shoot,  R = restart
Goal: Viruses, phishing links aur hackers ko shoot karo.
      Green UPDATE ko shoot MAT karo, usay touch karo to extra life milti hai.
Install:  pip install pygame
Run:      python cyber_defender.py
"""
import random
import sys

import pygame

pygame.init()
W, H = 600, 700
screen = pygame.display.set_mode((W, H))
pygame.display.set_caption("Cyber Defender")
clock = pygame.time.Clock()

font = pygame.font.SysFont("consolas", 20, bold=True)
big_font = pygame.font.SysFont("consolas", 48, bold=True)
small_font = pygame.font.SysFont("consolas", 16)

# Threat name, colour
THREATS = [
    ("VIRUS", (255, 70, 70)),
    ("PHISH", (255, 170, 0)),
    ("HACKER", (190, 90, 255)),
]
SAFE = ("UPDATE", (60, 220, 120))

TIPS = [
    "Tip: Asli bank kabhi OTP nahi maangta.",
    "Tip: Har account ka alag password rakho.",
    "Tip: Two-factor authentication on rakho.",
    "Tip: Click karne se pehle ruko, socho, phir karo.",
    "Tip: Phone aur apps update rakho.",
]

# Peeche chalte hue taare (background ke liye)
stars = [[random.randint(0, W), random.randint(0, H), random.randint(1, 3)] for _ in range(60)]


def reset():
    return {
        "player_x": W // 2,
        "bullets": [],
        "items": [],
        "score": 0,
        "lives": 3,
        "level": 1,
        "last_shot": 0,
        "last_spawn": 0,
        "over": False,
        "tip": random.choice(TIPS),
    }


def draw_text(text, f, color, center):
    img = f.render(text, True, color)
    screen.blit(img, img.get_rect(center=center))


def draw_player(x):
    y = H - 60
    # Shield jaisa shape
    points = [(x, y - 25), (x + 28, y - 10), (x + 22, y + 20), (x, y + 30), (x - 22, y + 20), (x - 28, y - 10)]
    pygame.draw.polygon(screen, (0, 200, 255), points)
    pygame.draw.polygon(screen, (255, 255, 255), points, 3)
    pygame.draw.circle(screen, (255, 255, 255), (x, y), 6)


def spawn_item(g):
    if random.random() < 0.15:
        name, color = SAFE
        safe = True
    else:
        name, color = random.choice(THREATS)
        safe = False
    w = 90
    rect = pygame.Rect(random.randint(0, W - w), -40, w, 34)
    speed = 2 + g["level"] * 0.6 + random.random()
    g["items"].append({"rect": rect, "name": name, "color": color, "safe": safe, "speed": speed, "y": -40.0})


def update(g, keys, now):
    # Player movement
    if keys[pygame.K_LEFT] or keys[pygame.K_a]:
        g["player_x"] -= 7
    if keys[pygame.K_RIGHT] or keys[pygame.K_d]:
        g["player_x"] += 7
    g["player_x"] = max(30, min(W - 30, g["player_x"]))

    # Shooting
    if keys[pygame.K_SPACE] and now - g["last_shot"] > 220:
        g["bullets"].append(pygame.Rect(g["player_x"] - 2, H - 95, 4, 16))
        g["last_shot"] = now

    # Spawning (level badhne par jaldi jaldi aate hain)
    delay = max(350, 1100 - g["level"] * 90)
    if now - g["last_spawn"] > delay:
        spawn_item(g)
        g["last_spawn"] = now

    # Bullets move
    for b in g["bullets"]:
        b.y -= 10
    g["bullets"] = [b for b in g["bullets"] if b.bottom > 0]

    player_rect = pygame.Rect(g["player_x"] - 28, H - 90, 56, 60)

    for item in g["items"][:]:
        item["y"] += item["speed"]
        item["rect"].y = int(item["y"])

        # Bullet hit
        hit = None
        for b in g["bullets"]:
            if item["rect"].colliderect(b):
                hit = b
                break
        if hit:
            g["bullets"].remove(hit)
            g["items"].remove(item)
            if item["safe"]:
                g["score"] = max(0, g["score"] - 15)  # safe cheez tod di!
            else:
                g["score"] += 10
            continue

        # Player touch
        if item["rect"].colliderect(player_rect):
            g["items"].remove(item)
            if item["safe"]:
                g["lives"] = min(5, g["lives"] + 1)
                g["score"] += 5
            else:
                g["lives"] -= 1
            continue

        # Neeche pohanch gaya
        if item["rect"].top > H:
            g["items"].remove(item)
            if not item["safe"]:
                g["lives"] -= 1

    g["level"] = 1 + g["score"] // 100
    if g["lives"] <= 0:
        g["over"] = True


def draw(g):
    screen.fill((8, 10, 30))
    for s in stars:
        s[1] += s[2] * 0.5
        if s[1] > H:
            s[0], s[1] = random.randint(0, W), 0
        pygame.draw.circle(screen, (90, 100, 160), (s[0], int(s[1])), 1)

    for item in g["items"]:
        pygame.draw.rect(screen, item["color"], item["rect"], border_radius=8)
        pygame.draw.rect(screen, (255, 255, 255), item["rect"], 2, border_radius=8)
        draw_text(item["name"], small_font, (10, 10, 20), item["rect"].center)

    for b in g["bullets"]:
        pygame.draw.rect(screen, (255, 255, 120), b)

    draw_player(g["player_x"])

    # HUD
    screen.blit(font.render(f"Score: {g['score']}", True, (255, 255, 255)), (12, 10))
    screen.blit(font.render(f"Level: {g['level']}", True, (0, 200, 255)), (W // 2 - 45, 10))
    screen.blit(font.render("Lives: " + "<3 " * g["lives"], True, (255, 90, 120)), (W - 190, 10))

    if g["over"]:
        overlay = pygame.Surface((W, H), pygame.SRCALPHA)
        overlay.fill((0, 0, 0, 170))
        screen.blit(overlay, (0, 0))
        draw_text("GAME OVER", big_font, (255, 80, 80), (W // 2, H // 2 - 70))
        draw_text(f"Final Score: {g['score']}", font, (255, 255, 255), (W // 2, H // 2 - 10))
        draw_text(g["tip"], small_font, (120, 255, 170), (W // 2, H // 2 + 35))
        draw_text("R dabao dobara khelne ke liye", font, (200, 200, 200), (W // 2, H // 2 + 85))


def main():
    game = reset()
    while True:
        now = pygame.time.get_ticks()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN and event.key == pygame.K_r and game["over"]:
                game = reset()

        if not game["over"]:
            update(game, pygame.key.get_pressed(), now)
        draw(game)
        pygame.display.flip()
        clock.tick(60)


if __name__ == "__main__":
    main()
# -new-repository-cyber-defender-
