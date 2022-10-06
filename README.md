# DA-in-GameDev-Lab2
# СБОР, ОБРАБОТКА И ВИЗУАЛИЗАЦИЯ ТЕСТОВОГО НАБОРА ДАННЫХ. [in GameDev]
Отчет по лабораторной работе #2 выполнил(а):
- Абакарова Кистаман Умарасхабовна
- X21IT_AI-01BL
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | # | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Познакомиться с программными средствами для организции
передачи данных между инструментами google, Python и Unity


Постановка задачи.
В данной лабораторной работе на языке python будет реализован функционал,
позволяющий генерировать стоимость товара (ресурса или игрового объекта) в виде
набора данных. Созданный набор данных будет передан в google-таблицу с целью
возможности дальнейшего их наглядного представляния и оптимизации. Также в
этой лабораторной работе на движке Unity будет реализован функционал,
позволяющий воспроизводить аудио-файлы со звуковой информацией в
зависимости от значений входного набора данных из таблицы.


## Задание 1
### Реализовать совместную работу и передачу данных в связке Python - Google-Sheets – Unity.

Сначала было необходимо в облачном сервисе google console подключить API для работы с google
sheets и google drive. Создать уникальный ключ для обращения.
Создать пустую таблицу в Google Sheets, для переноса данных.
В настройках доступа дать доступ по email, созданным в Google Cloud Service.

![APIKey](https://user-images.githubusercontent.com/48391156/194374305-8f1d2bdb-2363-413e-abdf-b55df534b40f.png)

```py
import gspread
import numpy as np
gc = gspread.service_account(filename='unityds-246dc8312ba8.json')
sh = gc.open("UnitySheets")
price = np.random.randint(2000, 10000, 11)
mon = list(range(1,11))
i = 0
while i <= len(mon):
    i += 1
    if i== 0:
        continue
    else:
        tempInf = ((price[i-1]-price[i-2])/price[i-2])*100
        tempInf = str(tempInf)
        tempInf = tempInf.replace('.', ',')
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(price[i-1]))
        sh.sheet1.update(('C' + str(i)), str(tempInf))
        print(tempInf)
```

Вот вывод в таблицы:
![Gsheets](https://user-images.githubusercontent.com/48391156/194130098-2e976663-0824-4dd0-87d5-9daf8f94d38c.png)

Был создан Unity проект, где будут воспроизводиться треки в зависимости от полученных данных из таблицы.
![unitysounds](https://user-images.githubusercontent.com/48391156/194374964-e8f814a2-ca16-4887-b564-aca003c5fcf2.png)

![image](https://user-images.githubusercontent.com/48391156/194391234-3ac22879-6716-4cd4-a2ce-3e8037a045b1.png)


Реализация получения данных из таблиц:
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string,float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (dataSet["Mon_" + i.ToString()] <= 10 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] > 10 & dataSet["Mon_" + i.ToString()] < 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] >= 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1db3SpD7IfABScVSAwswjPg_cWyLS1F8ffwPPdMFt3ac/values/Лист1?key=AIzaSyAF3FRZfN5RX_4LaW4X0a2NNEJQalXYOfw");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
        }
    }
```

В зависимости от полученных данных, воспроизводится аудио
```c#
   IEnumerator PlaySelectAudioGood()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = goodSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioNormal()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = normalSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioBad()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = badSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
}
```

## Задание 2


## Задание 3


## Выводы
Я получила базовые навыки работы с google cloud console, получать и использовать данные из таблиц.


| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
