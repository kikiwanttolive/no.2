# no.2
import tkinter as tk
import numpy as np

class ReversiGame:
    def __init__(self, root):
        self.root = root
        self.root.title("リバーシ")
        self.canvas = tk.Canvas(root, width=640, height=640, bg='green')
        self.canvas.pack()
        self.board = np.zeros((8, 8), dtype=int)
        self.initialize_board()
        self.current_player = 1  # 黒が先手
        self.draw_board()
        self.canvas.bind("<Button-1>", self.click_handler)
    
    def initialize_board(self):
        # 設定
        self.board[3, 3], self.board[4, 4] = 2, 2  # 白
        self.board[3, 4], self.board[4, 3] = 1, 1  # 黒

    def draw_board(self):
        # 盤面
        self.canvas.delete("all")
        for i in range(8):
            for j in range(8):
                x0, y0 = j * 80, i * 80
                x1, y1 = x0 + 80, y0 + 80
                self.canvas.create_rectangle(x0, y0, x1, y1, outline="black")
                if self.board[i, j] == 1:
                    self.canvas.create_oval(x0 + 10, y0 + 10, x1 - 10, y1 - 10, fill="black")
                elif self.board[i, j] == 2:
                    self.canvas.create_oval(x0 + 10, y0 + 10, x1 - 10, y1 - 10, fill="white")

    def click_handler(self, event):
        # マウス操作
        col = event.x // 80
        row = event.y // 80
        if self.valid_move(row, col, self.current_player):
            self.make_move(row, col, self.current_player)
            self.current_player = 3 - self.current_player
            self.draw_board()
            if not self.valid_moves_exist(self.current_player):
                if not self.valid_moves_exist(3 - self.current_player):
                    self.end_game()
                else:
                    self.current_player = 3 - self.current_player
                    self.draw_board()
                    if not self.valid_moves_exist(self.current_player):
                        self.end_game()

    def valid_move(self, row, col, player):
        # 判定
        if self.board[row, col] != 0:
            return False
        opponent = 3 - player
        directions = [(-1, 0), (1, 0), (0, -1), (0, 1), (-1, -1), (1, 1), (-1, 1), (1, -1)]
        for dr, dc in directions:
            r, c = row + dr, col + dc
            if 0 <= r < 8 and 0 <= c < 8 and self.board[r, c] == opponent:
                while 0 <= r < 8 and 0 <= c < 8:
                    r += dr
                    c += dc
                    if not (0 <= r < 8 and 0 <= c < 8):
                        break
                    if self.board[r, c] == player:
                        return True
                    if self.board[r, c] == 0:
                        break
        return False

    def make_move(self, row, col, player):
        # 重複
        self.board[row, col] = player
        opponent = 3 - player
        directions = [(-1, 0), (1, 0), (0, -1), (0, 1), (-1, -1), (1, 1), (-1, 1), (1, -1)]
        for dr, dc in directions:
            r, c = row + dr, col + dc
            if 0 <= r < 8 and 0 <= c < 8 and self.board[r, c] == opponent:
                positions_to_flip = []
                while 0 <= r < 8 and 0 <= c < 8:
                    if self.board[r, c] == 0:
                        break
                    if self.board[r, c] == player:
                        for rr, cc in positions_to_flip:
                            self.board[rr, cc] = player
                        break
                    positions_to_flip.append((r, c))
                    r += dr
                    c += dc

    def valid_moves_exist(self, player):
        # 確認
        for row in range(8):
            for col in range(8):
                if self.valid_move(row, col, player):
                    return True
        return False

    def end_game(self):
        # ゲーム終了
        black_count = np.sum(self.board == 1)
        white_count = np.sum(self.board == 2)
        result = f"ゲーム終了。黒: {black_count}, 白: {white_count}\n"
        if black_count > white_count:
            result += "黒 win！！！"
        elif white_count > black_count:
            result += "白 win！！！"
        else:
            result += "引き分け！！！"
        self.canvas.create_text(320, 320, text=result, font=("Helvetica", 24), fill="red")

if __name__ == "__main__":
    root = tk.Tk()
    game = ReversiGame(root)
    root.mainloop()
