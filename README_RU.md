# detect-file-type

> Определяет типа файла по сигнатурам

## Usage

```js
  var detect = require('detect-file-type');

  detect.fromFile('./image.jpg', function(err, result) {

    if (err) {
      return console.log(err);
    }

    console.log(result); // { ext: 'jpg', mime: 'image/jpeg' }
  });
```

## API

### fromFile(filePath, bufferLength, callback)
Определение типа файла находящегося на диске
- `filePath` - путь к файлу
- `bufferLength` - необязательный параметр. Размер буфера в байтах начиная с начала файла. По умолчания равен 100. В случае если размер файла менее 100 байт будет равен размеру файла.
- `callback`

### fromBuffer(buffer, callback)
Определение типа файла находящегося на диске
- `buffer` - uint8array
- `callback`

### addSignature(siganture)
Добавляет новую сигнатуру для определения файла к уже существующим сигнатурам
- `signature` - сигнатура. Читайте секцию о создании собственных сигнатур ниже.

## Сигнатуры и создание собственных сигнатур
Определение типа файла происходит благодаря сигнатурам.
Простейшая сигнатура в формате json выглядит следующим образом:
```json
{
  "type": "jpg",
  "ext": "jpg",
  "mime": "image/jpeg",
  "rules": [
    { "type": "equal", "start": 0, "end": 2, "bytes": "ffd8"  }
  ]
}
```
где:
- `type` - тип сигнатуры, в большинстве случаев равна полю `ext`
- `ext` - расширение файла
- `mime` - mime тип файла
- `rules` - список правил при помощи которых определяется тип файла

Рассмотрим более подробно поле `rules`:

**Данное поле должно является массивом объектов.**

- `type` - тип правила. Есть несколько типов правил: `equal`, `notEqual`, `contains`, `notContains`, `or`, `and`

Идущий далее набор полей зависит от типа правила.
#### Разберем следующий правила: equal, notEqual, contains и notContains.

- `equal` - обязательно содержит поле `bytes`. Получает срез буфера начиная со `start` (равен 0 если не определено) и заканчивая `end` (равен buffer.length если не определено). Далее сравнивает полученый срез со значением находящимся в поле `bytes`. В случае если значения полностью совпадают - правило считается успешно соблюдённым.
- `notEqual` - обязательно содержит поле `bytes`. Получает срез буфера начиная со `start` (равен 0 если не определено) и заканчивая `end` (равен buffer.length если не определено). Далее сравнивает полученый срез со значением находящимся в поле `bytes`. В случае если значения не совпадают - правило считается успешно соблюдённым.
- `contains` - обязательно содержит поле `bytes`. Получает срез буфера начиная со `start` (равен 0 если не определено) и заканчивая `end` (равен buffer.length если не определено). Далее происходит попытка найти вхождение последовательности байт находящихся в поле `bytes` в полученном срезе. Если вхождение найдено - правило считается успешно соблюдённым.
- `notContains` - обязательно содержит поле `bytes`. Получает срез буфера начиная со `start` (равен 0 если не определено) и заканчивая `end` (равен buffer.length если не определено). Далее происходит попытка найти вхождение последовательности байт находящихся в поле `bytes` в полученном срезе. Если вхождение не найдено - правило считается успешно соблюдённым.

#### Далее рассмотрим типы `or` и `and`.

Данные типы необходимы для определения типа файлов с более сложными сигнатурами. Например когда файл должен содержать несколько последовательностей байт в различных частях файла. Данные типы правил обязательно содержат поле `rules`, в котором указывается набор других правил. Далее следует пример более сложной сигнатуры использующую некоторую логику.

```json
{
  "type": "tif",
  "ext": "tif",
  "mime": "image/tiff",
  "rules": [
    { "type": "and", "rules":
      [
        { "type": "notEqual", "start": 8, "end": 10, "bytes": "4352" },
        { "type": "or", "rules":
            [
             { "type": "equal", "start": 0, "end": 4, "bytes": "49492a00" },
             { "type": "equal", "start": 0, "end": 4,  "bytes": "4d4d002a" }
           ]
          }
      ]
    }
   ]
 }
```

Разберем правило. Если срез буфера начинающийся с 8-го байта и заканчивающийся 10-ым байтом не равен "4352", **при этом** срез буфера начинающийся с 0 (нуля) и заканчивающийся 4-ым байтом равен "49492a00" **либо** равен "4d4d002a" можем считать что данные файл имеет формат tif.

- `or` - означает что любое из правил перечесленных в поле `rules` должно быть соблюдено. Если хотя бы одно правило соблюдено - считается успешно выполненым.
- `and` - означает каждое из правил перечесленных в поле `rules` должно быть соблюдено. Если все правила соблюдены - считается успешно выполненым. В случае, если хотя бы одно правило не соблюдено - cчитается проваленым.
 
Правила `or` и `and` могут иметь любой уровень вложенности.


## License

MIT © [Dmitry Pavlovsky](http://paloskin.me)
