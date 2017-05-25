import pygame as pg                 # simplifie l'écriture de pygame dans le code
import random                       # fonciton Alea
from settings import *              # importation ( fonction etc...) depuis les settings
from sprites import *               # importation ( fonction etc... ) depuis sprites

class Game:
    def __init__(self):
        # initialiser fenetre de jeux
        pg.init()       #initialise module pygame
        self.screen = pg.display.set_mode((WIDTH, HEIGHT))  # initialise l'affichage de la fenetre windos
        pg.display.set_caption(TITLE)      #  definie la légende de la fenêtre
        self.clock = pg.time.Clock()        #
        self.running = True

    def new(self):
        # lancer nouvelle partie
        self.all_sprites = pg.sprite.Group()       # crée un groupe avec tou les sprites
        self.platforms = pg.sprite.Group()
        self.player = Player(self)
        self.all_sprites.add(self.player)
        for plat in PLATFORM_LIST:       # éxplore la liste plateforme
            p = Platform(*plat)
            self.all_sprites.add(p)         "revoir execution des platform"
            self.platforms.add(p)
        self.run()

    def run(self):
        # boucle de jeux            permet le bon fonctionnement du jeux
        self.playing = True
        while self.playing:             # tant que le jeu est lancé
            self.clock.tick(FPS)        # execute correctement les frames par seconde
            self.events()               # les diférents event / update et draw/
            self.update()
            self.draw()

    def update(self):
        # boucle de jeux , update
        self.all_sprites.update()                       # dans boucle de jeux
        # verifie si le joueur percute une plateforme - seulement si il tombe
        if self.player.vel.y > 0:
            hits = pg.sprite.spritecollide(self.player, self.platforms, False)
            if hits:                                        # si collision
                self.player.pos.y = hits[0].rect.top
                self.player.vel.y = 0                       # donne a la velocité une valeur nul

        # faire descendre la fenêtre au 1/4
        if self.player.rect.top <= HEIGHT / 4:
            self.player.pos.y += abs(self.player.vel.y)      # donne la valeur absolue de la velocité au plats et a la position du player
            for plat in self.platforms:                         # pour pouvoir les faire descendre en même temps
                plat.rect.y += abs(self.player.vel.y)
                if plat.rect.top >= HEIGHT:                        # supprime les plats qui descende en dessous de l'écran ( qu'on ne voi plus )
                    plat.kill()

        #si le joueur meurt
        if self.player.rect.bottom > HEIGHT:                    # si le joueur descend de l'écran
            for sprite in self.all_sprites:
                sprite.rect.y -= max(self.player.vel.y, 10)        # donne une valeur max a la velocité qui
                if sprite.rect.bottom < 0:                          # si les sprites descende de l'écran
                    sprite.kill()                                   # alors on les supprime
        if len(self.platforms) == 0:                                # et donc si la liste de plateforme est vide
            self.playing = False                                    # alors RESTART

        while len(self.platforms) < 6:
            width = random.randrange(60, 100)                            # donne a width un nombre alea entre 50 et 100
            p = Platform(random.randrange(0, WIDTH - width),             # la position x -> compris entre 0 et et WIDTH - width
                         random.randrange(-50, -30),                      # position y -> compris entre -75 et -30 soit le haut de l'écran
                         width, 20)                                       # la longueur est donné par width et la hauteur est constante à 20
            self.platforms.add(p)
            self.all_sprites.add(p)                                        'revoir ' # ajoute les plateformes au groupe p



    def events(self):
        # boucle de jeux , events
        for event in pg.event.get():
            # verifie la fermeture de fenetre
            if event.type == pg.QUIT:
                if self.playing:
                    self.playing = False
                self.running = False
            if event.type == pg.KEYDOWN:             # appuie sur la touche espace pour sauter ( fais en dehors des " sprite /update  ; pour pas voler )
                if event.key == pg.K_SPACE:
                    self.player.jump()

    def draw(self):
        # boucle de jeux , dessin
        self.screen.fill(BLACK)
        self.all_sprites.draw(self.screen)
        #  apres avoir tou dessiner , actualise tout
        pg.display.flip()



g = Game()
g.show_start_screen()
while g.running:
    g.new()
    g.show_go_screen()

pg.quit()
