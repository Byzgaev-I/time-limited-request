# Домашнее задание «Реализация обработки запроса с ограничением по времени»


### Нужно выполнить и отправить на проверку задания  


### Инструкция по выполнению  
### Реализуйте Go-программу, которая:  
· напишите Go-программу, имитирующую обработку заказов (например, 5 штук), где каждый заказ запускается в своей горутине;  
· Используйте контексты с таймаутами:  
· для каждой горутины — context.WithTimeout(ctx, 3time.Second);  
· для всей системы — context.WithTimeout(context.Background(), 10time.Second).  
· корректно обрабатывает отмену по таймауту или сигналу от родительского контекста;  
· выводит в консоль:  
· начало обработки заказа;  
· успешное завершение;  
· отмену по таймауту.  

### Условия  
### Используйте только стандартную библиотеку Go:  
· context, time, sync, fmt, math/rand, os/signal, net/http;  
Обработка заказов должна запускаться через go run без пользовательского ввода;  
Для имитации обработки можно использовать time.Sleep и генерацию случайных задержек;  
Примените каналы для передачи результатов выполнения заказов;  
Используйте sync.WaitGroup для синхронного завершения всех горутин;  
Комментарии в коде — не обязательны, но приветствуются.  

### Чек-лист самопроверки  
Перед сдачей задания убедитесь, что:  
· каждый заказ запускается в отдельной горутине;  
· у каждой горутины настроен контекст с таймаутом 3 секунды;  
· для всей системы задан контекст с таймаутом 10 секунд;  
· реализована корректная обработка отмены по контексту;  
· используется sync.WaitGroup для ожидания завершения;  
· используется канал для передачи результатов;  
· в консоли отображается: начало обработки, завершение или отмена;  
· программа завершается корректно, без сбоев и утечек.  
  
### Как сдать задание  
Подготовьте .go-файл (или несколько).  
Назовите файл: GOCONTEXT-XX_ФамилияИмя (например: GOCONTEXT-07_ИвановСергей)  
Загрузите файл в облако и откройте доступ на просмотр и комментирование.  
Вставьте ссылку в поле «Ссылка на решение» в LMS.  
Добавьте комментарий эксперту при необходимости.  

### Критерии оценки  
Зачёт:  
· используется context.WithTimeout для всей системы и для отдельных горутин;  
· реализована отмена задач при превышении времени выполнения;  
· используются каналы для передачи или сбора результатов обработки;  
· применяется sync.WaitGroup для корректного ожидания завершения всех горутин;  
· код структурирован, читаем и разбит на логические части;  
· программа устойчива к ошибкам и не паникует.  


### Выполнение:  

### Система обработки заказов с Context 

## Описание
Система эмулирует обработку заказов интернет-магазина с использованием горутин и контекстов Go.

**Параметры:**
- Таймаут на заказ: 3 секунды
- Таймаут системы: 10 секунд
- Количество заказов: 5
- Режим обработки: параллельный

### Требования

- Go 1.18+
- Стандартная библиотека Go

### Установка и запуск

```bash
git clone <repository-url>
cd order-processing-system
go mod init order-processing
go run GOCONTEXT-78_БызгаевАлександр.go
```
### Структура проекта  
  
order-processing-system/  
├── GOCONTEXT-78_БызгаевАлександр.go  
├── go.mod  
└── README.md  
  
### Технологии  

### Использую пакеты:    

- context - управление жизненным циклом операций    
- sync - синхронизация горутин через WaitGroup  
- time - работа с таймерами и задержками  
- fmt - форматированный вывод  
- math/rand - генерация случайных значений  
- strings - операции со строками  

### Основные функции  
### processOrder  

- Принимает контекст с таймаутом 3 секунды    
- Имитирует обработку заказа (1-5 секунд)  
- Отправляет результат в канал  
- Обрабатывает отмену по контексту  

### main  
  
- Создает родительский контекст с таймаутом 10 секунд    
- Запускает 5 горутин для обработки заказов  
- Синхронизирует завершение через WaitGroup  
- Собирает результаты через буферизованный канал  
- Выводит статистику обработки  

### Алгоритм работы  

### Инициализация родительского контекста (10 сек)  
- Генерация 5 заказов  
- Запуск 5 горутин параллельно  
- Каждая горутина получает дочерний контекст (3 сек)  
- Имитация обработки случайным таймером (1-5 сек)  
- Обработка через select: успех или таймаут  
- Отправка результата в буферизованный канал  
- Ожидание завершения всех горутин (WaitGroup)  
- Сбор и вывод статистики  
- Корректное завершение программы  
  
```bash
go run GOCONTEXT-78_БызгаевАлександр.go
```

### Сам готовый код:  

```go
package main

import (
	"context"
	"fmt"
	"math/rand"
	"strings"
	"sync"
	"time"
)

type Order struct {
	ID       int
	Product  string
	Quantity int
}

type Result struct {
	OrderID int
	Success bool
	Message string
}

func processOrder(ctx context.Context, order Order, results chan<- Result, wg *sync.WaitGroup) {
	defer wg.Done()

	fmt.Printf("[СТАРТ] Заказ #%d: начало обработки (%s, %d шт.)\n",
		order.ID, order.Product, order.Quantity)

	processingTime := time.Duration(1+rand.Intn(5)) * time.Second
	done := make(chan bool)

	go func() {
		time.Sleep(processingTime)
		done <- true
	}()

	select {
	case <-done:
		result := Result{
			OrderID: order.ID,
			Success: true,
			Message: fmt.Sprintf("Успешно обработан за %.1f сек", processingTime.Seconds()),
		}
		fmt.Printf("[УСПЕХ] Заказ #%d: %s\n", order.ID, result.Message)
		results <- result

	case <-ctx.Done():
		result := Result{
			OrderID: order.ID,
			Success: false,
			Message: fmt.Sprintf("Отменён: %v", ctx.Err()),
		}
		fmt.Printf("[ОТМЕНА] Заказ #%d: %s (планировалось %.1f сек)\n",
			order.ID, result.Message, processingTime.Seconds())
		results <- result
	}
}

func main() {
	rand.Seed(time.Now().UnixNano())

	fmt.Println(strings.Repeat("=", 60))
	fmt.Println("СИСТЕМА ОБРАБОТКИ ЗАКАЗОВ")
	fmt.Println(strings.Repeat("=", 60))
	fmt.Println()

	systemCtx, systemCancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer systemCancel()

	fmt.Println("Таймаут системы: 10 секунд")
	fmt.Println("Таймаут на заказ: 3 секунды")
	fmt.Println()

	orders := []Order{
		{ID: 1, Product: "Ноутбук", Quantity: 1},
		{ID: 2, Product: "Мышка", Quantity: 5},
		{ID: 3, Product: "Клавиатура", Quantity: 2},
		{ID: 4, Product: "Монитор", Quantity: 1},
		{ID: 5, Product: "Наушники", Quantity: 3},
	}

	fmt.Printf("Всего заказов к обработке: %d\n\n", len(orders))

	results := make(chan Result, len(orders))
	var wg sync.WaitGroup
	startTime := time.Now()

	for _, order := range orders {
		wg.Add(1)
		orderCtx, orderCancel := context.WithTimeout(systemCtx, 3*time.Second)

		go processOrder(orderCtx, order, results, &wg)

		go func(cancel context.CancelFunc) {
			time.Sleep(3 * time.Second)
			cancel()
		}(orderCancel)
	}

	go func() {
		wg.Wait()
		close(results)
	}()

	fmt.Println()
	fmt.Println(strings.Repeat("-", 60))
	fmt.Println("РЕЗУЛЬТАТЫ ОБРАБОТКИ:")
	fmt.Println(strings.Repeat("-", 60))
	fmt.Println()

	successCount := 0
	failedCount := 0

	for result := range results {
		if result.Success {
			successCount++
		} else {
			failedCount++
		}
	}

	elapsed := time.Since(startTime)

	fmt.Println()
	fmt.Println(strings.Repeat("=", 60))
	fmt.Println("ИТОГОВАЯ СТАТИСТИКА:")
	fmt.Println(strings.Repeat("=", 60))
	fmt.Printf("Общее время работы: %.2f сек\n", elapsed.Seconds())
	fmt.Printf("Успешно обработано: %d заказов\n", successCount)
	fmt.Printf("Отменено: %d заказов\n", failedCount)
	fmt.Printf("Всего заказов: %d\n", len(orders))
	fmt.Println(strings.Repeat("=", 60))

	select {
	case <-systemCtx.Done():
		fmt.Println("\n[ТАЙМАУТ] Система завершена по таймауту (10 сек)")
	default:
		fmt.Println("\n[OK] Все заказы обработаны в рамках лимита времени")
	}

	fmt.Println("\nСистема завершена корректно\n")
}
```

### Запустил и проверил работу: 

![image](https://github.com/Byzgaev-I/time-limited-request/blob/main/Заказ%20-%201%20.png)

### Проверка 

```bash
go fmt GOCONTEXT-78_БызгаевАлександр.go
go vet GOCONTEXT-78_БызгаевАлександр.go
go run GOCONTEXT-78_БызгаевАлександр.go
```

![image](https://github.com/Byzgaev-I/time-limited-request/blob/main/Проверка.png) 

## Поэтапная проверка

### - Каждый заказ запускается в отдельной горутине  

```go  
  for _, order := range orders {
  wg.Add(1)
  go processOrder(orderCtx, order, results, &wg)
  }
```  
  ### Статус: Выполнено. 5 заказов обрабатываются параллельно в отдельных горутинах.   

  
### - У каждой горутины настроен контекст с таймаутом 3 секунды    

```go
  orderCtx, orderCancel := context.WithTimeout(systemCtx, 3*time.Second)
```
 ### Статус: Выполнено. Каждая горутина получает дочерний контекст с таймаутом 3 сек.  

  
### - Для всей системы задан контекст с таймаутом 10 секунд  
  
  systemCtx, systemCancel := context.WithTimeout(context.Background(), 10*time.Second)
  defer systemCancel()
  
  ### Статус: Выполнено. Родительский контекст ограничивает время работы всей системы.  
  
### - Реализована корректная обработка отмены по контексту

  ```go
  select {
  case <-done:
  // Успешная обработка
  case <-ctx.Done():
  // Отмена по таймауту
  }
  ```

  ### Статус: Выполнено. Select обрабатывает оба сценария: успешное завершение и отмену.  

  
### - Используется sync.WaitGroup для ожидания завершения
```go
  var wg sync.WaitGroup
  wg.Add(1)
  defer wg.Done()
  wg.Wait()
```
  
 ### Статус: Выполнено. WaitGroup гарантирует завершение всех горутин перед закрытием канала.  
  
### - Используется канал для передачи результатов  

```go
  results := make(chan Result, len(orders))  
  results <- result  
  for result := range results { ... }
``` 
 ### Статус: Выполнено. Буферизованный канал собирает результаты от всех горутин.  
  
### - В консоли отображается: начало обработки, завершение или отмена  

  [СТАРТ] Заказ #1: начало обработки  
  [УСПЕХ] Заказ #1: Успешно обработан  
  [ОТМЕНА] Заказ #2: Отменён  

###  Статус: Выполнено. Все этапы обработки логируются.  

### Программа завершается корректно, без сбоев и утечек.  

```go
defer systemCancel()
defer wg.Done()
close(results)
```

### Статус: Выполнено. Все ресурсы освобождаются корректно через defer и close.

























































































