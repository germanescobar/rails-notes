# Asociaciones

Las asociaciones se utilizan para definir relaciones entre tablas de una base de datos. Existen dos tipos de asociaciones que se pueden modelar en una base de datos relacional:

* One to many (uno a muchos)
* Many to many (muchos a muchos)

## One to many (uno a muchos)

En una relación uno a muchos **cada registro de una tabla está relacionado a un registro de otra tabla** a través de una llave foránea.

Por ejemplo, en nuestra aplicación de notas, cada nota está relacionada a un usuario.

Imagina que existe una tabla `users` con los siguientes campos:

| id | name  |
|----|-------|
| 1  | Pedro |
| 2  | Juan  |

La tabla `notes` tendría un campo relacionado (llave foránea) a `users`:

| id | user_id | content |
|----|---------|---------|
| 1  | 2       | Nota 1  |
| 2  | 2       | Nota 2  |
| 3  | 1       | Nota 3  |

En este caso, cada registro en la tabla `notes` está asociado a un registro en `users` a través de la llave foránea `user_id`.

Para definir una relación uno a muchos se deben realizar lo siguientes pasos:

1. Crear el modelo que no tiene la llave foránea. Por ejemplo:
```
$ rails g model User name
```

2. Crear el modelo que va a estar relacionado al anterior utilizando `references` para crear la referencia. Por ejemplo:
```
$ rails g model Note user:references content:text
````
El `user:references` crea la llave foránea (`user_id`) y le agrega el `belongs_to :user` a `Note`.

3. Agregar el `has_many` al primer modelo que se creó. Por ejemplo:
```ruby
class User < ApplictionRecord
  has_many :notes
end
```

Veamos ahora las acciones comunes que se realizan con las relaciones uno a muchos:

### Obtener el usuario de una nota

```
$ rails c
> note = Note.first
> note.user
```

### Ver las notas de un usuario

```
$ rails c
> user = User.first
> user.notes
```

### Crear una nota para un usuario

```
$ rails c
> user = User.first
> Note.create(user: user, content: "Nota 1")
> Note.create(user_id: user.id, content: "Nota 2")
> user.notes.create(content: "Nota 3")
```

### Ejercicio

Debes crear una nueva aplicación de Rails con dos modelos: `Course` y `Subject` (tema). Un `Course` tiene un `name`. Un `Subject` pertenece a un `Course` y tiene un `name`.

Debes implementar la siguiente funcionalidad:

* Listar cursos (En la lista debe aparecer el nombre y el número de temas)
* Ver el detalle de un curso (debe aparecer el nombre y la lista de temas).
* Crear un tema. En el detalle del curso debe existir un botón para crear un tema para ese curso.

## Many to many (muchos a muchos)

Una asociación muchos a muchos es un tipo de asociación en donde **un registro de una tabla puede estar relacionada a muchos registros de otra otra tabla**.

Imagina la relación entre cursos y estudiantes. Un estudiante puede estar relacionado a muchos cursos y un curso puede tener muchos estudiantes.

Para definir una relación muchos a muchos se debe **crear una tabla intermedia** que relacione las dos tablas.

Imagina que existe una tabla `courses` con los siguientes campos:

| id | name       |
|----|------------|
| 1  | HTML y CSS |
| 2  | Bootstrap  |

Imagina también que existe una tabla `students` con los siguientes campos:

| id | name       |
|----|------------|
| 1  | Pedro      |
| 2  | Juan       |

Para relacionar cursos con estudiantes necesitamos una tabla adicional. Para nombrarla se utiliza la combinación de los dos nombres de las tablas separados por raya al piso (`_`). En este ejemplo el nombre sería `courses_students` (también podría llamarse `students_courses` pero por notación se utiliza el orden alfabético).

La tabla `courses_students` tendría la siguiente estructura:

| course_id | student_id |
|-----------|------------|
| 1         | 1          |
| 1         | 2          |
| 2         | 2          |

¿Qué alumnos están asociados al curso de Bootstrap? ¿Qué alumno está asociado a los dos cursos?

A la tabla intermedia se le conoce como una **join table*.

Para implementar esta asociación seguiríamos estos pasos:

1. Crear los dos modelos. Por ejemplo:
```
$ rails g model Course name
$ rails g model Student name
```

2. Crear la tabla intermedia con una migración. Esta tabla no tiene un modelo relacionado:
```
$ rails g migration create_join_table_courses_students course:references student:references
```

3. Agregar `has_and_belongs_to_many` a los dos modelos. Por ejemplo:
```ruby
class Course < ApplictionRecord
  has_and_belongs_to_many :students
end

class Student < ApplicationRecord
  has_and_belongs_to_many :courses
end
```

Veamos ahora las acciones comunes que se realizan con las relaciones muchos a muchos:

### Obtener los cursos de un estudiante y viceversa (los estudiantes de un curso)

```
$ rails c
> student = Student.first
> student.courses
...
> course = Course.first
> course.students
...
```

### Relacionar un curso a un estudiante y viceversa (estudiantes a cursos)

```
$ rails c
> course = Course.first
> student = Student.first
> student.courses << course
```

No es necesario volver a guardar el modelo. La última línea hace el INSERT en la tabla intermedia. Lo podemos hacer al revés (relacionar el curso al estudiante) y tendríamos el mismo resultado.

```
$ rails c
> course = Course.first
> student = Student.first
> course.students << student
```

Si estamos creando un estudiante y queremos asociarle de una vez algunos cursos podemos hacer lo siguiente:

```
$ rails c
> course = Course.first
> Student.create(name: "Pedro", course_ids: [course.id])
```

### Desasociar un curso de un estudiante y viceversa

```
$ rails c
> course = Course.first
> student = Student.first
> course.students.delete(student)
```

Acá estamos asumiendo esos dos registros están asociados, aunque si no lo están no ocurre ningún error, simplemente no cambia nada en la base de datos.

### Ejercicio

Intenta hacer el reto "Productos" del tema "Ruby on Rails I" en la plataforma de Make it Real.

## has_many :through

`has_and_belongs_to_many` funciona bien cuando la tabla intermedia no tiene información adicional. Sin embargo, ¿qué pasaría si queremos saber la fecha en la que se matriculó el estudiante al curso?

En ese caso, en vez de crear una tabla intermedia sin modelo, vamos a crear una tabla intermedia **con modelo**. Al nuevo modelo lo vamos a llamar `Enrollment`. Los pasos serían los siguientes

1. Crear los tres modelos. Por ejemplo:
```
$ rails g model Course name
$ rails g model Student name
$ rails g model Enrollment course:references student:references enrolled_at:date
```

3. Agregar `has_many` y `has_many through` a los dos modelos. Por ejemplo:
```ruby
class Course < ApplictionRecord
  has_many :enrollments
  has_many :students, through: :enrollments
end

class Student < ApplicationRecord
  has_many :enrollments
  has_many :courses, through: :enrollments
end

class Enrollment < ApplicationRecord
  belongs_to :course
  belongs_to :student
end
```

## Asociación polimórfica

La asociación polimórfica es un tipo de asociación uno a muchos que se puede implementar en Ruby on Rails (en la base de datos no se puede implementar directamente) en donde una tabla puede estar asociada a muchas otras tablas.

Imagina el caso de una aplicación que tiene preguntas y respuestas. Tanto las preguntas como las respuestas tienen comentarios. En vez de crear una tabla para los comentarios de las preguntas y otra para los comentarios de las respuestas, podemos crear una única tabla de comentarios que tenga los comentarios tanto de las preguntas como de las respuestas.

La tabla de preguntas (`questions`) tendría la siguiente estructura:

| id | text                |
|----|---------------------|
| 1  | ¿Qué es Ruby?       |
| 2  | ¿Qué es JavaScript? |

La tabla de respuestas (`answers`):

| id | question_id | text                          |
|----|-------------|-------------------------------|
| 1  | 1           | Un lenguaje de programación   |
| 2  | 2           | Otro lenguaje de programación |
| 3  | 2           | No tengo ni idea              |

La tabla de comentarios (`comments`) tendría la siguiente estructura

| id | comentable_type | commentable_id | text                    |
|----|-----------------|----------------|-------------------------|
| 1  | Question        | 1              | Comentario a Question 1 |
| 2  | Answer          | 1              | Comentario a Answer 1   |

La tabla `comments` utiliza dos columnas para identificar la tabla y el id del registro al que va a estar relacionado cada registro.

Para implementar esta asociación seguiríamos estos pasos:

1. Crear los tres modelos. Por ejemplo:

```
$ rails g model Question text:text
$ rails g model Answer question:references text:text
$ rails g model Comment commentable:references{polymorphic} text:text
```

Fíjate que agregamos una referencia en `Comment` a `commentable`, que no es un modelo existente, es simplemente un nombre que le debemos dar a la relación.

2. Agregar las relaciones en los modelos:

```ruby
class Question < ApplicationRecord
  has_many :comments, as: :commentable
end

class Answer < ApplicationRecord
  has_many :comments, as: :commentable
end

class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end
```

## Ejercicios

### Ejercicio 1

Crear el diagrama de tablas y definir los modelos de una aplicación que le va a permitir a los usuarios que se registren publicar sus posts.

El usuario tiene un email y password. Un post tiene un título y un texto. Los posts tienen comentarios. Un comentario tiene un texto y está relacionado a un post y un usuario.

**Avanzado:** Es posible agregarle etiquetas (tags) a los posts. Se debe tener una lista de todas las etiquetas que han creado los usuarios.

### Ejercicio 2

Crear el diagrama de tablas y definir los modelos de una aplicación de preguntas y respuestas similar a StackOverflow que permita a los usuarios comentar sobre las preguntas y respuestas:

* Una pregunta está asociada a un usuario y tiene un texto.
* Una respuesta tiene un texto y está asociada a un usuario y una pregunta.
* Los comentarios tienen un texto y están asociados a un usuario y a una pregunta o respuesta.

Avanzado: Los usuarios pueden votar por las preguntas y respuestas. Se debe saber cuándo fue la fecha que se hizo cada voto.

### Ejercicio 3
