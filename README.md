# stm32-HC-SR04-SG90
Управление сервоприводом при помощи ультразвукового датчика. 

#include "mbed.h"
#include "HCSR04.h"
#include "HCSR04Blocking.h"
#include "SG90.h" // Определяем пины для сервопривода и ультразвукового датчика
PwmOut servo(D9);
DigitalOut trig(D7);
DigitalIn echo(D6); // Таймер для измерения времени эхо-сигнала
Timer timer;
const float DISTANCE_THRESHOLD = 30.0;  // Пороговое расстояние для открытия шлагбаума (в сантиметрах)
// Функция для установки угла сервопривода
void set_servo_angle(PwmOut &servo, float angle) {
float pulse_width = 0.001 + (angle / 180.0) * 0.001; // Рассчитаем длительность импульса на основе угла
servo.pulsewidth(pulse_width);
}

// Функция для получения расстояния от ультразвукового датчика
float measure_distance() {
    // Инициализация
    trig = 0;
    wait_us(2); // Ждем 2 мкс
    trig = 1; // Устанавливаем триггер в 1
    wait_us(10); // Держим 10 мкс
    trig = 0; // Устанавливаем триггер обратно в 0

    // Ожидание, пока Echo не станет 1
    while (echo == 0);

    // Измеряем время в микросекундах, пока Echo удерживается в 1
    timer.reset();
    timer.start();
    while (echo == 1);
    timer.stop();

    // Получаем время в микросекундах
    float duration = timer.elapsed_time().count();

    // Расчет расстояния в сантиметрах
    float distance = (duration * 0.0343) / 2;

    return distance;
}

int main() {
    // Настраиваем частоту ШИМ для сервопривода
    servo.period(0.04); // Период 20ms (50Hz)
    
    // Настраиваем UART для вывода данных
    BufferedSerial pc(USBTX, USBRX, 9600);

    while (true) {
        // Измеряем расстояние
        float distance = measure_distance();
        
        // Выводим результат в консоль
        printf("Distance: %.2f cm\r\n", distance);

        // Управляем шлагбаумом на основе измеренного расстояния
        if (distance < DISTANCE_THRESHOLD) {
            // Открываем шлагбаум (угол 90 градусов)
            set_servo_angle(servo, 90.0);
            printf("Barrier opened\r\n");
            wait_us(500000);
        } else {
            // Закрываем шлагбаум (угол 0 градусов)
            set_servo_angle(servo, 275.0);
            printf("Barrier closed\r\n");
        }

        // Задержка между измерениями
        wait_us(500000); // 500 мс
    
    }
}                                                        
