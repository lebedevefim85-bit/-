import pygame
import math
import time
import random
import asyncio

# Инициализация Pygame
pygame.init()

# Настройки экрана
WIDTH, HEIGHT = 900, 700
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Кто пишет отчет")

# Цвета
WHITE = (255, 255, 255)
BLACK = (40, 40, 40)
GOLD = (255, 215, 0)
SILVER = (192, 192, 192)
RED = (220, 20, 60)
GREEN = (50, 205, 50)
BLUE = (30, 144, 255)
PURPLE = (147, 112, 219)
DARK_BLUE = (25, 25, 112)
DANIIL_COLOR = (0, 100, 255)

# Параметры диска
CENTER = (WIDTH // 2, HEIGHT // 2 - 20)
RADIUS = 230
SECTORS = ["ДАННИЛ", "НИКИТА", "РАВИЛЬ"]
SECTOR_COLORS = [RED, GREEN, DANIIL_COLOR]

# Переменные игры
angle = 0
spinning = False
start_time = 0
result = None
spin_speed = 0
spin_count = 0
font_large = pygame.font.SysFont('arial', 52, bold=True)
font_medium = pygame.font.SysFont('arial', 36)
font_small = pygame.font.SysFont('arial', 28)

# Анимационные переменные
glow_effect = 0
glow_direction = 1
particles = []
celebration = False
celebration_time = 0

def create_particles(color=None):
    for _ in range(8):
        angle = random.uniform(0, 360)
        speed = random.uniform(2, 6)
        size = random.randint(4, 10)
        lifetime = random.uniform(1.5, 2.5)
        part_color = color if color else random.choice([BLUE, GOLD, GREEN, RED])
        particles.append({
            'pos': list(CENTER),
            'vel': [speed * math.cos(math.radians(angle)), speed * math.sin(math.radians(angle))],
            'size': size,
            'color': part_color,
            'lifetime': lifetime,
            'age': 0
        })

def update_particles():
    global particles
    for particle in particles[:]:
        particle['pos'][0] += particle['vel'][0]
        particle['pos'][1] += particle['vel'][1]
        particle['age'] += 0.04
        particle['vel'][1] += 0.08
        
        if particle['age'] >= particle['lifetime']:
            particles.remove(particle)

def draw_particles():
    for particle in particles:
        alpha = 255 * (1 - particle['age'] / particle['lifetime'])
        color = particle['color'] + (int(alpha),)
        s = pygame.Surface((particle['size']*2, particle['size']*2), pygame.SRCALPHA)
        pygame.draw.circle(s, color, (particle['size'], particle['size']), particle['size'])
        screen.blit(s, (particle['pos'][0]-particle['size'], particle['pos'][1]-particle['size']))

def draw_starry_background():
    screen.fill(DARK_BLUE)
    for _ in range(100):
        x = random.randint(0, WIDTH)
        y = random.randint(0, HEIGHT)
        size = random.randint(1, 3)
        brightness = random.randint(150, 255)
        pygame.draw.circle(screen, (brightness, brightness, brightness), (x, y), size)

def draw_disk():
    global glow_effect, glow_direction
    
    glow_effect += 0.06 * glow_direction
    if glow_effect > 1.0:
        glow_effect = 1.0
        glow_direction = -1
    elif glow_effect < 0.4:
        glow_effect = 0.4
        glow_direction = 1
    
    if spinning:
        glow_size = RADIUS + 20 + int(10 * glow_effect)
        s = pygame.Surface((glow_size*2, glow_size*2), pygame.SRCALPHA)
        pygame.draw.circle(s, (255, 255, 255, 100), (glow_size, glow_size), glow_size)
        screen.blit(s, (CENTER[0]-glow_size, CENTER[1]-glow_size))
    
    pygame.draw.circle(screen, SILVER, CENTER, RADIUS)
    pygame.draw.circle(screen, WHITE, CENTER, RADIUS - 12)
    
    sector_angle = 360 / len(SECTORS)
    for i in range(len(SECTORS)):
        start_angle = angle + i * sector_angle
        
        points = [CENTER]
        for a in range(int(start_angle), int(start_angle + sector_angle) + 2, 2):
            rad_angle = math.radians(a)
            x = CENTER[0] + (RADIUS - 15) * math.cos(rad_angle)
            y = CENTER[1] + (RADIUS - 15) * math.sin(rad_angle)
            points.append((x, y))
        points.append(CENTER)
        
        pygame.draw.polygon(screen, SECTOR_COLORS[i], points)
        pygame.draw.polygon(screen, BLACK, points, 2)
        
        text_angle_rad = math.radians(start_angle + sector_angle / 2)
        text_x = CENTER[0] + (RADIUS * 0.55) * math.cos(text_angle_rad)
        text_y = CENTER[1] + (RADIUS * 0.55) * math.sin(text_angle_rad)
        
        text_surface = font_medium.render(SECTORS[i], True, WHITE)
        text_rect = text_surface.get_rect(center=(text_x, text_y))
        
        bg_rect = text_rect.inflate(20, 10)
        pygame.draw.rect(screen, (0, 0, 0, 150), bg_rect, border_radius=5)
        pygame.draw.rect(screen, BLACK, bg_rect, 1, border_radius=5)
        
        screen.blit(text_surface, text_rect)
    
    pygame.draw.circle(screen, GOLD, CENTER, 25)
    pygame.draw.circle(screen, BLACK, CENTER, 25, 3)
    
    for i in range(8):
        decor_angle = i * 45
        rad_angle = math.radians(decor_angle)
        x = CENTER[0] + 15 * math.cos(rad_angle)
        y = CENTER[1] + 15 * math.sin(rad_angle)
        pygame.draw.line(screen, BLACK, CENTER, (x, y), 2)
    
    pointer_points = [
        (CENTER[0], CENTER[1] - RADIUS - 30),
        (CENTER[0] - 20, CENTER[1] - RADIUS + 5),
        (CENTER[0] + 20, CENTER[1] - RADIUS + 5)
    ]
    pygame.draw.polygon(screen, RED, pointer_points)
    pygame.draw.polygon(screen, BLACK, pointer_points, 3)
    
    pygame.draw.circle(screen, SILVER, (CENTER[0], CENTER[1] - RADIUS + 10), 12)
    pygame.draw.circle(screen, BLACK, (CENTER[0], CENTER[1] - RADIUS + 10), 12, 2)

def get_current_sector():
    normalized_angle = angle % 360
    pointer_angle = (normalized_angle + 180) % 360
    sector_angle = 360 / len(SECTORS)
    sector_index = int(pointer_angle / sector_angle)
    return SECTORS[sector_index]

def get_target_winner():
    global spin_count
    if spin_count <= 2:
        return "ДАННИЛ"
    else:
        return "ДАННИЛ" if random.random() < 0.5 else "НИКИТА"

def ensure_target_win(target_winner):
    sector_angle = 180 / len(SECTORS)
    target_index = SECTORS.index(target_winner)
    target_angle = (target_index * sector_angle + sector_angle / 2) % 360
    final_angle = (245 - target_angle) % 360
    random_offset = random.uniform(-10, 10)
    return (final_angle + random_offset) % 360

async def main():
    global running, angle, spinning, start_time, result, spin_speed, spin_count
    global glow_effect, glow_direction, particles, celebration, celebration_time
    
    running = True
    clock = pygame.time.Clock()
    
    while running:
        draw_starry_background()
        
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE and not spinning:
                    spinning = True
                    start_time = time.time()
                    spin_speed = 20
                    result = None
                    celebration = False
                    spin_count += 1
                    create_particles()
        
        if spinning or celebration:
            if random.random() < 0.2:
                create_particles()
            update_particles()
        
        if spinning:
            current_time = time.time()
            elapsed_time = current_time - start_time
            
            if elapsed_time < 5:
                progress = elapsed_time / 8
                if progress < 0.2:
                    spin_speed = 20 + 30 * progress * 5
                elif progress > 0.6:
                    spin_speed = 50 - 30 * (progress - 0.6) * 2.5
                else:
                    spin_speed = 50
                
                angle = (angle + spin_speed) % 360
            else:
                spinning = False
                celebration = True
                celebration_time = time.time()
                
                target_winner = get_target_winner()
                angle = ensure_target_win(target_winner)
                result = get_current_sector()
                
                if result != target_winner:
                    target_index = SECTORS.index(target_winner)
                    sector_angle = 360 / len(SECTORS)
                    angle = (target_index * sector_angle + sector_angle / 2 + 270 ) % 360
                    result = target_winner
                
                particle_color = RED if result == "ДАННИЛ" else GREEN
                for _ in range(25):
                    create_particles(particle_color)
        
        if celebration and time.time() - celebration_time > 3:
            celebration = False
        
        draw_particles()
        draw_disk()
        
        title = font_large.render("КТО ПИШЕТ ОТЧЕТ", True, GOLD)
        screen.blit(title, (WIDTH//2 - title.get_width()//2, 30))
        
        if result:
            result_color = RED if result == "ДАННИЛ" else GREEN
            result_text = f"🏆 Пишет отчет : {result}"
            result_surface = font_large.render(result_text, True, result_color)
            
            result_bg = pygame.Rect(WIDTH//2 - result_surface.get_width()//2 - 20, 
                                  HEIGHT - 180, 
                                  result_surface.get_width() + 40, 
                                  60)
            pygame.draw.rect(screen, (0, 0, 0, 180), result_bg, border_radius=15)
            pygame.draw.rect(screen, result_color, result_bg, 3, border_radius=15)
            
            screen.blit(result_surface, (WIDTH//2 - result_surface.get_width()//2, HEIGHT - 170))
            
            if celebration:
                win_text = "ПОЗДРАВЛЯЕМ! 🎉"
                win_surface = font_medium.render(win_text, True, GOLD)
                screen.blit(win_surface, (WIDTH//2 - win_surface.get_width()//2, HEIGHT - 220))
        
        if spinning:
            remaining = 5 - (time.time() - start_time)
            time_text = f"⏰ {remaining:.1f}с"
            time_surface = font_medium.render(time_text, True, (255, 100, 100))
            screen.blit(time_surface, (WIDTH//2 - time_surface.get_width()//2, 100))
        
        spin_text = f"Прокрутки: {spin_count}"
        spin_surface = font_small.render(spin_text, True, SILVER)
        screen.blit(spin_surface, (WIDTH - spin_surface.get_width() - 20, 10))
        
        if not spinning:
            instr_color = GOLD if result else (200, 200, 255)
            instr_text = "Нажмите SPACE чтобы крутить диск" if not result else "SPACE - крутить снова"
            instr_surface = font_small.render(instr_text, True, instr_color)
            screen.blit(instr_surface, (WIDTH//2 - instr_surface.get_width()//2, HEIGHT - 100))
        
        debug_text = f"Угол: {angle:.1f}° | Сектор: {get_current_sector() if spinning else result}"
        debug_surface = font_small.render(debug_text, True, SILVER)
        screen.blit(debug_surface, (10, 10))
        
        pygame.display.flip()
        clock.tick(60)
        await asyncio.sleep(0)
    
    pygame.quit()

# Запуск асинхронной версии для веба
asyncio.run(main())
