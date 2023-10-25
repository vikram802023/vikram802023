- 👋 Hi, I’m @vikram802023
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
vikram802023/vikram802023 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
DEFINT A-Z
IMPORT "gl.bi"

'Se define la función para cargar el modelo 3D
FUNCTION load_obj(filename$)
    vertices DIM SHARED
    faces DIM SHARED
    OPEN filename$ FOR INPUT AS #1
    DO WHILE NOT EOF(1)
        INPUT #1, line$
        IF LEFT$(line$, 2) = "v " THEN
            v = SPLIT(line$)
            vertices(INTEGER(v(1)), 1) = FLOAT(v(2))
            vertices(INTEGER(v(1)), 2) = FLOAT(v(3))
            vertices(INTEGER(v(1)), 3) = FLOAT(v(4))
        ELSEIF LEFT$(line$, 2) = "f " THEN
            f = SPLIT(line$)
            faces(INTEGER(f(1)), 1) = INTEGER(f(2))
            faces(INTEGER(f(1)), 2) = INTEGER(f(3))
            faces(INTEGER(f(1)), 3) = INTEGER(f(4))
        END IF
    LOOP
    CLOSE #1
    load_obj() = vertices, faces
END FUNCTION

'Se define la función para dibujar el modelo 3D
SUB draw_obj(vertices() AS SINGLE, faces() AS INTEGER)
    FOR face = 1 TO UBOUND(faces, 1)
        glBegin(GL_TRIANGLES)
        FOR vertex_index = 1 TO 3
            vertex = vertices(faces(face, vertex_index), )
            glVertex3fv(VARPTR(vertex(1)))
        NEXT
        glEnd()
    NEXT
END SUB

'Se define la clase para representar un objeto 3D
TYPE Object3D
    vertices() AS SINGLE       'Vertices del objeto
    faces() AS INTEGER         'Caras del objeto
    scale AS SINGLE            'Escala del objeto
    position() AS SINGLE       'Posición del objeto
END TYPE

SUB object3D_draw(obj AS Object3D)
    glPushMatrix()
    glTranslatef(obj.position(1), obj.position(2), obj.position(3))
    glScalef(obj.scale, obj.scale, obj.scale)
    draw_obj(obj.vertices, obj.faces)
    glPopMatrix()
END SUB

'Se define la clase para representar un auto
TYPE Car
    position() AS SINGLE       'Posición del auto
    speed AS SINGLE            'Velocidad del auto
    direction AS SINGLE        'Dirección del auto
    steering AS SINGLE         'Giro del auto
    wheel_angle AS SINGLE      'Ángulo de las ruedas
    body AS Object3D           'Cuerpo del auto
    wheels(1 TO 4) AS Object3D 'Ruedas del auto
END TYPE

SUB car_draw(car AS Car)
    glPushMatrix()
    glTranslatef(car.position(1), car.position(2), car.position(3))
    glRotatef(car.direction, 0, 0, 1)
    object3D_draw(car.body)
    FOR i = 1 TO 4
        glPushMatrix()
        IF i MOD 2 = 0 THEN
            glTranslatef(2.5, 1.5, 0)
        ELSE
            glTranslatef(2.5, -1.5, 0)
        END IF
        glRotatef(car.wheel_angle, 0, 0, 1)
        glRotatef(90, 0, 1, 0)
        object3D_draw(car.wheels(i))
        glPopMatrix()
    NEXT
    glPopMatrix()
END SUB

SUB car_accelerate(car AS Car, delta AS SINGLE)
    car.speed = car.speed + delta
    IF car.speed < 0 THEN
        car.speed = 0
    END IF
    IF car.speed > 200 THEN
        car.speed = 200
    END IF
END SUB

SUB car_steer_left(car AS Car, delta AS SINGLE)
    car.steering = car.steering - delta
    IF car.steering < -30 THEN
        car.steering = -30
    END IF
END SUB

SUB car_steer_right(car AS Car, delta AS SINGLE)
    car.steering = car.steering + delta
    IF car.steering > 30 THEN
        car.steering = 30
    END IF
END SUB

SUB car_update(car AS Car, delta AS SINGLE)
    'Mover el auto
    distance = car.speed * delta
    car.position(1) = car.position(1) + distance * COS((car.direction + car.steering) * PI / 180)
    car.position(2) = car.position(2) + distance * SIN((car.direction + car.steering) * PI / 180)

    'Rotar las ruedas
    IF car.speed <> 0 THEN
        car.wheel_angle = car.wheel_angle + car.steering * 90 / (car.speed * 0.1)
    END IF

    'Limitar el angulo de las ruedas
    IF car.wheel_angle > 30 THEN
        car.wheel_angle = 30
    END IF

    IF car.wheel_angle < -30 THEN
        car.wheel_angle = -30
    END IF
END SUB

'Se define la clase para representar una pista
TYPE Track
    vertices() AS SINGLE        'Vertices de la pista
    faces() AS INTEGER          'Caras de la pista
END TYPE

SUB track_draw(track AS Track)
    glColor3f(0.8, 0.8, 0.8)
    draw_obj(track.vertices, track.faces)
END SUB

'Funcion para calcular la distancia entre dos puntos
FUNCTION distance2D(x1 AS SINGLE, y1 AS SINGLE, x2 AS SINGLE, y2 AS SINGLE)
    distance2D = SQR((x2-x1)^2 + (y2-y1)^2)
END FUNCTION

'Se define la clase principal del juego
TYPE Game
    width AS INTEGER     'Ancho de la ventana
    height AS INTEGER    'Alto de la ventana
    car AS Car           'Objeto del auto
    track AS Track       'Objeto de la pista
    clock AS DOUBLE      'Reloj del juego
END TYPE

SUB game_run(game AS Game)
    game_over = 0
    DO WHILE NOT game_over

        'Actualizar el reloj del juego
        game.clock = TIMER

        'Actualizar el juego
        car_steer_left(game.car, 5)
        car_steer_right(game.car, 5)
        IF KEY(203) THEN
            car_steer_left(game.car, 5)
        ELSEIF KEY(205) THEN
            car_steer_right(game.car, 5)
        END IF
        car_accelerate(game.car, 1)
        IF KEY(200) THEN
            car_accelerate(game.car, 10)
        ELSEIF KEY(208) THEN
            car_accelerate(game.car, -20)
        END IF
       
        car_update(game.car, game.clock)
       
        'Dibujar el juego
        glClear(GL_COLOR_BUFFER_BIT OR GL_DEPTH_BUFFER_BIT)
        track_draw(game.track)
        car_draw(game.car)
        _DISPLAY

        'Revisar si se ha detectado el final del juego
        game_over = KEY(ESC)
    LOOP
END SUB

'Se crea y ejecuta el juego
DIM game AS Game
game.width = 800
game.height = 600
OPENGRAPHICS
gluPerspective(45,game.width/game.height,0.1,1000)
glTranslatef(0,-10,-50)
game.clock = TIMER
game.car.position(1) = 0
game.car.position(2) = 0
game.car.position(3) = 0
game.car.speed = 0
game.car.steering = 0
game.car.direction = 0
game.track.vertices(), game.track.faces() = load_obj("track.obj")
game.car.body.vertices(), game.car.body.faces() = load_obj("car_body.obj")
FOR i = 1 TO 4
    game.car.wheels(i).vertices(), game.car.wheels(i).faces() = load_obj("car_wheel.obj")
NEXT
game_run(game)
``` 

Este código utiliza las funciones y estructuras de datos compatibles con QB64 y utiliza las bibliotecas de gráficos incorporadas en QB64. Para ajustar el tamaño y la posición de los objetos de pantalla, la biblioteca GL (OpenGL) también se utiliza para proporcionar una perspectiva 3D. 

Ten en cuenta que los valores y descripciones escritos en los comentarios a lo largo del código original también deben ser asimilados para adaptarse a QB64.
