# Game
# Game
Tic Tac Toe Game
import pygame
import sys
import math

# Initialize
pygame.init()
WIDTH, HEIGHT = 600, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Neon XO Game")
clock = pygame.time.Clock()

# Colors
BLACK = (10, 10, 10)
NEON_BLUE = (0, 255, 255)
NEON_PINK = (255, 20, 147)
NEON_GREEN = (0, 255, 100)
WHITE = (255, 255, 255)
NEON_YELLOW = (255, 255, 0)

# Game Variables
board = [["" for _ in range(3)] for _ in range(3)]
current_player = "X"
game_over = False
winner = None
FONT = pygame.font.SysFont("consolas", 80)
LINE_WIDTH = 10
restart_button = None
start_time = pygame.time.get_ticks()

def draw_neon_line(start, end, color, width=LINE_WIDTH):
    glow_surface = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
    for glow in range(12, 0, -2):
        glow_color = (*color, 10 + 20 * (12 - glow))
        pygame.draw.line(glow_surface, glow_color, start, end, width + glow)
    screen.blit(glow_surface, (0, 0))
    pygame.draw.line(screen, color, start, end, width)

def draw_grid():
    for i in range(1, 3):
        # Vertical
        draw_neon_line((WIDTH // 3 * i, 0), (WIDTH // 3 * i, HEIGHT), NEON_BLUE)
        # Horizontal
        draw_neon_line((0, HEIGHT // 3 * i), (WIDTH, HEIGHT // 3 * i), NEON_BLUE)

def draw_marks():
    for row in range(3):
        for col in range(3):
            mark = board[row][col]
            if mark:
                color = NEON_PINK if mark == "X" else NEON_GREEN
                text = FONT.render(mark, True, color)
                text_rect = text.get_rect(center=(col * 200 + 100, row * 200 + 100))
                screen.blit(text, text_rect)

def check_winner():
    global game_over, winner
    lines = []

    for i in range(3):
        lines.append(board[i])  
        lines.append([board[0][i], board[1][i], board[2][i]])

    lines.append([board[0][0], board[1][1], board[2][2]])
    lines.append([board[0][2], board[1][1], board[2][0]])

    for line in lines:
        if line[0] == line[1] == line[2] != "":
            winner = line[0]
            game_over = True
            return

    if all(cell != "" for row in board for cell in row):
        game_over = True
        winner = "Draw"

def reset_game():
    global board, current_player, game_over, winner, start_time
    board = [["" for _ in range(3)] for _ in range(3)]
    current_player = "X"
    game_over = False
    winner = None
    start_time = pygame.time.get_ticks()

def draw_restart_button():
    button_rect = pygame.Rect(WIDTH // 2 - 100, HEIGHT - 80, 200, 50)
    pygame.draw.rect(screen, NEON_BLUE, button_rect, border_radius=12)
    label = pygame.font.SysFont("consolas", 28).render("Restart", True, BLACK)
    label_rect = label.get_rect(center=button_rect.center)
    screen.blit(label, label_rect)
    return button_rect

# Game Loop
while True:
    screen.fill(BLACK)

    if not game_over:
        draw_grid()
        draw_marks()

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if not game_over and event.type == pygame.MOUSEBUTTONDOWN:
            x, y = pygame.mouse.get_pos()
            row = y // 200
            col = x // 200
            if row < 3 and col < 3 and board[row][col] == "":
                board[row][col] = current_player
                check_winner()
                current_player = "O" if current_player == "X" else "X"

        elif game_over and event.type == pygame.MOUSEBUTTONDOWN:
            mouse_pos = pygame.mouse.get_pos()
            if restart_button and restart_button.collidepoint(mouse_pos):
                reset_game()

    # Show result
    if game_over:
        screen.fill(BLACK)
        time_elapsed = (pygame.time.get_ticks() - start_time) / 1000.0
        scale = 1 + 0.05 * math.sin(3 * time_elapsed)
        font_size = max(40, int(80 * scale))
        animated_font = pygame.font.SysFont("consolas", font_size, bold=True)

        msg = "Draw!" if winner == "Draw" else f"{winner} Wins!"
        result = animated_font.render(msg, True, NEON_YELLOW)
        result_rect = result.get_rect(center=(WIDTH // 2, HEIGHT // 2 - 60))
        screen.blit(result, result_rect)

        restart_button = draw_restart_button()
    else:
        restart_button = None

    pygame.display.update()
    clock.tick(60)
