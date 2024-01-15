import random
import curses

# Constants
SCREEN_WIDTH = 10
SCREEN_HEIGHT = 22
SHAPES = [
    [
        [1, 1, 1, 1]
    ],
    [
        [2, 2],
        [2, 2]
    ],
    [
        [3, 3, 3],
        []
    ],
    [
        [4, 4],
        [4, 4]
    ],
    [
        [],
        [5, 5, 5, 5]
    ],
    [
        [6, 6],
        [6, 6, 6]
    ],
    [
        [7, 7, 7],
        [7]
    ]
]
SHAPE_COLORS = [
    curses.color_pair(1),
    curses.color_pair(2),
    curses.color_pair(3),
    curses.color_pair(4),
    curses.color_pair(5),
    curses.color_pair(6),
    curses.color_pair(7)
]

def random_shape():
    return random.choice(SHAPES)

def rotate(shape):
    return [row[::-1] for row in shape[::-1]]

def is_valid_move(shape, board, x, y):
    for c, row in enumerate(shape):
        for r, cell in enumerate(row):
            if cell and board[r + y][c + x]:
                return False
    return True

def merge(shape, board, x, y):
    for c, row in enumerate(shape):
        for r, cell in enumerate(row):
            if cell:
                board[r + y][c + x] = cell

def clear_lines(board):
    lines_cleared = 0
    new_board = [[0 for _ in range(SCREEN_WIDTH)] for _ in range(SCREEN_HEIGHT)]
    for y, row in enumerate(board):
        if all(cell != 0 for cell in row):
            lines_cleared += 1
        else:
            new_board[y - lines_cleared] = row
    return new_board, lines_cleared

def draw_board(stdscr, board):
    stdscr.clear()
    for y, row in enumerate(board):
        for x, cell in enumerate(row):
            if cell:
                stdscr.addch(y, x, '█', SHAPE_COLORS[cell - 1])
    stdscr.refresh()

def main(stdscr):
    curses.curs_set(0)
    curses.init_pair(1, curses.COLOR_WHITE, curses.COLOR_BLACK)
    curses.init_pair(2, curses.COLOR_GREEN, curses.COLOR_BLACK)
    curses.init_pair(3, curses.COLOR_RED, curses.COLOR_BLACK)
    curses.init_pair(4, curses.COLOR_BLUE, curses.COLOR_BLACK)
    curses.init_pair(5, curses.COLOR_YELLOW, curses.COLOR_BLACK)
    curses.init_pair(6, curses.COLOR_CYAN, curses.COLOR_BLACK)
    curses.init_pair(7, curses.COLOR_MAGENTA, curses.COLOR_BLACK)

    board = [[0 for _ in range(SCREEN_WIDTH)] for _ in range(SCREEN_HEIGHT)]
    shape = random_shape()
    x, y = SCREEN_WIDTH // 2 - len(shape[0]) // 2, 0

    while True:
        draw_board(stdscr, board)
        key = stdscr.getch()

        if key == curses.KEY_DOWN:
            if is_valid_move(shape, board, x, y + 1):
                y += 1
        elif key == curses.KEY_UP:
            new_shape = rotate(shape)
            if is_valid_move(new_shape, board, x, y):
                shape = new_shape
        elif key == curses.KEY_LEFT:
            if is_valid_move(shape, board, x - 1, y):
                x -= 1
        elif key == curses.KEY_RIGHT:
            if is_valid_move(shape, board, x + 1, y):
                x += 1
        elif key == ord(' '):
            while is_valid_move(shape, board, x, y + 1):
                y += 1
            merge(shape, board, x, y - 1)
            shape = random_shape()
            x, y = SCREEN_WIDTH // 2 - len(shape[0]) // 2, 0

        if is_valid_move(shape, board, x, y):
            merge(shape, board, x, y)
            shape = random_shape()
            x, y = SCREEN_WIDTH // 2 - len(shape[0]) // 2, 0
        else:
            board, lines_cleared = clear_lines(board)
            if lines_cleared > 0:
                stdscr.addstr(0, 0, f'Lines cleared: {lines_cleared}')
                stdscr.refresh()
                curses.napms(500)
            if is_game_over(board):
                draw_board(stdscr, board)
                stdscr.addstr(10, 3, 'Game Over! Press any key to exit...')
                stdscr.refresh()
                break

def is_game_over(board):
    for x in range(SCREEN_WIDTH):
        if is_valid_move(SHAPES[0], board, x, 0):
            return False
    return True

if __name__ == '__main__':
    curses.wrapper(main)
    
