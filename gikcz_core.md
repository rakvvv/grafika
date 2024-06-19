```
#include <glad/glad.h>
#include <GLFW/glfw3.h>

#include <iostream>

void framebuffer_size_callback(GLFWwindow* window, int width, int height);
void processInput(GLFWwindow *window);

// settings
const unsigned int SCR_WIDTH = 800;
const unsigned int SCR_HEIGHT = 600;

const char *vertexShaderSource = "#version 330 core\n"
    "layout (location = 0) in vec3 aPos;\n"
    "void main()\n"
    "{\n"
    "   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
    "}\0";
const char *fragmentShaderSource = "#version 330 core\n"
    "out vec4 FragColor;\n"
    "void main()\n"
    "{\n"
    "   FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);\n"
    "}\n\0";

int main()
{
    // glfw: initialize and configure
    // ------------------------------
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

#ifdef __APPLE__
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
#endif

    // glfw window creation
    // --------------------
    GLFWwindow* window = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT, "LearnOpenGL", NULL, NULL);
    if (window == NULL)
    {
        std::cout << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    glfwMakeContextCurrent(window);
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

    // glad: load all OpenGL function pointers
    // ---------------------------------------
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
    {
        std::cout << "Failed to initialize GLAD" << std::endl;
        return -1;
    }


    // build and compile our shader program
    // ------------------------------------
    // vertex shader
    unsigned int vertexShader = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
    glCompileShader(vertexShader);
    // check for shader compile errors
    int success;
    char infoLog[512];
    glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
    if (!success)
    {
        glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
    }
    // fragment shader
    unsigned int fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
    glCompileShader(fragmentShader);
    // check for shader compile errors
    glGetShaderiv(fragmentShader, GL_COMPILE_STATUS, &success);
    if (!success)
    {
        glGetShaderInfoLog(fragmentShader, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::FRAGMENT::COMPILATION_FAILED\n" << infoLog << std::endl;
    }
    // link shaders
    unsigned int shaderProgram = glCreateProgram();
    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    glLinkProgram(shaderProgram);
    // check for linking errors
    glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
    if (!success) {
        glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << infoLog << std::endl;
    }
    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);

    // set up vertex data (and buffer(s)) and configure vertex attributes
    // ------------------------------------------------------------------
    // add a new set of vertices to form a second triangle (a total of 6 vertices); the vertex attribute configuration remains the same (still one 3-float position vector per vertex)
    float vertices[] = {
        // first triangle
        -0.9f, -0.5f, 0.0f,  // left 
        -0.0f, -0.5f, 0.0f,  // right
        -0.45f, 0.5f, 0.0f,  // top 
        // second triangle
         0.0f, -0.5f, 0.0f,  // left
         0.9f, -0.5f, 0.0f,  // right
         0.45f, 0.5f, 0.0f   // top 
    }; 

    unsigned int VBO, VAO;
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);
    // bind the Vertex Array Object first, then bind and set vertex buffer(s), and then configure vertex attributes(s).
    glBindVertexArray(VAO);

    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);

    // note that this is allowed, the call to glVertexAttribPointer registered VBO as the vertex attribute's bound vertex buffer object so afterwards we can safely unbind
    glBindBuffer(GL_ARRAY_BUFFER, 0); 

    // You can unbind the VAO afterwards so other VAO calls won't accidentally modify this VAO, but this rarely happens. Modifying other
    // VAOs requires a call to glBindVertexArray anyways so we generally don't unbind VAOs (nor VBOs) when it's not directly necessary.
    glBindVertexArray(0); 


    // uncomment this call to draw in wireframe polygons.
    //glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);

    // render loop
    // -----------
    while (!glfwWindowShouldClose(window))
    {
        // input
        // -----
        processInput(window);

        // render
        // ------
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // draw our first triangle
        glUseProgram(shaderProgram);
        glBindVertexArray(VAO); // seeing as we only have a single VAO there's no need to bind it every time, but we'll do so to keep things a bit more organized
        glDrawArrays(GL_TRIANGLES, 0, 6); // set the count to 6 since we're drawing 6 vertices now (2 triangles); not 3!
        // glBindVertexArray(0); // no need to unbind it every time 
 
        // glfw: swap buffers and poll IO events (keys pressed/released, mouse moved etc.)
        // -------------------------------------------------------------------------------
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    // optional: de-allocate all resources once they've outlived their purpose:
    // ------------------------------------------------------------------------
    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
    glDeleteProgram(shaderProgram);

    // glfw: terminate, clearing all previously allocated GLFW resources.
    // ------------------------------------------------------------------
    glfwTerminate();
    return 0;
}

// process all input: query GLFW whether relevant keys are pressed/released this frame and react accordingly
// ---------------------------------------------------------------------------------------------------------
void processInput(GLFWwindow *window)
{
    if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);
}

// glfw: whenever the window size changed (by OS or user resize) this callback function executes
// ---------------------------------------------------------------------------------------------
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
    // make sure the viewport matches the new window dimensions; note that width and 
    // height will be significantly larger than specified on retina displays.
    glViewport(0, 0, width, height);
}
```
kod do wyswitlania dwoch trojkatow

```
const char* vertexShaderSource = R"(
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;

out vec3 ourColor;

void main()
{
    gl_Position = vec4(aPos.x, -aPos.y, aPos.z, 1.0); // odwracamy współrzędną y
    ourColor = aColor;
}
)";
```
do odwracania


```
#include <glad/glad.h>
#define GLFW_INCLUDE_NONE
#include <GLFW/glfw3.h>
#include <openglDebug.h>
#include <demoShaderLoader.h>
#include <iostream>

#define USE_GPU_ENGINE 0
extern "C"
{
	__declspec(dllexport) unsigned long NvOptimusEnablement = USE_GPU_ENGINE;
	__declspec(dllexport) int AmdPowerXpressRequestHighPerformance = USE_GPU_ENGINE;
}

static void key_callback(GLFWwindow* window, int key, int scancode, int action, int mods)
{
	if (key == GLFW_KEY_ESCAPE && action == GLFW_PRESS)
		glfwSetWindowShouldClose(window, GLFW_TRUE);
}

const char* vertexShaderSource = R"(
#version 330 core
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos, 1.0);
}
)";

const char* fragmentShaderSource = R"(
#version 330 core
out vec4 FragColor;

void main()
{
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f); // Orange color
}
)";

int main(void)
{
	if (!glfwInit())
		return -1;

#pragma region report opengl errors to std
	//enable opengl debugging output.
	glfwWindowHint(GLFW_OPENGL_DEBUG_CONTEXT, true);
#pragma endregion

	//glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
	//glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 5);
	//glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE); //you might want to do this when testing the game for shipping

	GLFWwindow* window = glfwCreateWindow(640, 480, "Simple example", NULL, NULL);
	if (!window)
	{
		glfwTerminate();
		return -1;
	}

	glfwSetKeyCallback(window, key_callback);

	glfwMakeContextCurrent(window);
	gladLoadGLLoader((GLADloadproc)glfwGetProcAddress);
	glfwSwapInterval(1);

#pragma region report opengl errors to std
	glEnable(GL_DEBUG_OUTPUT);
	glEnable(GL_DEBUG_OUTPUT_SYNCHRONOUS);
	glDebugMessageCallback(glDebugOutput, 0);
	glDebugMessageControl(GL_DONT_CARE, GL_DONT_CARE, GL_DONT_CARE, 0, nullptr, GL_TRUE);
#pragma endregion

	// Define the vertex data
	float vertices[] = {
		-0.5f, -0.5f, 0.0f,
		 0.5f, -0.5f, 0.0f,
		 0.0f,  0.5f, 0.0f
	};

	// Create and compile the vertex shader
	unsigned int vertexShader = glCreateShader(GL_VERTEX_SHADER);
	glShaderSource(vertexShader, 1, &vertexShaderSource, nullptr);
	glCompileShader(vertexShader);

	// Check for vertex shader compilation errors
	int success;
	char infoLog[512];
	glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
	if (!success) {
		glGetShaderInfoLog(vertexShader, 512, nullptr, infoLog);
		std::cerr << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
	}

	// Create and compile the fragment shader
	unsigned int fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
	glShaderSource(fragmentShader, 1, &fragmentShaderSource, nullptr);
	glCompileShader(fragmentShader);

	// Check for fragment shader compilation errors
	glGetShaderiv(fragmentShader, GL_COMPILE_STATUS, &success);
	if (!success) {
		glGetShaderInfoLog(fragmentShader, 512, nullptr, infoLog);
		std::cerr << "ERROR::SHADER::FRAGMENT::COMPILATION_FAILED\n" << infoLog << std::endl;
	}

	// Link the vertex and fragment shader into a shader program
	unsigned int shaderProgram = glCreateProgram();
	glAttachShader(shaderProgram, vertexShader);
	glAttachShader(shaderProgram, fragmentShader);
	glLinkProgram(shaderProgram);

	// Check for linking errors
	glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
	if (!success) {
		glGetProgramInfoLog(shaderProgram, 512, nullptr, infoLog);
		std::cerr << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << infoLog << std::endl;
	}

	// Delete the shaders as they're linked into our program now and no longer necessary
	glDeleteShader(vertexShader);
	glDeleteShader(fragmentShader);

	// Create a vertex array object (VAO)
	unsigned int VAO;
	glGenVertexArrays(1, &VAO);

	// Create a vertex buffer object (VBO)
	unsigned int VBO;
	glGenBuffers(1, &VBO);

	// Bind the VAO
	glBindVertexArray(VAO);

	// Bind the VBO
	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

	// Configure the vertex attribute pointers
	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
	glEnableVertexAttribArray(0);

	// Unbind the VBO (not the VAO)
	glBindBuffer(GL_ARRAY_BUFFER, 0);

	// Render loop
	while (!glfwWindowShouldClose(window)) {
		int width, height;
		glfwGetFramebufferSize(window, &width, &height);
		glViewport(0, 0, width, height);
		glClear(GL_COLOR_BUFFER_BIT);

		// Use the shader program
		glUseProgram(shaderProgram);

		// Bind the VAO
		glBindVertexArray(VAO);

		// Draw the triangle
		glDrawArrays(GL_TRIANGLES, 0, 3);

		// Swap the buffers
		glfwSwapBuffers(window);
		glfwPollEvents();
	}

	// Clean up
	glDeleteVertexArrays(1, &VAO);
	glDeleteBuffers(1, &VBO);
	glDeleteProgram(shaderProgram);

	// Terminate GLFW
	glfwTerminate();
	return 0;
}
```
```
#version 330 core
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos, 1.0);
}
fragment.frag:

```
```
#version 330 core
out vec4 FragColor;

void main()
{
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f); // Orange color
}
```
https://github.com/meemknight/openglConfiguration/tree/master


# Kolokwia GiKCzK - Pytania

Pierdolony śmieć farciarski skurwiel zajebany.

## Kolokwium - Lab1

1. Napisz instrukcje warunkową w pseudokodzie, czy punkt jest wewnątrz trójkąta.

```
if(p.x > 0 && p.x < 1 && p.y > 0 && p.y < 1 && p.z > 0 && p.z < 1) {
  // draw
}
```

2. Czy światło padające na ścianę obiektu będzie miało te same parametry w cieniowaniu płaskim?

```
TAK
```

> cieniowanie płaskie - jedno obliczenie na jedną ścianę
>
> cieniowanie Gourarda - obliczenia na każdy wierzchołek 
>
> cieniowanie Phonga - obliczenia wymagane są dla każdego piksela obrazu

3. Czy narysuje sie linia bresenhamem:

![](https://cdn.discordapp.com/attachments/983759870429917225/987435619586297886/unknown.png)

```
NIE
```

4. Napisz pseudokod do barycentric

```java
  public Vec3f barycentric(Vec3f A, Vec3f B, Vec3f C, Vec3f P) {
      // wektor składający się ze współrzędnych x wektorów AB, AC, PA
      Vec3f v1 = new Vec3f(B.x - A.x, C.x - A.x, A.x - P.x);

      // wektor składający się ze współrzędnych y wektorów AB, AC, PA
      Vec3f v2 = new Vec3f(B.y - A.y, C.y - A.y, A.y - P.y);

      // iloczyn skalarny v1 i v2
      Vec3f cross = this.scalarDot(v1, v2);

      // wektor postaci: cross.x / cross.z, cross.y / cross.z
      Vec2f uv = new Vec2f(cross.x / cross.z, cross.y / cross.z);

      // współrzędne barycentryczne, uv.x, uv.y, 1- uv.x - uv.y
      Vec3f barycentric = new Vec3f(uv.x, uv.y, 1 - uv.x - uv.y);

      return barycentric;
  }
```

```
  barycentric(A, B, C, P) {
    BACA pod AP zrobił dwa wektory, jeden z samych x, drugi tylko z y
    potem go ukrzyżowali
    padało na niego promieniowanie xZYZz'a
    zwrócono koordy z krzyża (x, y, 1 - x - y)
  }
```

5. Macierze, to samo co zadanie 5 w LAB3

6. Jeżeli wartość w z-buforze o danych współrzędnych (x,y) jest ... niż przetwarzanego piksela, to framebuffer nie ejst aktualizowany.

```
MNIEJSZA
```

> https://pl.wikipedia.org/wiki/Bufor_Z

7. Z czego składa się kamera?

```
e – punkt położenia
g – wektor kierunku patrzenia
t – wektor view-up
```

## Kolokwium - Lab 2

2. Rodzaje shaderów w OpenGL.

https://developer.mozilla.org/en-US/docs/Games/Techniques/3D_on_the_web/GLSL_Shaders

```
Vertex Shader - Cieniowanie wierzchołkowe - dla poszczególnych wierzchołków
Fragment Shader - Cienowanie pikseli - wyliczanie kolorów pikseli (RGBA)

---

Vertex Shaders:     GL_VERTEX_SHADER
Evaluation Shaders: GL_TESS_CONTROL_SHADER and GL_TESS_EVALUATION_SHADER
Geometry Shaders:   GL_GEOMETRY_SHADER
Fragment Shaders:   GL_FRAGMENT_SHADER
Compute Shaders:    GL_COMPUTE_SHADER
```

1. Czy program narysuje linie Bresenhama?
2. Offset i Stride na floatach i wierzchołkach.
3. Mnożenie macierzy 3D, aby powstało przekształcenie na ekranie.
4. Pseudokod drawTriangle.
5. Czy światło padające na ścianę obiektu będzie miało te same parametry w cieniowaniu płaskim (Tak / Nie).

```
było w lab1/lab3
```

## Kolokwium - Lab 3

1. Jaki warunek muszą spełniać współrzędne barycentryczner punktu P (wyliczone względem trójkąta ABC; współrzędne to (u, v, 1 - u - v)), aby punkt P leżał na zewnątrz trójkąta ABC? Napisz w pseudokodzie warunek if:

```
if(p.x < 0 && p.x > 1 && p.y < 0 && p.y > 1 && p.z < 0 && p.z > 1) {
  // draw
}
```

2. Czy następujące zdanie jest prawdziwe?

> Zmienna typu uniform jest unikalna dla każdego programu shadera i może być dostępna z dowolnego shadera na dowolny etapie potoku graficznego.

```
TAK
```

> Przede wszystkim uniformy są globalne. Globalne oznacza, że zmienna uniform jest unikatowa dla każdego Program Object i można uzyskać do niej dostęp z dowolnego programu cieniującego. Po drugie, niezależnie od tego na jaką wartość ustawisz uniform, to zachowa tę wartości, dopóki nie zostanie ona zresetowana lub zaktualizowana.
> https://shot511.github.io/2018-09-07-zaawansowany-glsl/#obiekt-bufora-uniform%C3%B3w-ang-uniform-buffer-objects > https://shot511.github.io/2017-09-04-programy-cieniujace/

3. Czy poniższym algorytmem mozna narysować poprawnie linię przedstawioną na rysunku (pixel x0, y0 to pierwszy pixel z lewej strony)?

![](https://cdn.discordapp.com/attachments/973584069776519189/987403509362204702/unknown.png)

```
 public void drawLineBresenhamDEBIL(int x0, int y0, int x1, int y1){
    int white = 255 | (255 << 8) | (255 << 16) | (255 << 24);

    float err = 0.0f;
    float d = (float)(y1 - y0) / (x1 - x0);
    int y = y0;
    for(int x = x1; x < x0; x++){
        render.setRGB(x,y,white);
        err += d;
        if(err > 0.5){
            y += 1;
            err -= 1.0f;
        }
    }
}
```

```
NIE

Narysuje jakieś gówno na kształt wykresu funkcji logarytmicznej,
ale idealnie poziomej nie narysuje, ledwo 1 oktet pokryje, bez dołu
```

![co to za gówno](https://cdn.discordapp.com/attachments/973584069776519189/987404871919620096/unknown.png)

4. Napisz w pseudokodzie metodę dokonująca rasteryzacji trójkąta o współrzędnych ABC, zakładając, że mamy do dyspozycji metodę `barycentric(Point A, Point B, Point C, Point P)`, wyliczającą współrzędne barycentryczne. Należy uwzględnić Z-Buffer.

```java
public void drawTriangle(Vec3i A, Vec3i B, Vec3i C, Vec3i color) {
    Integer[] xs = {A.x, B.x, C.x};
    Integer[] ys = {A.y, B.y, C.y};
    int minx = Collections.min(Arrays.asList(xs));
    int maxx =  Collections.max(Arrays.asList(xs));
    int miny = Collections.min(Arrays.asList(ys));
    int maxy =  Collections.max(Arrays.asList(ys));

    for (int i = minx; i <= maxx; i++) {
        for (int j = miny; j <= maxy; j++) {
            Vec2i P = new Vec2i(i,j);
            Vec3f Bar = barycentric3i(A, B, C, P);

            float  z =  A.z*(1 - Bar.z) + B.z*Bar.z ;

            if ((Bar.x > 0 && Bar.x < 1 && Bar.y < 1 && Bar.y > 0 && Bar.z > 0 && Bar.z < 1)) {
                if (ZBuffer[i][j] >= z){
                    drawPoint(i, j, color);
                    ZBuffer[i][j] = z;
                }

            }
        }
    }
}
```

```
void drawTriangle(A, B, C, Color) {
  // W pętli iterujemy po punktach na ekranie
  // tych co tworzą zewnętrzne ramy trójkąta (aby oszczędzić obliczeń)

    // Tworzymy wektor P, obecny punkt (x, y) z pętli

    // Tworzymy wektor Bar, czyli współrzędne barycentryczne

    // Obliczamy wartość `z` ze wzoru:
    // A.z * (1 - Bar.z) + B.z * (Bar.z)

    // Sprawdzamy współrzędne barycentryczne
    // czy można narysować punkt (czy jest wewnątrz trójkąta)

      // Sprawdzamy czy wartość `z` jest <= od obecnej w z-bufferze

        // Jeżeli tak, to rysujemy punkt i aktualizujemy
        // zawartość zbuffera
}
```

5. Podaj sekwencję mnożenia macierzy, aby uzyskać macierz dokonująca transformacji współrzędnych **wirtualnego świata 3D** do współrzędnych ekranu:

`M = ...............`

Macierze oznaczaj dużą literą M, z odpowiednim indeksem dolnym

```
M = Mvp Mortho Mcam
```

? Slajd z transformacji projekcji (ang. projection transformation)
![](https://cdn.discordapp.com/attachments/983759870429917225/987424501509816340/unknown.png)

? Slajd z transformacji perspektywicznej (ang. viewport transformation)![](https://media.discordapp.net/attachments/646373314876538893/988350277797167104/unknown.png)

6. Punkt P ma następujące współrzędne barycentryczne względem pewnego trójkąta ABC: 0.62, 0.59, -0.22. Punkt P leży (zaznacz właściwą odpowiedź):

```
poza trójkątem
```

7. Zakładając, że dane w buforze są typu FLOAT, i zorganizowane są jak na rysunku podaj w bajtach offset i stride poszczególnych atrybutów wierzchołka:

```
1. stride: 32      offset: 0   (odległość do XYZ)
2. stride: 32      offset: 12  (odległość do ST)
3. stride: 32      offset: 20  (odległość do RGB)
```

```
stride is just sizeof SVertex
stride to do nastepnego zestawu atrybutów w następnym wierzchołku
offset to po prostu odegłość w bajtach od 0
FLOAT zajmuje 4 bajty, czyli jedna komórka tutaj na rysunku 4 bajty wpierdala
```

> https://stackoverflow.com/questions/22296510/what-does-stride-mean-in-opengles

![](https://media.discordapp.net/attachments/973584069776519189/987416881826783262/unknown.png)
![](https://media.discordapp.net/attachments/973584069776519189/987416844451328020/unknown.png)

## Kolokwium - lab4

To co lab3 X D

# Egzamin - Pytania

8-bitowy jednokanałowy obraz cyfrowy o rozmiarach HxW można zdefiniować jako funkcję:

```
F(x nal do (0,W)] x [y nal do (0,H)] -> [0, 255]
```

---

Dokończ zdanie: Algorytmy rasteryzacji transformują ...

```
Zamieniają geometrie 2D na fragmenty
```

---

Jeżeli wartość w z-buforze o danych współrzędnych (x,y) jest ... niż przetwarzanego piksela, to framebuffer nie jest aktualizowany.

```
mniejsza
```

---

Co jest prawdą o renderowaniu przez rasteryzację obiekt po obiekcie?

```
Działa jako potok przetwarzania
Działa na GPU
Idealne do zastosowań real-time
```

---

Podaj przykład przestrzeni kolorów (inny niż RGB) wykorzystywany często w urządzeniach druku kolorowego:

```
CMYK
```

---

Uzupełnij: Najbardziej efektywnym algorytmem eliminacji powierzchni zasłoniętych jest:

```
test z-bufora
```

---

Większość urządzeń do prezentacji grafiki to urządzenia:

```
rastrowe
```

---

Najczęściej wykorzystywanym modelem kamery w grafice komputerowej jest:

```
kamera otworkowa
```

---

Translacja 2D może być reprezentowana mnożeniem macierzy o wymiarach ... x ...

```
3x3
```

![](https://cdn.discordapp.com/attachments/646373314876538893/987452184310480947/unknown.png)

---

Wynikiem jakiej transformacji jest geometria we współrzędnych ekranu?

```
Viewport transform
```

---

Jakie współrzędne są rezultatem transformacji viewport?

```
współrzędne ekranu
```

---

W transformacji perspektywicznej rozmiar obiektu na obrazie jest ...

```
Zależny od głębokości (odległości od kamery)
```

---

Wymień programowalne etapy potoku graficznego:

```
- Etap przetwarzania geometrii (2)
- Etap przetwarzania pikesli (4)
```

---

Czy powyższym algorytmem (Bresenhama od typa) można narysować poprawnie następującą linię (pixel p0) zaznaczony na niebiesko?

![](https://cdn.discordapp.com/attachments/646373314876538893/987448188577406986/unknown.png)

```
nie
```

---

W cieniowaniu płaskim każdy piksel danej ściany ma indywidualnie wyliczane parametry oświetlenia.

```
nie
```

---

Macierz transformacji trójwymiarowego wektora [4, 2, 2] do wektora [8, 6, 1] będzie wyglądać następująco:

|     |     |     |
| --- | --- | --- |
| a   | b   | c   |
| d   | e   | f   |
| g   | h   | i   |

Proszę podać współczynniki w kolejności a, b, c, d, e ...
np. 1, 1, 1, 1, 1, 1, 1, 0, 0.25

![](https://media.discordapp.net/attachments/646373314876538893/987447893466185798/unknown.png)

---

Model opisujący coś, co ma pojawić się na obrazie to:

```
model geometryczny
```

---

Uzupełnij: ... i ... pozwalają GPU uzyskać wysoką wydajność.

```
potokowość i zrównoleglenie
```

---

Jeśli punkt p ma jedną współrzędną barycentryczną równą 0.1, a pozostałe mają wartość w przedziale (0, 1) to leży on ...

```
wewnątrz trójkąta
```

---

Podaj, co jest wejściem, a co wyjściem transformacji kamery

```
- Wejście
  - geometria we współrzędnych świata (sceny) (Obiekty o współrzędnych sceny)
- Wyjście
  - geometria we współrzędnych kamery (scena we współrzędnych kamery)
```

---

Dokończ zdanie: Algorytmy rasteryzacji transformują ...

```
zamieniają geometrie 2D na fragmenty
```

---

![](https://cdn.discordapp.com/attachments/983759870429917225/987442102327738368/unknown.png)

Uzupełnij zawartość instrukcji warunkowej tak, aby sprawdzała ona czy współrzędne barycentryczne bc są w przedziale (0, 1).

bc zawiera trzy pola: x, y, z.

```
bc.x > 0 && bc.x < 1 && bc.y > 0 && bc.y < 1 && bc.z > 0 && bc.z < 1
```

---

Opisz krótko, czym jest kanoniczna bryła widzenia i podaj jaki typ bryły występuje dla projekcji perspektywicznej.

```
jest to sześcian 2x2x2 ze środkiem w początku układu, dla projekcji perspektywicznej typ bryły to ostrosłup ścięty
```

---

Opisz czym jest fragment i na jakim etapie potoku graficznego powstaje.

```
Fragment to kandydat na bycie pikselem, jest to dyskretna współrzędna ekranu, która potem stanie się pikselem. Wiele fragmentów może zajmować tą samą współrzędną. Powstaje w procesie rasteryzacji z rzeczywistych współrzędnych wielokąta.
```

---

Jaką wartością powinna być zainicjalizowana komórka z-buffora?

```
maksymalna wartość `z` z danej sceny. `z` to głębokość.
```

---

Wymień 3 cechy rzutowania ortograficznego:

```
zachowuje równoległość linii
dystans od kamery nie wpływa na wielkość obiektu
nie odzwierciedla sposobu w jaki ludzkie oko postrzega obraz
```

---

Zazwyczaj jest ... zbiór pikseli możliwie wiernie przybliżających linię pomiędzy punktami p0 i p1 na wyświetlaczu rastrowym.

```
więcej niż jeden
```

---

![](https://media.discordapp.net/attachments/983759870429917225/987443264892305428/unknown.png)

Która wersja 4 linijki jest poprawna dla całkowitoliczbowego algorytmu rysowania linii Bresenhama?

```
int d = 2 * (y1-y0)
```

---

Operacja pozwalająca przedstawić obiekty trójwymiarowego świata na płaszczyźnie to:

```
rzutowanie
```

---

Podaj 3 sposoby reprezentacji rotacji:

```
Macierzowe jednorodne
Kąty Eulera
Kwaterniony
```

---

Kamera umieszczona jest ... wirtualną płaszczyzną obrazu.

```
za
```

---

Podaj przykład macierzy dokonującej transformacji pochylania Y-shearing w 2D operującej na współrzędnych jednorodnych:

|     |     |     |
| --- | --- | --- |
| a   | b   | c   |
| d   | e   | f   |
| g   | h   | i   |

Proszę podać współczynniki w kolejności a, b, c, d, e ...
np. 1, 1, 1, 1, 1, 1, 1, 0, 0.25

```
1,0,0,0.5,1,0,0,0,1
```

![](https://cdn.discordapp.com/attachments/646373314876538893/987447260046561380/unknown.png)

---

Podaj przykład macierzy dokonującej transformacji pochylania X-shearing w 2D operującej na współrzędnych jednorodnych:

|     |     |     |
| --- | --- | --- |
| a   | b   | c   |
| d   | e   | f   |
| g   | h   | i   |

Proszę podać współczynniki w kolejności a, b, c, d, e ...
np. 1, 1, 1, 1, 1, 1, 1, 0, 0.25

![](https://cdn.discordapp.com/attachments/646373314876538893/987449175492927498/unknown.png)

---

Translacja jest przykładem transformacji ...

```
nieliniowej
```

---

Kamera jest reprezentowana jako (uzupełnij definicje):

```
e – położenie
g – wektor kierunku patrzenia
t – wektor view-up
```

---

Jakie współrzędne są rezultatem transformacji kamery?

```
Kanoniczna bryła widzenia, geometria we współrzędnych kamery
```

---

Podaj dwa przykłady transformacji liniowych:

```
skalowanie, odbicie
```

---

Podaj ogólny algorytm ray-castingu:

```
1. Wprowadzenie promienia kamery w kierunku rzutni
2. Znalezienie punktu który przecina się jako pierwszy z obiektem ze sceny
3. Ustalenie jasności punktu
```

---

Wymień 3 własności obrazu i ich przykładowe wartości:

```
Liczba kanałów: 1 (czarno-biały), 3 (kolorowy RGB), 4 (kolorowy RGB z przezroczystością)
Głębia kolorów: 16 bitów na piksel
Rozdzielczość, np. 1920x1080px (FHD)
```

---

Translacja 3D może być reprezentowana mnożeniem macierzy o wymiarach

```
4x4
```

---

# Kodziki

### barycentric()

```java
public Vec3f barycentric(Vec3f A, Vec3f B, Vec3f C, Vec3f P) {
    // wektor składający się ze współrzędnych x wektorów AB, AC, PA
    Vec3f v1 = new Vec3f(B.x - A.x, C.x - A.x, A.x - P.x);

    // wektor składający się ze współrzędnych y wektorów AB, AC, PA
    Vec3f v2 = new Vec3f(B.y - A.y, C.y - A.y, A.y - P.y);

    // iloczyn skalarny v1 i v2
    Vec3f cross = this.scalarDot(v1, v2);

    // wektor postaci: cross.x / cross.z, cross.y / cross.z
    Vec2f uv = new Vec2f(cross.x / cross.z, cross.y / cross.z);

    // współrzędne barycentryczne, uv.x, uv.y, 1- uv.x - uv.y
    Vec3f barycentric = new Vec3f(uv.x, uv.y, 1 - uv.x - uv.y);

    return barycentric;
}
```

```java
public Vec3f barycentric(Vec3f A, Vec3f B, Vec3f C, Vec3f P) {
    // wektor składający się ze współrzędnych x wektorów AB, AC, PA

    // wektor składający się ze współrzędnych y wektorów AB, AC, PA

    // iloczyn skalarny v1 i v2

    // wektor postaci: cross.x / cross.z, cross.y / cross.z

    // współrzędne barycentryczne, uv.x, uv.y, 1- uv.x - uv.y
}
```

### drawTriangle()

```java
public void drawTriangle(Vec3i A, Vec3i B, Vec3i C, Vec3i color) {
    Integer[] xs = {A.x, B.x, C.x};
    Integer[] ys = {A.y, B.y, C.y};
    int minx = Collections.min(Arrays.asList(xs));
    int maxx =  Collections.max(Arrays.asList(xs));
    int miny = Collections.min(Arrays.asList(ys));
    int maxy =  Collections.max(Arrays.asList(ys));

    for (int i = minx; i <= maxx; i++) {
        for (int j = miny; j <= maxy; j++) {
            Vec2i P = new Vec2i(i,j);
            Vec3f Bar = barycentric3i(A, B, C, P);

            float  z =  A.z * (1 - Bar.z) + B.z * Bar.z ;

            if ((Bar.x > 0 && Bar.x < 1 && Bar.y < 1 && Bar.y > 0 && Bar.z > 0 && Bar.z < 1)) {
                if (ZBuffer[i][j] >= z){
                    drawPoint(i, j, color);
                    ZBuffer[i][j] = z;
                }

            }
        }
    }
}
```

```
void drawTriangle(A, B, C, Color) {
  // W pętli iterujemy po punktach na ekranie
  // tych co tworzą zewnętrzne ramy trójkąta (aby oszczędzić obliczeń)

    // Tworzymy wektor P, obecny punkt (x, y) z pętli

    // Tworzymy wektor Bar, czyli współrzędne barycentryczne

    // Obliczamy wartość `z` ze wzoru:
    // A.z * (1 - Bar.z) + B.z * (Bar.z)

    // Sprawdzamy współrzędne barycentryczne
    // czy można narysować punkt (czy jest wewnątrz trójkąta)

      // Sprawdzamy czy wartość `z` jest >= od obecnej w z-bufferze

        // Jeżeli tak, to rysujemy punkt i aktualizujemy
        // zawartość zbuffera
}
```

### drawLine()

```java
public void drawLineNaive(int x0, int y0, int x1, int y1) {
    float dx = x1 - x0;
    float dy = y1 - y0;
    float m = dy / dx;

    float det = x0 * y1 - x1 * y0;
    float c = (det / dx) * (-1);
    for (int i = x0; i <= x1; i++) {
        float y = m * i + c;
        drawPoint(i, Math.round(y));
    }
}
```


### drawLineBresenham()

```java
 public void drawLineBresenhamDEBIL(int x0, int y0, int x1, int y1){
     int white = 255 | (255 << 8) | (255 << 16) | (255 << 24);

     float err = 0.0f;
     float d = (float)(y1 - y0) / (x1 - x0);
     int y = y0;
     for(int x = x1; x < x0; x++){
         render.setRGB(x,y,white);
         err += d;
         if(err > 0.5){
             y += 1;
             err -= 1.0f;
         }
     }
 }
```

### współrzędne barycentryczne

Punkt w środku:

```java
if (p.x > 0 && p.x < 1 && p.y > 0 && p.y < 1 p.z > 0 && p.z < 1) {
  // draw
}
```

Punkt na zewnątrz:

```java
if (p.x < 0 && p.x > 1 && p.y < 0 && p.y > 1 p.z < 0 && p.z > 1) {
  // draw
}
```

Punkt na krawędzi:

```java
if ((p.x == 0 && p.y > 0 && p.y < 1 p.z > 0 && p.z < 1)
  ||(p.x > 0 && p.x < 1 && p.y == 0 p.z > 0 && p.z < 1)
  ||(p.x > 0 && p.x < 1 && p.y > 0 && p.y < 1 p.z == 0)
  ) {
  // draw
}
```

Punkt na wierzchołku:

```java
if ((p.x == 1 && p.y == 0 && p.z == 0)
  ||(p.x == 0 && p.y == 1 && p.z == 0)
  ||(p.x == 0 && p.y == 0 && p.z == 1)
  ) {
  // draw
}
```

# Notatki

Etapy potoku graficznego:

- **Etap aplikacji**:
  - Zadania: realizowana na CPU, część pracy możliwa do wykonania przez GPU, realizowane zadania to np. detekcja kolizji
  - Programowalna: Nie
- **Etap przetwarzania geometrii**:
  - Zadania: transformacja geometrii z 3D na 2D
  - Programowalna: Tak
- **Etap rasteryzacji**:
  - Zadania: zamiana geometrii 2D na piksele
  - Programowalna: Nie
- **Etap przetewrzania pikseli**:
  - Zadania: cieniowanie pikseli, łączenie pikseli
  - Programowalna: Tak



Rodzaje rzutowań:

- Rzutowanie równoległe prostokątne (ortograficzne):
  - zachowuje równoległość linii
  - pozwala realistycznie przedstawić wymiary obiektu
- Rzutowanie perspektywiczne:
  - obiekty znajdujące się dalej wydają się mniejsze
  - pozwala uzyskać naturalnie wyglądające dla człowieka obrazy



Rodzaje transformacji:

![](https://cdn.discordapp.com/attachments/973584069776519189/989257409333714984/unknown.png)

- **Modeling** transformation:

  - *Dane wejściowe*: współrzędne modelu (w jego własnym układzie)
  - *Cel*: umieścić model na wirtualnej scenie 3D

- **Camera** transformation:

  - *Dane wejściowe*: obiekty o współrzędnych sceny (world coordinates)

  - *Cel*: uzyskać scenę we współrzędnych kamery

  - *Postać macierzowa*:

    ![image-20220622125113732](https://media.discordapp.net/attachments/646373314876538893/989121055492829214/unknown.png)

- **Projection** transformation:

  - *Dane wejściowe*: obiekty w układzie kamery

  - *Cel*: rzutowanie obiektów na płaszczyznę

  - *Postać macierzowa*:

    ![image-20220622125238594](https://media.discordapp.net/attachments/646373314876538893/989121076095238164/unknown.png)

- **Viewport** transformation:

  - *Dane wejściowe*: sześcian 2x2x2, środek w początku układu

  - *Cel*: uzyskać współrzędne ekranu

  - *Postać macierzowa*:

    ![image-20220622124839412](https://media.discordapp.net/attachments/646373314876538893/989121093845528616/unknown.png)

  
