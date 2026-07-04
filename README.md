package interfaces;

public interface Actionable {
    void doAction();      // Включить / Запустить / Поехать
    void stopAction();    // Выключить / Остановить / Тормозить
    boolean isWorking();  // Вкл ли? / Активен ли?
}


package models;

import interfaces.Actionable;

public abstract class BaseEntity implements Actionable {
    // Поля protected (# в UML), чтобы наследники имели к ним прямой доступ
    protected String id;
    protected String name;
    protected boolean status;
    protected double mainMetric; // Ватты / Проценты нагрузки / Пробег и т.д.

    public BaseEntity(String id, String name, double mainMetric) {
        this.id = id;
        this.name = name;
        this.status = false; // По умолчанию всё выключено/остановлено
        this.mainMetric = mainMetric;
    }

    // Абстрактный метод, который каждый наследник переопределит по-своему
    public abstract String getDetails();

    // Геттеры и сеттеры для выполнения принципа инкапсуляции
    public String getId() { return id; }
    public void setId(String id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public boolean isWorking() { return status; }
    public void setStatus(boolean status) { this.status = status; }

    public double getMainMetric() { return mainMetric; }
    public void setMainMetric(double mainMetric) { this.mainMetric = mainMetric; }

    @Override
    public void doAction() { this.status = true; }

    @Override
    public void stopAction() { this.status = false; }
}

package models;

public class ConcreteTypeA extends BaseEntity {
    // Уникальное свойство конкретного подкласса (например: яркость, объем RAM, скорость)
    private int customProperty; 

    public ConcreteTypeA(String id, String name, double mainMetric, int customProperty) {
        super(id, name, mainMetric); // Передаем общие свойства в конструктор родителя
        this.customProperty = customProperty;
    }

    public int getCustomProperty() { 
        return customProperty; 
    }

    public void setCustomProperty(int customProperty) { 
        this.customProperty = customProperty; 
    }

    @Override
    public String getDetails() {
        return "Тип А: " + name + " (ID: " + id + 
               "), Уникальное свойство: " + customProperty + 
               ", Метрика: " + mainMetric + 
               " [" + (status ? "Активен" : "Пассивен") + "]";
    }
}



package logic;

import models.BaseEntity;
import java.util.function.Consumer;
import java.util.function.Predicate;

public class CustomPolicy<T extends BaseEntity> {
    private String policyName;
    private Predicate<T> condition; // Лямбда-условие (тест)
    private Consumer<T> action;     // Лямбда-действие (выполнение)

    public CustomPolicy(String policyName, Predicate<T> condition, Consumer<T> action) {
        this.policyName = policyName;
        this.condition = condition;
        this.action = action;
    }

    public String getPolicyName() { 
        return policyName; 
    }

    @SuppressWarnings("unchecked")
    public boolean apply(BaseEntity entity) {
        try {
            T casted = (T) entity; // Безопасно приводим к конкретному типу данных
            if (condition.test(casted)) {
                action.accept(casted);
                return true; // Возвращаем true, если автоматизация сработала
            }
        } catch (ClassCastException ignored) {
            // Если объект не того подтипа (например, применили сценарий ламп к розетке) — просто пропускаем
        }
        return false;
    }
}


package logic;

import models.BaseEntity;
import java.util.*;
import java.util.stream.Stream;

public class GroupManager {
    // Карта (Мапа). Ключ — имя группы (комната/проект), Значение — список элементов в ней
    private Map<String, List<BaseEntity>> groups = new HashMap<>();
    
    // Список всех загруженных сценариев автоматизации
    private List<CustomPolicy<? extends BaseEntity>> policies = new ArrayList<>();

    public void addEntity(String groupName, BaseEntity entity) {
        if (!groups.containsKey(groupName)) {
            groups.put(groupName, new ArrayList<>());
        }
        groups.get(groupName).add(entity);
    }

    public void addPolicy(CustomPolicy<?> policy) {
        policies.add(policy);
    }

    public BaseEntity getById(String id) {
        for (List<BaseEntity> list : groups.values()) {
            for (BaseEntity e : list) {
                if (e.getId().equals(id)) return e;
            }
        }
        return null;
    }

    // Выполнение сценариев с генерацией отчета на экран (строго по условию)
    public void runAutomation() {
        System.out.println("\n--- Запуск автоматизации ---");
        for (List<BaseEntity> list : groups.values()) {
            for (BaseEntity e : list) {
                for (CustomPolicy<? extends BaseEntity> policy : policies) {
                    if (policy.apply(e)) {
                        System.out.println("Политика '" + policy.getPolicyName() + "' успешно применена к: " + e.getName() + " (ID: " + e.getId() + ")");
                    }
                }
            }
        }
        System.out.println("--- Автоматизация завершена ---\n");
    }

    // Тот самый метод, возвращающий Stream для выполнения аналитики
    public Stream<BaseEntity> getAnalyticsStream() {
        List<BaseEntity> all = new ArrayList<>();
        for (List<BaseEntity> list : groups.values()) { 
            all.addAll(list); 
        }
        return all.stream();
    }

    public void printAll() {
        if (groups.isEmpty()) {
            System.out.println("Данные отсутствуют.");
            return;
        }
        groups.forEach((group, list) -> {
            System.out.println("Группа/Комната/Проект: " + group);
            for (BaseEntity e : list) {
                System.out.println("  " + e.getDetails());
            }
        });
    }
}



package main;

import logic.CustomPolicy;
import logic.GroupManager;
import models.*;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        GroupManager manager = new GroupManager();
        Scanner scanner = new Scanner(System.in);

        while (true) {
            System.out.println("\n===== КОНСОЛЬНОЕ МЕНЮ =====");
            System.out.println("1. Добавить элемент в группу");
            System.out.println("2. Переключить статус (Вкл/Выкл) по ID");
            System.out.println("3. Добавить базовое правило автоматизации");
            System.out.println("4. Добавить правило автоматизации ПО ВАРИАНТУ");
            System.out.println("5. Запустить автоматизацию (выполнить сценарии)");
            System.out.println("6. Выполнить аналитический запрос ПО ВАРИАНТУ");
            System.out.println("7. Показать все элементы (Мониторинг)");
            System.out.println("8. Выход");
            System.out.print("Ваш выбор: ");
            
            int menu = scanner.nextInt(); 
            scanner.nextLine(); // Очистка буфера после чтения числа

            if (menu == 1) {
                System.out.print("Введите имя группы (Комната/Проект): "); String group = scanner.nextLine();
                System.out.print("Тип (1 - Тип А, 2 - Тип Б...): "); int type = scanner.nextInt(); scanner.nextLine();
                System.out.print("Введите уникальный ID: "); String id = scanner.nextLine();
                System.out.print("Введите имя: "); String name = scanner.nextLine();
                System.out.print("Главная метрика (Вт / % / км): "); double metric = scanner.nextDouble();

                if (type == 1) {
                    manager.addEntity(group, new ConcreteTypeA(id, name, metric, 50));
                }
                System.out.println("Элемент добавлен.");

            } else if (menu == 2) {
                System.out.print("Введите ID: "); String id = scanner.nextLine();
                BaseEntity e = manager.getById(id);
                if (e != null) {
                    if (e.isWorking()) e.stopAction(); else e.doAction();
                    System.out.println("Статус изменен! Текущее состояние: " + (e.isWorking() ? "Работает" : "Остановлен"));
                } else { 
                    System.out.println("Объект с таким ID не найден."); 
                }

            } else if (menu == 3) {
                // Базовое общее правило: Сброс метрики у выключенного прибора (Сброс фантома)
                manager.addPolicy(new CustomPolicy<BaseEntity>("Сброс фантома", 
                    entity -> !entity.isWorking() && entity.getMainMetric() > 0, 
                    entity -> entity.setMainMetric(0.0)
                ));
                System.out.println("Базовое правило добавлено в систему.");

            } else if (menu == 4) {
                // ================================================================
                // МЕСТО ДЛЯ ИЗМЕНЕНИЙ: СЦЕНАРИЙ ПО ТВОЕМУ ИНДИВИДУАЛЬНОМУ ВАРИАНТУ
                // ================================================================
                // Пример: Если это объект типа ConcreteTypeA и его свойство > 40 -> выключить его
                manager.addPolicy(new CustomPolicy<ConcreteTypeA>("Правило Варианта", 
                    e -> e.getCustomProperty() > 40 && e.isWorking(), // <--- ТВОЕ УСЛОВИЕ
                    e -> e.stopAction()                              // <--- ТВОЕ ДЕЙСТВИЕ
                ));
                System.out.println("Индивидуальное правило автоматизации добавлено.");

            } else if (menu == 5) {
                manager.runAutomation();

            } else if (menu == 6) {
                // ================================================================
                // МЕСТО ДЛЯ ИЗМЕНЕНИЙ: АНАЛИТИКА STREAM API ПО ТВОЕМУ ВАРИАНТУ
                // ================================================================
                System.out.println("\n--- Результат аналитического запроса ---");
                
                // ШАБЛОН 1: Подсчет суммы (например, общая мощность включенных)
                double sum = manager.getAnalyticsStream()
                        .filter(e -> e.isWorking())
                        .mapToDouble(e -> e.getMainMetric())
                        .sum();
                System.out.println("Суммарный показатель работающих элементов: " + sum);

                /* // ШАБЛОН 2: Вывод ТОП-3 элементов по убыванию метрики
                System.out.println("ТОП-3 элементов:");
                manager.getAnalyticsStream()
                        .sorted((e1, e2) -> Double.compare(e2.getMainMetric(), e1.getMainMetric()))
                        .limit(3)
                        .forEach(e -> System.out.println(e.getName() + " -> " + e.getMainMetric()));
                */

            } else if (menu == 7) {
                manager.printAll();

            } else if (menu == 8) {
                System.out.println("Завершение сессии. Программа закрыта.");
                break;
            }
        }
        scanner.close();
    }
}
