# Лекция №3:<br>Окружность по Брезенхему<br>Алгоритм средней точки

## Основная идея:

Мы используем симметрию окружности. Нам не нужно считать всю окружность, достаточно рассчитать точки для одной октанты (1/8 части), а остальные достроить просто меняя знаки и координаты!

## Как это работает:

* Мы начинаем от точки (0, R).

* Мы движемся по шагам `x` и на каждом шаге решаем: опускаться ли нам вниз по `y` или нет.

* Решение мы принимаем based on "промежуточной ошибки" — мы смотрим на положение средней точки между двумя пикселями, которые мы можем выбрать дальше.

## Вывод алгоритма (шаг за шагом):

Уравнение окружности: `x2 + y2 = R2`

Наша функция: `F(x,y) = x2 + y2 − R2`

* Если `F(x,y) = 0` — точка на окружности.

* Если `F(x,y) > 0` — точка снаружи.

* Если `F(x,y) < 0` — точка внутри.

Вводим параметр решения (`P`):

На каждом шаге мы выбираем между пикселем `E (East)` и `SE (South-East)`.

Мы смотрим на среднюю точку между ними — `M`.

* Если `M` внутри окружности — выбираем `E`.

* Если `M` снаружи — выбираем `SE`.

И получаем наш алгоритм:

```txt

Инициализация:
x = 0
y = R
P = 1 - R (это и есть начальное значение ошибки)

Пока x <= y:

Рисуем точки во всех октантах по симметрии: (x, y), (y, x), и с разными знаками.

    Если P < 0:
        P = P + 2*x + 3

    Else:
        P = P + 2*(x - y) + 5
        y = y - 1

    x = x + 1

```

## Разбор "магии" коэффициентов (шаг за шагом)

Вспомним нашу функцию решения: `F(x, y) = x² + y² - R²`

Мы в точке `(x, y)`. Мы двигаемся на `x+1`.

Нам нужно выбрать между пикселями `E (x+1, y)` и `SE (x+1, y-1)`.

Мы смотрим на среднюю точку между ними: `M (x+1, y-0.5)`.

### 1. Считаем значение функции в точке `M`:

`F(M) = F(x+1, y-0.5) = (x+1)² + (y-0.5)² - R²`

Это и есть наша ошибка (`P`)!

Если `P < 0`, точка `M` внутри окружности, идем в `E`.

Если P >= 0, идем в SE.

### 2. Но мы не хотим работать с дробями! Умножим всё на 4, чтобы избавиться от 0.5 (получится 2 и 1 в квадратах):

`4 * F(M) = 4 * [ (x+1)² + (y-0.5)² - R² ] = 4*(x+1)² + 4*(y-0.5)² - 4*R² = 4*(x²+2x+1) + (4y² - 4y + 1) - 4R²`

Упрощаем:

`4*F(M) = 4x² + 8x + 4 + 4y² - 4y + 1 - 4R² = (4x² + 4y² - 4R²) + 8x - 4y + 5`

Замечаем, что `(4x² + 4y² - 4R²) = 4*(x² + y² - R²) = 4 * F(x, y)`.

Но `F(x, y)` мы не считали! Мы считали `P` на предыдущем шаге. И вот тут ключевой момент:

Наша переменная `error` в коде — это и есть `4 * F(M)` для текущей точки! Но чтобы не тащить за собой `4*R²`, ее преобразуют.

### 3. Вывод рекуррентной формулы (как меняется P):

* Если мы идем в E (т.е. P < 0), то следующая средняя точка будет M_new (x+2, y-0.5).
Разница `ΔP_east = 4*F(M_new) - 4*F(M) = ... = 8x + 12` (после всех вычислений).

* Если мы идем в SE (т.е. P >= 0), следующая точка M_new (x+2, y-1.5).
Разница `ΔP_south_east = 4*F(M_new) - 4*F(M) = ... = 8x - 8y + 20`.

### 4. Финальная оптимизация:

Чтобы избавиться от больших коэффициентов, все приращения можно снова поделить на 4. Но поскольку нам важны только знаки, а не абсолютные значения, это допустимо. После деления получатся те самые формулы:

* Для `E: ΔP = 2x + 3`

* Для `SE: ΔP = 2x - 2y + 5`


## Задачи на реализацию:

* База. Напиши функцию circle(cx, cy, radius) внутри твоего класса GC.

* Симметрия. Внутри цикла рисуй сразу 8 точек:

```txt
    putPixel(cx + x, cy + y)
    putPixel(cx - x, cy + y)
    putPixel(cx + x, cy - y)
    putPixel(cx - x, cy - y)
    putPixel(cx + y, cy + x)
    putPixel(cx - y, cy + x)
    putPixel(cx + y, cy - x)
    putPixel(cx - y, cy - x)
```

* Тест. Нарисуй несколько окружностей разного радиуса, чтобы проверить работу.


## Решение

```javascript
class GC {

  constructor(width = 320, height = 180, bgcolor = 'Black') {
    this.element = document.createElement('canvas')
    this.context = this.element.getContext('2d')
    this.element.width = width
    this.element.height = height
    this.context.fillStyle = bgcolor
    this.clean()
    this.imageData = this.context.getImageData(0, 0, this.element.width, this.element.height)
    this.rgba(0xff, 0xff, 0xff)
  }

  mount(element) {
    element.append(this.element)
  }

  clean() {
    this.context.fillRect(0, 0, this.element.width, this.element.height)
  }

  blit() {
    this.context.putImageData(this.imageData, 0, 0)    
  }

  rgba(r, g, b, a = 0xff) {
    this.r = r
    this.g = g
    this.b = b
    this.a = a
  }

  pixel(x, y) {
    const index = (y * this.imageData.width + x) * 4
    this.imageData.data[index + 0] = this.r
    this.imageData.data[index + 1] = this.g
    this.imageData.data[index + 2] = this.b
    this.imageData.data[index + 3] = this.a
  }

  line(x0, y0, x1, y1) { // brasenham's line algorithm

    let sx = (x1 - x0) < 0 ? -1 : 1
    let sy = (y1 - y0) < 0 ? -1 : 1

    let dx = Math.abs(x1 - x0)
    let dy = Math.abs(y1 - y0)

    const dx2 = dx * 2
    const dy2 = dy * 2

    let error = 0
    let x = x0
    let y = y0
    this.pixel(x, y)
    if (dx >= dy) {
      while (x !== x1) {
        x += sx
        error += dy2
        if (error > dx) {
          y += sy
          error -= dx2
        }
        this.pixel(x, y)
      }
    } else {
      while (y !== y1) {
        y += sy
        error += dx2
        if (error > dy) {
          x += sx
          error -= dy2
        }
        this.pixel(x, y)
      }
    }
  }

  circle(cx, cy, r) {

    //const f = (x, y, r2) => x * x + y * y - r2 > 0
    //const r2 = r * r

    const draw = (cx, cy, x, y) => {
      this.pixel(cx + x, cy + y)
      this.pixel(cx - x, cy + y)
      this.pixel(cx + x, cy - y)
      this.pixel(cx - x, cy - y)
      this.pixel(cx + y, cy + x)
      this.pixel(cx - y, cy + x)
      this.pixel(cx + y, cy - x)
      this.pixel(cx - y, cy - x)
    }

    let error = 1 - r

    let x = 0
    let y = r

    draw(cx, cy, x, y)

    while (x <= y) {
      ++x
      //if (f(x, y, r2)) --y
      if (error < 0) {
        error += 2 * x + 3
      } else {
        error += 2 * (x - y) + 5
        --y
      }
      draw(cx, cy, x, y)
    }

  }

}

const gc = new GC(640, 360)
gc.mount(document.body)

const cx = gc.element.width * .5
const cy = gc.element.height * .5

gc.rgba(0xff, 0xff, 0xff)
gc.circle(cx, cy, 100)

gc.rgba(0x00, 0xff, 0x00)
gc.circle(cx, cy, 1)

gc.rgba(0x00, 0xff, 0xff)
gc.circle(196, 128, 64)

gc.blit()

```
