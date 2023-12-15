Отчет по лабораторной работе #2 выполнил(а):
- Соколов Владимир Александрович
- РИ220941
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

## Цель работы
Научиться передавать в Unity данные из Google Sheets с помощью Python.

## Задание 1
### Выберите одну из компьютерных игр, приведите скриншот её геймплея и краткое описание концепта игры. Выберите одну из игровых переменных в игре (ресурсы, внутри игровая валюта, здоровье персонажей и т.д.), опишите её роль в игре, условия изменения / появления и диапазон допустимых значений. Постройте схему экономической модели в игре и укажите место выбранного ресурса в ней.

Ход работы:
- Good Pizza, Great Pizza (Хорошая пицца, Отличная пицца) – аркадныый симулятор пиццерии
- В игре необходимо готовить пиццу для разных людей. Среди них есть персонажи, которые влияют на глобальный сюжет. На заработтанные с пиццы деньги покупается различные улучшения для кухни и новые ингридиенты.
- Как игровую переменную я выбрал местную валюту. Если заказ приготовлен вовремя и там присутствуют все нужные ингридиенты, посетитель оставляет чаевые. В противном случае, он их не оставляет или вовсе забирает деньги назад
![image](https://github.com/WENEEDAPLAN/ALabs2/assets/117916056/b41a26c8-e449-4c04-9080-fa594c7e20d7)
![image](https://github.com/WENEEDAPLAN/ALabs2/assets/117916056/a20d20eb-1bdb-4d67-ba65-0a7c0c6a022a)

## Задание 2
### С помощью скрипта на языке Python заполните google-таблицу данными, описывающими выбранную игровую переменную в выбранной игре (в качестве таких переменных может выступать игровая валюта, ресурсы, здоровье и т.д.). Средствами google-sheets визуализируйте данные в google-таблице (постройте график, диаграмму и пр.) для наглядного представления выбранной игровой величины.

Ход работы:
- Сумма заказа от 1000 до 2000
- Чаевые в диапазоне 10 до 20 процентов
- С вероятностью 0.2 чаевые отрицательны (заказ не понравился) или равны нулю, с вероятностью 0.6 они положительны

```
import gspread
import numpy as np

gc = gspread.service_account(filename='aidew-407713-df6a4ae09be3.json')
sh = gc.open("AIDew")

# Очистить таблицу
sh.sheet1.clear()

price = np.random.randint(1000, 2000, 11)
tipsPercent = np.random.randint(10, 21, 11)
mon = list(range(1, 12))

i = 0
while i < len(mon):
    i += 1
    if i == 0:
        continue
    else:
        tip_amount = price[i - 1] * (tipsPercent[i - 1] / 100)
        total = price[i - 1] 

        choice = np.random.choice([-1, 0, 1], p=[0.2, 0.2, 0.6]) 
        tip_amount = choice * abs(tip_amount)
        tip_amount = round(tip_amount)  

        tempInf = str(total).replace('.', ',')
        
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(tempInf))
        sh.sheet1.update(('C' + str(i)), str(tip_amount))
        print(tempInf)

```

- Данные перезаписались:

![image](https://github.com/WENEEDAPLAN/ALabs2/assets/117916056/5b704a1d-3677-4445-ac74-864f75f905e1)
![image](https://github.com/WENEEDAPLAN/ALabs2/assets/117916056/14efc8ac-dcb6-47db-a538-1cf645947c21)


## Задание 3
### Настройте на сцене Unity воспроизведение звуковых файлов, описывающих динамику изменения выбранной переменной. Например, если выбрано здоровье главного персонажа вы можете выводить сообщения, связанные с его состоянием.
Ход работы:
- Если чаевые положительны, то звук хороший
- Если чаевые отрицательны, то звук плохой
- Если чаевые равны нулю, то звук нейтральный
  
```

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class SoundScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string, float> dataSet = new Dictionary<string, float>();
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
        if (!dataSet.ContainsKey("Mon_" + i.ToString())) return;

        if (dataSet["Mon_" + i.ToString()] > 0 &
            statusStart == false &
            i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] == 0 &
            statusStart == false &
            i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] < 0 &
            statusStart == false &
            i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest currentResp =
            UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1kcRVGb_vZ2B16U-F8HpCmbHDZ4kvN33Ao8mXHURY8wQ/values/A1:Z100?key=AIzaSyBVhYdPRiMWe6Bh0AyNNuaXbcqQxtLvjaY");
        yield return currentResp.SendWebRequest();
        string rawResp = currentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
        }
    }

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
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
}

```

https://github.com/WENEEDAPLAN/ALabs2/assets/117916056/e089cc64-a967-4b80-ad00-bf14dd703dd8


## Выводы
Во время данной лабораторной работы я приобрел навык заполнения Google Sheets с использованием Python. Также я изучил работу со звуками в Unity. Я успешно подключил необходимые сервисы, следуя предоставленной инструкции, и адаптировал их под свои переменные, предварительно их описав

## Powered by
**BigDigital Team: Denisov | Fadeev | Panov**
