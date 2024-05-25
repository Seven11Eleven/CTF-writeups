#### RenderQuest write-up
![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/c938b0a7-98df-4d89-86b3-6e39fa94b6a6)
Дают сайт, ничего не понятно, качаем исходники, которые даются в комплекте с задачей.
![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/f312d864-87e0-4512-a56e-1abf84882570)

обнаруживаем, что сайт написан на Go))
Исследуем код и обнаруживаем интересные строчки ![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/fb72f69d-4cf9-433a-8108-52545eb128bf)
```
out, err := exec.Command("sh", "-c", command).Output()
```
этот код запускает shell, который выполняет некую команду, значит уязвимость заключается в reverse shell'инге, можно воспользоваться webhook.site используя темплейты
![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/af769766-e2ff-439f-a1a4-ca3d1120f68e)
{{.FetchServerInfo, "ls"}} этот код создает темплейт, который напрямую использует функцию FetchServerInfo и запускает команду ls в шеле, которая перечисляет все файлы и директории в пространстве.

Вводим ссылку, которую создали в форму ввода и получаем следующее
![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/7e0e6470-6204-4a22-ba1b-6ea470dd184c)
Видим файловую архитектуру приложения, вспоминаем, что нам давали в исходниках, вспоминаем где находится flag.txt там
![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/aa40c486-0e3d-4584-a760-30e36e737848)
флаг находится на директорию выше, поэтому нужно выйти из challenge
![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/a05ce574-1533-42de-810b-f3f8ac07b0c2)
cd .. значит перейти в родительскую директорию ; ls -la покажет нам ВСЕ директории и файлы, даже скрытые
![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/ec9e2c92-426d-475f-8d57-3dfa41ef6a59)
обнаруживаем файл flag.txt, который имеет в названии некий шифр, видимо чтобы нельзя было просто catнуть флаг, копируем и добавляем новую команду в темплейт
![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/3fade326-33a7-406f-a3f9-c4fca3e17ef5)
теперь смотрим что выйдет

![image](https://github.com/Seven11Eleven/CTF-writeups/assets/120821293/b6a4e074-0c05-4dac-8017-abbcdc9d78ed)

Флаг найден, сабмиттим и готово
