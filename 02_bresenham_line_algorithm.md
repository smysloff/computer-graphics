# Лекция №2: "Алгоритм Брезенхема для рисования линии"

Зачем это нужно?

Компьютер не умеет рисовать идеально прямые линии. Он может только ставить пиксели в узлах целочисленной сетки. Наша задача — найти такие пиксели, которые наилучшим образом аппроксимируют идеальную линию от точки (x0, y0) до (x1, y1). Алгоритм Брезенхема — это гениальный и быстрый способ сделать это, используя только целочисленные вычисления (сложение и вычитание).

Основная идея:

На каждом шаге мы двигаемся по оси X на один пиксель и решаем, нужно ли нам двигаться по оси Y или оставаться на той же строке. Решение принимается на основе анализа ошибки — расстояния между идеальным положением линии и ближайшими пикселями.

Выведем алгоритм для случая, когда угол наклона меньше 45 градусов (0 < Δy/Δx < 1):

* Считаем приращения:
dx = x1 - x0
dy = y1 - y0

* Вводим переменную ошибки (решающий параметр):
Изначально error = 0

* Начальные координаты:
x = x0, y = y0

* Рисуем начальный пиксель.

* Цикл от x0 до x1:

    - Увеличиваем x на 1.

    - Увеличиваем ошибку на величину угла наклона: error = error + dy/dx

    - Если error > 0.5, это значит, что наша идеальная линия "ближе" к пикселю на строку выше (y+1). Тогда мы:

        - Увеличиваем y на 1.

        - Уменьшаем ошибку на 1 (error = error - 1), т.к. мы "скомпенсировали" эту ошибку шагом.

    - Рисуем пиксель в новой позиции (x, y).


Но! Мы договорились обойтись без дробей. Умножим все на 2 * dx:

    - Изначально: error = 0 -> error = 0

    - Сравнение с 0.5 -> сравнение с dx

    - error = error + dy/dx -> error = error + 2 * dy

    - error = error - 1 -> error = error - 2 * dx

Итоговый алгоритм (целочисленный, оптимизированный):

    - dx = x1 - x0

    - dy = y1 - y0

    - error = 0

    - y = y0

    - delta_err = 2 * dy - 2 * dx // Это мы позже выведем, но для начала используем другой вариант

    - Цикл для x от x0 до x1:

        - drawPixel(x, y)

        - error = error + 2 * dy

        - Если error > dx:

            - y = y + 1

            - error = error - 2 * dx


Практическая реализация на Canvas:

Мы не можем просто "поставить пиксель" стандартными методами, поэтому будем использовать наш низкоуровневый подход с ImageData.

Важные моменты для реализации:

* Алгоритм нужно обобщить для всех случаев (любые октанты).

* Нужно определить, по какой оси вести основной шаг (если dx больше dy, то ведем по X, иначе — по Y).


## Задачи на второй урок:

* Базовая реализация. Напиши функцию drawLine(x0, y0, x1, y1, r, g, b, a), которая рисует линию по алгоритму Брезенхема для случая, когда dx > dy > 0 (первый октант). Нарисуй несколько линий под разными углами.

* Обобщение. Усовершенствуй функцию, чтобы она могла рисовать линии в любом направлении (любые октанты). Вспомогательные шаги:
    - Определи знаки приращений dx и dy.
    - Определи, по какой оси вести основной шаг (если |dx| > |dy|, то по X, иначе по Y).
    - В зависимости от этого, адаптируй основной цикл.

* Тест. Нарисуй с помощью своей функции простой домик или звезду, чтобы проверить ее работу во всех направлениях.

* Необязательное, на смекалку: Сравни скорость работы твоей функции с стандартной ctx.lineTo(). Нарисуй 10 000 линий и замерь время.


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

}

const gc = new GC(640, 360)
gc.mount(document.body)
gc.rgba(0xff, 0x00, 0x00)

gc.rgba(0x00, 0xff, 0x00)
gc.line(320, 180, 420, 180)
gc.line(320, 180, 320, 280)
gc.line(320, 180, 220, 180)
gc.line(320, 180, 320,  80)

gc.rgba(0xff, 0x00, 0x00)
gc.line(320, 180, 420, 280)
gc.line(320, 180, 220, 280)
gc.line(320, 180, 220,  80)
gc.line(320, 180, 420,  80)

gc.rgba(0x00, 0x00, 0xff)
gc.line(320, 180, 420, 230)
gc.line(320, 180, 370, 280)
gc.line(320, 180, 270, 280)
gc.line(320, 180, 220, 230)
gc.line(320, 180, 220, 130)
gc.line(320, 180, 270,  80)
gc.line(320, 180, 370,  80)
gc.line(320, 180, 420, 130)

console.time('Custom Brasenhem Lines')
gc.rgba(0xff, 0xff, 0xff)
for (let i = 0; i < 10000; ++i) {
  const x0 = Math.floor(Math.random() * (gc.element.width + 1))
  const y0 = Math.floor(Math.random() * (gc.element.height + 1))
  const x1 = Math.floor(Math.random() * (gc.element.width + 1))
  const y1 = Math.floor(Math.random() * (gc.element.height + 1))
  gc.line(x0, y0, x1, y1)
}
console.timeEnd('Custom Brasenhem Lines') // 97-149ms
gc.blit()

console.time('Canvas Lines API')
gc.context.beginPath()
for (let i = 0; i < 10000; ++i) {
  const x0 = Math.floor(Math.random() * (gc.element.width + 1))
  const y0 = Math.floor(Math.random() * (gc.element.height + 1))
  const x1 = Math.floor(Math.random() * (gc.element.width + 1))
  const y1 = Math.floor(Math.random() * (gc.element.height + 1))
  gc.context.moveTo(x0, y0)
  gc.context.lineTo(x1, y1)
}
gc.context.strokeStyle = 'White'
gc.context.stroke()
console.timeEnd('Canvas Lines API') // 3-5ms
```

## Дополнительно

### Оптимизированный алгоритм:

```javascript

const { abs, sign } = Math

function line(x0, y0, x1, y1) {

  const dx = abs(x1 - x0)
  const dy = abs(y1 - y0)
  const sx = sign(x1 - x0)
  const sy = sign(y1 - y0)

  let err = dx - dy

  while (true) {
    putPixel(x0, y0)
    if (x0 === x1 && y0 === y1) break
    const e2 = 2 * err
    if (e2 > -dy) { err -= dy; x0 += sx }
    if (e2 <  dx) { err += dx; y0 += sy }
  }
}

```
