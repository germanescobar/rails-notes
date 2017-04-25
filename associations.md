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
```
