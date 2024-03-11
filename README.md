import turtle,os
import random

#Bredd,höjd delay i spelet på 100 ms. 
w = 1300
h = 800
food_size = 15
delay = 100

#Bakgrundsbild 
wn = turtle.Screen()
t= turtle.Turtle()
wn.addshape(os.path.expanduser("~/Desktop/grass.gif"))
t.shape(os.path.expanduser("~/Desktop/grass.gif"))



# Offset för riktning, alla med en delay på 20
offsets = {
    "up": (0, 20), #y-axel
    "down": (0, -20), #y-axel
    "left": (-20, 0), #x-axel
    "right": (20, 0) #x-axel
}

# Variabler för hastighet och highscore
score = 0
high_score = 0
initial_delay = delay  #Detta för att vår hastighet ökar i koden men vi vill senare att det ska återgå till normala

def reset():
    global snake, snake_dir, food_position, pen, score, delay, high_score
    snake = [[0, 0], [0, 20], [0, 40], [0, 60], [0, 80]]
    snake_dir = "up"
    food_position = get_random_food_position()
    food.goto(food_position)
    score = 0 
    update_score()  # Uppdaterar highscore på skärmen
    delay = initial_delay  # Återställer ormens ursprungliga hastighet
    move_snake()
    pen.color("darkgreen") #Färgen på "börja igen"

def move_snake():
    global snake_dir, delay #anger användningen av globala variablerna

#Skapa en ny huvudposition för ormen baserat på nuvarande riktning
    new_head = snake[-1].copy()
    new_head[0] = snake[-1][0] + offsets[snake_dir][0]
    new_head[1] = snake[-1][1] + offsets[snake_dir][1]

#Kontrollera kollisioner med ormens egen kropp eller om ormen är utanför gränsen
    if new_head in snake[:-1] or out_of_bounds(new_head):
        lost_game() #Anropa lost_game-funktionen om det finns en kollision
        return
    else:
        snake.append(new_head) #Lägg till det nya huvudet i ormen

        if not food_collision():
            snake.pop(0) #Om det inte finns någon kollision med mat, ta bort sista segmentet av ormen

        pen.clearstamps() #Rensa tidigare stämplar från skärmen

#Stämpla de uppdaterade ormsegmenten på skärmen
        for segment in snake:
            pen.goto(segment[0], segment[1])
            pen.stamp()

        screen.update() #Uppdatera skärmen för att återspegla ändringarna

        turtle.ontimer(move_snake, delay) #Schemalägg move_snake-funktionen att anropas efter en fördröjning

def out_of_bounds(pos):
    x, y = pos
    #Kontrollera om positionen är utanför spelrutfönstrets gränser
    return x < -w/2 or x > w/2 or y < -h/2 or y > h/2

def food_collision():#Denna funktion kollar om thet finns kollision mellan ormens huvud och det senaste som ormen åt, dvs mat 
    global food_position, score, delay, high_score
    if get_distance(snake[-1], food_position) < 20:#kalkylerar avståndet mellan ormens huvud och maten 
        food_position = get_random_food_position()#om avståndet är mindre än 20 så genrerar den positionen av maten 
        food.goto(food_position)
        score += 1  # ökar poäng med varje kollision med mat
        if score > high_score:
            high_score = score
        update_score()  # uppdaterar poängen
        delay -= 5  # minska delay för att öka speed
        return True
    return False

def get_random_food_position():
    x = random.randint(- w / 2 + food_size, w / 2 - food_size)#random.randit gör så att den genrerar olika platser mellan x och y koordinaterna 
    y = random.randint(- h / 2 + food_size, h / 2 - food_size)
    return (x, y)

def get_distance(pos1, pos2): #tar två argument som är tuples som reptresenterar x,y kordinaterna av två punktiker i 2D
    x1, y1 = pos1
    x2, y2 = pos2
    distance = ((y2 - y1) ** 2 + (x2 - x1) ** 2) ** 0.5 #funktion som kalkylerar avståndet 
    return distance

def update_score(): #denna funktion ansvarar för att uppdatera poängvisningen på skärmen. 
    score_display.clear() #Genom score_display kan det visa de aktuella poängen 'score' och det högsta poängen 'high score' på skärmen i mitten. 
    score_display.write("Score: {}  High Score: {}".format(score, high_score), align="center", font=("Georgia", 18, "normal")) #Detta visar nuvarande poäng och det högsta poängen på skärmen, med lämplig justering och typsnittsinställningar.

def go_up(): #denna funktion kallas när man ska styra ormen uppått. Den ändrar ormens riktning up om den åker ner. 
    global snake_dir
    if snake_dir != "down":
        snake_dir = "up"

def go_right():#denna funktion kallas när man ska styra ormen till höger. Den ändrar ormens riktning till höger om den åker till vänster.
    global snake_dir
    if snake_dir != "left":
        snake_dir = "right"

def go_down():#denna funktion kallas när man ska styra ormen nedått. Den ändrar ormens riktning ner om den åker upp. 
    global snake_dir
    if snake_dir != "up":
        snake_dir = "down"

#Denna kod definierar en funktion som används för att ändra ormens riktning till vänster när vänsterpiltangenten trycks ned.
def go_left():
    global snake_dir
    if snake_dir != "right": #kontrollerar om ormen inte redan rör sig åt höger. Om inte, fortsätter koden att ändra ormens riktning till vänster.
        snake_dir = "left"

#Denna kod definierar en funktion som används för att starta seplet om användaren trycker på mellanslagstangenten.
def start_game():
    global start_button
    start_button.clear()
    pen.clear()  # Tar bort meddelanden "Game Over" och "Press SPACE to play again".
    screen.onkey(go_up, "Up")
    screen.onkey(go_right, "Right")
    screen.onkey(go_down, "Down")
    screen.onkey(go_left, "Left")
    reset()

#funktion som körs när spelet är över(ormen har dött).
def lost_game():
    pen.clear() #Tar bort eventuella tidigare meddelanden. 
    pen.color("red") #Gör färger röd
    pen.goto(0, 50)  #Visar var det ska stå någonstans
    pen.write("Game Over!!!", align="center", font=("Georgia", 50, "normal")) #skriver gameover
    pen.goto(0, -50)
    pen.color("black")  # textfärg blir svart
    pen.write("Press SPACE to play again", align="center", font=("Georgia", 20, "normal"))
    screen.onkey(start_game, "space")  # mellanslagstangent för att köra igen

screen = turtle.Screen() #skapar en ny turtle skärm 
screen.setup(w, h)
screen.title("Snake")
screen.bgcolor("lightgreen") 
screen.tracer(0) #stänger av turtle-skärmen automatiskt

pen = turtle.Turtle("square") 
pen.penup() #Gör att det inte blir ett spår av ormen
pen.color("darkgreen")

food = turtle.Turtle()
food.shape("circle")
food.color("red")
food.shapesize(food_size / 20)
food.penup() #Flyttar maten utan att lämna märke från tidigare plats

# Create a Turtle to display the score
score_display = turtle.Turtle() #ny turtle för att visa score
score_display.penup() #lyfter pennan så inget ritas när ormen kör
score_display.color("black") #textfärg blir svart
score_display.goto(0, h/2 - 30)  # flyttar turtle till den önskade positionen för poängdisplayen på skärmen.

start_button = turtle.Turtle() #turtle för att skriva meddelande
start_button.speed(0) #turtles hastighet till 0
start_button.color("white") #färg blir vit
start_button.penup() #så att man inte ritar något när turtle kör
start_button.hideturtle() #dölj turtle
start_button.goto(0, -50) #flyttar turtle till angivna position
start_button.write("Press SPACE to Start\nUse arrow keys to play", align="center", font=("Georgia", 50, "normal"))

screen.listen()
screen.onkey(start_game, "space")

turtle.done() #avslutar turtle
