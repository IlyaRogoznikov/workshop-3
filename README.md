# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #3 выполнил:
- Рогозников Илья Алексеевич
- НМТ233719
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

## Задание 0
### 
Ход работы: 

шаг1:
Создать пустой проект на Unity

шаг 2: 
Скачайть папку с ML агентом. В созданный проект добавьть ML Agent, выбрав Window - Package Manager - Add Package from disk. Последовательно добавьть .json – файлы: o ml-agents-release_19 / com,unity.ml-agents / package.json o ml-agents-release_19 / com,unity.ml-agents.extensions / package.json
![image](https://github.com/user-attachments/assets/c6ea1512-ed39-4a97-86ac-00b62b37098b)
шаг 3:
Далее запускаем Anaconda Prompt для возможности запуска команд через консоль. Пишем серию команд для создания и активации нового ML-агента, а также для скачивания необходимых библиотек: o mlagents 0.28.0; o torch 1.7.1
![image](https://github.com/user-attachments/assets/727fcb88-3c85-4697-8720-197c49da1b85)
![image](https://github.com/user-attachments/assets/38c77e35-c7af-4dea-af9a-6fc082eba60c)
![image](https://github.com/user-attachments/assets/49e194cf-7ac7-4a8c-a74d-c6dab58bccdf)
![image](https://github.com/user-attachments/assets/e9fcafab-297a-45d8-9041-c4f6e23cd8dd)

- Создайте на сцене плоскость, куб и сферу так, как показано на рисунке ниже. Создайте простой C# скрипт-файл и подключите его к сфере:
![image](https://github.com/user-attachments/assets/c4b1eeab-700b-45c9-b278-60f6db857a56)
- В скрипт-файл RollerAgent.cs добавьте код, опубликованный в материалах лабораторных работ – по ссылке.
  ![image](https://github.com/user-attachments/assets/434f851c-df11-4360-b27d-c3085d09ed2c)
```py
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }
    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);

        if(distanceToTarget < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }
}


```
- Объекту «сфера» добавить компоненты Rigidbody, Decision Requester, Behavior Parameters и настройть их.
  ![image](https://github.com/user-attachments/assets/3297ceda-b693-491b-a9bb-69f301ff0cab)
- В корень проекта добавьте файл конфигурации нейронной сети. Запустить работу ml-агента
![image](https://github.com/user-attachments/assets/68af41fc-c400-48e0-b882-48ce73887b5a)
- Сделайте 3, 9, 27 копий модели «Плоскость-Сфера-Куб», запустите симуляцию сцены и наблюдайте за результатом обучения модели
  ![image](https://github.com/user-attachments/assets/852f63b3-4a3f-46a7-b1e8-7577f01f7702)
- Вывод:
Чем больше экземпляров модели «Плоскость-Сфера-Куб» мы используем, тем выше точность, которую мы достигаем, и тем быстрее это происходит за счет меньшего числа итераций.
## Задание 1
### Найдите внутри C# скрипта “коэффициент корреляции ” и сделать выводы о том, как он влияет на обучение модели
Коэффициент корреляции влияет на точность соответствия между эталонными значениями, которых необходимо достичь в процессе обучения, и фактическими результатами, полученными агентами. При увеличении коэффициента до значения 1.42 агенты будут меньше соответствовать эталонным результатам, но получать больше наград. Наоборот, при уменьшении коэффициента агенты будут действовать более точно в соответствии с эталоном, но их награды за действия будут снижаться.
## Задание 2
### Изменить параметры файла yaml-агента и определить какие параметры и как влияют на обучение модели. Привести описание не менее трех параметров
`trainer_type: ppo` — тип обучения, основанный на методе Proximal Policy Optimization (PPO), который использует механизм поощрений для оптимизации поведения агента.

- `max_steps` — максимальное количество шагов (steps), которое агент может выполнить за время тренировки. После достижения этого лимита обучение завершается.

- `time_horizon` — максимальное количество шагов, за которое учитываются награды перед обновлением модели. Этот параметр определяет временной диапазон, в течение которого агент запоминает свои действия и их последствия.
## Задание 3
### Приведите примеры, для каких игровых задачи и ситуаций могут использоваться примеры 1 и 2 с ML-Agent’ом. В каких случаях проще использовать ML-агент, а не писать программную реализацию решения?
- Поиск объекта агентом на сцене
Этот сценарий можно использовать, например, для поиска выхода из случайно сгенерированного лабиринта. В игре, где игроку нужно выбраться из лабиринта, временно может появляться существо, которое будет двигаться в сторону выхода, помогая игроку. Подобная механика реализована в игре Terraria, где игрок может встретить фею, показывающую место с сундуком сокровищ.

- Симулятор добычи ресурсов
Агент может использоваться для автоматической добычи ресурсов, выполняя роль игрового бота. Этот сценарий применим в онлайн RPG для создания иллюзии большого количества активных игроков, где боты собирают ресурсы или выполняют другие задачи.

Преимущество использования ML-агентов:
ML-агенты упрощают реализацию задач, где требуется реалистичное и естественное поведение. Стандартный программный код часто работает как машина, что может выглядеть неестественно в игровой среде. В отличие от этого, ML-агенты способны обучаться, адаптироваться, принимать сложные решения и демонстрировать поведение, которое выглядит живым и реалистичным.
## Выводы
В результате выполнения данной работы я освоил навыки работы с ML-агентами, а также научился визуализировать данные, полученные в процессе их обучения.
