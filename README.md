# CGLaba2

Глобальные переменные 
```c++
 GLuint gWorldLocation;
 glm::mat4 worldMatrix;
```

# Шейдер
 
Код шейдера
```c++
static char testVertShader[] = "                                                          
#version 330                                                                       
                                                                                    
layout (location = 0) in vec3 Position;                                             
                     
uniform mat4 gWorld; 
void main(){                                         
    gl_Position = gWorld * vec4(Position, 1.0);       
}";

static char testFragShader[] = "                            
#version 330                                             
                                                       
out vec4 FragColor;          
void main(){                             
    FragColor = vec4(0.0, 1.0, 1.0, 1.0);
}";
 
c++
Shader* a = new Shader();                         // Создаём шейдер
a->addShader(testFragShader, GL_FRAGMENT_SHADER); // Добавление 
a->addShader(testVertShader, GL_VERTEX_SHADER);
Shader::setShader(a);                             // Применение
```

```c++
Shader::Shader(){
    shaderProgram = glCreateProgram();
}

void Shader::addShader(char* pShaderText, GLenum shaderType_){
    if (shaderProgram == 0) {
        printf("Error creating shader program\n");
        exit(1);
    }

    createShader(pShaderText, shaderType_);

    GLint Success = 0;
    GLchar ErrorLog[1024] = { 0 };

    glLinkProgram(shaderProgram);
    glGetProgramiv(shaderProgram, GL_LINK_STATUS, &Success);
    if (Success == 0) {
        glGetProgramInfoLog(shaderProgram, sizeof(ErrorLog), NULL, ErrorLog);
        printf("Error linking shader program: '%s'\n", ErrorLog);
        exit(1);
    }

    glValidateProgram(shaderProgram);
    glGetProgramiv(shaderProgram, GL_VALIDATE_STATUS, &Success);
    if (!Success) {
        glGetProgramInfoLog(shaderProgram, sizeof(ErrorLog), NULL, ErrorLog);
        printf("Invalid shader program: '%s'\n", ErrorLog);
        exit(1);
    }
}

void Shader::createShader(const char* pShaderText, GLenum shaderType){
    GLuint ShaderObj = glCreateShader(shaderType);

    if (ShaderObj == 0) {
        printf("Error creating shader type %d\n", shaderType);
        exit(0);
    }

    const GLchar* p[1];
    p[0] = pShaderText;
    GLint Lengths[1];
    Lengths[0] = strlen(pShaderText);
    glShaderSource(ShaderObj, 1, p, Lengths);
    glCompileShader(ShaderObj);
    
    GLint success;
    glGetShaderiv(ShaderObj, GL_COMPILE_STATUS, &success);                     // Проверка           
    if (!success) {
        GLchar InfoLog[1024];
        glGetShaderInfoLog(ShaderObj, 1024, NULL, InfoLog);
        printf("Error compiling shader type %d: '%s'\n", shaderType, InfoLog);
        exit(1);
    }

    glAttachShader(shaderProgram, ShaderObj);
}

Shader* Shader::shader;

void Shader::setShader(Shader* s) {
    glUseProgram(s->shaderProgram);

    Info::gWorldLocation = glGetUniformLocation(s->shaderProgram, "gWorld");
    assert(Info::gWorldLocation != 0xFFFFFFFF);

    shader = s;
}
```
# Матрицы преобразований
## Вращение 
Вокруг X
```c++
	float temp = angle / ANGLE_TO_PI;
	float tCos = cosf(temp);
	float tSin = sinf(temp);
	glm::mat4 rot;
	rot[0][0] = 1.0f; rot[0][1] = 0.0f; rot[0][2] = 0.0f;  rot[0][3] = 0.0f;
	rot[1][0] = 0.0f; rot[1][1] = tCos; rot[1][2] = -tSin; rot[1][3] = 0.0f;
	rot[2][0] = 0.0f; rot[2][1] = tSin; rot[2][2] = tCos;  rot[2][3] = 0.0f;
	rot[3][0] = 0.0f; rot[3][1] = 0.0f; rot[3][2] = 0.0f;  rot[3][3] = 1.0f;
```
Вокруг Y
```c++
	glm::mat4 rot;
	rot[0][0] = tCos; rot[0][1] = 0.0f; rot[0][2] = -tSin;  rot[0][3] = 0.0f;
	rot[1][0] = 0.0f; rot[1][1] = 1.0f; rot[1][2] = 0.0f;   rot[1][3] = 0.0f;
	rot[2][0] = tSin; rot[2][1] = 0.0f; rot[2][2] = tCos;   rot[2][3] = 0.0f;
	rot[3][0] = 0.0f; rot[3][1] = 0.0f; rot[3][2] = 0.0f;   rot[3][3] = 1.0f;
```
Вокруг Z
```c++
	glm::mat4 rot;
	rot[0][0] = tCos; rot[0][1] = -tSin; rot[0][2] = 0.0f;  rot[0][3] = 0.0f;
	rot[1][0] = tSin; rot[1][1] = tCos;  rot[1][2] = 0.0f;  rot[1][3] = 0.0f;
	rot[2][0] = 0.0f; rot[2][1] = 0.0f;  rot[2][2] = 1.0f;  rot[2][3] = 0.0f;
	rot[3][0] = 0.0f; rot[3][1] = 0.0f;  rot[3][2] = 0.0f;  rot[3][3] = 1.0f;
```
## Деформирование 
```c++
void MatrixWorker::scaleX(glm::mat4& mat, double power){
	glm::mat4 squ;
	squ[0][0] = power; squ[0][1] = 0.0f; squ[0][2] = 0.0f; squ[0][3] = 0.0f;
	squ[1][0] = 0.0f;  squ[1][1] = 1.0f; squ[1][2] = 0.0f; squ[1][3] = 0.0f;
	squ[2][0] = 0.0f;  squ[2][1] = 0.0f; squ[2][2] = 1.0f; squ[2][3] = 0.0f;
	squ[3][0] = 0.0f;  squ[3][1] = 0.0f; squ[3][2] = 0.0f; squ[3][3] = 1.0f;

	mat = mat * squ;
}

void MatrixWorker::scaleY(glm::mat4& mat, double power){
	glm::mat4 squ;
	squ[0][0] = 1.0f; squ[0][1] = 0.0f; squ[0][2] = 0.0f;  squ[0][3] = 0.0f;
	squ[1][0] = 0.0f; squ[1][1] = power; squ[1][2] = 0.0f; squ[1][3] = 0.0f;
	squ[2][0] = 0.0f; squ[2][1] = 0.0f; squ[2][2] = 1.0f;  squ[2][3] = 0.0f;
	squ[3][0] = 0.0f; squ[3][1] = 0.0f; squ[3][2] = 0.0f;  squ[3][3] = 1.0f;

	mat = mat * squ;
}

void MatrixWorker::scaleZ(glm::mat4& mat, double power){
	glm::mat4 squ;
	squ[0][0] = 1.0f; squ[0][1] = 0.0f; squ[0][2] = 0.0f;  squ[0][3] = 0.0f;
	squ[1][0] = 0.0f; squ[1][1] = 1.0f; squ[1][2] = 0.0f;  squ[1][3] = 0.0f;
	squ[2][0] = 0.0f; squ[2][1] = 0.0f; squ[2][2] = power; squ[2][3] = 0.0f;
	squ[3][0] = 0.0f; squ[3][1] = 0.0f; squ[3][2] = 0.0f;  squ[3][3] = 1.0f;

	mat = mat * squ;
}
```
## Объединение преобразований:
У объекта есть матрицы преобразований
```c++
class Drawable{
...
	glm::mat4 translateTransform;
	glm::mat4 rotateTransform;
	glm::mat4 scaleTransform;
...
};
```
Для совместного применения преобразований перемножаем матрицы
```c++
glm::mat4 Drawable::getTrans(){
    glm::mat4 mat;
    MatrixWorker::single(mat);
    mat = mat * scaleTransform;
    mat = mat * rotateTransform;
    mat = mat * translateTransform;

    return mat;
}
```
При отрисовке применяем их к мировой матрице
```c++
void Drawable::paint() {
    glm::mat4 t = Info::worldMatrix;
    glm::mat4 trans = getTrans() * t;
    Info::setWorldMatrix(trans);

    paintMid();
       
    Info::setWorldMatrix(t);
}
```
```c++
void Info::setWorldMatrix(glm::mat4& mat){
	Info::worldMatrix = mat;
	glUniformMatrix4fv(Info::gWorldLocation, 1, GL_TRUE, &mat[0][0]);
}
```
# Проекция перспективы
Создание матрицы перспективы
```c++
void MatrixWorker::initPerspectiveProj(glm::mat4& mat) {
	const float ar = 1;
	const float zNear = 1;
	const float zFar = 100;
	const float zRange = zNear - zFar;
	const float tanHalfFOV = tanf(45.0 / 2.0 / ANGLE_TO_PI);

	mat[0][0] = 1.0f / (tanHalfFOV * ar); mat[0][1] = 0.0f;              mat[0][2] = 0.0f;                     mat[0][3] = 0.0f;
	mat[1][0] = 0.0f;                     mat[1][1] = 1.0f / tanHalfFOV; mat[1][2] = 0.0f;                     mat[1][3] = 0.0f;
	mat[2][0] = 0.0f;                     mat[2][1] = 0.0f;              mat[2][2] = (-zNear - zFar) / zRange; mat[2][3] = 2.0f * zFar * zNear / zRange;
	mat[3][0] = 0.0f;                     mat[3][1] = 0.0f;              mat[3][2] = 1.0f;                     mat[3][3] = 0.0f;
}
```

В функции отрисовки ставим матрицу перспективы как мировую 
```c++
MatrixWorker::single(World);
MatrixWorker::single(moveMat);

MatrixWorker::moveZ(moveMat, Scale / 100.0 + 5);
MatrixWorker::initPerspectiveProj(World);

World = moveMat * World;

Info::setWorldMatrix(World);
```

