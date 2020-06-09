# BigData_project

[Ссылка на презентацию](https://www.shorturl.at/krxU4)  
[Ссылка на датасет](https://www.kaggle.com/function9/blues-genre-midi-melodies)

### Постановка задачи

Продолжить мелодию по нескольким нотам, с учетом различных октав и длительностей.

Для выполнения данной задачи, мы сначала выделили основные этапы:
* поиск подходящего датасета;
* перевод нот в удобный для работы с нейросетью алфавит;
* составление словарей с нотами;
* поиск оптимальной архитектуры сети.


### Поиск датасета
Так как мы нашли датасет который состоит только из мелодий, нам не пришлось решать задачу отделения мелодии от аккомпанимента. С одной стороны это облегчило нам задачу, с другой наша система работает только с форматами `mid` и `midi`. [`MIDI`](https://ru.wikipedia.org/wiki/MIDI) - это стандарт цифровой звукозаписи на формат обмена данными между электронными музыкальными инструментами. [Ссылка на датасет.](https://www.kaggle.com/function9/blues-genre-midi-melodies)


### Перевод в ноты и из нот

Формат `MIDI` нам позволяет узнать, какая клавиша (нота) была зажата или отпущена. Каждому номеру соответствует определенная нота в определенной октаве. Так же в каждом сообщении передается время, которое прошло с предыдущего сообщения. Таким образом можно вычленить саму ноту и ее длительность. Обычно файлы `midi` состоят из команд, где каждая команда означает или зажатие клавиши или ее отпускание.  

Содержание команды: 
* `note_on`  - клавиша зажата
* `note_off` - клавиша отпущена
* `note` - номер клавиши
* `time` - количество тиков с момента предыдущей команды

В нашей системе было решено оставить стандартное обозначение нот в строчном варианте (adcdefg), в прописном варианте - нота с диезом (ABCDEFG). Соответственно из номера клавиши можно получить ноту и её октаву. Однако данных о том какая нота сейчас играет недостаточно, для воспроизведения мелодии также необходима ее длительность. В `midi` длительность выражается в тиках. Тики - это число, которое отражает такт мелодии. Обычно такт мелодии это 3/4 или 4/4. А тики показывают число ударов в 1/4. Поэтому, чтобы получить длительность ноты, нужно разделить длительность ноты в тиках на общее количество тиков (192) умноженное на 4. Отсюда выводится данная формула: 
``
Длительность = time/(192*4)
``

Еще считаются паузы; обозначаются р0х, где х - длительность паузы

#### Пример:

![таблица с переводом нот](pictures/table.png)

```
note_on channel=8 note=58 velocity=96 time=2
note_off channel=8 note=58 velocity=0 time=183
```

Включилась нота 58 - это ля диез 3 октавы. Значит код этой ноты будет А4. (0:это -1 октава, 1: субконтроктава и т.д.).

Теперь с длительностями. time = 2 в первой строчке означает, что с последней команды прошло всего 2 тика. Обычно тиков - 192. Значит пауза длилась 2/(192\*4). Это даже меньше 1/16 - а это самая короткая длительность в мелодии. Поэтому даже не учитываем эту паузу. Следующая строчка - так же нота. Выключение произошло через 183 тика после включения. Это 183/(192\*4) = примерно 1/4
 
А ну, и полный код этой ноты получился А42 - ля диез 3 октавы с длительностью 1/4.
Я в коде в комменте писала, какая длительность какой цифре соответствует: 0 - 1/16, 1 - 1/8, 2 - 1/4 и т.д.
```
0 - 1/16
1 - 1/8
2 - 1/4
3 - 3/8
4 - 1/2
5 - 3/4
6 - 1
```
Но кое-что не получается совсем. Тот код, который уже предсказывает - есть тот, который из лекции, есть тот, который я написала. Первый берет 10 нот из данных и предсказывает одну ноту. Я изменила немного со смещением и предсказыванием по своим же полученным данным. Но так он даже первую ноту предсказывает криво, хотя на вход даются 10 нот из данных. Я не понимаю, в чём беда..




## Архитектура нейросети

