import customtkinter as ctk
import random
import os
from PIL import Image
import pygame
import time

pygame.mixer.init()
ctk.set_appearance_mode("dark")

class JogoSniperFinal:
    def __init__(self, root):
        self.root = root
        self.root.title("Sniper Elite - Versão Final")
        self.root.geometry("800x800")
        
        self.caminho_pasta = r"C:\Users\vmeir\OneDrive\Área de Trabalho\Jogo teste"
        
        # --- CARREGAR RECURSOS ---
        try:
            self.som_tiro = pygame.mixer.Sound(os.path.join(self.caminho_pasta, "tiro.mp3"))
            self.som_dano = pygame.mixer.Sound(os.path.join(self.caminho_pasta, "dano.mp3"))
            self.som_grito = pygame.mixer.Sound(os.path.join(self.caminho_pasta, "grito.mp3"))
            
            self.pil_jogador = Image.open(os.path.join(self.caminho_pasta, "jogador.png")).convert("RGBA")
            self.pil_monstro = Image.open(os.path.join(self.caminho_pasta, "monstro.png")).convert("RGBA")
            self.pil_paisagem = Image.open(os.path.join(self.caminho_pasta, "paisagem.png")).convert("RGBA")
            self.pil_buraco = Image.open(os.path.join(self.caminho_pasta, "buraco.png")).convert("RGBA")
            self.pil_gato = Image.open(os.path.join(self.caminho_pasta, "gato.png")).convert("RGBA")
        except Exception as e:
            print(f"Erro ao carregar arquivos: {e}")

        self.dificuldade = "facil"
        self.exibir_menu_inicial()

    def exibir_menu_inicial(self):
        for widget in self.root.winfo_children():
            widget.destroy()

        img_paisagem_ctk = ctk.CTkImage(self.pil_paisagem, size=(800, 800))
        label_fundo = ctk.CTkLabel(self.root, image=img_paisagem_ctk, text="")
        label_fundo.place(x=0, y=0, relwidth=1, relheight=1)

        self.frame_menu = ctk.CTkFrame(self.root, fg_color="#222", corner_radius=20, border_width=2)
        self.frame_menu.place(relx=0.5, rely=0.5, anchor="center", relwidth=0.6, relheight=0.5)

        ctk.CTkLabel(self.frame_menu, text="SNIPER ELITE 2D", font=("Arial", 32, "bold")).pack(pady=30)
        
        btn_facil = ctk.CTkButton(self.frame_menu, text="FÁCIL (1s ataque)", fg_color="#28a745", 
                                 command=lambda: self.preparar_novo_jogo("facil"))
        btn_facil.pack(pady=10)

        btn_dificil = ctk.CTkButton(self.frame_menu, text="DIFÍCIL (0.5s + Lifesteal)", fg_color="#dc3545", 
                                   command=lambda: self.preparar_novo_jogo("dificil"))
        btn_dificil.pack(pady=10)

    def preparar_novo_jogo(self, diff):
        self.dificuldade = diff
        self.vidas_jogador = 3
        self.vidas_monstro = 3
        self.tiros_seguidos_no_monstro = 0
        self.acertos_totais = 0
        self.x_jogador = 400
        self.jogo_ativo = True
        
        for widget in self.root.winfo_children():
            widget.destroy()

        self.iniciar_interface()

    def iniciar_interface(self):
        img_paisagem_ctk = ctk.CTkImage(self.pil_paisagem, size=(800, 800))
        self.label_paisagem = ctk.CTkLabel(self.root, image=img_paisagem_ctk, text="")
        self.label_paisagem.place(x=0, y=0, relwidth=1, relheight=1)

        self.label_placar = ctk.CTkLabel(self.root, text="", font=("Arial", 18, "bold"), 
                                        text_color="white", fg_color="#333", corner_radius=10)
        self.label_placar.place(relx=0.5, rely=0.05, anchor="n")
        self.atualizar_placar()

        self.label_monstro = ctk.CTkLabel(self.label_paisagem, text="", fg_color="transparent")
        self.spawn_monstro()

        self.label_jogador = ctk.CTkLabel(self.label_paisagem, text="", fg_color="transparent")
        self.label_jogador.place(x=self.x_jogador, rely=0.9, anchor="center")
        self.atualizar_visual_jogador(0)

        self.label_buraco = ctk.CTkLabel(self.label_paisagem, image=ctk.CTkImage(self.pil_buraco, size=(30, 30)), text="")
        self.label_gato = ctk.CTkLabel(self.label_paisagem, image=ctk.CTkImage(self.pil_gato, size=(30, 30)), text="")

        self.label_monstro.bind("<Button-1>", self.clique_acerto)
        self.label_paisagem.bind("<Button-1>", self.clique_erro)
        self.root.bind("<a>", lambda e: self.mover_jogador(-35))
        self.root.bind("<d>", lambda e: self.mover_jogador(35))

        self.movimentar_monstro_loop()
        self.agendar_ataque_monstro()

    def agendar_ataque_monstro(self):
        if self.jogo_ativo:
            # Cadência de tiro: 500ms no difícil, 1000ms no fácil
            intervalo = 500 if self.dificuldade == "dificil" else 1000
            self.root.after(intervalo, self.lancar_gato)

    def lancar_gato(self):
        if not self.jogo_ativo: return
        x_m = self.label_monstro.winfo_x() + (self.label_monstro.winfo_width() // 2)
        y_m = self.label_monstro.winfo_y() + (self.label_monstro.winfo_height() // 2)
        
        divisor = 35 if self.dificuldade == "dificil" else 45
        vx, vy = (self.x_jogador - x_m) / divisor, (720 - y_m) / divisor
        
        self.label_gato.place(x=x_m, y=y_m)
        self.animar_gato(x_m, y_m, vx, vy)
        self.agendar_ataque_monstro()

    def animar_gato(self, x, y, vx, vy):
        if not self.jogo_ativo: return
        nx, ny = x + vx, y + vy
        if ((nx - self.x_jogador)**2 + (ny - 720)**2)**0.5 < 65:
            self.perder_vida_jogador()
            self.label_gato.place(x=-100, y=-100)
        elif ny > 800 or nx < -50 or nx > 850:
            self.label_gato.place(x=-100, y=-100)
        else:
            self.label_gato.place(x=nx, y=ny)
            self.root.after(20, lambda: self.animar_gato(nx, ny, vx, vy))

    def perder_vida_jogador(self):
        if not self.jogo_ativo: return
        self.som_dano.play()
        self.vidas_jogador -= 1
        if self.dificuldade == "dificil" and self.vidas_monstro < 3:
            self.vidas_monstro += 1
            self.label_monstro.configure(fg_color="#00FF00")
            self.root.after(200, lambda: self.label_monstro.configure(fg_color="transparent"))
        self.label_paisagem.configure(fg_color="#AA0000")
        self.root.after(100, lambda: self.label_paisagem.configure(fg_color="transparent"))
        self.atualizar_placar()
        if self.vidas_jogador <= 0: self.game_over()

    def clique_acerto(self, event):
        if not self.jogo_ativo: return
        self.acertos_totais += 1
        self.tiros_seguidos_no_monstro += 1
        if self.tiros_seguidos_no_monstro >= 2:
            self.vidas_monstro -= 1
            self.tiros_seguidos_no_monstro = 0
            self.som_grito.play()
        if self.vidas_monstro <= 0:
            self.vitoria()
        else:
            x_m = self.label_monstro.winfo_x() + (self.label_monstro.winfo_width() // 2)
            y_m = self.label_monstro.winfo_y() + (self.label_monstro.winfo_height() // 2)
            self.efeito_tiro(x_m, y_m)
            self.atualizar_placar()
            self.spawn_monstro()
        return "break"

    def clique_erro(self, event):
        if self.jogo_ativo:
            self.tiros_seguidos_no_monstro = 0
            self.efeito_tiro(event.x, event.y)
            self.atualizar_placar()

    def movimentar_monstro_loop(self):
        if self.jogo_ativo:
            self.spawn_monstro()
            self.root.after(2000, self.movimentar_monstro_loop)

    def spawn_monstro(self):
        if not self.jogo_ativo: return
        tam = random.randint(35, 75)
        self.label_monstro.configure(image=ctk.CTkImage(self.pil_monstro, size=(tam, tam)))
        self.label_monstro.place(x=random.randint(50, 700), y=random.randint(100, 500))

    def efeito_tiro(self, x, y):
        self.som_tiro.play()
        self.label_jogador.place(rely=0.86)
        self.atualizar_visual_jogador(-15)
        self.label_buraco.place(x=x-15, y=y-15)
        self.root.after(100, lambda: [self.atualizar_visual_jogador(0), self.label_jogador.place(rely=0.9)])
        self.root.after(200, lambda: self.label_buraco.place(x=-100, y=-100))

    def atualizar_visual_jogador(self, angulo):
        img = ctk.CTkImage(self.pil_jogador.rotate(angulo, expand=True), size=(160, 160))
        self.label_jogador.configure(image=img)

    def mover_jogador(self, delta):
        nx = self.x_jogador + delta
        if 80 <= nx <= 720: 
            self.x_jogador = nx
            self.label_jogador.place(x=nx)

    def atualizar_placar(self):
        v_j, v_m = "❤️" * self.vidas_jogador, "👾" * self.vidas_monstro
        self.label_placar.configure(text=f"[{self.dificuldade.upper()}] VOCÊ: {v_j} | BOSS: {v_m}\nCombo: {self.tiros_seguidos_no_monstro}/2")

    def vitoria(self): self.mostrar_menu_final("VITÓRIA!", "gold")
    def game_over(self): self.mostrar_menu_final("GAME OVER", "#FF4444")

    def mostrar_menu_final(self, titulo, cor):
        self.jogo_ativo = False
        frame = ctk.CTkFrame(self.label_paisagem, fg_color="#222", corner_radius=20, border_width=2)
        frame.place(relx=0.5, rely=0.5, anchor="center", relwidth=0.5, relheight=0.4)
        ctk.CTkLabel(frame, text=titulo, font=("Arial", 35, "bold"), text_color=cor).pack(pady=20)
        ctk.CTkButton(frame, text="REPETIR", fg_color="#28a745", command=self.exibir_menu_inicial).pack(pady=10)
        ctk.CTkButton(frame, text="SAIR", fg_color="#dc3545", command=self.root.destroy).pack(pady=10)

if __name__ == "__main__":
    janela = ctk.CTk()
    app = JogoSniperFinal(janela)
    janela.mainloop()
